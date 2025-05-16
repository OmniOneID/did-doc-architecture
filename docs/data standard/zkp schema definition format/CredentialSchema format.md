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

Credential Schema Format
==

- Subject: Definition of ZKP Credential Schema Structure
- Author: Park Dongjun
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

- [Credential Schema Format](#credential-schema-format)
  - [Revision History](#revision-history)
  - [1. Overview](#1-overview)
    - [1.1. Reference Documents](#11-reference-documents)
  - [2. Credential Schema Structure](#2-credential-schema-structure)
  - [3. Example](#3-example)
    - [3.1. Driver’s License Credential](#31-drivers-license-credential)
    - [3.2. Driver’s License Credential Schema](#32-drivers-license-credential-schema)

<!-- /TOC -->

<div style="page-break-after: always;"></div>

## 1. Overview

This document defines the structure and related information of the Credential Schema used in OpenDID ZKP.

A Credential Schema includes:

- Credential title and description
- Credential Schema
- Attributes to be issued

The purpose of this document is to define the Credential Schema format.
The format is entirely defined using OSD (OpenDID Schema Definition Language).

### 1.1. Reference Documents

| Reference Name        | Document Title                           | Location |
| ---------------------- | ---------------------------------------- | -------- |
| [OSD]                  | OpenDID Schema Definition Language       |          |
| [ZKP-DATA-SPEC]        | (OpenDID) ZKP Data Specification         |          |
| [CREDENTIAL-FORMAT]    | (OpenDID) Credential Format               |          |

<div style="page-break-after: always;"></div>

## 2. Credential Schema Structure

- Defines the basic structure for credential issuance
  - Acts as a template defining attributes included in the credential
  - Issuers generate credentials based on this schema
- Used for ZKP-based proofs
  - Attributes (`attrNames`) can be selectively disclosed individually
  - Example: A user can disclose only 'zkpsex' (gender) while keeping 'zkpbirth' (birthdate) private
- Registered on the blockchain ledger
  - The issuer registers the Credential Schema on the ledger for public use
  - Verifiers use it to validate the structure and authenticity of credentials

For items not defined here, refer to `[ZKP-DATA-SPEC]`.

`CredentialSchema` is a Credential Schema object.

```c#
def string schema-identifier : "id", issuerDID:2:schemaName:version

def object CredentialSchema: "Credential schema"
{
    + schema-identifier    "id"          : "Credential Schema Identifier"
    + string               "name"        : "Credential Schema Name"
    + string               "version"     : "Credential Schema Version"
    + array(string)        "attrNames"   : "Attribute Names"
    + array(object)        "attrTypes"   : "Attribute Types"
    {
      + string               "label"     : "Attribute Label"
      + ATTR_TYPE            "type"      : "Attribute Type"
    }
    + string               "tag"         : "Credential Schema Tag"
}
```

- `~/id`: Unique identifier of the Credential Schema
- `~/name`: Name of the schema (e.g., "mdl")
- `~/version`: Version information of the schema
- `~/attrNames`: List of attribute names included in the credential (e.g., zkpsex, zkpsort, zkpaddr, zkpbirth)
- `~/attrTypes`: List of attribute types included in the credential
- `~/tag`: Tag for distinguishing or filtering the schema

<div style="page-break-after: always;"></div>

## 3. Example

This chapter provides a concrete example to aid understanding of Credential and Credential Schema.

Context:

- The issuer issues a "Driver’s License Credential" to a user.
- Attributes in the Driver’s License Credential are decided as:
  - Gender, Date of Birth, License Number, Address
- These attributes are written in JSON according to the OpenDID Credential Schema format.

### 3.1. Driver’s License Credential

```json
{
  "schemaId": "did:omn:NcYxiDXkpYi6ov5FcYDi1e:2:mdl:1.0",
  "credDefId": "did:omn:NcYxiDXkpYi6ov5FcYDi1e:3:CL:did:omn:NcYxiDXkpYi6ov5FcYDi1e:2:mdl:1.0:Tag1",
  "values": {
    "zkpsex": {
      "encoded": "5944657099558967239210949258394887428692050081607692519917050011144233115103",
      "raw": "male"
    },
    "zkpbirth": {
      "encoded": "20010101",
      "raw": "20010101"
    },
    "zkpasort": {
      "encoded": "1111",
      "raw": "1111"
    },
    "zkpaddr": {
      "encoded": "21224889883149768092131882119095173851218656983871668929472739274089421294476",
      "raw": "seoul"
    }
  },
  "signature": {
    "pCredential": {
      "a": "...",
      "e": "...",
      "v": "...",
      "q": "...",
      "m2": "..."
    }
  },
  "signatureCorrectnessProof": {
    "se": "...",
    "c": "..."
  }
}
```

### 3.2. Driver’s License Credential Schema

```json
{
  "id": "did:omn:NcYxiDXkpYi6ov5FcYDi1e:2:mdl:1.0",
  "name": "mdl",
  "version": "1.0",
  "attrNames": [
    "zkpsex",
    "zkpasort",
    "zkpaddr",
    "zkpbirth"
  ],
  "attrTypes": [
    {
      "label": "zkpsex",
      "type": "String"
    },
    {
      "label": "zkpasort",
      "type": "String"
    },
    {
      "label": "zkpaddr",
      "type": "String"
    },
    {
      "label": "zkpbirth",
      "type": "Number"
    }
  ],
  "tag": "Tag1"
}
```

