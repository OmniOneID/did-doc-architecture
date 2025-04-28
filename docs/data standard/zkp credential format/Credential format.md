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

Credential Format
==

- Subject: Definition of Credential Data Format
- Author: Dongjun Park
- Date: 2025-04-30
- Version: v1.0.0

Revision History
---

| Version | Date       | Description                         |
| ------- | ---------- | ----------------------------------- |
| v0.0.1  | 2025-04-30 | Initial draft                       |

<div style="page-break-after: always;"></div>

---

<!-- TOC tocDepth:2..4 chapterDepth:2..6 -->

- [Credential Format](#credential-format)
  - [Revision History](#revision-history)
  - [1. Overview](#1-overview)
    - [1.1. Reference Documents](#11-reference-documents)
  - [2. Credential Structure](#2-credential-structure)
    - [2.1. Credential Object](#21-credential-object)
  - [3. Example](#3-example)
    - [3.1. Driver License Credential](#31-driver-license-credential)
<!-- /TOC -->

<div style="page-break-after: always;"></div>

## 1. Overview

A Credential is a digital certificate provided by the Issuer to the Holder and is used in decentralized identity (DID) systems.  
This Credential is based on a Zero-Knowledge Proof (ZKP) scheme, allowing a Verifier to validate its authenticity without accessing the original data.

The `schemaId` and `credDefId` identify the schema and credential definition used, showing what structure it follows and which issuer issued it.  
The `values` field contains the data attributes of the Credential, including the raw (original) and encoded (hashed/encrypted) values.  
The `signature` field includes the Issuer’s signature, which guarantees the integrity of the Credential.  
The `signatureCorrectnessProof` proves the correctness of the signature.

### 1.1. Reference Documents

| Reference Name     | Document Name                                 | Location |
| ------------------ | --------------------------------------------- | -------- |
| [OSD]              | OpenDID Schema Definition Language            |          |
| [ZKP-DATA-SPEC]    | OpenDID ZKP Data Specification                |          |

<div style="page-break-after: always;"></div>

## 2. Credential Structure

For undefined items, refer to `[ZKP-DATA-SPEC]`.

### 2.1. Credential Object

The `Vc` object is a Verifiable Credential issued by the Issuer to the Holder and has the following structure:

- Credential Metadata  
- Claim(s)  
- Proof(s)  

```c#
def object Credential: "credential"
{
    + identifier "credentialId" : "credential ID"
    + identifier "schemaId"     : "credentialSchema ID"
    + identifier "credDefId"    : "credentialDefinition ID"
    + object "values"           : "attributes", min_extend(1)
    {
        /* 
        ... Example (order must be preserved)
        - string "zkpsex" : "gender"
        - string "zkpasort" : "license number"
        - string "zkpaddr" : "address"
        - string "zkpbirth" : "birthdate"
        */
        + object $attributeValue : "AttributeValue"
        {
            + string "encoded" : ""
            + string "raw"     : "male" : "e.g. gender"
        }
    }
    + object "signature"         : "Signature data of the Credential"
    {
        + object "pCredential"   : "Each primary signature value and data"
        {
            + string "a"  : "Part of the signature created by the issuer."
            + string "e"  : "Public exponent used in signature verification."
            + string "v"  : "Large integer that is the core of the signature."
            + string "q"  : "Additional signature-related parameter."
            + string "m2" : "Additional element used in the Credential signature."
        }
    }
    + object "signatureCorrectnessProof"  : "Proof of signature correctness"
    {
        + string "se" : "Hash value that proves the signature has not been tampered."
        + string "c"  : "Auxiliary hash value for integrity verification."
    }
}
```
## 3. Example

This chapter provides additional explanations with specific examples to aid in understanding credentials.

### 3.1. Driver License Credential

```json
{
    "credentialId": "c6c46b76-0e1a-4332-9b41-1da9486cc09d",
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
        "a": "98875982616817464633446891595850223533338411118852331160492527902169982228434029982618083326551648373193472476076894086997067665256283142273346340431437577250581484730590711742478557216812708790671985725266496619229680790669974139868981005492763885479913264796575799952457157386217492982204962146543313028709351893901260065259028337430667792552513931453862210064851344216522920238659738030073723033457162009578015627852583048897931073666627858310281576631376971155583747094049925757999596572278240148754936102490597998189670415889983066072890051459353712714407884048509882924446592539578523129548570067780613771471225",
        "e": "259344723055062059907025491480697571938277889515152306249728583105665800713306759149981690559193987143012367913206299323899696942213235956742930187915937583288713959081860286100011",
        "v": "8003924528980512398665527067127206266102422402791865023680891265619728173261543086760776450200919501258934598949174486396261669148424714014628130969280934470907242101006736888939481197975337689570671161083517178370223598759671540247362731738187523961909627048405867761596654163809689210798730735846208050667898988958710182094741280880538710613866440611020224145864423224919664894975316451427153988498339959712486188492566961039276030600245123196489748427094263070793219195734069072981493123869034302082523825829831149811966382321241856439567453297105353672638411753270559068998865936632021041413306604752688853438089609419594092615437287364256556768634693661959647007446043985345893860951199419955518999300976556995229954165280936026024650575748334945692337106641322437529029881610050345037088967089439026437965556183547",
        "q": "111798412089149600739490511497622746191860661175817654311989700207873971697240493753602368312189338924220377961307929215656011926173950041368230363191794859746540233560073662357579840312618694411089384217244540514660420817158040769973784809060960292911236520125257310737080617217074994382477660358021886934884455091188453321828799626851014648531925876012314308767827289174175081008875875608865210130923798523714544872388030529560444562940581236108988851890367324005325854260240911070911323451689048347526677470071595403336997101835728837268110056274805332553970635010036151099684795930623168620437794075616110852780876",
        "m2": "88474552647414202211113104363819791025935012128158586480381797774592934936761"
      }
    },
    "signatureCorrectnessProof": {
      "se": "14167165735114063931501467052665778486563967544247295358718314972194209028368089604093559836384564729118012665054795331385964273316409459623136054192820370912553785403429839133719096642681883310516475988854071892781818205915615944419656047379029228964299049154931990408315825875592338475372435479041382476184655688142044432931017042191731620597788630738400041345397278185860118264766880952624575885973004166453519351010621786216341722962293800945112452296891369490464333734071005997801868059822163827351343950964118209638922256773121214988939646396055222956441887363910202316519956100752834333802738122592193221298520",
      "c": "19166554180334812160357905473487482537686440399208470721755111141041307412699"
    }
  }
```