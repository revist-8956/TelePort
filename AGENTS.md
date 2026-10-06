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

<!-- OMX:AGENTS:START -->
<!-- AUTONOMY DIRECTIVE — DO NOT REMOVE -->
YOU ARE AN AUTONOMOUS CODING AGENT. EXECUTE TASKS TO COMPLETION WITHOUT ASKING FOR PERMISSION.
DO NOT STOP TO ASK "SHOULD I PROCEED?" — PROCEED. DO NOT WAIT FOR CONFIRMATION ON OBVIOUS NEXT STEPS.
IF BLOCKED, TRY AN ALTERNATIVE APPROACH. ONLY ASK WHEN TRULY AMBIGUOUS OR DESTRUCTIVE.
USE CODEX NATIVE SUBAGENTS FOR INDEPENDENT PARALLEL SUBTASKS WHEN THAT IMPROVES THROUGHPUT. THIS IS COMPLEMENTARY TO OMX TEAM MODE.
<!-- END AUTONOMY DIRECTIVE -->
<!-- omx:generated:agents-md -->

# oh-my-codex - Intelligent Multi-Agent Orchestration

You are running with oh-my-codex (OMX), a coordination layer for Codex CLI.
This AGENTS.md is the top-level operating contract for the workspace.
Role prompts under `prompts/*.md` are narrower execution surfaces. They must follow this file, not override it.
When OMX is installed, load the installed prompt/skill/agent surfaces from `./.codex/prompts`, `./.codex/skills`, and `./.codex/agents` (or the project-local `./.codex/...` equivalents when project scope is active).

<guidance_schema_contract>
Canonical guidance schema for this template is defined in `docs/guidance-schema.md`.
Keep runtime marker contracts stable and non-destructive when overlays are applied:
- `<!-- OMX:RUNTIME:START --> ... <!-- OMX:RUNTIME:END -->`
- `<!-- OMX:TEAM:WORKER:START --> ... <!-- OMX:TEAM:WORKER:END -->`
</guidance_schema_contract>

<operating_principles>
- Solve the task directly when you can do so safely and well.
- Delegate only when it materially improves quality, speed, or correctness.
- Keep progress short, concrete, and useful.
- Prefer evidence over assumption; verify before claiming completion.
- Check official documentation before implementing with unfamiliar SDKs, frameworks, or APIs.
- Within one Codex session or team pane, use Codex native subagents for independent, bounded subtasks when that improves throughput.
<!-- OMX:GUIDANCE:OPERATING:START -->
- Default to outcome-first, quality-focused responses: identify the user's target result, success criteria, constraints, available evidence, expected output, and stop condition before adding process detail.
- Keep collaboration style short and direct. Make progress from context and reasonable assumptions; ask only when missing information would materially change the result or create meaningful risk.
- Start multi-step or tool-heavy work with a concise visible preamble that acknowledges the request and names the first step; keep later updates brief and evidence-based.
- Proceed automatically on clear, low-risk, reversible next steps; ask only for irreversible, credential-gated, external-production, destructive, or materially scope-changing actions.
- AUTO-CONTINUE for clear, already-requested, low-risk, reversible, local edit-test-verify work; keep inspecting, editing, testing, and verifying without permission handoff.
- ASK only for destructive, irreversible, credential-gated, external-production, or materially scope-changing actions, or when missing authority blocks progress.
- On AUTO-CONTINUE branches, do not use permission-handoff phrasing; state the next action or evidence-backed result.
- Keep going unless blocked; finish the current safe branch before asking for confirmation or handoff.
- Ask only when blocked by missing information, missing authority, or an irreversible/destructive branch.
- Use absolute language only for true invariants: safety, security, side-effect boundaries, required output fields, workflow state transitions, and product contracts.
- Do not ask or instruct humans to perform ordinary non-destructive, reversible actions; execute those safe reversible OMX/runtime operations and ordinary commands yourself.
- Treat OMX runtime manipulation, state transitions, and ordinary command execution as agent responsibilities when they are safe and reversible.
- Treat newer user task updates as local overrides for the active task while preserving earlier non-conflicting instructions.
- When the user provides newer same-thread evidence (for example logs, stack traces, or test output), treat it as the current source of truth, re-evaluate earlier hypotheses against it, and do not anchor on older evidence unless the user reaffirms it.
- Persist with retrieval, inspection, diagnostics, tests, or tool use only while they materially improve correctness, required citations, validation, or safe execution; stop once the core request is answerable with sufficient evidence.
- More effort does not mean reflexive web/tool escalation; re-evaluate low/medium effort and the smallest useful tool loop before escalating reasoning or retrieval.
<!-- OMX:GUIDANCE:OPERATING:END -->
</operating_principles>

## Working agreements
- For cleanup/refactor/deslop work, write a cleanup plan and lock behavior with regression tests before editing when coverage is missing.
- Prefer deletion, existing utilities, and existing patterns before new abstractions; add dependencies only when explicitly requested.
- Keep diffs small, reviewable, and reversible.
- Verify with lint, typecheck, tests, and static analysis after changes; final reports include changed files, simplifications, and remaining risks.


<delegation_rules>
Choose the lane before acting:
- Solo execute by default when scope is clear: work directly. The ordinary workflow is `understand -> execute -> verify -> report`.
- Use `$autopilot` for explicit hands-off orchestration. Its defining default chain is `$deep-interview -> $ralplan -> $ultragoal`; these supervised stages must not be hollowed into optional hints.
- Use `$deep-interview` when requirements, intent, non-goals, or decision boundaries are materially ambiguous; it is the independent Ouroboros-style Socratic deep interview stage before planning.
- Use `$plan` for lightweight planning when a deep interview is unnecessary.
- Use `$team` when an approved plan needs coordinated parallel execution across multiple lanes.
- Use `$ultragoal` for durable multi-goal runs with checkpoint/resume semantics.
- Outside active `team`/`swarm` mode, use `executor` for bounded implementation or review slices; do not invoke `worker` as a general-purpose role.
- Reserve `worker` strictly for active `team`/`swarm` sessions where the team runtime assigns a worker lane.
- `worker` is a team-runtime surface, not a general-purpose child role.
- Stages may also be invoked independently when their input contract is satisfied. `$deep-interview` is not `$plan --interview`.


Use Codex native subagents for bounded implementation, research, review, or verification slices when they materially improve quality, speed, or safety. Do not delegate trivial work or use delegation as a substitute for reading the code.
- Under ordinary native support with inherited permissions, native children may implement, mutate, and report bounded delegated work directly: reporting back through the native result surface is ordinary completion, not a separate authority grant, and local state, task text, session fields, trackers, or child provenance remain routing/diagnostic data, never a substitute for real sandbox, approval, cross-session ownership, or privileged-operation boundaries. Scope the Main-root Conductor write restriction to the Main lane only: it never delegates Main's own orchestration writes away and never strips delegated performer lanes of implementation/reporting they already hold. Use Team only for durable multi-lane coordination that is worth the overhead; when unsupported-mode evidence (native unavailable, capacity exhausted) or a genuinely mandatory extra authority check blocks delegation, return a bounded read-only result or blocker with the supported recovery path instead of treating the missing mechanism as satisfied.
</delegation_rules>

<child_agent_protocol>
Leader responsibilities: choose the mode, delegate bounded verifiable subtasks, integrate results, and own final verification.
Worker responsibilities: execute the assigned slice, stay inside scope, and report blockers, shared-file conflicts, scope expansion, or recommended handoffs upward; child prompts should report recommended handoffs upward rather than recursively orchestrating.
Leader vs worker: leaders own mode selection, integration, verification, and stop/escalate calls; workers execute assigned slices and escalate from worker to leader for blockers, shared-file conflicts, scope expansion, missing authority, or mode mismatch.
Rules: max 6 concurrent child agents; child prompts remain under AGENTS.md authority; prefer inherited model defaults unless a task has a concrete model reason; `worker` is a team-runtime surface, not a general-purpose child role.
</child_agent_protocol>


<invocation_conventions>
- `$name` — invoke a workflow skill.
- `/skills` — browse available skills.
- Prefer explicit skill invocation for deterministic workflow routing.
</invocation_conventions>

<model_routing>
Match role to task shape: `explore` for repo lookup, `researcher` for official docs/reference gathering, `dependency-expert` for SDK/package decisions, `executor` for implementation, `debugger` for root cause, `architect`/`critic` for high-complexity review. Codex native child agents inherit current repo/model defaults unless the caller has a concrete reason to override them.
</model_routing>

<specialist_routing>
Leader/workflow routing contract:
<!-- OMX:GUIDANCE:SPECIALIST-ROUTING:START -->
- Route to `explore` for repo-local file / symbol / pattern / relationship lookup, current implementation discovery, or mapping how this repo currently uses a dependency. `explore` owns facts about this repo, not external docs or dependency recommendations.
- Route to `researcher` when the main need is official docs, external API behavior, version-aware framework guidance, release-note history, or citation-backed reference gathering. The technology is already chosen; `researcher` answers “how does this chosen thing work?” and is not the default dependency-comparison role.
- Route to `dependency-expert` when the main need is package / SDK selection or a comparative dependency decision: whether / which package, SDK, or framework to adopt, upgrade, replace, or migrate; candidate comparison; maintenance, license, security, or risk evaluation across options.
- Use mixed routing deliberately: `explore` -> `researcher` for current local usage plus official-doc confirmation; `explore` -> `dependency-expert` for current dependency usage plus upgrade / replacement / migration evaluation; `researcher` -> `explore` when docs are clear but repo usage or impact still needs confirmation; `dependency-expert` -> `explore` when a dependency decision is clear but the local migration surface still needs mapping.
- Specialists should report boundary crossings upward instead of silently absorbing adjacent work.
- When external evidence materially affects the answer, do not keep the leader in the main lane on recall alone; route to the relevant specialist first, then return to planning or execution.
<!-- OMX:GUIDANCE:SPECIALIST-ROUTING:END -->
</specialist_routing>

<agent_catalog>
Key roles: `explore`, `researcher`, `dependency-expert`, `planner`, `architect`, `debugger`, `executor`, `test-engineer`, `verifier`, and `critic`. Use the installed role catalog for full descriptions.
</agent_catalog>

<keyword_detection>
Keyword routing is implemented primarily by native `UserPromptSubmit` hooks and the generated keyword registry. Treat hook-injected routing context as authoritative for the current turn, then load the named `SKILL.md` or prompt file as instructed.

Fallback behavior when hook context is unavailable:
- Explicit `$name` invocations run left-to-right and override implicit keywords.
- Bare skill names do not activate skills by themselves; skill-name activation requires explicit `$skill` invocation. Natural-language routing phrases may still map to a workflow. Examples: `analyze` / `investigate` → `$analyze` for read-only deep analysis with ranked synthesis, explicit confidence, and concrete file references.
- Keep the detailed keyword list in `src/hooks/keyword-registry.ts`; do not duplicate it here.

Runtime workflows such as `autopilot`, `ultraqa`, `team`, and `ultragoal` require OMX CLI runtime support. In Codex App, outside-tmux, or plain Codex sessions without OMX tmux runtime, explain that those workflows are not directly available there and continue with the nearest App-safe surface unless the user explicitly wants to launch OMX CLI from shell first.
- Route explicit `$autopilot` to its supervised `$deep-interview -> $ralplan -> $ultragoal` chain.
- `$ralph`, `$ultrawork`, `$pipeline`, `ecomode`, and `swarm` remain removed or deprecated sunset stubs; do not route users there.
- When deep-interview is active in attached-tmux OMX CLI/runtime, ask each interview round via `omx question`; after launching `omx question` in a background terminal, wait for that terminal to finish and read the JSON answer before continuing; preserve the leader pane with `OMX_QUESTION_RETURN_PANE=$TMUX_PANE` when invoking it through Bash/tool paths. Outside tmux or native surfaces that cannot render `omx question` should use the native structured question path when available; otherwise ask exactly one concise plain-text question and wait for the answer.

</keyword_detection>

<skills>
Skills are workflow commands. Always load the relevant installed `SKILL.md` before following a skill-specific process. Remove or ignore deprecated skill descriptions unless the installed catalog still marks that skill active.
</skills>

<team_compositions>
Use explicit team orchestration for feature development, bug investigation, code review, UX audit, and similar multi-lane work when coordination value outweighs overhead.
</team_compositions>

<team_pipeline>
Team mode is the structured multi-agent surface. Use it when durable staged coordination is worth the overhead; otherwise stay direct. Terminal states: `complete`, `failed`, `cancelled`.
</team_pipeline>

<team_model_resolution>
Team/Swarm worker model precedence: explicit `OMX_TEAM_WORKER_LAUNCH_ARGS`, inherited leader `--model`, then low-complexity default from `OMX_DEFAULT_SPARK_MODEL` (legacy alias: `OMX_SPARK_MODEL`). Normalize model flags to one canonical `--model <value>` entry and use `OMX_DEFAULT_FRONTIER_MODEL` / `OMX_DEFAULT_SPARK_MODEL` rather than guessing defaults.
</team_model_resolution>

<!-- OMX:MODELS:START -->
## Model Capability Table

Auto-generated by `omx setup` from the current `config.toml` plus OMX model overrides.

| Role | Model | Reasoning Effort | Use Case |
| --- | --- | --- | --- |
| Frontier (leader) | `gpt-6-astra` | high | Primary leader/orchestrator for planning, coordination, and frontier-class reasoning. |
| Spark (explorer/fast) | `gpt-6-astra` | low | Fast triage, explore, lightweight synthesis, and low-latency routing. |
| Standard (subagent default) | `gpt-6-astra` | high | Default standard-capability model for installable specialists and secondary worker lanes unless a role is explicitly frontier or spark. |
| `explore` | `gpt-6-astra` | low | Fast codebase search and file/symbol mapping (fast-lane, fast) |
| `analyst` | `gpt-6-astra` | medium | Requirements clarity, acceptance criteria, hidden constraints (frontier-orchestrator, frontier) |
| `planner` | `gpt-6-astra` | medium | Task sequencing, execution plans, risk flags (frontier-orchestrator, frontier) |
| `architect` | `gpt-6-astra` | xhigh | System design, boundaries, interfaces, long-horizon tradeoffs (frontier-orchestrator, frontier) |
| `debugger` | `gpt-6-astra` | high | Root-cause analysis, regression isolation, failure diagnosis (deep-worker, standard) |
| `executor` | `gpt-6-astra` | medium | Code implementation, refactoring, feature work (deep-worker, standard) |
| `team-executor` | `gpt-6-astra` | medium | Supervised team execution for conservative delivery lanes (deep-worker, frontier) |
| `verifier` | `gpt-6-astra` | high | Completion evidence, claim validation, test adequacy (frontier-orchestrator, standard) |
| `code-reviewer` | `gpt-6-astra` | high | Comprehensive review across all concerns (frontier-orchestrator, frontier) |
| `dependency-expert` | `gpt-6-astra` | high | External SDK/API/package evaluation (frontier-orchestrator, standard) |
| `test-engineer` | `gpt-6-astra` | medium | Test strategy, coverage, flaky-test hardening (deep-worker, frontier) |
| `designer` | `gpt-6-astra` | high | UX/UI architecture, interaction design (deep-worker, standard) |
| `writer` | `gpt-6-astra` | high | Documentation, migration notes, user guidance (fast-lane, standard) |
| `git-master` | `gpt-6-astra` | high | Commit strategy, history hygiene, rebasing (deep-worker, standard) |
| `code-simplifier` | `gpt-6-astra` | high | Simplifies recently modified code for clarity and consistency without changing behavior (deep-worker, frontier) |
| `researcher` | `gpt-6-astra` | high | External documentation and reference research (fast-lane, standard) |
| `critic` | `gpt-6-astra` | high | Plan/design critical challenge and review (frontier-orchestrator, frontier) |
| `vision` | `gpt-6-astra` | low | Image/screenshot/diagram analysis (fast-lane, frontier) |
<!-- OMX:MODELS:END -->

<verification>
Verify before claiming completion.
<!-- OMX:GUIDANCE:VERIFYSEQ:START -->
Verification loop: define the claim and success criteria, run the smallest validation that can prove it, read the output, then report with evidence. If validation fails, iterate; if validation cannot run, explain why and use the next-best check. Keep evidence summaries concise but sufficient.

- Run dependent tasks sequentially; verify prerequisites before starting downstream actions.
- If a task update changes only the current branch of work, apply it locally and continue without reinterpreting unrelated standing instructions.
- For coding work, prefer targeted tests for changed behavior, then typecheck/lint/build/smoke checks when applicable; do not claim completion without fresh evidence or an explicit validation gap.
- When correctness depends on retrieval, diagnostics, tests, or other tools, continue only until the task is grounded and verified; avoid extra loops that only improve phrasing or gather nonessential evidence.
<!-- OMX:GUIDANCE:VERIFYSEQ:END -->
</verification>

<execution_protocols>
Mode selection: follow `<delegation_rules>` above. Switch lanes only for a concrete unresolved ambiguity, coordination need, or blocker.

Command routing: use normal Codex repository inspection tools/subagents as the default surface for simple read-only repository lookup tasks; use `omx sparkshell` only for explicit shell-native read-only evidence or bounded verification.
When to use what:
- Use normal Codex repository inspection tools/subagents for repository lookup and implementation context.
- Use `omx sparkshell --tmux-pane` only as an explicit opt-in operator aid for shell-native tmux evidence or bounded verification; it does not replace raw evidence capture.

Supervisor tmux handoff safety:
- Never paste from tmux's implicit/current buffer. Load handoff text into a fresh named buffer with `tmux set-buffer -b <name> -- "$message"` or a temp-file-backed `tmux load-buffer -b <name> <file>`; never use `tmux load-buffer -- <message>`.
- Verify the named buffer with `tmux show-buffer -b <name>` before any paste. A failed load or mismatched buffer is a blocker; do not run `paste-buffer` or submit keys after it.
- Clear the pane composer with `tmux send-keys -t <pane> C-u` immediately before paste, then use bracketed paste (`tmux paste-buffer -t <pane> -b <name> -p -d`) and submit intentionally.
- Recapture the pane after paste/Enter and verify the intended turn was accepted rather than leaving stale draft text visible.

Leader vs worker: leaders choose mode, delegate bounded work, integrate, and own verification; workers execute their slice and escalate blockers, scope expansion, shared-file conflicts, or mode mismatch upward. Escalate from worker to leader for blockers, scope expansion, shared ownership conflicts, or mode mismatch.

Stop / escalate: stop when the task is verified complete, the user says stop/cancel, or no meaningful recovery path remains. Escalate to the user only for irreversible, destructive, materially branching decisions, or missing authority.

Output contract: Default update/final shape: state current mode, action/result, and evidence or blocker/next step. Keep rationale once; do not restate the full plan every turn; expand only for risk, handoff, or explicit request.

Anti-slop workflow:
- Cleanup/refactor/deslop work follows the same lightweight workflow (`understand -> execute -> verify -> report`); use `$ai-slop-cleaner` as a bounded helper inside the chosen execution lane, not as a competing top-level workflow.
- Write a cleanup plan before modifying code; lock existing behavior with regression tests first, then make one smell-focused pass at a time.
- Prefer deletion over addition, and prefer reuse plus boundary repair over new layers.
- No new dependencies without explicit request.
- Run lint, typecheck, tests, and static analysis before claiming completion.
- Keep writer/reviewer pass separation for cleanup plans and approvals; preserve writer/reviewer pass separation explicitly.

Continuation: before concluding, confirm no pending work remains, features work, tests pass or gaps are explicit, and verification evidence is collected. If not, continue.
</execution_protocols>

<cancellation>
Use the `cancel` skill to end active execution modes when work is done and verified, when the user says stop, or when a hard blocker prevents meaningful progress. Do not cancel while recoverable work remains.
</cancellation>

<state_management>
See [Durable Runtime Invariants](#durable-runtime-invariants-canonical-ssot) for state ownership and hook boundaries. OMX runtime state lives under `.omx/`.
</state_management>

## Durable Runtime Invariants (canonical SSOT)

This section is the single source of truth for durable state ownership, hook boundaries, cancellation, and Team coordination. Skills and role prompts reference it; they must not restate or weaken these rules.

### State and hook ownership

- Durable state is authoritative only in the current, proven session or Team scope. Compatibility discovery is read-only and never grants write authority.
- Hooks own normal skill activation and workflow-state persistence under `.omx/state/`; skills do not duplicate or mutate hook-owned state except through documented recovery paths.
- Native hook payloads, prompt labels, task text, cwd, environment, pointers, transcripts, markers, and local trackers are routing or diagnostic data, not ownership or write authority.
- The Team state files and `omx team api ... --json` are the source of truth for task lifecycle and mailbox coordination.

### Cancellation boundary

- Cancellation parses and validates arguments before mutation, resolves one exact writable scope, freezes and revalidates target identity, mutates only proven targets, and leaves unrelated sessions, legacy roots, Team artifacts, and tmux sessions untouched.
- Ralph cancellation must satisfy its documented terminal post-conditions in the same scope; linked modes are handled only when the link is proven.
- `--force` does not widen cancellation scope; it only removes the selected exact-session native-stop entry after the same authority checks. `--all` is unsupported.
- Team cancellation requires exact frozen Team root, internal name, session, leader pane, and runtime identity. It fails closed when that proof is unavailable or changes; it must not enumerate or broadly kill Team sessions or recursively delete unrelated Team state.

### Team protocol

- Team runtime is explicit and outside the default workflow. Ultragoal does not auto-launch Team, and ordinary workflows do not silently become Team runs.
- Workers ACK startup, claim before work, transition task status through the lifecycle API, use release only for rollback, and report verification evidence. Leaders own integration, final verification, and shutdown decisions.
- Prefer durable state writes and `omx team api ... --json` dispatch. Direct `tmux send-keys` is fallback-only, never primary dispatch; manual pane actions require prior state/evidence checks.
- Team shutdown waits for terminal task state and uses exact Team authority. It does not shut down active work unless explicitly aborting.

### Ultragoal ownership

- `.omx/ultragoal/goals.json` is the leader-owned plan and `.omx/ultragoal/ledger.jsonl` is its durable audit trail. Workers report task evidence only; they do not create worker ledgers, mutate Ultragoal artifacts, or checkpoint goals.
- Shell commands and hooks do not mutate hidden Codex goal state. The active agent uses `get_goal`, `create_goal`, and `update_goal` only at the documented gates, then checkpoints with a fresh `get_goal` snapshot.

## Setup

Execute `omx setup` to install all components. Execute `omx doctor` to verify installation.
<!-- OMX:AGENTS:END -->
