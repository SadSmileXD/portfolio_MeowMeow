# MeowMeow

> 라이브서비스 게임의 추가 콘텐츠를 실제 플레이 흐름으로 연결한 Unity 클라이언트 개발 프로젝트

## 목차

1. [프로젝트 소개](#프로젝트-소개)
2. [하이라이트](#하이라이트)
3. [기술 스택](#기술-스택)
4. [팀원 ](#팀원)
5. [주요 기여 및 담당 역할](#주요-기여-및-담당-역할)
6. [세부 구현](#세부-구현)
   - [1. Firebase Authentication](#1-firebase-authentication)
   - [2. MeowMeowStar 피드 조회 비용 제어](#2-meowmeowstar-피드-조회-비용-제어)
   - [3. Event Bus 기반 UI 이벤트 처리](#3-event-bus-기반-ui-이벤트-처리)
   - [4. 이미지 편집](#4-이미지-편집)
   - [5. Sticker 동적 생성](#5-sticker-동적-생성)
   - [6. Google Sheets → ScriptableObject 데이터 파이프라인](#6-google-sheets--scriptableobject-데이터-파이프라인)
   - [7. Profile 데이터 로컬 캐싱](#7-profile-데이터-로컬-캐싱)
   - [8. 추억의 뽑기판 데이터 파이프라인](#8-추억의-뽑기판-데이터-파이프라인)
   - [9. 기능별 ScriptableObject 분리](#9-기능별-scriptableobject-분리)
   - [10. Firestore Wrapper](#10-firestore-wrapper)
   - [11. UI 해상도 대응](#11-ui-해상도-대응)


---

# 프로젝트 소개

**MeowMeow**는 라이브서비스 게임의 추가 콘텐츠를 실제 플레이 흐름으로 연결한 Unity 클라이언트 개발 프로젝트입니다.

힐링 고양이 스토리 게임을 기반으로 SNS 형태의 피드 콘텐츠와 등급별 보상 뽑기 콘텐츠를 구현하고, Firebase를 활용한 사용자 인증 및 데이터 저장 구조를 구성했습니다.


### 주요 콘텐츠

- **MeowMeowStar**
  - SNS 형태의 피드 꾸미기 콘텐츠
  - 이미지 선택 및 편집
  - 댓글 / 해시태그 선택
- **추억의 뽑기판**
  - 뽑기권을 사용한 등급별 보상 획득 콘텐츠
  - 누적 횟수에 따른 보상 연출
- **로그인**
  - Google Login
  - Firebase Authentication
- **데이터 관리**
  - Firebase Firestore
  - Google Sheets → ScriptableObject

### 플레이 흐름

```text
Login
  ↓
Main
  ↓
콘텐츠 선택
 ├── MeowMeowStar
 └── 추억의 뽑기판
```

---

# 하이라이트

### 데이터 파이프라인

- Google Sheets → ScriptableObject → Runtime 구조
- 런타임 네트워크 파싱 제거
- Firestore 사용자 데이터 관리

### 서버 데이터 조회

- Firestore 전체 문서 조회 방식 개선
- 기준 필드 + `WhereGreaterThanOrEqualTo` + `Limit(N)`을 이용한 필요한 데이터 중심 조회

### 구조 설계

- Event Bus를 이용한 UI 직접 참조 감소
- 기능별 ScriptableObject 분리
- Firestore Wrapper를 통한 데이터 접근 추상화



---

# 기술 스택

| 분류 | 기술 | 활용 |
|---|---|---|
| Engine | Unity | 클라이언트 개발 |
| Language | C# | 게임 로직 및 시스템 구현 |
| Backend | Firebase | 인증 / 사용자 데이터 |
| Database | Firestore | 사용자 및 콘텐츠 데이터 |
| Data | Google Sheets | 기획 데이터 관리 |
| Runtime Data | ScriptableObject | 로컬 데이터 관리 |
| Shader | HLSL | 이미지 편집 효과 |
| Animation | DOTween | UI / 콘텐츠 연출 |


---

# 팀원 

| 구분 | 내용 |
|---|---|
| 개발 | 4명 |
| 기획 | 8명 |
| 기간 | 2026.05.21 ~ 2026.07.10 |
| 역할 | Unity Client Developer |

---

# 주요 기여 및 담당 역할

저는 프로젝트에서 **MeowMeowStar, 추억의 뽑기판, 로그인, 공통 시스템, 데이터 관리 및 UI**를 담당했습니다.

| 영역 | 담당 내용 |
|---|---|
| MeowMeowStar | 피드 조회, 이미지 편집, Sticker, 댓글 / 해시태그 |
| 추억의 뽑기판 | 뽑기 로직, 보상 데이터, 누적 보상 연출 |
| Login | Google Login / Firebase Authentication |
| 데이터 | Firestore, Google Sheets → ScriptableObject |
| 공통 시스템 | Data Manager, Audio Manager, Event Bus |
| UI | 이미지 편집 UI, 해상도 대응 |
| 구조 | 기능별 ScriptableObject, Firestore Wrapper |

---

# 세부 구현

## 1. Firebase Authentication

### 관련 코드

- [`GoogleSignInService.cs`](https://github.com/SadSmileXD/portfolio_MeowMeow/blob/main/MeowMeow/Assets/_Project/02_Scripts/Auth/GoogleSignInService.cs)
- [`UnityAuthService.cs`](https://github.com/SadSmileXD/portfolio_MeowMeow/blob/main/MeowMeow/Assets/_Project/02_Scripts/Auth/UnityAuthService.cs)
- [`LoginUI.cs`](https://github.com/SadSmileXD/portfolio_MeowMeow/blob/main/MeowMeow/Assets/_Project/02_Scripts/UI/LoginUI.cs)

### 설계 방향

Google Login을 Firebase Authentication과 연결하고 로그인 상태에 따라 게임 진입 UI를 제어하도록 구현했습니다.

또한 로그인 정보를 기준으로 사용자 데이터의 저장 경로를 결정하도록 구성했습니다.

### 동작 흐름

```text
Google Login
    ↓
Firebase Authentication
    ↓
계정 확인 / 생성
    ↓
로그인 완료
    ↓
게임 진입 UI 활성화
```

### 구현 결과

- 로그인 상태에 따른 게임 진입 UI 제어
- 사용자 계정 기준 데이터 저장 경로 설정

---

## 2. MeowMeowStar 피드 조회 비용 제어

### 관련 코드

- [`Extensionshuffle.cs`](https://github.com/SadSmileXD/portfolio_MeowMeow/blob/main/MeowMeow/Assets/FireStoreManager/important/extension%20method/Extensionshuffle.cs)

### 문제

기존 방식에서는 전체 피드 문서를 조회한 후 랜덤하게 N개의 데이터를 선택했습니다.

```text
전체 문서 조회
      ↓
랜덤 N개 선택
```

문서 수가 증가할수록 조회되는 문서량도 증가하기 때문에 **불필요한 Firestore 읽기 비용이 발생할 수 있었습니다.**

### 해결

랜덤 조회를 위한 기준 필드를 사용하고 범위 조건과 `Limit(N)`을 적용했습니다.

```text
기준 필드
    ↓
WhereGreaterThanOrEqualTo or WhereLessThan
    ↓
Limit(N)
    ↓
필요한 피드 조회
```

### 결과

전체 문서를 조회하지 않고 필요한 수량을 중심으로 데이터를 조회하도록 변경했습니다.

조회 비용을 `Limit(N)`을 통해 제한할 수 있도록 개선했습니다.

### Trade-off

랜덤 조회를 위한 기준 필드가 추가로 필요하며 데이터 분포와 누락 가능성을 고려해야 합니다.

---

## 3. Event Bus 기반 UI 이벤트 처리

### 관련 코드

- [`SubscribeManager.cs`](https://github.com/SadSmileXD/portfolio_MeowMeow/tree/main/MeowMeow/Assets/_Project/02_Scripts/Managers) 

### 문제

UI 간 직접 참조가 증가하면 Publisher가 Subscriber의 구체적인 구현을 알아야 하는 구조가 될 수 있습니다.

### 해결

Event Bus를 통해 편집에 필요한 데이터를 이벤트에 담아 발행하도록 구성했습니다.

```text
편집 버튼
    ↓
Event Publish
    ↓
Subscriber
    ↓
이미지 편집 UI
```

Publisher가 Subscriber의 구현을 직접 참조하지 않도록 구성했습니다.

### 장점

- UI 간 직접 참조 감소
- Publisher / Subscriber의 결합도 감소
- Subscriber 구현 변경 시 Publisher의 수정 범위 감소

### 주의점

이벤트 기반 구조는 직접 참조 방식보다 이벤트 흐름 추적이 어려울 수 있기 때문에 **필요한 범위에서 제한적으로 사용했습니다.**

---

## 4. 이미지 편집

### 관련 코드

- `CGEditor`
- Image Edit Shader

### 구현 목적

이미지 편집 UI에서 사용자가 변경한 값을 실시간으로 Shader에 전달하여 **이미지 편집 결과를 실시간으로 화면에 반영**했습니다.

### 동작 흐름

```text
Slider 값 변경
      ↓
Shader Property 갱신
      ↓
Fragment Shader
      ↓
색상 보정 연산
      ↓
UI 실시간 반영
```

### 구현 내용

- Slider 값 변경 감지
- Shader Property 갱신
- Fragment Shader에서 색상 보정
- 실시간 UI 반영

---

## 5. Sticker 동적 생성

### 관련 코드

- `StickerCanvas`

### 문제

사용자가 이미지 편집 과정에서 Sticker를 동적으로 추가할 수 있어 객체 생성 및 메모리 사용량을 고려할 필요가 있었습니다.

### 측정

Sticker 최대 생성 제한 상태에서 메모리 점유율을 측정했습니다.

**측정 결과: 약 8.2 KB**

### 결정

```text
Sticker 추가
    ↓
Instantiate
    ↓
사용
    ↓
Destroy
```

Object Pooling 대신 필요한 시점에 생성하고 사용 후 제거하는 방식을 선택했습니다.

Sticker의 사용 빈도와 동시 생성 제한이 낮아 상시 Pool을 유지하는 것보다 **구조를 단순하게 유지하고 관리 포인트를 줄이는 것**을 우선했습니다.

---

## 6. Google Sheets → ScriptableObject 데이터 파이프라인

### 관련 코드

- `GoogleSheetManager`
- `ScriptableObject Data`
- Editor Tool

### 문제

댓글과 해시태그 등의 기획 데이터를 런타임에서 직접 네트워크로 파싱하는 구조는 런타임 의존성을 증가시킬 수 있었습니다.

### 해결

Editor에서 Google Sheets 데이터를 ScriptableObject로 변환하고 런타임에서는 생성된 ScriptableObject를 사용하도록 구성했습니다.

```text
Google Sheets
      ↓
Editor Parse
      ↓
ScriptableObject
      ↓
Runtime
```

### Workflow

```text
기획 데이터 변경
      ↓
Editor 버튼 클릭
      ↓
SO 데이터 갱신
      ↓
Runtime에서 로컬 SO 사용
```

### 결과

- 런타임 네트워크 파싱 제거
- 기획 데이터 변경 시 Editor에서 재동기화
- 런타임 데이터 의존성 단순화

---

## 7. Profile 데이터 로컬 캐싱

### 문제

Profile에 진입할 때마다 Firebase에서 데이터를 다시 조회할 가능성이 있었습니다.

### 해결

Firebase에 데이터를 저장하면서 동시에 로컬 데이터를 사용하도록 구성했습니다.

```text
Firebase
   ↓
Profile Data
   ↓
Local Data
   ↓
Profile 재진입
   ↓
Local Data 사용
```

### 결과

Profile 진입 시 로컬 데이터를 사용하여 반복적인 Firestore 조회를 줄였습니다.

### Trade-off

기기 변경 시 로컬 데이터 복구에 한계가 있지만, 해당 기능에서는 반복 조회 비용 절감을 우선했습니다.

---

## 8. 추억의 뽑기판 데이터 파이프라인

### 관련 코드

- Gacha Data
- Reward Data
- ScriptableObject

### 설계 방향

외부 기획 데이터를 코드와 분리하여 보상 테이블이 변경될 경우 코드 수정 범위를 줄였습니다.

```text
Google Sheets
      ↓
ScriptableObject
      ↓
등급별 Reward List
      ↓
List Shuffle
      ↓
뽑기
      ↓
Reward ID 반환
```

### 런타임 흐름

```text
뽑기 클릭
    ↓
뽑기권 확인 / 차감
    ↓
버튼 순서 기반 Index 전달
    ↓
보상 ID 반환
```

---

## 9. 기능별 ScriptableObject 분리

### 문제

하나의 클래스에 여러 기능을 계속 추가하면 코드가 증가하고 메소드 탐색 및 수정 범위가 커질 수 있었습니다.

### Before

```text
하나의 클래스
 ├── Gacha
 ├── Audio
 ├── Data
 ├── ...
 └── 기능 증가
```

### After

```text
Gacha SO
Audio SO
Data SO
...
```

필요한 기능을 Inspector에서 연결할 수 있도록 기능별 ScriptableObject로 분리했습니다.

### 결과

- 기능별 책임 분리
- 수정 범위 감소
- Inspector를 통한 기능 연결
- 기능 확장 시 기존 클래스 영향 최소화

---

## 10. Firestore Wrapper

### 문제

Game Logic에서 Firestore API와 데이터 경로를 직접 관리하면 호출부에 긴 Firestore 경로가 노출되고 코드 가독성이 저하될 수 있었습니다.

### Before

```text
Game Logic
    ↓
Firestore API
    ↓
직접 경로 지정
```

### After

```text
Game Logic
    ↓
Firestore Wrapper
    ↓
공통 Manager
    ↓
Firestore
```

Inspector에서 데이터 경로를 관리하고 Wrapper를 통해 Firestore에 접근하도록 구성했습니다.

### 결과

- Firestore 호출부의 직접 의존성 감소
- 경로 관리 일관성 확보
- 호출부 가독성 향상

---

## 11. UI 해상도 대응

### 구현 내용

다양한 화면 비율에서도 UI의 크기와 상대 위치가 유지되도록 Canvas Scaler와 Anchor를 사용했습니다.

### 설정

```text
Canvas Scaler
 └─ Scale With Screen Size
      └─ Screen Match Mode
           └─ Expand

Anchor Presets
      ↓
상대 위치 유지
```

### 검증 해상도

- 1080 × 2340
- 1080 × 2560
- 1600 × 2560
- 1700 × 2500

### 결과

화면 비율이 달라져도 UI가 잘리지 않고 상대적인 위치를 유지하는지 검증했습니다.

---
 