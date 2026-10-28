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

- 주제
    - OID4VCI로 발급하는 ISO mdoc Credential 데이터 형식 정의
    - Credential Configuration과 mdoc 구조의 대응 규칙 정의
- 작성: 오픈소스개발팀
- 일자: 2026-10-22
- 버전: v1.0.0


개정이력
---

| 버전   | 일자       | 변경 내용 |
| ------ | ---------- | --------- |
| v1.0.0 | 2026-10-22 | 초안      |


<div style="page-break-after: always;"></div>

목차
---

<!-- TOC tocDepth:2..4 chapterDepth:2..6 -->

- [1. 개요](#1-개요)
    - [1.1. 참조문서](#11-참조문서)
- [2. 공통 규정](#2-공통-규정)
    - [2.1. Credential 형식](#21-credential-형식)
    - [2.2. 표기 규칙](#22-표기-규칙)
    - [2.3. 인코딩 규칙](#23-인코딩-규칙)
        - [2.3.1. CBOR 값 인코딩](#231-cbor-값-인코딩)
        - [2.3.2. 날짜 인코딩](#232-날짜-인코딩)
        - [2.3.3. 증명사진 (portrait)](#233-증명사진-portrait)
- [3. mdoc Credential 구조](#3-mdoc-credential-구조)
    - [3.1. 구조 개요](#31-구조-개요)
    - [3.2. 데이터 타입 및 상수](#32-데이터-타입-및-상수)
    - [3.3. `IssuerSignedItem` object](#33-issuersigneditem-object)
    - [3.4. `IssuerSigned` object](#34-issuersigned-object)
    - [3.5. `MobileSecurityObject` object](#35-mobilesecurityobject-object)
        - [3.5.1. `valueDigests`](#351-valuedigests)
        - [3.5.2. `deviceKeyInfo`](#352-devicekeyinfo)
        - [3.5.3. `validityInfo`](#353-validityinfo)
        - [3.5.4. `status`](#354-status)
        - [3.5.5. MSO 확장 필드](#355-mso-확장-필드)
    - [3.6. `IssuerAuth` (COSE_Sign1)](#36-issuerauth-cose_sign1)
        - [3.6.1. 헤더와 Issuer 키 식별](#361-헤더와-issuer-키-식별)
        - [3.6.2. 서명 생성](#362-서명-생성)
        - [3.6.3. 서명 검증](#363-서명-검증)
    - [3.7. Credential 응답](#37-credential-응답)
- [4. Credential Configuration](#4-credential-configuration)
    - [4.1. docType과 네임스페이스](#41-doctype과-네임스페이스)
    - [4.2. `MdocCredentialConfiguration` object](#42-mdoccredentialconfiguration-object)
    - [4.3. Claim 정의와 data element](#43-claim-정의와-data-element)
- [5. 예시](#5-예시)
    - [5.1. `mDocPID` Credential Configuration](#51-mdocpid-credential-configuration)
    - [5.2. `IssuerSigned`](#52-issuersigned)
    - [5.3. Credential 응답](#53-credential-응답)

<!-- /TOC -->


<div style="page-break-after: always;"></div>

## 1. 개요

본 문서는 OpenDID 3.0이 OID4VCI로 발급하는 ISO mdoc Credential의 데이터 구성을 정의한다.
Credential 구조는 `[ISO-18013-5]`의 `IssuerSigned`를 따르며, Credential Issuer와 월렛 간 전달 방식은 `[OID4VCI]`의 ISO mdoc Credential Format Profile을 따른다.
OID4VCI 발급 절차 전반은 `[OID4VCI-ISSUANCE]`를 참조한다.

OpenDID는 mdoc을 `mso_mdoc-did` 형식으로 발급한다.
이 형식은 MSO(Mobile Security Object)에 대한 Issuer 서명인 IssuerAuth를 Issuer DID 키로 생성하고, COSE `kid` 헤더에 Issuer DID key URL을 담아 서명 키를 식별한다.

아래 사항에 대해서는 본 문서에서 정의한 것을 준수하여야 한다.

- `IssuerSigned`, `IssuerSignedItem`, MSO의 구성
- IssuerAuth 서명과 Issuer 키 식별 방식
- Holder 바인딩 키(Device Key)와 상태 참조(`status`)의 위치 및 형식
- Credential Configuration의 `doctype`과 mdoc docType, 네임스페이스의 대응
- Credential 응답의 인코딩

아래 사항은 _본 문서의 범위 밖_ 이므로 해당 표준이나 구현을 따른다.

- 문서 유형별 data element의 정의 (PID, mDL 등은 각 문서 유형의 규정을 따른다)
- 제출(Presentation) 단계의 `DeviceResponse`, `DeviceSigned` 구조 (`[ISO-18013-5]`, `[ISO-18013-7]` 참조)
- Credential 요청 및 응답 암호화 (`[OID4VCI-ISSUANCE]` 참조)


### 1.1. 참조문서

| 참조명             | 문서명                                                                                  | 위치                                                                 |
| ------------------ | --------------------------------------------------------------------------------------- | -------------------------------------------------------------------- |
| [ISO-18013-5]      | ISO/IEC 18013-5 Personal identification — ISO-compliant driving licence — Part 5: Mobile driving licence (mDL) application |                                    |
| [ISO-18013-7]      | ISO/IEC TS 18013-7 Personal identification — ISO-compliant driving licence — Part 7: Mobile driving licence (mDL) add-on functions |                            |
| [OID4VCI]          | OpenID for Verifiable Credential Issuance 1.0 (Appendix A. Credential Format Profiles - ISO mdoc) | https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0.html |
| [RFC8949]          | Concise Binary Object Representation (CBOR)                                             | https://www.rfc-editor.org/rfc/rfc8949                               |
| [RFC8943]          | Concise Binary Object Representation (CBOR) Tags for Date                               | https://www.rfc-editor.org/rfc/rfc8943                               |
| [RFC9052]          | CBOR Object Signing and Encryption (COSE): Structures and Process                       | https://www.rfc-editor.org/rfc/rfc9052                               |
| [RFC9053]          | CBOR Object Signing and Encryption (COSE): Initial Algorithms                           | https://www.rfc-editor.org/rfc/rfc9053                               |
| [TOKEN-STATUS]     | Token Status List                                                                       | https://datatracker.ietf.org/doc/draft-ietf-oauth-status-list/       |
| [OSD]              | OpenDID Schema Definition Language                                                      | [OpenDID Schema Definition Language_ko.md](../../guide/osd/OpenDID%20Schema%20Definition%20Language_ko.md) |
| [OID4VCI-ISSUANCE] | (OpenDID) OID4VCI 기반 VC 발급                                                          | [OID4VCI Issuance_ko.md](../../concepts/OID4VCI%20Issuance_ko.md)    |
| [DID-DOC-FORMAT]   | (OpenDID) DID Document format                                                           | [DID Document format_ko.md](../did%20document%20format/DID%20Document%20format_ko.md) |


<div style="page-break-after: always;"></div>

## 2. 공통 규정

### 2.1. Credential 형식

| Format         | Credential                     | Issuer 키 식별                          | Holder 바인딩                                  | 상태 참조                 |
| -------------- | ------------------------------ | --------------------------------------- | ---------------------------------------------- | ------------------------- |
| `mso_mdoc-did` | ISO mdoc (`IssuerSigned`)      | IssuerAuth `kid` 헤더의 Issuer DID key URL | MSO `deviceKeyInfo.deviceKey` (Proof의 Holder 공개키) | MSO `status.status_list` |

- `mso_mdoc-did`는 OpenDID가 정의한 OID4VCI format 식별자이며, Credential Configuration의 `format`과 Issuer Metadata의 `credential_configurations_supported`에 사용한다.
- Credential은 `[ISO-18013-5]`의 `IssuerSigned` 구조를 CBOR로 인코딩한 것이다.
- IssuerAuth는 Issuer DID Document의 서명키로 서명하므로, 검증자는 신뢰저장소에 등록된 Issuer DID Document로 서명을 검증한다 (`[OID4VCI-ISSUANCE]` [4.6](../../concepts/OID4VCI%20Issuance_ko.md#46-credential-형식) 참조).
- mdoc은 상태 관리가 적용되는 형식이므로 Credential 요청의 Proof는 Holder DID URL `kid`로 키를 식별해야 한다 (`[OID4VCI-ISSUANCE]` [4.5](../../concepts/OID4VCI%20Issuance_ko.md#45-credential-요청과-holder-바인딩) 참조).


### 2.2. 표기 규칙

mdoc은 JSON이 아닌 CBOR(`[RFC8949]`) 데이터이다. 본 문서는 데이터 구조를 `[OSD]`로 정의하되 다음 규칙을 추가로 적용한다.

- CBOR 고유 타입(`tstr`, `bstr`, `uint`, `tdate`, `fullDate` 등)은 [3.2](#32-데이터-타입-및-상수)의 사용자 정의 타입으로 표현한다.
- 멤버 이름이 숫자인 경우(`"1"`, `"4"`, `"-1"` 등)는 문자열이 아닌 CBOR 정수 레이블을 의미한다. COSE 헤더와 COSE_Key가 이에 해당한다.
- `#6.24(bstr .cbor X)`는 `X`를 CBOR로 인코딩한 byte string을 tag 24(Encoded CBOR data item)로 감싼 값을 의미한다.

예시는 CBOR 진단 표기(diagnostic notation)로 작성한다.

| 표기                 | 의미                                                  |
| -------------------- | ----------------------------------------------------- |
| `"text"`             | text string (tstr)                                    |
| `h'0a1b...'`         | byte string (bstr). 예시에서는 일부만 표시할 수 있다. |
| `'text'`             | 문자열을 UTF-8로 인코딩한 byte string (bstr)          |
| `24(<< ... >>)`      | 내부 데이터를 CBOR로 인코딩한 bstr에 tag 24를 붙인 값 |
| `0("...")`, `1004("...")` | tag 0(date-time), tag 1004(full-date)를 붙인 tstr |
| `/ ... /`            | 주석                                                  |


### 2.3. 인코딩 규칙

#### 2.3.1. CBOR 값 인코딩

data element 값(`elementValue`)과 MSO 확장 필드 값은 Issuer가 전달한 값의 타입에 따라 아래와 같이 인코딩한다.

| 값의 타입   | CBOR 인코딩       | 비고                                                       |
| ----------- | ----------------- | ---------------------------------------------------------- |
| 문자열      | tstr              | 날짜 규칙 대상 키는 tag를 붙인다 ([2.3.2](#232-날짜-인코딩)). |
| 정수        | int (uint / nint) |                                                            |
| 불리언      | bool              |                                                            |
| byte 배열   | bstr              | 예: `portrait` ([2.3.3](#233-증명사진-portrait))           |
| map         | map (tstr 키)     | 하위 값에도 같은 규칙을 재귀적으로 적용한다.               |
| list        | array             | 각 항목을 같은 규칙으로 인코딩한다. 항목 자체에는 키가 없으므로 날짜 tag를 붙이지 않는다. |

#### 2.3.2. 날짜 인코딩

아래 키의 문자열 값에는 키 이름에 따라 CBOR tag를 붙인다.

| 키 이름       | CBOR 인코딩                     | 값 형식                  | 예시                         |
| ------------- | ------------------------------- | ------------------------ | ---------------------------- |
| `birth_date`  | tag 1004 + tstr (full-date)     | `YYYY-MM-DD`             | `1004("1990-01-01")`         |
| `expiry_date` | tag 1004 + tstr (full-date)     | `YYYY-MM-DD`             | `1004("2036-10-21")`         |
| `issue_date`  | tag 0 + tstr (tdate)            | RFC 3339 date-time       | `0("2026-10-22T09:30:00Z")`  |

- tag는 키 이름이 정확히 일치할 때만 붙는다. 위 표에 없는 날짜 data element(예: PID의 `issuance_date`)는 tag 없이 tstr로 인코딩된다.
- 같은 이름의 키가 map 값 내부(예: mDL `driving_privileges[]`의 `issue_date`, `expiry_date`)에 있어도 같은 tag를 붙인다.
- tag를 붙일 때 값의 형식은 변환하거나 검사하지 않는다. 따라서 Issuer는 위 표의 형식에 맞는 문자열을 전달해야 한다.
- full-date tag(1004)는 `[RFC8943]`, date-time tag(0)는 `[RFC8949]`에 정의되어 있다.

MSO의 `validityInfo` 일시는 위 규칙과 별개로 항상 tdate로 인코딩한다 ([3.5.3](#353-validityinfo) 참조).

#### 2.3.3. 증명사진 (portrait)

`portrait`(증명사진)는 JPEG 이미지 데이터를 bstr로 인코딩한다.

- OpenDID Issuer 서버의 Webview 발급에서 `value_type`이 `jpeg`인 Claim은 Base64 문자열로 입력받으며, JPEG 형식(SOI/EOI 마커)과 크기(최대 2MB)를 검사한다.
- Issuer 서버는 발급 시 `portrait`의 Base64 문자열을 디코딩하여 byte 배열로 전달하므로 `elementValue`는 bstr이 된다.
- 문자열로 전달된 값은 tstr로 인코딩되므로, `portrait`는 반드시 byte 배열로 전달해야 한다.


<div style="page-break-after: always;"></div>

## 3. mdoc Credential 구조

### 3.1. 구조 개요

OID4VCI로 발급하는 mdoc Credential은 `IssuerSigned` 구조이며, 아래와 같이 구성된다.

```
IssuerSigned
├── nameSpaces
│   └── <네임스페이스>                       예: "org.omnione.pid.1"
│       └── [ IssuerSignedItemBytes, ... ]   #6.24(bstr .cbor IssuerSignedItem)
│               └── { digestID, random, elementIdentifier, elementValue }
└── issuerAuth                               COSE_Sign1
    ├── protected     { 1 (alg): -7 (ES256) }
    ├── unprotected   { 4 (kid): Issuer DID key URL }
    ├── payload       MobileSecurityObjectBytes   #6.24(bstr .cbor MobileSecurityObject)
    │       └── { version, digestAlgorithm, valueDigests, deviceKeyInfo,
    │             docType, validityInfo, status }
    └── signature     ES256 서명 (R || S, 64 bytes)
```

- 각 data element는 `IssuerSignedItem`으로 만들어지고, 그 digest가 MSO의 `valueDigests`에 저장된다.
- IssuerAuth는 MSO에 서명하므로, data element는 digest를 통해 간접적으로 Issuer 서명의 보호를 받는다.
- 하나의 Credential은 하나의 네임스페이스만 포함한다 ([4.1](#41-doctype과-네임스페이스) 참조).


### 3.2. 데이터 타입 및 상수

여기에 정의되지 않은 항목은 `[ISO-18013-5]`를 참조한다.

```c#
//--------------------------------------------------------------------------------
// CBOR 데이터 타입
//--------------------------------------------------------------------------------
def string tstr    : "CBOR text string (major type 3)"
def string bstr    : "CBOR byte string (major type 2)"
def int    uint    : "CBOR unsigned integer (major type 0)", min_value(0)
def string tdate   : "tag 0을 붙인 RFC 3339 date-time 문자열. ex: 0('2026-10-22T02:10:57Z')"
def string fullDate: "tag 1004를 붙인 RFC 3339 full-date 문자열. ex: 1004('1990-01-01')"
def string any     : "임의의 CBOR 데이터 항목 (2.3절의 인코딩 규칙 적용)"
def string url     : "URL"

//--------------------------------------------------------------------------------
// mdoc 데이터 타입
//--------------------------------------------------------------------------------
def string nameSpace            : "네임스페이스. ex: 'org.omnione.pid.1'"
def string dataElementIdentifier: "data element 식별자. ex: 'family_name'"
def int    digestID             : "IssuerSignedItem의 digest 식별자", min_value(0)
def string digest               : "SHA-256 digest (32 bytes bstr)"
def string didKeyUrl            : "DID key URL. ex: 'did:omn:issuer?versionId=1#assert'"

def string IssuerSignedItemBytes    : "#6.24(bstr .cbor IssuerSignedItem)"
def string MobileSecurityObjectBytes: "#6.24(bstr .cbor MobileSecurityObject)"

//--------------------------------------------------------------------------------
// 상수
//--------------------------------------------------------------------------------
def enum MDOC_FORMAT: "OID4VCI Credential format 식별자"
{
    "mso_mdoc-did": "Issuer DID 키로 IssuerAuth를 서명한 ISO mdoc",
}

def enum DOC_TYPE: "mdoc 문서 유형 (docType)"
{
    "org.omnione.pid.1"      : "OmniOne PID",
    "org.iso.18013.5.1.mDL"  : "모바일 운전면허증 (mDL)",
}

def enum NAME_SPACE: "data element 네임스페이스"
{
    "org.omnione.pid.1"      : "OmniOne PID",
    "org.iso.18013.5.1"      : "모바일 운전면허증 (mDL)",
}

def enum MSO_VERSION: "MSO 버전"
{
    "1.0",
}

def enum DIGEST_ALGORITHM: "digest 알고리즘"
{
    "SHA-256",
}

def enum COSE_ALG: "COSE 알고리즘 식별자"
{
    -7: "ES256 (ECDSA, P-256, SHA-256)",
}

def enum COSE_KEY_TYPE: "COSE_Key 키 유형 (kty)"
{
    2: "EC2",
}

def enum COSE_EC_CURVE: "COSE_Key 타원곡선 (crv)"
{
    1: "P-256",
}
```


### 3.3. `IssuerSignedItem` object

`IssuerSignedItem`은 하나의 data element와 digest 계산에 필요한 값을 담는다.

```c#
def object IssuerSignedItem: "Issuer가 서명하는 data element"
{
    + digestID              "digestID"         : "digest 식별자"
    + bstr                  "random"           : "digest 계산용 난수 salt", byte_length(16)
    + dataElementIdentifier "elementIdentifier": "data element 식별자"
    + any                   "elementValue"     : "data element 값"
}
```

- `digestID`: 0부터 1씩 증가하는 정수를 Claim 순서대로 할당한다. 네임스페이스 내에서 유일하다.
- `random`: data element마다 새로 생성한 16 bytes 난수이다. 같은 값의 data element라도 digest가 달라지도록 하여 값 추측을 막는다.
- `elementIdentifier`: Credential Configuration의 `credential_metadata.claims[].path`의 마지막 항목이다 ([4.3](#43-claim-정의와-data-element) 참조).
- `elementValue`: [2.3](#23-인코딩-규칙)의 인코딩 규칙에 따라 인코딩한다.

`IssuerSignedItem`은 CBOR로 인코딩한 후 tag 24로 감싼 `IssuerSignedItemBytes` 형태로 `IssuerSigned`와 digest 계산에 사용한다.


### 3.4. `IssuerSigned` object

`IssuerSigned`는 Credential Issuer가 Holder에게 발급하는 mdoc Credential이다.

```c#
def object IssuerSigned: "OID4VCI로 발급하는 mdoc Credential"
{
    + object "nameSpaces": "네임스페이스별 data element 목록"
    {
        + array(IssuerSignedItemBytes) $nameSpace: "IssuerSignedItemBytes 목록",
            variable_type(nameSpace), min_extend(1), max_extend(1)
    }
    + IssuerAuth "issuerAuth": "MSO에 대한 Issuer 서명 (COSE_Sign1)"
}
```

- `nameSpaces`: Credential의 네임스페이스 하나를 키로 가지며, 값은 `IssuerSignedItemBytes` 배열이다. 배열 순서는 `digestID` 순서와 같다.
- `issuerAuth`: [3.6](#36-issuerauth-cose_sign1) 참조


### 3.5. `MobileSecurityObject` object

MSO(Mobile Security Object)는 data element의 digest, Holder 바인딩 키, 유효기간, 상태 참조를 담으며 IssuerAuth의 payload로 서명된다.

```c#
def object MobileSecurityObject: "MSO (Mobile Security Object)"
{
    + MSO_VERSION      "version"        : "MSO 버전", value("1.0")
    + DIGEST_ALGORITHM "digestAlgorithm": "digest 알고리즘", value("SHA-256")
    + object           "valueDigests"   : "네임스페이스별 digest 목록"
    {
        + object $nameSpace: "IssuerSigned의 네임스페이스",
            variable_type(nameSpace), min_extend(1), max_extend(1)
        {
            + digest $digestID: "IssuerSignedItemBytes의 SHA-256 digest",
                variable_type(digestID), min_extend(1)
        }
    }
    + object "deviceKeyInfo": "Holder 바인딩 키 정보"
    {
        + DeviceKey "deviceKey": "Holder 공개키 (COSE_Key)"
    }
    + DOC_TYPE "docType"     : "문서 유형"
    + object   "validityInfo": "MSO 유효기간"
    {
        + tdate "signed"    : "MSO 서명 일시"
        + tdate "validFrom" : "유효기간 시작 일시"
        + tdate "validUntil": "유효기간 종료 일시"
    }

    //---------------------------------------------------------------------------
    // MSO 확장 필드 (IssuerAuth 서명 범위에 포함)
    //---------------------------------------------------------------------------
    - object "status": "Credential 상태 참조"
    {
        + object "status_list": "Token Status List 참조"
        {
            + uint "idx": "Status List 내 Credential의 index"
            + url  "uri": "Status List URI"
        }
    }
}

def object DeviceKey: "Holder 공개키 (COSE_Key, EC2 P-256)"
{
    + COSE_KEY_TYPE "1" : "kty", value(2)
    + COSE_EC_CURVE "-1": "crv", value(1)
    + bstr          "-2": "x 좌표", byte_length(32)
    + bstr          "-3": "y 좌표", byte_length(32)
}
```

- `docType`: Credential Configuration의 `doctype`에서 결정한다 ([4.1](#41-doctype과-네임스페이스) 참조). MSO의 키 이름은 `docType`, Credential Configuration의 키 이름은 `doctype`이다.
- `[ISO-18013-5]`에 정의된 `validityInfo.expectedUpdate`, `deviceKeyInfo.keyAuthorizations`, `deviceKeyInfo.keyInfo`는 포함하지 않는다.

#### 3.5.1. `valueDigests`

`valueDigests`는 `{네임스페이스: {digestID: digest}}` 형태이며, digest는 다음과 같이 계산한다.

```
valueDigests[nameSpace][digestID] = SHA-256( IssuerSignedItemBytes )
```

- digest 계산 대상은 tag 24를 포함한 `IssuerSignedItemBytes` 전체의 CBOR 인코딩이다.
- `IssuerSigned.nameSpaces`의 모든 `IssuerSignedItemBytes`에 대해 digest가 하나씩 존재한다.

#### 3.5.2. `deviceKeyInfo`

`deviceKeyInfo.deviceKey`는 Holder 바인딩 키이며, Credential 요청의 Proof로 확인한 Holder 공개키이다.

- 월렛은 Proof JWT의 `kid`에 Holder DID URL을 지정한다. Credential Issuer는 Holder DID Document를 조회하여 `authentication`에 등록된 P-256 공개키로 Proof 서명을 검증하고, 이 공개키를 `deviceKey`로 사용한다.
- `deviceKey`는 COSE_Key(`[RFC9052]`, `[RFC9053]`)로 인코딩하며, `kty`(EC2), `crv`(P-256), `x`, `y`만 포함한다.
- 하나의 Credential 요청에 여러 Proof를 포함하면 Proof 키마다 mdoc을 하나씩 발급하며, 각 mdoc의 `deviceKey`는 해당 Proof의 공개키이다.
- 검증자는 제출 단계에서 `deviceKey`로 DeviceAuth 서명을 검증하여 Holder 바인딩을 확인한다 (본 문서의 범위 밖).

#### 3.5.3. `validityInfo`

| 항목         | 값                                             |
| ------------ | ---------------------------------------------- |
| `signed`     | MSO 생성 시각 (UTC, 초 단위로 절삭)            |
| `validFrom`  | `signed`와 같은 값                             |
| `validUntil` | `signed` + 365일 (31,536,000초)                |

- 세 값 모두 소수점 이하 초가 없는 tdate(`0("YYYY-MM-DDThh:mm:ssZ")`)로 인코딩한다.
- MSO 유효기간은 발급 시점 기준으로 고정되며 Credential Configuration으로 변경할 수 없다.
- MSO 유효기간은 data element인 `expiry_date`(문서 만료일)와 별개이다.

#### 3.5.4. `status`

`status`는 Credential의 상태를 확인하기 위한 Token Status List 참조이며 `[TOKEN-STATUS]`의 `status_list` 형식을 따른다.

| 항목                 | 타입 | 설명                                                        |
| -------------------- | ---- | ----------------------------------------------------------- |
| `status.status_list.uri` | tstr | Status List URI. `{Issuer 기본 URL}/status-lists/{listId}` |
| `status.status_list.idx` | uint | Status List에서 해당 Credential에 할당된 index              |

- Credential Issuer는 Credential 발급 시 Status List의 index를 할당하고 `status`를 MSO 확장 필드로 추가한다. 따라서 `status`는 IssuerAuth 서명 범위에 포함된다.
- OpenDID Issuer 서버는 `mso_mdoc-did` Credential에 항상 `status`를 포함한다.
- 상태 값과 상태 전이, Status List 조회 방법은 [Credential Status format](Credential%20Status%20format_ko.md)과 `[OID4VCI-ISSUANCE]` [5장](../../concepts/OID4VCI%20Issuance_ko.md#5-credential-상태-관리)을 참조한다.

#### 3.5.5. MSO 확장 필드

Credential Issuer는 MSO 표준 필드 외의 필드를 MSO에 추가할 수 있으며, 추가한 필드는 IssuerAuth 서명 범위에 포함된다.

- 확장 필드의 이름은 비어 있을 수 없다.
- 아래 MSO 표준 필드 이름은 예약어이며 확장 필드로 사용할 수 없다. 사용하면 발급이 실패한다.
    - `version`, `digestAlgorithm`, `valueDigests`, `deviceKeyInfo`, `docType`, `validityInfo`
- 확장 필드 값은 [2.3](#23-인코딩-규칙)의 인코딩 규칙을 따른다.
- OpenDID 발급에서 사용하는 확장 필드는 `status`뿐이다.


### 3.6. `IssuerAuth` (COSE_Sign1)

IssuerAuth는 MSO에 대한 Issuer 서명으로, `[RFC9052]`의 COSE_Sign1 구조이다.
tag 18(COSE_Sign1 tag)을 붙이지 않은 4개 항목의 배열로 인코딩한다.

```c#
def array(any) IssuerAuth: "COSE_Sign1 [protected, unprotected, payload, signature]", count(4)

def object IssuerAuthProtectedHeader: "protected header (bstr .cbor로 인코딩)"
{
    + COSE_ALG "1": "alg", value(-7)
}

def object IssuerAuthUnprotectedHeader: "unprotected header"
{
    + bstr "4": "kid. Issuer DID key URL(didKeyUrl)을 UTF-8로 인코딩한 값"
}
```

| index | 항목          | 타입                                          | 값                                        |
| :---: | ------------- | --------------------------------------------- | ----------------------------------------- |
| 0     | `protected`   | bstr .cbor `IssuerAuthProtectedHeader`        | `<< {1: -7} >>`                           |
| 1     | `unprotected` | `IssuerAuthUnprotectedHeader`                 | `{4: 'did:omn:issuer?versionId=1#assert'}` |
| 2     | `payload`     | bstr (`MobileSecurityObjectBytes`의 인코딩)   | `<< 24(<< MSO >>) >>`                     |
| 3     | `signature`   | bstr (64 bytes)                               | ES256 서명값 (R과 S를 연결한 64 bytes)    |

#### 3.6.1. 헤더와 Issuer 키 식별

- `protected`에는 `alg`(1)만 포함하며 값은 `-7`(ES256)이다.
- `unprotected`에는 `kid`(4)만 포함한다. `kid`는 Issuer DID key URL을 UTF-8로 인코딩한 bstr이다.
    - 형식: `{Issuer DID}?versionId={DID Document 버전}#{키 id}` (예: `did:omn:issuer?versionId=1#assert`)
    - 키 id는 Issuer DID Document의 서명키(`assert`)이며, `assertionMethod`에 등록된 키이다 (`[DID-DOC-FORMAT]` [3.4](../did%20document%20format/DID%20Document%20format_ko.md#34-provider-did-document) 참조).
- MSO에는 Issuer 식별자가 없으며, Issuer는 `kid`의 DID로만 식별된다.

#### 3.6.2. 서명 생성

Credential Issuer는 다음 순서로 IssuerAuth를 생성한다.

1. Claim마다 `IssuerSignedItemBytes`를 만들고 digest를 계산한다.
1. MSO를 구성하고 확장 필드(`status`)를 추가한 뒤, `MobileSecurityObjectBytes`(`#6.24(bstr .cbor MobileSecurityObject)`)로 인코딩한다.
1. `protected` 헤더(`{1: -7}`)와 `unprotected` 헤더(`{4: kid}`)를 구성한다.
1. 서명 대상인 Sig_structure를 구성한다 (`[RFC9052]` 4.4절).
    ```
    Sig_structure = [ "Signature1", protected, h'', payload ]
    ```
    - `external_aad`는 빈 bstr(`h''`)이다.
    - `payload`는 `MobileSecurityObjectBytes`의 인코딩이다.
1. Sig_structure의 CBOR 인코딩을 SHA-256으로 해시하고, Issuer 월렛의 서명키(`kid`의 키 id)로 서명한다.
1. 서명값을 Issuer DID Document의 공개키로 검증 가능한 64 bytes `R || S` 형식으로 정규화한다.
1. `[protected, unprotected, payload, signature]` 배열을 `issuerAuth`로 설정한다.

#### 3.6.3. 서명 검증

검증자는 `IssuerSigned`를 다음과 같이 검증한다.

1. `issuerAuth`의 `protected` 헤더에서 `alg`가 `-7`(ES256)인지 확인한다.
1. `unprotected` 헤더의 `kid`를 UTF-8로 디코딩하여 Issuer DID key URL을 얻는다.
1. 신뢰저장소에서 해당 버전(`versionId`)의 Issuer DID Document를 조회하고, 키 id와 같은 공개키로 Sig_structure에 대한 서명을 검증한다.
1. `kid`는 서명 범위에 포함되지 않으므로, `kid`의 DID가 신뢰하는 Issuer의 DID인지 별도로 확인한다.
1. `payload`에서 tag 24를 해제하여 MSO를 얻고, `docType`과 `validityInfo`(현재 시각이 `validFrom`~`validUntil` 범위인지)를 확인한다.
1. `nameSpaces`의 각 `IssuerSignedItemBytes`에 대해 SHA-256 digest를 계산하여 `valueDigests[nameSpace][digestID]`와 비교한다.
1. `status.status_list`의 `uri`와 `idx`로 Credential 상태를 확인한다 (`[OID4VCI-ISSUANCE]` [5.5](../../concepts/OID4VCI%20Issuance_ko.md#55-검증자의-상태-확인) 참조).

> **참고**: 본 절은 본 문서의 데이터 구조에 따른 검증 항목을 설명한다. 제출 단계의 DeviceAuth 검증과 OpenDID 검증 사업자 서버의 구현은 별도 문서를 참조한다.


### 3.7. Credential 응답

Credential Endpoint는 `IssuerSigned`를 CBOR로 인코딩하고, padding 없는 base64url로 인코딩한 문자열을 `credentials[].credential`에 담아 응답한다.

```c#
def object CredentialResponse: "Credential 응답"
{
    + array(object) "credentials": "발급된 Credential 목록", min_count(1)
    {
        + string "credential": "base64url(IssuerSigned의 CBOR 인코딩), padding 없음"
    }
}
```

- 하나의 요청에 여러 Proof를 포함하면 Proof 키마다 Credential을 하나씩 발급하여 `credentials` 배열에 담는다.
- Issuer Metadata의 `credential_response_encryption.encryption_required`가 `true`이면 응답 전체를 JWE로 암호화한다 (`[OID4VCI-ISSUANCE]` [4.5](../../concepts/OID4VCI%20Issuance_ko.md#45-credential-요청과-holder-바인딩) 참조).
- `notification_id`는 포함하지 않는다.


<div style="page-break-after: always;"></div>

## 4. Credential Configuration

mdoc Credential은 Credential Configuration에 정의된 `doctype`과 Claim 정의에 따라 발급된다.
Credential Configuration의 Metadata는 Issuer Metadata의 `credential_configurations_supported`에 Configuration ID를 키로 공개된다 (`[OID4VCI-ISSUANCE]` [3.1](../../concepts/OID4VCI%20Issuance_ko.md#31-credential-configuration) 참조).

### 4.1. docType과 네임스페이스

Credential Issuer는 Credential 요청에서 확인한 Credential Configuration의 Metadata에서 `doctype`을 읽고, MSO의 `docType`과 data element의 네임스페이스를 결정한다.
mDL(`org.iso.18013.5.1.mDL`)은 네임스페이스 `org.iso.18013.5.1`을 사용하고, 그 외의 `doctype`은 `doctype` 값을 docType과 네임스페이스에 그대로 사용한다.

| Credential Configuration `doctype` | MSO `docType`             | 네임스페이스              | 문서 유형 |
| ---------------------------------- | ------------------------- | ------------------------- | --------- |
| `org.omnione.pid.1`                | `org.omnione.pid.1`       | `org.omnione.pid.1`       | OmniOne PID |
| `org.iso.18013.5.1.mDL`            | `org.iso.18013.5.1.mDL`   | `org.iso.18013.5.1`       | mDL       |

- 모든 data element는 결정된 하나의 네임스페이스에 포함된다.
- Metadata에 `doctype`이 없거나 빈 값이면 발급이 실패한다.

> **참고**: `org.omnione.pid.1`처럼 `doctype`을 docType과 네임스페이스에 그대로 사용하려면 사용자 정의 문서 유형을 지원하는 mdoc SDK 버전이 필요하다. 지원하지 않는 버전에서는 SDK에 내장된 문서 유형 외의 `doctype`으로 발급하면 실패한다. `credential_metadata.claims[].path`의 네임스페이스는 `doctype`과 같은 값을 사용해야 한다.

### 4.2. `MdocCredentialConfiguration` object

`credential_configurations_supported`의 mdoc 항목은 다음과 같이 구성한다.

```c#
def object MdocCredentialConfiguration: "credential_configurations_supported의 mdoc 항목"
{
    + MDOC_FORMAT   "format" : "Credential 형식", value("mso_mdoc-did")
    + DOC_TYPE      "doctype": "문서 유형 (4.1절)"
    - string        "scope"  : "Authorization Code 흐름에서 사용하는 OAuth 2.0 scope"
    - array(string) "cryptographic_binding_methods_supported": "Holder 바인딩 방식"
    - array(int)    "credential_signing_alg_values_supported": "IssuerAuth 서명 알고리즘 (COSE 알고리즘 식별자)"
    - object        "proof_types_supported": "지원 Proof 유형"
    {
        + object "jwt": "JWT Proof"
        {
            + array(string) "proof_signing_alg_values_supported": "Proof 서명 알고리즘"
        }
    }
    - object "credential_metadata": "Credential 표시 정보 및 Claim 정의"
    {
        - array(object) "display": "Credential 표시 정보"
        {
            + string "name"  : "Credential 이름"
            - string "locale": "언어 태그. ex: 'en'"
            - object "logo"  : "로고"
            {
                + url    "uri"     : "로고 이미지 URI"
                - string "alt_text": "대체 텍스트"
            }
        }
        - array(MdocClaimDescription) "claims": "Claim 정의 목록"
    }
}

def object MdocClaimDescription: "mdoc Claim 정의"
{
    + array(string) "path"     : "[네임스페이스, data element 식별자]", count(2)
    - bool          "mandatory": "필수 여부", default(false)
    - array(object) "display"  : "Claim 표시 정보"
    {
        - string "name"  : "Claim 이름"
        - string "locale": "언어 태그"
    }
    - string        "value_type": "(OpenDID 확장) Webview 입력값 타입 (4.3절)", default("string")
    - array(string) "enum"      : "(OpenDID 확장) Webview 입력 허용 값 목록"
}
```

| 항목 | OpenDID 규칙 |
| ---- | ------------ |
| `format` | `mso_mdoc-did` |
| `doctype` | [4.1](#41-doctype과-네임스페이스)의 표에 있는 값 |
| `cryptographic_binding_methods_supported` | Holder 바인딩 키는 COSE_Key로 MSO에 포함되므로 `cose_key`를 포함한다. |
| `credential_signing_alg_values_supported` | IssuerAuth는 `-7`(ES256)로 서명하므로 `[-7]`을 공개한다. |
| `proof_types_supported` | `jwt` Proof만 서명을 검증하며, Proof 서명 알고리즘은 `ES256`이다. |
| `credential_metadata.claims[].path` | `[네임스페이스, data element 식별자]`. 네임스페이스는 [4.1](#41-doctype과-네임스페이스)에서 결정되는 값과 같아야 한다. |
| `credential_metadata.claims[].mandatory` | Webview 발급에서 필수 입력 여부로 사용한다. |

### 4.3. Claim 정의와 data element

- Credential Issuer는 `credential_metadata.claims[].path`의 마지막 항목을 data element 식별자(`elementIdentifier`)로 사용한다.
- 발급되는 data element는 Credential Issuer가 해당 사용자에 대해 보유한 Claim 값으로 구성된다. 값이 없는 선택 Claim은 mdoc에 포함되지 않는다.
- OpenDID Issuer 서버의 Webview 발급(사용자에 의한 발급 개시)에서는 `value_type`에 따라 입력값을 검사하고 변환한다.

| `value_type`              | 입력값 처리                                         | `elementValue` 인코딩                               |
| ------------------------- | --------------------------------------------------- | --------------------------------------------------- |
| `string` (기본값)         | 그대로 사용 (`enum`이 있으면 허용 값인지 검사)       | tstr                                                |
| `full-date`, `date`       | `YYYY-MM-DD` 형식 검사                              | tstr (`birth_date`, `expiry_date`는 tag 1004)       |
| `uint`, `integer`         | 정수로 변환 (`uint`는 0 이상)                       | int                                                 |
| `boolean`                 | `true` 또는 `false`                                 | bool                                                |
| `list`                    | JSON 배열로 변환                                    | array                                               |
| `jpeg`                    | Base64 JPEG 검사 (최대 2MB)                         | bstr (`portrait`, [2.3.3](#233-증명사진-portrait)) |
| 그 외 (예: `places`, `nationalities`) | 그대로 사용 (`enum`이 있으면 허용 값인지 검사) | tstr                                         |


<div style="page-break-after: always;"></div>

## 5. 예시

### 5.1. `mDocPID` Credential Configuration

Issuer 서버 샘플 Metadata의 `mDocPID` Credential Configuration이다. `claims`는 일부만 표시하였다.

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

- `doctype`이 `org.omnione.pid.1`이므로 MSO `docType`과 네임스페이스는 모두 `org.omnione.pid.1`이다.
- 샘플 Metadata에는 `policy`(`batch_size`, `one_time_use`) 항목도 있으나 OpenDID 발급에는 사용하지 않는다 (`[OID4VCI-ISSUANCE]` [6장](../../concepts/OID4VCI%20Issuance_ko.md#6-구현-범위-및-제약) 참조).

### 5.2. `IssuerSigned`

`mDocPID`로 발급한 `IssuerSigned`의 CBOR 진단 표기이다. data element와 bstr 값은 일부만 표시하였다.

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
        "elementValue": "2026-10-22"            / tag 없음 /
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

- `issuance_date`는 날짜 tag 규칙 대상이 아니므로 tstr로 인코딩된다 ([2.3.2](#232-날짜-인코딩) 참조).
- `validUntil`은 `signed`로부터 365일 후이다.
- 검증자는 `kid`의 `did:omn:issuer` DID Document(`versionId=1`)에서 `assert` 키로 서명을 검증한다.

### 5.3. Credential 응답

```json
{
  "credentials": [
    {
      "credential": "omppc3N1ZXJBdXRohEOhASahBFghZGlkOm9tbjppc3N1ZXI_dmVyc2lvbklkPTEjYXNzZXJ0..."
    }
  ]
}
```

- `credential`은 [5.2](#52-issuersigned)의 `IssuerSigned`를 CBOR로 인코딩한 후 padding 없는 base64url로 인코딩한 값이다.
