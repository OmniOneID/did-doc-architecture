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

- 주제
    - Profile 구조 정의
- 작성: 오픈소스개발팀
- 일자: 2025-05-09
- 버전: v2.0.0

개정이력
---

| 버전   | 일자       | 변경 내용                                           |
| ------ | ---------- | --------------------------------------------------- |
| v1.0.0 | 2024-09-03 | 초안                                                |
| v2.0.0 | 2025-05-09 | ZKP 데이터 추가                                        |


<div style="page-break-after: always;"></div>

목차
---

<!-- TOC tocDepth:2..4 chapterDepth:2..6 -->

- [1. 개요](#1-개요)
    - [1.1. 목적](#11-목적)
    - [1.2. 참조문서](#12-참조문서)
- [2. 공통사항](#2-공통사항)
    - [2.1. 데이터 타입 및 상수](#21-데이터-타입-및-상수)
- [3. Issue Profile](#3-issue-profile)
    - [3.1. Issue Profile 구조](#31-issue-profile-구조)
        - [3.1.1. `IssueProfile` object](#311-issueprofile-object)
    - [3.2. Issue Profile 예시](#32-issue-profile-예시)
- [4. Verify Profile](#4-verify-profile)
    - [4.1. Verify Profile 구조](#41-verify-profile-구조)
        - [4.1.1. `VerifyProfile` object](#411-verifyprofile-object)
    - [4.2. Verify Profile 예시](#42-verify-profile-예시)
- [5. ProofRequest Profile](#5-proofrequest-profile)
    - [5.1. ProofRequest Profile 구조](#51-proofrequest-profile-구조)
        - [5.1.1. `ProofRequestProfile` object](#511-proofrequestprofile-object)
    - [5.2. ProofRequest Profile 예시](#52-proofrequest-profile-예시)    

<!-- /TOC -->


<div style="page-break-after: always;"></div>

## 1. 개요

Profile은 Issuer가 VC를 발급하거나 Verifier가 VP를 제출받으려고 할 때 **Holder에게 제공하는 요청정보**이며,
아래와 같이 세 가지 종류가 있다.

- Issue Profile: 발급 요청정보
- Verify Profile: 검증 요청정보
- ProofRequest Profile : ZKP 검증 요청정보

Profile을 Holder의 단말에 어떻게 전달하는지는 본 문서의 범위 밖이나, 아래와 같은 사례가 있음을 밝힌다.

- **사례 1**: Profile을 QR 코드로 인코딩하여 화면에 출력
- **사례 2**: Profile을 다른 형태의 데이터로 직렬화하여 PUSH 요청 메시지에 담아 PUSH 메시지 전송
- **사례 3**: Profile을 다운로드 할 수 있는 URL을 이메일, NFC, BLE 등 다양한 인터페이스로 전달


### 1.1. 목적

본 문서는 각종 Profile의 구조를 정의하고 Profile의 사용법에 대하여 설명함을 목적으로 한다.
앞에서 언급한 바와 같이 Profile을 엔티티 간에 송수신하는 방법은 본 문서에서 설명하지 않으므로
각 구현체의 요구사항에 맞게 별도로 구현하여야 한다.

### 1.2. 참조문서

| 참조명             | 문서명                                      | 위치 |
| ------------------ | ------------------------------------------- | ---- |
| [OSD]              | OpenDID Schema Definition Language          |      |
| [DATA-SPEC]        | (OpenDID) 데이터 명세서(Data Specification) |      |
| [DATA-SPEC]        | (OpenDID) ZKP 데이터 명세서(ZKP Data Specification) |      |
| [VC-SCHEMA-FORMAT] | (OpenDID) VC Schema format                  |      |

## 2. 공통사항

### 2.1. 데이터 타입 및 상수

여기에 정의되지 않은 항목은 `[DATA-SPEC]`을 참조한다.

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
    "IssueProfile" : "발급 요청정보",
    "VerifyProfile": "검증 요청정보",
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

def enum VERIFY_AUTH_TYPE: "제출용 인증수단"
{
    0x00000000: "인증수단 제한없음",
    0x00000001: "무인증",
    0x00000002: "PIN",
    0x00000004: "BIO",
    0x00000006: "PIN or BIO",
    0x00008006: "PIN and BIO,
}
```


<div style="page-break-after: always;"></div>

## 3. Issue Profile

Issue Profile은 Issuer가 발급하고자 하는 VC 및 발급 수행방법에 대한 정보를 Holder에게
전달하기 위한 데이터 묶음이며 구체적으로 아래와 같은 정보를 포함한다.

| 분류          | 내용                                                                                        |
| ------------- | ------------------------------------------------------------------------------------------- |
| Issuer 정보   | • 이름, DID, 참조 URL                                                                       |
| VC Schema     | • VC Schema URL 또는 VC Schema를 직접 포함<br>(Profile 용량에 제한이 없는 경우만 직접 포함) |
| Credential Offer| • ZKP credential을 포함한 VC를 발급받기 위한 정보<br>(ZKP 포함여부에 따라 미포함 될수 있음) |
| 발급 수행방법 | • 발급 요청 API URL<br>• 요청정보 암호화 관련 정보(수신자 공개키, nonce, 알고리즘 등)       |

### 3.1. Issue Profile 구조

#### 3.1.1. `IssueProfile` object

```c#
def object IssueProfile: "Issue Profile"
{
    //--------------------------------------------------------------------------
    // Profile Metadata
    //--------------------------------------------------------------------------
    + uuid         "id"         : "profile id"
    + PROFILE_TYPE "type"       : "profile type", value("IssueProfile)
    + string       "title"      : "profile 제목"
    - string       "description": "profile 설명", default(""), emptiable(true)
    - LogoImage    "logo"       : "발급에 대한 로고 이미지"
    + ENCODING     "encoding"   : "인코딩", default("UTF-8")
    + LANGUAGE     "language"   : "언어 코드"

    //--------------------------------------------------------------------------
    // Profile Contents
    //--------------------------------------------------------------------------
    + object "profile": "profile contents"
    {
        + ProviderDetail "issuer": "issuer 정보"

        + object "credentialSchema": "VC Schema 정보"
        {
            + url                    "id"   : "VC Schema URL"
            + CREDENTIAL_SCHEMA_TYPE "type" : "VC Schema format type"
            - multibase              "value": "VC Schema를 multibase로 인코딩한 값"
        }

        - object "credentialOffer": "Credential Offer 정보"
        {    
            + nonce "nonce"                      : "nonce"
            + schema-identifier "schemaId"       : "CredentialSchema 식별자"
            + definition-identifier "credDefId"  : "CredentialDefinition 식별자"
            + object "keyCorrectnessProof"       : "KeyCorrectnessProof"
            {
                + string "c"           : "hash"
                + string "xzCap"       : "xzCap"
                + object "xrCap"       : "xrCap", min_extend(1)
                {
                    /* 
                    ... 예시 (순서 보장 필요)
                    - string "zkpsex"
                    - string "zkpasort"
                    - string "zkpaddr"
                    - string "zkpbirth"
                    .
                    .
                    */
                    + string $attributeName : "각 attribute에 대한 커밋 값", min_extend(1)
                    + string "masterSecret" : "사용자가 소유한 master secret에 대한 커밋 값" 
                }
            }
        }

        + object "process": "발급 수행방법"
        {
            + array(url) "endpoints"  : "발급 API endpoint 목록"
            + ReqE2e     "reqE2e"     : "E2E 요청 정보(proof 없음)"
            + multibase  "issuerNonce": "issuer nonce", byte_length(16)
        }
    }

    //--------------------------------------------------------------------------
    // Proof
    //--------------------------------------------------------------------------
    + AssertProof "proof": "profile에 대한 issuer 서명"
}
```

- `~/profile/credentialSchema`: VC Schema 정보
    - `id`: VC Schema를 다운로드할 수 있는 URL
    - `value`: multibase 인코딩된 VC Schema (Base64 추천)
        - 클라이언트가 VC Schema를 다운로드할 수 없다고 판단되는 경우 값으로 전달
        - Issue Profile 용량이 커지므로 Profile 전달 방법(QR 코드 등)에 따라 주의 필요
- `~/profile/credentialOffer`: 발급자가 사용자에게 Credential을 발급해주기 전에 생성<br>
    (ZKP Data Specification #4.2. CredentialOffer 참조)
- `~/profile/process`
    - `endpoints`: 발급 API의 path 부분을 제외한 service endpoint 파트
    - `reqE2e`: E2E 암호화용 키교환 요청 정보
    - `issuerNonce`: 향후 발급요청정보 서명 시 `issuerNonce`를 반드시 포함해야 함

### 3.2. Issue Profile 예시

```json
{
    "id": "24a60adc-19f0-45c3-934b-fc9f350c7d4c",
    "type": "IssueProfile",
    "title": "우산대 학생증 발급",
    "description": "우산대학교 학생증을 발급합니다.",
    "encoding": "UTF-8",
    "language": "ko",   
    "profile": {
        "issuer": {
            "did": "did:example:woosanuniv",
            "certVcRef": "https://woosan.ac.kr/cert-vc/0528",
            "name": "우산대학교",
            "ref": "https://woosan.ac.kr"
        },
        "credentialSchema": {
            "id": "https://woosan.ac.kr/schema/student_id_v2.json",
            "type": "OsdSchemaCredential"
        },
        // ZKP 포함여부에 따라서 추가
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

Verify Profile은 Verifier가 제출받고자 하는 VC 및 VP 제출방법에 대한 정보를 Holder에게
전달하기 위한 데이터 묶음이며 구체적으로 아래와 같은 정보를 포함한다.

| 분류        | 내용                                                                                                             |
| ----------- | ---------------------------------------------------------------------------------------------------------------- |
| Verify 정보 | • 이름, DID, 참조 URL                                                                                            |
| ProofRequest     | • 전체 제출 여부<br>• 제출 가능한 VC Schema 및 Issuer 목록<br>• 사용자 화면에 보여줄 claim 목록<br>• 필수로 제출해야 하는 claim 목록 |
| VP 제출방법 | • 제출 API URL<br>• 제출정보 암호화 관련 정보(수신자 공개키, nonce, 알고리즘 등)<br>• 제출용 인증수단            |

VC Schema를 여러 개 지정한 경우 이것이 AND 조건인지 OR 조건인지는 본 문서의 범위 밖이다.
즉, Verifier와 Profile 수신자가 사전에 정의한 방법으로 처리한다.

### 4.1. Verify Profile 구조

#### 4.1.1. `VerifyProfile` object

```c#
def object VerifyProfile: "Verify Profile"
{
    //--------------------------------------------------------------------------
    // Profile Metadata
    //--------------------------------------------------------------------------
    + uuid         "id"         : "profile id"
    + PROFILE_TYPE "type"       : "profile type", value("VerifyProfile)
    + string       "title"      : "profile 제목"
    - string       "description": "profile 설명", default(""), emptiable(true)
    - LogoImage    "logo"       : "제출에 대한 로고 이미지"
    + ENCODING     "encoding"   : "인코딩", default("UTF-8")
    + LANGUAGE     "language"   : "언어 코드"

    //--------------------------------------------------------------------------
    // Profile Contents
    //--------------------------------------------------------------------------
    + object "profile": "profile contents"
    {
        + ProviderDetail "verifier": "verifier 정보"

        + object "filter": "제출할 VC 선택을 위한 필터"
        {
            + array(object) "credentialSchemas": "제출 가능한 VC Schema 별 claim 및 issuer"
            {
                // VC Schema information
                + url                    "id"        : "VC Schema URL"
                + CREDENTIAL_SCHEMA_TYPE "type"      : "VC Schema format type"
                - multibase              "value"     : "VC Schema를 multibase로 인코딩한 값"
                // Claim constraints
                - bool             "presentAll"    : "전체 claim 제출 여부", default(false)
                - array(claimCode) "displayClaims" : "화면 출력 claim 목록", emptiable(false)
                - array(claimCode) "requiredClaims": "필수 제출 claim 목록", emptiable(false)
                - array(did)       "allowedIssuers": "허용된 issuer의 did 목록", emptiable(false)
            }
        }

        + object "process": "VP 제출방법"
        {
            - array(url)       "endpoints"    : "제출 API endpoint 목록"
            + ReqE2e           "reqE2e"       : "E2E 요청 정보(proof 없음)"
            + multibase        "verifierNonce": "verifier nonce", byte_length(16)
            - VERIFY_AUTH_TYPE "authType"     : "제출용 인증수단"
        }
    }

    //--------------------------------------------------------------------------
    // Proof
    //--------------------------------------------------------------------------
    + AssertProof "proof": "profile에 대한 verifier 서명"
}
```

- `~/profile/filter/credentialSchemas[]`
    - `presentAll`: 전체 claim 제출 여부
        - true: 전체 제출
            - `displayClaims` 값은 무시되고 전체 claim이 화면에 보여야 함
            - `requiredClaims` 값은 무시되고 전체 claim이 필수 제출이어야 함
        - false: 부분 제출 허용
    - `displayClaims`
        - 선택/필수 여부와 무관하게 사용자 화면에 보여줄 claim 목록
        - `requiredClaims`에 포함된 항목일 경우 선택 해제를 하지 못하도록 해야 함
        - `displayClaims`에 없으나 `requiredClaims`에는 있는 항목의 제출 여부는 TAS의 정책으로 결정
        - 미지정의 경우 모든 claim을 화면에 표시
    - `requiredClaims`
        - 반드시 제출해야 하는 claim 목록
        - 미지정의 경우 필수 제출 claim 없음
    - `allowedIssuers`
        - 허용된 Issuer의 DID 목록
        - Holder가 보유한 VC가 Issuer 목록에 포함될 때만 제출 가능
        - 지정하지 않으면 해당 VC Schema로 발급된 모든 VC 제출 가능
- `~/profile/process`
    - `endpoints`: 제출 API의 path 부분을 제외한 service endpoint 파트
    - `reqE2e`: E2E 암호화용 키교환 요청 정보
    - `verifierNonce`: 향후 VP 서명 시 `verifierNonce`를 반드시 포함해야 함
    - `authType`: VP 제출 시 사용자 인증에 대한 조건

### 4.2. Verify Profile 예시

```json
{
    "id": "d1f26925-6743-4609-9932-e909dda0299f",
    "type": "VerifyProfile",
    "title": "MyShopping 학생증 확인",
    "description": "개인식별자 제출 미동의 시 할인이 불가합니다.",
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

ProofRequest Profile은 Verifier가 ZKP(Zero-Knowledge Proof) 기반의 검증을 수행하기 위해 Holder에게 요구하는 증명 요청 정보를 구조화하여 전달하는 데이터 묶음이다. 이 프로파일은 ZKP Proof를 생성하기 위한 조건들을 명시하며, 구체적으로 아래와 같은 정보를 포함한다.


| 분류        | 내용                                                                                                             |
| ----------- | ---------------------------------------------------------------------------------------------------------------- |
| Verify 정보 | • 이름, DID, 참조 URL                                                                                            |
| ProofRequest      | • 증명 요청의 이름<br>• Holder가 속성 값을 직접 증명해야 하는 항목들<br>• Holder가 조건(≥, ≤ 등)만 만족시키면 되는 항목들 |
| ZKP Proof 제출방법 | • 제출정보 암호화 관련 정보(수신자 공개키, nonce, 알고리즘 등)           |


### 5.1. ProofRequest Profile 구조

#### 5.1.1. `ProofRequestProfile` object

```c#
def object ProofRequestProfile: "ProofRequest Profile"
{
    //--------------------------------------------------------------------------
    // Profile Metadata
    //--------------------------------------------------------------------------
    + uuid         "id"         : "profile id"
    + PROFILE_TYPE "type"       : "profile type", value("VerifyProfile)
    + string       "title"      : "profile 제목"
    - string       "description": "profile 설명", default(""), emptiable(true)
    - LogoImage    "logo"       : "제출에 대한 로고 이미지"
    + ENCODING     "encoding"   : "인코딩", default("UTF-8")
    + LANGUAGE     "language"   : "언어 코드"

    //--------------------------------------------------------------------------
    // Profile Contents
    //--------------------------------------------------------------------------
    + object "profile": "profile contents"
    {
        + ProviderDetail "verifier": "verifier 정보"

        + object "proofRequest" : "ProofRequest 정보" 
        {
            + string "name"                  : "proofs 대상 이름"
            + string "version"               : "version"
            + nonce  "nonce"                 : "nonce"
            - object "requestedAttributes"   : "AttributeInfo", min_extend(1)
            - object "requestedPredicates"   : "PredicateInfo", min_extend(1)
        }

        + ReqE2e           "reqE2e"       : "E2E 요청 정보(proof 없음)"
    }

    //--------------------------------------------------------------------------
    // Proof
    //--------------------------------------------------------------------------
    + AssertProof "proof": "profile에 대한 verifier 서명"
}
```

- `~/profile/proofRequest`: 검증자가 증명자(사용자)에게 요구하는 증명 조건을 정의 
    (ZKP Data Specification #4.5. ProofRequest 참조)

### 5.2. ProofRequest Profile 예시

```json
{
    "id": "d1f26925-6743-4609-9932-e909dda0299f",
    "type": "VerifyProfile",
    "title": "MyShopping 학생증 확인",
    "description": "개인식별자 제출 미동의 시 할인이 불가합니다.",
    "encoding": "UTF-8",
    "language": "ko",   
    "profile": {
        "verifier": {
            "did": "did:example:myshopping",
            "certVcRef": "https://myshopping.com/cert-vc/1",
            "name": "MyShpping",
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