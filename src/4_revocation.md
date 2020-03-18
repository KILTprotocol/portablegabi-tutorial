# Revocation

An attester can revoke any credential they attested.
This is achieved by using a witness which is contained inside a credential and a whitelist containing all non-revoked witnesses.
If an attester revokes a credential, they remove the associated witness from the whitelist and publish a new version of this whitelist.
Note that **witnesses are added to the whitelist implicitly**.
Therefore, adding witnesses to the whitelist requires no change.
Since this whitelist is implemented using accumulators, it is called _accumulator_.
Further documentation on how this accumulator works can be found in the [IRMA docs](https://irma.app/docs/revocation/#cryptography).

## Example

In order to revoke a credential, the attester needs their key pair, the witness of credential they want to revoke (created in `issueAttestation`) and the accumulator.

```js
const portablegabi = require('@KILTprotocol/portablegabi')

const attester = new portablegabi.Attester(pubKey, privKey)
const accumulator0 = await attester.createAccumulator()

// Issue attestations and store witnesses.
const accumulator1 = await attester.revokeAttestation({
  accumulator: accumulator0,
  // The list of witnesses associated with the credentials which should get revoked.
  witnesses: [witness0, witness2, witness3]
})
// Publish accumulator1.
```

After an attester publishes a new accumulator, all claimers should update their credential attested by this specific attester to their newest available accumulator.
In order to update the credential, **the claimer needs the complete history of all new accumulators since their last update**.

```ts
const claimer = await portablegabi.Claimer.buildFromMnemonic('siege decrease quantum control snap ride position strategy fire point airport include')
let credential = () => {
    // Request an attestation from an attester.
    // Build a credential.
    // ...
    return credential
}()

// How to update your credential?
// The Credential is updated to accumulator 55, the newest accumulator has index 59.
const newCredential = await credential.update({
    attesterPubKey: attestersPublicKey,
    accumulators: [accumulator56, accumulator57, accumulator58, accumulator59],
})
```
