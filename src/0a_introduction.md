# Introduction

The Portablegabi library provides an easy to use API for signing, verifying and revoking JSON objects.
The intent of this library is to enable the claimer, who possesses the signed JSON object, to prove to a third party, called verifier, that specific properties are present inside the JSON object and that the object was signed by a trusted attester.
The important benefits of Portablegabi are that the claimer stays anonymous during the verification.
The verifier is not able to link two verification sessions to a single identity and learns nothing about the user besides the shared properties.

## Terminology

**Attester**: An entity which signs JSON objects.

**Claim**: A JSON object.

**Claimer**: An entity which is in possession of a credential.

**Credential**: A JSON object which can be used to create a verifiable presentation.

**Presentation**: A JSON object which contains properties and a signature from an attester.

**Verifier**: An entity which requests signed properties from the claimer in form of a presentation.

**Accumulator**: A whitelist which contains all non-revocation witnesses and can be used to prove that a credential was not revoked.

**Witness**: A unique number tied to a credential which is required to proof whether it has been revoked or not.

## Technical overview

The cryptographic primitives of Portablegabi are implemented in [Gabi](https://github.com/privacybydesign/gabi) which is maintained by the [privacy by design foundation](https://privacybydesign.foundation/en/) and also used inside [IRMA](https://www.irmacard.org).
Gabi makes use of CL-Signatures and is based on the Idemix Specification.

Portablegabi provides a protocol for [attestation](2_attestation.md) and [verification](3_verification.md) of claims.
The main goals of Portablegabi are *selective disclosure* and *unlinkability*.
*Selective disclosure* enables the claimer to only present a subset of the information contained inside his attested JSON object.
The *unlinkability* feature hinders the verifier to link two verification sessions of the same claimer together.
The claimer can interact with the same verifier multiple times without the verifier being able to tell if he talked to the same claimer. Please note that this only holds true if the claimer does not reveal attributes which uniquely identify him. Otherwise, the verifier would be able to link multiple sessions together.

The library also provides a scheme to support revocation of attestations using a distributed ledger.
Each attestation contains a non-revocation witness which proves that the attestation is still valid and has not been revoked.
Every witness is by default contained inside a mathematical accumulator which is written on a blockchain.
If the attester wishes to revoke an attestation, he removes the witness from the accumulator and updates the blockchain with the new accumulator.

## Architecture of Portablegabi

Portablegabi consists of a part written in [Go](https://golang.org) which wraps the Gabi library and can be compiled to [WASM](https://webassembly.org).
The second part is a [Typescript](http://www.typescriptlang.org/index.html) layer which uses the WASM-Module and provides an API for attestation, verification and revocation. It provides usage of with and without a [Substrate](https://www.parity.io/substrate/)-based blockchain implementing the [`portablegabi-pallet`](https://github.com/KILTprotocol/portablegabi-pallet). 

TODO: Add section about unproven crypto.