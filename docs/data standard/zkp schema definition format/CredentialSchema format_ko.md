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

- 주제
    - ZKP Credential Schema 구조 정의
- 작성: Park Dongjun
- 일자: 2025-05-13
- 버전: v1.0.1

개정이력
---

| 버전   | 일자       | 변경 내용 |
| ------ | ---------- | ---------|
| v1.0.1 | 2025-05-13 | 네임스페이스 추가(Suhyun Forten Lee) |
| v1.0.0 | 2025-04-30 | 초안      |


<div style="page-break-after: always;"></div>

---

<!-- TOC tocDepth:2..4 chapterDepth:2..6 -->

- [Credential Schema format](#credential-schema-format)
  - [개정이력](#개정이력)
  - [1. 개요](#1-개요)
    - [1.1. 참조문서](#11-참조문서)
  - [2. Credential Schema 구조](#2-credential-schema-구조)
  - [3. 예시](#3-예시)
    - [3.1. 운전면허증 Credential](#31-운전면허증-credential)
    - [3.2. `운전면허증 Credential Schema`](#32-운전면허증-credential-schema)

<!-- /TOC -->

<div style="page-break-after: always;"></div>


## 1. 개요

본 문서는 OpenDID ZKP에서 사용하는 Credential Schema의 구조와 관련 정보를 정의한다.

Credential Schema는 다음의 정보를 포함한다.

- Credential 제목 및 설명
- Credential Schema
- 발급할 어트리뷰트


본 문서는 Credential Schema를 정의하기 위한 문서 형식인 Credential Schema format을 정의하는 것을 목적으로 한다.
format은 모두 OSD (OpenDID Schema Definition Language)로 정의한다.

### 1.1. 참조문서

| 참조명              | 문서명                                          | 위치 |
| ------------------- | ----------------------------------------------- | ---- |
| [OSD]               | OpenDID Schema Definition Language              |      |
| [DATA-SPEC]         | (OpenDID) 데이터 명세서(Data Specification) |      |
| [ZKP-DATA-SPEC]     | (OpenDID) ZKP 데이터 명세서(ZKP Data Specification) |      |
| [CREDENTIAL-FORMAT] | (OpenDID) Credential format                     |      |


<div style="page-break-after: always;"></div>

## 2. Credential Schema 구조
- 증명서 발급을 위한 기본구조 정의 
  - 증명서에 포함할 속성을 정의하는 템플릿 역할
  - 발급자는 이 스키마 기반으로 증명서를 생성함
- ZKP 기반증명에 사용
  - attrNames 속성들은 ZKP 프로토콜에서 개별적으로 선택적 공개가 가능
  - 예를 들어 사용자는 'zkpsex'(성별)만 공개하고 'zkpbirth'(생년월일)은 비공개로 유지할 수 있음
- 블록체인 원장에 등록됨
  - Credential Schema는 발급자가 원장에 등록하여 공개적으로 사용할 수 있도록 함
  - 검증자(Verifier)는 이를 사용하여 증명서의 구조와 유효성을 검증


여기에 정의되지 않은 항목은 `[ZKP-DATA-SPEC]`을 참조한다.

`CredentialSchema`는 Credential Schema object이다.

```c#
def string schema-identifier : "id", issuerDID:2:schemaName:version

def object CredentialSchema: "Credential schema"
{
    + schema-identifier    "id"          : "Credential Schema 식별자"
    + string               "name"        : "Credential Schema 이름"
    + string               "version"     : "Credential Schema 버전"
    + array(string)        "attrNames"   : "attribute 별 이름"
    + array(object)        "attrTypes"   : "attribute 별 네임스페이스 및 타입"
    {
      + object "namespace": "attribute namespace"
      {
        + namespaceId "id"  : "attribute namespace", emptiable(true)
        + string      "name": "namespace 이름"
        - url         "ref" : "namespace에 대한 정보 페이지 URL"
      }
      + array(object) "items": "attribute 정의 목록", min_count(1)
      {
        + string        "label"     : "attribute 별 라벨"
        + string        "caption"   : "attribute 별 이름"
        + ATTR_TYPE     "type"      : "attribute 별 타입"
        - object        "i18n"      : "기타 언어의 attribute 이름"
        {
          + string $lang: "기타언어 attribute 이름", variable_type(LANGUAGE), min_extend(1)
        }
      }
    }
    + string               "tag"         : "Credential Schema 테그"
}
```

- `~/id`: 해당 Credential Schema의 고유식별자
- `~/name`: 스키마의 이름 (예제에서는 'mdl' 이라는 이름을 사용)
- `~/version`: 해당 스키마의 버전정보
- `~/attrNames`: 증명서에 포함될 속성들의 이름 목록 (네임스페이스 식별자 기준 저장)
- `~/attrTypes`: 증명서에 포함될 속성들의 네임스페이스 및 타입 등 목록
- `~/tag`: 스키마를 구별하거나 필터링하는데 사용되는 태그

<div style="page-break-after: always;"></div>

## 3. 예시

본 장에서는 Credential, Credential Schema 등 복잡한 개념의 이해를 돕기 위해 구체적인 예를 들어 추가로 설명한다.
상황은 다음과 같다.

- 이슈어가 특정 사용자에게 "운전면허증 Credential"를 발급한다.
- 운전면허증 Credential 내 어트리뷰트는 아래 목록과 같이 하기로 결정하였다.
    - 성별, 생년월일, 면허번호, 주소
- 상기 어트리뷰트를 OpenDID Credential Schema format에 맞춰 JSON으로 작성한다.

### 3.1. 운전면허증 Credential

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

### 3.2. `운전면허증 Credential Schema`

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
- `~/id`: Credential Schema의 고유 식별자(ID). 특정 스키마를 참조하기 위해 사용됨.
- `~/name`: 스키마의 이름(Name). (예: "mdl")
- `~/version`: 해당 스키마의 버전정보
- `~/attrNames`: 증명서에 포함될 속성들의 이름 목록 (네임스페이스 식별자 기준 저장)
- `~/attrTypes`: 증명서에 포함될 속성들의 타입 등 목록
- `~/tag`: Credential Definition에서 사용되는 태그(Tag). 같은 Credential Definition을 구분하는 데 사용될 수 있음.



