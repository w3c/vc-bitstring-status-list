# Bitstring Status List Explained

## Authors

- Mahmoud Alkhraishi (Mavennet)
- Manu Sporny (Digital Bazaar)
- Dave Longley (Digital Bazaar)
- Orie Steele (Transmute)
- Mike Prorock (mesur.io)

## Participate

- [Issue tracker](https://github.com/w3c/vc-bitstring-status-list/issues)
- [Discussion forum](https://lists.w3.org/Archives/Public/public-vc-wg/)

## Introduction

This specification describes a privacy-preserving, space-efficient, and high-performance mechanism for publishing status information such as suspension or revocation of Verifiable Credentials.

## Goals

Provide a location for an issuer of a
[Verifiable Credential](https://www.w3.org/TR/vc-data-model-2.0/) to link to a
location where a verifier can check to see if a credential has been suspended or
revoked.

## How does it work?

At the most basic level, status information for all verifiable credentials
issued by an issuer are expressed as simple binary values. The issuer keeps a
bitstring list of all verifiable credentials it has issued. Each verifiable
credential is associated with a position in the list. If the binary value of the
position in the list is 1 (one), the verifiable credential is revoked; if it is
0 (zero), it is not revoked. The verifier can then check the status of a
verifiable credential by looking up the position of the verifiable credential in
the bitstring list. This is the basic model used for Revocation and Suspension.

A status list may have multiple states, exposed via a series of status messages.
These status messages are intended to be extensible and interoperable with
existing status message vocabularies. The status messages are intended to be
machine-readable. The status messages are intended to be used by verifiers to
determine the status of a verifiable credential.

## Examples

### Example 1: Single State StatusList Credential

A `StatusList` credential contains a list with a singular purpose: "revocation".
The list is a bitstring, where each bit represents the status of a verifiable
credential. The list is encoded as a base64url-encoded bitstring. The bitstring
is decoded to a list of booleans. The list of booleans is then used to determine
the status of a verifiable credential, as shown.

```JSON
{
  "@context": [
    "https://www.w3.org/ns/credentials/v2"
  ],
  "id": "https://example.com/credentials/status/3",
  "type": ["VerifiableCredential", "BitstringStatusListCredential"],
  "issuer": "did:example:12345",
  "validFrom": "2021-04-05T14:27:40Z",
  "credentialSubject": {
    "id": "https://example.com/status/3#list",
    "type": "BitstringStatusList",
    "statusPurpose": "revocation",
    "encodedList": "uH4sIAAAAAAAAA-3BMQEAAADCoPVPbQwfoAAAAAAAAAAAAAAAAAAAAIC3AYbSVKsAQAAA"
  }
}
```

An issuer can then issue a credential with a `credentialStatus` field that utilizes the issued `statusListCredential`.

```JSON
{
  "@context": [
    "https://www.w3.org/ns/credentials/v2"
  ],
  "id": "https://example.com/credentials/23894672394",
  "type": ["VerifiableCredential"],
  "issuer": "did:example:12345",
  "validFrom": "2021-04-05T14:27:42Z",
  "credentialStatus": {
      "id": "https://example.com/credentials/status/3#94567",
      "type": "BitstringStatusListEntry",
      "statusPurpose": "revocation",
      "statusListIndex": "94567",
      "statusListCredential": "https://example.com/credentials/status/3"
  },
  "credentialSubject": {
    "id": "did:example:6789",
    "type": "Person"
  }
}
```

### Example 2: Multiple State StatusList Credential

A `StatusList` credential may optionally contain a list of status messages. The following example defines statuses for "`0x0`", "`0x1`", and "`0x2`":

```JSON
{
  "@context": [
    "https://www.w3.org/ns/credentials/v2"
  ],
  "id": "https://example.com/credentials/status/3",
  "type": ["VerifiableCredential", "BitstringStatusListCredential"],
  "issuer": "did:example:12345",
  "validFrom": "2021-04-05T14:27:40Z",
  "credentialSubject": {
    "id": "https://example.com/status/3#list",
    "type": "BitstringStatusList",
    "ttl": 500,
    "statusPurpose": "status",
    "reference": "https://example.org/status-dictionary/",
    "size": 2,
    "statusMessage": [
      {"status":"0x0", "message":"valid"},
      {"status":"0x1", "message":"invalid"},
      {"status":"0x2", "message":"pending_review"}
    ],
    "encodedList": "uH4sIAAAAAAAAA-3BMQEAAADAAAAAAAAAAAAAAAAAAAAAAAAAAAIC3AYbSVKsAQAAA"
  }
}
```

An example credential issued to use a multiple state `statusList` looks
identical to one issued for a single use.

## Detailed design discussion

There are a variety of privacy and performance considerations that are made when
designing, publishing, and processing status lists. The following are some of
the considerations that are being discussed by the Working Group.

### Privacy Considerations

#### Correlation between holder, time, and verifier

A privacy concern arises when there is a one-to-one mapping between a verifiable
credential and a URL where its status is published. This type of mapping enables
the website that publishes the URL to correlate the holder, time, and verifier,
whenever the status is checked. This could enable the issuer to discover the
type of interaction the holder is having with the verifier, such as providing an
age verification credential when entering a bar. Being tracked by the issuer of
a driver's license when entering an establishment violates a privacy expectation
that many people have today.

#### Malicious Issuers and Verifiers

The group privacy protections offered by this specification can generally be
circumvented by malicious issuers and verifiers. Its privacy benefits can only
be realized when issuers and verifiers intend to avoid tracking or sharing the
presentation of particular credentials.

A malicious issuer might intentionally attack group privacy by creating a unique
status list per credential issued in order to establish a 1-1 mapping to track
when a verifier processes a specific credential. Similarly, they could establish
another a 1-1 mapping by using a different cryptographic key for every
credential issued that is tracked in a status list.

A malicious verifier might intentionally attack group privacy by sharing
information from presented credentials with a malicious issuer.

### Performance Considerations

There are various performance considerations that are explored when designing
status lists. One such consideration is the bandwidth and processing burden a
published list places both on the server and on the client fetching the
information. Bundling the statuses of large sets of credentials into a single
list can help with group privacy. However, doing so can place an impossible
burden on both server and client if the size of the status information is as
much as a few hundred bytes per credential across a population of hundreds of
millions of holders.

### Addressing Privacy and Performance

The specification proposes a highly compressible, bitstring-based status list
mechanism with strong privacy-preserving characteristics, that is compatible
with the architecture of the Web, is highly space-efficient, and lends itself
well to content distribution networks. For example, using this specification to
achieve a number of beneficial privacy and performance goals, it is possible to
construct a status list to handle 100,000 verifiable credentials at a maximum
size of roughly 12,500 bytes. When only a few hundred of these credentials have
been revoked, the size of the list is less than a few hundred bytes while
providing privacy for a group of 100,000 individuals.

* It is possible for verifiers to increase the privacy of holders whose
  verifiable credentials are being checked by caching status lists that have
  been fetched from remote servers. By caching the content locally, less
  correlatable information can be inferred from verifier-based access patterns
  on the status list.

* The use of content distribution networks (CDNs) by issuers can increase the
  privacy of holders by reducing or eliminating requests for the status lists
  directly from the issuer. Often, a request for a revocation list can be served
  by an edge device and thus be faster and reduce the load on the server, as
  well as cloaking verifiers and holders from issuers.
