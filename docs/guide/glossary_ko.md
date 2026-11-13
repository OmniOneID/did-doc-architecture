Glossary
==

- 주제: OpenDID 문서 용어 모음
- 작성: 오픈소스개발팀
- 일자: 2026-11-12
- 버전: v1.0.0

개정이력
---

| 버전   | 일자       | 변경 내용 |
| ------ | ---------- | --------- |
| v1.0.0 | 2026-11-12 | 최초 작성 |

목차
---

- [1. 개요](#1-개요)
- [2. Wallet Token](#2-wallet-token)
- [3. Server Token](#3-server-token)
- [4. OID4VCI 발급](#4-oid4vci-발급)
- [5. OID4VP 제출](#5-oid4vp-제출)
- [6. mdoc 근접 제출](#6-mdoc-근접-제출)
- [7. SD-JWT VC](#7-sd-jwt-vc)

<div style="page-break-after: always;"></div>

## 1. 개요

이 문서는 OpenDID 문서의 용어 절에 정의된 용어를 모은 것이다.
용어는 정의된 문서별로 구분하며, 같은 용어라도 문서의 문맥에 따라 설명이 다를 수 있다.
각 절의 출처 문서에서 용어가 사용되는 문맥을 확인할 수 있다.

## 2. Wallet Token

출처: [Wallet Token](./../concepts/token_ko.md#wallet-token-용어)

| 용어 | 설명 |
| ---- | ---- |
| Wallet Token Seed | Wallet이 생성하여 App에 전달하는 초기 시드 값입니다. |
| Wallet Token Data | 인가앱 서비스가 생성하여 App에 전달하는 데이터로, WalletTokenSeed를 기반으로 생성됩니다. 이는 Wallet Token 생성을 위한 주요 데이터입니다. |
| Nonce | Wallet이 생성하는 일회용 값으로, WalletTokenData와 조합하여 Wallet Token을 생성하는 데 사용됩니다. |
| Wallet Token | Wallet에 접근할 수 있는 권한을 부여하는 토큰으로, App이 Wallet과 상호작용할 때 사용됩니다. |

## 3. Server Token

출처: [Server Token](./../concepts/token_ko.md#server-token-용어)

| 용어 | 설명 |
| ---- | ---- |
| Signed Wallet Info | Wallet이 인가앱에 전달하는 서명된 월렛 정보로서, TA가 Wallet을 검증하는데 사용됩니다. |
| Attested App Info | 인가앱 서비스가 인가앱에 전달하는 인증된 앱 정보로서, TA가 인가앱을 검증하는데 사용됩니다. |
| Server Token Seed | 인가앱이 생성하여 TA에 전달하는 초기 시드 값으로, ServerTokenData 생성을 위한 기본 값입니다. |
| Server Token Data | TA가 생성하여 인가앱에 전달하는 데이터로, ServerTokenSeed를 기반으로 생성됩니다. 이는 Server Token 생성을 위한 주요 데이터입니다. |
| IV | 초기화 벡터로, 암호화 과정에서 사용되는 값입니다. |
| Session Key | TA와 인가앱 간의 세션 키로, 서버토큰 발급 전에 이미 교환된 상태입니다. |
| Server Token | 서버에 접근할 수 있는 권한을 부여하는 토큰으로, 인가앱이 서버와 상호작용할 때 사용됩니다. |

## 4. OID4VCI 발급

출처: [OID4VCI 발급](./../concepts/OID4VCI%20Issuance_ko.md#12-용어)

| 용어 | 설명 |
| ---- | ---- |
| Credential Issuer | OID4VCI Credential을 발급하는 주체. OpenDID에서는 Issuer 서버가 수행한다. |
| Authorization Server | Access Token을 발급하는 OAuth 2.0 인가 서버. OpenDID Issuer 서버에 내장되어 있다. |
| Credential Offer | 발급을 개시하기 위해 Credential Issuer가 월렛에 전달하는 정보. 발급할 Credential과 인가 방식(grant)을 담는다. |
| Credential Configuration | 발급할 Credential의 형식, 식별자, 표시 정보 등을 정의한 설정. Issuer Metadata의 `credential_configurations_supported`로 공개된다. |
| Pre-Authorized Code | Credential Issuer가 사용자 인증을 미리 완료한 상태에서 발급하는 일회용 인가 코드 |
| tx_code | Pre-Authorized Code와 함께 사용하는 거래 코드(PIN). 별도 채널로 사용자에게 전달된다. |
| c_nonce | Credential 요청 Proof의 재사용을 막기 위해 Credential Issuer가 발급하는 일회용 nonce |
| Proof | 월렛이 Holder 키로 서명하여 키 소유를 증명하는 JWT |
| Token Status List | Credential 상태를 비트 배열로 공개하는 목록 ([TOKEN-STATUS]) |
| List Provider | OID4VCI Issuer 목록을 관리하고 월렛에 제공하는 사업자. OpenDID에서는 TAS가 수행한다. |

## 5. OID4VP 제출

출처: [OID4VP 제출](./../concepts/OID4VP%20Presentation_ko.md#12-용어)

| 용어 | 설명 |
| ---- | ---- |
| Verifier | VP를 요청하고 제출받아 검증하는 주체. OpenDID에서는 Verifier 서버가 수행한다. |
| Authorization Request | 검증자가 월렛에 제출을 요청하는 정보. 제출할 Credential과 응답 방식을 담는다. |
| Request Object | Authorization Request를 담아 검증자가 서명한 JWT |
| `request_uri` | 월렛이 Request Object를 조회하는 URI |
| DCQL | Digital Credentials Query Language. 제출할 Credential과 Claim을 지정하는 요청 언어 ([OID4VP]) |
| VP Token | 월렛이 검증자에게 제출하는 Credential 제출 데이터 |
| KB-JWT | Key Binding JWT. SD-JWT VC 제출 시 Holder 키로 서명하여 키 소유를 증명하는 JWT ([SD-JWT-VC]) |
| DeviceResponse | mdoc 제출 데이터 ([ISO-18013-5]) |
| `response_uri` | 월렛이 VP Token을 전송하는 검증자의 URI |

## 6. mdoc 근접 제출

출처: [mdoc 근접 제출](./../concepts/mdoc%20Proximity%20Presentation_ko.md#12-용어)

| 용어 | 설명 |
| ---- | ---- |
| mdoc | ISO/IEC 18013-5에 정의된 모바일 문서 형식의 Credential ([MDOC-FORMAT]) |
| mdoc 리더 | 근접 제출로 mdoc을 요청하고 검증하는 앱 |
| 소지자 | mdoc을 보관하고 제출하는 월렛 |
| Device Engagement | 소지자와 mdoc 리더가 연결 방식과 세션 키 교환 정보를 주고받는 단계 |
| Handover | NFC로 Device Engagement 정보를 전달하는 방식. static과 negotiated가 있다. |
| SessionTranscript | 세션을 식별하는 데이터. 세션 키 생성과 DeviceAuth 서명에 사용한다. |
| DeviceRequest | mdoc 리더가 요청하는 문서(docType)와 데이터 요소 |
| DeviceResponse | 소지자가 제출하는 mdoc 데이터 |
| issuerAuth | 발급자가 mdoc에 한 서명 |
| DeviceAuth | 소지자가 기기 키로 DeviceResponse를 인증한 값 |

## 7. SD-JWT VC

출처: [SD-JWT VC](./../data%20standard/oid4vc%20credential%20format/SD-JWT%20VC%20format_ko.md#12-용어)

| 용어 | 설명 |
| ---- | ---- |
| SD-JWT VC | Issuer-signed JWT와 Disclosure로 구성되어 Claim 단위의 선택적 공개를 지원하는 VC |
| Issuer-signed JWT | Issuer가 서명한 JWT. Issuer-signed Claim과 Disclosure의 digest 목록을 담는다. |
| Disclosure | 선택적 공개 Claim 1개의 salt, Claim 이름, Claim 값을 담은 문자열 |
| Digest | Disclosure의 해시값. Issuer-signed JWT의 `_sd`에 포함된다. |
| Issuer-signed Claim | Issuer-signed JWT에 그대로 포함되어 항상 공개되는 Claim. Holder가 제출 시 제외할 수 없다. |
| 선택적 공개 Claim | Disclosure로 전달되어 Holder가 제출 시 공개 여부를 선택하는 Claim |
| Holder 바인딩 | Credential을 Holder 키에 결합하는 것. `cnf` Claim에 Holder 공개키를 포함한다. |
| KB-JWT | Key Binding JWT. 제출 시 Holder가 바인딩 키로 서명하여 키 소유를 증명하는 JWT |
