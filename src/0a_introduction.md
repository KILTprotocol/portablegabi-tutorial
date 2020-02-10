# Introduction

The portablegabi library provides an easy to use API for signing, verifying and revoking json objects.
The intend of this library is to enable the claimer, who possesses the signed json object, to prove to a third party, called verifier, that specific properties are present inside the json object and that the object was signed by a trusted attester.
The important benefits of portablegabi are that the claimer stays anonymous during the verification.
The verifier is not able to link two verification sessions to a single identity and learns nothing about the user besides the shared properties.

## Terminology

**Attester**: An entity which signs json objects.

**Claim**: A json object.

**Claimer**: An entity which is in possession of a credential.

**Credential**: A json object which can be used to create presentation.

**Presentation**: A json object which contains properties and a signature from an attester.

**Verifier**: An entity which request sign properties from the claimer.

**Accumulator**: The whitelist which contains all non revocation witnesses and can be used to prove that the credential was not revoked.

## Technical overview

The cryptographic primitives of Portablegabi are implemented in [Gabi](https://github.com/privacybydesign/gabi) which is maintained by the [privacy by design foundation](https://privacybydesign.foundation/en/) and also used inside [IRMA](https://www.irmacard.org).
Gabi makes use of CL-Signatures and is based on the Idemix Specification.

Portablegabi provides a protocol for [attestation](2_attestation.md) and [verification](3_verification.md) of claims.
The main goals of Portablegabi are *selective disclosure* and *unlinkability*.
*Selective disclosure* enables the claimer to only present a subset of the information contained inside his attested json object.
The *unlinkability* feature hinders the verifier to link two verification sessions of the same claimer together.
The claimer can interact with the same verifier multiple times without the verifier being able to tell if he talked to the same claimer.
It is important to note that this is without considering the revealed attributes.
If the claimer reveals attributes that uniquely identify him, the verifier will be able to link multiple sessions together.

It also provides a scheme to support revocation of attestations using a distributed ledger.
Each attestation contains a non revocation witness which proves that the attestation is not revoked.
Every witness is by default contained inside a mathematical accumulator which is written on a blockchain.
If the attester wishes to revoke an attestation he removes he witness from the accumulator and updates the blockchain with the new accumulator.

## Architecture of Portablegabi

Portablegabi consists of a part written in [Go](https://golang.org) which wraps the Gab library and can be compiled to [WASM](https://webassembly.org).
The second part is a Typescript layer which uses the WASM-Module and provides an API for attestation, verification and revocation.
