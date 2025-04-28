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

Credential Request Format
==

- Subject: Definition of ZKP Credential Request Structure
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

- [Credential Request Format](#credential-request-format)
  - [Revision History](#revision-history)
  - [1. Overview](#1-overview)
    - [1.1. Reference Documents](#11-reference-documents)
  - [2. CredentialRequest Structure](#2-credentialrequest-structure)
    - [2.1. CredentialRequest Object](#21-credentialrequest-object)
  - [3. Example](#3-example)
    - [3.1. Driver’s License CredentialRequest](#31-drivers-license-credentialrequest)

<!-- /TOC -->

<div style="page-break-after: always;"></div>

## 1. Overview

This document defines the structure and related information of the Credential Request used in OpenDID ZKP.

The Proof Request includes the following information:

- Title and description of the Credential Request
- Credential Request itself
- Attributes to be issued

The purpose of this document is to define the document format for Credential Request, called the Credential Request format.
The format is defined entirely using OSD (OpenDID Schema Definition Language).

### 1.1. Reference Documents

| Reference Name              | Document Title                               | Location |
| --------------------------- | -------------------------------------------- | -------- |
| [OSD]                       | OpenDID Schema Definition Language           |          |
| [ZKP-DATA-SPEC]             | (OpenDID) ZKP Data Specification             |          |
| [CREDENTIAL-REQUEST-FORMAT] | (OpenDID) Credential Request Format          |          |

<div style="page-break-after: always;"></div>

## 2. CredentialRequest Structure

For items not defined here, refer to `[ZKP-DATA-SPEC]`.

### 2.1. CredentialRequest Object

`CredentialRequest` is a Credential Request object.

```c#
def object CredentialRequest: "credential request"
{    
    + string    "proverDID"                                                : "prover DID"
    + definition-identifier "credDefId"                                    : "Credential Definition Identifier"
    + nonce    "nonce"                                                     : "nonce"
    + BlindedCredentialSecrets    "blindedMs"                              : "BlindedCredentialSecrets"
    + BlindedCredentialSecretsCorrectnessProof "blindedMsCorrectnessProof" : "BlindedCredentialSecretsCorrectnessProof"
}
```

- `~/proverDID`: The holder's DID, a unique identifier representing the subject
- `~/credDefId`: ID of the Credential Definition for the requested Credential. It points to the Credential Definition defined by the issuing organization, specifying the attributes and signing scheme for the Credential.
- `~/nonce`: A random nonce value to ensure the uniqueness of the request, maintaining security between Credential Offer and Credential Request.
- `~/blindedMs`: Blinded master secret
- `~/blindedMsCorrectnessProof`: Proof of the blinded master secret

<div style="page-break-after: always;"></div>

## 3. Example

This chapter provides an additional explanation with a specific example to aid understanding of the `CredentialRequest` concept.

- It is written in JSON according to the OpenDID Credential Request format.

### 3.1. Driver’s License CredentialRequest

```json
{
  "proverDID": "proverDID",
  "credDefId": "did:omn:NcYxiDXkpYi6ov5FcYDi1e:3:CL:did:omn:NcYxiDXkpYi6ov5FcYDi1e:2:schemaname:1.0:Tag1",
  "nonce": "1068995366822249097155600",
  "blindedMs": {
    "u": "10679991811650711848768425216683207546491227659900804094805932705507562488703092949797182055643626416369521786762621854083090126784466347663032697756490208368398394709964807887752061811372443336632810814957193078896089155286567310640005544357666552252291978844731742199418086678319628091129536102581277420864716283202498495664463166195122547548912743953823150988581658837131713169884469760569119071985844602430535777227228432711090136397498514806384271727937489889191000842223475821276163895096214377629643865574830573316348893425280147904666725164577575334795129716620654317029238864209863007914497893364195384614037",
    "hiddenAttributes": [
      "master_secret"
    ],
    "committedAttrs": {}
  },
  "blindedMsCorrectnessProof": {
    "c": "41371370778274456065789735068945549136946655928972626196405349872098675041396",
    "vDashCap": "537132447907651655710668360045088458013344753746169872702633918418350868401754668114190348956779680937227890626650576036957010815485548780356919795790390576079494998154644248433124465350362297987407101548671650775027191638673149630749728530165695690878058704543013313855460050891839776933929408530296078406764841766205193804905123347836354959051832654745953476678227692251307131910805714936794638723174330907547093124888619476935289131656067156995206988122350221272456606674355644053990149262249448862805155262474999019971714862574886756893083180930053757640543274232415031875760249963278818320927456146384895641941146405947063280051419456000786148623447196636757495712619488795851722997826592815864714583995024462585",
    "mCaps": {
      "masterSecret": "13040340850079303655869232120130921018882740985122585922749570432142176822976842922005110128291719609300742029254350129389559398615433088887829881946988784399082135738528704911842"
    },
    "rCaps": {}
  }
}
```
