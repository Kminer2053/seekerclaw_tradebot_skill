# 운영 체크리스트

라이브 스왑 전·하트비트·장애 대응 시 **스킬만** 따른다. 용어: **hold**(이번 주기 실행 중단), **kill_and_hold**(`seekerclaw-status-guard`의 강한 중단·`critical` 등), **recover_and_resume**(`seekerclaw-ops-recovery`의 조건부 재개). 상세 복구 매핑은 `skills/seekerclaw-ops-recovery/SKILL.md`를 본다.

## Pre-flight (라이브 전)

- `docs/quickstart.md`와 본 문서를 읽었다.
- `HEARTBEAT.md`에 `docs/heartbeat-template.md` 절차가 반영됐다.
- `memory/tradebot-state.json` 스키마·폴백을 `docs/contracts/skill-io-contracts.md`로 확인했다.
- 텔레그램(또는 앱) **명시적 승인** 없이 라이브 스왑하지 않는다.

## Per-heartbeat checks (주기마다)

- `seekerclaw-status-guard`: `proceed` / `hold` / `kill_and_hold` 기록(`memory/tradebot-last-guard.json`).
- **hold** 또는 **kill_and_hold**면 이번 주기는 스왑·신규 주문 없음.
- `proceed`일 때만 초안·승인·실행 파이프라인(`seekerclaw-draft-and-approve` → 승인 → `seekerclaw-execute-approved`).
- 이상 징후 시 `seekerclaw-ops-recovery` 후 `tradebot-state.json`·`tradebot-heartbeat-log.md` 갱신.

## Incident response (장애·복구)

| 원인(식별자) | 조치 요약 |
| --- | --- |
| `daily_limit_reached` | **hold** — UTC 자정 리셋까지 라이브 중단(예: `live_swap_blocked: true`). |
| `scenario_expired` | pending 정리(`memory/tradebot-pending-scenario.json` 초기화) 후 **re-draft**·재승인. |
| `rpc_error` 연속 3회 | `severity: critical` 갱신 — 다음 가드에서 **kill_and_hold**에 해당하도록 상태를 맞춘다. |

- 그 외: `seekerclaw-ops-recovery` 출력의 `reasons`·`actions`·`evidence.events_reviewed`를 따른다.

## Resume criteria (재개 조건)

- **recover_and_resume**은 원인이 해소되고 `severity`를 `ok` 또는 `warn`으로 낮출 수 있을 때만 선택한다.
- 재개 전 **반드시** `seekerclaw-status-guard`를 다시 실행해 `proceed`인지 확인한다.
- **kill_and_hold** 또는 `critical`에서의 재개는 운영자가 상태·알림을 정리한 뒤에만 수행한다.
