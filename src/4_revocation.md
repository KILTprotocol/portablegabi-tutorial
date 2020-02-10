# Revocation

An attester can revoke specific credentials.
This is done using a witness which is contained inside a credential and a whitelist which contains all non revoked witnesses.
If an attester revokes a credential he removes the associate witness from the whitelist and publishes a new version of this whitelist.
Witnesses are added to the whitelist implicitly.
Adding witnesses to the whitelist requires therefore no change.
Since this whitelist is implemented using accumulators, it is called *accumulator*.

## Example

In order to revoke a credential, the attester needs his key pair, the witness for the credential he wants to revoke and the accumulator.

```ts
const attester = new GabiAttester(publicKey, privateKey)
const accumulator0 = await attester.createAccumulator()

// issue attestations and store witnesses...

const accumulator1 = await attester.revokeAttestation({
  accumulator0,
  // the list of witnesses associated with the credentials which should get revoked
  [witness0, witness2, witness23]
})
// publish accumulator1...
```

After a new accumulator has been published, all claimers should update their credential to the newest available accumulator.
In order to update their credential the claimer needs the complete history of all new accumulator since his last update.

```ts
const claimer = await GabiClaimer.buildFromScratch()
let credential = () => {
    // request attestation from attester
    // build credential
    // ...
    return credential
}

// credential is updated to accumulator 55, the newest accumulator has index 59
const accumulators = [accumulator56, accumulator57, accumulator58, accumulator59]
for (let acc in accumulators) {
    credential = await claimer.updateCredential({
        credential,
        attesterPubKey: attestersPublicKey,
        accumulator: acc,
    })
}
```
