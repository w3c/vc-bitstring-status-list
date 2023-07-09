# StatusList2021 Explained

## Authors

- Mahmoud Alkhraishi (Mavennet)
- Manu Sporny (Digital Bazaar)
- Dave Longley (Digital Bazaar)
- Orie Steele (Transmute)
- Mike Prorock (mesur.io)

## Participate

- [Issue tracker](https://github.com/w3c/vc-status-list-2021/issues)
- [Discussion forum](https://lists.w3.org/Archives/Public/public-vc-wg/)

## Introduction

This specification describes a privacy-preserving, space-efficient, and high-performance mechanism for publishing status information such as suspension or revocation of Verifiable Credentials.

## Goals

Provide a location for an issuer of a Verifiable Credential [VC-Data-Model](https://www.w3.org/TR/vc-data-model/) to link to a location where a verifier can check to see if a credential has been suspended or revoked.

## How does it work?

At the most basic level, status information for all verifiable credentials issued by an issuer are expressed as simple binary values. The issuer keeps a bitstring list of all verifiable credentials it has issued. Each verifiable credential is associated with a position in the list. If the binary value of the position in the list is 1 (one), the verifiable credential is revoked; if it is 0 (zero), it is not revoked. The verifier can then check the status of a verifiable credential by looking up the position of the verifiable credential in the bitstring list. This is the basic model used for Revocation and Suspension.

A status list may have multiple states, exposed via a series of status messages. These status messages are intended to be extensible and interoperable with existing status message vocabularies. The status messages are intended to be both human-readable and machine-readable. The status messages are intended to be used by verifiers to determine the status of a verifiable credential.

The Working Group is still discussing the unification of a design between status lists with a single state (such as "revoked" or "suspended") and status lists with multiple states (exposed via a series of status messages). We are seeking implementer feedback on what a unified design should look like from an ease of implementation, privacy, and security standpoint.

## Examples

### Example 1: Single State StatusList Credential

A Status List credential contains a list embedded in it, this list has a singular purpose namely "revocation". The list is a bitstring, where each bit represents the status of a verifiable credential. The list is encoded as a base64url-encoded bitstring. The bitstring is decoded to a byte array, which is then decoded to a list of integers. The list of integers is then converted to a list of booleans. The list of booleans is then used to determine the status of a verifiable credential.:

```JSON
{
  "@context": [
    "https://www.w3.org/2018/credentials/v1",
    "https://w3id.org/vc/status-list/2021/v1"
  ],
  "id": "https://example.com/credentials/status/3",
  "type": ["VerifiableCredential", "StatusList2021Credential"],
  "issuer": "did:example:12345",
  "issued": "2021-04-05T14:27:40Z",
  "credentialSubject": {
    "id": "https://example.com/status/3#list",
    "type": "StatusList2021",
    "statusPurpose": "revocation",
    "encodedList": "H4sIAAAAAAAAA-3BMQEAAADCoPVPbQwfoAAAAAAAAAAAAAAAAAAAAIC3AYbSVKsAQAAA"
  },
  "proof": { ... }
}
```

An issuer can then issue a credential with a `credentialStatus` field that utilizes the issued statusListCredential:

```JSON
{
  "@context": [
    "https://www.w3.org/2018/credentials/v1",
    "https://w3id.org/vc/status-list/2021/v1"
  ],
  "id": "https://example.com/credentials/23894672394",
  "type": ["VerifiableCredential"],
  "issuer": "did:example:12345",
  "issued": "2021-04-05T14:27:42Z",
  "credentialStatus": {
    "id": "https://example.com/credentials/status/3#94567",
    "type": "StatusList2021Entry",
    "statusPurpose": "revocation",
    "statusListIndex": "94567",
    "statusListCredential": "https://example.com/credentials/status/3"
  },
  "credentialSubject": {
    "id": "did:example:6789",
    "type": "Person"
  },
  "proof": { ... }
}
```

### Example 2: Multiple State StatusList Credential

A StatusList credential may optionally contain a list of status messages. The following example defines statuses for "0x0", "0x1", "0x2":

```JSON
{
  "@context": [
    "https://www.w3.org/2018/credentials/v1",
    "https://w3id.org/vc/status-list/2021/v1"
  ],
  "id": "https://example.com/credentials/status/3",
  "type": ["VerifiableCredential", "StatusList2021Credential"],
  "issuer": "did:example:12345",
  "issued": "2021-04-05T14:27:40Z",
  "credentialSubject": {
    "id": "https://example.com/status/3#list",
    "type": "StatusList2021",
    "ttl": 500,
    "statusPurpose": "status",
    "reference": "https://example.org/status-dictionary/",
    "size": 2,
    "statusMessages": [ 
        {"status":"0x0", "value":"valid"},
        {"status":"0x1", "value":"invalid"},
        {"status":"0x2", "value":"pending_review"},
        ...
    ],
    "encodedList": "H4sIAAAAAAAAA-3BMQEAAADAAAAAAAAAAAAAAAAAAAAAAAAAAAIC3AYbSVKsAQAAA"
  }
}
```

An example credential issued to use a multiple statusMessage statusList looks identical to one issued for a single use.

## Detailed design discussion

There are a variety of privacy and performance considerations that are made when designing, publishing, and processing status lists. The following are some of the considerations that are being discussed by the Working Group.

### Privacy Considerations:

#### Correlation between holder, time, and verifier

A privacy consideration arises when there is a one-to-one mapping between a verifiable credential and a URL where the status is published. This type of mapping enables the website that publishes the URL to correlate the holder, time, and verifier when the status is checked. This could enable the issuer to discover the type of interaction the holder is having with the verifier, such as providing an age verification credential when entering a bar. Being tracked by the issuer of a driver's license when entering an establishment violates a privacy expectation that many people have today.

#### Malicious Issuers and Verifiers

In general, the herd privacy protections offered by this specification can be circumvented by malicious issuers and verifiers. Its privacy benefits can only be realized when issuers and verifiers intend to avoid tracking or sharing the presentation of particular credentials.

A malicious issuer might intentionally attack herd privacy by creating a unique status list per credential issued in order to establish a 1-1 mapping to track when a verifier processes a specific credential. Similarly, they could establish another a 1-1 mapping by using a different cryptographic key for every credential issued that is tracked in a status list.

A malicious verifier might intentionally attack herd privacy by sharing information from presented credentials with a malicious issuer.

### Performance Considerations

Similarly, there are performance considerations that are explored when designing status lists. One such consideration is where the list is published and the burden it places from a bandwidth and processing perspective, both on the server and the client fetching the information. In order to meet privacy expectations, it is useful to bundle the status of large sets of credentials into a single list to help with herd privacy. However, doing so can place an impossible burden on both the server and client if the status information is as much as a few hundred bytes in size per credential across a population of hundreds of millions of holders.

### Addressing Privacy and Performance

The specification proposes a highly compressible, bitstring-based status list mechanism with strong privacy-preserving characteristics, that is compatible with the architecture of the Web, is highly space-efficient, and lends itself well to content distribution networks. As an example of using this specification to achieve a number of beneficial privacy and performance goals, it is possible to create a status list that can be constructed for 100,000 verifiable credentials that is roughly 12,500 bytes in size in the worst case. In a case where a few hundred credentials have been revoked, the size of the list is less than a few hundred bytes while providing privacy in a herd of 100,000 individuals.

1. It is possible for verifiers to increase the privacy of the holder whose verifiable credential is being checked by caching status lists that have been fetched from remote servers. By caching the content locally, less correlatable information can be inferred from verifier-based access patterns on the status list.
2. The use of content distribution networks by issuers can increase the privacy of holders by reducing or eliminating requests for the status lists from the issuer. Often, a request for a revocation list will be served by an edge device and thus be faster and reduce the load on the server as well as cloaking verifiers and holders from issuers.
