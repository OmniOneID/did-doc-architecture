# OmniOne Open DID 강의 시리즈 : 실습 튜토리얼
> OmniOne Open DID를 직접 설치하고 실행해볼 수 있도록 실습 튜토리얼을 준비했습니다.<br>
> 이 시리즈는 다양한 환경에서 Open DID 서버를 구축하는 과정을 단계별로 안내합니다.

---

**목차**
- [1강. Open DID 실습 교육 개요](#1강-open-did-실습-교육-개요)
- [2-1강. Open DID 서버 설치 (Orchestrator)](#2-1강-open-did-서버-설치-orchestrator)
- [2-2강. Open DID 서버 설치 (Gradle)](#2-2강-open-did-서버-설치-gradle)
- [2-3강. Open DID 서버 설치 (IDE)](#2-3강-open-did-서버-설치-ide)
- [2-4강. Open DID 서버 설치 (Docker)](#2-4강-open-did-서버-설치-docker)
- [3강. Open DID 서버 등록 (Trust Registry)](#3강-open-did-서버-등록-trust-registry)
- [4강. Open DID App 설치](#4강-open-did-app-설치)
- [5강. 사용자 등록](#5강-사용자-등록)
- [6강. VC 발급 (Issuer-Initiated)](#6강-vc-발급-issuer-initiated)
- [7강. VC 발급 (User-Initiated)](#7강-vc-발급-user-initiated)
- [8강. VP 제출](#8강-vp-제출)
- [9강. ZKP Credential 제출](#9강-zkp-credential-제출)

<br>

### 1강. Open DID 실습 교육 개요
<iframe width="450" height="250" src="https://www.youtube.com/embed/00xW3fn0RBQ?si=pYFh0kstyYpVGnnH" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen style="padding-right: 100px;"></iframe>

OpenDID 실습 교육의 전체 흐름과 학습 목표를 소개합니다.

📚 **이번 강의에서 다루는 내용**
- OpenDID 실습 전체 흐름 이해
- User 등록 → VC 발급 → VP 제출 최종 결과 소개
- OpenDID 아키텍처 구성 요소 및 역할
- DID, VC, VP 등 핵심 용어 정리
- 설치 방식(Orchestrator/Gradle/IDE/Docker) 선택 안내

⚠️ 본 강의는 Open DID 릴리즈 버전 2.0.0.0 버전 기준입니다.

---

### 2-1강. Open DID 서버 설치 (Orchestrator)
<iframe width="450" height="250" src="https://www.youtube.com/embed/B3MuPBrrddE?si=WfNkEK6zqyGYgKb9" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen style="padding-right: 100px;"></iframe>

Orchestrator를 이용한 서버 설치 방법을 실습합니다.

📚 **이번 강의에서 다루는 내용**
- Orchestrator 기반 Open DID 서버 구성
- 서버 설치 및 기본 설정 절차
- 서버 구동 및 상태 확인
- 실습에 필요한 사전 환경 점검

⚠️ 본 강의는 Open DID 릴리즈 버전 2.0.0.0 버전 기준입니다.

---

### 2-2강. Open DID 서버 설치 (Gradle)
<iframe width="450" height="250" src="https://www.youtube.com/embed/Gsd_rgnVO4I?si=8ypT8rp6aQsS2VdM" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen style="padding-right: 100px;"></iframe>

Gradle 빌드를 통해 직접 서버를 설치하고 구동하는 방법을 실습합니다.

📚 **이번 강의에서 다루는 내용**
- Gradle 기반 서버 설치 개요
- 프로젝트 폴더 구조 및 설정 파일 이해
- Gradle 빌드 및 서버 실행
- 서버 구동 결과 확인

⚠️ 본 강의는 Open DID 릴리즈 버전 2.0.0.0 버전 기준입니다.

---

### 2-3강. Open DID 서버 설치 (IDE)
<iframe width="450" height="250" src="https://www.youtube.com/embed/wEXOBSXu6Is?si=myfQcn1psy636FsO" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen style="padding-right: 100px;"></iframe>

IntelliJ IDEA를 활용해 개발 환경에서 서버를 설치하고 실행하는 방법을 실습합니다.

📚 **이번 강의에서 다루는 내용**
- IDE 기반 서버 실행 환경 구성
- 프로젝트 구조 및 설정 파일 확인
- IDE 빌드 및 서버 실행
- 실행 결과 및 로그 확인

⚠️ 본 강의는 Open DID 릴리즈 버전 2.0.0.0 버전 기준입니다.

---

### 2-4강. Open DID 서버 설치 (Docker)
<iframe width="450" height="250" src="https://www.youtube.com/embed/pTwI3pzldS4?si=ZpoC0jEcxk-vpLda" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen style="padding-right: 100px;"></iframe>

Docker 컨테이너 기반으로 서버를 구성하는 방법을 실습합니다.

📚 **이번 강의에서 다루는 내용**
- Docker 기반 서버 구성 개요
- Docker Image 생성 방법
- Docker Compose를 이용한 서버 실행
- 컨테이너 상태 및 로그 확인

⚠️ 본 강의는 Open DID 릴리즈 버전 2.0.0.0 버전 기준입니다.

---

### 3강. Open DID 서버 등록 (Trust Registry)
<iframe width="450" height="250" src="https://www.youtube.com/embed/Png9jnR98OM?si=phk_fMdYwRlSwpvs" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen style="padding-right: 100px;"></iframe>

설치된 Open DID 서버를 신뢰 저장소에 등록하는 방법을 실습합니다.

📚 **이번 강의에서 다루는 내용**
- 서버 등록 개요 및 흐름 이해
- TA 서버 등록 실습
- Issuer 서버 등록 실습
- 빠른 등록 방법 소개

⚠️ 본 강의는 Open DID 릴리즈 버전 2.0.0.0 버전 기준입니다.

---

### 4강. Open DID App 설치
<iframe width="450" height="250" src="https://www.youtube.com/embed/g3FwjgP9m8w?si=VTen_SlPuo8Ri3xm" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen style="padding-right: 100px;"></iframe>

Open DID 모바일 앱을 설치하는 방법을 실습합니다.

📚 **이번 강의에서 다루는 내용**
- 앱 설치 개요 및 준비 사항
- 프로젝트 폴더 구조 안내
- 앱 설치 및 실행 실습
- 설치 결과 확인

⚠️ 본 강의는 Open DID 릴리즈 버전 2.0.0.0 버전 기준입니다.

---

### 5강. 사용자 등록
<iframe width="450" height="250" src="https://www.youtube.com/embed/5xTM6pHN9_Q?si=cDGVakJqInIYgqLb" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen style="padding-right: 100px;"></iframe>

Open DID 모바일 앱을 통해 사용자를 등록하는 방법을 실습합니다.

📚 **이번 강의에서 다루는 내용**
- 사용자 등록 개요
- 사용자 등록 사전 설정
- TA 어드민 설정 실습
- 사용자 등록 실습

⚠️ 본 강의는 Open DID 릴리즈 버전 2.0.0.0 버전 기준입니다.

---

### 6강. VC 발급 (Issuer-Initiated)
<iframe width="450" height="250" src="https://www.youtube.com/embed/v8PGJy1b6jM?si=TTl5cmv1Ke5OdKmb" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen style="padding-right: 100px;"></iframe>

Issuer 서버에서 발급 요청을 받아 VC를 발급받는 과정을 실습합니다.

📚 **이번 강의에서 다루는 내용**
- VC 개념 및 발급 흐름 설명
- ZKP 정책 설정 실습
- VC 정책 설정 실습
- VC 발급 실습

⚠️ 본 강의는 Open DID 릴리즈 버전 2.0.0.0 버전 기준입니다.

---

### 7강. VC 발급 (User-Initiated)
<iframe width="450" height="250" src="https://www.youtube.com/embed/PeTYccwPn3M?si=u0JxKwtARXYrEQ54" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen style="padding-right: 100px;"></iframe>

사용자가 모바일 앱을 통해 VC 발급을 직접 요청하는 과정을 실습합니다.

📚 **이번 강의에서 다루는 내용**
- VC 발급 흐름 이해
- VC 정책 설정 실습
- 사용자 주도 VC 발급 실습
- 발급 결과 확인

⚠️ 본 강의는 Open DID 릴리즈 버전 2.0.0.0 버전 기준입니다.

---

### 8강. VP 제출
<iframe width="450" height="250" src="https://www.youtube.com/embed/UphkkBjQyxg?si=49GHPaumcYFrdtFJ" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen style="padding-right: 100px;"></iframe>

발급받은 VC를 검증자에게 제출하는 과정을 실습합니다.

📚 **이번 강의에서 다루는 내용**
- VP 개념 및 제출 흐름 설명
- VP 정책 이해
- VP 정책 설정 실습
- VP 제출 실습

⚠️ 본 강의는 Open DID 릴리즈 버전 2.0.0.0 버전 기준입니다.

---

### 9강. ZKP Credential 제출
<iframe width="450" height="250" src="https://www.youtube.com/embed/Xt97pUTpzLY?si=1skhMHbFpgY9Fq-4" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen style="padding-right: 100px;"></iframe>

영지식 증명 기반 인증서를 제출하는 방법을 실습합니다.

📚 **이번 강의에서 다루는 내용**
- ZKP 개념 및 제출 흐름 이해
- ZKP 정책 이해
- ZKP 정책 설정 실습
- ZKP 제출 실습

⚠️ 본 강의는 Open DID 릴리즈 버전 2.0.0.0 버전 기준입니다.
