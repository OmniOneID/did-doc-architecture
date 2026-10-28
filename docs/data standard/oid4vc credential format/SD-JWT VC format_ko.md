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

SD-JWT VC format
==

- 주제
    - OID4VCI로 발급하는 SD-JWT VC 데이터 형식 정의
    - Credential Configuration과 SD-JWT VC의 대응 관계 정의
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
    - [1.2. 용어](#12-용어)
- [2. 공통 규정](#2-공통-규정)
    - [2.1. Credential 형식](#21-credential-형식)
    - [2.2. SD-JWT VC 구조](#22-sd-jwt-vc-구조)
    - [2.3. 데이터 타입 및 상수](#23-데이터-타입-및-상수)
    - [2.4. Issuer-signed JWT](#24-issuer-signed-jwt)
        - [2.4.1. JWT Header](#241-jwt-header)
        - [2.4.2. JWT Payload](#242-jwt-payload)
        - [2.4.3. 서명](#243-서명)
    - [2.5. Disclosure](#25-disclosure)
    - [2.6. Claim의 공개 구분](#26-claim의-공개-구분)
        - [2.6.1. 예약 Claim 이름](#261-예약-claim-이름)
    - [2.7. Holder 바인딩](#27-holder-바인딩)
    - [2.8. 상태 참조](#28-상태-참조)
    - [2.9. Key Binding JWT](#29-key-binding-jwt)
- [3. Credential Configuration](#3-credential-configuration)
    - [3.1. Credential Configuration 구조](#31-credential-configuration-구조)
    - [3.2. SD-JWT VC와의 대응](#32-sd-jwt-vc와의-대응)
    - [3.3. Credential Configuration 예시](#33-credential-configuration-예시)
- [4. SD-JWT VC 예시](#4-sd-jwt-vc-예시)
    - [4.1. Issuer-signed JWT Header](#41-issuer-signed-jwt-header)
    - [4.2. Issuer-signed JWT Payload](#42-issuer-signed-jwt-payload)
    - [4.3. Disclosure](#43-disclosure)
    - [4.4. Compact 직렬화](#44-compact-직렬화)
    - [4.5. Credential 응답](#45-credential-응답)
    - [4.6. 제출 형식](#46-제출-형식)

<!-- /TOC -->


<div style="page-break-after: always;"></div>

## 1. 개요

본 문서는 OpenDID 3.0이 OID4VCI(OpenID for Verifiable Credential Issuance)로 발급하는 SD-JWT VC의 데이터 구성을 정의한다.
기본적인 규정은 IETF의 `[SD-JWT]`(Selective Disclosure for JSON Web Tokens)와 `[SD-JWT-VC]`(SD-JWT-based Verifiable Credentials)를 준수하며,
Credential Configuration은 `[OID4VCI]`의 Credential Issuer Metadata를 따른다.
발급 절차와 상태 관리의 개념은 `[OID4VCI-ISSUANCE]`를 참조한다.

아래 사항에 대해서는 본 문서에서 정의한 것을 준수하여야 한다.

- Issuer-signed JWT의 Header와 Claim 구성
- Disclosure 구성과 선택적 공개 대상의 구분
- Holder 바인딩(`cnf`)과 상태 참조(`status`) 구성
- Credential Configuration과 SD-JWT VC의 대응 관계

아래 사항은 _본 문서의 범위 밖_ 이므로 구현체에 따라 자유롭게 정의할 수 있다.

- `vct`별 Subject Claim의 종류와 의미
- Token Status List의 형식과 상태 관리 절차 (`[TOKEN-STATUS]`, `[OID4VCI-ISSUANCE]` 참조)
- 제출(Presentation) 프로토콜
- 기타 본 문서의 규정을 기본으로 항목을 추가하는 경우


### 1.1. 참조문서

| 참조명             | 문서명                                                   | 위치                                                                                                   |
| ------------------ | -------------------------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| [SD-JWT]           | RFC 9901 Selective Disclosure for JSON Web Tokens        | https://www.rfc-editor.org/rfc/rfc9901                                                                 |
| [SD-JWT-VC]        | SD-JWT-based Verifiable Credentials (SD-JWT VC)          | https://datatracker.ietf.org/doc/draft-ietf-oauth-sd-jwt-vc/                                           |
| [OID4VCI]          | OpenID for Verifiable Credential Issuance 1.0            | https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0.html                              |
| [TOKEN-STATUS]     | Token Status List                                        | https://datatracker.ietf.org/doc/draft-ietf-oauth-status-list/                                         |
| [RFC7515]          | JSON Web Signature (JWS)                                 | https://www.rfc-editor.org/rfc/rfc7515                                                                 |
| [RFC7517]          | JSON Web Key (JWK)                                       | https://www.rfc-editor.org/rfc/rfc7517                                                                 |
| [RFC7518]          | JSON Web Algorithms (JWA)                                | https://www.rfc-editor.org/rfc/rfc7518                                                                 |
| [RFC7800]          | Proof-of-Possession Key Semantics for JSON Web Tokens    | https://www.rfc-editor.org/rfc/rfc7800                                                                 |
| [OSD]              | OpenDID Schema Definition Language                       | [OpenDID Schema Definition Language_ko.md](../../guide/osd/OpenDID%20Schema%20Definition%20Language_ko.md) |
| [DATA-SPEC]        | (OpenDID) 데이터 명세서(Data Specification)              | [Data Specification_ko.md](../data%20specification/Data%20Specification_ko.md)                         |
| [DID-DOC-FORMAT]   | (OpenDID) DID Document format                            | [DID Document format_ko.md](../did%20document%20format/DID%20Document%20format_ko.md)                  |
| [OID4VCI-ISSUANCE] | (OpenDID) OID4VCI 기반 VC 발급                           | [OID4VCI Issuance_ko.md](../../concepts/OID4VCI%20Issuance_ko.md)                                      |


### 1.2. 용어

| 용어                | 설명                                                                                          |
| ------------------- | --------------------------------------------------------------------------------------------- |
| SD-JWT VC           | Issuer-signed JWT와 Disclosure로 구성되어 Claim 단위의 선택적 공개를 지원하는 VC              |
| Issuer-signed JWT   | Issuer가 서명한 JWT. Issuer-signed Claim과 Disclosure의 digest 목록을 담는다.                 |
| Disclosure          | 선택적 공개 Claim 1개의 salt, Claim 이름, Claim 값을 담은 문자열                              |
| Digest              | Disclosure의 해시값. Issuer-signed JWT의 `_sd`에 포함된다.                                    |
| Issuer-signed Claim | Issuer-signed JWT에 그대로 포함되어 항상 공개되는 Claim. Holder가 제출 시 제외할 수 없다.     |
| 선택적 공개 Claim   | Disclosure로 전달되어 Holder가 제출 시 공개 여부를 선택하는 Claim                             |
| Holder 바인딩       | Credential을 Holder 키에 결합하는 것. `cnf` Claim에 Holder 공개키를 포함한다.                 |
| KB-JWT              | Key Binding JWT. 제출 시 Holder가 바인딩 키로 서명하여 키 소유를 증명하는 JWT                 |


<div style="page-break-after: always;"></div>

## 2. 공통 규정

### 2.1. Credential 형식

OpenDID 3.0은 SD-JWT VC를 아래 Credential Format 식별자로 발급한다.

| Format          | Credential | Issuer 키 식별             | JWT Header `typ` | Holder 바인딩 |
| --------------- | ---------- | -------------------------- | ---------------- | ------------- |
| `dc+sd-jwt-did` | SD-JWT VC  | Issuer DID key URL (`kid`) | `dc+sd-jwt-did`  | `cnf.jwk`     |

- `dc+sd-jwt-did`는 OpenDID에서 정의한 Format 식별자로, Issuer 서명 키를 신뢰저장소에 등록된 Issuer DID Document에서 확인하는 SD-JWT VC를 나타낸다.
- Credential Configuration의 `format`과 Issuer-signed JWT Header의 `typ`에 같은 값을 사용한다.


### 2.2. SD-JWT VC 구조

SD-JWT VC는 Issuer-signed JWT와 Disclosure를 `~`로 연결한 Compact 직렬화 문자열이다.
발급 시에는 KB-JWT 없이 마지막 `~`로 끝난다.

```
<Issuer-signed JWT>~<Disclosure 1>~<Disclosure 2>~...~<Disclosure N>~
```

제출 시 Holder는 공개할 Disclosure만 선택하고, 필요하면 마지막에 KB-JWT를 붙인다 ([2.9](#29-key-binding-jwt) 참조).

```
<Issuer-signed JWT>~<Disclosure i>~...~<Disclosure k>~<KB-JWT>
```

| 구성              | 형식         | 설명                                                                                  |
| ----------------- | ------------ | ------------------------------------------------------------------------------------- |
| Issuer-signed JWT | `compactJws` | Issuer가 서명한 JWT. Issuer-signed Claim과 Disclosure의 digest 목록(`_sd`)을 포함한다. |
| Disclosure        | `disclosure` | 선택적 공개 Claim 1개. 0개 이상 포함한다.                                             |
| KB-JWT            | `compactJws` | (제출 시, 선택) Holder가 `cnf` 키로 서명한 JWT                                        |


### 2.3. 데이터 타입 및 상수

여기에 정의되지 않은 타입(`did`, `didKeyUrl`, `url`, `LANGUAGE`)은 `[DATA-SPEC]`을 참조한다.

```c#
def string base64url  : "base64url 인코딩 문자열 (padding 없음)"
def string compactJws : "JWS Compact Serialization 문자열 (header.payload.signature)"
def string disclosure : "Disclosure. JSON 배열을 UTF-8로 인코딩한 후 base64url로 인코딩한 문자열"
def string sdDigest   : "Disclosure digest. base64url(SHA-256(ASCII(disclosure)))"
def string salt       : "16바이트 난수를 base64url로 인코딩한 문자열", length(22)
def int    numericDate: "NumericDate. 1970-01-01T00:00:00Z부터의 초", min_value(0)

def enum CREDENTIAL_FORMAT: "Credential format"
{
    "dc+sd-jwt-did": "Issuer DID로 서명 키를 식별하는 SD-JWT VC",
}

def enum JWT_TYPE: "JWT Header typ"
{
    "dc+sd-jwt-did": "Issuer-signed JWT",
    "kb+jwt"       : "Key Binding JWT",
}

def enum JWS_ALGORITHM: "JWS 서명 알고리즘"
{
    "ES256" : "ECDSA using P-256(Secp256r1) and SHA-256",
    "ES256K": "ECDSA using secp256k1 and SHA-256",
}

def enum SD_HASH_ALGORITHM: "Disclosure digest 해시 알고리즘"
{
    "sha-256",
}

def enum BINDING_METHOD: "Holder 바인딩 방식"
{
    "jwk": "cnf.jwk에 Holder 공개키 포함",
}

def enum CLAIM_VALUE_TYPE: "Claim 값 타입 (OpenDID 확장)"
{
    "string"   : "문자열",
    "full-date": "날짜 (YYYY-MM-DD)",
    "date"     : "날짜 (YYYY-MM-DD)",
    "uint"     : "0 이상의 정수",
    "integer"  : "정수",
    "boolean"  : "참거짓",
    "list"     : "JSON 배열",
    "jpeg"     : "JPEG 이미지 (base64 인코딩 문자열)",
}
```


### 2.4. Issuer-signed JWT

#### 2.4.1. JWT Header

```c#
def object IssuerJwtHeader: "Issuer-signed JWT header"
{
    + JWS_ALGORITHM "alg": "서명 알고리즘"
    + didKeyUrl     "kid": "Issuer 서명 키의 DID key URL"
    + JWT_TYPE      "typ": "JWT 타입", value("dc+sd-jwt-did")
}
```

- `alg`: Issuer 서명 키의 알고리즘에 따라 결정한다.
    - Secp256r1 키: "ES256"
    - Secp256k1 키: "ES256K"
    - 그 외 키 알고리즘으로는 발급할 수 없다.
    - OpenDID Issuer 서버는 Secp256r1 키로 서명하며, Credential Configuration의 `credential_signing_alg_values_supported`에 "ES256"을 공개한다.
- `kid`: Issuer DID Document의 `assertionMethod`에 등록된 `assert` 키를 가리키는 DID key URL이다. `versionId` query와 키 fragment를 포함한다.
    - 예: `did:omn:issuer?versionId=1#assert`
- `typ`: 항상 "dc+sd-jwt-did"이다.
- 위 3개 외의 Header 항목은 포함하지 않는다.

#### 2.4.2. JWT Payload

```c#
def object IssuerJwtPayload: "Issuer-signed JWT payload"
{
    //---------------------------------------------------------------------------
    // Issuer-signed Claim (항상 공개)
    //---------------------------------------------------------------------------
    + did              "iss"   : "Issuer DID"
    + numericDate      "iat"   : "발급 일시"
    + numericDate      "exp"   : "만료 일시"
    - numericDate      "nbf"   : "유효기간 시작 일시"
    + string           "vct"   : "Credential 타입"
    + Confirmation     "cnf"   : "Holder 바인딩 키"
    - CredentialStatus "status": "Credential 상태 참조"

    //---------------------------------------------------------------------------
    // Selective Disclosure
    //---------------------------------------------------------------------------
    - array(sdDigest)   "_sd"    : "선택적 공개 Claim의 Disclosure digest 목록", emptiable(false)
    - SD_HASH_ALGORITHM "_sd_alg": "digest 해시 알고리즘", default("sha-256")
}
```

- `iss`: `kid`에서 query와 fragment를 제외한 Issuer DID이다.
- `iat`, `exp`: `exp`는 `iat`로부터 365일 후로 설정한다.
- `nbf`: OpenDID는 현재 `nbf`를 포함하지 않는다. 포함된 경우 검증자는 유효기간 시작 일시를 확인한다.
- `vct`: Credential Configuration의 `vct` 값을 그대로 사용한다 ([3.2](#32-sd-jwt-vc와의-대응) 참조).
- `cnf`: [2.7](#27-holder-바인딩) 참조
- `status`: [2.8](#28-상태-참조) 참조. OpenDID Issuer 서버는 `dc+sd-jwt-did` Credential에 항상 포함한다.
- `_sd`: Subject Claim이 1개 이상이면 포함한다. 순서는 무작위이며, decoy digest는 추가하지 않는다.
- `_sd_alg`: OpenDID는 `_sd_alg`를 생략하며, 이 경우 digest 해시 알고리즘은 "sha-256"이다.
- `sub`, `jti`는 포함하지 않는다. Holder DID는 Credential에 포함하지 않으며, Holder는 `cnf`로 바인딩된다.

#### 2.4.3. 서명

Issuer-signed JWT는 `[RFC7515]`의 JWS Compact Serialization으로 서명한다.

1. Header와 Payload를 각각 JSON으로 직렬화한 후 base64url로 인코딩하여 `.`으로 연결한다(signing input).
1. signing input의 SHA-256 해시를 Issuer의 `assert` 키로 ECDSA 서명한다.
1. 서명값은 `[RFC7518]`에 따라 R과 S를 연결한 64바이트 값을 base64url로 인코딩한다.

검증자는 `kid`의 DID로 신뢰저장소에서 Issuer DID Document를 조회하고, `kid`의 `versionId`와 fragment에 해당하는 공개키로 서명을 검증한다 (`[DID-DOC-FORMAT]` 참조).


### 2.5. Disclosure

Disclosure는 선택적 공개 Claim 1개를 나타내며, 아래 3개 항목을 가진 JSON 배열을 인코딩한 문자열이다.

| index | 항목        | 타입         | 설명                                                                   |
| :---: | ----------- | ------------ | ---------------------------------------------------------------------- |
|   0   | salt        | `salt`       | 16바이트 난수(SecureRandom)를 base64url로 인코딩한 22자 문자열         |
|   1   | Claim 이름  | `string`     | Subject Claim의 최상위 이름                                            |
|   2   | Claim 값    | JSON 값      | Claim 값. 객체나 배열인 경우 값 전체를 하나의 Disclosure에 포함한다.   |

Disclosure와 digest는 아래 순서로 생성한다.

1. JSON 배열 `[salt, Claim 이름, Claim 값]`을 공백 없이 직렬화한다.
1. 직렬화한 문자열을 UTF-8로 인코딩한 후 base64url로 인코딩한다. 이 값이 Disclosure이다.
1. Disclosure 문자열(ASCII)의 SHA-256 해시를 base64url로 인코딩한다. 이 값이 digest이며, Issuer-signed JWT의 `_sd`에 포함한다.

OpenDID는 Disclosure를 아래 규칙으로 구성한다.

- Subject Claim의 최상위 항목마다 Disclosure를 1개 만든다.
- 배열 요소 단위의 Disclosure(항목 2개인 배열)는 사용하지 않는다.
- 객체 Claim 내부 항목에 대한 Disclosure(재귀적 선택적 공개)는 사용하지 않는다.
- Compact 직렬화에서 Disclosure의 순서는 `_sd`의 순서와 무관하다.

검증자는 각 Disclosure의 digest를 계산하여 `_sd`에 포함되어 있는지 확인한 후, Claim 이름과 값을 Payload에 복원한다.


### 2.6. Claim의 공개 구분

| 구분                | Claim                                       | 설명                                                                     |
| ------------------- | ------------------------------------------- | ------------------------------------------------------------------------ |
| Issuer-signed Claim | `iss`, `iat`, `exp`, `vct`, `cnf`, `status` | Issuer-signed JWT에 그대로 포함되어 항상 공개된다. Disclosure를 만들지 않는다. |
| SD-JWT 구조 Claim   | `_sd`, `_sd_alg`                            | 선택적 공개를 위한 구조 정보                                             |
| 선택적 공개 Claim   | Subject Claim (예: `family_name`)           | Disclosure로 전달되며, Holder가 제출 시 공개 여부를 선택한다.            |

- 프로토콜 Claim인 `status`는 검증자가 Disclosure 없이 상태를 확인할 수 있도록 Issuer-signed Claim으로 포함한다.
- 같은 이름의 Claim을 Issuer-signed Claim과 선택적 공개 Claim에 동시에 포함할 수 없다.

#### 2.6.1. 예약 Claim 이름

아래 이름은 Issuer가 관리하는 Claim이므로 Subject Claim 이름으로 사용할 수 없다.

| Claim 이름                              | 용도                                              |
| --------------------------------------- | ------------------------------------------------- |
| `iss`, `iat`, `exp`, `nbf`, `vct`, `cnf` | Issuer-signed Claim                               |
| `status`                                | Credential 상태 참조                              |
| `_sd`, `_sd_alg`                        | SD-JWT 구조                                       |
| `sub`, `jti`                            | 사용하지 않음 (Holder는 `cnf`로 바인딩)           |
| `credentialStatus`, `credential_status` | 상태 참조와의 혼동 방지                           |

- Issuer는 Issuer-signed Claim에 예약 이름(`iss`, `iat`, `exp`, `nbf`, `vct`, `cnf`, `_sd`, `_sd_alg`)을 다시 지정할 수 없다.
- 사용자 주도 발급(Webview)에서는 `iss`, `sub`, `iat`, `nbf`, `exp`, `jti`, `vct`, `cnf`, `status`, `credentialStatus`, `credential_status` 이름의 Claim 정의를 입력 항목에서 제외한다.


### 2.7. Holder 바인딩

```c#
def object Confirmation: "Holder 바인딩 정보"
{
    + object "jwk": "Holder 공개키 (JWK)"
    {
        + string    "kty": "key type", value("EC")
        + string    "use": "key use", value("sig")
        + string    "crv": "curve", value("P-256")
        + base64url "x"  : "공개키 x 좌표"
        + base64url "y"  : "공개키 y 좌표"
    }
}
```

- `cnf.jwk`는 Credential 요청의 Proof JWT로 확인한 Holder 공개키이다 (`[OID4VCI-ISSUANCE]` 4.5 참조).
    - Proof JWT Header의 `kid`는 Holder DID URL이며, Issuer는 Holder DID Document의 `authentication`에 등록된 P-256 키를 사용한다.
    - Issuer는 이 키를 `[RFC7517]` 형식의 JWK로 다시 구성하여 `cnf.jwk`에 포함한다. `kid` 등 다른 JWK 항목은 포함하지 않는다.
- 하나의 Credential 요청에 Proof가 여러 개이면 Proof 키마다 Credential을 하나씩 발급하며, 각 Credential의 `cnf.jwk`는 해당 Proof 키이다.
- 검증자는 제출된 KB-JWT의 서명을 `cnf.jwk`로 검증하여 Holder 바인딩을 확인한다 ([2.9](#29-key-binding-jwt) 참조).


### 2.8. 상태 참조

```c#
def object CredentialStatus: "Credential 상태 참조"
{
    + object "status_list": "Token Status List 참조"
    {
        + int "idx": "Status List 내 index", min_value(0)
        + url "uri": "Status List URI"
    }
}
```

- `status`는 `[TOKEN-STATUS]`의 Referenced Token 형식을 따른다.
- `uri`는 `{Issuer 기본 URL}/status-lists/{listId}` 형식이며, Issuer는 발급 시 Credential 형식별 Status List와 index를 할당한다.
- `status`는 Issuer-signed Claim이다. Issuer는 Credential 생성 후 Issuer-signed JWT의 `status`가 할당한 `uri`, `idx`와 일치하는지 확인하며, 일치하지 않으면 발급하지 않는다.
- 상태 값(`VALID`, `INVALID`, `SUSPENDED`), 상태 전이, Status List JWT 형식은 [Credential Status format](Credential%20Status%20format_ko.md)과 `[OID4VCI-ISSUANCE]` 5장을 참조한다.
- OpenDID Issuer 서버는 `dc+sd-jwt-did` Credential에 항상 상태 참조를 포함하므로, Credential 요청의 Proof JWT는 Holder DID URL `kid`를 사용해야 한다.


### 2.9. Key Binding JWT

KB-JWT는 제출 시 Holder가 `cnf.jwk`에 대응하는 개인키로 서명하여 Credential 소유를 증명하는 JWT이다.
KB-JWT는 선택 항목이며, 검증자가 Holder 바인딩 확인을 요구하는 경우 포함해야 한다.
KB-JWT를 포함하지 않으면 제출 문자열은 마지막 `~`로 끝난다.

```c#
def object KbJwtHeader: "KB-JWT header"
{
    + JWS_ALGORITHM "alg": "서명 알고리즘", value("ES256")
    + JWT_TYPE      "typ": "JWT 타입", value("kb+jwt")
}

def object KbJwtPayload: "KB-JWT payload"
{
    + string      "aud"    : "검증자 식별자"
    + string      "nonce"  : "검증자가 제시한 nonce"
    + numericDate "iat"    : "KB-JWT 생성 일시"
    + base64url   "sd_hash": "제출한 SD-JWT의 해시"
}
```

- `sd_hash`: KB-JWT를 제외한 제출 문자열 `<Issuer-signed JWT>~<Disclosure i>~...~<Disclosure k>~`의 SHA-256 해시를 base64url로 인코딩한 값이다.
- OpenDID SD-JWT SDK의 검증자는 기본적으로 아래 사항을 확인한다.
    - `cnf.jwk`로 KB-JWT 서명 검증
    - `aud`, `nonce`가 검증자가 기대한 값과 일치
    - `sd_hash` 필수 및 일치
    - `iat`가 미래가 아니며, 생성 후 5분 이내 (허용 시각 오차 60초)


<div style="page-break-after: always;"></div>

## 3. Credential Configuration

### 3.1. Credential Configuration 구조

Credential Configuration은 발급할 Credential의 정의로서, Admin Console에서 등록하며
Issuer Metadata의 `credential_configurations_supported`에 Configuration ID를 키로 공개된다 (`[OID4VCI-ISSUANCE]` 3.1 참조).
SD-JWT VC의 Credential Configuration은 아래 구조를 가진다.

```c#
def object SdJwtVcCredentialConfiguration: "credential_configurations_supported 항목 (SD-JWT VC)"
{
    + CREDENTIAL_FORMAT     "format": "Credential format", value("dc+sd-jwt-did")
    + string                "vct"   : "Credential 타입"
    - string                "scope" : "OAuth 2.0 scope"
    - array(BINDING_METHOD) "cryptographic_binding_methods_supported": "Holder 바인딩 방식"
    - array(JWS_ALGORITHM)  "credential_signing_alg_values_supported": "Credential 서명 알고리즘"
    - object                "proof_types_supported": "Proof 유형별 지원 정보"
    {
        + object "jwt": "jwt Proof"
        {
            + array(JWS_ALGORITHM) "proof_signing_alg_values_supported": "Proof 서명 알고리즘"
        }
    }
    - object "credential_metadata": "Credential 표시 및 Claim 정보"
    {
        - array(CredentialDisplay) "display": "Credential 표시 정보"
        - array(ClaimDescription)  "claims" : "Claim 정의 목록"
    }
}

def object CredentialDisplay: "Credential 표시 정보"
{
    + string   "name"  : "Credential 이름"
    - LANGUAGE "locale": "언어 코드"
    - object   "logo"  : "로고"
    {
        + url    "uri"     : "로고 이미지 URL"
        - string "alt_text": "대체 텍스트"
    }
}

def object ClaimDescription: "Claim 정의"
{
    + array(string)    "path"      : "Claim 경로", min_count(1)
    - bool             "mandatory" : "필수 여부", default(false)
    - CLAIM_VALUE_TYPE "value_type": "값 타입 (OpenDID 확장)", default("string")
    - array(string)    "enum"      : "허용 값 목록 (OpenDID 확장)"
    - array(object)    "display"   : "Claim 표시 정보"
    {
        + string   "name"  : "Claim 표시 이름"
        - LANGUAGE "locale": "언어 코드"
    }
}
```

- `proof_types_supported`: OpenDID는 `jwt` Proof의 서명을 검증하며, Holder 키가 P-256이므로 `proof_signing_alg_values_supported`는 ["ES256"]이다.
- `value_type`, `enum`: `[OID4VCI]`에 정의되지 않은 OpenDID 확장 항목으로, 사용자 주도 발급(Webview)의 입력 검증과 값 변환에 사용한다.


### 3.2. SD-JWT VC와의 대응

| Credential Configuration                       | SD-JWT VC                    | 설명                                                                            |
| ---------------------------------------------- | ---------------------------- | ------------------------------------------------------------------------------- |
| `format`                                       | Header `typ`                 | "dc+sd-jwt-did"                                                                  |
| `vct`                                          | Payload `vct`                | 값을 그대로 사용한다. 값이 없거나 비어 있으면 발급하지 않는다.                  |
| `credential_signing_alg_values_supported`      | Header `alg`                 | Issuer 서명 키 알고리즘에 따른 `alg` 값을 공개한다.                             |
| `cryptographic_binding_methods_supported`      | `cnf`                        | "jwk": Holder 공개키를 `cnf.jwk`에 포함한다.                                     |
| `proof_types_supported.jwt`                    | `cnf.jwk`                    | Proof JWT로 확인한 Holder 공개키가 `cnf.jwk`가 된다.                            |
| `credential_metadata.claims[].path[0]`         | Disclosure의 Claim 이름      | Subject Claim의 최상위 이름                                                     |
| `credential_metadata.claims[].value_type`      | Disclosure의 Claim 값 타입   | 아래 표 참조                                                                    |
| `credential_metadata.claims[].mandatory`       | -                            | 사용자 주도 발급에서 입력 필수 여부                                             |
| `credential_metadata.claims[].display`         | -                            | 월렛의 Claim 표시 정보. Credential에는 포함하지 않는다.                         |
| `credential_metadata.display`                  | -                            | 월렛의 Credential 표시 정보. Credential에는 포함하지 않는다.                    |

- Issuer는 발급할 Credential 유형(Configuration ID 또는 `credential_identifiers`)에 해당하는 Credential Configuration을 찾아 `vct`를 가져온다.
- Subject Claim 값은 Issuer가 사용자별로 보관한 Claim 데이터이며, 사용자 주도 발급에서는 Webview 입력값이다.
- OpenDID 3.0은 Subject Claim의 최상위 이름 단위로 Disclosure를 만들므로, Claim 정의의 `path`는 최상위 Claim 이름 1개로 구성한다.

사용자 주도 발급에서 입력값은 `value_type`에 따라 아래와 같이 검증하고 변환하여 Disclosure의 Claim 값이 된다.

| `value_type`          | 입력 검증                              | Claim 값 (JSON) |
| --------------------- | -------------------------------------- | --------------- |
| "string" (기본값)     | `enum`이 있으면 목록 중 하나           | string          |
| "full-date", "date"   | YYYY-MM-DD 날짜                        | string          |
| "uint"                | 0 이상의 정수                          | number          |
| "integer"             | 정수                                   | number          |
| "boolean"             | "true" 또는 "false"                    | boolean         |
| "list"                | JSON 배열                              | array           |
| "jpeg"                | base64 인코딩된 JPEG 이미지 (최대 2MB) | string          |


### 3.3. Credential Configuration 예시

아래는 Configuration ID가 `VerifiableIdSD`이고 `vct`가 `org:omnione:pid:1`인 PID Credential Configuration의 예시이다.

```json
{
  "VerifiableIdSD": {
    "format": "dc+sd-jwt-did",
    "vct": "org:omnione:pid:1",
    "scope": "org.omnione.pid_vc_sd_jwt",
    "cryptographic_binding_methods_supported": ["jwk"],
    "credential_signing_alg_values_supported": ["ES256"],
    "proof_types_supported": {
      "jwt": {
        "proof_signing_alg_values_supported": ["ES256"]
      }
    },
    "credential_metadata": {
      "display": [
        {
          "name": "PID (SD-JWT VC)",
          "locale": "en",
          "logo": {
            "uri": "https://issuer.example.com/public/pid.png",
            "alt_text": "A square figure of a PID"
          }
        },
        { "name": "신분증명 (SD-JWT VC)", "locale": "ko" }
      ],
      "claims": [
        {
          "path": ["family_name"],
          "mandatory": true,
          "value_type": "string",
          "display": [
            { "name": "Family Name(s)", "locale": "en" },
            { "name": "성", "locale": "ko" }
          ]
        },
        {
          "path": ["given_name"],
          "mandatory": true,
          "value_type": "string",
          "display": [
            { "name": "Given Name(s)", "locale": "en" },
            { "name": "이름", "locale": "ko" }
          ]
        },
        {
          "path": ["birthdate"],
          "mandatory": true,
          "value_type": "full-date",
          "display": [
            { "name": "Birth Date", "locale": "en" },
            { "name": "생년월일", "locale": "ko" }
          ]
        },
        {
          "path": ["nationalities"],
          "mandatory": true,
          "value_type": "list",
          "display": [
            { "name": "Nationalities", "locale": "en" },
            { "name": "국적", "locale": "ko" }
          ]
        },
        {
          "path": ["email_address"],
          "mandatory": false,
          "value_type": "string",
          "display": [
            { "name": "Email Address", "locale": "en" },
            { "name": "이메일", "locale": "ko" }
          ]
        },
        {
          "path": ["issuing_authority"],
          "mandatory": true,
          "value_type": "string",
          "display": [
            { "name": "Issuance Authority", "locale": "en" },
            { "name": "발급 기관", "locale": "ko" }
          ]
        },
        {
          "path": ["issuing_country"],
          "mandatory": true,
          "value_type": "string",
          "display": [
            { "name": "Issuing Country", "locale": "en" },
            { "name": "발급 국가", "locale": "ko" }
          ]
        },
        {
          "path": ["date_of_issuance"],
          "mandatory": true,
          "value_type": "full-date",
          "display": [
            { "name": "Issuance Date", "locale": "en" },
            { "name": "발급일", "locale": "ko" }
          ]
        },
        {
          "path": ["date_of_expiry"],
          "mandatory": true,
          "value_type": "full-date",
          "display": [
            { "name": "Expiry Date", "locale": "en" },
            { "name": "만료일", "locale": "ko" }
          ]
        }
      ]
    }
  }
}
```


<div style="page-break-after: always;"></div>

## 4. SD-JWT VC 예시

아래는 [3.3](#33-credential-configuration-예시)의 Credential Configuration으로 발급한 SD-JWT VC의 예시이다.
선택 항목인 `email_address`는 입력되지 않아 Disclosure가 없다.

### 4.1. Issuer-signed JWT Header

```json
{
    "alg": "ES256",
    "kid": "did:omn:issuer?versionId=1#assert",
    "typ": "dc+sd-jwt-did"
}
```

### 4.2. Issuer-signed JWT Payload

- `iat`: 2026-10-22T01:00:00Z
- `exp`: 2027-10-22T01:00:00Z (`iat` + 365일)

```json
{
    "iss": "did:omn:issuer",
    "iat": 1792630800,
    "exp": 1824166800,
    "vct": "org:omnione:pid:1",
    "cnf": {
        "jwk": {
            "kty": "EC",
            "use": "sig",
            "crv": "P-256",
            "x": "N4QFeZ_n4e2uqgJ8PJPLcrTYlFltNiG1F7V5Lu27mxc",
            "y": "T92_uJkiTIxn-dLk2v8pwRFbnJPY98RT8ACK0hOVdWs"
        }
    },
    "status": {
        "status_list": {
            "idx": 42,
            "uri": "https://issuer.example.com/status-lists/1"
        }
    },
    "_sd": [
        "pIXxRoVX4B_Mfrwi9WzsgxmTsTGm47Oe-LIvi3ELVJ4",
        "jIYrxCNHmbkup3RXQHKEEnCRmB3q4fGn-CIStwmiX8E",
        "YX4yigTb4eJb5hb_6xjUKrUjwohvO0_I6xPnKQoxh98",
        "VNAb-e6XyjymxdoNID1MhzU5EwSpzH1CCeZCT3jkbq0",
        "GZs049wbu7k1MH8CqQ2gC8Rse3cKQfcGWXXNtgBn95c",
        "E6V5T0j3XZCXUBu23S4MAmCFxlLYtZM7ib7vEbp9Ouw",
        "nQ8h8dnZoLCp2m6EsTGB5GBF3mH-MbzDkrIaWXsiCq8",
        "nJ8WDdR2TObrAM_QMmTgkwTDqQQWus91kog93UKaSiE"
    ]
}
```

### 4.3. Disclosure

`family_name` Claim의 Disclosure는 아래와 같이 생성한다.

- JSON 배열

    ```json
    ["rlqUSNosDPHSPTo8ALbTiw","family_name","Kim"]
    ```

- Disclosure (base64url)

    ```
    WyJybHFVU05vc0RQSFNQVG84QUxiVGl3IiwiZmFtaWx5X25hbWUiLCJLaW0iXQ
    ```

- digest (base64url(SHA-256(Disclosure))): `_sd`의 6번째 항목

    ```
    E6V5T0j3XZCXUBu23S4MAmCFxlLYtZM7ib7vEbp9Ouw
    ```

전체 Disclosure는 아래와 같다.

| Claim 이름          | JSON 배열                                                                              | digest                                        |
| ------------------- | -------------------------------------------------------------------------------------- | --------------------------------------------- |
| `family_name`       | `["rlqUSNosDPHSPTo8ALbTiw","family_name","Kim"]`                                       | `E6V5T0j3XZCXUBu23S4MAmCFxlLYtZM7ib7vEbp9Ouw` |
| `given_name`        | `["IQ8yMU6js9Sonxu9McEmgg","given_name","Raon"]`                                       | `pIXxRoVX4B_Mfrwi9WzsgxmTsTGm47Oe-LIvi3ELVJ4` |
| `birthdate`         | `["HTeXcwGWHV4klVl5Dg_V9g","birthdate","1990-01-01"]`                                  | `GZs049wbu7k1MH8CqQ2gC8Rse3cKQfcGWXXNtgBn95c` |
| `nationalities`     | `["JSV1EOobkDmD64NVGNctbw","nationalities",["KR"]]`                                    | `VNAb-e6XyjymxdoNID1MhzU5EwSpzH1CCeZCT3jkbq0` |
| `issuing_authority` | `["ncJq9BZtdLXDxwsg2mLshw","issuing_authority","Ministry of the Interior and Safety"]` | `jIYrxCNHmbkup3RXQHKEEnCRmB3q4fGn-CIStwmiX8E` |
| `issuing_country`   | `["NcEO3hJeGEyBMQEZE2M4kQ","issuing_country","KR"]`                                    | `nJ8WDdR2TObrAM_QMmTgkwTDqQQWus91kog93UKaSiE` |
| `date_of_issuance`  | `["_nx6oBO9iNvCD10KxeOX8g","date_of_issuance","2026-10-22"]`                           | `YX4yigTb4eJb5hb_6xjUKrUjwohvO0_I6xPnKQoxh98` |
| `date_of_expiry`    | `["_WABunoJPOIHwbUpTduEgA","date_of_expiry","2036-10-21"]`                             | `nQ8h8dnZoLCp2m6EsTGB5GBF3mH-MbzDkrIaWXsiCq8` |

### 4.4. Compact 직렬화

발급된 SD-JWT VC의 Compact 직렬화 형식은 아래와 같다.
가독성을 위해 줄을 나누고 일부를 `...`으로 생략하였으며, 실제 값에는 줄바꿈이 없다.

```
eyJhbGciOiJFUzI1NiIsImtpZCI6ImRpZDpvbW46aXNzdWVyP3ZlcnNpb25JZD0xI2Fzc2VydCIsInR5cCI6ImRjK3NkLWp3dC1kaWQifQ
.eyJpc3MiOiJkaWQ6b21uOmlzc3VlciIsImlhdCI6MTc5MjYzMDgwMCwiZXhwIjoxODI0MTY2ODAwLCJ2Y3QiOiJvcmc6b21uaW9uZTpwaWQ6MSIs...
.<Issuer 서명>
~WyJybHFVU05vc0RQSFNQVG84QUxiVGl3IiwiZmFtaWx5X25hbWUiLCJLaW0iXQ
~WyJJUTh5TVU2anM5U29ueHU5TWNFbWdnIiwiZ2l2ZW5fbmFtZSIsIlJhb24iXQ
~WyJIVGVYY3dHV0hWNGtsVmw1RGdfVjlnIiwiYmlydGhkYXRlIiwiMTk5MC0wMS0wMSJd
~WyJKU1YxRU9vYmtEbUQ2NE5WR05jdGJ3IiwibmF0aW9uYWxpdGllcyIsWyJLUiJdXQ
~WyJuY0pxOUJadGRMWER4d3NnMm1Mc2h3IiwiaXNzdWluZ19hdXRob3JpdHkiLCJNaW5pc3RyeSBvZiB0aGUgSW50ZXJpb3IgYW5kIFNhZmV0eSJd
~WyJOY0VPM2hKZUdFeUJNUUVaRTJNNGtRIiwiaXNzdWluZ19jb3VudHJ5IiwiS1IiXQ
~WyJfbng2b0JPOWlOdkNEMTBLeGVPWDhnIiwiZGF0ZV9vZl9pc3N1YW5jZSIsIjIwMjYtMTAtMjIiXQ
~WyJfV0FCdW5vSlBPSUh3YlVwVGR1RWdBIiwiZGF0ZV9vZl9leHBpcnkiLCIyMDM2LTEwLTIxIl0
~
```

### 4.5. Credential 응답

Credential Issuer는 Compact 직렬화 문자열을 Credential 응답의 `credentials[].credential`로 전달한다 (`[OID4VCI]` 참조).

```json
{
  "credentials": [
    {
      "credential": "eyJhbGciOiJFUzI1NiIsImtpZCI6ImRpZDpvbW46aXNzdWVyP3ZlcnNpb25JZD0xI2Fzc2VydCIsInR5cCI6ImRjK3NkLWp3dC1kaWQifQ.eyJpc3Mi...~WyJybHFVU05v...~...~"
    }
  ]
}
```

### 4.6. 제출 형식

아래는 Holder가 `family_name`, `given_name`, `birthdate`만 공개하고 KB-JWT를 붙인 제출 형식의 예시이다.

```
<Issuer-signed JWT>
~WyJybHFVU05vc0RQSFNQVG84QUxiVGl3IiwiZmFtaWx5X25hbWUiLCJLaW0iXQ
~WyJJUTh5TVU2anM5U29ueHU5TWNFbWdnIiwiZ2l2ZW5fbmFtZSIsIlJhb24iXQ
~WyJIVGVYY3dHV0hWNGtsVmw1RGdfVjlnIiwiYmlydGhkYXRlIiwiMTk5MC0wMS0wMSJd
~<KB-JWT>
```

- KB-JWT Header

    ```json
    {
        "alg": "ES256",
        "typ": "kb+jwt"
    }
    ```

- KB-JWT Payload

    ```json
    {
        "aud": "https://verifier.example.com",
        "nonce": "n-0S6_WzA2Mj",
        "iat": 1792717200,
        "sd_hash": "cIHtoFLSf6s0EnPehFj6QAJHBUf87QgkQSjcdX6m9dE"
    }
    ```

- `sd_hash`는 `<Issuer-signed JWT>~<family_name Disclosure>~<given_name Disclosure>~<birthdate Disclosure>~`의 SHA-256 해시이다.
