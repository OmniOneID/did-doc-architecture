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
Profile format
==

- Subject
    - Profile Structure Definition
- Author: OpenSource Development Team
- Date: 2025-05-09
- Version: v2.0.0

Revision History
---

| Version | Date       | Changes         |
| ------- | ---------- | --------------- |
| v1.0.0  | 2024-09-03 | Initial version |
| v2.0.0 | 2025-05-09 | Add ZKP data     |


<div style="page-break-after: always;"></div>

Table of Contents
---

<!-- TOC tocDepth:2..4 chapterDepth:2..6 -->

- [1. Overview](#1-overview)
  - [1.1. Purpose](#11-purpose)
- [2. General Matters](#2-general-matters)
  - [2.1. Data Types and Constants](#21-data-types-and-constants)
- [3. Issue Profile](#3-issue-profile)
  - [3.1. Issue Profile Structure](#31-issue-profile-structure)
    - [3.1.1. `IssueProfile` object](#311-issueprofile-object)
  - [3.2. Issue Profile Example](#32-issue-profile-example)
- [4. Verify Profile](#4-verify-profile)
  - [4.1. Verify Profile Structure](#41-verify-profile-structure)
    - [4.1.1. `VerifyProfile` object](#411-verifyprofile-object)
  - [4.2. Verify Profile Example](#42-verify-profile-example)
- [5. ProofRequest Profile](#5-proofrequest-profile)
    - [5.1. ProofRequest Profile Structure](#51-proofrequest-profile-structure)
        - [5.1.1. `ProofRequestProfile` object](#511-proofrequestprofile-object)
    - [5.2. ProofRequest Profile Example](#52-proofrequest-profile-example)    

<!-- /TOC -->

<div style="page-break-after: always;"></div>

## 1. Overview

The Profile is the request information provided to the Holder when the Issuer issues a VC or the Verifier receives a VP presentation. There are two types as follows:

- Issue Profile: Issuance Request Information
- Verify Profile: Verification Request Information

How the Profile is transmitted to the Holder's device is beyond the scope of this document, but examples include:

- **Example 1**: Encoding the Profile into a QR code and displaying it on a screen
- **Example 2**: Serializing the Profile into other forms of data and including it in a PUSH request message to send a PUSH message
- **Example 3**: Transmitting a downloadable URL for the Profile via various interfaces such as email, NFC, BLE, etc.

### 1.1. Purpose

The purpose of this document is to define the structure of various Profiles and explain how to use them. As mentioned earlier, the methods of transmitting Profiles between entities are not covered in this document and should be implemented separately according to the requirements of each implementation.

## 2. General Matters

### 2.1. Data Types and Constants

For items not defined here, refer to `[DATA-SPEC]`.

```c#
def enum PROOF_TYPE: "proof type"
{
    "RsaSignature2018",
    "Secp256k1Signature2018",
    "Secp256r1Signature2018",
}

def enum PROOF_PURPOSE: "proof purpose"
{
    "assertionMethod",
    "authentication",
    "keyAgreement",
    "capabilityInvocation",
    "capabilityDelegation",
}

def enum CREDENTIAL_SCHEMA_TYPE: "credential schema type"
{
    "OsdSchemaCredential": "OSD VC Schema",
}

def enum PROFILE_TYPE: "profile type"
{
    "IssueProfile" : "Issuance Request Information",
    "VerifyProfile": "Verification Request Information",
}

def enum ECC_CURVE_TYPE: "ECC curve type"
{
    "Secp256k1",
    "Secp256r1",
}

def enum SYMMETRIC_CIPHER_TYPE: "symmetric cipher type"
{
    "AES-128-CBC",
    "AES-128-ECB",
    "AES-256-CBC",
    "AES-256-ECB",
}

def enum SYMMETRIC_PADDING_TYPE: "symmetric padding type"
{
    "NOPAD",
    "PKCS5",
}

def enum VERIFY_AUTH_TYPE: "authentication type for presentation"
{
    0x00000000: "No restrictions on authentication methods",
    0x00000001: "No Authentication",
    0x00000002: "PIN",
    0x00000004: "BIO",
    0x00000006: "PIN or BIO",
    0x00008006: "PIN and BIO",
}
```

<div style="page-break-after: always;"></div>

## 3. Issue Profile

The Issue Profile is a data bundle that the Issuer uses to provide the Holder with information about the issuance procedure and the VC to be issued. Specifically, it includes the following information:

| Category       | Details                                                                                      |
| -------------- | -------------------------------------------------------------------------------------------- |
| Issuer Info    | • Name, DID, Reference URL                                                                   |
| VC Schema      | • VC Schema URL or directly includes the VC Schema<br>(Includes directly only if there is no limit on Profile size) |
| Issuance Method | • Issuance request API URL<br>• Information related to request encryption (recipient public key, nonce, algorithm, etc.) |


### 3.1. Issue Profile Structure

#### 3.1.1. `IssueProfile` object

```c#
def object IssueProfile: "Issue Profile"
{
    //--------------------------------------------------------------------------
    // Profile Metadata
    //--------------------------------------------------------------------------
    + uuid         "id"         : "profile id"
    + PROFILE_TYPE "type"       : "profile type", value("IssueProfile")
    + string       "title"      : "profile title"
    - string       "description": "profile description", default(""), emptiable(true)
    - LogoImage    "logo"       : "logo image for issuance"
    + ENCODING     "encoding"   : "encoding", default("UTF-8")
    + LANGUAGE     "language"   : "language code"

    //--------------------------------------------------------------------------
    // Profile Contents
    //--------------------------------------------------------------------------
    + object "profile": "profile contents"
    {
        + ProviderDetail "issuer": "issuer information"

        + object "credentialSchema": "VC Schema information"
        {
            + url                    "id"   : "VC Schema URL"
            + CREDENTIAL_SCHEMA_TYPE "type" : "VC Schema format type"
            - multibase              "value": "VC Schema encoded in multibase"
        }

        - object "credentialOffer": "Credential Offer information"
        {    
            + nonce "nonce"                      : "nonce"
            + schema-identifier "schemaId"       : "CredentialSchema identifier"
            + definition-identifier "credDefId"  : "CredentialDefinition identifier"
            + object "keyCorrectnessProof"       : "KeyCorrectnessProof"
            {
                + string "c"           : "hash"
                + string "xzCap"       : "xzCap"
                + object "xrCap"       : "xrCap", min_extend(1)
                {
                    /* 
                    ... Example (order must be preserved)
                    - string "zkpsex"
                    - string "zkpasort"
                    - string "zkpaddr"
                    - string "zkpbirth"
                    .
                    .
                    */
                    + string $attributeName : "Commitment value for each attribute", min_extend(1)
                    + string "masterSecret" : "Commitment value for the user's master secret" 
                }
            }
        }
        + object "process": "issuance method"
        {
            + array(url) "endpoints"  : "list of issuance API endpoints"
            + ReqE2e     "reqE2e"     : "E2E request information (without proof)"
            + multibase  "issuerNonce": "issuer nonce", byte_length(16)
        }
    }

    //--------------------------------------------------------------------------
    // Proof
    //--------------------------------------------------------------------------
    + AssertProof "proof": "issuer's signature for the profile"
}
```
- `~/profile/credentialSchema`: VC Schema Information
    - `id`: URL to download the VC Schema
    - `value`: VC Schema encoded in multibase (Base64 recommended)
        - Used when a client cannot download the VC Schema
        - Be cautious of Profile size as it may become large depending on the Profile transmission method (e.g., QR code)
- `~/profile/credentialOffer`: Created by the issuer before issuing a Credential to the user  
  (See ZKP Data Specification #4.2: CredentialOffer)
- `~/profile/process`
    - `endpoints`: Service endpoint part excluding the path portion of the issuance API
    - `reqE2e`: Key exchange request information for E2E encryption
    - `issuerNonce`: Must include `issuerNonce` when signing the issuance request information in the future


### 3.2. Issue Profile Example

```json
{
    "id": "24a60adc-19f0-45c3-934b-fc9f350c7d4c",
    "type": "IssueProfile",
    "title": "Woosan University Student ID Issuance",
    "description": "Issuance of Woosan University Student ID.",
    "encoding": "UTF-8",
    "language": "ko",
    "profile": {
        "issuer": {
            "did": "did:example:woosanuniv",
            "certVcRef": "https://woosan.ac.kr/cert-vc/0528",
            "name": "Woosan University",
            "ref": "https://woosan.ac.kr"
        },
        "credentialSchema": {
            "id": "https://woosan.ac.kr/schema/student_id_v2.json",
            "type": "OsdSchemaCredential"
        },
        // Additional depending on whether ZKP is included
        "credentialOffer":{
            "nonce": "1068995366822249097155600",
            "schemaId": "did:example:woosanuniv:2:student_id:1.0",
            "credDefId": "did:example:woosanuniv:3:CL:did:example:woosanuniv:2:student_id:1.0:Tag1",
            "keyCorrectnessProof": {
               "c": "61980984485776724933402455877937134233912022625919280439072090002286297246205",
               "xzCap": "108...850",
               "xrCap": {
                  "zkpname": "453...106",
                  "zkpbirth": "187...442",
                  "masterSecret": "211...206"
               }
            }

        "process": {
            "endpoints": ["https://woosan.ac.kr/issue"],
            "reqE2e": {
                "nonce": "uVGhlIHF1aWNrIGJy",
                "curve": "Secp256r1",
                "publicKey": "zgg7kPM4Fv6id6YaLJ2v5uEdX4Y73kZBvuJCxieAAVQna",
                "cipher": "AES-256-CBC",
                "padding": "PKCS5"
            },
            "issuerNonce": "ucmthamw7ZmtkamFhZGZlZg"
        }
    },
    "proof": {
        "type": "Secp256r1Signature2018",
        "created": "2024-04-29T11:27:30Z",
        "verificationMethod": "did:example:woosanuniv?versionId=1#assert",
        "proofPurpose": "assertionMethod",
        "proofValue": "zDgYdYMUYHURJLD7xdnWRiqWCEY5u5fKzZs6Z...MzLHoPiPQ9sSVfRrs1D"
    }
}
```

<div style="page-break-after: always;"></div>

## 4. Verify Profile

The Verify Profile is a data bundle that the Verifier uses to provide the Holder with information about the VP presentation method and the VC to be presented. Specifically, it includes the following information:

| Category    | Details                                                                                                          |
| ----------- | ---------------------------------------------------------------------------------------------------------------- |
| Verify Info | • Name, DID, Reference URL                                                                                       |
| Filter      | • Submission of All Claims<br>• List of eligible VC Schemas and Issuers<br>• List of claims to be displayed on the user’s screen<br>• List of claims that must be submitted |
| VP Submission Method | • Submission API URL<br>• Information related to submission encryption (recipient public key, nonce, algorithm, etc.)<br>• Authentication type for presentation |

Determining whether multiple specified VC Schemas are treated as AND or OR conditions is beyond the scope of this document.
It should be handled in accordance with predefined methods between the Verifier and the Profile recipient.

### 4.1. Verify Profile Structure

#### 4.1.1. `VerifyProfile` object

```c#
def object VerifyProfile: "Verify Profile"
{
    //--------------------------------------------------------------------------
    // Profile Metadata
    //--------------------------------------------------------------------------
    + uuid         "id"         : "profile id"
    + PROFILE_TYPE "type"       : "profile type", value("VerifyProfile")
    + string       "title"      : "profile title"
    - string       "description": "profile description", default(""), emptiable(true)
    - LogoImage    "logo"       : "logo image for presentation"
    + ENCODING     "encoding"   : "encoding", default("UTF-8")
    + LANGUAGE     "language"   : "language code"

    //--------------------------------------------------------------------------
    // Profile Contents
    //--------------------------------------------------------------------------
    + object "profile": "profile contents"
    {
        + ProviderDetail "verifier": "verifier information"

        + object "filter": "filter for selecting VCs to be presented"
        {
            + array(object) "credentialSchemas": "claims and issuers per acceptable VC Schema"
            {
                // VC Schema information
                + url                    "id"        : "VC Schema URL"
                + CREDENTIAL_SCHEMA_TYPE "type"      : "VC Schema format type"
                - multibase              "value"     : "The value of the VC Schema encoded in multibase"
                // Claim constraints
                - bool             "presentAll"    : "Whether all claims are submitted", default(false)
                - array(claimCode) "displayClaims" : "List of claims displayed on the screen", emptiable(false)
                - array(claimCode) "requiredClaims": "List of mandatory claims to be submitted", emptiable(false)
                - array(did)       "allowedIssuers": "List of allowed issuer DIDs", emptiable(false)
            }
        }

        + object "process": "VP presentation method"
        {
            - array(url)       "endpoints"    : "list of submission API endpoints"
            + ReqE2e           "reqE2e"       : "E2E request information (without proof)"
            + multibase        "verifierNonce": "verifier nonce", byte_length(16)
            - VERIFY_AUTH_TYPE "authType"     : "authentication type for presentation"
        }
    }

    //--------------------------------------------------------------------------
    // Proof
    //--------------------------------------------------------------------------
    + AssertProof "proof": "verifier's signature for the profile"
}
```
- `~/profile/filter/credentialSchemas[]`
    - `presentAll`: Whether all claims are submitted
        - true: Submit all claims
            - `displayClaims` value should be ignored, and all claims must be displayed on the screen
            - `requiredClaims` value should be ignored, and all claims must be submitted as mandatory
        - false: Partial submission is allowed
    - `displayClaims`
        - The list of claims to be displayed to the user, regardless of whether they are optional or required
        - Items included in `requiredClaims` should not be allowed to be deselected
        - `displayClaims`에 없으나 `requiredClaims`에는 있는 항목의 제출 여부는 TAS의 정책으로 결정
        - If unspecified, all claims should be displayed on the screen
    - `requiredClaims`
        - The list of claims that must be submitted
        - If not specified, there are no mandatory claims to be submitted
    - `allowedIssuers`
        - List of allowed Issuer DIDs
        - The VC held by the Holder can only be submitted if it is included in the Issuer list
        - If not specified, any VC issued under the specified VC Schema can be submitted
- `~/profile/process`
    - `endpoints`: Service endpoint part excluding the path portion of the submission API
    - `reqE2e`: Key exchange request information for E2E encryption
    - `verifierNonce`: Must include `verifierNonce` when signing the VP in the future
    - `authType`: Conditions for user authentication when submitting VP

### 4.2. Verify Profile Example

```json
{
    "id": "d1f26925-6743-4609-9932-e909dda0299f",
    "type": "VerifyProfile",
    "title": "MyShopping Student ID Verification",
    "description": "Discounts are not available if personal identifier submission is not agreed.",
    "encoding": "UTF-8",
    "language": "ko",
    "profile": {
        "verifier": {
            "did": "did:example:myshopping",
            "certVcRef": "https://myshopping.com/cert-vc/1",
            "name": "MyShopping",
            "ref": "https://myshopping.com"
        },
        "filter": {
            "credentialSchemas": [
                {
                    "id": "https://school-id.com/schema/student_id_v1.json",
                    "type": "OsdSchemaCredential",
                    "displayClaims": [
                        "com.school_id.v1.school_name",
                        "com.school_id.v1.pii",
                        "com.school_id.v1.student_id",
                        "com.school_id.v1.student_name"
                    ],
                    "requiredClaims": [
                        "com.school_id.v1.school_name",
                        "com.school_id.v1.student_name"
                    ]
                }
            ]
        },
        "process": {
            "endpoints": ["https://myshopping.com/verify"],
            "reqE2e": {
                "nonce": "uTX0SWpBnrG-gvkQg1MzUFQ",
                "curve": "Secp256r1",
                "publicKey": "zpuheLvAneYCdu3hjpdqF9BotnEpM2v7BmidRq5QBLKej",
                "cipher": "AES-256-CBC",
                "padding": "PKCS5"
            },
            "verifierNonce": "uYXNlNjQgZW5jb2Rpbmcgcw",
            "authType": 6
        }
    },
    "proof": {
        "type": "Secp256r1Signature2018",
        "created": "2024-04-29T11:27:30Z",
        "verificationMethod": "did:example:myshopping?versionId=1#assert",
        "proofPurpose": "assertionMethod",
        "proofValue": "zDgYdYMUYHURJLD7xdnWRiqWCEY5u5fKzZs6Z...MzLHoPiPQ9sSVfRrs1D"
    }
}
```

## 5. ProofRequest Profile

The ProofRequest Profile is a structured data bundle through which the Verifier conveys the requirements for a Zero-Knowledge Proof (ZKP)-based verification to the Holder. This profile specifies the necessary conditions for generating a ZKP proof and includes the following information.


| Category    | Details                                                                                                          |
| ----------- | ---------------------------------------------------------------------------------------------------------------- |
| Verify Info | • Name, DID, Reference URL  |
| ProofRequest      | • Name of the proof request <br>• Attributes the Holder must disclose explicitlyt<br>• Predicates the Holder must satisfy (e.g., ≥, ≤) |
| ZKP Proof Submission Method | • Information related to encrypted submission (recipient public key, nonce, algorithm, etc.) |


### 5.1. ProofRequest Profile Structure

#### 5.1.1. `ProofRequestProfile` object

```c#
def object ProofRequestProfile: "ProofRequest Profile"
{
    //--------------------------------------------------------------------------
    // Profile Metadata
    //--------------------------------------------------------------------------
    + uuid         "id"         : "profile id"
    + PROFILE_TYPE "type"       : "profile type", value("VerifyProfile")
    + string       "title"      : "profile title"
    - string       "description": "profile description", default(""), emptiable(true)
    - LogoImage    "logo"       : "logo image for presentation"
    + ENCODING     "encoding"   : "encoding", default("UTF-8")
    + LANGUAGE     "language"   : "language code"

    //--------------------------------------------------------------------------
    // Profile Contents
    //--------------------------------------------------------------------------
    + object "profile": "profile contents"
    {
        + ProviderDetail "verifier": "verifier information"

        + object "proofRequest" : "ProofRequest information" 
        {
            + string "name"                  : "proofs name"
            + string "version"               : "version"
            + nonce  "nonce"                 : "nonce"
            - object "requestedAttributes"   : "AttributeInfo", min_extend(1)
            - object "requestedPredicates"   : "PredicateInfo", min_extend(1)
        }

        + ReqE2e           "reqE2e"       : "E2E request information (without proof)"
    }

    //--------------------------------------------------------------------------
    // Proof
    //--------------------------------------------------------------------------
    + AssertProof "proof": "verifier's signature for the profile"
}
```

- `~/profile/proofRequest`: Defines the proof requirements requested by the verifier from the prover (user)  
  (See ZKP Data Specification #4.5: ProofRequest)

### 5.2. ProofRequest Profile Example

```json
{
    "id": "d1f26925-6743-4609-9932-e909dda0299f",
    "type": "VerifyProfile",
    "title": "MyShopping Student ID Verification",
    "description": "Discounts are not available if personal identifier submission is not agreed.",
    "encoding": "UTF-8",
    "language": "ko",
    "profile": {
        "verifier": {
            "did": "did:example:myshopping",
            "certVcRef": "https://myshopping.com/cert-vc/1",
            "name": "MyShopping",
            "ref": "https://myshopping.com"
        },
        "proofRequest": {
            "name": "myShopping",
            "nonce": "1068995366822249097155600",
            "requestedAttributes": {
            "attributeReferent1": {
                "name": "zkpname",
                "restrictions": [
                {
                    "credDefId": "did:example:woosanuniv:3:CL:did:example:woosanuniv:2:student_id:1.0:Tag1"
                }
                ]
            }
            },
            "requestedPredicates": {
            "predicateReferent1": {
                "name": "zkpbirth",
                "pType": "LE",
                "pValue": 20200103,
                "restrictions": [
                {
                    "credDefId": "did:example:woosanuniv:3:CL:did:example:woosanuniv:2:student_id:1.0:Tag1"
                }
                ]
            }
            }
		},
        "reqE2e": {
            "nonce": "uTX0SWpBnrG-gvkQg1MzUFQ",
            "curve": "Secp256r1",
            "publicKey": "zpuheLvAneYCdu3hjpdqF9BotnEpM2v7BmidRq5QBLKej",
            "cipher": "AES-256-CBC",
            "padding": "PKCS5"
        },
    },  
    "proof": {
        "type": "Secp256r1Signature2018",
        "created": "2024-04-29T11:27:30Z",
        "verificationMethod": "did:example:myshopping?versionId=1#assert",
        "proofPurpose": "assertionMethod",
        "proofValue": "zDgYdYMUYHURJLD7xdnWRiqWCEY5u5fKzZs6Z...MzLHoPiPQ9sSVfRrs1D"
    }
}
```