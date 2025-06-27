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

Proof Request format
==

- 주제
    - ZKP Proof Request 구조 정의
- 작성: 박동준
- 일자: 2025-04-30
- 버전: v1.0.0

개정이력
---

| 버전   | 일자       | 변경 내용 |
| ------ | ---------- | --------- |
| v1.0.0 | 2025-04-30 | 초안      |


<div style="page-break-after: always;"></div>

---

<!-- TOC tocDepth:2..4 chapterDepth:2..6 -->

- [Proof Request format](#proof-request-format)
  - [개정이력](#개정이력)
  - [1. 개요](#1-개요)
    - [1.1. 참조문서](#11-참조문서)
  - [2. Proof Request 구조](#2-proof-request-구조)
    - [2.1. 데이터 타입 및 상수](#21-데이터-타입-및-상수)
    - [2.2. `ProofRequest` object](#22-proofrequest-object)
  - [3. 예시](#3-예시)
    - [3.1. 운전면허증 ProofRequest](#31-운전면허증-proofrequest)

<!-- /TOC -->

<div style="page-break-after: always;"></div>

## 1. 개요
본 문서는 OpenDID ZKP에서 사용하는 Proof Request 구조와 관련 정보를 정의한다.

검증자(Verifier)가 피증명인(Holder)에게 특정 정보를 증명하도록 요청할 때 사용되는 Proof Request입니다.
Proof Request는 영지식 증명(ZKP) 방식을 활용하여 원본 데이터를 직접 공개하지 않고도 특정 속성 또는 조건을 만족함을 증명할 수 있도록 합니다.

본 문서는 Proof Request를 정의하기 위한 문서 형식인 Proof Request format을 정의하는 것을 목적으로 한다.
format은 모두 OSD (OpenDID Schema Definition Language)로 정의한다.

### 1.1. 참조문서

| 참조명                   | 문서명                                           | 위치 |
| ---------------------- | ----------------------------------------------- | ---- |
| [OSD]                  | OpenDID Schema Definition Language              |      |
| [ZKP-DATA-SPEC]        | (OpenDID) 데이터 명세서(ZKP Data Specification)    |      |
| [PROOF-REQUEST-FORMAT] | (OpenDID) Proof Request format                  |      |


<div style="page-break-after: always;"></div>

## 2. Proof Request 구조
검증자가 증명자(사용자)에게 요구하는 증명 조건을 정의하는 구조입니다.

여기에 정의되지 않은 항목은 `[ZKP-DATA-SPEC]`을 참조한다.

### 2.1. 데이터 타입 및 상수

```c#
def string identifier : "id", issuerDID:2:schemaName:version
```

### 2.2. `ProofRequest` object
`ProofRequest`는 Proof Request object이다.

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

- `~/name`: Proof Request의 이름. (예: "mdl")
- `~/nonce`: 난수 값 (증명 요청의 고유성을 보장)
- `~/requestedAttributes`: 사용자가 공개해야 하는 속성 정보
  - `~/requestedAttributes/name`: 요청된 속성 이름 (예: "zkpsex" → 성별)
  - `~/requestedAttributes/restriction`: 특정 Credential Definition(credDefId)에서 발급된 증명서만 허용
- `~/requestedPredicates`: 사용자가 조건을 만족해야 하는 속성 정보
  - `~/requestedPredicates/name`: 요청된 속성 이름 (예: "zkpbirth" → 생년월일)
  - `~/requestedPredicates/pType`: 비교 연산자 ("LE": ≤ (작거나 같음), "GE": ≥, "EQ": =)
  - `~/requestedPredicates/pValue`: 비교할 기준 값 (예: 20200103 → 2020년 1월 3일)
  - `~/requestedPredicates/restrictions`: 특정 Credential Definition(credDefId)에서 발급된 증명서만 허용

<div style="page-break-after: always;"></div>

## 3. 예시

본 장에서는 ProofRequest 개념의 이해를 돕기 위해 구체적인 예를 들어 추가로 설명한다.
상황은 다음과 같다.
- 검증기관에서 사용자에게 특정 발급기관의 "Credential Definition ID"를 제안한다.
- 제안한 Credential은 운전면허증으로 Credential 내 어트리뷰트는 아래 목록과 같이 하기로 결정하였다.
    - 성별, 생년월일, 면허번호, 주소
    - 생년월일은 2020년 1월 3일 보다 작거나 같은 조건이다.
- 상기 어트리뷰트를 OpenDID ProofRequest format에 맞춰 JSON으로 작성한다.

이 ProofRequest는 검증자가 "특정 기관에서 발급한 Credential을 통해 사용자의 성별·주소·식별번호를 확인하고, 생년월일이 2020년 1월 3일 이전인지 확인하는" 증명을 요청하는 것입니다.
사용자는 이를 만족하는 Credential을 제시하고, 불필요한 정보 공개 없이 영지식증명 방식으로 증명할 수 있습니다.

### 3.1. 운전면허증 ProofRequest

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



