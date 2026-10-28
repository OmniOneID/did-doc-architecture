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

ISO mdoc format
==

- Topic
    - Definition of the data format of the ISO mdoc Credential issued via OID4VCI
    - Definition of the mapping rules between the Credential Configuration and the mdoc structure
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
- [2. General Regulations](#2-general-regulations)
    - [2.1. Credential Format](#21-credential-format)
    - [2.2. Notation](#22-notation)
    - [2.3. Encoding Rules](#23-encoding-rules)
        - [2.3.1. CBOR Value Encoding](#231-cbor-value-encoding)
        - [2.3.2. Date Encoding](#232-date-encoding)
        - [2.3.3. Portrait Image (portrait)](#233-portrait-image-portrait)
- [3. mdoc Credential Structure](#3-mdoc-credential-structure)
    - [3.1. Structure Overview](#31-structure-overview)
    - [3.2. Data Types and Constants](#32-data-types-and-constants)
    - [3.3. `IssuerSignedItem` object](#33-issuersigneditem-object)
    - [3.4. `IssuerSigned` object](#34-issuersigned-object)
    - [3.5. `MobileSecurityObject` object](#35-mobilesecurityobject-object)
        - [3.5.1. `valueDigests`](#351-valuedigests)
        - [3.5.2. `deviceKeyInfo`](#352-devicekeyinfo)
        - [3.5.3. `validityInfo`](#353-validityinfo)
        - [3.5.4. `status`](#354-status)
        - [3.5.5. MSO Extension Fields](#355-mso-extension-fields)
    - [3.6. `IssuerAuth` (COSE_Sign1)](#36-issuerauth-cose_sign1)
        - [3.6.1. Headers and Issuer Key Identification](#361-headers-and-issuer-key-identification)
        - [3.6.2. Signature Generation](#362-signature-generation)
        - [3.6.3. Signature Verification](#363-signature-verification)
    - [3.7. Credential Response](#37-credential-response)
- [4. Credential Configuration](#4-credential-configuration)
    - [4.1. docType and Namespace](#41-doctype-and-namespace)
    - [4.2. `MdocCredentialConfiguration` object](#42-mdoccredentialconfiguration-object)
    - [4.3. Claim Definitions and Data Elements](#43-claim-definitions-and-data-elements)
- [5. Examples](#5-examples)
    - [5.1. `mDocPID` Credential Configuration](#51-mdocpid-credential-configuration)
    - [5.2. `IssuerSigned`](#52-issuersigned)
    - [5.3. Credential Response](#53-credential-response)

<!-- /TOC -->


<div style="page-break-after: always;"></div>

## 1. Overview

This document defines the data structure of the ISO mdoc Credential that OpenDID 3.0 issues via OID4VCI.
The Credential structure follows the `IssuerSigned` structure of `[ISO-18013-5]`, and the delivery between the Credential Issuer and the wallet follows the ISO mdoc Credential Format Profile of `[OID4VCI]`.
For the overall OID4VCI issuance procedure, refer to `[OID4VCI-ISSUANCE]`.

OpenDID issues mdoc in the `mso_mdoc-did` format.
In this format, IssuerAuth, the Issuer signature over the MSO (Mobile Security Object), is generated with the Issuer DID key, and the signing key is identified by the Issuer DID key URL in the COSE `kid` header.

The following must adhere to the definitions in this document.

- Composition of `IssuerSigned`, `IssuerSignedItem`, and the MSO
- IssuerAuth signature and Issuer key identification
- Location and format of the Holder binding key (Device Key) and the status reference (`status`)
- Mapping between the Credential Configuration `doctype` and the mdoc docType and namespace
- Encoding of the Credential response

The following are _out of the scope_ of this document and follow the relevant standards or implementations.

- Definition of data elements per document type (PID, mDL, etc. follow the rules of each document type)
- The `DeviceResponse` and `DeviceSigned` structures of the presentation phase (see `[ISO-18013-5]`, `[ISO-18013-7]`)
- Credential request and response encryption (see `[OID4VCI-ISSUANCE]`)


### 1.1. Reference Documents

| Reference          | Document Name                                                                           | Location                                                             |
| ------------------ | --------------------------------------------------------------------------------------- | -------------------------------------------------------------------- |
| [ISO-18013-5]      | ISO/IEC 18013-5 Personal identification — ISO-compliant driving licence — Part 5: Mobile driving licence (mDL) application |                                    |
| [ISO-18013-7]      | ISO/IEC TS 18013-7 Personal identification — ISO-compliant driving licence — Part 7: Mobile driving licence (mDL) add-on functions |                            |
| [OID4VCI]          | OpenID for Verifiable Credential Issuance 1.0 (Appendix A. Credential Format Profiles - ISO mdoc) | https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0.html |
| [RFC8949]          | Concise Binary Object Representation (CBOR)                                             | https://www.rfc-editor.org/rfc/rfc8949                               |
| [RFC8943]          | Concise Binary Object Representation (CBOR) Tags for Date                               | https://www.rfc-editor.org/rfc/rfc8943                               |
| [RFC9052]          | CBOR Object Signing and Encryption (COSE): Structures and Process                       | https://www.rfc-editor.org/rfc/rfc9052                               |
| [RFC9053]          | CBOR Object Signing and Encryption (COSE): Initial Algorithms                           | https://www.rfc-editor.org/rfc/rfc9053                               |
| [TOKEN-STATUS]     | Token Status List                                                                       | https://datatracker.ietf.org/doc/draft-ietf-oauth-status-list/       |
| [OSD]              | OpenDID Schema Definition Language                                                      | [OpenDID Schema Definition Language.md](../../guide/osd/OpenDID%20Schema%20Definition%20Language.md) |
| [OID4VCI-ISSUANCE] | (OpenDID) OID4VCI-based VC Issuance                                                     | [OID4VCI Issuance.md](../../concepts/OID4VCI%20Issuance.md)          |
| [DID-DOC-FORMAT]   | (OpenDID) DID Document format                                                           | [DID Document format.md](../did%20document%20format/DID%20Document%20format.md) |


<div style="page-break-after: always;"></div>

## 2. General Regulations

### 2.1. Credential Format

| Format         | Credential                     | Issuer Key Identification               | Holder Binding                                 | Status Reference          |
| -------------- | ------------------------------ | --------------------------------------- | ---------------------------------------------- | ------------------------- |
| `mso_mdoc-did` | ISO mdoc (`IssuerSigned`)      | Issuer DID key URL in the IssuerAuth `kid` header | MSO `deviceKeyInfo.deviceKey` (Holder public key from the Proof) | MSO `status.status_list` |

- `mso_mdoc-did` is an OID4VCI format identifier defined by OpenDID, and is used in the `format` of the Credential Configuration and in `credential_configurations_supported` of the Issuer Metadata.
- The Credential is the CBOR encoding of the `IssuerSigned` structure of `[ISO-18013-5]`.
- Since IssuerAuth is signed with the signing key of the Issuer DID Document, the verifier verifies the signature with the Issuer DID Document registered in the Trust Repository (see `[OID4VCI-ISSUANCE]` [4.6](../../concepts/OID4VCI%20Issuance.md#46-credential-formats)).
- Since mdoc is a format to which status management applies, the Proof of the Credential request must identify the key with a Holder DID URL `kid` (see `[OID4VCI-ISSUANCE]` [4.5](../../concepts/OID4VCI%20Issuance.md#45-credential-request-and-holder-binding)).


### 2.2. Notation

mdoc is CBOR (`[RFC8949]`) data, not JSON. This document defines data structures with `[OSD]`, applying the following additional rules.

- CBOR-specific types (`tstr`, `bstr`, `uint`, `tdate`, `fullDate`, etc.) are expressed with the user-defined types in [3.2](#32-data-types-and-constants).
- A member name that is a number (`"1"`, `"4"`, `"-1"`, etc.) denotes a CBOR integer label, not a string. This applies to COSE headers and COSE_Key.
- `#6.24(bstr .cbor X)` denotes the byte string of the CBOR encoding of `X`, wrapped with tag 24 (Encoded CBOR data item).

Examples are written in CBOR diagnostic notation.

| Notation             | Meaning                                               |
| -------------------- | ----------------------------------------------------- |
| `"text"`             | text string (tstr)                                    |
| `h'0a1b...'`         | byte string (bstr). Examples may show only part of the value. |
| `'text'`             | byte string (bstr) of the UTF-8 encoding of the text  |
| `24(<< ... >>)`      | bstr of the CBOR encoding of the inner data, with tag 24 |
| `0("...")`, `1004("...")` | tstr with tag 0 (date-time) or tag 1004 (full-date) |
| `/ ... /`            | comment                                               |


### 2.3. Encoding Rules

#### 2.3.1. CBOR Value Encoding

Data element values (`elementValue`) and MSO extension field values are encoded according to the type of the value passed by the Issuer, as follows.

| Value Type  | CBOR Encoding     | Remarks                                                    |
| ----------- | ----------------- | ---------------------------------------------------------- |
| String      | tstr              | Keys subject to the date rules get a tag ([2.3.2](#232-date-encoding)). |
| Integer     | int (uint / nint) |                                                            |
| Boolean     | bool              |                                                            |
| Byte array  | bstr              | e.g., `portrait` ([2.3.3](#233-portrait-image-portrait))   |
| Map         | map (tstr keys)   | The same rules are applied recursively to the nested values. |
| List        | array             | Each item is encoded with the same rules. Since the items themselves have no key, no date tag is added to them. |

#### 2.3.2. Date Encoding

A CBOR tag is added to the string value of the following keys, according to the key name.

| Key Name      | CBOR Encoding                   | Value Format             | Example                      |
| ------------- | ------------------------------- | ------------------------ | ---------------------------- |
| `birth_date`  | tag 1004 + tstr (full-date)     | `YYYY-MM-DD`             | `1004("1990-01-01")`         |
| `expiry_date` | tag 1004 + tstr (full-date)     | `YYYY-MM-DD`             | `1004("2036-10-21")`         |
| `issue_date`  | tag 0 + tstr (tdate)            | RFC 3339 date-time       | `0("2026-10-22T09:30:00Z")`  |

- The tag is added only when the key name matches exactly. Date data elements not listed in the table above (e.g., `issuance_date` of PID) are encoded as tstr without a tag.
- The same tag is added to a key with the same name inside a map value (e.g., `issue_date` and `expiry_date` of mDL `driving_privileges[]`).
- When the tag is added, the value format is neither converted nor checked. Therefore, the Issuer must pass a string in the format shown in the table above.
- The full-date tag (1004) is defined in `[RFC8943]`, and the date-time tag (0) in `[RFC8949]`.

The `validityInfo` timestamps of the MSO are always encoded as tdate, independently of the rules above (see [3.5.3](#353-validityinfo)).

#### 2.3.3. Portrait Image (portrait)

`portrait` (portrait image) encodes the JPEG image data as bstr.

- In the Webview issuance of the OpenDID Issuer server, a Claim whose `value_type` is `jpeg` is entered as a Base64 string, and its JPEG format (SOI/EOI markers) and size (up to 2 MB) are checked.
- At issuance, the Issuer server decodes the Base64 string of `portrait` and passes it as a byte array, so `elementValue` becomes bstr.
- A value passed as a string is encoded as tstr, so `portrait` must always be passed as a byte array.


<div style="page-break-after: always;"></div>

## 3. mdoc Credential Structure

### 3.1. Structure Overview

The mdoc Credential issued via OID4VCI is an `IssuerSigned` structure composed as follows.

```
IssuerSigned
├── nameSpaces
│   └── <namespace>                          e.g., "org.omnione.pid.1"
│       └── [ IssuerSignedItemBytes, ... ]   #6.24(bstr .cbor IssuerSignedItem)
│               └── { digestID, random, elementIdentifier, elementValue }
└── issuerAuth                               COSE_Sign1
    ├── protected     { 1 (alg): -7 (ES256) }
    ├── unprotected   { 4 (kid): Issuer DID key URL }
    ├── payload       MobileSecurityObjectBytes   #6.24(bstr .cbor MobileSecurityObject)
    │       └── { version, digestAlgorithm, valueDigests, deviceKeyInfo,
    │             docType, validityInfo, status }
    └── signature     ES256 signature (R || S, 64 bytes)
```

- Each data element is made into an `IssuerSignedItem`, and its digest is stored in `valueDigests` of the MSO.
- Since IssuerAuth signs the MSO, data elements are indirectly protected by the Issuer signature through their digests.
- A single Credential contains only one namespace (see [4.1](#41-doctype-and-namespace)).


### 3.2. Data Types and Constants

Items not defined here refer to `[ISO-18013-5]`.

```c#
//--------------------------------------------------------------------------------
// CBOR data types
//--------------------------------------------------------------------------------
def string tstr    : "CBOR text string (major type 3)"
def string bstr    : "CBOR byte string (major type 2)"
def int    uint    : "CBOR unsigned integer (major type 0)", min_value(0)
def string tdate   : "RFC 3339 date-time string with tag 0. ex: 0('2026-10-22T02:10:57Z')"
def string fullDate: "RFC 3339 full-date string with tag 1004. ex: 1004('1990-01-01')"
def string any     : "Any CBOR data item (encoding rules of section 2.3 apply)"
def string url     : "URL"

//--------------------------------------------------------------------------------
// mdoc data types
//--------------------------------------------------------------------------------
def string nameSpace            : "Namespace. ex: 'org.omnione.pid.1'"
def string dataElementIdentifier: "Data element identifier. ex: 'family_name'"
def int    digestID             : "Digest identifier of an IssuerSignedItem", min_value(0)
def string digest               : "SHA-256 digest (32 bytes bstr)"
def string didKeyUrl            : "DID key URL. ex: 'did:omn:issuer?versionId=1#assert'"

def string IssuerSignedItemBytes    : "#6.24(bstr .cbor IssuerSignedItem)"
def string MobileSecurityObjectBytes: "#6.24(bstr .cbor MobileSecurityObject)"

//--------------------------------------------------------------------------------
// Constants
//--------------------------------------------------------------------------------
def enum MDOC_FORMAT: "OID4VCI Credential format identifier"
{
    "mso_mdoc-did": "ISO mdoc whose IssuerAuth is signed with the Issuer DID key",
}

def enum DOC_TYPE: "mdoc document type (docType)"
{
    "org.omnione.pid.1"      : "OmniOne PID",
    "org.iso.18013.5.1.mDL"  : "Mobile driving licence (mDL)",
}

def enum NAME_SPACE: "Data element namespace"
{
    "org.omnione.pid.1"      : "OmniOne PID",
    "org.iso.18013.5.1"      : "Mobile driving licence (mDL)",
}

def enum MSO_VERSION: "MSO version"
{
    "1.0",
}

def enum DIGEST_ALGORITHM: "Digest algorithm"
{
    "SHA-256",
}

def enum COSE_ALG: "COSE algorithm identifier"
{
    -7: "ES256 (ECDSA, P-256, SHA-256)",
}

def enum COSE_KEY_TYPE: "COSE_Key key type (kty)"
{
    2: "EC2",
}

def enum COSE_EC_CURVE: "COSE_Key elliptic curve (crv)"
{
    1: "P-256",
}
```


### 3.3. `IssuerSignedItem` object

`IssuerSignedItem` contains one data element and the values required for the digest calculation.

```c#
def object IssuerSignedItem: "Data element signed by the Issuer"
{
    + digestID              "digestID"         : "Digest identifier"
    + bstr                  "random"           : "Random salt for the digest calculation", byte_length(16)
    + dataElementIdentifier "elementIdentifier": "Data element identifier"
    + any                   "elementValue"     : "Data element value"
}
```

- `digestID`: Integers starting from 0 and incremented by 1 are assigned in Claim order. It is unique within the namespace.
- `random`: 16-byte random value newly generated for each data element. It makes digests differ even for data elements with the same value, preventing value guessing.
- `elementIdentifier`: The last item of `credential_metadata.claims[].path` of the Credential Configuration (see [4.3](#43-claim-definitions-and-data-elements)).
- `elementValue`: Encoded according to the encoding rules in [2.3](#23-encoding-rules).

`IssuerSignedItem` is CBOR-encoded and wrapped with tag 24, and this `IssuerSignedItemBytes` form is used in `IssuerSigned` and in the digest calculation.


### 3.4. `IssuerSigned` object

`IssuerSigned` is the mdoc Credential that the Credential Issuer issues to the Holder.

```c#
def object IssuerSigned: "mdoc Credential issued via OID4VCI"
{
    + object "nameSpaces": "Data elements per namespace"
    {
        + array(IssuerSignedItemBytes) $nameSpace: "List of IssuerSignedItemBytes",
            variable_type(nameSpace), min_extend(1), max_extend(1)
    }
    + IssuerAuth "issuerAuth": "Issuer signature over the MSO (COSE_Sign1)"
}
```

- `nameSpaces`: Has the single namespace of the Credential as its key, and an array of `IssuerSignedItemBytes` as its value. The array order is the same as the `digestID` order.
- `issuerAuth`: See [3.6](#36-issuerauth-cose_sign1)


### 3.5. `MobileSecurityObject` object

The MSO (Mobile Security Object) contains the digests of the data elements, the Holder binding key, the validity period, and the status reference, and is signed as the payload of IssuerAuth.

```c#
def object MobileSecurityObject: "MSO (Mobile Security Object)"
{
    + MSO_VERSION      "version"        : "MSO version", value("1.0")
    + DIGEST_ALGORITHM "digestAlgorithm": "Digest algorithm", value("SHA-256")
    + object           "valueDigests"   : "Digests per namespace"
    {
        + object $nameSpace: "Namespace of IssuerSigned",
            variable_type(nameSpace), min_extend(1), max_extend(1)
        {
            + digest $digestID: "SHA-256 digest of IssuerSignedItemBytes",
                variable_type(digestID), min_extend(1)
        }
    }
    + object "deviceKeyInfo": "Holder binding key information"
    {
        + DeviceKey "deviceKey": "Holder public key (COSE_Key)"
    }
    + DOC_TYPE "docType"     : "Document type"
    + object   "validityInfo": "MSO validity period"
    {
        + tdate "signed"    : "MSO signing time"
        + tdate "validFrom" : "Start of the validity period"
        + tdate "validUntil": "End of the validity period"
    }

    //---------------------------------------------------------------------------
    // MSO extension fields (covered by the IssuerAuth signature)
    //---------------------------------------------------------------------------
    - object "status": "Credential status reference"
    {
        + object "status_list": "Token Status List reference"
        {
            + uint "idx": "Index of the Credential in the Status List"
            + url  "uri": "Status List URI"
        }
    }
}

def object DeviceKey: "Holder public key (COSE_Key, EC2 P-256)"
{
    + COSE_KEY_TYPE "1" : "kty", value(2)
    + COSE_EC_CURVE "-1": "crv", value(1)
    + bstr          "-2": "x coordinate", byte_length(32)
    + bstr          "-3": "y coordinate", byte_length(32)
}
```

- `docType`: Determined from the `doctype` of the Credential Configuration (see [4.1](#41-doctype-and-namespace)). The key name is `docType` in the MSO and `doctype` in the Credential Configuration.
- `validityInfo.expectedUpdate`, `deviceKeyInfo.keyAuthorizations`, and `deviceKeyInfo.keyInfo` defined in `[ISO-18013-5]` are not included.

#### 3.5.1. `valueDigests`

`valueDigests` has the form `{namespace: {digestID: digest}}`, and the digest is calculated as follows.

```
valueDigests[nameSpace][digestID] = SHA-256( IssuerSignedItemBytes )
```

- The digest is calculated over the CBOR encoding of the entire `IssuerSignedItemBytes`, including tag 24.
- There is exactly one digest for each `IssuerSignedItemBytes` in `IssuerSigned.nameSpaces`.

#### 3.5.2. `deviceKeyInfo`

`deviceKeyInfo.deviceKey` is the Holder binding key, which is the Holder public key confirmed by the Proof of the Credential request.

- The wallet sets the Holder DID URL in the `kid` of the Proof JWT. The Credential Issuer retrieves the Holder DID Document, verifies the Proof signature with the P-256 public key registered in `authentication`, and uses this public key as `deviceKey`.
- `deviceKey` is encoded as a COSE_Key (`[RFC9052]`, `[RFC9053]`) and contains only `kty` (EC2), `crv` (P-256), `x`, and `y`.
- If a single Credential request contains multiple Proofs, one mdoc is issued per Proof key, and the `deviceKey` of each mdoc is the public key of the corresponding Proof.
- In the presentation phase, the verifier confirms the Holder binding by verifying the DeviceAuth signature with `deviceKey` (out of the scope of this document).

#### 3.5.3. `validityInfo`

| Item         | Value                                          |
| ------------ | ---------------------------------------------- |
| `signed`     | MSO creation time (UTC, truncated to seconds)  |
| `validFrom`  | Same value as `signed`                         |
| `validUntil` | `signed` + 365 days (31,536,000 seconds)       |

- All three values are encoded as tdate without fractional seconds (`0("YYYY-MM-DDThh:mm:ssZ")`).
- The MSO validity period is fixed relative to the issuance time and cannot be changed by the Credential Configuration.
- The MSO validity period is independent of the `expiry_date` data element (document expiry date).

#### 3.5.4. `status`

`status` is a Token Status List reference for checking the status of the Credential, and follows the `status_list` format of `[TOKEN-STATUS]`.

| Item                 | Type | Description                                                 |
| -------------------- | ---- | ----------------------------------------------------------- |
| `status.status_list.uri` | tstr | Status List URI. `{Issuer base URL}/status-lists/{listId}` |
| `status.status_list.idx` | uint | Index assigned to the Credential in the Status List        |

- The Credential Issuer assigns a Status List index at Credential issuance and adds `status` to the MSO as an extension field. Therefore, `status` is covered by the IssuerAuth signature.
- The OpenDID Issuer server always includes `status` in `mso_mdoc-did` Credentials.
- For status values, status transitions, and Status List retrieval, refer to [Credential Status format](Credential%20Status%20format.md) and `[OID4VCI-ISSUANCE]` [Chapter 5](../../concepts/OID4VCI%20Issuance.md#5-credential-status-management).

#### 3.5.5. MSO Extension Fields

The Credential Issuer can add fields other than the standard MSO fields to the MSO, and the added fields are covered by the IssuerAuth signature.

- The name of an extension field cannot be empty.
- The following standard MSO field names are reserved and cannot be used as extension fields. Using them causes the issuance to fail.
    - `version`, `digestAlgorithm`, `valueDigests`, `deviceKeyInfo`, `docType`, `validityInfo`
- Extension field values follow the encoding rules in [2.3](#23-encoding-rules).
- The only extension field used in OpenDID issuance is `status`.


### 3.6. `IssuerAuth` (COSE_Sign1)

IssuerAuth is the Issuer signature over the MSO, with the COSE_Sign1 structure of `[RFC9052]`.
It is encoded as an array of four items without tag 18 (the COSE_Sign1 tag).

```c#
def array(any) IssuerAuth: "COSE_Sign1 [protected, unprotected, payload, signature]", count(4)

def object IssuerAuthProtectedHeader: "protected header (encoded as bstr .cbor)"
{
    + COSE_ALG "1": "alg", value(-7)
}

def object IssuerAuthUnprotectedHeader: "unprotected header"
{
    + bstr "4": "kid. UTF-8 encoding of the Issuer DID key URL (didKeyUrl)"
}
```

| index | Item          | Type                                          | Value                                     |
| :---: | ------------- | --------------------------------------------- | ----------------------------------------- |
| 0     | `protected`   | bstr .cbor `IssuerAuthProtectedHeader`        | `<< {1: -7} >>`                           |
| 1     | `unprotected` | `IssuerAuthUnprotectedHeader`                 | `{4: 'did:omn:issuer?versionId=1#assert'}` |
| 2     | `payload`     | bstr (encoding of `MobileSecurityObjectBytes`) | `<< 24(<< MSO >>) >>`                    |
| 3     | `signature`   | bstr (64 bytes)                               | ES256 signature (64 bytes of R and S concatenated) |

#### 3.6.1. Headers and Issuer Key Identification

- `protected` contains only `alg` (1), whose value is `-7` (ES256).
- `unprotected` contains only `kid` (4). `kid` is a bstr of the UTF-8 encoding of the Issuer DID key URL.
    - Format: `{Issuer DID}?versionId={DID Document version}#{key id}` (e.g., `did:omn:issuer?versionId=1#assert`)
    - The key id is the signing key (`assert`) of the Issuer DID Document, registered in `assertionMethod` (see `[DID-DOC-FORMAT]` [3.4](../did%20document%20format/DID%20Document%20format.md#34-provider-did-document)).
- The MSO has no Issuer identifier, and the Issuer is identified only by the DID of `kid`.

#### 3.6.2. Signature Generation

The Credential Issuer generates IssuerAuth in the following order.

1. Create `IssuerSignedItemBytes` for each Claim and calculate its digest.
1. Compose the MSO, add the extension field (`status`), and encode it as `MobileSecurityObjectBytes` (`#6.24(bstr .cbor MobileSecurityObject)`).
1. Compose the `protected` header (`{1: -7}`) and the `unprotected` header (`{4: kid}`).
1. Compose the Sig_structure to be signed (`[RFC9052]` Section 4.4).
    ```
    Sig_structure = [ "Signature1", protected, h'', payload ]
    ```
    - `external_aad` is an empty bstr (`h''`).
    - `payload` is the encoding of `MobileSecurityObjectBytes`.
1. Hash the CBOR encoding of the Sig_structure with SHA-256, and sign it with the signing key of the Issuer wallet (the key id of `kid`).
1. Normalize the signature value to the 64-byte `R || S` format that can be verified with the public key of the Issuer DID Document.
1. Set the array `[protected, unprotected, payload, signature]` as `issuerAuth`.

#### 3.6.3. Signature Verification

The verifier verifies `IssuerSigned` as follows.

1. Check that `alg` in the `protected` header of `issuerAuth` is `-7` (ES256).
1. Decode `kid` of the `unprotected` header as UTF-8 to obtain the Issuer DID key URL.
1. Retrieve the Issuer DID Document of the given version (`versionId`) from the Trust Repository, and verify the signature over the Sig_structure with the public key whose id equals the key id.
1. Since `kid` is not covered by the signature, separately check that the DID of `kid` is the DID of a trusted Issuer.
1. Remove tag 24 from `payload` to obtain the MSO, and check `docType` and `validityInfo` (whether the current time is within `validFrom`–`validUntil`).
1. For each `IssuerSignedItemBytes` in `nameSpaces`, calculate the SHA-256 digest and compare it with `valueDigests[nameSpace][digestID]`.
1. Check the Credential status with `uri` and `idx` of `status.status_list` (see `[OID4VCI-ISSUANCE]` [5.5](../../concepts/OID4VCI%20Issuance.md#55-status-check-by-verifier)).

> **Note**: This section describes the verification items based on the data structure of this document. For DeviceAuth verification in the presentation phase and the implementation of the OpenDID verifier server, refer to separate documents.


### 3.7. Credential Response

The Credential Endpoint encodes `IssuerSigned` as CBOR, encodes it with base64url without padding, and returns the string in `credentials[].credential`.

```c#
def object CredentialResponse: "Credential response"
{
    + array(object) "credentials": "List of issued Credentials", min_count(1)
    {
        + string "credential": "base64url(CBOR encoding of IssuerSigned), without padding"
    }
}
```

- If a single request contains multiple Proofs, one Credential is issued per Proof key and included in the `credentials` array.
- If `credential_response_encryption.encryption_required` of the Issuer Metadata is `true`, the entire response is encrypted as JWE (see `[OID4VCI-ISSUANCE]` [4.5](../../concepts/OID4VCI%20Issuance.md#45-credential-request-and-holder-binding)).
- `notification_id` is not included.


<div style="page-break-after: always;"></div>

## 4. Credential Configuration

An mdoc Credential is issued according to the `doctype` and Claim definitions of the Credential Configuration.
The Metadata of the Credential Configuration is published in `credential_configurations_supported` of the Issuer Metadata, keyed by the Configuration ID (see `[OID4VCI-ISSUANCE]` [3.1](../../concepts/OID4VCI%20Issuance.md#31-credential-configuration)).

### 4.1. docType and Namespace

The Credential Issuer reads `doctype` from the Metadata of the Credential Configuration identified in the Credential request, and determines the MSO `docType` and the data element namespace.
An mDL (`org.iso.18013.5.1.mDL`) uses the namespace `org.iso.18013.5.1`, and any other `doctype` is used as-is for both the docType and the namespace.

| Credential Configuration `doctype` | MSO `docType`             | Namespace                 | Document Type |
| ---------------------------------- | ------------------------- | ------------------------- | ------------- |
| `org.omnione.pid.1`                | `org.omnione.pid.1`       | `org.omnione.pid.1`       | OmniOne PID   |
| `org.iso.18013.5.1.mDL`            | `org.iso.18013.5.1.mDL`   | `org.iso.18013.5.1`       | mDL           |

- All data elements are included in the single determined namespace.
- If `doctype` is missing from the Metadata or is empty, the issuance fails.

> **Note**: Using a `doctype` such as `org.omnione.pid.1` as-is for both the docType and the namespace requires an mdoc SDK version that supports custom document types. With a version that does not support them, issuance with a `doctype` other than the document types built into the SDK fails. The namespace in `credential_metadata.claims[].path` must be the same value as `doctype`.

### 4.2. `MdocCredentialConfiguration` object

The mdoc entry of `credential_configurations_supported` is composed as follows.

```c#
def object MdocCredentialConfiguration: "mdoc entry of credential_configurations_supported"
{
    + MDOC_FORMAT   "format" : "Credential format", value("mso_mdoc-did")
    + DOC_TYPE      "doctype": "Document type (section 4.1)"
    - string        "scope"  : "OAuth 2.0 scope used in the Authorization Code flow"
    - array(string) "cryptographic_binding_methods_supported": "Holder binding methods"
    - array(int)    "credential_signing_alg_values_supported": "IssuerAuth signing algorithms (COSE algorithm identifiers)"
    - object        "proof_types_supported": "Supported Proof types"
    {
        + object "jwt": "JWT Proof"
        {
            + array(string) "proof_signing_alg_values_supported": "Proof signing algorithms"
        }
    }
    - object "credential_metadata": "Credential display information and Claim definitions"
    {
        - array(object) "display": "Credential display information"
        {
            + string "name"  : "Credential name"
            - string "locale": "Language tag. ex: 'en'"
            - object "logo"  : "Logo"
            {
                + url    "uri"     : "Logo image URI"
                - string "alt_text": "Alternative text"
            }
        }
        - array(MdocClaimDescription) "claims": "List of Claim definitions"
    }
}

def object MdocClaimDescription: "mdoc Claim definition"
{
    + array(string) "path"     : "[namespace, data element identifier]", count(2)
    - bool          "mandatory": "Whether the Claim is mandatory", default(false)
    - array(object) "display"  : "Claim display information"
    {
        - string "name"  : "Claim name"
        - string "locale": "Language tag"
    }
    - string        "value_type": "(OpenDID extension) Webview input value type (section 4.3)", default("string")
    - array(string) "enum"      : "(OpenDID extension) Allowed Webview input values"
}
```

| Item | OpenDID Rule |
| ---- | ------------ |
| `format` | `mso_mdoc-did` |
| `doctype` | A value listed in the table of [4.1](#41-doctype-and-namespace) |
| `cryptographic_binding_methods_supported` | Since the Holder binding key is included in the MSO as a COSE_Key, `cose_key` is included. |
| `credential_signing_alg_values_supported` | Since IssuerAuth is signed with `-7` (ES256), `[-7]` is published. |
| `proof_types_supported` | Only `jwt` Proof signatures are verified, and the Proof signing algorithm is `ES256`. |
| `credential_metadata.claims[].path` | `[namespace, data element identifier]`. The namespace must be the same as the value determined in [4.1](#41-doctype-and-namespace). |
| `credential_metadata.claims[].mandatory` | Used as the mandatory input flag in Webview issuance. |

### 4.3. Claim Definitions and Data Elements

- The Credential Issuer uses the last item of `credential_metadata.claims[].path` as the data element identifier (`elementIdentifier`).
- The issued data elements consist of the Claim values that the Credential Issuer holds for the user. Optional Claims without a value are not included in the mdoc.
- In the Webview issuance (user-initiated issuance) of the OpenDID Issuer server, input values are checked and converted according to `value_type`.

| `value_type`              | Input Value Handling                                | `elementValue` Encoding                             |
| ------------------------- | --------------------------------------------------- | --------------------------------------------------- |
| `string` (default)        | Used as-is (checked against `enum` if present)      | tstr                                                |
| `full-date`, `date`       | `YYYY-MM-DD` format check                           | tstr (tag 1004 for `birth_date` and `expiry_date`)  |
| `uint`, `integer`         | Converted to an integer (`uint` must be 0 or greater) | int                                               |
| `boolean`                 | `true` or `false`                                   | bool                                                |
| `list`                    | Converted to a JSON array                           | array                                               |
| `jpeg`                    | Base64 JPEG check (up to 2 MB)                      | bstr (`portrait`, [2.3.3](#233-portrait-image-portrait)) |
| Others (e.g., `places`, `nationalities`) | Used as-is (checked against `enum` if present) | tstr                                      |


<div style="page-break-after: always;"></div>

## 5. Examples

### 5.1. `mDocPID` Credential Configuration

This is the `mDocPID` Credential Configuration of the Issuer server sample Metadata. Only some of the `claims` are shown.

```json
{
  "credential_configurations_supported": {
    "mDocPID": {
      "format": "mso_mdoc-did",
      "doctype": "org.omnione.pid.1",
      "scope": "org.omnione.pid_mdoc",
      "cryptographic_binding_methods_supported": ["jwk", "cose_key"],
      "credential_signing_alg_values_supported": [-7],
      "proof_types_supported": {
        "jwt": {
          "proof_signing_alg_values_supported": ["ES256"]
        }
      },
      "credential_metadata": {
        "display": [
          {
            "name": "PID (MSO Mdoc)",
            "locale": "en",
            "logo": {
              "uri": "https://examplestate.com/public/pid.png",
              "alt_text": "A square figure of a PID"
            }
          }
        ],
        "claims": [
          {
            "path": ["org.omnione.pid.1", "family_name"],
            "display": [{ "locale": "en", "name": "Family Name(s)" }],
            "mandatory": true,
            "value_type": "string"
          },
          {
            "path": ["org.omnione.pid.1", "given_name"],
            "display": [{ "locale": "en", "name": "Given Name(s)" }],
            "mandatory": true,
            "value_type": "string"
          },
          {
            "path": ["org.omnione.pid.1", "birth_date"],
            "display": [{ "locale": "en", "name": "Birth Date" }],
            "mandatory": true,
            "value_type": "full-date"
          },
          {
            "path": ["org.omnione.pid.1", "nationality"],
            "display": [{ "locale": "en", "name": "Nationality" }],
            "mandatory": true,
            "value_type": "nationalities"
          },
          {
            "path": ["org.omnione.pid.1", "issuance_date"],
            "display": [{ "locale": "en", "name": "Issuance Date" }],
            "mandatory": true
          },
          {
            "path": ["org.omnione.pid.1", "expiry_date"],
            "display": [{ "locale": "en", "name": "Expiry Date" }],
            "mandatory": true
          },
          {
            "path": ["org.omnione.pid.1", "portrait"],
            "display": [{ "locale": "en", "name": "Portrait Image" }],
            "mandatory": false,
            "value_type": "jpeg"
          }
        ]
      }
    }
  }
}
```

- Since `doctype` is `org.omnione.pid.1`, both the MSO `docType` and the namespace are `org.omnione.pid.1`.
- The sample Metadata also has a `policy` (`batch_size`, `one_time_use`) item, but it is not used in OpenDID issuance (see `[OID4VCI-ISSUANCE]` [Chapter 6](../../concepts/OID4VCI%20Issuance.md#6-implementation-scope-and-constraints)).

### 5.2. `IssuerSigned`

This is the CBOR diagnostic notation of an `IssuerSigned` issued with `mDocPID`. Only some of the data elements and bstr values are shown.

```
{
  "nameSpaces": {
    "org.omnione.pid.1": [
      24(<< {
        "digestID": 0,
        "random": h'0aaa3ab0b484e0bb79b43d74c63fa8de',
        "elementIdentifier": "family_name",
        "elementValue": "Kim"
      } >>),
      24(<< {
        "digestID": 1,
        "random": h'457e9555f42935983a9f5b2e1254745b',
        "elementIdentifier": "given_name",
        "elementValue": "MDOC"
      } >>),
      24(<< {
        "digestID": 2,
        "random": h'51715cd36b23bd5d08b7158d871a8c5a',
        "elementIdentifier": "birth_date",
        "elementValue": 1004("1990-01-01")      / full-date /
      } >>),
      24(<< {
        "digestID": 3,
        "random": h'ef61d6e3f8ae17197f34f8b15aece972',
        "elementIdentifier": "nationality",
        "elementValue": "KR"
      } >>),
      24(<< {
        "digestID": 4,
        "random": h'88e724abaeef4c5c7e22aeeb0c0b8b30',
        "elementIdentifier": "issuance_date",
        "elementValue": "2026-10-22"            / no tag /
      } >>),
      24(<< {
        "digestID": 5,
        "random": h'9b65443a01880ef9831014d9f08131b6',
        "elementIdentifier": "expiry_date",
        "elementValue": 1004("2036-10-21")      / full-date /
      } >>),
      24(<< {
        "digestID": 6,
        "random": h'337c401127134137ebd03235f731993f',
        "elementIdentifier": "portrait",
        "elementValue": h'ffd8ffe0...ffd9'      / JPEG bytes /
      } >>)
    ]
  },
  "issuerAuth": [
    << { 1: -7 } >>,                                / protected: alg = ES256 /
    { 4: 'did:omn:issuer?versionId=1#assert' },    / unprotected: kid /
    << 24(<< {                                      / payload: MobileSecurityObjectBytes /
      "version": "1.0",
      "digestAlgorithm": "SHA-256",
      "valueDigests": {
        "org.omnione.pid.1": {
          0: h'7a1ad808...8356faa7',
          1: h'085c08b1...88e981db',
          2: h'18d066b1...e06f9e1c',
          3: h'3cac0005...291728d7',
          4: h'ef077760...a3d7c4be',
          5: h'4f2edc79...7c4c44c3',
          6: h'0b674036...a4001e16'
        }
      },
      "deviceKeyInfo": {
        "deviceKey": {
          1: 2,                                     / kty: EC2 /
          -1: 1,                                    / crv: P-256 /
          -2: h'a5b397d1...7f64896d',               / x /
          -3: h'10ba5609...7154359d'                / y /
        }
      },
      "docType": "org.omnione.pid.1",
      "validityInfo": {
        "signed": 0("2026-10-22T02:10:57Z"),
        "validFrom": 0("2026-10-22T02:10:57Z"),
        "validUntil": 0("2027-10-22T02:10:57Z")
      },
      "status": {
        "status_list": {
          "idx": 42,
          "uri": "https://issuer.example.com/status-lists/1"
        }
      }
    } >>) >>,
    h'5f1c...9a3e'                                  / signature: R || S (64 bytes) /
  ]
}
```

- `issuance_date` is not subject to the date tag rules, so it is encoded as tstr (see [2.3.2](#232-date-encoding)).
- `validUntil` is 365 days after `signed`.
- The verifier verifies the signature with the `assert` key of the `did:omn:issuer` DID Document (`versionId=1`) given by `kid`.

### 5.3. Credential Response

```json
{
  "credentials": [
    {
      "credential": "omppc3N1ZXJBdXRohEOhASahBFghZGlkOm9tbjppc3N1ZXI_dmVyc2lvbklkPTEjYXNzZXJ0..."
    }
  ]
}
```

- `credential` is the value of the `IssuerSigned` in [5.2](#52-issuersigned), encoded as CBOR and then as base64url without padding.
