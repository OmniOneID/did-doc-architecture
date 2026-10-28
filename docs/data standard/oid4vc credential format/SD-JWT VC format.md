---
puppeteer:
    pdf:
        format: A4
        displayHeaderFooter: true
        landscape: false
        scale: 0.8
        margin:
            top: 1.2cm
            right: 1cm
            bottom: 1cm
            left: 1cm
    image:
        quality: 100
        fullPage: false
---

SD-JWT VC format
==

- Topic
    - Definition of the SD-JWT VC data format issued over OID4VCI
    - Definition of the relationship between Credential Configuration and SD-JWT VC
- Author: OpenSource Development Team
- Date: 2026-10-22
- Version: v1.0.0


Revision History
---

| Version | Date       | Changes         |
| ------- | ---------- | --------------- |
| v1.0.0  | 2026-10-22 | Initial version |


<div style="page-break-after: always;"></div>

Table of Contents
---

<!-- TOC tocDepth:2..4 chapterDepth:2..6 -->

- [1. Overview](#1-overview)
    - [1.1. Reference Documents](#11-reference-documents)
    - [1.2. Terminology](#12-terminology)
- [2. General Regulations](#2-general-regulations)
    - [2.1. Credential Format](#21-credential-format)
    - [2.2. SD-JWT VC Structure](#22-sd-jwt-vc-structure)
    - [2.3. Data Types and Constants](#23-data-types-and-constants)
    - [2.4. Issuer-signed JWT](#24-issuer-signed-jwt)
        - [2.4.1. JWT Header](#241-jwt-header)
        - [2.4.2. JWT Payload](#242-jwt-payload)
        - [2.4.3. Signature](#243-signature)
    - [2.5. Disclosure](#25-disclosure)
    - [2.6. Claim Disclosure Classes](#26-claim-disclosure-classes)
        - [2.6.1. Reserved Claim Names](#261-reserved-claim-names)
    - [2.7. Holder Binding](#27-holder-binding)
    - [2.8. Status Reference](#28-status-reference)
    - [2.9. Key Binding JWT](#29-key-binding-jwt)
- [3. Credential Configuration](#3-credential-configuration)
    - [3.1. Credential Configuration Structure](#31-credential-configuration-structure)
    - [3.2. Mapping to SD-JWT VC](#32-mapping-to-sd-jwt-vc)
    - [3.3. Credential Configuration Example](#33-credential-configuration-example)
- [4. SD-JWT VC Example](#4-sd-jwt-vc-example)
    - [4.1. Issuer-signed JWT Header](#41-issuer-signed-jwt-header)
    - [4.2. Issuer-signed JWT Payload](#42-issuer-signed-jwt-payload)
    - [4.3. Disclosure](#43-disclosure)
    - [4.4. Compact Serialization](#44-compact-serialization)
    - [4.5. Credential Response](#45-credential-response)
    - [4.6. Presentation Format](#46-presentation-format)

<!-- /TOC -->


<div style="page-break-after: always;"></div>

## 1. Overview

This document defines the data structure of the SD-JWT VC that OpenDID 3.0 issues over OID4VCI (OpenID for Verifiable Credential Issuance).
The basic regulations comply with the IETF `[SD-JWT]` (Selective Disclosure for JSON Web Tokens) and `[SD-JWT-VC]` (SD-JWT-based Verifiable Credentials),
and the Credential Configuration follows the Credential Issuer Metadata of `[OID4VCI]`.
For the concepts of the issuance procedure and status management, refer to `[OID4VCI-ISSUANCE]`.

The following must adhere to the definitions in this document.

- Header and claim structure of the Issuer-signed JWT
- Disclosure structure and the classification of selectively disclosable claims
- Holder binding (`cnf`) and status reference (`status`) structure
- Mapping between Credential Configuration and SD-JWT VC

The following are _out of the scope_ of this document and can be defined freely by the implementation.

- Types and meanings of Subject claims for each `vct`
- Token Status List format and status management procedures (see `[TOKEN-STATUS]`, `[OID4VCI-ISSUANCE]`)
- Presentation protocol
- Additional items based on the regulations of this document


### 1.1. Reference Documents

| Reference          | Document Name                                            | Location                                                                                               |
| ------------------ | -------------------------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| [SD-JWT]           | RFC 9901 Selective Disclosure for JSON Web Tokens        | https://www.rfc-editor.org/rfc/rfc9901                                                                 |
| [SD-JWT-VC]        | SD-JWT-based Verifiable Credentials (SD-JWT VC)          | https://datatracker.ietf.org/doc/draft-ietf-oauth-sd-jwt-vc/                                           |
| [OID4VCI]          | OpenID for Verifiable Credential Issuance 1.0            | https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0.html                              |
| [TOKEN-STATUS]     | Token Status List                                        | https://datatracker.ietf.org/doc/draft-ietf-oauth-status-list/                                         |
| [RFC7515]          | JSON Web Signature (JWS)                                 | https://www.rfc-editor.org/rfc/rfc7515                                                                 |
| [RFC7517]          | JSON Web Key (JWK)                                       | https://www.rfc-editor.org/rfc/rfc7517                                                                 |
| [RFC7518]          | JSON Web Algorithms (JWA)                                | https://www.rfc-editor.org/rfc/rfc7518                                                                 |
| [RFC7800]          | Proof-of-Possession Key Semantics for JSON Web Tokens    | https://www.rfc-editor.org/rfc/rfc7800                                                                 |
| [OSD]              | OpenDID Schema Definition Language                       | [OpenDID Schema Definition Language.md](../../guide/osd/OpenDID%20Schema%20Definition%20Language.md)   |
| [DATA-SPEC]        | (OpenDID) Data Specification                             | [Data Specification.md](../data%20specification/Data%20Specification.md)                               |
| [DID-DOC-FORMAT]   | (OpenDID) DID Document format                            | [DID Document format.md](../did%20document%20format/DID%20Document%20format.md)                        |
| [OID4VCI-ISSUANCE] | (OpenDID) OID4VCI-based VC Issuance                      | [OID4VCI Issuance.md](../../concepts/OID4VCI%20Issuance.md)                                            |


### 1.2. Terminology

| Term                           | Description                                                                                                   |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------- |
| SD-JWT VC                      | A VC composed of an Issuer-signed JWT and Disclosures that supports selective disclosure per claim            |
| Issuer-signed JWT              | The JWT signed by the Issuer. It contains the Issuer-signed claims and the list of Disclosure digests.        |
| Disclosure                     | A string containing the salt, claim name, and claim value of one selectively disclosable claim                |
| Digest                         | The hash of a Disclosure. It is included in `_sd` of the Issuer-signed JWT.                                   |
| Issuer-signed claim            | A claim included as-is in the Issuer-signed JWT and always disclosed. The Holder cannot omit it when presenting. |
| Selectively disclosable claim  | A claim delivered as a Disclosure, which the Holder chooses whether to disclose when presenting               |
| Holder binding                 | Binding the Credential to a Holder key. The Holder public key is included in the `cnf` claim.                 |
| KB-JWT                         | Key Binding JWT. A JWT that the Holder signs with the bound key at presentation to prove key possession       |


<div style="page-break-after: always;"></div>

## 2. General Regulations

### 2.1. Credential Format

OpenDID 3.0 issues SD-JWT VCs with the following Credential Format identifier.

| Format          | Credential | Issuer Key Identification  | JWT Header `typ` | Holder Binding |
| --------------- | ---------- | -------------------------- | ---------------- | -------------- |
| `dc+sd-jwt-did` | SD-JWT VC  | Issuer DID key URL (`kid`) | `dc+sd-jwt-did`  | `cnf.jwk`      |

- `dc+sd-jwt-did` is a Format identifier defined by OpenDID. It denotes an SD-JWT VC whose Issuer signing key is resolved from the Issuer DID Document registered in the trusted repository.
- The same value is used for `format` of the Credential Configuration and `typ` of the Issuer-signed JWT Header.


### 2.2. SD-JWT VC Structure

An SD-JWT VC is a compact serialization string that joins the Issuer-signed JWT and the Disclosures with `~`.
At issuance, it ends with a trailing `~` without a KB-JWT.

```
<Issuer-signed JWT>~<Disclosure 1>~<Disclosure 2>~...~<Disclosure N>~
```

At presentation, the Holder selects only the Disclosures to disclose and, if needed, appends a KB-JWT at the end (see [2.9](#29-key-binding-jwt)).

```
<Issuer-signed JWT>~<Disclosure i>~...~<Disclosure k>~<KB-JWT>
```

| Component         | Format       | Description                                                                                     |
| ----------------- | ------------ | ----------------------------------------------------------------------------------------------- |
| Issuer-signed JWT | `compactJws` | The JWT signed by the Issuer. It contains the Issuer-signed claims and the Disclosure digest list (`_sd`). |
| Disclosure        | `disclosure` | One selectively disclosable claim. Zero or more are included.                                   |
| KB-JWT            | `compactJws` | (At presentation, optional) A JWT signed by the Holder with the `cnf` key                       |


### 2.3. Data Types and Constants

For types not defined here (`did`, `didKeyUrl`, `url`, `LANGUAGE`), refer to `[DATA-SPEC]`.

```c#
def string base64url  : "base64url-encoded string (without padding)"
def string compactJws : "JWS Compact Serialization string (header.payload.signature)"
def string disclosure : "Disclosure. A JSON array encoded in UTF-8 and then base64url-encoded"
def string sdDigest   : "Disclosure digest. base64url(SHA-256(ASCII(disclosure)))"
def string salt       : "16-byte random value encoded in base64url", length(22)
def int    numericDate: "NumericDate. Seconds since 1970-01-01T00:00:00Z", min_value(0)

def enum CREDENTIAL_FORMAT: "Credential format"
{
    "dc+sd-jwt-did": "SD-JWT VC whose signing key is identified by the Issuer DID",
}

def enum JWT_TYPE: "JWT Header typ"
{
    "dc+sd-jwt-did": "Issuer-signed JWT",
    "kb+jwt"       : "Key Binding JWT",
}

def enum JWS_ALGORITHM: "JWS signature algorithm"
{
    "ES256" : "ECDSA using P-256(Secp256r1) and SHA-256",
    "ES256K": "ECDSA using secp256k1 and SHA-256",
}

def enum SD_HASH_ALGORITHM: "Disclosure digest hash algorithm"
{
    "sha-256",
}

def enum BINDING_METHOD: "Holder binding method"
{
    "jwk": "Holder public key included in cnf.jwk",
}

def enum CLAIM_VALUE_TYPE: "Claim value type (OpenDID extension)"
{
    "string"   : "String",
    "full-date": "Date (YYYY-MM-DD)",
    "date"     : "Date (YYYY-MM-DD)",
    "uint"     : "Integer greater than or equal to 0",
    "integer"  : "Integer",
    "boolean"  : "True or false",
    "list"     : "JSON array",
    "jpeg"     : "JPEG image (base64-encoded string)",
}
```


### 2.4. Issuer-signed JWT

#### 2.4.1. JWT Header

```c#
def object IssuerJwtHeader: "Issuer-signed JWT header"
{
    + JWS_ALGORITHM "alg": "Signature algorithm"
    + didKeyUrl     "kid": "DID key URL of the Issuer signing key"
    + JWT_TYPE      "typ": "JWT type", value("dc+sd-jwt-did")
}
```

- `alg`: Determined by the algorithm of the Issuer signing key.
    - Secp256r1 key: "ES256"
    - Secp256k1 key: "ES256K"
    - Credentials cannot be issued with other key algorithms.
    - The OpenDID Issuer server signs with a Secp256r1 key and publishes "ES256" in `credential_signing_alg_values_supported` of the Credential Configuration.
- `kid`: The DID key URL pointing to the `assert` key registered in `assertionMethod` of the Issuer DID Document. It includes the `versionId` query and the key fragment.
    - Example: `did:omn:issuer?versionId=1#assert`
- `typ`: Always "dc+sd-jwt-did".
- No Header members other than the three above are included.

#### 2.4.2. JWT Payload

```c#
def object IssuerJwtPayload: "Issuer-signed JWT payload"
{
    //---------------------------------------------------------------------------
    // Issuer-signed claims (always disclosed)
    //---------------------------------------------------------------------------
    + did              "iss"   : "Issuer DID"
    + numericDate      "iat"   : "Issuance date and time"
    + numericDate      "exp"   : "Expiration date and time"
    - numericDate      "nbf"   : "Validity start date and time"
    + string           "vct"   : "Credential type"
    + Confirmation     "cnf"   : "Holder binding key"
    - CredentialStatus "status": "Credential status reference"

    //---------------------------------------------------------------------------
    // Selective Disclosure
    //---------------------------------------------------------------------------
    - array(sdDigest)   "_sd"    : "List of Disclosure digests of selectively disclosable claims", emptiable(false)
    - SD_HASH_ALGORITHM "_sd_alg": "Digest hash algorithm", default("sha-256")
}
```

- `iss`: The Issuer DID, which is `kid` without the query and fragment.
- `iat`, `exp`: `exp` is set to 365 days after `iat`.
- `nbf`: OpenDID currently does not include `nbf`. If included, the verifier checks the validity start date and time.
- `vct`: The `vct` value of the Credential Configuration is used as-is (see [3.2](#32-mapping-to-sd-jwt-vc)).
- `cnf`: See [2.7](#27-holder-binding)
- `status`: See [2.8](#28-status-reference). The OpenDID Issuer server always includes it in `dc+sd-jwt-did` Credentials.
- `_sd`: Included if there is at least one Subject claim. The order is random, and no decoy digests are added.
- `_sd_alg`: OpenDID omits `_sd_alg`; in this case, the digest hash algorithm is "sha-256".
- `sub` and `jti` are not included. The Holder DID is not included in the Credential; the Holder is bound through `cnf`.

#### 2.4.3. Signature

The Issuer-signed JWT is signed using the JWS Compact Serialization of `[RFC7515]`.

1. Serialize the Header and Payload as JSON, encode each in base64url, and join them with `.` (signing input).
1. Sign the SHA-256 hash of the signing input with ECDSA using the Issuer's `assert` key.
1. Following `[RFC7518]`, the signature value is the 64-byte concatenation of R and S, encoded in base64url.

The verifier retrieves the Issuer DID Document from the trusted repository using the DID in `kid`, and verifies the signature with the public key corresponding to the `versionId` and fragment of `kid` (see `[DID-DOC-FORMAT]`).


### 2.5. Disclosure

A Disclosure represents one selectively disclosable claim and is an encoded string of a JSON array with the following three elements.

| index | Element     | Type         | Description                                                                          |
| :---: | ----------- | ------------ | ------------------------------------------------------------------------------------ |
|   0   | salt        | `salt`       | A 22-character string that is a 16-byte random value (SecureRandom) encoded in base64url |
|   1   | Claim name  | `string`     | Top-level name of the Subject claim                                                  |
|   2   | Claim value | JSON value   | Claim value. For an object or array, the entire value is included in one Disclosure. |

The Disclosure and digest are created in the following order.

1. Serialize the JSON array `[salt, claim name, claim value]` without whitespace.
1. Encode the serialized string in UTF-8 and then in base64url. This value is the Disclosure.
1. Encode the SHA-256 hash of the Disclosure string (ASCII) in base64url. This value is the digest, which is included in `_sd` of the Issuer-signed JWT.

OpenDID constructs Disclosures according to the following rules.

- One Disclosure is created for each top-level item of the Subject claims.
- Array element Disclosures (arrays with two elements) are not used.
- Disclosures for items inside an object claim (recursive selective disclosure) are not used.
- In the compact serialization, the order of the Disclosures is unrelated to the order of `_sd`.

The verifier computes the digest of each Disclosure, checks that it is included in `_sd`, and then restores the claim name and value into the Payload.


### 2.6. Claim Disclosure Classes

| Class                          | Claims                                      | Description                                                                         |
| ------------------------------ | ------------------------------------------- | ----------------------------------------------------------------------------------- |
| Issuer-signed claim            | `iss`, `iat`, `exp`, `vct`, `cnf`, `status` | Included as-is in the Issuer-signed JWT and always disclosed. No Disclosure is created. |
| SD-JWT structure claim         | `_sd`, `_sd_alg`                            | Structural information for selective disclosure                                     |
| Selectively disclosable claim  | Subject claims (e.g., `family_name`)        | Delivered as Disclosures; the Holder chooses whether to disclose them when presenting. |

- The protocol claim `status` is included as an Issuer-signed claim so that the verifier can check the status without a Disclosure.
- A claim with the same name cannot be included as both an Issuer-signed claim and a selectively disclosable claim.

#### 2.6.1. Reserved Claim Names

The following names are claims managed by the Issuer and cannot be used as Subject claim names.

| Claim Name                              | Purpose                                           |
| --------------------------------------- | ------------------------------------------------- |
| `iss`, `iat`, `exp`, `nbf`, `vct`, `cnf` | Issuer-signed claims                              |
| `status`                                | Credential status reference                       |
| `_sd`, `_sd_alg`                        | SD-JWT structure                                  |
| `sub`, `jti`                            | Not used (the Holder is bound through `cnf`)      |
| `credentialStatus`, `credential_status` | Avoid confusion with the status reference         |

- The Issuer cannot reassign reserved names (`iss`, `iat`, `exp`, `nbf`, `vct`, `cnf`, `_sd`, `_sd_alg`) as Issuer-signed claims.
- In user-initiated issuance (Webview), claim definitions named `iss`, `sub`, `iat`, `nbf`, `exp`, `jti`, `vct`, `cnf`, `status`, `credentialStatus`, or `credential_status` are excluded from the input fields.


### 2.7. Holder Binding

```c#
def object Confirmation: "Holder binding information"
{
    + object "jwk": "Holder public key (JWK)"
    {
        + string    "kty": "key type", value("EC")
        + string    "use": "key use", value("sig")
        + string    "crv": "curve", value("P-256")
        + base64url "x"  : "Public key x coordinate"
        + base64url "y"  : "Public key y coordinate"
    }
}
```

- `cnf.jwk` is the Holder public key verified through the Proof JWT of the Credential Request (see `[OID4VCI-ISSUANCE]` 4.5).
    - The `kid` in the Proof JWT Header is the Holder DID URL, and the Issuer uses the P-256 key registered in `authentication` of the Holder DID Document.
    - The Issuer reconstructs this key as a JWK in the `[RFC7517]` format and includes it in `cnf.jwk`. Other JWK members such as `kid` are not included.
- If one Credential Request contains multiple Proofs, one Credential is issued per Proof key, and `cnf.jwk` of each Credential is the corresponding Proof key.
- The verifier confirms Holder binding by verifying the signature of the presented KB-JWT with `cnf.jwk` (see [2.9](#29-key-binding-jwt)).


### 2.8. Status Reference

```c#
def object CredentialStatus: "Credential status reference"
{
    + object "status_list": "Token Status List reference"
    {
        + int "idx": "Index within the Status List", min_value(0)
        + url "uri": "Status List URI"
    }
}
```

- `status` follows the Referenced Token format of `[TOKEN-STATUS]`.
- `uri` has the format `{Issuer base URL}/status-lists/{listId}`, and the Issuer allocates a Status List per Credential format and an index at issuance.
- `status` is an Issuer-signed claim. After creating the Credential, the Issuer checks that `status` in the Issuer-signed JWT matches the allocated `uri` and `idx`; if not, the Credential is not issued.
- For status values (`VALID`, `INVALID`, `SUSPENDED`), status transitions, and the Status List JWT format, refer to [Credential Status format](Credential%20Status%20format.md) and Chapter 5 of `[OID4VCI-ISSUANCE]`.
- Since the OpenDID Issuer server always includes the status reference in `dc+sd-jwt-did` Credentials, the Proof JWT of the Credential Request must use a Holder DID URL `kid`.


### 2.9. Key Binding JWT

A KB-JWT is a JWT that the Holder signs at presentation with the private key corresponding to `cnf.jwk` to prove possession of the Credential.
The KB-JWT is optional, and it must be included when the verifier requires Holder binding confirmation.
If no KB-JWT is included, the presentation string ends with a trailing `~`.

```c#
def object KbJwtHeader: "KB-JWT header"
{
    + JWS_ALGORITHM "alg": "Signature algorithm", value("ES256")
    + JWT_TYPE      "typ": "JWT type", value("kb+jwt")
}

def object KbJwtPayload: "KB-JWT payload"
{
    + string      "aud"    : "Verifier identifier"
    + string      "nonce"  : "Nonce presented by the verifier"
    + numericDate "iat"    : "KB-JWT creation date and time"
    + base64url   "sd_hash": "Hash of the presented SD-JWT"
}
```

- `sd_hash`: The SHA-256 hash, encoded in base64url, of the presentation string without the KB-JWT: `<Issuer-signed JWT>~<Disclosure i>~...~<Disclosure k>~`.
- By default, the verifier of the OpenDID SD-JWT SDK checks the following.
    - KB-JWT signature verification with `cnf.jwk`
    - `aud` and `nonce` match the values expected by the verifier
    - `sd_hash` is present and matches
    - `iat` is not in the future and is within 5 minutes of creation (allowed clock skew of 60 seconds)


<div style="page-break-after: always;"></div>

## 3. Credential Configuration

### 3.1. Credential Configuration Structure

A Credential Configuration is the definition of a Credential to be issued. It is registered in the Admin Console
and published in `credential_configurations_supported` of the Issuer Metadata with the Configuration ID as the key (see `[OID4VCI-ISSUANCE]` 3.1).
The Credential Configuration of an SD-JWT VC has the following structure.

```c#
def object SdJwtVcCredentialConfiguration: "credential_configurations_supported entry (SD-JWT VC)"
{
    + CREDENTIAL_FORMAT     "format": "Credential format", value("dc+sd-jwt-did")
    + string                "vct"   : "Credential type"
    - string                "scope" : "OAuth 2.0 scope"
    - array(BINDING_METHOD) "cryptographic_binding_methods_supported": "Holder binding methods"
    - array(JWS_ALGORITHM)  "credential_signing_alg_values_supported": "Credential signature algorithms"
    - object                "proof_types_supported": "Supported information per proof type"
    {
        + object "jwt": "jwt Proof"
        {
            + array(JWS_ALGORITHM) "proof_signing_alg_values_supported": "Proof signature algorithms"
        }
    }
    - object "credential_metadata": "Credential display and claim information"
    {
        - array(CredentialDisplay) "display": "Credential display information"
        - array(ClaimDescription)  "claims" : "List of claim definitions"
    }
}

def object CredentialDisplay: "Credential display information"
{
    + string   "name"  : "Credential name"
    - LANGUAGE "locale": "Language code"
    - object   "logo"  : "Logo"
    {
        + url    "uri"     : "Logo image URL"
        - string "alt_text": "Alternative text"
    }
}

def object ClaimDescription: "Claim definition"
{
    + array(string)    "path"      : "Claim path", min_count(1)
    - bool             "mandatory" : "Whether mandatory", default(false)
    - CLAIM_VALUE_TYPE "value_type": "Value type (OpenDID extension)", default("string")
    - array(string)    "enum"      : "List of allowed values (OpenDID extension)"
    - array(object)    "display"   : "Claim display information"
    {
        + string   "name"  : "Claim display name"
        - LANGUAGE "locale": "Language code"
    }
}
```

- `proof_types_supported`: OpenDID verifies the signature of `jwt` Proofs, and since the Holder key is P-256, `proof_signing_alg_values_supported` is ["ES256"].
- `value_type`, `enum`: OpenDID extension members not defined in `[OID4VCI]`, used for input validation and value conversion in user-initiated issuance (Webview).


### 3.2. Mapping to SD-JWT VC

| Credential Configuration                       | SD-JWT VC                     | Description                                                                         |
| ---------------------------------------------- | ----------------------------- | ----------------------------------------------------------------------------------- |
| `format`                                       | Header `typ`                  | "dc+sd-jwt-did"                                                                      |
| `vct`                                          | Payload `vct`                 | Used as-is. If the value is missing or empty, the Credential is not issued.         |
| `credential_signing_alg_values_supported`      | Header `alg`                  | Publishes the `alg` value corresponding to the Issuer signing key algorithm.        |
| `cryptographic_binding_methods_supported`      | `cnf`                         | "jwk": The Holder public key is included in `cnf.jwk`.                               |
| `proof_types_supported.jwt`                    | `cnf.jwk`                     | The Holder public key verified through the Proof JWT becomes `cnf.jwk`.             |
| `credential_metadata.claims[].path[0]`         | Claim name of the Disclosure  | Top-level name of the Subject claim                                                 |
| `credential_metadata.claims[].value_type`      | Claim value type of the Disclosure | See the table below                                                            |
| `credential_metadata.claims[].mandatory`       | -                             | Whether input is mandatory in user-initiated issuance                               |
| `credential_metadata.claims[].display`         | -                             | Claim display information for the wallet. Not included in the Credential.          |
| `credential_metadata.display`                  | -                             | Credential display information for the wallet. Not included in the Credential.     |

- The Issuer finds the Credential Configuration corresponding to the Credential type to be issued (Configuration ID or `credential_identifiers`) and takes its `vct`.
- Subject claim values are the claim data the Issuer keeps per user; in user-initiated issuance, they are the Webview input values.
- Since OpenDID 3.0 creates Disclosures per top-level Subject claim name, the `path` of a claim definition consists of a single top-level claim name.

In user-initiated issuance, input values are validated and converted according to `value_type` as follows and become the claim values of the Disclosures.

| `value_type`          | Input Validation                         | Claim Value (JSON) |
| --------------------- | ---------------------------------------- | ------------------ |
| "string" (default)    | One of the list if `enum` is present     | string             |
| "full-date", "date"   | YYYY-MM-DD date                          | string             |
| "uint"                | Integer greater than or equal to 0       | number             |
| "integer"             | Integer                                  | number             |
| "boolean"             | "true" or "false"                        | boolean            |
| "list"                | JSON array                               | array              |
| "jpeg"                | Base64-encoded JPEG image (up to 2MB)    | string             |


### 3.3. Credential Configuration Example

The following is an example of a PID Credential Configuration whose Configuration ID is `VerifiableIdSD` and whose `vct` is `org:omnione:pid:1`.

```json
{
  "VerifiableIdSD": {
    "format": "dc+sd-jwt-did",
    "vct": "org:omnione:pid:1",
    "scope": "org.omnione.pid_vc_sd_jwt",
    "cryptographic_binding_methods_supported": ["jwk"],
    "credential_signing_alg_values_supported": ["ES256"],
    "proof_types_supported": {
      "jwt": {
        "proof_signing_alg_values_supported": ["ES256"]
      }
    },
    "credential_metadata": {
      "display": [
        {
          "name": "PID (SD-JWT VC)",
          "locale": "en",
          "logo": {
            "uri": "https://issuer.example.com/public/pid.png",
            "alt_text": "A square figure of a PID"
          }
        },
        { "name": "신분증명 (SD-JWT VC)", "locale": "ko" }
      ],
      "claims": [
        {
          "path": ["family_name"],
          "mandatory": true,
          "value_type": "string",
          "display": [
            { "name": "Family Name(s)", "locale": "en" },
            { "name": "성", "locale": "ko" }
          ]
        },
        {
          "path": ["given_name"],
          "mandatory": true,
          "value_type": "string",
          "display": [
            { "name": "Given Name(s)", "locale": "en" },
            { "name": "이름", "locale": "ko" }
          ]
        },
        {
          "path": ["birthdate"],
          "mandatory": true,
          "value_type": "full-date",
          "display": [
            { "name": "Birth Date", "locale": "en" },
            { "name": "생년월일", "locale": "ko" }
          ]
        },
        {
          "path": ["nationalities"],
          "mandatory": true,
          "value_type": "list",
          "display": [
            { "name": "Nationalities", "locale": "en" },
            { "name": "국적", "locale": "ko" }
          ]
        },
        {
          "path": ["email_address"],
          "mandatory": false,
          "value_type": "string",
          "display": [
            { "name": "Email Address", "locale": "en" },
            { "name": "이메일", "locale": "ko" }
          ]
        },
        {
          "path": ["issuing_authority"],
          "mandatory": true,
          "value_type": "string",
          "display": [
            { "name": "Issuance Authority", "locale": "en" },
            { "name": "발급 기관", "locale": "ko" }
          ]
        },
        {
          "path": ["issuing_country"],
          "mandatory": true,
          "value_type": "string",
          "display": [
            { "name": "Issuing Country", "locale": "en" },
            { "name": "발급 국가", "locale": "ko" }
          ]
        },
        {
          "path": ["date_of_issuance"],
          "mandatory": true,
          "value_type": "full-date",
          "display": [
            { "name": "Issuance Date", "locale": "en" },
            { "name": "발급일", "locale": "ko" }
          ]
        },
        {
          "path": ["date_of_expiry"],
          "mandatory": true,
          "value_type": "full-date",
          "display": [
            { "name": "Expiry Date", "locale": "en" },
            { "name": "만료일", "locale": "ko" }
          ]
        }
      ]
    }
  }
}
```


<div style="page-break-after: always;"></div>

## 4. SD-JWT VC Example

The following is an example of an SD-JWT VC issued with the Credential Configuration in [3.3](#33-credential-configuration-example).
The optional `email_address` was not entered, so it has no Disclosure.

### 4.1. Issuer-signed JWT Header

```json
{
    "alg": "ES256",
    "kid": "did:omn:issuer?versionId=1#assert",
    "typ": "dc+sd-jwt-did"
}
```

### 4.2. Issuer-signed JWT Payload

- `iat`: 2026-10-22T01:00:00Z
- `exp`: 2027-10-22T01:00:00Z (`iat` + 365 days)

```json
{
    "iss": "did:omn:issuer",
    "iat": 1792630800,
    "exp": 1824166800,
    "vct": "org:omnione:pid:1",
    "cnf": {
        "jwk": {
            "kty": "EC",
            "use": "sig",
            "crv": "P-256",
            "x": "N4QFeZ_n4e2uqgJ8PJPLcrTYlFltNiG1F7V5Lu27mxc",
            "y": "T92_uJkiTIxn-dLk2v8pwRFbnJPY98RT8ACK0hOVdWs"
        }
    },
    "status": {
        "status_list": {
            "idx": 42,
            "uri": "https://issuer.example.com/status-lists/1"
        }
    },
    "_sd": [
        "pIXxRoVX4B_Mfrwi9WzsgxmTsTGm47Oe-LIvi3ELVJ4",
        "jIYrxCNHmbkup3RXQHKEEnCRmB3q4fGn-CIStwmiX8E",
        "YX4yigTb4eJb5hb_6xjUKrUjwohvO0_I6xPnKQoxh98",
        "VNAb-e6XyjymxdoNID1MhzU5EwSpzH1CCeZCT3jkbq0",
        "GZs049wbu7k1MH8CqQ2gC8Rse3cKQfcGWXXNtgBn95c",
        "E6V5T0j3XZCXUBu23S4MAmCFxlLYtZM7ib7vEbp9Ouw",
        "nQ8h8dnZoLCp2m6EsTGB5GBF3mH-MbzDkrIaWXsiCq8",
        "nJ8WDdR2TObrAM_QMmTgkwTDqQQWus91kog93UKaSiE"
    ]
}
```

### 4.3. Disclosure

The Disclosure of the `family_name` claim is created as follows.

- JSON array

    ```json
    ["rlqUSNosDPHSPTo8ALbTiw","family_name","Kim"]
    ```

- Disclosure (base64url)

    ```
    WyJybHFVU05vc0RQSFNQVG84QUxiVGl3IiwiZmFtaWx5X25hbWUiLCJLaW0iXQ
    ```

- digest (base64url(SHA-256(Disclosure))): the 6th item of `_sd`

    ```
    E6V5T0j3XZCXUBu23S4MAmCFxlLYtZM7ib7vEbp9Ouw
    ```

All Disclosures are as follows.

| Claim Name          | JSON Array                                                                             | digest                                        |
| ------------------- | -------------------------------------------------------------------------------------- | --------------------------------------------- |
| `family_name`       | `["rlqUSNosDPHSPTo8ALbTiw","family_name","Kim"]`                                       | `E6V5T0j3XZCXUBu23S4MAmCFxlLYtZM7ib7vEbp9Ouw` |
| `given_name`        | `["IQ8yMU6js9Sonxu9McEmgg","given_name","Raon"]`                                       | `pIXxRoVX4B_Mfrwi9WzsgxmTsTGm47Oe-LIvi3ELVJ4` |
| `birthdate`         | `["HTeXcwGWHV4klVl5Dg_V9g","birthdate","1990-01-01"]`                                  | `GZs049wbu7k1MH8CqQ2gC8Rse3cKQfcGWXXNtgBn95c` |
| `nationalities`     | `["JSV1EOobkDmD64NVGNctbw","nationalities",["KR"]]`                                    | `VNAb-e6XyjymxdoNID1MhzU5EwSpzH1CCeZCT3jkbq0` |
| `issuing_authority` | `["ncJq9BZtdLXDxwsg2mLshw","issuing_authority","Ministry of the Interior and Safety"]` | `jIYrxCNHmbkup3RXQHKEEnCRmB3q4fGn-CIStwmiX8E` |
| `issuing_country`   | `["NcEO3hJeGEyBMQEZE2M4kQ","issuing_country","KR"]`                                    | `nJ8WDdR2TObrAM_QMmTgkwTDqQQWus91kog93UKaSiE` |
| `date_of_issuance`  | `["_nx6oBO9iNvCD10KxeOX8g","date_of_issuance","2026-10-22"]`                           | `YX4yigTb4eJb5hb_6xjUKrUjwohvO0_I6xPnKQoxh98` |
| `date_of_expiry`    | `["_WABunoJPOIHwbUpTduEgA","date_of_expiry","2036-10-21"]`                             | `nQ8h8dnZoLCp2m6EsTGB5GBF3mH-MbzDkrIaWXsiCq8` |

### 4.4. Compact Serialization

The compact serialization of the issued SD-JWT VC is as follows.
For readability, lines are broken and parts are abbreviated with `...`; the actual value contains no line breaks.

```
eyJhbGciOiJFUzI1NiIsImtpZCI6ImRpZDpvbW46aXNzdWVyP3ZlcnNpb25JZD0xI2Fzc2VydCIsInR5cCI6ImRjK3NkLWp3dC1kaWQifQ
.eyJpc3MiOiJkaWQ6b21uOmlzc3VlciIsImlhdCI6MTc5MjYzMDgwMCwiZXhwIjoxODI0MTY2ODAwLCJ2Y3QiOiJvcmc6b21uaW9uZTpwaWQ6MSIs...
.<Issuer signature>
~WyJybHFVU05vc0RQSFNQVG84QUxiVGl3IiwiZmFtaWx5X25hbWUiLCJLaW0iXQ
~WyJJUTh5TVU2anM5U29ueHU5TWNFbWdnIiwiZ2l2ZW5fbmFtZSIsIlJhb24iXQ
~WyJIVGVYY3dHV0hWNGtsVmw1RGdfVjlnIiwiYmlydGhkYXRlIiwiMTk5MC0wMS0wMSJd
~WyJKU1YxRU9vYmtEbUQ2NE5WR05jdGJ3IiwibmF0aW9uYWxpdGllcyIsWyJLUiJdXQ
~WyJuY0pxOUJadGRMWER4d3NnMm1Mc2h3IiwiaXNzdWluZ19hdXRob3JpdHkiLCJNaW5pc3RyeSBvZiB0aGUgSW50ZXJpb3IgYW5kIFNhZmV0eSJd
~WyJOY0VPM2hKZUdFeUJNUUVaRTJNNGtRIiwiaXNzdWluZ19jb3VudHJ5IiwiS1IiXQ
~WyJfbng2b0JPOWlOdkNEMTBLeGVPWDhnIiwiZGF0ZV9vZl9pc3N1YW5jZSIsIjIwMjYtMTAtMjIiXQ
~WyJfV0FCdW5vSlBPSUh3YlVwVGR1RWdBIiwiZGF0ZV9vZl9leHBpcnkiLCIyMDM2LTEwLTIxIl0
~
```

### 4.5. Credential Response

The Credential Issuer delivers the compact serialization string in `credentials[].credential` of the Credential Response (see `[OID4VCI]`).

```json
{
  "credentials": [
    {
      "credential": "eyJhbGciOiJFUzI1NiIsImtpZCI6ImRpZDpvbW46aXNzdWVyP3ZlcnNpb25JZD0xI2Fzc2VydCIsInR5cCI6ImRjK3NkLWp3dC1kaWQifQ.eyJpc3Mi...~WyJybHFVU05v...~...~"
    }
  ]
}
```

### 4.6. Presentation Format

The following is an example of a presentation in which the Holder discloses only `family_name`, `given_name`, and `birthdate` and appends a KB-JWT.

```
<Issuer-signed JWT>
~WyJybHFVU05vc0RQSFNQVG84QUxiVGl3IiwiZmFtaWx5X25hbWUiLCJLaW0iXQ
~WyJJUTh5TVU2anM5U29ueHU5TWNFbWdnIiwiZ2l2ZW5fbmFtZSIsIlJhb24iXQ
~WyJIVGVYY3dHV0hWNGtsVmw1RGdfVjlnIiwiYmlydGhkYXRlIiwiMTk5MC0wMS0wMSJd
~<KB-JWT>
```

- KB-JWT Header

    ```json
    {
        "alg": "ES256",
        "typ": "kb+jwt"
    }
    ```

- KB-JWT Payload

    ```json
    {
        "aud": "https://verifier.example.com",
        "nonce": "n-0S6_WzA2Mj",
        "iat": 1792717200,
        "sd_hash": "cIHtoFLSf6s0EnPehFj6QAJHBUf87QgkQSjcdX6m9dE"
    }
    ```

- `sd_hash` is the SHA-256 hash of `<Issuer-signed JWT>~<family_name Disclosure>~<given_name Disclosure>~<birthdate Disclosure>~`.
