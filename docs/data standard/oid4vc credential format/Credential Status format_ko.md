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

Credential Status format
==

- 주제
    - OID4VCI로 발급한 Credential의 상태 관리 데이터 형식 정의
    - Credential 내 상태 참조, Status List Token, Credential 폐기 요청·응답 데이터 정의
- 작성: 오픈소스개발팀
- 일자: 2026-10-22
- 버전: v1.0.0


개정이력
---

| 버전   | 일자       | 변경 내용                                    |
| ------ | ---------- | -------------------------------------------- |
| v1.0.0 | 2026-10-22 | 초안                                         |


<div style="page-break-after: always;"></div>

목차
---

<!-- TOC tocDepth:2..4 chapterDepth:2..6 -->

- [1. 개요](#1-개요)
    - [1.1. 참조문서](#11-참조문서)
- [2. 공통 규정](#2-공통-규정)
    - [2.1. 적용 대상 Credential](#21-적용-대상-credential)
    - [2.2. 상태 값](#22-상태-값)
        - [2.2.1. 상태 값 정의](#221-상태-값-정의)
        - [2.2.2. 상태 전이](#222-상태-전이)
        - [2.2.3. 상태당 Bit 수](#223-상태당-bit-수)
    - [2.3. 데이터 타입 및 상수](#23-데이터-타입-및-상수)
- [3. Credential 내 상태 참조](#3-credential-내-상태-참조)
    - [3.1. `CredentialStatusRef` object](#31-credentialstatusref-object)
    - [3.2. Status List URI](#32-status-list-uri)
    - [3.3. Index](#33-index)
    - [3.4. SD-JWT VC](#34-sd-jwt-vc)
    - [3.5. ISO mdoc](#35-iso-mdoc)
- [4. Status List Token](#4-status-list-token)
    - [4.1. Status List 조회](#41-status-list-조회)
    - [4.2. JOSE Header](#42-jose-header)
    - [4.3. Payload](#43-payload)
    - [4.4. `lst` 인코딩](#44-lst-인코딩)
        - [4.4.1. 상태 Byte 배열](#441-상태-byte-배열)
        - [4.4.2. 압축 및 인코딩](#442-압축-및-인코딩)
    - [4.5. 상태 값 확인](#45-상태-값-확인)
    - [4.6. 예시](#46-예시)
- [5. Credential 폐기 (OpenDID 확장)](#5-credential-폐기-opendid-확장)
    - [5.1. 처리 흐름](#51-처리-흐름)
    - [5.2. c_nonce 응답](#52-c_nonce-응답)
    - [5.3. Revocation Token](#53-revocation-token)
        - [5.3.1. JOSE Header](#531-jose-header)
        - [5.3.2. Payload](#532-payload)
        - [5.3.3. 검증 규칙](#533-검증-규칙)
    - [5.4. 폐기 요청](#54-폐기-요청)
    - [5.5. 폐기 응답](#55-폐기-응답)
    - [5.6. 오류 응답](#56-오류-응답)
    - [5.7. 예시](#57-예시)

<!-- /TOC -->


<div style="page-break-after: always;"></div>

## 1. 개요

본 문서는 OpenDID 3.0에서 OID4VCI로 발급한 Credential(SD-JWT VC, ISO mdoc)의 상태 관리에 사용하는 데이터 형식을 정의한다.
상태 관리는 `[TOKEN-STATUS]`(Token Status List)를 준수하며, 월렛에 의한 Credential 폐기는 OpenDID의 OID4VCI 확장으로 정의한다.
상태 관리의 전반적인 개념과 절차는 `[OID4VCI-ISSUANCE]`의 5장을 참조한다.

본 문서에서 정의하는 데이터와 생성·사용 주체는 다음과 같다.

| 데이터                     | 생성 주체                     | 사용 주체               | 참조                                                 |
| -------------------------- | ----------------------------- | ----------------------- | ---------------------------------------------------- |
| Credential 내 상태 참조    | Credential Issuer (발급 시)   | 월렛, 검증자            | [3](#3-credential-내-상태-참조)                      |
| Status List Token          | Status List Endpoint          | 검증자, 월렛            | [4](#4-status-list-token)                            |
| Revocation Token, 폐기 요청 | 월렛                          | Credential Issuer       | [5](#5-credential-폐기-opendid-확장)                 |

아래 사항에 대해서는 본 문서에서 정의한 것을 준수하여야 한다.

- Credential 내 상태 참조의 구조와 위치
- 상태 값과 상태 전이 규칙
- Status List Token의 구조와 `lst` 인코딩 규칙
- Revocation Token과 Credential 폐기 요청·응답 형식

아래 사항은 _본 문서의 범위 밖_ 이므로 구현체에 따라 자유롭게 정의할 수 있다.

- Status List의 저장 및 운영 방식 (저장 구조, List 교체, 비활성화 등)
- 관리자에 의한 상태 변경 API
- 검증 사업자 서버의 Status List 확인 기능
- Credential 발급 절차 (`[OID4VCI-ISSUANCE]` 참조)


### 1.1. 참조문서

| 참조명             | 문서명                                                   | 위치                                                                                                    |
| ------------------ | -------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| [TOKEN-STATUS]     | Token Status List (draft-ietf-oauth-status-list)         | https://datatracker.ietf.org/doc/draft-ietf-oauth-status-list/                                          |
| [OID4VCI]          | OpenID for Verifiable Credential Issuance 1.0            | https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0.html                               |
| [SD-JWT-VC]        | SD-JWT-based Verifiable Credentials (SD-JWT VC)          | https://datatracker.ietf.org/doc/draft-ietf-oauth-sd-jwt-vc/                                            |
| [ISO-18013-5]      | ISO/IEC 18013-5 Mobile driving licence (mDL) application |                                                                                                         |
| [RFC7515]          | JSON Web Signature (JWS)                                 | https://www.rfc-editor.org/rfc/rfc7515                                                                  |
| [RFC7518]          | JSON Web Algorithms (JWA)                                | https://www.rfc-editor.org/rfc/rfc7518                                                                  |
| [RFC7519]          | JSON Web Token (JWT)                                     | https://www.rfc-editor.org/rfc/rfc7519                                                                  |
| [RFC1950]          | ZLIB Compressed Data Format Specification version 3.3    | https://www.rfc-editor.org/rfc/rfc1950                                                                  |
| [RFC1951]          | DEFLATE Compressed Data Format Specification version 1.3 | https://www.rfc-editor.org/rfc/rfc1951                                                                  |
| [RFC4648]          | The Base16, Base32, and Base64 Data Encodings            | https://www.rfc-editor.org/rfc/rfc4648                                                                  |
| [OSD]              | OpenDID Schema Definition Language                       | [OpenDID Schema Definition Language_ko.md](../../guide/osd/OpenDID%20Schema%20Definition%20Language_ko.md) |
| [DATA-SPEC]        | (OpenDID) 데이터 명세서(Data Specification)              | [Data Specification_ko.md](../data%20specification/Data%20Specification_ko.md)                          |
| [DID-DOC-FORMAT]   | (OpenDID) DID Document format                            | [DID Document format_ko.md](../did%20document%20format/DID%20Document%20format_ko.md)                   |
| [OID4VCI-ISSUANCE] | (OpenDID) OID4VCI 기반 VC 발급                           | [OID4VCI Issuance_ko.md](../../concepts/OID4VCI%20Issuance_ko.md)                                       |
| [P240]             | (OpenDID) P240 : OID4VCI Credential 폐기 Sequence        | [P240_oid4vci_revoke_credential_ko.puml](../../sequence/detailed/P240_oid4vci_revoke_credential_ko.puml) |
| [ISSUER-API]       | (OpenDID) Issuer API                                     | did-issuer-server `docs/api/Issuer_API_ko.md`                                                           |


<div style="page-break-after: always;"></div>

## 2. 공통 규정

### 2.1. 적용 대상 Credential

상태 관리는 Issuer DID 키로 서명하는 아래 형식의 Credential에 적용된다.

| Format          | Credential | 상태 참조 위치                              | 참조                       |
| --------------- | ---------- | ------------------------------------------- | -------------------------- |
| `dc+sd-jwt-did` | SD-JWT VC  | Issuer 서명 JWT Payload의 `status` Claim    | [3.4](#34-sd-jwt-vc)       |
| `mso_mdoc-did`  | ISO mdoc   | MSO(Mobile Security Object)의 `status` 항목 | [3.5](#35-iso-mdoc)        |

- 상태 관리가 적용되는 Credential을 발급하려면 Credential 요청의 Proof JWT가 `kid`(Holder DID URL)로 Holder 키를 식별해야 한다.
  Credential Issuer는 이 Holder DID를 발급 기록에 저장하며, 월렛의 폐기 요청을 검증할 때 사용한다([5](#5-credential-폐기-opendid-확장) 참조).
- Status List는 Credential 형식별로 따로 생성된다. 따라서 SD-JWT VC와 mdoc은 서로 다른 Status List를 참조한다.
- Credential 한 건은 하나의 Status List의 index 하나를 참조한다.


### 2.2. 상태 값

#### 2.2.1. 상태 값 정의

| 상태        | 값  | 설명                                                                     |
| ----------- | --- | ------------------------------------------------------------------------ |
| `VALID`     | 0   | • 유효<br>• Credential 발급 시 초기 상태                                  |
| `INVALID`   | 1   | • 폐기됨<br>• 최종 상태로, 다시 유효하게 할 수 없음                        |
| `SUSPENDED` | 2   | • 일시 정지됨<br>• `VALID`로 복귀 가능                                     |
| `RESERVED`  | 3   | • 예약 값<br>• 최종 상태이며, OpenDID 3.0의 상태 전이로는 이 값으로 변경할 수 없음 |

- `VALID`, `INVALID`, `SUSPENDED`의 값은 `[TOKEN-STATUS]`에서 정의한 Status Type 값과 같다.
- Status List에서 Credential에 할당되지 않은 index의 값은 0이다.
- 발급에 실패한 Credential에 할당된 index는 재사용하지 않으며, 해당 값은 0으로 남는다.

#### 2.2.2. 상태 전이

![Credential 상태 전이](../../concepts/images/oid4vci_state_credential_status_ko.svg)

| 현재 상태   | 변경 가능 상태          |
| ----------- | ----------------------- |
| `VALID`     | `SUSPENDED`, `INVALID`  |
| `SUSPENDED` | `VALID`, `INVALID`      |
| `INVALID`   | 없음 (최종 상태)        |
| `RESERVED`  | 없음 (최종 상태)        |

- 현재 상태와 같은 상태로의 변경 요청은 허용되며, 아무 변경도 일어나지 않는다.
- 새 상태 값은 해당 Status List의 `bits`로 표현할 수 있어야 한다([2.2.3](#223-상태당-bit-수) 참조).
- 상태가 변경되면 Status List의 해당 index 값이 바뀌고, 이전 상태·새 상태·사유·변경자·변경 시각이 이력으로 기록된다.
- 상태 변경 주체는 다음과 같다.
    - Issuer 관리자: Admin Console에서 변경하며 사유를 반드시 입력한다.
    - 월렛: [5](#5-credential-폐기-opendid-확장)의 폐기 요청으로 `INVALID`로만 변경할 수 있다. 사유는 `USER_REQUEST`, 변경자는 Holder DID로 기록된다.

#### 2.2.3. 상태당 Bit 수

Status List는 index 하나의 상태 값을 `bits`개의 Bit로 표현한다.

| `bits` | 표현 가능한 값 | 표현 가능한 상태                                 |
| ------ | -------------- | ------------------------------------------------ |
| 1      | 0 ~ 1          | `VALID`, `INVALID`                               |
| 2      | 0 ~ 3          | `VALID`, `INVALID`, `SUSPENDED`, `RESERVED`      |
| 4      | 0 ~ 15         | `VALID`, `INVALID`, `SUSPENDED`, `RESERVED`      |
| 8      | 0 ~ 255        | `VALID`, `INVALID`, `SUSPENDED`, `RESERVED`      |

- `bits`는 1, 2, 4, 8 중 하나이며, 기본값은 2이다.
- `bits`가 1인 Status List의 Credential은 `SUSPENDED`로 변경할 수 없다.
- Issuer 서버가 자동으로 생성하는 Status List의 기본 설정은 다음과 같다.

| 항목       | 기본값  | 설명                                |
| ---------- | ------- | ----------------------------------- |
| `bits`     | 2       | 상태당 Bit 수                       |
| `capacity` | 100,000 | Status List에 할당할 수 있는 index 수 |
| `ttl`      | 300     | Status List Token 캐시 유효 시간(초) |


### 2.3. 데이터 타입 및 상수

여기에 정의되지 않은 항목은 `[DATA-SPEC]`을 참조한다.

```c#
def string statusListUri  : "Status List URI (절대 URI)"
def int    statusListIndex: "Status List index", min_value(0)
def int    numericDate    : "NumericDate. 1970-01-01T00:00:00Z부터의 초 ([RFC7519])"
def string base64url      : "padding 없는 base64url 문자열 ([RFC4648] 5절)"
def string compactJws     : "JWS Compact Serialization 문자열 ([RFC7515])"

def enum CREDENTIAL_STATUS: "Status List에 기록하는 Credential 상태 값"
{
    0: "VALID",
    1: "INVALID",
    2: "SUSPENDED",
    3: "RESERVED",
}

def enum CREDENTIAL_STATUS_NAME: "Credential 상태 이름"
{
    "VALID"    : "유효 (0)",
    "INVALID"  : "폐기 (1)",
    "SUSPENDED": "일시 정지 (2)",
    "RESERVED" : "예약 (3)",
}

def enum STATUS_BITS: "상태당 Bit 수"
{
    1, 2, 4, 8
}

def enum STATUS_CREDENTIAL_FORMAT: "상태 관리가 적용되는 Credential 형식"
{
    "dc+sd-jwt-did": "SD-JWT VC",
    "mso_mdoc-did" : "ISO mdoc",
}
```


<div style="page-break-after: always;"></div>

## 3. Credential 내 상태 참조

### 3.1. `CredentialStatusRef` object

Credential은 자신의 상태가 기록된 Status List의 URI와 index를 상태 참조로 포함한다.

```c#
def object StatusListRef: "Token Status List 참조"
{
    + statusListUri   "uri": "Status List URI"
    + statusListIndex "idx": "Status List 내 index"
}

def object CredentialStatusRef: "Credential 상태 참조"
{
    + StatusListRef "status_list": "Token Status List 참조"
}
```

```json
{
    "status_list": {
        "uri": "https://issuer.example.com/status-lists/1",
        "idx": 7
    }
}
```

같은 구조가 다음 위치에서 사용된다.

| 사용 위치                          | 항목명              | 참조                                   |
| ---------------------------------- | ------------------- | -------------------------------------- |
| SD-JWT VC의 Issuer 서명 JWT Payload | `status`            | [3.4](#34-sd-jwt-vc)                   |
| mdoc의 MSO                         | `status`            | [3.5](#35-iso-mdoc)                    |
| Revocation Token Payload           | `credential_status` | [5.3.2](#532-payload)                  |
| Credential 폐기 응답               | `credential_status` | [5.5](#55-폐기-응답) (`status` 추가)   |


### 3.2. Status List URI

Status List URI(`status_list.uri`)는 다음 형식을 가진다.

```
{Issuer 기본 URL}/status-lists/{listId}
```

- `{Issuer 기본 URL}`: Issuer 서버의 기본 URL 설정(`issuer.base-url`)에서 끝의 `/`를 제거한 값
- `{listId}`: Status List 식별자 (Issuer 서버에서는 Status List의 숫자 ID)
- 해당 형식의 Status List가 없는 상태에서 첫 Credential을 발급하면 Status List가 자동으로 생성된다.
- Status List URI는 Status List를 생성할 때 확정되어 저장되며, 이후 설정이 바뀌어도 변하지 않는다. 발급된 Credential은 발급 당시의 URI를 계속 사용한다.
- Status List URI는 Status List Token의 `sub`와 같다([4.3](#43-payload) 참조).
- 월렛과 검증자는 Credential에 포함된 URI를 그대로 사용해야 하며, URI를 조합하거나 변경하지 않는다.
  또한 Status List URI에서 Credential 폐기 Endpoint URL을 유추하지 않는다([5.1](#51-처리-흐름) 참조).


### 3.3. Index

- Index(`status_list.idx`)는 0부터 시작하는 정수이며, 같은 Status List 안에서 발급 순서대로 1씩 증가하며 할당된다.
- Index는 Status List의 `capacity`보다 작다. 할당할 수 있는 Status List가 없으면 Credential 발급이 실패한다.
- `uri`와 `idx`의 조합은 발급된 Credential 한 건을 유일하게 식별한다.
- 한 번 할당된 index는 재사용하지 않는다. Credential 생성에 실패한 경우도 같다.
- 하나의 Credential 요청에 여러 Proof가 포함되어 Credential이 여러 건 발급되면, Credential마다 index를 따로 할당한다.
- Index는 SD-JWT VC에서는 JSON number, mdoc에서는 CBOR unsigned integer로 표현한다.


### 3.4. SD-JWT VC

`dc+sd-jwt-did` 형식의 SD-JWT VC는 Issuer 서명 JWT Payload의 최상위 Claim `status`에 `CredentialStatusRef`를 포함한다.

- `status`는 Selective Disclosure 대상이 아니다. Disclosure로 만들지 않고 `_sd` 배열에도 포함하지 않으므로,
  월렛이 어떤 Claim을 제출하든 `status`는 항상 Issuer 서명 Payload와 함께 검증자에게 전달된다.
- `status`는 Issuer 서명 범위에 포함되므로 월렛이나 제3자가 변경할 수 없다.
- Credential Issuer는 Credential을 생성한 뒤 서명된 Payload의 `status.status_list`가 할당한 `uri`, `idx`와 일치하는지 확인하고 발급을 완료한다.

```json
// Issuer 서명 JWT Header
{
    "alg": "ES256",
    "typ": "dc+sd-jwt-did",
    "kid": "did:omn:issuer?versionId=1#assert"
}
```

```json
// Issuer 서명 JWT Payload (일부 값 생략)
{
    "iss": "did:omn:issuer",
    "iat": 1792630800,
    "exp": 1824166800,
    "vct": "org:omnione:pid:1",
    "cnf": {
        "jwk": { "kty": "EC", "crv": "P-256", "x": "...", "y": "..." }
    },
    "_sd": ["...", "..."],
    "_sd_alg": "sha-256",
    "status": {
        "status_list": {
            "uri": "https://issuer.example.com/status-lists/1",
            "idx": 7
        }
    }
}
```


### 3.5. ISO mdoc

`mso_mdoc-did` 형식의 mdoc은 MSO(MobileSecurityObject)의 최상위 항목 `status`에 `CredentialStatusRef`를 CBOR map으로 포함한다.

- MSO는 IssuerAuth(COSE_Sign1)의 payload로서 Issuer DID 키로 서명되므로, `status`도 Issuer 서명 범위에 포함된다.
- `status`는 Name Space의 Data Element(IssuerSignedItem)가 아니므로 선택적 제출 대상이 아니다.
- CBOR map의 key는 text string이며, `uri`는 text string, `idx`는 unsigned integer이다.

```
/ MobileSecurityObject (CBOR diagnostic notation, 다른 항목 생략) /
{
    "docType": "org.omnione.pid.1",
    / version, digestAlgorithm, valueDigests, deviceKeyInfo, validityInfo 생략 /
    "status": {
        "status_list": {
            "uri": "https://issuer.example.com/status-lists/2",
            "idx": 7
        }
    }
}
```

> **참고**: SD-JWT VC와 mdoc은 형식별로 서로 다른 Status List를 사용하므로, 위 예시의 두 Credential은 `idx`가 같아도 서로 다른 Status List(`/status-lists/1`, `/status-lists/2`)를 참조한다.


<div style="page-break-after: always;"></div>

## 4. Status List Token

Status List Token은 Status List 하나의 상태 값 전체를 담아 Issuer DID 키로 서명한 JWT이다(`[TOKEN-STATUS]`).

### 4.1. Status List 조회

| 항목                 | 값                                                       |
| -------------------- | -------------------------------------------------------- |
| Method               | `GET`                                                    |
| URL                  | Credential의 `status_list.uri`                           |
| 인증                 | 없음 (Public)                                            |
| 응답 `Content-Type`  | `application/statuslist+jwt`                             |
| 응답 Body            | Status List Token (JWS Compact Serialization)            |
| 응답 `Cache-Control` | `max-age={ttl}, public`                                  |
| CORS                 | 모든 Origin의 교차 출처 요청 허용                        |

- 요청에 `Accept` 헤더를 포함하는 경우 `application/statuslist+jwt`(또는 `*/*`)를 허용해야 한다.
- Status List Token은 조회 요청마다 현재 상태로 새로 생성되고 서명된다. 따라서 `iat`은 응답 생성 시각이다.
- 더 이상 신규 index를 할당하지 않는(비활성화된) Status List도 계속 조회할 수 있다.
- 응답의 `Cache-Control`은 `max-age`를 토큰의 `ttl`과 같은 값(초)으로 설정한다.


### 4.2. JOSE Header

```c#
def object StatusListTokenHeader: "Status List Token JOSE Header"
{
    + string    "alg": "서명 알고리즘", value("ES256")
    + didKeyUrl "kid": "Status List 서명 키의 Issuer DID URL"
    + string    "typ": "토큰 타입", value("statuslist+jwt")
}
```

- `kid`: Credential을 발급한 Issuer의 DID URL(키 fragment 포함). 예: `did:omn:issuer?versionId=1#assert`
- 서명은 P-256 키를 사용하는 ES256(`[RFC7518]`)이며, 서명 값은 32 Byte `R`과 32 Byte `S`를 연결한 64 Byte 값이다(`[RFC7515]`).

```json
{
    "alg": "ES256",
    "kid": "did:omn:issuer?versionId=1#assert",
    "typ": "statuslist+jwt"
}
```


### 4.3. Payload

```c#
def object StatusListTokenPayload: "Status List Token Payload"
{
    + statusListUri "sub"        : "Status List URI"
    + numericDate   "iat"        : "토큰 생성 시각"
    + numericDate   "exp"        : "토큰 만료 시각 (iat + ttl)"
    + int           "ttl"        : "캐시 유효 시간(초)", min_value(1)
    + object        "status_list": "Status List"
    {
        + STATUS_BITS "bits": "상태당 Bit 수"
        + base64url   "lst" : "압축 후 인코딩한 상태 Byte 배열"
    }
}
```

- `sub`: Status List URI. Credential의 `status_list.uri`와 같다.
- `exp`: `iat + ttl`
- `ttl`: Status List Token을 캐시할 수 있는 시간(초). 기본값은 300이다.
- `status_list.bits`: Status List의 상태당 Bit 수 ([2.2.3](#223-상태당-bit-수) 참조)
- `status_list.lst`: [4.4](#44-lst-인코딩)에 따라 인코딩한 상태 값 배열
- Payload에는 `iss`를 포함하지 않는다. 서명자는 Header의 `kid`로 식별한다.

```json
{
    "sub": "https://issuer.example.com/status-lists/1",
    "iat": 1792630800,
    "exp": 1792631100,
    "ttl": 300,
    "status_list": {
        "bits": 2,
        "lst": "eNpTaQUAAM8Aqg"
    }
}
```


### 4.4. `lst` 인코딩

#### 4.4.1. 상태 Byte 배열

Status List의 상태 값은 다음 규칙으로 Byte 배열에 기록한다.

- Byte 배열의 길이는 `ceil(capacity × bits / 8)` Byte이다.
- index `i`의 상태 값은 Bit 위치 `i × bits`부터 `bits`개의 Bit에 기록한다.
    - Byte 위치: `floor(i × bits / 8)`
    - Byte 내 Bit 위치(shift): `(i × bits) mod 8`
- 하나의 Byte 안에서는 최하위 Bit(LSB)부터 낮은 index를 채운다. 즉, 낮은 index가 Byte의 하위 Bit에 위치한다.
- `bits`가 1, 2, 4, 8 중 하나이므로 하나의 상태 값이 두 Byte에 걸치지 않는다.
- 상태 값이 기록되지 않은 index는 0이다.

`bits`가 2인 경우 Byte와 index의 관계는 다음과 같다.

| Byte | Bit 7-6 | Bit 5-4 | Bit 3-2 | Bit 1-0 |
| ---- | ------- | ------- | ------- | ------- |
| 0    | idx 3   | idx 2   | idx 1   | idx 0   |
| 1    | idx 7   | idx 6   | idx 5   | idx 4   |
| ...  | ...     | ...     | ...     | ...     |

`bits`가 1인 경우에는 Byte 0의 Bit 0이 idx 0, Bit 7이 idx 7이며, Byte 1의 Bit 0이 idx 8이다.

#### 4.4.2. 압축 및 인코딩

1. 상태 Byte 배열을 ZLIB 형식(`[RFC1950]`)으로 압축한다. 압축 알고리즘은 DEFLATE(`[RFC1951]`)이며, 최고 압축 레벨(9)을 사용한다.
    - ZLIB Header와 Adler-32 Checksum을 포함한 ZLIB Stream이며, Raw DEFLATE가 아니다.
1. 압축 결과를 padding 없는 base64url(`[RFC4648]` 5절)로 인코딩한다.
1. 인코딩 결과를 `status_list.lst`에 설정한다.

> **참고**: 압축 레벨은 `lst` 문자열에만 영향을 주며 해제한 Byte 배열에는 영향을 주지 않는다. 검증자는 압축 레벨과 무관하게 ZLIB 해제를 수행한다.


### 4.5. 상태 값 확인

검증자(또는 월렛)는 다음 절차로 Credential의 상태를 확인한다.

1. Credential에서 상태 참조(`status.status_list`의 `uri`, `idx`)를 추출한다.
    - SD-JWT VC: Issuer 서명 JWT Payload의 `status`
    - mdoc: MSO의 `status`
1. `uri`로 Status List Token을 조회한다([4.1](#41-status-list-조회) 참조).
1. Header의 `typ`이 `statuslist+jwt`, `alg`가 `ES256`인지 확인하고, `kid`가 가리키는 Issuer DID Document의 키로 서명을 검증한다.
1. Payload의 `sub`가 Credential의 `uri`와 같은지, `exp`가 지나지 않았는지 확인한다.
1. `lst`를 base64url 디코딩한 뒤 ZLIB 해제하여 상태 Byte 배열을 얻는다.
1. `idx × bits`가 상태 Byte 배열의 Bit 길이(`Byte 수 × 8`)보다 작은지 확인한다.
1. 다음 식으로 상태 값을 구하고 [2.2.1](#221-상태-값-정의)에 따라 해석한다.

```python
raw   = zlib.decompress(base64url_decode(lst))
bit   = idx * bits
value = (raw[bit // 8] >> (bit % 8)) & ((1 << bits) - 1)
```

- 검증자는 `ttl`(또는 `Cache-Control`의 `max-age`) 동안 같은 URI의 Status List Token을 재사용할 수 있으며, `exp`가 지나면 다시 조회한다.
- 캐시된 Status List Token에는 최대 `ttl` 동안 이전 상태가 남아 있을 수 있다.


### 4.6. 예시

이해를 돕기 위해 `capacity`가 8, `bits`가 2인 Status List를 사용한다. 실제 Issuer 서버가 생성하는 Status List의 `capacity`는 100,000이다.

**■ 상태 값**

| idx | 0       | 1         | 2           | 3       | 4         | 5         | 6       | 7           |
| --- | ------- | --------- | ----------- | ------- | --------- | --------- | ------- | ----------- |
| 상태 | `VALID` | `INVALID` | `SUSPENDED` | `VALID` | `INVALID` | `INVALID` | `VALID` | `SUSPENDED` |
| 값  | 0       | 1         | 2           | 0       | 1         | 1         | 0       | 2           |

**■ 상태 Byte 배열** (길이 `ceil(8 × 2 / 8)` = 2 Byte)

| Byte | idx 3 (Bit 7-6) | idx 2 (Bit 5-4) | idx 1 (Bit 3-2) | idx 0 (Bit 1-0) | 값     |
| ---- | --------------- | --------------- | --------------- | --------------- | ------ |
| 0    | `00`            | `10`            | `01`            | `00`            | `0x24` |

| Byte | idx 7 (Bit 7-6) | idx 6 (Bit 5-4) | idx 5 (Bit 3-2) | idx 4 (Bit 1-0) | 값     |
| ---- | --------------- | --------------- | --------------- | --------------- | ------ |
| 1    | `10`            | `00`            | `01`            | `01`            | `0x85` |

**■ 압축 및 인코딩**

| 단계                   | 결과                                         |
| ---------------------- | -------------------------------------------- |
| 상태 Byte 배열         | `24 85`                                      |
| ZLIB 압축 (레벨 9)     | `78 DA 53 69 05 00 00 CF 00 AA`              |
| base64url (`lst`)      | `eNpTaQUAAM8Aqg`                             |

같은 상태 값을 `capacity`가 100,000인 Status List(25,000 Byte)에 기록하면 `lst`는 다음과 같다.

```
eNrtwQEBAAAIAqDGeLXfDhHIHwAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAwIQC3MsAqg
```

**■ 상태 값 확인**

- idx 7: `bit = 7 × 2 = 14` → Byte 1, shift 6 → `(0x85 >> 6) & 0x3 = 2` → `SUSPENDED`
- idx 5: `bit = 5 × 2 = 10` → Byte 1, shift 2 → `(0x85 >> 2) & 0x3 = 1` → `INVALID`

**■ Status List Token 응답**

```http
HTTP/1.1 200 OK
Content-Type: application/statuslist+jwt
Cache-Control: max-age=300, public

eyJhbGciOiJFUzI1NiIsImtpZCI6ImRpZDpvbW46aXNzdWVyP3ZlcnNpb25JZD0xI2Fzc2VydCIsInR5cCI6InN0YXR1c2xpc3Qrand0In0.eyJzdWIiOiJodHRwczovL2lzc3Vlci5leGFtcGxlLmNvbS9zdGF0dXMtbGlzdHMvMSIsImlhdCI6MTc5MjYzMDgwMCwiZXhwIjoxNzkyNjMxMTAwLCJ0dGwiOjMwMCwic3RhdHVzX2xpc3QiOnsiYml0cyI6MiwibHN0IjoiZU5wVGFRVUFBTThBcWcifX0.<signature>
```

Header와 Payload를 디코딩한 값은 [4.2](#42-jose-header), [4.3](#43-payload)의 예시와 같다. `<signature>`는 Issuer DID 키로 생성한 ES256 서명 값의 자리 표시이다.


<div style="page-break-after: always;"></div>

## 5. Credential 폐기 (OpenDID 확장)

월렛은 보유한 Credential(`dc+sd-jwt-did`, `mso_mdoc-did`) 한 건의 폐기를 Credential Issuer에 직접 요청할 수 있다.
이 기능은 OpenDID의 OID4VCI 확장이며, Access Token과 TAS를 사용하지 않고 Holder DID 서명으로 요청자를 확인한다.

| 항목                                                  | 구분              |
| ----------------------------------------------------- | ----------------- |
| Issuer Metadata 조회, `nonce_endpoint`와 `c_nonce`    | `[OID4VCI]` 표준  |
| Credential 내 상태 참조, Status List Token            | `[TOKEN-STATUS]`  |
| `credential_revocation_endpoint`                      | OpenDID 확장      |
| Revocation Token(`opendid-credential-revocation+jwt`)과 폐기 요청·응답 | OpenDID 확장      |

Revocation Token은 JWT(`[RFC7519]`)와 JWS(`[RFC7515]`)를 사용하지만, 표준 OID4VCI 폐기 프로토콜을 의미하지 않는다.

### 5.1. 처리 흐름

상세 절차는 `[P240]`을 참조한다.

| 단계 | 요청                                         | 응답                     | 데이터                                    |
| ---- | -------------------------------------------- | ------------------------ | ----------------------------------------- |
| 1    | `POST {nonce_endpoint}`                      | `c_nonce`                | [5.2](#52-c_nonce-응답)                   |
| 2    | (월렛) Revocation Token 생성 및 Holder DID 키로 서명 | -                  | [5.3](#53-revocation-token)               |
| 3    | `POST {credential_revocation_endpoint}`      | Credential 폐기 응답     | [5.4](#54-폐기-요청), [5.5](#55-폐기-응답) |
| 4    | (선택) `GET {status_list.uri}`               | Status List Token        | [4](#4-status-list-token)                 |

- 월렛은 Issuer Metadata의 `nonce_endpoint`와 `credential_revocation_endpoint` 값을 그대로 사용하며, 경로를 직접 조합하지 않는다.
  OpenDID Issuer 서버는 `credential_revocation_endpoint`를 `{credential_issuer}/credential-lifecycle/revoke`로 고정 생성한다.
- 폐기 대상은 Credential에 포함된 상태 참조(`status_list{uri, idx}`)로 식별한다.


### 5.2. c_nonce 응답

월렛은 `nonce_endpoint`에 Body 없이 `POST` 요청을 보내고 `c_nonce`를 받는다(`[OID4VCI]`).

```c#
def object NonceResponse: "c_nonce 응답"
{
    + base64url "c_nonce": "16 Byte 난수를 padding 없는 base64url로 인코딩한 값"
}
```

- `c_nonce`는 10분 동안 유효하며 한 번만 사용할 수 있다.
- 하나의 `c_nonce`는 Credential 발급 Proof와 Credential 폐기 요청을 통틀어 한 번만 소비된다.

```json
{
    "c_nonce": "8qvJQO7TjZlE5M0wL8OaGA"
}
```


### 5.3. Revocation Token

Revocation Token은 폐기할 Credential의 상태 참조와 `c_nonce`를 담아 Holder DID 키로 서명한 JWT이며, JWS Compact Serialization으로 전달한다.

- 서명 키는 Holder DID Document의 `authentication`에 등록된 P-256(secp256r1) 키이며, 서명 알고리즘은 ES256이다.
- JWS Signing Input은 `BASE64URL(Header) + "." + BASE64URL(Payload)`이며, 서명 값은 32 Byte `R`과 32 Byte `S`를 연결한 64 Byte 값이다(`[RFC7515]`).

#### 5.3.1. JOSE Header

```c#
def object RevocationTokenHeader: "Revocation Token JOSE Header"
{
    + string    "alg": "서명 알고리즘", value("ES256")
    + string    "typ": "토큰 타입", value("opendid-credential-revocation+jwt")
    + didKeyUrl "kid": "Holder DID의 authentication 키 URL"
}
```

- `alg`: `ES256`만 허용한다. `none`과 다른 알고리즘은 허용하지 않는다.
- `kid`: 키 fragment를 포함한 Holder DID URL. 예: `did:omn:holder?versionId=1#pin`
    - `kid`는 `did:`로 시작하고 `?` 또는 `#`를 포함해야 한다.
    - `kid`의 DID 부분(첫 `?` 앞, `?`가 없으면 `#` 앞)은 Payload의 `iss`와 같아야 한다.
    - `kid`가 가리키는 키는 Holder DID Document의 `authentication`에 포함되어야 한다. 키는 `#` 뒤의 fragment로 비교한다.

#### 5.3.2. Payload

```c#
def object RevocationTokenPayload: "Revocation Token Payload"
{
    + did                 "iss"              : "Holder DID"
    + numericDate         "iat"              : "토큰 생성 시각"
    + numericDate         "exp"              : "토큰 만료 시각"
    + string              "nonce"            : "nonce_endpoint가 반환한 c_nonce"
    + CredentialStatusRef "credential_status": "폐기할 Credential의 상태 참조"
}
```

- `iss`: 토큰을 서명한 Holder DID. Credential 발급 시 Proof `kid`로 확인되어 발급 기록에 저장된 Holder DID와 같아야 한다.
- `iat`, `exp`: `exp`는 `iat`보다 커야 하며, `iat` 이후 5분(300초) 이내여야 한다.
- `nonce`: `nonce_endpoint`가 반환한 `c_nonce` 값을 그대로 사용한다.
- `credential_status.status_list`: 폐기할 Credential에 포함된 `uri`, `idx`를 그대로 사용한다. Credential Issuer는 `uri` 문자열과 `idx`가 정확히 일치하는 발급 기록을 찾는다.
- `aud`, `sub`, `jti`, `nbf`는 사용하지 않으며, 그 밖의 Claim은 무시한다.
- 폐기 사유는 전달하지 않는다. Credential Issuer는 사유를 `USER_REQUEST`로 기록한다.

```json
{
    "iss": "did:omn:holder",
    "iat": 1792634400,
    "exp": 1792634700,
    "nonce": "8qvJQO7TjZlE5M0wL8OaGA",
    "credential_status": {
        "status_list": {
            "uri": "https://issuer.example.com/status-lists/1",
            "idx": 7
        }
    }
}
```

#### 5.3.3. 검증 규칙

Credential Issuer는 다음 순서로 폐기 요청을 검증하며, 실패하면 해당 오류 코드로 응답한다([5.6](#56-오류-응답) 참조).
시각 검증에는 60초의 Clock Skew를 허용한다.

| 순서 | 검증 항목                                                                                   | 오류 코드      |
| ---- | ------------------------------------------------------------------------------------------- | -------------- |
| 1    | `revocation_token`이 비어 있지 않고 세 부분으로 된 JWS Compact Serialization인지             | `SSRVISS00302` |
| 2    | Header `alg`가 `ES256`, `typ`이 `opendid-credential-revocation+jwt`인지                     | `SSRVISS00302` |
| 3    | `kid`와 `iss`가 있고, `kid`의 DID 부분이 `iss`와 같은지                                       | `SSRVISS00302` |
| 4    | `iat`, `exp`가 있고, `exp > iat`, `exp - iat ≤ 300`초인지                                    | `SSRVISS00302` |
| 5    | `iat`이 현재 시각 + 60초 이후가 아니고, `exp`가 현재 시각 - 60초 이전이 아닌지                 | `SSRVISS00302` |
| 6    | `nonce`, `credential_status.status_list.uri`가 있고, `idx`가 0 이상의 정수인지               | `SSRVISS00302` |
| 7    | `iss`의 DID Document 조회                                                                    | `SSRVISS00103` |
| 8    | DID Document의 `id`가 `iss`와 같고, `kid`가 `authentication`에 포함되며 해당 키가 있는지      | `SSRVISS00302` |
| 9    | 공개키 디코딩 및 ES256 서명 검증 (P-256 키가 아니면 `SSRVISS00302`)                           | `SSRVISS00413` |
| 10   | `uri` + `idx`에 해당하는 발급 기록이 있고 발급 완료(`ISSUED`) 상태인지                       | `SSRVISS00206` |
| 11   | 발급 기록의 Holder DID가 `iss`와 같은지                                                      | `SSRVISS00302` |
| 12   | `c_nonce`가 존재하고 만료·사용되지 않았는지 (확인 후 소비)                                   | `SSRVISS00210` |
| 13   | 상태를 `INVALID`로 변경                                                                      | `SSRVISS00011` |

- 현재 상태가 이미 `INVALID`이면 상태를 변경하지 않고 이력도 추가하지 않으며, 기존 상태 변경 시각으로 성공 응답한다. 이 경우에도 `c_nonce`는 소비된다.
- 그 외에는 `VALID` 또는 `SUSPENDED`에서 `INVALID`로 변경하고, 사유 `USER_REQUEST`, 변경자 Holder DID로 이력을 기록한다.
- `c_nonce` 소비와 상태 변경은 하나의 트랜잭션으로 처리한다. 상태 변경이 실패하면 `c_nonce` 소비도 취소된다.


### 5.4. 폐기 요청

| 항목           | 값                                                  |
| -------------- | --------------------------------------------------- |
| Method         | `POST`                                              |
| URL            | Issuer Metadata의 `credential_revocation_endpoint`  |
| 인증           | 없음 (Revocation Token의 Holder DID 서명)           |
| `Content-Type` | `application/json;charset=utf-8`                    |

```c#
def object CredentialRevocationRequest: "Credential 폐기 요청"
{
    + compactJws "revocation_token": "Revocation Token"
}
```


### 5.5. 폐기 응답

요청이 승인되면 HTTP 200과 함께 다음 응답을 반환한다.

```c#
def object CredentialRevocationResponse: "Credential 폐기 응답"
{
    + object "credential_status": "처리한 Credential 상태"
    {
        + StatusListRef          "status_list": "Status List 참조"
        + CREDENTIAL_STATUS_NAME "status"     : "변경된 상태", value("INVALID")
    }
    + utcDatetime "revoked_at": "상태 변경 시각"
}
```

- `credential_status.status_list`: 요청한 Revocation Token의 `uri`, `idx`
- `credential_status.status`: 항상 `INVALID`
- `revoked_at`: ISO 8601 형식의 UTC 시각이며, 소수점 이하 초가 포함될 수 있다. 이미 `INVALID`였던 Credential은 기존 상태 변경 시각이다.
- 응답을 받지 못한 월렛은 새 `c_nonce`를 받아 다시 요청한다. Credential이 이미 `INVALID`이면 같은 결과로 성공 응답한다.
- 폐기가 완료되면 Status List의 해당 index 값은 1(`INVALID`)이 된다.


### 5.6. 오류 응답

오류가 발생하면 다음 형식으로 응답한다.

```c#
def object ErrorResponse: "오류 응답"
{
    + errorCode "code"       : "오류 코드"
    + string    "description": "오류 설명"
}
```

| HTTP | 오류 코드      | 설명                                                                                                     |
| ---- | -------------- | -------------------------------------------------------------------------------------------------------- |
| 400  | `SSRVISS00206` | Status List URI와 index에 해당하는 발급 Credential이 없거나 발급 완료 상태가 아님                         |
| 400  | `SSRVISS00210` | `nonce`가 존재하지 않거나, 만료되었거나, 이미 사용됨                                                      |
| 400  | `SSRVISS00302` | `revocation_token`의 형식·Header·Claim·유효 시간 검증 실패, Holder DID 키 확인 실패 또는 Holder DID 불일치 |
| 400  | `SSRVISS00413` | JWT 서명 검증 또는 Holder 공개키 디코딩 실패                                                              |
| 500  | `SSRVISS00103` | Holder DID Document 조회 실패                                                                            |
| 500  | `SSRVISS00011` | Credential 상태 변경 처리 실패                                                                           |
| 500  | `SSRVISS00702` | Request Body를 읽을 수 없음                                                                              |

- Revocation Token에 `nonce` Claim 자체가 없으면 `SSRVISS00210`이 아니라 `SSRVISS00302`로 응답한다.
- 이미 `INVALID`인 Credential에 대한 요청은 오류가 아니라 성공으로 처리한다.


### 5.7. 예시

**■ c_nonce 요청 및 응답**

```http
POST /nonce HTTP/1.1
Host: issuer.example.com
Content-Length: 0
```

```http
HTTP/1.1 200 OK
Content-Type: application/json;charset=utf-8

{
    "c_nonce": "8qvJQO7TjZlE5M0wL8OaGA"
}
```

**■ Credential 폐기 요청**

[3.4](#34-sd-jwt-vc)의 SD-JWT VC(`/status-lists/1`, idx 7)를 폐기하는 예시이다.

```http
POST /credential-lifecycle/revoke HTTP/1.1
Host: issuer.example.com
Content-Type: application/json;charset=utf-8

{
    "revocation_token": "eyJhbGciOiJFUzI1NiIsInR5cCI6Im9wZW5kaWQtY3JlZGVudGlhbC1yZXZvY2F0aW9uK2p3dCIsImtpZCI6ImRpZDpvbW46aG9sZGVyP3ZlcnNpb25JZD0xI3BpbiJ9.eyJpc3MiOiJkaWQ6b21uOmhvbGRlciIsImlhdCI6MTc5MjYzNDQwMCwiZXhwIjoxNzkyNjM0NzAwLCJub25jZSI6IjhxdkpRTzdUalpsRTVNMHdMOE9hR0EiLCJjcmVkZW50aWFsX3N0YXR1cyI6eyJzdGF0dXNfbGlzdCI6eyJ1cmkiOiJodHRwczovL2lzc3Vlci5leGFtcGxlLmNvbS9zdGF0dXMtbGlzdHMvMSIsImlkeCI6N319fQ.<signature>"
}
```

```json
// revocation_token Header (디코딩)
{
    "alg": "ES256",
    "typ": "opendid-credential-revocation+jwt",
    "kid": "did:omn:holder?versionId=1#pin"
}
```

Payload를 디코딩한 값은 [5.3.2](#532-payload)의 예시와 같다. `<signature>`는 Holder DID 키로 생성한 ES256 서명 값의 자리 표시이다.

**■ Credential 폐기 응답**

```http
HTTP/1.1 200 OK
Content-Type: application/json;charset=utf-8

{
    "credential_status": {
        "status_list": {
            "uri": "https://issuer.example.com/status-lists/1",
            "idx": 7
        },
        "status": "INVALID"
    },
    "revoked_at": "2026-10-22T02:00:01.123456Z"
}
```

**■ 오류 응답**

```http
HTTP/1.1 400 Bad Request
Content-Type: application/json

{
    "code": "SSRVISS00210",
    "description": "The issuer nonce is not valid."
}
```

**■ 폐기 후 Status List**

[4.6](#46-예시)의 Status List에서 idx 7이 `SUSPENDED`(2)에서 `INVALID`(1)로 변경되면 Byte 1이 `0x85`에서 `0x45`로 바뀐다.

| 단계                   | 결과                                         |
| ---------------------- | -------------------------------------------- |
| 상태 Byte 배열         | `24 45`                                      |
| ZLIB 압축 (레벨 9)     | `78 DA 53 71 05 00 00 8F 00 6A`              |
| base64url (`lst`)      | `eNpTcQUAAI8Aag`                             |

- idx 7: `(0x45 >> 6) & 0x3 = 1` → `INVALID`
