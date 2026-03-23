# 🎟️ 길목 (Gilmok)
대규모 트래픽에 대응하는 지능형 마이크로서비스(MSA) 기반 온라인 티켓 예매 시스템

## 📖 프로젝트 개요

**길목(Gilmok)**은 수많은 사용자가 동시에 몰리는 티켓팅 순간에도 서버 다운과 무한 로딩을 방지하고, 공정한 대기 경험을 제공하기 위해 개발된 시스템입니다.

마이크로서비스 아키텍처(MSA)를 도입해 도메인 간 결합도를 낮추고, Redis 기반의 멱등성이 보장되는 대기열 시스템과 AI 기반 지능형 트래픽 제어를 통해 극한의 트래픽 상황에서도 고가용성(HA)을 유지하도록 설계했습니다.

## 🎯 핵심 목표 및 기술적 도전

- 스파이크 트래픽 방어: Redis 대기열을 통한 1차 트래픽 흡수와 Token Bucket 기반 유입 제어
- 공정한 선착순 보장: 새로고침이나 재접속 시에도 순번이 유지되는 멱등성 로직 구현
- 지능형 정책 조율: Prometheus 메트릭과 Gemini 기반 AI 추천으로 적정 입장 허용량 제안
- 무중단 운영: GitHub Actions와 Nginx 기반의 배포 파이프라인으로 안정적인 운영 환경 구축

## 🛠 기술 스택

### Backend

![Java](https://img.shields.io/badge/Java_17-007396?style=for-the-badge&logo=openjdk&logoColor=white)
![Spring Boot](https://img.shields.io/badge/Spring_Boot_3-6DB33F?style=for-the-badge&logo=springboot&logoColor=white)
![Spring Cloud Gateway](https://img.shields.io/badge/Spring_Cloud_Gateway-6DB33F?style=for-the-badge&logo=spring&logoColor=white)
![Spring Security](https://img.shields.io/badge/Spring_Security-6DB33F?style=for-the-badge&logo=springsecurity&logoColor=white)
![JWT](https://img.shields.io/badge/JWT-000000?style=for-the-badge&logo=jsonwebtokens&logoColor=white)
![Spring Data JPA](https://img.shields.io/badge/Spring_Data_JPA-6DB33F?style=for-the-badge&logo=spring&logoColor=white)

### Data & Storage

![MySQL](https://img.shields.io/badge/MySQL-4479A1?style=for-the-badge&logo=mysql&logoColor=white)
![Redis](https://img.shields.io/badge/Redis-DC382D?style=for-the-badge&logo=redis&logoColor=white)
![Valkey](https://img.shields.io/badge/Valkey-2F8CBB?style=for-the-badge&logo=redis&logoColor=white)

### DevOps & Infrastructure

![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Docker Compose](https://img.shields.io/badge/Docker_Compose-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=for-the-badge&logo=githubactions&logoColor=white)
![Nginx](https://img.shields.io/badge/Nginx-009639?style=for-the-badge&logo=nginx&logoColor=white)
![OCI](https://img.shields.io/badge/Oracle_Cloud-F80000?style=for-the-badge&logo=oracle&logoColor=white)
![AWS EC2](https://img.shields.io/badge/AWS_EC2-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)

### Monitoring & AI

![Prometheus](https://img.shields.io/badge/Prometheus-E6522C?style=for-the-badge&logo=prometheus&logoColor=white)
![OpenTelemetry](https://img.shields.io/badge/OpenTelemetry-000000?style=for-the-badge&logo=opentelemetry&logoColor=white)
![Grafana](https://img.shields.io/badge/Grafana-F46800?style=for-the-badge&logo=grafana&logoColor=white)
![Loki](https://img.shields.io/badge/Loki-F2CC0C?style=for-the-badge&logo=grafana&logoColor=black)
![Google Gemini](https://img.shields.io/badge/Google_Gemini-8E75B2?style=for-the-badge&logo=google-gemini&logoColor=white)

### Frontend

![React](https://img.shields.io/badge/React-20232A?style=for-the-badge&logo=react&logoColor=61DAFB)
![Vite](https://img.shields.io/badge/Vite-646CFF?style=for-the-badge&logo=vite&logoColor=white)
![Bootstrap](https://img.shields.io/badge/Bootstrap-7952B3?style=for-the-badge&logo=bootstrap&logoColor=white)

## ⚙️ 시스템 아키텍처

[![시스템 아키텍처](./image/architecture.png)](./image/architecture.png)

## 🗂️ ERD

[![ERD](./image/erd.png)](./image/erd.png)

## 🖥️ 화면 정의서


[![화면 정의서](./image/visily.png)](./image/visily.png)

## 🚀 주요 기능 및 핵심 엔지니어링 포인트

### 1) ⚡ 고성능 대기열 시스템

- Redis 기반 대기열로 급격한 트래픽을 1차 흡수
- Token Bucket과 멱등성 처리로 새로고침/재접속에도 동일한 대기 순번 유지
- 대기열 진입과 입장 흐름을 분리해 API 서버 과부하 전파를 최소화

### 2) 🛡️ 방어적 인증·인가 구조

- 세션리스 `JWT` 인증 구조 채택
- 로그아웃 또는 탈취 의심 토큰은 Redis 블랙리스트로 관리
- 대기열 통과 사용자에게만 짧은 수명의 입장 토큰을 발급해 서버 자원 독점을 방지

### 3) 🧠 AI 기반 트래픽 정책 추천

- `Prometheus` 메트릭을 수집해 현재 대기 상태와 시스템 부하를 분석
- `Gemini` 기반 추천으로 적정 동시 접속 허용량과 제어 정책을 제안
- 운영자가 경험에만 의존하지 않고 데이터 기반으로 정책을 조정할 수 있도록 설계

### 4) 🚦 고가용성 및 복원력

- `Resilience4j` 기반 Circuit Breaker로 장애 전파를 차단
- Redis 또는 외부 의존성 문제 발생 시 Fail-Open / Fallback 전략 적용
- 배포와 운영 환경에서 관측 가능성을 높여 빠른 복구를 지원

## 📦 마이크로서비스 레포지토리 구성

본 시스템은 MSA 구성을 채택해 각 도메인의 생명주기를 분리했습니다.

| Repository | Description |
| --- | --- |
| `api-repo` | 비즈니스 로직, 대기열, 예매, 좌석 선점, AI 추천 및 프론트엔드 |
| `auth-repo` | 인증, 인가 로직 및 JWT 토큰 관리 |
| `gateway-repo` | WebFlux 기반 API 라우팅 게이트웨이 |
| `common-repo` | 여러 서비스에서 함께 사용하는 공통 Java 모듈 |
| `deploy-repo` | 멀티 클라우드 배포, 모니터링, 운영 스크립트 관리 |

## 👥 팀원 소개

|  |  |  |
|-----------------|-----------------|-----------------|
| [차건희](https://github.com/gunheecha) | <img src="./image/차건희.png" alt="차건희" width="100"> | <ul><li>CI/CD 파이프라인 구축</li><li>`gateway-repo` 개발</li><li>메트릭/로그 수집 및 관측 환경 구성</li><li>AI 정책 추천 기능 구현</li></ul> |
| [최지우](https://github.com/zlor0) | <img src="./image/최지우.png" alt="최지우" width="100"> | <ul><li>정책 관리 기능 구현</li><li>관리자 화면 개발</li></ul> |
| [한재호](https://github.com/hjh009) | <img src="./image/한재호.png" alt="한재호" width="100"> | <ul><li>대기열 시스템 설계 및 구현</li><li>큐 처리 및 대기 순번 로직 개발</li></ul> |
| [황준현](https://github.com/junhyun1001) | <img src="./image/황준현.png" alt="황준현" width="100"> | <ul><li>인증/인가 기능 구현</li><li>입장 토큰 발급 로직 개발</li></ul> |

## 🧪 로컬 실행 방법

### 1) 요구 사항

- `Java 17`
- `Node.js 18+`
- `Docker` / `Docker Compose`

### 2) 서비스 실행 예시

```bash
cd common-repo && ./gradlew build
cd ../auth-repo && ./gradlew bootRun
cd ../api-repo && ./gradlew bootRun
cd ../gateway-repo && ./gradlew bootRun
cd ../api-repo/frontend && npm install && npm run dev
```

### 3) 참고 사항

- 저장소별 상세 설정은 각 레포 내부 문서를 기준으로 확인합니다.
- 배포 및 모니터링 구성은 `deploy-repo`에서 관리합니다.
- 추가 문서는 추후 `docs`에 정리할 수 있습니다.
