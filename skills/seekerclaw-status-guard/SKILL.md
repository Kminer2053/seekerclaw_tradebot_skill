---
name: seekerclaw-status-guard
description: "하트비트 첫 단계에서 워크스페이스 상태 파일과 알림을 읽고 proceed/hold/kill_and_hold를 결정할 때 사용한다. 라이브 스왑 전 반드시 호출한다."
version: "0.2.0"
emoji: "🛡️"
requires:
  bins: []
  env: []
---

# seekerclaw-status-guard

## 목적

이번 하트비트에서 **트레이딩 실행을 진행해도 되는지** 판단한다. 별도 서버 API는 없으며, 기록된 상태와 운영 규칙만 사용한다.

## 입력

- 권장: 워크스페이스 `memory/tradebot-state.json`  
  - `severity`: `ok` | `warn` | `critical`
  - `entry_blocked`: boolean
  - `live_swap_blocked`: boolean
  - `alerts`: string[]
  - `recommended_actions`: string[] (선택)
- 보조: 직전 하트비트 로그 `memory/tradebot-heartbeat-log.md` 마지막 수 줄
- 파일이 없으면 안전 쪽으로 기본값: `severity: warn`, `entry_blocked: true`, `live_swap_blocked: true`, `alerts: ["state_file_missing"]`

## 동작

1. 위 필드를 읽어 구조화 출력을 만든다.
2. 아래 정책을 **모두** 적용한다.

## 출력 계약

응답 본문에 아래 JSON 형태를 포함한다.

```json
{
  "decision": "proceed | hold | kill_and_hold",
  "reasons": ["..."],
  "actions": ["..."],
  "evidence": {
    "severity": "ok | warn | critical",
    "entry_blocked": false,
    "live_swap_blocked": false,
    "alerts": [],
    "recommended_actions": []
  }
}
```

## 기본 정책

- `severity == "critical"` → `hold` 또는 `kill_and_hold`(운영자가 완전 중단을 원하면 후자)
- `entry_blocked == true` → `hold`
- 라이브 스왑 경로에서 `live_swap_blocked == true` → `hold`
- 위에 해당 없으면 `proceed`

## 기록

결정 후 `memory/tradebot-last-guard.json`에 동일 구조로 저장한다.
