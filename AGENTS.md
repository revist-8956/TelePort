# AGENTS.md

이 문서는 이 저장소에서 작업하는 AI 에이전트를 위한 실행 규칙이다. 프로젝트의 **왜(Why), 도메인, 범위, 최종 의사결정은 `CONTEXT.md`가 Source of Truth**다.

## 1. 가장 먼저 읽을 것

작업 전에 반드시 다음 순서로 확인한다.

1. `CONTEXT.md`
2. 현재 작업 대상 디렉터리의 코드와 테스트
3. 관련 Experiment 문서가 존재하면 해당 문서
4. 이 `AGENTS.md`

`CONTEXT.md`의 `FINAL` 결정은 사용자의 명시적 요청 없이 변경하지 않는다.

---

## 2. 예상 저장소 구조

```text
/
├─ README.md
├─ AGENTS.md
├─ CONTEXT.md
├─ backend/            # Spring Boot 애플리케이션
├─ frontend/           # Vite + React
├─ experiments/        # 장애 재현 및 분석 보고서
├─ infra/              # k8s / ArgoCD / Istio / observability / AWS 관련 정의
├─ ml/                 # Kubeflow pipeline, training, KServe 관련 코드
└─ docs/               # 아키텍처 / ADR / 관측성 / Experiment 결과 / MLOps 결과
```

실제 구조가 이미 존재하면 **기존 구조를 우선**한다. 단순히 이 문서와 다르다는 이유로 대규모 이동/리팩터링을 하지 않는다.

`docs/`는 최종 기술 산출물을 모으는 공간이다. 최소한 다음 범주의 결과를 문서화한다.

- Grafana Dashboard
- Tempo Distributed Trace
- 주요 Experiment Before / After
- Monolith → MSA 전환 과정
- Kubeflow / KServe MLOps Pipeline

개별 Experiment의 상세 재현 절차와 원본 evidence는 `experiments/`에 두고, `docs/`에는 사람이 프로젝트의 기술적 성과를 빠르게 이해할 수 있도록 정리된 결과와 링크를 둔다.

---

## 3. 빌드 / 실행 / 테스트 명령 규칙

### Backend

Backend build tool은 저장소에 이미 존재하는 wrapper를 따른다. **Gradle과 Maven을 임의로 전환하지 않는다.**

Gradle wrapper가 있는 경우:

```bash
cd backend
./gradlew bootRun
./gradlew test
./gradlew build
```

Maven wrapper가 있는 경우:

```bash
cd backend
./mvnw spring-boot:run
./mvnw test
./mvnw verify
```

두 wrapper가 모두 없으면 새로운 build system을 임의로 추가하지 말고, 현재 프로젝트 설정을 확인한 뒤 최소 변경으로 진행한다.

### Frontend

패키지 매니저는 lockfile을 기준으로 결정한다. 이미 `package-lock.json`, `pnpm-lock.yaml`, `yarn.lock` 중 하나가 존재하면 그것을 따른다.

npm 기준 기본 명령:

```bash
cd frontend
npm ci
npm run dev
npm run build
npm run lint
```

테스트 스크립트가 정의되어 있으면:

```bash
npm test
```

테스트 스크립트가 없는 프로젝트에 임의의 테스트 프레임워크를 추가하지 않는다. 필요 시 변경 이유를 먼저 문서화한다.

### Infrastructure / ML

실제 명령은 해당 디렉터리 README 또는 스크립트를 우선한다. 클러스터/클라우드에 영향을 주는 명령은 로컬 정적 검증보다 뒤에 수행한다.

---

## 4. 구현 원칙

### 4.1 Monolith First

프로젝트는 **처음부터 MSA로 시작하지 않는다.**

Spring Boot Monolith에서 먼저 다음을 경험해야 한다.

- JPA / Query 문제
- Index 설계
- Transaction 경계
- Connection Pool
- Lock / Isolation
- Race Condition
- Cache
- Timeout / Retry
- Payment / Wallet / Ledger 정합성

MSA는 실제 문제를 관측하고 설명할 수 있게 된 이후 도입한다.

### 4.2 정합성 문제는 인프라로 덮지 않는다

다음과 같은 문제의 Primary Fix는 Backend다.

- Double Booking
- Lost Update
- Duplicate Payment
- Wallet Overdraft
- Ledger invariant 위반
- Idempotency 실패
- Event ordering에 따른 잘못된 상태 전이

Pod를 늘리거나 Redis/Kafka를 추가하는 것으로 정합성 버그를 해결했다고 간주하지 않는다.

### 4.3 Backend Fix와 Platform Mitigation을 분리한다

Timeout / Retry / Cache / Messaging / Connection Pool / Cascading Failure처럼 경계에 있는 문제는 반드시 아래를 나눠 기록한다.

```text
Root Cause Layer
Primary Fix
Application Fix
Platform Mitigation
Trade-off
```

### 4.4 Observability는 기능이 아니라 진단 도구다

Prometheus/Grafana/Loki/Tempo/OpenTelemetry를 “설치 완료”로 끝내지 않는다. Experiment의 원인을 찾는 데 실제로 사용한다.

### 4.5 프런트엔드는 학습 목표를 침범하지 않게 작게 유지한다

Frontend는 **Vite + React**를 사용한다.

목적은 다음에 한정한다.

- 공연/회차/좌석 조회
- 좌석 예약
- 결제/환불 흐름 확인
- Wallet / Ledger / Settlement 상태 확인
- Experiment 시나리오를 사람이 확인할 최소 UI

화려한 UI, 복잡한 상태관리, 디자인 시스템 구축은 핵심 목표가 아니다. 백엔드/분산시스템 실험보다 우선하지 않는다.

---

## 5. 코드 스타일 / 설계 규칙

### Backend

- Controller에 비즈니스 로직을 넣지 않는다.
- Transaction 경계를 의식적으로 설계한다. 외부 네트워크 호출을 긴 DB Transaction 내부에 무심코 넣지 않는다.
- Entity를 API Response로 직접 노출하지 않는다.
- 데이터 정합성은 가능한 경우 애플리케이션 검증뿐 아니라 DB Constraint로도 보호한다.
- 멱등성이 필요한 API/Consumer는 멱등성 키 또는 비즈니스 식별자를 명시한다.
- Retry를 추가할 때 반드시 대상 작업의 멱등성을 확인한다.
- 동시성 문제를 `synchronized` 하나로 끝내지 않는다. DB/분산 환경에서의 의미를 확인한다.
- Lock을 추가할 때 Lock 범위, 경합도, Timeout, Deadlock 가능성을 함께 기록한다.
- 성능 최적화는 측정 전 추측으로 적용하지 않는다.

### Frontend

- React component는 가능한 작고 명확하게 유지한다.
- Backend API 상태와 UI 상태를 혼동하지 않는다.
- 핵심 도메인 규칙을 Frontend에서 권위 있게 판단하지 않는다. 서버가 Source of Truth다.
- Experiment를 위해 의도적으로 발생시키는 backend failure를 UI에서 숨기지 않는다. HTTP status / 오류 메시지를 확인할 수 있게 한다.

### Messaging

- Kafka delivery를 exactly-once라고 가정하지 않는다.
- Consumer는 기본적으로 중복 전달 가능성을 고려한다.
- 이벤트에 명확한 event id / aggregate id / occurred-at / schema version을 고려한다.
- DB 상태 변경 + 이벤트 발행의 dual-write 문제를 무시하지 않는다.

---

## 6. 테스트 규칙

새로운 로직에는 가능한 범위에서 테스트를 추가한다.

우선순위:

1. Domain / Service unit test
2. Repository / Transaction integration test
3. API integration test
4. Concurrency test
5. Experiment 전용 load / failure test

동시성/금융 Experiment는 단일 정상 케이스 테스트만으로 완료 처리하지 않는다.

DB/Redis/Kafka의 실제 동작이 중요한 테스트는 mock만으로 끝내지 말고 integration test를 우선 고려한다.

---

## 7. Experiment 작업 규칙

전체 Experiment는 `CONTEXT.md`의 카탈로그를 따른다. 그중 `IMPLEMENT`로 지정된 40개가 필수 구현 대상이다.

각 Experiment는 가능하면 다음 구조를 사용한다.

```text
experiments/
└─ <ID>-<slug>/
   ├─ README.md
   ├─ load-test/        # 필요 시
   ├─ manifests/        # 필요 시 장애 주입 설정
   └─ evidence/         # 결과 이미지/요약 데이터 등
```

각 보고서에는 최소한 다음 내용을 남긴다.

```markdown
# <ID> <Experiment Name>

## Scenario
## Hypothesis
## Reproduction
## Expected / Actual
## Observation
## Root Cause
## Application Fix
## Platform Mitigation
## Before / After
## Responsibility
## Trade-off / Lesson
```

완료 조건:

- 재현 절차가 반복 가능하다.
- Metrics / Logs / Traces 중 최소 하나 이상으로 현상을 관측한다.
- 원인을 코드/쿼리/설정 수준에서 설명한다.
- Backend Fix와 Infra Mitigation을 구분한다.
- Before/After를 남긴다.
- Trade-off를 기록한다.

---

## 8. 금지 / 주의 구역

사용자의 명시적 요청 없이 다음을 하지 않는다.

- EKS 도입
- 실제 PG / 실제 카드 / 실제 금전 사용
- 초기 범위에 Resale / Escrow 추가
- 처음부터 모든 서비스를 MSA로 분리
- 핵심 40 Experiment 완료 전에 범위를 무작정 늘림
- LLM을 MLOps 첫 모델로 선택
- ELK / Terraform / Ansible을 필수 선행조건으로 변경
- 프런트엔드를 프로젝트 중심으로 확장
- 정합성 버그를 단순 scale-out으로 “해결” 처리
- 실패 재현 없이 최적화부터 적용

---

## 9. 의사결정 변경 방법

기존 `FINAL` 결정을 변경해야 할 경우:

1. 변경하려는 결정 확인
2. 변경 이유와 학습상의 이점 정리
3. 기존 Experiment / Phase에 미치는 영향 확인
4. 사용자의 명시적 승인을 받은 뒤 `CONTEXT.md` 갱신

새로운 구현 세부사항은 `FINAL` 결정과 충돌하지 않는 범위에서 자율적으로 선택할 수 있다.

---

## 10. 작업 완료 전 체크

코드 변경 후 최소한 다음을 확인한다.

```text
[ ] 기존 범위/아키텍처 원칙과 충돌하지 않는가?
[ ] 관련 테스트가 통과하는가?
[ ] 동시성/정합성 변경이라면 실패 케이스도 검증했는가?
[ ] Retry/Timeout을 추가했다면 중복 실행 영향을 검토했는가?
[ ] Experiment라면 Before/After와 책임 경계를 기록했는가?
[ ] 불필요하게 Infra 또는 Domain 범위를 확장하지 않았는가?
```
