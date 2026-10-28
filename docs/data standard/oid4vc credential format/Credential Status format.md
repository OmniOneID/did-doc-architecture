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

Credential Status format
==

- Topic
    - Definition of the status management data formats for Credentials issued via OID4VCI
    - Definition of the status reference in Credentials, the Status List Token, and the Credential revocation request/response data
- Author: OpenSource Development Team
- Date: 2026-10-22
- Version: v1.0.0


Revision History
---

| Version | Date       | Changes                                          |
| ------- | ---------- | ------------------------------------------------ |
| v1.0.0  | 2026-10-22 | Initial version                                  |


<div style="page-break-after: always;"></div>

Table of Contents
---

<!-- TOC tocDepth:2..4 chapterDepth:2..6 -->

- [1. Overview](#1-overview)
    - [1.1. Reference Documents](#11-reference-documents)
- [2. General Regulations](#2-general-regulations)
    - [2.1. Applicable Credentials](#21-applicable-credentials)
    - [2.2. Status Values](#22-status-values)
        - [2.2.1. Status Value Definitions](#221-status-value-definitions)
        - [2.2.2. Status Transitions](#222-status-transitions)
        - [2.2.3. Bits per Status](#223-bits-per-status)
    - [2.3. Data Types and Constants](#23-data-types-and-constants)
- [3. Status Reference in Credentials](#3-status-reference-in-credentials)
    - [3.1. `CredentialStatusRef` object](#31-credentialstatusref-object)
    - [3.2. Status List URI](#32-status-list-uri)
    - [3.3. Index](#33-index)
    - [3.4. SD-JWT VC](#34-sd-jwt-vc)
    - [3.5. ISO mdoc](#35-iso-mdoc)
- [4. Status List Token](#4-status-list-token)
    - [4.1. Status List Retrieval](#41-status-list-retrieval)
    - [4.2. JOSE Header](#42-jose-header)
    - [4.3. Payload](#43-payload)
    - [4.4. `lst` Encoding](#44-lst-encoding)
        - [4.4.1. Status Byte Array](#441-status-byte-array)
        - [4.4.2. Compression and Encoding](#442-compression-and-encoding)
    - [4.5. Status Value Check](#45-status-value-check)
    - [4.6. Example](#46-example)
- [5. Credential Revocation (OpenDID Extension)](#5-credential-revocation-opendid-extension)
    - [5.1. Processing Flow](#51-processing-flow)
    - [5.2. c_nonce Response](#52-c_nonce-response)
    - [5.3. Revocation Token](#53-revocation-token)
        - [5.3.1. JOSE Header](#531-jose-header)
        - [5.3.2. Payload](#532-payload)
        - [5.3.3. Verification Rules](#533-verification-rules)
    - [5.4. Revocation Request](#54-revocation-request)
    - [5.5. Revocation Response](#55-revocation-response)
    - [5.6. Error Response](#56-error-response)
    - [5.7. Example](#57-example)

<!-- /TOC -->


<div style="page-break-after: always;"></div>

## 1. Overview

This document defines the data formats used for the status management of Credentials (SD-JWT VC, ISO mdoc) issued via OID4VCI in OpenDID 3.0.
Status management complies with `[TOKEN-STATUS]` (Token Status List), and Credential revocation by the wallet is defined as an OpenDID extension to OID4VCI.
For the overall concepts and procedures of status management, refer to Chapter 5 of `[OID4VCI-ISSUANCE]`.

The data defined in this document and their producers and consumers are as follows.

| Data                                  | Producer                          | Consumer                | Reference                                            |
| ------------------------------------- | --------------------------------- | ----------------------- | ---------------------------------------------------- |
| Status reference in Credentials       | Credential Issuer (at issuance)   | Wallet, Verifier        | [3](#3-status-reference-in-credentials)              |
| Status List Token                     | Status List Endpoint              | Verifier, Wallet        | [4](#4-status-list-token)                            |
| Revocation Token, revocation request  | Wallet                            | Credential Issuer       | [5](#5-credential-revocation-opendid-extension)      |

The following must adhere to the definitions in this document.

- Structure and location of the status reference in Credentials
- Status values and status transition rules
- Structure of the Status List Token and the `lst` encoding rules
- Revocation Token and Credential revocation request/response formats

The following are _out of the scope_ of this document and can be defined freely by the implementation.

- Storage and operation of Status Lists (storage structure, list replacement, deactivation, etc.)
- API for status changes by the administrator
- Status List check function of the Verifier server
- Credential issuance procedure (refer to `[OID4VCI-ISSUANCE]`)


### 1.1. Reference Documents

| Reference          | Document Name                                            | Location                                                                                                |
| ------------------ | -------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| [TOKEN-STATUS]     | Token Status List (draft-ietf-oauth-status-list)         | https://datatracker.ietf.org/doc/draft-ietf-oauth-status-list/                                          |
| [OID4VCI]          | OpenID for Verifiable Credential Issuance 1.0            | https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0.html                               |
| [SD-JWT-VC]        | SD-JWT-based Verifiable Credentials (SD-JWT VC)          | https://datatracker.ietf.org/doc/draft-ietf-oauth-sd-jwt-vc/                                            |
| [ISO-18013-5]      | ISO/IEC 18013-5 Mobile driving licence (mDL) application |                                                                                                         |
| [RFC7515]          | JSON Web Signature (JWS)                                 | https://www.rfc-editor.org/rfc/rfc7515                                                                  |
| [RFC7518]          | JSON Web Algorithms (JWA)                                | https://www.rfc-editor.org/rfc/rfc7518                                                                  |
| [RFC7519]          | JSON Web Token (JWT)                                     | https://www.rfc-editor.org/rfc/rfc7519                                                                  |
| [RFC1950]          | ZLIB Compressed Data Format Specification version 3.3    | https://www.rfc-editor.org/rfc/rfc1950                                                                  |
| [RFC1951]          | DEFLATE Compressed Data Format Specification version 1.3 | https://www.rfc-editor.org/rfc/rfc1951                                                                  |
| [RFC4648]          | The Base16, Base32, and Base64 Data Encodings            | https://www.rfc-editor.org/rfc/rfc4648                                                                  |
| [OSD]              | OpenDID Schema Definition Language                       | [OpenDID Schema Definition Language.md](../../guide/osd/OpenDID%20Schema%20Definition%20Language.md)    |
| [DATA-SPEC]        | (OpenDID) Data Specification                             | [Data Specification.md](../data%20specification/Data%20Specification.md)                                |
| [DID-DOC-FORMAT]   | (OpenDID) DID Document format                            | [DID Document format.md](../did%20document%20format/DID%20Document%20format.md)                         |
| [OID4VCI-ISSUANCE] | (OpenDID) OID4VCI-based VC Issuance                      | [OID4VCI Issuance.md](../../concepts/OID4VCI%20Issuance.md)                                             |
| [P240]             | (OpenDID) P240 : OID4VCI Credential Revocation Sequence  | [P240_oid4vci_revoke_credential.puml](../../sequence/detailed/P240_oid4vci_revoke_credential.puml)      |
| [ISSUER-API]       | (OpenDID) Issuer API                                     | did-issuer-server `docs/api/Issuer_API.md`                                                              |


<div style="page-break-after: always;"></div>

## 2. General Regulations

### 2.1. Applicable Credentials

Status management applies to Credentials of the following formats, which are signed with the Issuer DID key.

| Format          | Credential | Location of the status reference                       | Reference                  |
| --------------- | ---------- | ------------------------------------------------------ | -------------------------- |
| `dc+sd-jwt-did` | SD-JWT VC  | `status` Claim in the Issuer-signed JWT Payload        | [3.4](#34-sd-jwt-vc)       |
| `mso_mdoc-did`  | ISO mdoc   | `status` item in the MSO (Mobile Security Object)      | [3.5](#35-iso-mdoc)        |

- To issue a Credential to which status management applies, the Proof JWT of the Credential request must identify the Holder key with `kid` (Holder DID URL).
  The Credential Issuer stores this Holder DID in the issuance record and uses it to verify revocation requests from the wallet (refer to [5](#5-credential-revocation-opendid-extension)).
- A separate Status List is created for each Credential format. Therefore, SD-JWT VCs and mdocs reference different Status Lists.
- One Credential references one index of one Status List.


### 2.2. Status Values

#### 2.2.1. Status Value Definitions

| Status      | Value | Description                                                                        |
| ----------- | ----- | ---------------------------------------------------------------------------------- |
| `VALID`     | 0     | • Valid<br>• Initial status at Credential issuance                                  |
| `INVALID`   | 1     | • Revoked<br>• Final status; cannot be made valid again                             |
| `SUSPENDED` | 2     | • Temporarily suspended<br>• Can return to `VALID`                                  |
| `RESERVED`  | 3     | • Reserved value<br>• Final status; no status transition in OpenDID 3.0 leads to this value |

- The values of `VALID`, `INVALID`, and `SUSPENDED` are the same as the Status Type values defined in `[TOKEN-STATUS]`.
- The value of an index in the Status List that is not allocated to any Credential is 0.
- An index allocated to a Credential whose issuance failed is not reused, and its value remains 0.

#### 2.2.2. Status Transitions

![Credential Status Transitions](../../concepts/images/oid4vci_state_credential_status.svg)

| Current Status | Allowed Next Status     |
| -------------- | ----------------------- |
| `VALID`        | `SUSPENDED`, `INVALID`  |
| `SUSPENDED`    | `VALID`, `INVALID`      |
| `INVALID`      | None (final status)     |
| `RESERVED`     | None (final status)     |

- A request to change to the same status as the current status is allowed, and no change occurs.
- The new status value must be representable with the `bits` of the Status List (refer to [2.2.3](#223-bits-per-status)).
- When the status is changed, the value at the corresponding index of the Status List is updated, and the previous status, new status, reason, changer, and change time are recorded in the history.
- The status can be changed by the following parties.
    - Issuer administrator: changes the status in the Admin Console, and must enter a reason.
    - Wallet: can change the status only to `INVALID` with the revocation request in [5](#5-credential-revocation-opendid-extension). The reason is recorded as `USER_REQUEST`, and the changer as the Holder DID.

#### 2.2.3. Bits per Status

A Status List represents the status value of one index with `bits` bits.

| `bits` | Representable values | Representable statuses                           |
| ------ | -------------------- | ------------------------------------------------ |
| 1      | 0 ~ 1                | `VALID`, `INVALID`                               |
| 2      | 0 ~ 3                | `VALID`, `INVALID`, `SUSPENDED`, `RESERVED`      |
| 4      | 0 ~ 15               | `VALID`, `INVALID`, `SUSPENDED`, `RESERVED`      |
| 8      | 0 ~ 255              | `VALID`, `INVALID`, `SUSPENDED`, `RESERVED`      |

- `bits` is one of 1, 2, 4, and 8, and the default is 2.
- A Credential in a Status List whose `bits` is 1 cannot be changed to `SUSPENDED`.
- The default settings of the Status List automatically created by the Issuer server are as follows.

| Item       | Default | Description                                        |
| ---------- | ------- | -------------------------------------------------- |
| `bits`     | 2       | Bits per status                                    |
| `capacity` | 100,000 | Number of indexes that can be allocated in the Status List |
| `ttl`      | 300     | Cache lifetime of the Status List Token (seconds)  |


### 2.3. Data Types and Constants

For items not defined here, refer to `[DATA-SPEC]`.

```c#
def string statusListUri  : "Status List URI (absolute URI)"
def int    statusListIndex: "Status List index", min_value(0)
def int    numericDate    : "NumericDate. Seconds since 1970-01-01T00:00:00Z ([RFC7519])"
def string base64url      : "base64url string without padding ([RFC4648] Section 5)"
def string compactJws     : "JWS Compact Serialization string ([RFC7515])"

def enum CREDENTIAL_STATUS: "Credential status value recorded in the Status List"
{
    0: "VALID",
    1: "INVALID",
    2: "SUSPENDED",
    3: "RESERVED",
}

def enum CREDENTIAL_STATUS_NAME: "Credential status name"
{
    "VALID"    : "Valid (0)",
    "INVALID"  : "Revoked (1)",
    "SUSPENDED": "Suspended (2)",
    "RESERVED" : "Reserved (3)",
}

def enum STATUS_BITS: "Bits per status"
{
    1, 2, 4, 8
}

def enum STATUS_CREDENTIAL_FORMAT: "Credential formats to which status management applies"
{
    "dc+sd-jwt-did": "SD-JWT VC",
    "mso_mdoc-did" : "ISO mdoc",
}
```


<div style="page-break-after: always;"></div>

## 3. Status Reference in Credentials

### 3.1. `CredentialStatusRef` object

A Credential includes, as its status reference, the URI and index of the Status List in which its status is recorded.

```c#
def object StatusListRef: "Token Status List reference"
{
    + statusListUri   "uri": "Status List URI"
    + statusListIndex "idx": "Index in the Status List"
}

def object CredentialStatusRef: "Credential status reference"
{
    + StatusListRef "status_list": "Token Status List reference"
}
```

```json
{
    "status_list": {
        "uri": "https://issuer.example.com/status-lists/1",
        "idx": 7
    }
}
```

The same structure is used in the following locations.

| Location                                   | Item name           | Reference                                      |
| ------------------------------------------ | ------------------- | ---------------------------------------------- |
| Issuer-signed JWT Payload of the SD-JWT VC | `status`            | [3.4](#34-sd-jwt-vc)                           |
| MSO of the mdoc                            | `status`            | [3.5](#35-iso-mdoc)                            |
| Revocation Token Payload                   | `credential_status` | [5.3.2](#532-payload)                          |
| Credential revocation response             | `credential_status` | [5.5](#55-revocation-response) (`status` added) |


### 3.2. Status List URI

The Status List URI (`status_list.uri`) has the following format.

```
{Issuer base URL}/status-lists/{listId}
```

- `{Issuer base URL}`: the base URL setting of the Issuer server (`issuer.base-url`) with any trailing `/` removed
- `{listId}`: Status List identifier (in the Issuer server, the numeric ID of the Status List)
- If a Credential is issued when no Status List exists for its format, a Status List is created automatically.
- The Status List URI is fixed and stored when the Status List is created, and does not change even if the settings change later. Issued Credentials keep using the URI from the time of issuance.
- The Status List URI is the same as `sub` of the Status List Token (refer to [4.3](#43-payload)).
- Wallets and verifiers must use the URI included in the Credential as it is, and must not compose or modify the URI.
  They also must not derive the Credential revocation Endpoint URL from the Status List URI (refer to [5.1](#51-processing-flow)).


### 3.3. Index

- The index (`status_list.idx`) is an integer starting from 0, and is allocated in issuance order within the same Status List, increasing by 1.
- The index is less than the `capacity` of the Status List. If no Status List is available for allocation, Credential issuance fails.
- The combination of `uri` and `idx` uniquely identifies one issued Credential.
- An index, once allocated, is not reused. This also applies when Credential generation fails.
- If multiple Credentials are issued because one Credential request contains multiple Proofs, an index is allocated to each Credential separately.
- The index is represented as a JSON number in SD-JWT VCs and as a CBOR unsigned integer in mdocs.


### 3.4. SD-JWT VC

An SD-JWT VC of the `dc+sd-jwt-did` format includes `CredentialStatusRef` in the top-level Claim `status` of the Issuer-signed JWT Payload.

- `status` is not subject to Selective Disclosure. It is not made into a Disclosure and is not included in the `_sd` array,
  so `status` is always delivered to the verifier together with the Issuer-signed Payload, regardless of which Claims the wallet presents.
- Since `status` is within the scope of the Issuer signature, it cannot be modified by the wallet or a third party.
- After generating the Credential, the Credential Issuer checks that `status.status_list` in the signed Payload matches the allocated `uri` and `idx`, and then completes the issuance.

```json
// Issuer-signed JWT Header
{
    "alg": "ES256",
    "typ": "dc+sd-jwt-did",
    "kid": "did:omn:issuer?versionId=1#assert"
}
```

```json
// Issuer-signed JWT Payload (some values omitted)
{
    "iss": "did:omn:issuer",
    "iat": 1792630800,
    "exp": 1824166800,
    "vct": "org:omnione:pid:1",
    "cnf": {
        "jwk": { "kty": "EC", "crv": "P-256", "x": "...", "y": "..." }
    },
    "_sd": ["...", "..."],
    "_sd_alg": "sha-256",
    "status": {
        "status_list": {
            "uri": "https://issuer.example.com/status-lists/1",
            "idx": 7
        }
    }
}
```


### 3.5. ISO mdoc

An mdoc of the `mso_mdoc-did` format includes `CredentialStatusRef` as a CBOR map in the top-level item `status` of the MSO (MobileSecurityObject).

- Since the MSO is the payload of IssuerAuth (COSE_Sign1) and is signed with the Issuer DID key, `status` is also within the scope of the Issuer signature.
- `status` is not a Data Element (IssuerSignedItem) of a Name Space, so it is not subject to selective disclosure.
- The keys of the CBOR map are text strings; `uri` is a text string and `idx` is an unsigned integer.

```
/ MobileSecurityObject (CBOR diagnostic notation, other items omitted) /
{
    "docType": "org.omnione.pid.1",
    / version, digestAlgorithm, valueDigests, deviceKeyInfo, validityInfo omitted /
    "status": {
        "status_list": {
            "uri": "https://issuer.example.com/status-lists/2",
            "idx": 7
        }
    }
}
```

> **Note**: Since SD-JWT VCs and mdocs use different Status Lists per format, the two Credentials in the examples above reference different Status Lists (`/status-lists/1`, `/status-lists/2`) even though their `idx` is the same.


<div style="page-break-after: always;"></div>

## 4. Status List Token

The Status List Token is a JWT that contains all status values of one Status List and is signed with the Issuer DID key (`[TOKEN-STATUS]`).

### 4.1. Status List Retrieval

| Item                     | Value                                                    |
| ------------------------ | -------------------------------------------------------- |
| Method                   | `GET`                                                    |
| URL                      | `status_list.uri` of the Credential                      |
| Authentication           | None (Public)                                            |
| Response `Content-Type`  | `application/statuslist+jwt`                             |
| Response Body            | Status List Token (JWS Compact Serialization)            |
| Response `Cache-Control` | `max-age={ttl}, public`                                  |
| CORS                     | Cross-origin requests from all Origins are allowed       |

- If the request includes an `Accept` header, it must accept `application/statuslist+jwt` (or `*/*`).
- The Status List Token is newly generated and signed with the current status for each retrieval request. Therefore, `iat` is the time the response is generated.
- A Status List that no longer allocates new indexes (deactivated) can still be retrieved.
- `max-age` of the response `Cache-Control` is set to the same value (in seconds) as `ttl` of the token.


### 4.2. JOSE Header

```c#
def object StatusListTokenHeader: "Status List Token JOSE Header"
{
    + string    "alg": "Signature algorithm", value("ES256")
    + didKeyUrl "kid": "Issuer DID URL of the Status List signing key"
    + string    "typ": "Token type", value("statuslist+jwt")
}
```

- `kid`: DID URL (including the key fragment) of the Issuer that issued the Credential. Example: `did:omn:issuer?versionId=1#assert`
- The signature is ES256 (`[RFC7518]`) using a P-256 key, and the signature value is the 64-byte concatenation of the 32-byte `R` and the 32-byte `S` (`[RFC7515]`).

```json
{
    "alg": "ES256",
    "kid": "did:omn:issuer?versionId=1#assert",
    "typ": "statuslist+jwt"
}
```


### 4.3. Payload

```c#
def object StatusListTokenPayload: "Status List Token Payload"
{
    + statusListUri "sub"        : "Status List URI"
    + numericDate   "iat"        : "Token issuance time"
    + numericDate   "exp"        : "Token expiration time (iat + ttl)"
    + int           "ttl"        : "Cache lifetime (seconds)", min_value(1)
    + object        "status_list": "Status List"
    {
        + STATUS_BITS "bits": "Bits per status"
        + base64url   "lst" : "Status byte array, compressed and encoded"
    }
}
```

- `sub`: Status List URI. It is the same as `status_list.uri` of the Credential.
- `exp`: `iat + ttl`
- `ttl`: Time (in seconds) for which the Status List Token can be cached. The default is 300.
- `status_list.bits`: Bits per status of the Status List (refer to [2.2.3](#223-bits-per-status))
- `status_list.lst`: Array of status values encoded according to [4.4](#44-lst-encoding)
- The Payload does not include `iss`. The signer is identified by `kid` in the Header.

```json
{
    "sub": "https://issuer.example.com/status-lists/1",
    "iat": 1792630800,
    "exp": 1792631100,
    "ttl": 300,
    "status_list": {
        "bits": 2,
        "lst": "eNpTaQUAAM8Aqg"
    }
}
```


### 4.4. `lst` Encoding

#### 4.4.1. Status Byte Array

The status values of a Status List are written to a byte array according to the following rules.

- The length of the byte array is `ceil(capacity × bits / 8)` bytes.
- The status value of index `i` is written to the `bits` bits starting at bit position `i × bits`.
    - Byte position: `floor(i × bits / 8)`
    - Bit position within the byte (shift): `(i × bits) mod 8`
- Within one byte, lower indexes are filled starting from the least significant bit (LSB). That is, a lower index is located in the lower bits of the byte.
- Since `bits` is one of 1, 2, 4, and 8, one status value never spans two bytes.
- The value of an index whose status value is not written is 0.

When `bits` is 2, the relationship between bytes and indexes is as follows.

| Byte | Bit 7-6 | Bit 5-4 | Bit 3-2 | Bit 1-0 |
| ---- | ------- | ------- | ------- | ------- |
| 0    | idx 3   | idx 2   | idx 1   | idx 0   |
| 1    | idx 7   | idx 6   | idx 5   | idx 4   |
| ...  | ...     | ...     | ...     | ...     |

When `bits` is 1, bit 0 of byte 0 is idx 0, bit 7 is idx 7, and bit 0 of byte 1 is idx 8.

#### 4.4.2. Compression and Encoding

1. Compress the status byte array in the ZLIB format (`[RFC1950]`). The compression algorithm is DEFLATE (`[RFC1951]`), and the highest compression level (9) is used.
    - The result is a ZLIB stream that includes the ZLIB header and the Adler-32 checksum; it is not raw DEFLATE.
1. Encode the compression result in base64url without padding (`[RFC4648]` Section 5).
1. Set the encoding result to `status_list.lst`.

> **Note**: The compression level affects only the `lst` string, not the decompressed byte array. Verifiers perform ZLIB decompression regardless of the compression level.


### 4.5. Status Value Check

The verifier (or wallet) checks the status of a Credential with the following procedure.

1. Extract the status reference (`uri`, `idx` of `status.status_list`) from the Credential.
    - SD-JWT VC: `status` in the Issuer-signed JWT Payload
    - mdoc: `status` in the MSO
1. Retrieve the Status List Token with `uri` (refer to [4.1](#41-status-list-retrieval)).
1. Check that `typ` in the Header is `statuslist+jwt` and `alg` is `ES256`, and verify the signature with the key of the Issuer DID Document indicated by `kid`.
1. Check that `sub` in the Payload is the same as `uri` of the Credential, and that `exp` has not passed.
1. Decode `lst` with base64url and decompress it with ZLIB to obtain the status byte array.
1. Check that `idx × bits` is less than the bit length of the status byte array (`number of bytes × 8`).
1. Obtain the status value with the following expression, and interpret it according to [2.2.1](#221-status-value-definitions).

```python
raw   = zlib.decompress(base64url_decode(lst))
bit   = idx * bits
value = (raw[bit // 8] >> (bit % 8)) & ((1 << bits) - 1)
```

- The verifier can reuse the Status List Token of the same URI for `ttl` (or `max-age` of `Cache-Control`), and retrieves it again after `exp` has passed.
- A cached Status List Token may still contain the previous status for up to `ttl`.


### 4.6. Example

For ease of understanding, a Status List with a `capacity` of 8 and `bits` of 2 is used. The `capacity` of the Status List actually created by the Issuer server is 100,000.

**■ Status values**

| idx    | 0       | 1         | 2           | 3       | 4         | 5         | 6       | 7           |
| ------ | ------- | --------- | ----------- | ------- | --------- | --------- | ------- | ----------- |
| Status | `VALID` | `INVALID` | `SUSPENDED` | `VALID` | `INVALID` | `INVALID` | `VALID` | `SUSPENDED` |
| Value  | 0       | 1         | 2           | 0       | 1         | 1         | 0       | 2           |

**■ Status byte array** (length `ceil(8 × 2 / 8)` = 2 bytes)

| Byte | idx 3 (Bit 7-6) | idx 2 (Bit 5-4) | idx 1 (Bit 3-2) | idx 0 (Bit 1-0) | Value  |
| ---- | --------------- | --------------- | --------------- | --------------- | ------ |
| 0    | `00`            | `10`            | `01`            | `00`            | `0x24` |

| Byte | idx 7 (Bit 7-6) | idx 6 (Bit 5-4) | idx 5 (Bit 3-2) | idx 4 (Bit 1-0) | Value  |
| ---- | --------------- | --------------- | --------------- | --------------- | ------ |
| 1    | `10`            | `00`            | `01`            | `01`            | `0x85` |

**■ Compression and encoding**

| Step                       | Result                                       |
| -------------------------- | -------------------------------------------- |
| Status byte array          | `24 85`                                      |
| ZLIB compression (level 9) | `78 DA 53 69 05 00 00 CF 00 AA`              |
| base64url (`lst`)          | `eNpTaQUAAM8Aqg`                             |

If the same status values are written to a Status List with a `capacity` of 100,000 (25,000 bytes), `lst` is as follows.

```
eNrtwQEBAAAIAqDGeLXfDhHIHwAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAwIQC3MsAqg
```

**■ Status value check**

- idx 7: `bit = 7 × 2 = 14` → byte 1, shift 6 → `(0x85 >> 6) & 0x3 = 2` → `SUSPENDED`
- idx 5: `bit = 5 × 2 = 10` → byte 1, shift 2 → `(0x85 >> 2) & 0x3 = 1` → `INVALID`

**■ Status List Token response**

```http
HTTP/1.1 200 OK
Content-Type: application/statuslist+jwt
Cache-Control: max-age=300, public

eyJhbGciOiJFUzI1NiIsImtpZCI6ImRpZDpvbW46aXNzdWVyP3ZlcnNpb25JZD0xI2Fzc2VydCIsInR5cCI6InN0YXR1c2xpc3Qrand0In0.eyJzdWIiOiJodHRwczovL2lzc3Vlci5leGFtcGxlLmNvbS9zdGF0dXMtbGlzdHMvMSIsImlhdCI6MTc5MjYzMDgwMCwiZXhwIjoxNzkyNjMxMTAwLCJ0dGwiOjMwMCwic3RhdHVzX2xpc3QiOnsiYml0cyI6MiwibHN0IjoiZU5wVGFRVUFBTThBcWcifX0.<signature>
```

The decoded Header and Payload are the same as the examples in [4.2](#42-jose-header) and [4.3](#43-payload). `<signature>` is a placeholder for the ES256 signature value generated with the Issuer DID key.


<div style="page-break-after: always;"></div>

## 5. Credential Revocation (OpenDID Extension)

The wallet can directly request the Credential Issuer to revoke one Credential (`dc+sd-jwt-did`, `mso_mdoc-did`) it holds.
This feature is an OpenDID extension to OID4VCI, and it identifies the requester by a Holder DID signature without using an Access Token or TAS.

| Item                                                                         | Category           |
| ---------------------------------------------------------------------------- | ------------------ |
| Issuer Metadata retrieval, `nonce_endpoint` and `c_nonce`                    | `[OID4VCI]` standard |
| Status reference in Credentials, Status List Token                           | `[TOKEN-STATUS]`   |
| `credential_revocation_endpoint`                                             | OpenDID extension  |
| Revocation Token (`opendid-credential-revocation+jwt`) and revocation request/response | OpenDID extension  |

The Revocation Token uses JWT (`[RFC7519]`) and JWS (`[RFC7515]`), but it does not represent a standard OID4VCI revocation protocol.

### 5.1. Processing Flow

For the detailed procedure, refer to `[P240]`.

| Step | Request                                              | Response                         | Data                                                       |
| ---- | ---------------------------------------------------- | -------------------------------- | ---------------------------------------------------------- |
| 1    | `POST {nonce_endpoint}`                              | `c_nonce`                        | [5.2](#52-c_nonce-response)                                |
| 2    | (Wallet) Create the Revocation Token and sign it with the Holder DID key | -            | [5.3](#53-revocation-token)                                |
| 3    | `POST {credential_revocation_endpoint}`              | Credential revocation response   | [5.4](#54-revocation-request), [5.5](#55-revocation-response) |
| 4    | (Optional) `GET {status_list.uri}`                   | Status List Token                | [4](#4-status-list-token)                                  |

- The wallet uses the values of `nonce_endpoint` and `credential_revocation_endpoint` in the Issuer Metadata as they are, and does not compose the paths itself.
  The OpenDID Issuer server always generates `credential_revocation_endpoint` as `{credential_issuer}/credential-lifecycle/revoke`.
- The Credential to be revoked is identified by the status reference (`status_list{uri, idx}`) included in the Credential.


### 5.2. c_nonce Response

The wallet sends a `POST` request without a body to `nonce_endpoint` and receives `c_nonce` (`[OID4VCI]`).

```c#
def object NonceResponse: "c_nonce response"
{
    + base64url "c_nonce": "16-byte random value encoded in base64url without padding"
}
```

- `c_nonce` is valid for 10 minutes and can be used only once.
- One `c_nonce` is consumed only once across both Credential issuance Proofs and Credential revocation requests.

```json
{
    "c_nonce": "8qvJQO7TjZlE5M0wL8OaGA"
}
```


### 5.3. Revocation Token

The Revocation Token is a JWT that contains the status reference of the Credential to be revoked and `c_nonce`, signed with the Holder DID key, and is delivered in JWS Compact Serialization.

- The signing key is a P-256 (secp256r1) key registered in `authentication` of the Holder DID Document, and the signature algorithm is ES256.
- The JWS Signing Input is `BASE64URL(Header) + "." + BASE64URL(Payload)`, and the signature value is the 64-byte concatenation of the 32-byte `R` and the 32-byte `S` (`[RFC7515]`).

#### 5.3.1. JOSE Header

```c#
def object RevocationTokenHeader: "Revocation Token JOSE Header"
{
    + string    "alg": "Signature algorithm", value("ES256")
    + string    "typ": "Token type", value("opendid-credential-revocation+jwt")
    + didKeyUrl "kid": "Authentication key URL of the Holder DID"
}
```

- `alg`: Only `ES256` is allowed. `none` and other algorithms are not allowed.
- `kid`: Holder DID URL including the key fragment. Example: `did:omn:holder?versionId=1#pin`
    - `kid` must start with `did:` and contain `?` or `#`.
    - The DID part of `kid` (before the first `?`, or before `#` if there is no `?`) must be the same as `iss` in the Payload.
    - The key indicated by `kid` must be included in `authentication` of the Holder DID Document. Keys are compared by the fragment after `#`.

#### 5.3.2. Payload

```c#
def object RevocationTokenPayload: "Revocation Token Payload"
{
    + did                 "iss"              : "Holder DID"
    + numericDate         "iat"              : "Token issuance time"
    + numericDate         "exp"              : "Token expiration time"
    + string              "nonce"            : "c_nonce returned by nonce_endpoint"
    + CredentialStatusRef "credential_status": "Status reference of the Credential to be revoked"
}
```

- `iss`: Holder DID that signed the token. It must be the same as the Holder DID confirmed with the Proof `kid` at Credential issuance and stored in the issuance record.
- `iat`, `exp`: `exp` must be greater than `iat`, and must be within 5 minutes (300 seconds) after `iat`.
- `nonce`: Use the `c_nonce` value returned by `nonce_endpoint` as it is.
- `credential_status.status_list`: Use `uri` and `idx` included in the Credential to be revoked as they are. The Credential Issuer looks up the issuance record whose `uri` string and `idx` match exactly.
- `aud`, `sub`, `jti`, and `nbf` are not used, and other Claims are ignored.
- No revocation reason is delivered. The Credential Issuer records the reason as `USER_REQUEST`.

```json
{
    "iss": "did:omn:holder",
    "iat": 1792634400,
    "exp": 1792634700,
    "nonce": "8qvJQO7TjZlE5M0wL8OaGA",
    "credential_status": {
        "status_list": {
            "uri": "https://issuer.example.com/status-lists/1",
            "idx": 7
        }
    }
}
```

#### 5.3.3. Verification Rules

The Credential Issuer verifies the revocation request in the following order, and responds with the corresponding error code on failure (refer to [5.6](#56-error-response)).
A clock skew of 60 seconds is allowed in time verification.

| Order | Verification item                                                                                       | Error code     |
| ----- | ------------------------------------------------------------------------------------------------------- | -------------- |
| 1     | `revocation_token` is not empty and is a JWS Compact Serialization with three parts                      | `SSRVISS00302` |
| 2     | Header `alg` is `ES256` and `typ` is `opendid-credential-revocation+jwt`                                 | `SSRVISS00302` |
| 3     | `kid` and `iss` exist, and the DID part of `kid` is the same as `iss`                                    | `SSRVISS00302` |
| 4     | `iat` and `exp` exist, `exp > iat`, and `exp - iat ≤ 300` seconds                                        | `SSRVISS00302` |
| 5     | `iat` is not later than the current time + 60 seconds, and `exp` is not earlier than the current time - 60 seconds | `SSRVISS00302` |
| 6     | `nonce` and `credential_status.status_list.uri` exist, and `idx` is a non-negative integer               | `SSRVISS00302` |
| 7     | Retrieval of the DID Document of `iss`                                                                  | `SSRVISS00103` |
| 8     | `id` of the DID Document is the same as `iss`, `kid` is included in `authentication`, and the key exists | `SSRVISS00302` |
| 9     | Public key decoding and ES256 signature verification (`SSRVISS00302` if the key is not a P-256 key)      | `SSRVISS00413` |
| 10    | An issuance record for `uri` + `idx` exists and is in the issuance-completed (`ISSUED`) state           | `SSRVISS00206` |
| 11    | The Holder DID of the issuance record is the same as `iss`                                               | `SSRVISS00302` |
| 12    | `c_nonce` exists, has not expired, and has not been used (consumed after the check)                      | `SSRVISS00210` |
| 13    | Change of the status to `INVALID`                                                                        | `SSRVISS00011` |

- If the current status is already `INVALID`, the status is not changed, no history is added, and a success response is returned with the existing status change time. Even in this case, `c_nonce` is consumed.
- Otherwise, the status is changed from `VALID` or `SUSPENDED` to `INVALID`, and the history is recorded with the reason `USER_REQUEST` and the Holder DID as the changer.
- The consumption of `c_nonce` and the status change are processed in one transaction. If the status change fails, the consumption of `c_nonce` is also rolled back.


### 5.4. Revocation Request

| Item           | Value                                                      |
| -------------- | ---------------------------------------------------------- |
| Method         | `POST`                                                     |
| URL            | `credential_revocation_endpoint` in the Issuer Metadata    |
| Authentication | None (Holder DID signature of the Revocation Token)        |
| `Content-Type` | `application/json;charset=utf-8`                           |

```c#
def object CredentialRevocationRequest: "Credential revocation request"
{
    + compactJws "revocation_token": "Revocation Token"
}
```


### 5.5. Revocation Response

If the request is approved, the following response is returned with HTTP 200.

```c#
def object CredentialRevocationResponse: "Credential revocation response"
{
    + object "credential_status": "Status of the processed Credential"
    {
        + StatusListRef          "status_list": "Status List reference"
        + CREDENTIAL_STATUS_NAME "status"     : "Changed status", value("INVALID")
    }
    + utcDatetime "revoked_at": "Status change time"
}
```

- `credential_status.status_list`: `uri` and `idx` of the requested Revocation Token
- `credential_status.status`: Always `INVALID`
- `revoked_at`: UTC time in ISO 8601 format, which may include fractional seconds. For a Credential that was already `INVALID`, it is the existing status change time.
- A wallet that did not receive a response obtains a new `c_nonce` and requests again. If the Credential is already `INVALID`, a success response with the same result is returned.
- When the revocation is completed, the value at the corresponding index of the Status List becomes 1 (`INVALID`).


### 5.6. Error Response

If an error occurs, the response has the following format.

```c#
def object ErrorResponse: "Error response"
{
    + errorCode "code"       : "Error code"
    + string    "description": "Error description"
}
```

| HTTP | Error code     | Description                                                                                                          |
| ---- | -------------- | -------------------------------------------------------------------------------------------------------------------- |
| 400  | `SSRVISS00206` | No issued Credential exists for the Status List URI and index, or it is not in the issuance-completed state            |
| 400  | `SSRVISS00210` | `nonce` does not exist, has expired, or has already been used                                                         |
| 400  | `SSRVISS00302` | Verification of the format, Header, Claims, or validity period of `revocation_token` failed, Holder DID key confirmation failed, or the Holder DID does not match |
| 400  | `SSRVISS00413` | JWT signature verification or Holder public key decoding failed                                                       |
| 500  | `SSRVISS00103` | Retrieval of the Holder DID Document failed                                                                           |
| 500  | `SSRVISS00011` | Processing of the Credential status change failed                                                                     |
| 500  | `SSRVISS00702` | The Request Body cannot be read                                                                                       |

- If the Revocation Token does not contain the `nonce` Claim at all, the response is `SSRVISS00302`, not `SSRVISS00210`.
- A request for a Credential that is already `INVALID` is processed as a success, not as an error.


### 5.7. Example

**■ c_nonce request and response**

```http
POST /nonce HTTP/1.1
Host: issuer.example.com
Content-Length: 0
```

```http
HTTP/1.1 200 OK
Content-Type: application/json;charset=utf-8

{
    "c_nonce": "8qvJQO7TjZlE5M0wL8OaGA"
}
```

**■ Credential revocation request**

This example revokes the SD-JWT VC (`/status-lists/1`, idx 7) in [3.4](#34-sd-jwt-vc).

```http
POST /credential-lifecycle/revoke HTTP/1.1
Host: issuer.example.com
Content-Type: application/json;charset=utf-8

{
    "revocation_token": "eyJhbGciOiJFUzI1NiIsInR5cCI6Im9wZW5kaWQtY3JlZGVudGlhbC1yZXZvY2F0aW9uK2p3dCIsImtpZCI6ImRpZDpvbW46aG9sZGVyP3ZlcnNpb25JZD0xI3BpbiJ9.eyJpc3MiOiJkaWQ6b21uOmhvbGRlciIsImlhdCI6MTc5MjYzNDQwMCwiZXhwIjoxNzkyNjM0NzAwLCJub25jZSI6IjhxdkpRTzdUalpsRTVNMHdMOE9hR0EiLCJjcmVkZW50aWFsX3N0YXR1cyI6eyJzdGF0dXNfbGlzdCI6eyJ1cmkiOiJodHRwczovL2lzc3Vlci5leGFtcGxlLmNvbS9zdGF0dXMtbGlzdHMvMSIsImlkeCI6N319fQ.<signature>"
}
```

```json
// revocation_token Header (decoded)
{
    "alg": "ES256",
    "typ": "opendid-credential-revocation+jwt",
    "kid": "did:omn:holder?versionId=1#pin"
}
```

The decoded Payload is the same as the example in [5.3.2](#532-payload). `<signature>` is a placeholder for the ES256 signature value generated with the Holder DID key.

**■ Credential revocation response**

```http
HTTP/1.1 200 OK
Content-Type: application/json;charset=utf-8

{
    "credential_status": {
        "status_list": {
            "uri": "https://issuer.example.com/status-lists/1",
            "idx": 7
        },
        "status": "INVALID"
    },
    "revoked_at": "2026-10-22T02:00:01.123456Z"
}
```

**■ Error response**

```http
HTTP/1.1 400 Bad Request
Content-Type: application/json

{
    "code": "SSRVISS00210",
    "description": "The issuer nonce is not valid."
}
```

**■ Status List after revocation**

When idx 7 in the Status List of [4.6](#46-example) is changed from `SUSPENDED` (2) to `INVALID` (1), byte 1 changes from `0x85` to `0x45`.

| Step                       | Result                                       |
| -------------------------- | -------------------------------------------- |
| Status byte array          | `24 45`                                      |
| ZLIB compression (level 9) | `78 DA 53 71 05 00 00 8F 00 6A`              |
| base64url (`lst`)          | `eNpTcQUAAI8Aag`                             |

- idx 7: `(0x45 >> 6) & 0x3 = 1` → `INVALID`
