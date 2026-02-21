<div align="center">
  
# UniBooker 
</div>

<br/>

<img width="1758" height="303" alt="Group 144" src="https://github.com/user-attachments/assets/bb4b0055-a3d4-415c-8855-b1114ed4276b" />

<br /><br/>

## 📌 프로젝트 소개

**UniBooker**는 기업별로 상이한 예약 도메인을 유연하게 커스터마이징할 수 있도록 설계된 **B2B 예약 플랫폼**입니다.

동시 예약 트래픽으로 인한 실제 장애 경험을 바탕으로, **대규모 트래픽 환경에서도 오버부킹 없이 안정적으로 예약을 처리하는 것**을 목표로 개발했습니다.

초기 모놀리식 구조에서 출발하여 트래픽 문제와 확장성 한계를 겪으며 **MSA 구조로 전환**했고, 그 과정에서 **대기열 시스템, 분산 락, 이벤트 기반 통계 서버**를 설계·구현했습니다.
<br><br><br>

## 📘 프로젝트 개요
- 기업별로 다른 예약 정책을 수용하기 위한 **유연한 리소스/예약 모델**
- 동시 예약 트래픽으로 인한 장애 경험 후 **대기열 시스템 및 구조 개선**
- 모놀리식 → **MSA 전환**을 통해 서비스 확장성과 장애 격리 경험
- 예약 수, 취소율, 리소스 이용 현황, 성별·연령대별 예약 비율 등 **운영 의사결정에 필요한 지표를 제공하는 통계 대시보드 설계·구현**
<br><br><br>

## 👤 My Role

### 담당 역역
- **리소스 도메인 서버 설계 및 구현**
  - 기업이 예약 대상(시설, 좌석 등)을 자유롭게 정의할 수 있는 구조 설계
- **예약 타입(예약형·신청형·좌석형) 분류 및 UI 연동 구조 설계**
- **정규 운영 시간과 특정 날짜 예외 상황을 분리한 데이터 모델 설계 및 구현**
- **MSA 초기 구조 설계**
  - 헥사고날 아키텍처 기반 서비스 템플릿 설계 및 공유
- **이벤트 기반 통계 서버 설계 및 구현**
  - 여러 서비스 데이터를 집계하는 Aggregation Server 구현
 
이밖에도 서비스·시스템 아키텍처 설계, DB 구조 설계 및 ERD, 기획 아이디어 논의, Figma 기반 화면 설계 등 시스템 전반적으로 참여하였습니다.
<br><br><br>

## 🔍 주요 문제 해결 & 기술적 의사결정
### 1️⃣ 기업별 예약 도메인 커스터마이징 문제
**문제 상황**
- 기업마다 예약 시 필요한 고객 정보와 리소스(예약 상품) 설명 필드가 다름
  - 어떤 곳은 "회의 참여 부서", 어떤 곳은 "참여 인원", 어떤 곳은 "좌석 번호"가 중요
- 리소스를 추가할 때마다 동일한 설정을 반복 입력해야 하는 비효율 발생

**해결 전략**
- **리소스 그룹(Resource Group) 개념 도입**
  - 그룹 단위로 공통 설명 필드 / 사용자 입력 필드 / 예약 정책 정의
  - 개별 리소스는 그룹 설정을 상속

**효과**
- 리소스 생성 및 관리 과정 단순화
- 기업별 요구사항을 코드 수정 없이 설정으로 흡수

---
### 2️⃣ 다양한 예약 형태를 수용하기 위한 구조 설계
**문제 상황**
- 회의실 예약: 시간 단위 예약
- 동아리 모집: 신청형
- 공연·버스 예약: 좌석 선택 중심

**설계 결정**
예약을 **3가지 타입으로 명확히 분리**

<br/>

| 타입    | 설명                                    |
| ---------- | ---------------------------------------------- |
| 예약형 | 시간 단위 예약 (회의실 등)                |
| 신청형 | 시간 개념 없는 참여 신청 (동아리 모집 등)                |
| 좌석형 | 좌석 선택이 핵심 (버스, 공연장 등)                |

- 예약 타입에 따라
  - 프론트 UI
  - 서버 처리 로직
  - 검증 규칙 분기
- **새로운 예약 형태 추가 시 기존 구조를 깨지 않도록 설계**

---
### 3️⃣ 예약 가능 시간 계산 로직 설계
**배경**
- MSA 구조에서
  - 예약 데이터는 **예약 서버**
  - 리소스 정보는 **리소스 서버**
- 리소스 서버가 예약 서버 DB에 직접 의존하지 않도록 분리 필요

**정규 시간 슬롯 설계**
- 리소스 생성 시 요일 × 시간 단위로 **정규 시간 단위 슬롯 자동 생성**
- 관리자가 활성화(`is_active = true`)한 슬롯만 사용

예) 월요일 00:00~01:00, 01:00~02:00 ...

**예외 시간 슬롯 설계**
- 특정 날짜의 휴무 또는 운영 시간 변경
- 정규 슬롯과 별도 테이블로 관리(저장 형태도 상이)

예)
- `2025-12-01 휴무`
- `2025-12-13 13:00~15:00`
- 휴게시간은 분리 저장

**계산 방식**
1. 정규 시간 슬롯 기준으로 예약 가능 시간 계산
2. 예외 시간 슬롯이 있으면 덮어쓰기 (우선순위 ↑)

**의사결정 포인트**
- 수정 시 **기존 예외 슬롯 전부 삭제 후 재생성**
- 복잡한 시간 병합/분기 로직 대신 단순하고 예측 가능한 방식 선택

---
### 4️⃣ 대규모 트래픽 대응 : 대기열 시스템
**문제 상황**
- 동시 예약 요청으로 서버 장애 발생
- 응답 시간 30초 이상, 실패율 약 36%

**해결**
- Redis ZSET 기반 **대기열 서버 독립 구성**
- Gateway → Queue → API 흐름으로 트래픽 제어

**결과**
- 실패율 0%
- 응답 시간 1초 이내

---
### 5️⃣ 예약 동시성 제어 전략
**시도한 방식**
1. `synchronized` → 단일 서버 한계
2. DB 비관적 락 → 병목 발생

**최종 선택**
- Redis 분산 락 적용
- DB 접근 전 충돌 차단

---
### 6️⃣ MSA 전환 & 웹 소켓
**문제 상황**
- 모놀리식에서는 정상 동작하던 WebSocket 기반 알림 기능이 Gateway 도입 후 MSA 구조로 전환하면서 연결되지 않는 문제 발생
- 기존 HTTP 요청과 동일한 인증 필터를 WebSocket 요청에도 적용하면서 **초기 핸드셰이크 및 장시간 연결 유지 과정에서 인증 실패 발생**

**고민 과정**
- WebSocket은 일반 HTTP 요청과 달리 **연결 이후 지속적인 세션 유지가 필수적인 통신 방식**
- 모든 요청을 인증 필터를 거치게 할 경우:
  - 연결 유지 불안정
  - 알림 전달 실패
- 해결 방향 선택지:
1. 알림 기능을 별도의 모놀리식 서버로 분리
2. MSA 구조 내에서 WebSocket 통신 방식 자체를 재설계
<br/>→ 시스템 일관성을 유지하기 위해 2번 방식을 선택

**해결 전략**
> WebSocket 관련 설정은 MSA 환경에서의 인증 및 세션 유지 특성을 고려해 반복적인 테스트와 수정 과정을 거쳐 안정화했습니다.
- Gateway 레벨에서 WebSocket 관련 요청을 API 요청과 분리
- WebSocket 연결 요청은 인증 필터를 거치지 않고 대상 서버로 전달
- 서버는 연결 유지 상태를 기준으로 알림을 전달하도록 설계
- 클라이언트 측에서는 연결 끊김 발생 시 자동 재연결 처리

**결과**
- MSA 구조에서도 WebSocket 기반 실시간 알림 기능 정상 동작
- 모놀리식 서버를 별도로 유지하지 않고 시스템 구조 단순화
- Gateway 도입 이후에도 실시간 기능을 안정적으로 제공 가능

<br><br><br>

## 📈 결과
- 대기열 도입 후 예약 실패율 **36% → 0%**
- 트래픽 증가 상황에서도 안정적인 예약 처리
- 기업별 예약 도메인을 수용 가능한 구조 확보
<br><br><br>

## 💡 배운 점
- 기술 선택은 유행이 아니라 **문제 경험 이후에 의미를 가진다**



## 🌐 접속 주소

### [플랫폼 관리자 바로가기](https://www.unibooker.n-e.kr/super/login)

- ID : super@unibooker.com
- PW : super1234!

### [기업 관리자 바로가기](https://www.unibooker.n-e.kr/admin/login)

- ID : admin.cho@hanwha.com
- PW : Admin1234!

### [고객 바로가기](https://www.unibooker.n-e.kr/c/hanwha-systems)

- ID : user.jeon@hanwha.com
- PW : qwer1234!


<br><br>

## 🛠 Teck Stack
- **Backend**: Java, Spring Boot, JPA
- **Infra**: AWS EC2, RDS, Redis, Kafka
- **Architecture**: MSA, Event-driven
- **DB**: MySQL

<br><br>

## 🏗️ 시스템 아키텍처 [🔗](https://github.com/beyond-sw-camp/be17-fin-LinkVerse-UniBooker-BE/wiki/3.-%EC%8B%9C%EC%8A%A4%ED%85%9C-%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98)
### V1 ![3. 시스템아키텍처_v1](https://github.com/beyond-sw-camp/be17-fin-LinkVerse-UniBooker-BE/blob/develop/docs/3.%20시스템%20아키텍처_v1.png)
<br/>

### V2 ![4. 시스템아키텍처_v2](https://github.com/beyond-sw-camp/be17-fin-LinkVerse-UniBooker-BE/blob/develop/docs/3.%20시스템%20아키텍처_v2.png)
<br>

<br><br>

## 🛢️ ERD [🔗](https://github.com/beyond-sw-camp/be17-fin-LinkVerse-UniBooker-BE/wiki/5.-ERD)
![5. ERD](https://github.com/user-attachments/assets/0b21618e-43e3-4f0f-97cf-3f959b31c888)

<br><br>

## 🖥 Swagger [🔗](https://github.com/beyond-sw-camp/be17-fin-LinkVerse-UniBooker-BE/wiki/6.-Swagger-UI)
> [Swagger-UI 링크로 이동하기](https://www.unibooker.kro.kr/webjars/swagger-ui/index.html)

<br><br>

## 📺 기능 테스트
> 추후 추가 예정

<br><br>

## 📄 프로젝트 상세 문서
#### 📌 프로젝트 기획서
> [프로젝트 기획서 보러가기](https://github.com/beyond-sw-camp/be17-fin-LinkVerse-UniBooker-BE/wiki/1.-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8-%EA%B8%B0%ED%9A%8D%EC%84%9C)
#### 📌 요구사항 정의서
> [요구사항 정의서 보러가기](https://github.com/beyond-sw-camp/be17-fin-LinkVerse-UniBooker-BE/wiki/2.-%EC%9A%94%EA%B5%AC%EC%82%AC%ED%95%AD-%EC%A0%95%EC%9D%98%EC%84%9C)
#### 📌 WBS
> [WBS 보러가기](https://github.com/beyond-sw-camp/be17-fin-LinkVerse-UniBooker-BE/wiki/4.-WBS)
