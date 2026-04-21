---
name: seekerclaw-ops-recovery
description: "스왑 거절·RPC 오류·한도·만료 등 이상 발생 시 원인을 수집하고 hold 또는 recover_and_resume을 결정할 때 사용한다. 운영 절차는 docs/OPERATIONS_CHECKLIST.md와 동일 용어(hold, recover_and_resume, kill_and_hold)를 쓴다."
version: "0.2.0"
emoji: "🔧"
requires:
  bins: []
  env: []
---

# seekerclaw-ops-recovery

## 목적

실패·경보 상황에서 **안전한 정지 또는 조건부 재개** 절차를 밟는다. 외부 감사 API 대신 앱 로그·메모리·텔레그램 기록을 사용한다.

## 기본 절차

1. `memory/tradebot-state.json`을 읽어 `severity`, `alerts`를 갱신한다.
2. 최근 오류를 수집한다:
   - SeekerClaw 로그 또는 `/logs` 등
   - 지갑·스왑 UI 오류 메시지
   - 텔레그램에 남은 거절·실패 요약
3. **복구 원인 → 조치**(체크리스트와 동일; `evidence.events_reviewed`·`reasons`에 아래 식별자를 쓴다):

   | 원인 | 조치 |
   | --- | --- |
   | `daily_limit_reached` | **hold** — UTC 자정 리셋까지 라이브 중단(`live_swap_blocked: true` 등). |
   | `scenario_expired` | pending 정리(`memory/tradebot-pending-scenario.json` 초기화) 후 **re-draft**·재승인. |
   | `rpc_error` **연속 3회** (`rpc_error(3x)`) | `memory/tradebot-state.json`에서 **`severity: critical`** 갱신. 이후 **`seekerclaw-status-guard`는 `kill_and_hold`**가 되도록 알림·필드를 맞춘다. 본 스킬 출력 `decision`은 즉시 재개가 불가하면 **`hold`**. |

   - **일반 거절·기타 한도**: 상황에 따라 위와 같이 **hold** 또는 상태 갱신.
   - **운영자 `/deny` 또는 사용자 거절**: pending 삭제, idempotency에서 해당 승인 종료 처리.
4. 조치 요약을 `memory/tradebot-heartbeat-log.md`에 한 줄 기록한다.

## 출력 계약

```json
{
  "decision": "hold | recover_and_resume",
  "reasons": ["..."],
  "actions": ["operator_review", "clear_pending", "..."],
  "evidence": {
    "severity": "warn | critical",
    "events_reviewed": ["daily_limit_reached", "scenario_expired", "rpc_error", "swap_failed", "user_rejected", "..."]
  }
}
```

## 재개(recover_and_resume)

- **`recover_and_resume`**: 원인이 해소되고 `severity`를 `ok` 또는 `warn`으로 낮출 수 있을 때만 선택한다.
- **`kill_and_hold`**: 이 스킬의 출력 enum은 아니다. `rpc_error(3x)` 등으로 `critical`을 올린 뒤 **다음 하트비트의 `seekerclaw-status-guard`가 `kill_and_hold`를 내는 것**이 게이트다.
- 재개 전에 반드시 `seekerclaw-status-guard`를 다시 실행한다(`docs/OPERATIONS_CHECKLIST.md`의 Resume criteria).
