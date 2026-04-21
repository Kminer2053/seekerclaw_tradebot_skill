# Skill I/O Contracts

`bot-core` JSON API는 사용하지 않습니다. 스킬은 모델이 따라야 할 **구조화 응답**을 아래 형태로 맞춥니다. 실제 저장은 워크스페이스 `memory/` 파일 등에 기록하는 것을 권장합니다.

## 공통 출력 형식

```json
{
  "decision": "string",
  "reasons": ["string"],
  "actions": ["string"],
  "evidence": {}
}
```

---

## 1) seekerclaw-status-guard

문서 전반에서 **status-guard**라고 하면 스킬 **seekerclaw-status-guard**를 뜻한다.

### 입력

- 워크스페이스 상태 스냅샷(권장: `memory/tradebot-state.json`)
- 필요 시 동일 워크스페이스의 최근 하트비트·운영 메모 요약

### 상태 파일 스키마(권장, `memory/tradebot-state.json`)

**정상 예시**

```json
{
  "severity": "ok",
  "entry_blocked": false,
  "live_swap_blocked": false,
  "alerts": [],
  "recommended_actions": ["request_approval"]
}
```

**누락·손상 시 스킬 폴백 스냅샷**(파일을 읽지 못할 때 이와 동등한 필드로 간주)

```json
{
  "severity": "warn",
  "entry_blocked": true,
  "live_swap_blocked": true,
  "alerts": ["state_file_missing"],
  "recommended_actions": []
}
```

파일 누락·손상으로 위 폴백과 동등한 스냅샷을 쓸 때는 스왑·체결·신규 주문 등 거래 실행은 하지 않되, **seekerclaw-status-guard** 실행과 구조화 출력 기록(예: `memory/tradebot-last-guard.json`)은 계속한다.

### 상태 필드 → seekerclaw-status-guard `evidence` 매핑

| 상태 파일 필드 | `evidence` 필드 | 비고 |
| --- | --- | --- |
| `severity` | `evidence.severity` | 동일 키·값 전달 |
| `entry_blocked` | `evidence.entry_blocked` | 동일 |
| `live_swap_blocked` | `evidence.live_swap_blocked` | 동일 |
| `alerts` | `evidence.alerts` | 동일(배열) |
| `recommended_actions` | `evidence.recommended_actions` | 동일(배열); 출력 `actions` 후보로도 사용 가능 |

### 출력

```json
{
  "decision": "proceed | hold | kill_and_hold",
  "reasons": ["severity=critical", "entry_blocked", "..."],
  "actions": ["request_approval", "pause_live_swaps", "..."],
  "evidence": {
    "severity": "ok | warn | critical",
    "entry_blocked": false,
    "live_swap_blocked": false,
    "alerts": [],
    "recommended_actions": []
  }
}
```

### `decision` 값 (요약)

- **hold**: 게이트 미통과 등으로 이번 주기에서 거래 실행을 하지 않는 보수적 중단. 재개 여지가 있을 수 있다.
- **kill_and_hold**: **hold**보다 강한 중단(예: `critical`, 운영자 조치 전까지 유지). 거래 실행은 하지 않는다.
- **proceed**: 스냅샷 기준으로 승인·실행 파이프라인으로 진행 가능. 실제 체결은 별도 승인과 실행 직전 재가드 후에만 수행한다.

---

## 2) seekerclaw-draft-and-approve

### 입력

- seekerclaw-status-guard 결과 (`decision=proceed`)
- 전략 컨텍스트(심볼·수량·슬리피지 등)

### 출력

```json
{
  "decision": "await_approval | hold",
  "reasons": ["draft_recorded", "..."],
  "actions": ["notify_operator_for_approval"],
  "evidence": {
    "draft_scenario_id": "...",
    "recorded_to": "memory/tradebot-pending-scenario.json"
  }
}
```

---

## 3) seekerclaw-execute-approved

### 입력

- 텔레그램(또는 앱)에서의 **명시적 승인** 기록
- `approval_id`(또는 동등 식별자)
- 실행 직전 seekerclaw-status-guard 재실행 결과(`proceed`만 허용)

### 출력

```json
{
  "decision": "executed | hold",
  "reasons": ["gates_ok", "daily_limit", "..."],
  "actions": ["verify_wallet_tx", "append_memory_log"],
  "evidence": {
    "approval_id": "...",
    "idempotency_key_used": true,
    "tx_signature": "...",
    "simulation_only": false
  }
}
```

---

## 4) seekerclaw-ops-recovery

### 입력

- 최근 실패·거절 메시지(앱 로그, 텔레그램, 지갑 오류)
- 갱신 대상 `tradebot-state.json`

### 복구 원인(예시, `docs/OPERATIONS_CHECKLIST.md`와 동일)

- `daily_limit_reached` → **hold**(UTC 리셋까지)
- `scenario_expired` → pending 정리 후 re-draft
- `rpc_error` 연속 3회 → `severity: critical` → 다음 **seekerclaw-status-guard**는 **kill_and_hold**에 해당

**kill_and_hold**는 이 스킬의 `decision` 값이 아니라 가드 출력이다.

### 출력

```json
{
  "decision": "hold | recover_and_resume",
  "reasons": ["daily_limit_reached", "scenario_expired", "rpc_error_streak", "..."],
  "actions": ["clear_pending", "redraft", "operator_review", "..."],
  "evidence": {
    "severity": "warn | critical",
    "events_reviewed": ["daily_limit_reached", "scenario_expired", "rpc_error", "swap_failed", "user_rejected", "..."]
  }
}
```
