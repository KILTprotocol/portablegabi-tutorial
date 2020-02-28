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

1. Connect to the chain and add an accumulator (since there is none by default)
2. Attest a claim
3. Revoke the attested claim from 2. and (automatically) update the accumulator
4. Check out multiple verifications with different timestamps

```javascript
const portablegabi = require("@kiltprotocol/portablegabi");

const privKey = new portablegabi.AttesterPrivateKey(
  '{"XMLName":{"Space":"","Local":""},"Counter":0,"ExpiryDate":1610554062,"P":"iDYKxuFGt1Xv1aqMLaagjrOPX0hjkOlFrKOp4NPnSBHmQ9SFETUX1M43q3jLsGz+UEWFS3+SS9QpP4CTkl3p/w==","Q":"92MJOhwjESn7QohCCY1oBxsToAfccGoKtE3sBoaNxHWoowSiCy8fMG+B1sO5QU+bV3i1xwvVno9o30RcMoXEaw==","PPrime":"RBsFY3CjW6r36tVGFtNQR1nHr6QxyHSi1lHU8GnzpAjzIepCiJqL6mcb1bxl2DZ/KCLCpb/JJeoUn8BJyS70/w==","QPrime":"e7GEnQ4RiJT9oUQhBMa0A42J0APuODUFWib2A0NG4jrUUYJRBZePmDfA62HcoKfNq7xa44Xqz0e0b6IuGULiNQ==","ECDSA":"MHcCAQEEILO+g4uSDheZ6PSLxR7olFzUhZpeO9tQu84hX6UeIevaoAoGCCqGSM49AwEHoUQDQgAEKvmUz3HIZy890jE78CC9V9BuN8taO+L8GjAeS14v0CL7GCFZ1GMnaSZi4WG3mOjJlJ80CnMowIbUT3Fw1TluFw==","NonrevSk":null}'
);
const pubKey = new portablegabi.AttesterPublicKey(
  '{"XMLName":{"Space":"","Local":""},"Counter":0,"ExpiryDate":1610554062,"N":"g6DWNN/cWep9/lCc6gg0tA8wS1y5LgQx2/fM/wMpYJE8MTZ9SJ3y9kjIBAeSb4aY3vsFhRp8aWsEZzAA0Qu0kW4bzyKN1RU7A0tlmkmDetCxu7Gy2zQMHlTg4YkAVxVYAIIIWhHKHrVLzH7zCsuXos1qm/sthByVdEXv4HPjCZU=","Z":"BiDMFSNGKLIcHJY3tmh2vgiW7D3f5g5b+6Bjf0ns3/rPOg8x0BJ+CzqOLQL+loNIomOzBm/Pk36q3pmPPFMfug80AwUlZOvKTrzj29Agq4DF7p4jruElRyZsdGNjlFkVzILFT/9yrXfjD/9DAHXGm6/4unVnwKP4I0j1r9sLYtg=","S":"Bxm9bNpNLZUM6gy74aR0HW2DadFuy/l+MOdZkG2BiFxbTEP24GXBYA3+d1xajplWEm2iLF4w2OeviIpr8VIzDNy6dXRyGcTnGzj6sVeGlR5u3N+8M2XNH1pNEymLQQbUAt3ogYSWiJW88bxHCf3AZiS91XT1Zh3ENCS9NsyGzt8=","G":"Angd7BuIjTeWGsVLGVCtv+5dx1TMEUr/Z5Fhk7OFUNBexY8fuNfzxfeclgSQpC+nyIAFHc3RB+3Fcs2vOSygopVfLEJo9h7dSjtlcxSZ1wE8YNgouHwfVuq4KWixzIk7Le+IeUzNaQNOL9SI3h5mlxJ5QOO2Src+BPQuFjXPSfI=","H":"U1MyQqwl1LrZY5G61Z2ZDM3zWQKv78HOluCrtxCDBsMvYNRLvhbppOhOdsnG3axN5NIH01/R6mlYojBDg9L7xSwR+1QpmHGUbwkemADlUZQ9c98Up1ORKxNW0asQJdPHV4NGqjQbDfJzejdGJwd95scmSpqLNvRTT+L0iW0ln4A=","T":"BEIUJ5pXzFZPeoB3us341EWxwE7HByM4NaPYRS6YVtDcJdz+H9EEKdUcXhUVrJAQ2OZy2FP0+SNvQVk8AxWDiD73tHUUKDnkMoKSkHPnEnsCInGHr4iTYE2zp8/uEBFxNppq5SP9gQOzE2qekGket2co0W/+jKNtg63u1udlZjo=","R":["OpuoX8xEvGaULH7ir3G/W9zBB1gmYN6lllJsk8+QGGQxydbrtoQiFfhU1Tyqm59sq3GIhksiYB6Th6jYq3BIFKVynX993FPYU2HS2dceFk5kvymIx33u2nTyMzFvox2b6IkKHKXfbtx/VWWlVYcywFOAOiQ1Xa7dXDx1ebuGowE=","Jamoy887kQjyTKjHwgFGxOKugcGIxdUhK9pE/nDTFttU6ndo5qm04AVB5n4WUaFurrKlNSIICheAXI10kIy37Ogr1N4Ge/7TbyZ/hXB8DBzoJbD3MVpXblq9hrhEkb+yyJ9uipnKckflQBWGzl+grXV17SWVhd5TKpUrMw1cDYs=","YGogpko2T4xWQjipZN691tpWJYffyX5evzh2EJAZSpP3evnMbro0Et5Bk+2NY9yt/GoJW8qkVkwEdaYU0jQiGS27F3aJ5e00VOCnZ6bIXJKgcTTxqc5c9NrpJVWNX9n5G590OVTNqlLUOFw3/mIY26A2MKxsa56j2K0V4IM0FI4=","Jca8++mT6d93MK0S8Fb6rtu7TpV9TGqM0mSvO0JKuyRvEro3anRbvZ8sHRLt2q2ePIyCQHz2eUc4iJ1vQLnzMxVavQ3xS5AAS27Tw+xM64JhWV6BFDqZgaEcu22jEi+Rrjjqss2nmC6CQYJZt5g5P0dXGV2JKDcrUaGCtzc4cNE=","ZIV6MWKglRL5B9vv5RmBigbieiuebmy/mcpycXlyQcoZEeNCzuGs/JgRnGr05umbcsQ5ZNSS3TKiL5CM/Z4fanuSu6jNnVoHvSkxI3x28ZpMV8C43CXkS6smmiZP+2SSL419Q247ZbP04T5wHcZ6GooCLxnfx5DeEtRze3UU1Wk=","IbwQtY9iF7C/rNKkTilHP5jEj9r3aI1tRVU9WeMzE9yxrE0mggzpcoCM0lJFLcqVyWhKD3PWssuXwNiLJipUL+sH/u8Qk8Bu6sv/USlUU7sgSJ4akl2Lp+5oYSkzHiZTeJtLg0OVGZnka3pGxzg0ihkkT6Bdk8K2OicTNxlHzgI=","ZQ9/qIgvOx/8dyXlAFeZH+2lriSPaj/NDzPCxR9sXqBYJskSkSrdGogxP2RZeAGyDh7NvwUtvBDQ/vLKz/O3ANPUOnaRx1n4uBF+uBdt0h3Ml/DckhL5k2+nHQsnZWPFxkdpatCIFWcvYuldx+gXLePBaRmNnKMoxAgT+tJnJcw=","ZGfBOqHujseUhLZdfs8kq+/kmG3yMwUAmQrGgTdNej8npNsOyD/Am/SoPdSjpr1enuMgBzva/bjn3/z8nncpia65+v9Pn5831UuFp8h53/1WaEHvN/yctnIKb8k1IRtPlSvnfq7qwC/sIGvHq+ZTj3/ie57rTSkSMrmdFL8PMM0=","TM38T4ekWiNWICCgry7GsppfVt2ImPv4SL//f/J3beP34K1afJCsHk50XJwi8qyMz8HqEVK2sWvMQzJ8Amct4sAfRYIZNmqH7mSR7LwIXvihwv1dUlJv2R7MLTjEGkEnJHE5cCR0K5GxjeQSSgNHAu33MOth3ipsK9ZmF+slSkI=","YwMb/IVn2NsA4y8ZiiBxCWoOg0tsqyYKTakxDZnRhw+wHwhnA3+T87X4tOSAx+dYlmtj3UQzUAeFRYztr2YTrF2boS/YFeAiVh6swPgFOScvmOuf5O4fJn7z+iXr+ivgFccswxBhxqa9MdF8ReqHaVouj8LLyk33fZgWduwfnA=="],"EpochLength":432000,"Params":{"LePrime":120,"Lh":256,"Lm":256,"Ln":1024,"Lstatzk":80,"Le":597,"LeCommit":456,"LmCommit":592,"LRA":1104,"LsCommit":593,"Lv":1700,"LvCommit":2036,"LvPrime":1104,"LvPrimeCommit":1440},"Issuer":"","ECDSA":"MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAEKvmUz3HIZy890jE78CC9V9BuN8taO+L8GjAeS14v0CL7GCFZ1GMnaSZi4WG3mOjJlJ80CnMowIbUT3Fw1TluFw==","NonrevPk":null}'
);

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

## Example 1: Complete process for with multiple credentials to showcase timestamp

// TODO: Add me


