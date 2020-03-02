# Set up 

You are required to have an active [substrate](https://www.parity.io/substrate/) blockchain implementing the [`portablegabi-pallet`](https://github.com/KILTprotocol/portablegabi-pallet). You could clone and set up this [`portablegabi-node`](https://github.com/KILTprotocol/portablegabi-node) basic substrate template implementing the portablegabi-pallet.

### Build the chain

```bash
git clone git@github.com:KILTprotocol/portablegabi-node.git
cd portablegabi-node
./scripts/init.sh
cargo build --release
```

### Start the chain

```bash
./target/release/node-portablegabi --dev
```

# Run examples

The purpose of the chain is to store each attester's accumulator and give access to old revisions as well as these are required when updating older credentials.
Therefore, we have added some chain functionality to both the credential and attester classes.

## Example 1: Complete process for single credential with revocation

 In the following we will run a complete exemplary chain process: 

1. Connect to the chain and add an accumulator
2. Attest a claim
3. Revoke the attested claim from 2. and (automatically) update the accumulator
4. Check out multiple verifications with different timestamps

```javascript
const portablegabi = require("@kiltprotocol/portablegabi");

const privKey = new portablegabi.AttesterPrivateKey('...');
const pubKey = new portablegabi.AttesterPublicKey('...');

async function exec() {
  /** (1) Chain phase */
  // (1.1) connect to the chain
  const chain = await portablegabi.connect({
    pgabiModName: "portablegabiPallet"
  });
  console.log("Connected to chain");

  // (1.2) create Alice identity
  const attester = await portablegabi.AttesterChain.buildFromURI(
    pubKey,
    privKey,
    "//Alice"
  );

  // (1.3) create a fresh accumulator
  const accumulator0 = await attester.createAccumulator();

  // (1.4) put the accumulator on chain
  await attester.updateAccumulator(accumulator0);

  // check whether it has actually been added to chain
  // we need to wait for next block since updating the accumulator is a transaction
  await chain.waitForNextBlock();
  console.log(
    "Latest accumulator === accumulator0? Expected true, received",
    (await chain.getLatestAccumulator(attester.address)).valueOf() ===
      accumulator0.valueOf()
  );

  /** (2) Attestation phase */
  // (2.1) the attester initiates the attestation session:
  const {
    message: startAttestationMsg,
    session: attestationSession
  } = await attester.startAttestation();

  // (2.2) the claimer answers with an attestation request
  const claimer = await portablegabi.Claimer.buildFromMnemonic(
    "siege decrease quantum control snap ride position strategy fire point airport include"
  );
  const claim = {
    name: "George Ericson",
    age: 24,
    drivers_license: {
      id: "127128204193",
      category: "B2",
      licensing_authority: "Berlin A52452"
    }
  };
  const {
    message: attestationRequest,
    session: claimerSession
  } = await claimer.requestAttestation({
    // the received attestation message
    startAttestationMsg,
    // the claim which should get attested.
    claim,
    // the public key of the attester.
    attesterPubKey: attester.publicKey
  });

  // (2.3) the attester issues an attestation
  const {
    // the attestation should be sent over to the claimer
    attestation,
    // the witness should be stored for later revocation
    witness
  } = await attester.issueAttestation({
    attestationSession,
    attestationRequest,
    // the update is used to generate a non-revocation witness
    accumulator: accumulator0
  });
  const credential = await claimer.buildCredential({
    claimerSession,
    attestation
  });

  /** (3) Revocation phase */

  // revoke attestation and receive new accumulator whitelist
  const accumulator1 = await attester.revokeAttestation({
    witnesses: [witness],
    accumulator: accumulator0
  });
  // check whether accumulator1 is latest accumulator on chain
  await chain.waitForNextBlock();
  console.log(
    "Latest accumulator === accumulator1? Expected true, received",
    (await chain.getLatestAccumulator(attester.address)).valueOf() ===
      accumulator1.valueOf()
  );

  /** (4) Verification phase */
  // get the exact timestamp of the revocation for simplicity
  const timeAtRev = await accumulator1.getDate(attester.publicKey);

  // (4.1) verifier sends nonce and context to claimer + requests disclosed attributes
  // note: the requested timestamp at revocation
  const {
    session: verifierSession,
    message: presentationReq
  } = await portablegabi.Verifier.requestPresentation({
    requestedAttributes: ["age", "drivers_license.category"],
    reqUpdatedAfter: timeAtRev
  });

  // (4.2) claimer builds presentation with revoked credential
  // note: he needs to update as the credential was build before timeAtRev
  const presentation = await claimer
    .buildPresentation({
      credential,
      presentationReq,
      attesterPubKey: attester.publicKey
    })
    .catch(() => {
      console.log(
        "Caught expected error when building presentation due to credential not matching the required timestamp"
      );
      const updatedCredential = credential.updateFromChain({
        attesterPubKey: attester.publicKey,
        attesterChainAddress: attester.address
      });
      return claimer.buildPresentation({
        credential: updatedCredential,
        presentationReq,
        attesterPubKey: attester.publicKey
      });
    });

  // (4.3) verifier checks presentation for non revocation, valid data and matching attester's public key
  // expect success because credential is still valid in accumulator0
  const {
    verified: verifiedRev0
  } = await portablegabi.Verifier.verifyPresentation({
    proof: presentation,
    verifierSession,
    attesterPubKey: attester.publicKey,
    latestAccumulator: accumulator0
  });
  // expect failure because credential is invalid in accumulator1
  const {
    verified: verifiedRev1
  } = await portablegabi.Verifier.verifyPresentation({
    proof: presentation,
    verifierSession,
    attesterPubKey: attester.publicKey,
    latestAccumulator: accumulator1
  });
  console.log(
    "Cred verified w/ timestamp at revocation and old accumulator? Expected true, received",
    verifiedRev0
  );
  console.log(
    "Cred verified w/ timestamp at revocation and new accumulator? Expected false, received",
    verifiedRev1
  );
}
exec()
  .catch(e => console.log(e))
  .finally(() => process.exit(1));
```
