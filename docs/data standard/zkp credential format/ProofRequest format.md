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

Proof Request Format
==

- Subject: Definition of ZKP Proof Request Structure
- Author: Dongjun Park
- Date: 2025-04-30
- Version: v1.0.0

Revision History
---

| Version | Date       | Changes  |
| ------- | ---------- | -------- |
| v1.0.0  | 2025-04-30 | Draft    |

<div style="page-break-after: always;"></div>

---

<!-- TOC tocDepth:2..4 chapterDepth:2..6 -->

- [Proof Request Format](#proof-request-format)
  - [Revision History](#revision-history)
  - [1. Overview](#1-overview)
    - [1.1. Reference Documents](#11-reference-documents)
  - [2. Proof Request Structure](#2-proof-request-structure)
    - [2.1. Data Types and Constants](#21-data-types-and-constants)
    - [2.2. ProofRequest Object](#22-proofrequest-object)
  - [3. Example](#3-example)
    - [3.1. Driver’s License ProofRequest](#31-drivers-license-proofrequest)

<!-- /TOC -->

<div style="page-break-after: always;"></div>

## 1. Overview

This document defines the structure and related information of the Proof Request used in OpenDID ZKP.

A Proof Request is used when a verifier requests a holder to prove specific information.
It utilizes zero-knowledge proof (ZKP) techniques, allowing proof of specific attributes or conditions without disclosing the original data.

The purpose of this document is to define the Proof Request format.
The format is defined entirely using OSD (OpenDID Schema Definition Language).

### 1.1. Reference Documents

| Reference Name          | Document Title                           | Location |
| ------------------------ | ---------------------------------------- | -------- |
| [OSD]                    | OpenDID Schema Definition Language       |          |
| [ZKP-DATA-SPEC]          | (OpenDID) ZKP Data Specification         |          |
| [PROOF-REQUEST-FORMAT]   | (OpenDID) Proof Request Format            |          |

<div style="page-break-after: always;"></div>

## 2. Proof Request Structure

This defines the structure that specifies the proof conditions required by the verifier.

For items not defined here, refer to `[ZKP-DATA-SPEC]`.

### 2.1. Data Types and Constants

```c#
def string identifier : "id", issuerDID:2:schemaName:version
```

### 2.2. ProofRequest Object

`ProofRequest` is a Proof Request object.

```c#
def object ProofRequest: "proofRequest"
{
    + string "name"                      : "name"
    + nonce "nonce"                      : "nonce"
    - object "requestedAttributes"       : "AttributeInfo", min_extend(1)
    {
        + string "name"                  : "referent Key"
        + array(string) "restrictions"   : "restrictions"
    }
    - object "requestedPredicates"       : "PredicateInfo", min_extend(1)
    {
        + PREDICATE_TYPE "pType"         : "predicate type"
        + string "pValue"                : "predicate value"
        + array(string) "restrictions"   : "restrictions"
    }
}
```

- `~/name`: Name of the Proof Request (e.g., "mdl")
- `~/nonce`: Random number value (ensures uniqueness of the request)
- `~/requestedAttributes`: Attributes that must be disclosed by the user
  - `~/requestedAttributes/name`: Name of the requested attribute (e.g., "zkpsex" → gender)
  - `~/requestedAttributes/restriction`: Only credentials issued by a specific Credential Definition (credDefId) are allowed
- `~/requestedPredicates`: Attributes for which conditions must be satisfied
  - `~/requestedPredicates/name`: Name of the requested attribute (e.g., "zkpbirth" → date of birth)
  - `~/requestedPredicates/pType`: Comparison operator ("LE": ≤, "GE": ≥, "EQ": =)
  - `~/requestedPredicates/pValue`: Comparison reference value (e.g., 20200103 → January 3, 2020)
  - `~/requestedPredicates/restrictions`: Only credentials issued by a specific Credential Definition (credDefId) are allowed

<div style="page-break-after: always;"></div>

## 3. Example

This chapter provides an example to aid understanding of the ProofRequest concept.

Context:
- The verifier proposes a specific Credential Definition ID to the user.
- The proposed credential is a driver's license, with attributes decided as:
    - Gender, Date of Birth, License Number, Address
    - The Date of Birth must be earlier than or equal to January 3, 2020.
- These attributes are written in JSON format according to the OpenDID ProofRequest format.

This ProofRequest asks the user to prove gender, address, and identification number from a credential issued by a specific authority, and also to prove that their birth date is earlier than January 3, 2020, using zero-knowledge proof without revealing unnecessary information.

### 3.1. Driver’s License ProofRequest

```json
{
  "name": "mdl",
  "nonce": "1068995366822249097155600",
  "requestedAttributes": {
    "attributeReferent1": {
      "name": "zkpsex",
      "restrictions": [
        {
          "credDefId": "did:omn:NcYxiDXkpYi6ov5FcYDi1e:3:CL:did:omn:NcYxiDXkpYi6ov5FcYDi1e:2:schemaname:1.0:Tag1"
        }
      ]
    },
    "attributeReferent2": {
      "name": "zkpaddr",
      "restrictions": [
        {
          "credDefId": "did:omn:NcYxiDXkpYi6ov5FcYDi1e:3:CL:did:omn:NcYxiDXkpYi6ov5FcYDi1e:2:schemaname:1.0:Tag1"
        }
      ]
    },
    "attributeReferent3": {
      "name": "zkpasort",
      "restrictions": [
        {
          "credDefId": "did:omn:NcYxiDXkpYi6ov5FcYDi1e:3:CL:did:omn:NcYxiDXkpYi6ov5FcYDi1e:2:schemaname:1.0:Tag1"
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
          "credDefId": "did:omn:NcYxiDXkpYi6ov5FcYDi1e:3:CL:did:omn:NcYxiDXkpYi6ov5FcYDi1e:2:schemaname:1.0:Tag1"
        }
      ]
    }
  }
}
```
