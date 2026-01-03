<div align="center">
  
# UniBooker 
</div>

<br/>

<img width="1758" height="303" alt="Group 144" src="https://github.com/user-attachments/assets/bb4b0055-a3d4-415c-8855-b1114ed4276b" />

<br /><br/>

## 📌 프로젝트 소개

**UniBooker**는 기업별로 상이한 예약 도메인을 유연하게 커스터마이징할 수 있도록 설계된 예약 시스템으로,
대규모 트래픽 환경에서도 **오버부킹 없이 안정적인 예약 처리**를 목표로 한 플랫폼입니다.
<br><br><br>

## 📘 프로젝트 개요
- 기업별로 다른 예약 정책을 수용하기 위한 **유연한 리소스/예약 모델**
- 동시 예약 트래픽으로 인한 장애 경험 후 **대기열 시스템 및 구조 개선**
- 모놀리식 → **MSA 전환**을 통해 서비스 확장성과 장애 격리 경험
- 예약, 리소스 관리, 통계 대시보드 기능 제공
<br><br><br>

## 👤 My Role

### 담당 역역
- 리소스 / 리소스 그룹 핵심 도메인 설계
- 예약 타입(예약형·신청형·좌석형) 분류 및 UI 연동 구조 설계
- 정규·예외 시간 슬롯 데이터 모델링 및 예약 가능 시간 계산 로직 구현
- MSA 전환 과정에서 서비스 분리 기준 정의 및 통신 구조 설계
- 통계 서버 설계
<br><br><br>

## 🔍 주요 문제 해결 & 기술적 의사결정
### 1️⃣ 기업별 예약 도메인 커스터마이징 문제
**문제 상황**
- 기업마다 예약 시 필요한 정보와 리소스 설명 필드가 상이
- 리소스 생성 시마다 동일한 필드를 반복 입력해야 하는 비효율 발생

**해결 전략**
- **리소스 그룹 도입**
  - 그룹 단위로 공통 리소스 필드 / 사용자 입력 필드 정의
  - 그룹 하위 리소스는 해당 설정을 상속

**효과**
- 예약 생성 과정 단순화
- 기업별 도메인 차이를 코드 수정 없이 흡수

---
### 2️⃣ 다양한 예약 형태를 수용하기 위한 구조 설계
**문제 상황**
- 회의실 예약, 동아리 신청, 좌석 예매 등 예약 형태가 상이

**설계 결정**
- 예약을 크게 **예약형 / 신청형 / 좌석형**으로 분류
- 예약 타입에 따라 **프론트 UI와 처리 로직을 분기**

<br/>

| 타입    | 설명                                    |
| ---------- | ---------------------------------------------- |
| 예약형 | 시간 단위로 예약 (회의실 등)                |
| 신청형 | 시간 개념 없이 참여 신청 (동아리 모집 등)                |
| 좌석형 | 좌석 선택이 핵심 (버스, 공연장 등)                |

---
### 3️⃣ 예약 가능 시간 계산 로직 설계
**배경**
- MSA 구조로 인해 예약 데이터는 예약 서버에 존재
- 리소스 서버에서는 예약 데이터에 직접 의존하지 않도록 분리 필요

**정규 시간 슬롯 설계**
- 리소스 생성 시 요일 × 시간 단위로 **정규 시간 슬롯 자동 생성**
- 관리자가 선택한 슬롯만 `is_active = true`

예) 월요일 00:00~01:00, 01:00~02:00 ...

**예외 시간 슬롯 설계**
- 특정 날짜의 휴무 또는 시간 변경 대응
- 정규 슬롯과 다른 형태로 저장

예)
- `2025-12-01 휴무`
- `2025-12-13 13:00~15:00`
- 휴게시간은 분리 저장

**계산 방식**
1. 정규 시간 슬롯 기준으로 예약 가능 시간 계산
2. 예외 시간 슬롯으로 덮어쓰기 (우선순위 ↑)

**의사결정 포인트**
- 수정 시 **기존 예외 슬롯 전부 삭제 후 재생성**
- 복잡한 시간 병합/분기 로직 대신 단순하고 안정적인 방식 선택

---
### 4️⃣ 대규모 트래픽 대응 : 대기열 시스템
**문제 상황**
- 동시 예약 요청으로 서버 장애 발생
- 응답 시간 30초 이상, 실패율 약 36%

**해결**
- Redis ZSET 기반 **대기열 서버 독립 구성**
- Gateway → Queue → API 흐름으로 트래픽 제어

**결과**
- 응답 시간 1초 이내
- 실패율 0%

---
### 5️⃣ 예약 동시성 제어 전략
**시도한 방식**
1. `synchronized` → 단일 서버 한계
2. DB 비관적 락 → 병목 발생

**최종 선택**
- Redis 분산 락 적용
- DB 접근 전 충돌 차단

---
### 6️⃣ MSA 전환 & 통계 서버 설계
**MSA 전환 이슈**
- 서비스 분리 후 DB 공유 불가
- 서비스 간 직접 호출 증가 시 장애 전파 위험

**해결**
- Kafka 이벤트 기반 데이터 동기화
- 서비스 간 결합도 최소화

**통계 서버**
- 별도 DB 없이 **Aggregation Server 역할**
- 프론트 단일 요청 → 통계 서버에서 여러 서버 데이터 수집 후 계산
<br><br><br>

## 📈 결과
- 대기열 도입 후 예약 실패율 **36% → 0%**
- 트래픽 증가 상황에서도 안정적인 예약 처리
- 기업별 예약 도메인을 수용 가능한 구조 확보
<br><br><br>

## 💡 배운 점
- 기술 선택은 유행이 아니라 **문제 경험 이후에 의미를 가진다**



## 🌐 접속 주소

### [플랫폼 관리자 바로가기](https://www.unibooker.kro.kr/super/login)

- ID : super@unibooker.com
- PW : super1234

### [기업 관리자 바로가기](https://www.unibooker.kro.kr/admin/login)

- ID : admin@unibooker.com
- PW : Lqwer1234!

### [고객 바로가기](https://www.unibooker.kro.kr/c/hanwha-systems)

- ID : test111@test.com
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
