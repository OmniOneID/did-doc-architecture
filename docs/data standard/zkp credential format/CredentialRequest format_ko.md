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

Credential Request format
==

- 주제
    - ZKP Credential Request 구조 정의
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

- [Credential Request format](#credential-request-format)
  - [개정이력](#개정이력)
  - [1. 개요](#1-개요)
    - [1.1. 참조문서](#11-참조문서)
  - [2. CredentialRequest 구조](#2-credentialrequest-구조)
    - [2.1. `CredentialRequest` object](#21-credentialrequest-object)
  - [3. 예시](#3-예시)
    - [3.1. 운전면허증 CredentialRequest](#31-운전면허증-credentialrequest)

<!-- /TOC -->


<div style="page-break-after: always;"></div>

## 1. 개요

본 문서는 OpenDID ZKP에서 사용하는 Credential Request 구조와 관련 정보를 정의한다.
Proof Request는 다음의 정보를 포함한다.

- Credential Request 제목 및 설명
- Credential Request
- 발급할 어트리뷰트


본 문서는 Credential Request를 정의하기 위한 문서 형식인 Credential Request format을 정의하는 것을 목적으로 한다.
format은 모두 OSD (OpenDID Schema Definition Language)로 정의한다.

### 1.1. 참조문서

| 참조명                        | 문서명                                          | 위치 |
| --------------------------- | ----------------------------------------------- | ---- |
| [OSD]                       | OpenDID Schema Definition Language              |      |
| [ZKP-DATA-SPEC]             | (OpenDID) 데이터 명세서(ZKP Data Specification) |      |
| [CREDENTIAL-REQUEST-FORMAT] | (OpenDID) Credential Request format             |      |


<div style="page-break-after: always;"></div>

## 2. CredentialRequest 구조

여기에 정의되지 않은 항목은 `[ZKP-DATA-SPEC]`을 참조한다.

### 2.1. `CredentialRequest` object

`CredentialRequest`는 Credential Request object이다.

```c#
def object CredentialRequest: "credential request"
{    
    + string    "proverDID"                                                : "prover DID"
    + definition-identifier "credDefId"                                    : "Credential Definition 식별자"
    + nonce    "nonce"                                                     : "nonce"
    + BlindedCredentialSecrets    "blindedMs"                              : "BlindedCredentialSecrets"
    + BlindedCredentialSecretsCorrectnessProof "blindedMsCorrectnessProof" : "BlindedCredentialSecretsCorrectnessProof"
}
```

- `~/proverDID`: Holder의 DID, 신원주체를 나타내는 고유 식별자
- `~/credDefId`: 요청하는 Credential의 Credential Definition ID. 발급 기관이 정의한 Credential Definition을 가리키며, 발급할 Credential의 속성과 서명 체계를 정의함.
- `~/nonce`: 요청의 유일성을 보장하기 위한 무작위 난수(Nonce). Credential Offer와 Credential Request 간의 보안성을 유지하는 역할.
- `~/blindedMs`: blinded master secret
- `~/blindedMsCorrectnessProof`: blinded master secret proof

<div style="page-break-after: always;"></div>

## 3. 예시

본 장에서는 CredentialRequest 개념의 이해를 돕기 위해 구체적인 예를 들어 추가로 설명한다.

- OpenDID Credential Request format에 맞춰 JSON으로 작성한다.

### 3.1. 운전면허증 CredentialRequest

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



