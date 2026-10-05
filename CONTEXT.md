# CONTEXT.md — Ticketing × Finance Backend Lab

> 이 문서는 AI 에이전트가 프로젝트의 **비전, 도메인, 확정 범위, 설계 철학, Experiment 계획, 아키텍처 진화 이유**를 이해하기 위한 Source of Truth다.
>
> 구현 방법과 명령 규칙은 `AGENTS.md`를 따른다.

---

# 0. Context Contract

## Status Legend

- **FINAL**: 현재 확정된 결정. 사용자의 명시적 요청 없이 바꾸지 않는다.
- **BACKLOG**: 학습 가치가 있지만 핵심 범위 완료 이후 검토한다.
- **OPTIONAL**: 사용해도 되지만 프로젝트 완료 조건은 아니다.
- **NON-GOAL**: 현재 프로젝트가 의도적으로 하지 않는 것.

## Project Identity — FINAL

프로젝트 가칭은 **Ticketing × Finance Backend Lab / Ticket-Lab**이다.

이 프로젝트는 DevOps/MLOps 엔지니어가 백엔드 개발자의 실제 고충과 설계 제약을 경험하기 위한 **고트래픽 티켓팅 + 금융 정합성 학습 프로젝트**다.

핵심 질문:

> **“10명이 사용하던 Spring Boot 애플리케이션을 10,000명이 동시에 사용하면 어떤 일이 일어나는가?”**

프로젝트의 성과는 기능 수가 아니라 다음 사이클을 얼마나 많이, 정확하게 수행했는가로 평가한다.

```text
정상 구현
  ↓
의도적 장애/부하 주입
  ↓
Metrics / Logs / Traces로 관측
  ↓
Root Cause Analysis
  ↓
Backend Fix / Platform Mitigation 분리
  ↓
Before / After 측정
  ↓
Trade-off 기록
```

---

# 1. 왜 티켓팅 + 금융인가

## Ticketing — FINAL

티켓팅은 **한정된 자원인 좌석을 많은 사용자가 동시에 획득**하려는 도메인이다. 그래서 다음 주제가 억지 없이 자연스럽게 발생한다.

- Race Condition
- Double Booking
- Optimistic / Pessimistic Lock
- Transaction Isolation
- Cache / Hot Key / Stampede
- Waiting Queue / Thundering Herd
- Connection Pool 고갈
- Timeout / Retry / Cascading Failure
- Kafka / Event-Driven
- Idempotency

단순 CRUD 서비스보다 “왜 이런 기술이 필요한지”를 장애를 통해 설명하기 좋다.

## Finance — FINAL

금융 기능을 결합하면 **좌석 정합성뿐 아니라 금전 정합성**을 동시에 다룰 수 있다.

추가로 발생하는 중요한 문제:

- Duplicate Payment
- Unknown Payment State
- Refund consistency
- Wallet concurrent withdrawal
- Ledger invariant
- Settlement restartability / duplication
- Reconciliation
- Saga / Compensation

금융은 프로젝트의 목적을 바꾸는 별도 서비스가 아니라 **백엔드 정합성과 분산 트랜잭션을 깊게 학습하기 위한 도메인 확장**이다.

---

# 2. 프로젝트 목표

## Backend Learning — FINAL

다음을 실제 구현과 실패 시나리오를 통해 학습한다.

- Spring MVC
- Validation / Exception Handling
- JPA / Hibernate
- SQL / Query plan / Index
- Transaction
- Isolation Level
- Lock
- Connection Pool
- Idempotency
- Cache
- Timeout / Retry / Circuit Breaker
- Concurrent request handling
- Batch / Restartability
- Event-Driven Architecture

특히 **트래픽 증가에 따른 동시성/경쟁 상태 문제는 필수 핵심 영역**이다.

## Architecture Learning — FINAL

MSA를 먼저 적용하는 것이 아니라 다음 순서로 학습한다.

```text
Monolith
  ↓
실제 병목 / 장애 / 확장 단위 문제 관측
  ↓
왜 MSA가 필요한지 설명
  ↓
MSA 전환
  ↓
분산 시스템의 새로운 실패 모드 경험
```

## Platform Learning — FINAL

기존 DevOps 경험을 활용하되, 플랫폼을 단순 설치 체크리스트로 사용하지 않는다.

목표는 다음 질문에 답하는 것이다.

> 이 장애의 근본 원인은 Application인가 Platform인가?
>
> Backend에서 반드시 고쳐야 하는가, Infra가 완화할 수 있는가, 둘 다 필요한가?

## MLOps Learning — FINAL

Kubeflow를 LLM 전용 도구로 보지 않는다.

일반 ML 모델로 다음 lifecycle을 먼저 구현한다.

```text
Data
  ↓
Preprocessing
  ↓
Training
  ↓
Evaluation
  ↓
Model Artifact
  ↓
Serving
  ↓
Monitoring / Retraining 후보
```

---

# 3. 확정 기술 스택

## Application — FINAL

| 영역 | 기술 | 목적 |
|---|---|---|
| Backend | Spring Boot | 핵심 애플리케이션 |
| Distributed | Spring Cloud | Gateway / resilience / 분산 환경 학습 |
| Frontend | **Vite + React** | 최소 사용자 UI 및 흐름 확인 |

Frontend는 백엔드 학습을 지원하는 수준으로 유지한다. 복잡한 디자인 시스템이나 프런트엔드 아키텍처 연구가 본 프로젝트의 목표는 아니다.

## Data / Messaging / AWS — FINAL

- AWS VPC
- RDS MySQL
- Kafka on EC2
- Redis
- SQS
- 필요에 따른 EC2

모든 기술을 첫 Phase부터 넣을 필요는 없다. **문제가 나타난 뒤 필요성을 설명할 수 있을 때 도입**하는 것을 우선한다.

## Kubernetes / Platform — FINAL

- Self-managed Kubernetes
- **No EKS**
- Prometheus
- Grafana
- Loki
- Tempo
- OpenTelemetry
- ArgoCD
- Istio
- GitHub Actions

## MLOps — FINAL

- Kubeflow
- KServe

## Optional

- ELK — OPTIONAL, 가볍게 경험 가능
- Terraform — OPTIONAL
- Ansible — OPTIONAL
- Katib — OPTIONAL / 후속 확장

---

# 4. 명시적 Non-Goals / 후순위

## NON-GOAL — 실제 금융 연결

실제 카드, 실제 PG, 실제 금전은 사용하지 않는다.

대신:

- Mock Payment Gateway
- 가상 Wallet
- 가상 금액

을 사용하여 금융 시스템의 **상태 관리와 정합성 문제**에 집중한다.

## BACKLOG — Resale / Escrow

Resale Marketplace와 Escrow는 좋은 확장 주제지만 초기 범위에서 제외한다.

이유:

- 기술 학습보다 비즈니스 로직 양이 급격히 증가한다.
- 현재 핵심 목표인 Backend / Distributed / MLOps 학습을 지연시킬 수 있다.

핵심 40 Experiment와 MLOps가 완료된 뒤 선택적으로 추가한다.

## NON-GOAL — LLM First

LLM은 초기 MLOps 단계의 필수 요소가 아니다.

일반 ML → Kubeflow → KServe lifecycle을 먼저 완료한 후 선택적으로 추가한다.

## NON-GOAL — 처음부터 MSA

처음부터 서비스별로 분리하지 않는다.

MSA는 Monolith의 실제 문제를 경험한 뒤 도입한다.

## NON-GOAL — 기능이 많은 상용 서비스 복제

프로젝트의 핵심은 기능 수가 아니다.

**장애 재현, 관측, 분석, 해결, 측정**이 최우선이다.

---

# 5. 도메인 범위

## 초기 핵심 도메인 — FINAL

```text
User
Event
Performance
Seat
Reservation
Ticket
Payment
Refund
Wallet
Ledger
Settlement
```

### Ticketing Flow

```text
공연 조회
  ↓
회차 조회
  ↓
좌석 조회
  ↓
좌석 선점 / 예약
  ↓
결제
  ↓
티켓 확정
```

### Cancel / Refund Flow

```text
예약 취소 요청
  ↓
환불
  ↓
예약/티켓 상태 변경
  ↓
좌석 재오픈
```

### Finance Scope — FINAL

| 도메인 | 역할 | 주요 학습 포인트 |
|---|---|---|
| Payment | 결제 요청 / 승인 / 상태 관리 | Idempotency, Unknown State |
| Refund | 취소에 따른 환불 | Saga / Compensation |
| Wallet | 가상 잔액 | Concurrent withdrawal, Lost Update |
| Ledger | 금전 이동 원장 | Atomicity, Invariant, Reconciliation |
| Settlement | 주최자/플랫폼 정산 시뮬레이션 | Batch Restart, Duplicate prevention |

Ledger는 단순 `balance` 컬럼 변경만으로 끝내지 않고, 원장을 통해 금융 정합성 문제를 학습하는 방향을 우선한다.

---

# 6. 아키텍처 진화 경로

## Stage A — Spring Boot Monolith — FINAL

처음에는 가능한 한 단순하게 시작한다.

```text
Vite + React
     ↓
Spring Boot Monolith
 ├─ Event / Performance / Seat
 ├─ Reservation / Ticket
 ├─ Payment / Refund
 ├─ Wallet / Ledger
 └─ Settlement
     ↓
RDS MySQL
```

Redis / Kafka / MSA를 “좋아 보이기 때문에” 먼저 넣지 않는다.

먼저 다음을 경험한다.

- JPA N+1
- Slow Query
- Index 문제
- Transaction boundary
- Lock / Deadlock
- Isolation anomaly
- Connection Pool
- Double Booking
- Duplicate Payment
- Wallet race

## Stage B — Monolith를 트래픽과 장애로 압박 — FINAL

부하 테스트와 장애 주입을 통해 단순한 코드가 트래픽에서 어떻게 깨지는지 확인한다.

```text
Load Test
  ↓
N+1 / Slow Query
Lock / Deadlock
Connection Pool
Timeout / Retry
Cache Stampede
Double Booking
Duplicate Payment
Wallet Race
```

모든 중요한 문제는 가능하면 Metrics / Logs / Traces와 함께 확인한다.

## Stage C — MSA / Event-Driven — FINAL

Monolith에서 다음 문제가 충분히 관측된 뒤 분리한다.

- 특정 기능만 독립적으로 확장하고 싶은 문제
- 장애 전파
- 배포 결합
- 확장 단위 결합
- 긴 동기 호출 체인

후보 구조:

```text
Spring Cloud Gateway
        │
 ┌──────┼─────────┬────────────┐
 Event  Reservation Payment   Wallet/Ledger
                  │
                Kafka
                  │
            Settlement / Workers
```

이 단계에서 반드시 학습할 패턴:

- Saga
- Compensating Transaction
- Transactional Outbox
- Inbox / Idempotent Consumer
- Retry / Backoff
- DLQ
- At-least-once delivery
- Event Ordering
- Eventual Consistency

## Stage D — Kubernetes / Platform — FINAL

```text
GitHub Actions
      ↓
Container Registry
      ↓
GitOps Repo
      ↓
ArgoCD
      ↓
Self-managed Kubernetes (AWS EC2)
      │
      ├─ Istio
      ├─ Prometheus / Grafana
      ├─ Loki
      └─ Tempo / OpenTelemetry
```

AWS 측 주요 구성:

```text
VPC
RDS MySQL
EC2 Kafka
Redis
SQS
EC2-based Kubernetes
```

Infra는 설치 자체가 목표가 아니라 Experiment의 **Platform mitigation / failure domain**을 검증하는 데 사용한다.

## Stage E — ML / MLOps — FINAL

첫 모델은 티켓 수요 / Peak RPS 예측이다.

```text
Ticket / Traffic History
          ↓
          S3
          ↓
    Kubeflow Pipeline
Preprocess → Train → Evaluate
          ↓
     Model Artifact
          ↓
        KServe
          ↓
Demand / Peak RPS Prediction API
```

---

# 7. Backend vs Infra 책임 경계

## 핵심 원칙 — FINAL

> **데이터가 틀렸거나 비즈니스 불변식이 깨졌다면 Backend가 1차 책임이다.**
>
> **가용성, 용량, 네트워크, 배포 환경 문제는 Infra가 주도한다.**
>
> Timeout / Retry / Cache / Messaging은 Shared 영역이므로 Application Fix와 Platform Mitigation을 분리한다.

| 문제 영역 | Primary | Application 관점 | Platform 관점 |
|---|---|---|---|
| 데이터 정합성 / Business Invariant | Backend | Constraint, Transaction, Lock, Idempotency | Scale-out로 해결하지 않음 |
| 비즈니스 의미가 있는 Retry / 보상 | Backend | Saga, Compensation, State Machine | Mesh는 비즈니스 의미를 모름 |
| CPU / Memory / Node / AZ / Probe | Infra | 앱 효율 개선 가능 | Resource / HA / Probe / Deployment 주 책임 |
| Timeout / Retry / Circuit Breaker | Shared | 상태/의미 기반 정책 | 네트워크 레벨 보호 정책 |
| Cache | Shared | Key / TTL / consistency 전략 | Redis capacity / HA |
| Kafka Lag / Messaging | Shared | 처리량, 멱등성, backpressure | Partition / Broker / Scaling |
| Observability | Shared | Instrumentation / correlation | Collector / retention / alert |

대표 판단 예시:

```text
Double Booking
→ Backend Root Cause
→ Unique Constraint / Transaction / Lock이 Primary Fix
→ Pod scale-out은 해결책이 아님

Connection Pool Exhaustion
→ Shared
→ 긴 Transaction 축소가 Root Fix일 수 있음
→ Pool size / DB capacity / metrics는 Platform Mitigation

CPU Throttling
→ Infra Primary
→ requests / limits 설계가 핵심
→ 앱 최적화는 Secondary일 수 있음
```

---

# 8. Experiment 운영 정책

## 전체 정책 — FINAL

- 전체 Experiment 후보는 **86개**를 카탈로그로 기록한다.
- 그중 **40개를 실제 구현 대상으로 고정**한다.
- 나머지는 BACKLOG다.
- 핵심 40개 완료 전 범위를 무작정 늘리지 않는다.
- 각 Experiment는 독립적으로 완료 가능한 작은 학습 산출물이어야 한다.

## 공통 Experiment Template — FINAL

```markdown
# EXP-XX <Name>

## Scenario
의도적으로 어떤 조건을 만든다.

## Hypothesis
어떤 문제가 왜 발생할 것으로 예상하는가.

## Expected / Actual
예상과 실제를 비교한다.

## Observation
Metrics / Logs / Traces / DB / Kafka 상태를 기록한다.

## Root Cause
근본 원인을 설명한다.

## Application Fix
백엔드에서 가능한 근본 해결/완화책.

## Platform Mitigation
Infra / K8s / Istio / AWS에서 가능한 완화책.

## Before / After
P50/P95/P99, Error Rate, Throughput, Lock Wait, Pool, Lag 등.

## Responsibility
Root Cause Layer / Primary Fix / Secondary Mitigation.

## Lesson / Trade-off
왜 이 해결책을 선택했는지와 부작용.
```

## Experiment 완료 조건 — FINAL

1. 문제를 반복해서 재현할 수 있다.
2. 자동화 스크립트 또는 명확한 재현 절차가 있다.
3. Metrics / Logs / Traces 중 최소 하나 이상으로 현상을 관측한다.
4. Root Cause를 코드 / SQL / DB / 설정 수준에서 설명할 수 있다.
5. Backend Fix와 Infra Mitigation을 분리한다. 해당하지 않으면 `N/A`로 명시한다.
6. 수정 전/후 수치 또는 명확한 동작 차이를 기록한다.
7. Trade-off / 재발 방지 포인트를 기록한다.

---

# 9. 실제 구현할 핵심 Experiment 40개 — FINAL

## P1 — Monolith / DB · JPA (6)

| # | ID | Experiment | Primary | 핵심 실패 시나리오 |
|---:|---|---|---|---|
| 1 | BE-01 | JPA N+1 | Backend | 공연/예약 목록 조회에서 Query 수 폭증 |
| 2 | BE-03 | Index 없음 | Backend | 예약 검색이 Full Scan으로 전환 |
| 3 | BE-04 | 잘못된 Composite Index | Backend | Index가 존재하지만 활용되지 않음 |
| 4 | BE-05 | 불필요한 Entity Loading | Backend | 불필요한 연관 로딩으로 DB/Heap 부담 |
| 5 | BE-06 | Long Transaction | Backend | 외부 호출/긴 로직으로 Connection 장시간 점유 |
| 6 | BE-15 | Isolation Anomaly | Backend | Non-repeatable Read / Phantom Read |

## P2 — 동시성 / 금융 정합성 (12)

| # | ID | Experiment | Primary | 핵심 실패 시나리오 |
|---:|---|---|---|---|
| 7 | BE-07 | Lost Update | Backend | 동시에 상태/Wallet 수정 후 업데이트 유실 |
| 8 | BE-08 | Double Booking | Backend | 동일 좌석 예약이 동시에 성공 |
| 9 | BE-09 | Deadlock | Backend | 서로 다른 순서로 Row Lock 획득 |
| 10 | BE-10 | Optimistic Lock 충돌 | Backend | 고경합에서 Version conflict / Retry 폭증 |
| 11 | BE-11 | Duplicate Payment | Backend | Client Retry로 이중 결제 |
| 12 | BE-13 | Wallet Overdraft | Backend | 동시 출금으로 잔액 초과 사용 |
| 13 | BE-14 | Ledger 불일치 | Backend | Debit/Credit 또는 원장/잔액 불일치 |
| 14 | FIN-02 | Payment 성공 후 응답 유실 | Backend | PG 성공 후 결과를 모르는 Unknown State |
| 15 | FIN-03 | PG 성공 / 내부 DB 실패 | Backend | 외부 성공과 내부 상태 불일치 |
| 16 | FIN-04 | Refund 성공 / Reservation 업데이트 실패 | Backend | 환불과 예약 상태 불일치 |
| 17 | FIN-08 | Settlement Batch 중간 장애 | Backend | 처리 도중 프로세스 종료 |
| 18 | FIN-09 | Settlement 중복 실행 | Backend | 재실행으로 동일 정산 중복 |

## P3 — 트래픽 / Resilience (9)

| # | ID | Experiment | Primary | 핵심 실패 시나리오 |
|---:|---|---|---|---|
| 19 | MIX-01 | Connection Pool 고갈 | Shared | Hikari active/pending 포화 |
| 20 | MIX-02 | DB Connection 폭증 | Shared | Pod 증가가 RDS connection 폭증 유발 |
| 21 | MIX-03 | Timeout Chain | Shared | Gateway/App/DB timeout 불일치 |
| 22 | MIX-04 | Retry Storm | Shared | 다층 Retry가 요청량 증폭 |
| 23 | MIX-05 | Cascading Failure | Shared | Payment 장애가 상위 서비스로 전파 |
| 24 | MIX-06 | Cache Stampede | Shared | TTL 만료 순간 DB 요청 집중 |
| 25 | MIX-08 | Cache Hot Key | Shared | 특정 공연/좌석 Key 집중 |
| 26 | MIX-13 | Thundering Herd | Shared | 티켓 오픈 순간 대량 요청 동시 진입 |
| 27 | MIX-16 | Graceful Shutdown 실패 | Shared | 배포/종료 중 요청/메시지 유실 |

## P4 — MSA / Event-Driven (7)

| # | ID | Experiment | Primary | 핵심 실패 시나리오 |
|---:|---|---|---|---|
| 28 | EVT-03 | Consumer duplicate | Backend | 동일 이벤트 여러 번 소비 |
| 29 | EVT-04 | Consumer crash after DB commit | Backend | DB 반영 후 offset commit 전 장애 |
| 30 | EVT-05 | 이벤트 순서 역전 | Backend | Completed/Cancelled 순서 역전 |
| 31 | EVT-07 | Consumer Lag | Shared | 소비 지연 누적 |
| 32 | EVT-08 | Poison Message | Shared | 특정 이벤트 반복 실패 |
| 33 | EVT-09 | Schema 변경으로 Consumer 장애 | Backend | Producer/Consumer 호환성 문제 |
| 34 | EVT-10 | DB commit / Kafka publish 불일치 | Backend | Dual-write 불일치 |

## P5 — Platform / Observability (6)

| # | ID | Experiment | Primary | 핵심 실패 시나리오 |
|---:|---|---|---|---|
| 35 | INF-01 | CPU Throttling | Infra | CPU limit이 너무 낮아 latency 증가 |
| 36 | INF-02 | OOMKilled | Infra + Backend | Heap/off-heap/container limit 문제 |
| 37 | INF-04 | Readiness 잘못 설정 | Infra | 준비 안 된 Pod로 트래픽 전달 |
| 38 | INF-05 | Liveness 오용 | Infra | 느린 Pod 반복 재시작 |
| 39 | INF-16 | Bad Deployment | Infra | 신규 버전 error rate 급증 |
| 40 | OBS-01 | Slow API root cause 찾기 | Shared | P99 상승 원인을 trace로 식별 |

---

# 10. 전체 Experiment 카탈로그 — 86개

`IMPLEMENT`는 핵심 40개에 포함, `BACKLOG`는 후속 후보다.

## 10.1 Backend 중심 (18)

| ID | 상태 | Experiment | Primary | Focus | Failure Scenario |
|---|---|---|---|---|---|
| BE-01 | IMPLEMENT | JPA N+1 | Backend | Fetch Join, EntityGraph, Batch Fetch | 목록 조회에서 Query 수 폭증 |
| BE-02 | BACKLOG | 잘못된 Pagination | Backend | Query/Pagination 설계 | JOIN + pagination으로 성능/메모리 문제 |
| BE-03 | IMPLEMENT | Index 없음 | Backend | EXPLAIN, Index | Full Scan으로 지연 증가 |
| BE-04 | IMPLEMENT | 잘못된 Composite Index | Backend | Composite Index | 조건/정렬에서 Index 미활용 |
| BE-05 | IMPLEMENT | 불필요한 Entity Loading | Backend | Projection, DTO | DB/Heap 부담 증가 |
| BE-06 | IMPLEMENT | Long Transaction | Backend | Transaction Boundary | Connection 장시간 점유 |
| BE-07 | IMPLEMENT | Lost Update | Backend | Lock, Atomic Update | 동시 수정 유실 |
| BE-08 | IMPLEMENT | Double Booking | Backend | Constraint, Lock | 동일 좌석 중복 성공 |
| BE-09 | IMPLEMENT | Deadlock | Backend | Lock Ordering, Retry | 교착 상태 |
| BE-10 | IMPLEMENT | Optimistic Lock 충돌 | Backend | Version, Retry | 고경합 충돌 폭증 |
| BE-11 | IMPLEMENT | Duplicate Payment | Backend | Idempotency Key | 중복 결제 |
| BE-12 | BACKLOG | Duplicate Refund | Backend | Idempotency | 환불 재시도로 중복 환불 |
| BE-13 | IMPLEMENT | Wallet Overdraft | Backend | Isolation, Atomic Update | 잔액 초과 사용 |
| BE-14 | IMPLEMENT | Ledger 불일치 | Backend | Atomicity, Invariant | Debit/Credit 또는 원장/잔액 불일치 |
| BE-15 | IMPLEMENT | Isolation Anomaly | Backend | Isolation Level | Non-repeatable / Phantom Read |
| BE-16 | BACKLOG | LazyInitializationException | Backend | Persistence Context | Transaction 외 Lazy 접근 실패 |
| BE-17 | BACKLOG | Huge Response | Backend | Pagination, Streaming | 메모리/네트워크 부담 |
| BE-18 | BACKLOG | 잘못된 API Retry 처리 | Backend | API semantics | 비멱등 POST 중복 상태 변경 |

## 10.2 Backend + Infra 경계 (18)

| ID | 상태 | Experiment | Primary | Focus | Failure Scenario |
|---|---|---|---|---|---|
| MIX-01 | IMPLEMENT | Connection Pool 고갈 | Shared | TX 축소 + Pool 관측 | Hikari pending 증가 |
| MIX-02 | IMPLEMENT | DB Connection 폭증 | Shared | Pool + HPA/DB Capacity | Scale-out이 DB connection 폭증 유발 |
| MIX-03 | IMPLEMENT | Timeout Chain | Shared | Timeout Budget | 계층별 timeout 불일치 |
| MIX-04 | IMPLEMENT | Retry Storm | Shared | Backoff/Jitter | Retry가 장애를 증폭 |
| MIX-05 | IMPLEMENT | Cascading Failure | Shared | Circuit Breaker, Bulkhead | 하위 장애가 상위로 전파 |
| MIX-06 | IMPLEMENT | Cache Stampede | Shared | TTL jitter, Lock | TTL 동시 만료 시 DB 폭격 |
| MIX-07 | BACKLOG | Cache Penetration | Shared | Negative Cache, Rate Limit | 없는 key 반복 조회 |
| MIX-08 | IMPLEMENT | Cache Hot Key | Shared | Key 분산, Redis Scaling | 단일 key 집중 |
| MIX-09 | BACKLOG | Kafka Consumer Lag | Shared | Consumer + Partition | 생산보다 소비가 느림 |
| MIX-10 | BACKLOG | Poison Message | Shared | Retry/DLQ | 특정 메시지 반복 실패 |
| MIX-11 | BACKLOG | Duplicate Event | Shared | Inbox, Idempotency | 동일 이벤트 재처리 |
| MIX-12 | BACKLOG | Event Loss | Shared | Transactional Outbox | DB commit 후 publish 전 장애 |
| MIX-13 | IMPLEMENT | Thundering Herd | Shared | Queue, Rate Limit, Scaling | 오픈 순간 대량 요청 |
| MIX-14 | BACKLOG | Slow Downstream | Shared | Async, Timeout | Mock PG 지연이 상위에 전파 |
| MIX-15 | BACKLOG | API Rate Explosion | Shared | Quota, Gateway/WAF | 비정상 대량 요청 |
| MIX-16 | IMPLEMENT | Graceful Shutdown 실패 | Shared | Shutdown, Drain | In-flight 요청/메시지 유실 |
| MIX-17 | BACKLOG | Pod Scale-out 지연 | Shared | Queue, HPA/KEDA | 트래픽 증가보다 scale-out 늦음 |
| MIX-18 | BACKLOG | Cold Start | Shared | Warmup, Startup Probe | 초기화 전 트래픽 수신 |

## 10.3 Infra 중심 (18)

| ID | 상태 | Experiment | Primary | Focus | Failure Scenario |
|---|---|---|---|---|---|
| INF-01 | IMPLEMENT | CPU Throttling | Infra | requests/limits | 낮은 CPU limit로 latency 증가 |
| INF-02 | IMPLEMENT | OOMKilled | Infra + Backend | JVM/K8s Memory | Heap/off-heap/limit 불일치 |
| INF-03 | BACKLOG | HPA 오작동 | Infra | HPA Metric | 잘못된 metric으로 과소/과대 확장 |
| INF-04 | IMPLEMENT | Readiness 잘못 설정 | Infra | Readiness Probe | 준비 안 된 Pod로 요청 전달 |
| INF-05 | IMPLEMENT | Liveness 오용 | Infra | Liveness Probe | 재시작 루프가 장애 증폭 |
| INF-06 | BACKLOG | Node Failure | Infra | Rescheduling, PDB | Worker EC2 장애 |
| INF-07 | BACKLOG | AZ 장애 | Infra | Multi-AZ | 특정 AZ 상실 |
| INF-08 | BACKLOG | Kafka Broker Failure | Infra | Replication, ISR | Broker 장애 |
| INF-09 | BACKLOG | Redis Failure | Infra | HA / Failover | Redis node 장애 |
| INF-10 | BACKLOG | Network Latency | Infra | Timeout resilience | 서비스간 지연 주입 |
| INF-11 | BACKLOG | Packet Loss | Infra | Network resilience | 간헐적 통신 실패 |
| INF-12 | BACKLOG | DNS 장애 | Infra | DNS resilience | CoreDNS / discovery 장애 |
| INF-13 | BACKLOG | Disk Full | Infra | Storage monitoring | Kafka/Node 디스크 포화 |
| INF-14 | BACKLOG | Log Explosion | Infra | Sampling/Retention | 로그로 저장/네트워크 포화 |
| INF-15 | BACKLOG | Certificate Expiry | Infra | Cert lifecycle | mTLS 인증서 문제 |
| INF-16 | IMPLEMENT | Bad Deployment | Infra | Canary, Rollback | 신규 버전 오류율 증가 |
| INF-17 | BACKLOG | Dependency Region Failure | Infra | Failover/Degradation | 외부/리전 의존성 장애 |
| INF-18 | BACKLOG | Observability 장애 | Infra | Collector HA | Metric/Trace 수집 손실 |

## 10.4 금융 도메인 (10)

| ID | 상태 | Experiment | Primary | Focus | Failure Scenario |
|---|---|---|---|---|---|
| FIN-01 | BACKLOG | 동일 Payment 두 번 요청 | Backend | Idempotency | 동일 구매 의도 중복 요청 |
| FIN-02 | IMPLEMENT | Payment 성공 후 응답 유실 | Backend | Reconciliation, Query | PG 성공 후 Unknown State |
| FIN-03 | IMPLEMENT | PG 성공 / 내부 DB 실패 | Backend | Saga/Compensation | 외부/내부 상태 불일치 |
| FIN-04 | IMPLEMENT | Refund 성공 / Reservation 업데이트 실패 | Backend | Saga/Compensation | 환불/예약 상태 불일치 |
| FIN-05 | BACKLOG | Wallet 동시 출금 | Backend | Concurrency Control | 동시 결제가 잔액 초과 |
| FIN-06 | BACKLOG | Wallet 잔액 vs Ledger 불일치 | Backend | Reconciliation | 파생 잔액과 원장 불일치 |
| FIN-07 | BACKLOG | Debit만 저장되고 Credit 실패 | Backend | Atomicity | Double-entry invariant 위반 |
| FIN-08 | IMPLEMENT | Settlement Batch 중간 장애 | Backend | Restartability | 중간 프로세스 종료 |
| FIN-09 | IMPLEMENT | Settlement 중복 실행 | Backend | Idempotent Batch | 동일 정산 중복 지급 |
| FIN-10 | BACKLOG | Event 중복으로 정산 2번 | Backend | Exactly-once illusion | 중복 이벤트가 금전 상태 재변경 |

## 10.5 Kafka / Event-Driven (12)

| ID | 상태 | Experiment | Primary | Focus | Failure Scenario |
|---|---|---|---|---|---|
| EVT-01 | BACKLOG | Producer publish 실패 | Shared | Retry / failure handling | 발행 자체 실패 |
| EVT-02 | BACKLOG | Producer 중복 publish | Shared | Idempotent Producer | 재시도로 중복 발행 |
| EVT-03 | IMPLEMENT | Consumer duplicate | Backend | Inbox / Idempotent Consumer | 동일 이벤트 다회 소비 |
| EVT-04 | IMPLEMENT | Consumer crash after DB commit | Backend | At-least-once | DB 반영 후 offset 전 장애 |
| EVT-05 | IMPLEMENT | 이벤트 순서 역전 | Backend | Ordering, State Machine | 이벤트가 기대 순서와 다르게 도착 |
| EVT-06 | BACKLOG | 특정 Partition Hotspot | Shared | Partition Key | 부하 편중 |
| EVT-07 | IMPLEMENT | Consumer Lag | Shared | Backpressure, Scaling | 지연 누적 |
| EVT-08 | IMPLEMENT | Poison Message | Shared | DLQ | 반복 실패 이벤트 |
| EVT-09 | IMPLEMENT | Schema 변경으로 Consumer 장애 | Backend | Schema Evolution | 버전 호환성 실패 |
| EVT-10 | IMPLEMENT | DB commit / Kafka publish 불일치 | Backend | Outbox | Dual-write inconsistency |
| EVT-11 | BACKLOG | Outbox event 중복 publish | Backend | Inbox, Idempotency | publisher retry로 중복 이벤트 |
| EVT-12 | BACKLOG | Saga 중간 실패 | Backend | Compensation | 일부 성공 후 후속 실패 |

## 10.6 Observability (10)

| ID | 상태 | Experiment | Primary | Focus | Failure Scenario |
|---|---|---|---|---|---|
| OBS-01 | IMPLEMENT | Slow API root cause 찾기 | Shared | Tempo / OpenTelemetry | P99 상승 원인을 trace로 식별 |
| OBS-02 | BACKLOG | DB Query 병목 찾기 | Shared | Prometheus + MySQL | Query/Lock 지연 식별 |
| OBS-03 | BACKLOG | Connection Pool Saturation | Shared | Micrometer | Pool과 latency 상관관계 |
| OBS-04 | BACKLOG | Kafka Lag 장애 관측 | Shared | Prometheus | Lag와 처리 지연 대시보드 |
| OBS-05 | BACKLOG | Error 요청 추적 | Shared | Loki + Tempo | Trace ID로 로그/trace 연결 |
| OBS-06 | BACKLOG | 특정 User 요청 전체 추적 | Shared | OpenTelemetry | 서비스 경계 전체 흐름 |
| OBS-07 | BACKLOG | 로그 폭증 | Shared | Loki / Retention | 저장/검색/비용 영향 |
| OBS-08 | BACKLOG | Sampling 때문에 원인 누락 | Shared | Trace Sampling | 희귀 오류 trace 누락 |
| OBS-09 | BACKLOG | SLO 위반 감지 | Shared | Prometheus / SLO | latency/error budget 판단 |
| OBS-10 | BACKLOG | Alert Fatigue | Shared | Alertmanager | 중요 신호가 과도한 알람에 묻힘 |

---

# 11. ML / MLOps 결정

## 첫 모델 — FINAL

**티켓 오픈 수요 / Peak RPS 예측**

후보 모델:

- XGBoost
- LightGBM
- scikit-learn 계열

예시 Feature:

- 공연/아티스트 인기도
- Venue capacity
- Ticket price
- Day of week / hour
- 과거 공연 RPS
- Page view
- Wishlist / 관심 수
- 티켓 오픈까지 남은 시간

Target 예시:

- 오픈 후 5분 Peak RPS
- 첫 10분 판매량

Data / Pipeline:

```text
Ticket / Traffic Data
       ↓
       S3
       ↓
Kubeflow Pipeline
Preprocess
       ↓
Train
       ↓
Evaluate
       ↓
Model Artifact
       ↓
KServe
       ↓
Prediction API
```

## 두 번째 모델 — BACKLOG

Fraud Detection / 이상 결제 행동 탐지.

## Optional Expansion

- Katib hyperparameter tuning
- LLM / LLMOps

LLM은 일반 ML lifecycle 완료 이후 검토한다.

---

# 12. 시간 계획

## 계획 범위 — FINAL 기준선

핵심 40 Experiment + 금융 범위 + MSA + MLOps까지 **약 340~440시간**을 안전한 계획 범위로 둔다.

Infra/Kubernetes/Observability 경험이 이미 있다는 전제를 반영한다.

| 주당 투자 | 예상 달력 기간 |
|---:|---:|
| 8h | 약 10~13개월 |
| 12h | 약 7~9개월 |
| 15h | 약 5~7개월 |
| 20h | 약 4~5개월 |

## 중간 종료 지점

### Checkpoint A — 약 100~140h

- Monolith
- DB/JPA
- Transaction
- 핵심 동시성

여기까지도 독립적인 백엔드 학습 프로젝트로 성립한다.

### Checkpoint B — 약 220~300h

- MSA
- Kafka
- 분산 시스템 패턴
- 주요 트래픽/Resilience 문제

프로젝트의 핵심 목표 대부분을 달성한다.

### Checkpoint C — 약 340~440h

- 핵심 40 Experiment 완료
- 금융 심화
- Kubeflow / KServe

최종 완성 지점이다.

각 Experiment는 독립 산출물로 남겨, 중간 중단 시에도 완료된 학습 결과가 남게 한다.

---

# 13. Definition of Done — FINAL

프로젝트의 최종 완료 조건:

1. Spring Boot Monolith에서 Ticket / Reservation / Payment / Refund / Wallet / Ledger / Settlement 핵심 흐름이 동작한다.
2. Vite + React 기반 최소 UI로 주요 사용자 흐름을 확인할 수 있다.
3. 핵심 Experiment 40개를 재현, 관측, 분석, 수정하고 보고서로 남긴다.
4. Monolith → MSA 전환 이유를 실제 측정/장애 경험으로 설명할 수 있다.
5. Saga, Outbox, Inbox/Idempotent Consumer, Retry/DLQ, Event Ordering을 실제 실패 시나리오로 구현한다.
6. AWS + self-managed Kubernetes(No EKS) + GitHub Actions + ArgoCD + Istio 환경에서 운영한다.
7. Prometheus/Grafana/Loki/Tempo/OpenTelemetry를 장애 분석에 사용한다.
8. 일반 ML 모델을 Kubeflow Pipeline으로 학습하고 KServe로 서빙한다.
9. README / Architecture / Experiment Index에서 전체 진화 과정과 Before/After 결과를 탐색할 수 있다.

---

# 14. AI가 프로젝트를 해석할 때 지켜야 할 우선순위

어떤 구현 선택이 충돌할 때 다음 순서로 판단한다.

```text
1. 학습 목표
2. 데이터/금전 정합성
3. Experiment 재현 가능성
4. 장애 관측 가능성
5. 단순한 설계
6. 성능
7. 편의성 / 화려함
```

예를 들어 더 복잡한 MSA가 “멋있어 보인다”는 이유는 Monolith-first 원칙을 뒤집을 근거가 아니다.

---

# 15. Copy-ready Context

이 프로젝트는 DevOps/MLOps 엔지니어가 백엔드를 학습하기 위한 **고트래픽 티켓팅 + 금융 정합성 프로젝트**다. Frontend는 Vite + React, Backend는 Spring Boot/Spring Cloud를 사용한다. 초기 금융 범위는 Payment, Refund, Wallet, Ledger, Settlement이며 Resale/Escrow는 후순위다. 실제 PG/실제 돈은 사용하지 않는다. Spring Boot Monolith로 시작해 N+1, Index, Transaction, Lock, Connection Pool, Timeout, Cache, Double Booking, Duplicate Payment, Wallet/Ledger 정합성 문제를 의도적으로 재현한다. 이후 실제 문제를 근거로 Spring Cloud 기반 MSA와 Kafka Event-Driven 구조로 전환하며 Saga, Outbox, Inbox/Idempotency, Retry/DLQ, Event Ordering을 학습한다. AWS(VPC, RDS MySQL, EC2 Kafka, Redis, SQS), self-managed Kubernetes(No EKS), Prometheus/Grafana/Loki/Tempo/OpenTelemetry, ArgoCD, Istio, GitHub Actions를 사용한다. 전체 Experiment 86개를 카탈로그로 관리하고 그중 40개를 실제 구현한다. 각 실험은 재현 → 관측 → RCA → Backend Fix / Platform Mitigation → Before/After → Trade-off를 기록한다. MLOps 단계에서는 먼저 XGBoost/LightGBM/scikit-learn 계열의 티켓 수요/Peak RPS 예측 모델을 Kubeflow Pipeline으로 학습하고 KServe로 서빙한다. 프로젝트의 핵심 산출물은 기능 개수가 아니라 장애 연구 기록과 Monolith → MSA → MLOps로의 진화 근거다.
