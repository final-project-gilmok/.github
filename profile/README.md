<div align="center">

# 🎟️ 길목 (Gilmok) Ticketing Reservation System

**대규모 트래픽에 대응하는 지능형 마이크로서비스(MSA) 기반 온라인 티켓 예매 시스템**

</div>

<br/>

## 📖 프로젝트 개요
**길목(Gilmok)**은 수많은 사용자가 동시에 몰리는 티켓팅 이벤트 순간에도 **서버 다운과 무한 로딩을 방지하고 공정한 대기 경험을 제공**하기 위해 개발된 시스템입니다. 
마이크로서비스 아키텍처(MSA)를 도입하여 도메인 간 결합도를 낮추고, Redis 기반의 멱등성이 보장되는 대기열 시스템과 AI 기반 지능형 트래픽 제어 기술을 통해 극한의 트래픽 상황에서도 고가용성(HA)을 유지하도록 설계되었습니다.

<br/>

## 🎯 핵심 목표 및 기술적 도전
- **스파이크 트래픽 방어**: Redis 대기열을 통한 1차 트래픽 흡수(Throttling) 및 Token Bucket 기반 제어
- **공정한 선착순(FIFO) 보장**: 새로고침이나 재접속 시에도 본인의 순번이 완벽히 유지되는 멱등성(Idempotency) 로직 구현
- **지능형 관측 및 정책 조율**: Prometheus 메트릭과 Gemini 2.5 Flash를 활용한 AI 기반 최적의 적정 입장 허용량 제안
- **무중단 운영(CI/CD)**: GitHub Actions와 Nginx를 활용한 OCI/AWS 멀티 클라우드 무중단 배포(Blue-Green/Rolling) 파이프라인 구축

<br/>

## 🛠 기술 스택 (Tech Stack)

### **Backend**
- Java 17
- Spring Boot 3.x
- Spring Cloud Gateway (WebFlux / 비동기 논블로킹)
- Spring Security & JWT
- Spring Data JPA, QueryDSL

### **Data & Storage**
- MySQL (Aiven Managed Database)
- Valkey / Redis (대기열 ZSET, 분산 락, JWT 블랙리스트 관리에 사용)

### **DevOps & Infrastructure**
- Docker & Docker Compose
- GitHub Actions (CI/CD)
- Oracle Cloud Infrastructure (OCI) / AWS EC2
- Nginx (Reverse Proxy & Blue-Green Load Balancing)

### **Monitoring & AI**
- PLG Stack: Promtail, Loki, Grafana (중앙 집중형 로깅)
- Prometheus (메트릭 수집)
- Google Gemini 2.5 Flash API (AI 기반 TPS 정책 추천)

<br/>

## ⚙️ 시스템 아키텍처 (System Architecture)
<details>
<summary>아키텍처 구성도 보기</summary>
(여기에 작성하신 시스템 아키텍처 이미지 링크나 주소를 넣어주세요)
</details>

1. **Routing**: 모든 외부 트래픽은 Nginx를 거쳐 `Spring Cloud Gateway`로 인입됩니다.
2. **Auth Layer (`auth-repo`)**: 액세스 및 리프레시 토큰(RTR 방식을 적용)을 발급하고, 만료 및 탈취 의심 시 Redis 블랙리스트로 관리합니다.
3. **API Layer (`api-repo`)**: 예매, 대기열, 정책 로직을 담당합니다.
4. **Data Layer**: 데이터베이스(MySQL) 접근을 최소화하기 위해 짧은 수명(5분)의 **One-Time 입장 토큰**을 인터셉터 레벨에서 검증하고, 티켓 좌석 락킹은 Redis 분산 락으로 제어합니다.

<br/>

## 🚀 주요 기능 및 핵심 엔지니어링 포인트

### 1) ⚡ 초경량·고성능 대기열 시스템 (Virtual Waiting Room)
- **단 1 RTT로 최적화된 Lua 스크립트**: Spring 서버와 Redis 간의 잦은 네트워크 통신(최대 7번)을 단 한 번의 Lua 스크립트로 압축하여 Token Bucket 및 동시성 제어를 원자적(Atomic)으로 처리.
- **새로고침 방어(Idempotency)**: 대기 리스트 진입 시 [UserId](cci:1://file:///Users/junhyun/Dev/DevCourse/AIBE/AIBE4_Final_Project_Team3/api-repo/src/main/java/kr/gilmok/api/queue/service/QueueService.java:290:4-298:5) 해시 맵핑을 통해 무분별한 새로고침 시에도 대기열 순번이 고정되는 멱등성 확보.

### 2) 🛡️ 방어적 JWT 인증 & 인가 아키텍처
- **Stateless 구조 및 블랙리스트**: 세션리스 JWT 방식을 채택하고, 탈취 및 로그아웃된 토큰은 `AccessTokenBlocklistFilter`를 통해 Redis에서 `logout` 상태로 관리 후 TTL 만료 시 자동 삭제.
- **입장 전용 One-Time 토큰**: 대기열을 통과한 유저에게만 발급되며, 예매 API 인터셉터 단위에서 검증 후 즉각 폐기되는 1회용 토큰(jti)으로 불필요한 서버 자원 독점 방지.

### 3) 🧠 AI 기반 트래픽 정책 추천 (AI Policy Recommendation)
- 실시간 수집된 **Prometheus 메트릭(대기 상태, 에러율, текущ RPS)** 데이터를 바탕으로, **Gemini 2.5 Flash** 모델이 운영 환경에 맞는 동시 접속 허용량 및 차단 룰을 추천 및 자동 적용. 빠르고 가벼운 Flash 모델로 비용 효율 및 응답 속도 극대화.

### 4) 🚦 고가용성 및 복원력 (Resilience)
- **Fail-Open & Emergency Stop**: 과부하 발생 시 또는 AI의 차단 판단에 따라, Circuit Breaker(`Resilience4j`)를 통해 트래픽을 즉시 격리하거나 대체 우회(Fallback) 처리. (Redis 장애가 API 서비스 장애로 전파되는 폭포수 현상 차단)

<br/>

## 📦 마이크로서비스 레포지토리 구성
본 시스템은 MSA 구성을 채택하여, 각 도메인별 생명주기를 완벽히 분리했습니다.
- [**api-repo**](https://github.com/Organization이름/api-repo): 비즈니스 로직(대기열, 예매, 좌석 선점, AI 추천) 및 프론트엔드
- [**auth-repo**](https://github.com/Organization이름/auth-repo): 인증, 인가 로직 및 JWT 토큰 관리
- [**gateway-repo**](https://github.com/Organization이름/gateway-repo): WebFlux 기반의 API 라우팅 게이트웨이
- [**deploy-repo**](https://github.com/Organization이름/deploy-repo): 멀티 클라우드 무중단 배포 및 PLG 모니터링 환경 세팅용 스크립트

<br/>

## 👩‍💻 팀원 소개 및 역할
- **박준현**: (역할을 적어주세요. 예: 아키텍처 설계, CI/CD 구축, Redis 대기열 구현 등)
- **팀원2**: (역할을 적어주세요)
- **팀원3**: (역할을 적어주세요)
- **팀원4**: (역할을 적어주세요)

<br/>

---
**💡 로컬 실행 방법 및 기여(Contributing) 가이드**는 각 레포지토리의 README 문서 및 상세 Wiki를 참조해 주세요.
