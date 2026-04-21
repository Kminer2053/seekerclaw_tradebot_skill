# SeekerClaw Skill-Only Tradebot Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 별도 `bot-core` 없이 SeekerClaw 기본 기능과 커스텀 스킬만으로 승인 기반 트레이딩 루프를 안정적으로 운영한다.

**Architecture:** 모든 판단과 실행은 `skills/seekerclaw-*` + 워크스페이스 `memory/tradebot-*.json` + `HEARTBEAT.md` 조합으로 처리한다. 실행 게이트는 `status-guard`가 단일 진입점이 되고, `draft-and-approve`와 `execute-approved`는 승인/중복방지 파일을 공통 계약으로 공유한다. 실패/거절/한도 이벤트는 `ops-recovery`가 상태 파일을 갱신하고 다음 루프를 `hold` 또는 `recover_and_resume`으로 통제한다.

**Tech Stack:** SeekerClaw 내장 스킬 시스템(SKILL.md), HEARTBEAT.md, Telegram `/approve` `/deny`, Solana/Jupiter 내장 기능, 워크스페이스 메모리 파일(JSON/Markdown)

---

## File Structure

- `skills/seekerclaw-status-guard/SKILL.md`: 실행 가능 여부 판단 규칙과 구조화 출력 계약
- `skills/seekerclaw-draft-and-approve/SKILL.md`: 초안 생성, 승인 요청, 중복 드래프트 방지
- `skills/seekerclaw-execute-approved/SKILL.md`: 승인 후 실행, 2차 가드, idempotency 적용
- `skills/seekerclaw-ops-recovery/SKILL.md`: 오류 분류, 안전 정지/재개 기준
- `docs/heartbeat-template.md`: 하트비트 실제 운용 절차 템플릿
- `docs/idempotency-policy.md`: 승인 건별 중복 실행 방지 규칙
- `docs/contracts/skill-io-contracts.md`: 스킬 간 공통 응답 형식
- `docs/quickstart.md`: 설치/운영자 실행 순서
- `docs/OPERATIONS_CHECKLIST.md` (신규): 실운영 점검표

---

### Task 1: 상태 파일 스키마 고정

**Files:**
- Modify: `docs/contracts/skill-io-contracts.md`
- Modify: `docs/heartbeat-template.md`
- Test: `docs/quickstart.md` (수동 검증 절차 추가)

- [ ] **Step 1: 스키마 테스트 케이스 작성(문서 기준)**

```json
{
  "severity": "ok",
  "entry_blocked": false,
  "live_swap_blocked": false,
  "alerts": [],
  "recommended_actions": ["request_approval"]
}
```

- [ ] **Step 2: 누락/손상 파일 케이스도 명시**

```json
{
  "severity": "warn",
  "entry_blocked": true,
  "live_swap_blocked": true,
  "alerts": ["state_file_missing"]
}
```

- [ ] **Step 3: `status-guard` evidence 필드와 1:1 매핑 표 추가**

Run: 문서에 `state field -> guard evidence field` 표 삽입  
Expected: `severity`, `entry_blocked`, `live_swap_blocked`, `alerts`, `recommended_actions`가 모두 대응됨

- [ ] **Step 4: 검증 절차 업데이트**

Run: `docs/quickstart.md`에 "상태 파일 없을 때 hold가 나와야 함" 수동 체크 추가  
Expected: 신규 운영자가 실패 기본값을 이해하고 재현 가능

- [ ] **Step 5: Commit**

```bash
git add docs/contracts/skill-io-contracts.md docs/heartbeat-template.md docs/quickstart.md
git commit -m "docs: fix canonical tradebot state schema and guard mapping"
```

### Task 2: 승인 대기 흐름 단일화

**Files:**
- Modify: `skills/seekerclaw-draft-and-approve/SKILL.md`
- Modify: `skills/seekerclaw-execute-approved/SKILL.md`
- Test: `docs/quickstart.md` (수동 시나리오)

- [ ] **Step 1: 중복 승인 대기 금지 규칙 명시**

```text
if memory/tradebot-pending-scenario.json exists and valid:
  decision = hold
  actions += ["notify_operator_existing_pending"]
```

- [ ] **Step 2: 승인 소스 고정**

Run: `execute-approved` 문서에 승인 소스를 `/approve`(또는 앱 동등 UI 승인)로 제한  
Expected: 채팅 내 일반 문구는 승인으로 인정하지 않음

- [ ] **Step 3: 무승인 실행 금지 문구를 출력 계약에 반영**

```json
{
  "decision": "hold",
  "reasons": ["approval_missing"],
  "actions": ["request_explicit_approval"]
}
```

- [ ] **Step 4: 수동 테스트 항목 추가**

Run: "승인 없이 execute 시도 -> hold" 시나리오를 `quickstart`에 추가  
Expected: 운영자가 1회 재현 가능

- [ ] **Step 5: Commit**

```bash
git add skills/seekerclaw-draft-and-approve/SKILL.md skills/seekerclaw-execute-approved/SKILL.md docs/quickstart.md
git commit -m "docs: enforce explicit approval path before execution"
```

### Task 3: Idempotency 파일 운영 완성

**Files:**
- Modify: `docs/idempotency-policy.md`
- Modify: `skills/seekerclaw-execute-approved/SKILL.md`
- Test: `docs/quickstart.md` (재시작 재현 절차)

- [ ] **Step 1: 파일 예시를 승인 다건 구조로 보강**

```json
{
  "records": {
    "ap-1234": {
      "idempotency_key": "exec-approved:ap-1234:2026-04-22",
      "last_result": { "ok": true, "tx_signature": "sig1" }
    }
  }
}
```

- [ ] **Step 2: 재시작 후 재사용 절차 명시**

Run: "앱 재시작 -> 같은 approval_id -> 기존 key 재사용 -> 재실행 금지" 절차 추가  
Expected: 이중 체결 방지 흐름을 운영자가 그대로 따라할 수 있음

- [ ] **Step 3: 실패 케이스 분기 정의**

```text
if last_result.ok == false:
  do not mint new key
  route to ops-recovery
```

- [ ] **Step 4: 금지사항 명확화**

Run: 랜덤 키 매 주기 생성 금지 문구를 execute skill에도 중복 명시  
Expected: 정책과 실행 스킬 간 충돌 없음

- [ ] **Step 5: Commit**

```bash
git add docs/idempotency-policy.md skills/seekerclaw-execute-approved/SKILL.md docs/quickstart.md
git commit -m "docs: finalize on-device idempotency and restart behavior"
```

### Task 4: 복구 정책을 운영 체크리스트로 연결

**Files:**
- Modify: `skills/seekerclaw-ops-recovery/SKILL.md`
- Create: `docs/OPERATIONS_CHECKLIST.md`
- Modify: `docs/quickstart.md`

- [ ] **Step 1: 복구 원인 분류표 확정**

```text
daily_limit_reached -> hold until UTC reset
scenario_expired -> clear pending + re-draft
rpc_error(3x) -> severity=critical + kill_and_hold
```

- [ ] **Step 2: 신규 운영 체크리스트 작성**

Run: `docs/OPERATIONS_CHECKLIST.md` 생성, 아래 섹션 포함  
Expected sections: Pre-flight, Per-heartbeat checks, Incident response, Resume criteria

- [ ] **Step 3: 체크리스트와 스킬의 용어 통일**

Run: `hold`, `recover_and_resume`, `kill_and_hold` 용어를 모든 문서에서 동일 표기  
Expected: 운영 중 혼동 용어 0개

- [ ] **Step 4: quickstart에 체크리스트 링크 추가**

Run: quickstart 마지막에 운영 전 필독 링크 추가  
Expected: 신규 운영자가 quickstart만 보고 체크리스트로 이어짐

- [ ] **Step 5: Commit**

```bash
git add skills/seekerclaw-ops-recovery/SKILL.md docs/OPERATIONS_CHECKLIST.md docs/quickstart.md
git commit -m "docs: add operations checklist and recovery runbook"
```

### Task 5: HEARTBEAT 적용용 최종 템플릿 잠금

**Files:**
- Modify: `docs/heartbeat-template.md`
- Modify: `docs/heartbeat-flow.md`
- Test: `docs/quickstart.md` (드라이런 체크)

- [ ] **Step 1: 실제 붙여넣기용 HEARTBEAT 섹션 최소화**

Run: 템플릿을 "복붙 블록" + "설명 블록"으로 분리  
Expected: 사용자는 복붙 블록만 복사해도 동작

- [ ] **Step 2: 주기당 최대 실행 횟수 강제 문구 추가**

```text
per cycle:
  max_live_swap_attempt = 1
  max_new_draft = 1
```

- [ ] **Step 3: 10회 드라이런 점검 절차 작성**

Run: quickstart에 "10회 하트비트 동안 실행/보류 카운트 확인" 단계 추가  
Expected: 승인 없는 execute 0회 확인 가능

- [ ] **Step 4: 실패 모드 문서화**

Run: `state_file_missing`, `approval_missing`, `rpc_error_burst` 3개 표준 오류를 템플릿에 명시  
Expected: 로그 해석 기준 통일

- [ ] **Step 5: Commit**

```bash
git add docs/heartbeat-template.md docs/heartbeat-flow.md docs/quickstart.md
git commit -m "docs: lock heartbeat template for safe skill-only execution"
```

---

## Spec Coverage Self-Review

- 승인 기반 실행, 중복 방지, 복구 절차, 하트비트 분기, 운영 체크리스트까지 모두 개별 Task로 매핑됨
- `bot-core`/HTTP 의존 항목은 전부 제외됨
- 모든 작업은 문서/스킬 변경으로 제한되어 현재 목표(스킬 전용)와 일치

## No Placeholder Check

- "TODO/추후 작성" 없이 각 단계에 파일/명령/기대 결과를 명시
- 각 Task의 수동 검증 경로를 quickstart에 연결

## Type/Term Consistency Check

- 상태 필드: `severity`, `entry_blocked`, `live_swap_blocked` 고정
- 결정값: `proceed`, `hold`, `kill_and_hold`, `recover_and_resume` 고정
- 승인 식별: `approval_id` 고정

---

Plan complete and saved to `docs/superpowers/plans/2026-04-22-seekerclaw-skill-only-tradebot.md`. Two execution options:

1. Subagent-Driven (recommended), 태스크별 병렬/순차 분리해서 빠르게 진행
2. Inline Execution, 지금 세션에서 제가 연속으로 직접 수행
