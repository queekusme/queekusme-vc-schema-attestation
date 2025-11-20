%%%
title = "Verifiable Credential Schema Attestation"
abbrev = "VC Schema Attestation"
ipr = "trust200902"
area = "Security"
workgroup = "TODO Workgroup"
keyword = ["security", "attestation", "schema"]

[seriesInfo]
name = "Internet-Draft"
value = "draft-queekusme-vc-schema-attestation-latest"
stream = "IETF"
status = "standard"

[[author]]
initials="A.J."
surname="Kennedy"
fullname="Annabelle Kennedy"
    [author.address]
    email = "queekusme@gmail.com"

%%%

.# Abstract

This specification defines request and response formats for Attesting Issuers and Verifiers for individual Verifiable
Credential Schema via a Schema Attestation Document

{mainmatter}

# Introduction

## Trust in the Issuer-Holder-Verifier Model

Implementations of the Issuer-Holder-Verifier model have created solutions for the problem of trust in the issuance of
and attestation of verifiers for verifiable credentials. Wallets and verifiers **MAY** subscribe to specific trust
anchors to verify:

- A Verifiable Credential has been issued by an authorised issuer
- A verifier is authorised to receive a Verifiable Credential

Whilst certain wallets can subscribe to a closed system for trust, others **MAY** wish to accept credentials from any
issuer allowing the holder of the aforementioned Credential to present and have verified any Verifiable Credential.

The Wallet would need to determine the correct trust anchors to subscribe to in order to ascertain the correct level of
trust in Credential issuers and verifiers

## Requirements Notation and Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in RFC 2119 [@!RFC2119].

## Terms and Definitions

This specification uses the terms "Holder", "Issuer", "Verifier", "Disclosure", "Selectively Disclosable JWT (SD-JWT)",
"Key Binding", "Key Binding JWT (KB-JWT)", "Selectively Disclosable JWT with Key Binding (SD-JWT+KB)" defined by
[@!I-D.ietf-oauth-selective-disclosure-jwt].

Verifiable Credential (VC):
:  An assertion with claims about a Subject that is cryptographically secured by an Issuer (usually by a digital
   signature).

## Verifiable Credential Schema documents

Verifiable Credential Schema are documents which define the structure of verifiable credentials in a manner which allows
verifiers and other external software and tooling to determine if a Credential conforms to the schema design.

The Verifiable Credential formats which utilize schema documents are:

- W3C Verifiable Credential Data Model (VCDM) 2.0 [@W3C.VCDM] which uses JSON-LD as its structure definition
- SD-JWT-based Verifiable Credentials [@!I-D.ietf-oauth-sd-jwt-vc] which uses its own defined `vct` structure definition.

MDoc Credential formats are outside of the scope of this document

Both [@W3C.VCDM] and [@!I-D.ietf-oauth-sd-jwt-vc] permit schema document definition via URL request.

# Verifiable Credential Schema Attestation Document

## Schema Attestation Document

The Schema Attestation Document is a Selective Disclosure JWT [@!I-D.ietf-oauth-selective-disclosure-jwt] which attests
to the validity to issue or verify a Verifiable Credential of a specific Schema.

### Schema Attestation Document Claims {#schema-attestation-claims}

The specification defines the following JWT claims:

* `iss`: **REQUIRED**. The issuer of the document, **MUST** be a [@W3C.DID] identifier
* `sub`: **REQUIRED**. The string schema document URL. This being the [@W3C.VCDM] `credentialSchema`.`id` URL or
  the [@!I-D.ietf-oauth-sd-jwt-vc] `vct` URL.
* `sub#integrity`: **REQUIRED**. A string containing the [@!W3C.SRI] integrity of the Schema Document this Attestation
  points to.
* `authorized_issuers`: **OPTIONAL**. An array of strings containing the issuers which are authorised to issue credentials
  matching the Schema. `authorized_issuers` **MAY** be an empty array.
* `authorized_verifiers`: **OPTIONAL**. An array of strings containing the verifiers which are authorised to issue
  credentials matching the Schema. `authorized_issuers` **MAY** be an empty array.
* `nonce`: **REQUIRED** if included in `POST` Request Parameters, otherwise **OPTIONAL**. The nonce provided by the
  requester, ensures signature freshness

JWT claims `iat`, `nbf` and `exp` **SHOULD** be used to establish timeframes for the acceptance and disposal of expired
Attestation JWTs

#### Selective Disclosure

The Selective Disclosure JWT Response **SHOULD** use Decoy Digests to disguise the actual number of authorized issuers
and verifiers.

The `iat`, `nbf` and `exp` claims **MUST NOT** be Selectively Disclosable in SD-JWT responses where provided.

The `iss`, `sub`, `sub#integrity` and `nonce` claims **MUST NOT** be Selectively Disclosable.

The `authorized_issuers` and `authorized_verifiers` claims **SHOULD** be Selectively Disclosable, all array values of
`authorized_issuers` and `authorized_verifiers` **SHOULD** be Selectively Disclosable.

#### Examples

The following is a non-normative example of the raw attestation data of an unsecured payload of a Schema Attestation
Document:

<{{examples/01/user_claims.json}}

The following is a non-normative example of how the unsecured payload of the raw attestation data above can be presented
as an SD-JWT with `authorized_issuers` and `authorized_verifiers` claims selectively disclosable:

<{{examples/01/sd_jwt_payload.json}}

The following are the Disclosures belonging to the SD-JWT payload above:

{{examples/01/disclosures.md}}

### Schema Document Attestation Well-known Endpoint {#schema-document-well-known}

Schema designers **MAY** make a Schema Attestation Document available at the location formed by inserting the well known
string `/.well-known/schema-attestation` between the host component and the path component of the Schema URL.

If a Schema Attestation Document is not made available at the well-known address, trust  **SHOULD** defer back to other
methods for determining trust.

The following is a non-normative example of an HTTP request for the Schema Attestation Document
when the Schema Document url is `http://example.com/example-credential.json`

```
GET /.well-known/schema-attestation/example-credential.json HTTP/1.1
Host: example.com
```
### Request Parameters

The following parameters are defined to be included in the request to the Request URI Endpoint:

* `issuer`: **OPTIONAL**. The Issuer to receive attestation information for if present in the Attestation Document
* `verifier`: **OPTIONAL**. The Verifier to receive attestation information for if present in the Attestation Document
* `nonce`: **REQUIRED**. A nonce ensuring the freshness of the signature in JWT responses

The following is a non-normative example of an HTTP request for the Schema Attestation Document requesting attestation
information for an issuer `http://example.com/issuer`

```
POST /.well-known/schema-attestation/example-credential.json HTTP/1.1
Host: example.com
Content-Type: application/x-www-form-urlencoded

issuer=http%3A%2F%2Fexample.com%2Fissuer
```

## Schema Attestation Document Response

A successful response **MUST** use the `200 OK HTTP`. The Content-Type header **MUST** be `application/sd-jwt`.

The document **MUST** return any issuer or verifier Disclosure matching the issuer or verifier provided in any Request
Parameters.

### Example

The following is a non-normative example of a Schema Attestation Document Response including Disclosures for attesting
issuer `http://example.com/issuer` provided via request parameters"

<{{examples/01/sd_jwt_presentation.txt}}

Below is a non-normative example of the verified SD-JWT claims the recipient of a Schema Attestation Document would
receive when resolving Selective Disclosures in a Schema Attestation Document.

<{{examples/01/verified_contents.json}}

### JOSE Header

The `typ` value in the JOSE Header **MUST** be `schema-attestation+sd-jwt`.

The following is a non-normative example of a decoded SD-JWT header:

```
{
  "alg": "ES256",
  "typ": "schema-attestation+sd-jwt"
}
```

## Verifying Schema Attestation Documents

Recipients of a Schema Attestation Document **MUST** verify the returned SD-JWT.

Verification of the issuer of a Schema Attestation Document **MUST** follow current Best Practices for verifying JWT
signatures.

### Verifying Issuers in `authorized_issuers`

For an Issuer to be attested as Authorized to issue a Credential matching the respective Schema the value in the
`issuer` claim of a [@W3C.VCDM] Credential or the `iss` claim of a [@!I-D.ietf-oauth-sd-jwt-vc] Credential **MUST** be
present in the `authorized_issuers` array.

### Verifying verifiers in `authorized_verifiers`

For a verifier to be attested as Authorised to receive a Credential matching the respective Schema the value in the
`authorized_verifiers` array **MUST** match one of the following:

* For an [@OAUTH.OID4VP] presentation which presents the credential request as a `request_uri` JWT, the `iss` value MUST
  match the value in `authorized_verifiers`
* TODO: Identify other methods for identifying verifiers within a credential request

## Schema Extension

[@!I-D.ietf-oauth-sd-jwt-vc] permits `vct` documents to extend from another `vct` document. As a Schema Attestation
Document attests for a specific version of a Schema document, the document the `vct` extends from **MAY** have its own
attestation document.

Schema Document Requests **MUST NOT** inherit Attestation from higher order Schema Attestation Documents.

# Security Considerations

## Issuer/Verifier Request for Information Discovery

Requests to the well-known endpoint **MAY** use the `issuer` and `verifier` Request parameters in order to perform
information discovery on the Disclosures within a the Attestation Document.

Schema Attestation Document Providers **SHOULD** make effort to prevent Information Discovery utilizing methods such as
Rate Limiting.

Nonce values which are reused **SHOULD** be rejected if multiple requests from the same requester are received with the
same nonce value.

## Schema Extension as a method for attaining a trusted status.

An attacker **MAY** extend a known Schema document and provide their own Schema Attestation document in order to show
themselves as 'verified' to Wallets and Verifiers.

Whilst extension remains a valid method for building upon existing schema, Wallets **SHOULD NOT** treat extended
documents as an attempt to bypass trust. Wallets **SHOULD** show credentials with extended Schema as Trusted, so long as
the issuer is correctly included as an `authorized_issuers` in the extended Schema's Schema Attestation Document.

[@OAUTH.OID4VP] includes the specification for Digital Credentials Query Language (DCQL). DCQL introduces `meta` which
allows for the specification of metadata associated with a Credential request.

For [@!I-D.ietf-oauth-sd-jwt-vc] Credentials, the `meta` object **MAY** contain the `vct_values` key, which contains a
non-empty array of strings. A wallet **MAY** return Credentials that inherit from any type specified. This means that
imitation credentials **MAY** be provided and cannot be relied upon to filter imitation credentials.

Verifiers **SHOULD** ensure that the Schema in use within a Verifiable Credential matches the expected Schema.

<reference anchor="IANA.well-known" target="https://www.iana.org/assignments/well-known-uris">
    <front>
        <title>Well-Known URIs</title>
        <author>
            <organization>IANA</organization>
        </author>
        <date/>
    </front>
</reference>

<reference anchor="OAUTH.OID4VP" target="https://openid.net/specs/openid-4-verifiable-presentations-1_0.html">
    <front>
        <author initials="O." surname="Terbu" fullname="Oliver Terbu">
            <organization>
                <organizationName>MATTR</organizationName>
            </organization>
        </author>
        <author initials="T." surname="Lodderstedt" fullname="Torsten Lodderstedt">
            <organization>
                <organizationName>SPRIND</organizationName>
            </organization>
        </author>
        <author initials="K." surname="Yasuda" fullname="Kristina Yasuda">
            <organization>
                <organizationName>SPRIND</organizationName>
            </organization>
        </author>
        <author initials="D." surname="Fett" fullname="Daniel Fett">
            <organization>
                <organizationName>Authlete</organizationName>
            </organization>
        </author>
        <author initials="J." surname="Heenan" fullname="Joseph Heenan">
            <organization>
                <organizationName>Authlete</organizationName>
            </organization>
        </author>
        <title>OpenID for Verifiable Presentations 1.0</title>
        <date day="9" month="July" year="2025"/>
    </front>
</reference>

<reference anchor="W3C.DID" target="https://www.w3.org/TR/did-1.0/">
    <front>
        <author initials="M." surname="Sporny" fullname="Manu Sporny">
            <organization>
                <organizationName>Digital Bazaar</organizationName>
            </organization>
        </author>
        <author initials="D." surname="Longley" fullname="Dave Longley">
            <organization>
                <organizationName>Digital Bazaar</organizationName>
            </organization>
        </author>
        <author initials="M." surname="Sabadello" fullname="Markus Sabadello">
            <organization>
                <organizationName>Danube Tech</organizationName>
            </organization>
        </author>
        <author initials="D." surname="Reed" fullname="Drummond Reed">
            <organization>
                <organizationName>Evernym/Avast</organizationName>
            </organization>
        </author>
        <author initials="O." surname="Steele" fullname="Orie Steele">
            <organization>
                <organizationName>Transmute</organizationName>
            </organization>
        </author>
        <author initials="C." surname="Allen" fullname="Christopher Allen">
            <organization>
                <organizationName>Blockchain Commons</organizationName>
            </organization>
        </author>
        <title>Decentralized Identifiers (DIDs) v1.0</title>
        <date day="19" month="July" year="2022"/>
    </front>
</reference>

<reference anchor="W3C.SRI" target="https://www.w3.org/TR/SRI/">
    <front>
        <author initials="D." surname="Akhawe" fullname="Devdatta Akhawe">
            <organization>
                <organizationName>Dropbox, Inc.</organizationName>
            </organization>
        </author>
        <author initials="F." surname="Braun" fullname="Frederik Braun">
            <organization>
                <organizationName>Mozilla</organizationName>
            </organization>
        </author>
        <author initials="F." surname="Marier" fullname="François Marier">
            <organization>
                <organizationName>Mozilla</organizationName>
            </organization>
        </author>
        <author initials="J." surname="Weinberger" fullname="Joel Weinberger">
            <organization>
                <organizationName>Google, Inc.</organizationName>
            </organization>
        </author>
        <title>Subresource Integrity</title>
        <date day="23" month="June" year="2016"/>
    </front>
</reference>

<reference anchor="W3C.VCDM" target="https://www.w3.org/TR/vc-data-model-2.0/">
    <front>
        <author initials="M." surname="Sporny" fullname="Manu Sporny">
            <organization>
                <organizationName>Digital Bazaar</organizationName>
            </organization>
        </author>
        <author initials="D." surname="Longley" fullname="Dave Longley">
            <organization>
                <organizationName>Digital Bazaar</organizationName>
            </organization>
        </author>
        <author initials="D." surname="Chadwick" fullname="David Chadwick">
            <organization>
                <organizationName>Crossword Cybersecurity PLC</organizationName>
            </organization>
        </author>
        <author initials="O." surname="Steele" fullname="Orie Steele">
            <organization>
                <organizationName>Transmute</organizationName>
            </organization>
        </author>
        <title>Verifiable Credentials Data Model v2.0</title>
        <date day="10" month="February" year="2024"/>
    </front>
</reference>

{backmatter}

# IANA Considerations

## JSON Web Token Claims Registration

- Claim Name: "sub#integrity"
- Claim Description: JWT VC sub claim "integrity metadata" value
- Change Controller: IETF
- Specification Document(s): [[ (#schema-attestation-claims) of this specification ]]

- Claim Name: "authorized_issuers"
- Claim Description: Authorized Issuers for a specific VC Schema
- Change Controller: IETF
- Specification Document(s): [[ (#schema-attestation-claims) of this specification ]]

- Claim Name: "authorized_verifiers"
- Claim Description: Authorized Verifiers for a specific VC Schema
- Change Controller: IETF
- Specification Document(s): [[ (#schema-attestation-claims) of this specification ]]

## Well-Known URI Registry

This specification requests the well-known URI defined in (#schema-document-well-known)
in the IANA "Well-Known URIs" registry [@IANA.well-known] established by [@!RFC5785].

# Acknowledgements {#Acknowledgements}

There are no acknowledgements at this time.

# Document History

-00

* Initial Document
