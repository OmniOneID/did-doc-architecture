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

Credential Schema format
==

- Subject
    - Definition of ZKP Credential Schema structure
- Author: Park Dongjun
- Date: 2025-05-13
- Version: v1.0.1

Revision History
---

| Version | Date       | Changes |
|--------|------------|---------|
| v1.0.1 | 2025-05-13 | Added namespace (Suhyun Forten Lee) |
| v1.0.0 | 2025-04-30 | Initial draft |

<div style="page-break-after: always;"></div>

---

<!-- TOC tocDepth:2..4 chapterDepth:2..6 -->

- [Credential Schema format](#credential-schema-format)
  - [Revision History](#revision-history)
  - [1. Overview](#1-overview)
    - [1.1. Reference Documents](#11-reference-documents)
  - [2. Credential Schema Structure](#2-credential-schema-structure)
  - [3. Example](#3-example)
    - [3.1. Driver’s License Credential](#31-drivers-license-credential)
    - [3.2. `Driver’s License Credential Schema`](#32-drivers-license-credential-schema)

<!-- /TOC -->

<div style="page-break-after: always;"></div>

## 1. Overview

This document defines the structure and related information of the Credential Schema used in OpenDID ZKP.

A Credential Schema includes the following information:

- Title and description of the credential
- Credential Schema
- Attributes to be issued

The purpose of this document is to define the document format for defining Credential Schemas.
The format is entirely defined in OSD (OpenDID Schema Definition Language).

### 1.1. Reference Documents

| Reference Name       | Document Title                                    | Location |
|----------------------|---------------------------------------------------|----------|
| [OSD]                | OpenDID Schema Definition Language                |          |
| [DATA-SPEC]          | (OpenDID) Data Specification                      |          |
| [ZKP-DATA-SPEC]      | (OpenDID) ZKP Data Specification                  |          |
| [CREDENTIAL-FORMAT]  | (OpenDID) Credential format                       |          |

<div style="page-break-after: always;"></div>

## 2. Credential Schema Structure

- Defines the base structure for credential issuance
  - Serves as a template defining attributes to be included in a credential
  - The issuer uses this schema to generate credentials
- Used in ZKP-based proofs
  - `attrNames` can be selectively disclosed in ZKP protocols
  - For example, the user may disclose only ‘zkpsex’ (gender) while keeping ‘zkpbirth’ (birthdate) private
- Registered on the blockchain ledger
  - Issuers register the Credential Schema to make it publicly available
  - Verifiers use it to verify structure and validity of credentials

Any item not defined here should refer to `[ZKP-DATA-SPEC]`.

`CredentialSchema` is a Credential Schema object.

```c#
def string schema-identifier : "id", issuerDID:2:schemaName:version

def object CredentialSchema: "Credential schema"
{
    + schema-identifier    "id"          : "Credential Schema identifier"
    + string               "name"        : "Credential Schema name"
    + string               "version"     : "Credential Schema version"
    + array(string)        "attrNames"   : "Names of attributes"
    + array(object)        "attrTypes"   : "Namespace and types of each attribute"
    {
      + object "namespace": "attribute namespace"
      {
        + namespaceId "id"  : "attribute namespace", emptiable(true)
        + string      "name": "namespace name"
        - url         "ref" : "URL for namespace information"
      }
      + array(object) "items": "List of attribute definitions", min_count(1)
      {
        + string        "label"     : "Label for each attribute"
        + string        "caption"   : "Name of each attribute"
        + ATTR_TYPE     "type"      : "Type of each attribute"
        - object        "i18n"      : "Attribute name in other languages"
        {
          + string $lang: "Name in other language", variable_type(LANGUAGE), min_extend(1)
        }
      }
    }
    + string               "tag"         : "Credential Schema tag"
}
```

- `~/id`: Unique identifier of the Credential Schema
- `~/name`: Name of the schema (e.g., "mdl")
- `~/version`: Schema version information
- `~/attrNames`: List of attribute names (stored by namespace identifier)
- `~/attrTypes`: List of attribute namespaces and types
- `~/tag`: Used for distinguishing or filtering schemas

<div style="page-break-after: always;"></div>

## 3. Example

This section provides examples to aid understanding of complex concepts such as Credential and Credential Schema.
Scenario:

- The issuer issues a "Driver’s License Credential" to a specific user.
- The attributes in the credential are: gender, birthdate, license number, and address.
- These attributes are written in JSON according to the OpenDID Credential Schema format.

### 3.1. Driver’s License Credential

```json
{
  "schemaId": "did:omn:NcYxiDXkpYi6ov5FcYDi1e:2:mdl:1.0",
  "credDefId": "did:omn:NcYxiDXkpYi6ov5FcYDi1e:3:CL:did:omn:NcYxiDXkpYi6ov5FcYDi1e:2:mdl:1.0:Tag1",
  "values": {
    "org.rso.10001.zkpcity": {
      "encoded": "21224889883149768092131882119095173851218656983871668929472739274089421294476",
      "raw": "seoul"
    },
    "org.rso.10001.zkpphone": {
      "encoded": "91244139883149338092131882119112173851233256983871739294727392740894442944243",
      "raw": "1111"
    },
    "org.rso.10001.zkpsalary": {
      "encoded": "21224889883149768092131882119095173851218656983871668929472739274089421294476",
      "raw": "seoul"
    },
    "org.rso.10002.zkpsex": {
      "encoded": "5944657099558967239210949258394887428692050081607692519917050011144233115103",
      "raw": "male"
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

### 3.2. `Driver’s License Credential Schema`

```json
{
  "id": "did:omn:NcYxiDXkpYi6ov5FcYDi1e:2:mdl:1.0",
  "name": "mdl",
  "version": "1.0",
  "attrNames": [
    "org.rso.10001.zkpcity",
    "org.rso.10001.zkpphone",
    "org.rso.10001.zkpsalary",
    "org.rso.10002.zkpsex"
  ],
  "attrTypes": [
    {
      "namespace": {
        "id": "org.rso.10001",
        "name": "RSO 10001 Namespace",
        "ref": "https://www.rso.org/standard/10001.html"
      },
      "items": [
        {
          "label": "zkpcity",
          "caption": "City",
          "type": "String",
          "i18n": {
            "ko": "도시",
            "en": "City"
          }
        },
        {
          "label": "zkpphone",
          "caption": "Phone",
          "type": "String"
        },
        {
          "label": "zkpsalary",
          "caption": "Salary",
          "type": "Number"
        }
      ]
    },
    {
      "namespace": {
        "id": "org.rso.10002",
        "name": "RSO 10002 Namespace"
      },
      "items": [
        {
          "label": "zkpsex",
          "caption": "Gender",
          "type": "String"
        }
      ]
    }
  ],
  "tag": "Tag1"
}
```

- `~/id`: Unique identifier (ID) of the Credential Schema. Used to reference a specific schema.
- `~/name`: Name of the schema. (e.g., "mdl")
- `~/version`: Version of the schema
- `~/attrNames`: List of attribute names (stored by namespace identifier)
- `~/attrTypes`: List of attribute types
- `~/tag`: Tag used in Credential Definition, which can help distinguish between different credential definitions
