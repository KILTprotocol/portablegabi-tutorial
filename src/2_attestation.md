# Attestation

During the attestation the attester signs a claim, sends the signature over to the claimer who then builds his credential using the claim and the signature.

```mermaid
sequenceDiagram
    participant A as Attester
    participant C as Claimer

    note left of A: The attester chooses two random numbers (context and nonce) and sends them to the claimer.
    A->>C: Initiate Attestation
    note right of C: The claimer commits on his master secret and send his claim to the attester.
    C->>A: Attestation Request
    note left of A: The attester validates and signs the claim and sends the signature back to the claimer.
    A->>C: Attestation Response
    note right of C: The claimer builds his credential using the signature.
```

Before an attester can create attestations, she has to generate a key pair and publish her public key.

```ts
const portablegabi = require("@KILTprotocol/portablegabi")
// Build a new attester.
// Generating a new key pair will take around 10-30 minutes.
// const attester = await portablegabi.GabiAttester.create(365, 70)

// for this example you could use the provided keys. Note: never use those keys in production!!!
const attester = await new portablegabi.GabiAttester(pubKey, privKey)

// create a new accumulator (which is used for revocation)
let accumulator = await attester.createAccumulator()
console.log("Accumulator: ", accumulator.valueOf())

// Build a new claimer and generate a new master key.
// const claimer = await portablegabi.GabiClaimer.create()
// or use a mnemonic for:
const claimer = await portablegabi.GabiClaimer.buildFromMnemonic('siege decrease quantum control snap ride position strategy fire point airport include')
```

After the attester and claimer have both generated their keys, the attestation session can be initiated by the attester.

```js
// The attester initiates the attestation session:
const {
    message: startAttestationMsg,
    session: attestationSession,
} = await attester.startAttestation()

// the claimer answers with an attestation request
const claim = {
    age: 15,
    name: "George",
}

const {
    message: attestationRequest,
    session: claimerSession,
} = await claimer.requestAttestation({
    // The received attestation message
    startAttestationMsg,
    // The claim which should get attested.
    claim,
    // The public key of the attester.
    attesterPubKey: attester.publicKey,
})

// The attester should check the claim he is about to attest.
const receivedClaim = attestationRequest.getClaim()
// do checks on receivedClaim

// if everything checks out the attester issues an attestation.
const {
    // the attestation should be send over to the claimer.
    attestation,
    // the witness should be stored for later revocation.
    witness
} = await attester.issueAttestation({
    attestationSession,
    attestationRequest,
    // The update is used to generate a non revocation witness
    accumulator,
})
console.log("Witness: ", witness.valueOf())

// After the claimer has received his attestation he can build his credential
const credential = await claimer.buildCredential({
    claimerSession,
    attestation,
})
console.log("Credential: ", credential.valueOf())
```

After the attestation session has finished the attester has a witness which can be used to revoke the attestation and the claimer received a credential with which he can generate presentations for a validator.
