<!-- seekerclaw-tradebot-pack:begin -->

## Tradebot pack (seekerclaw_tradebot_skill)

위에 있는 기존 하트비트 절차는 그대로 두고, **이 섹션만** 본 저장소의 승인 기반 트레이딩 루프용이다. 앱 하트비트 주기(예: 5분)는 기기 설정을 따른다.

### 운영자 알림 언어(기본값 + 선택)
- 기본 알림 언어는 한국어(`ko`)로 둔다.
- 선택 파일: `memory/tradebot-operator-preferences.json`
- 예시:
```json
{
  "notification_language": "ko",
  "fallback_language": "en"
}
```
- `notification_language` 값이 `ko`면 운영자 알림은 한국어로 작성한다.
- 값이 없거나 파싱 실패면 `ko`를 기본으로 사용한다.
- 영어 보조 문구가 필요하면 같은 알림에 짧게 병기하되, 본문 첫 줄은 선택 언어를 따른다.

### 주기당 한도 (명시)
max_live_swap_attempt = 1
max_new_draft = 1
텔레그램 동일 경고 요약: 주기당 1회

### 표준 실패 모드 (이름·대응 요약)
- state_file_missing — 상태 파일 없음/손상: 거래 실행 계열 금지, status-guard·가드 로깅은 진행, recovery·체크리스트 따름.
- approval_missing — 명시적 승인 없음: execute-approved 호출 금지·hold, 승인 요청만.
- rpc_error_burst — RPC 연속 실패: 백오프, 연속 3주기 등은 severity·kill_and_hold(OPERATIONS_CHECKLIST).

### 체크리스트 (트레이딩 팩)
1. 상태 파일 읽기 — memory/tradebot-state.json 요약. 없거나 JSON 깨짐이면 계약 폴백과 동등; 스왑·체결·신규 주문 없음. seekerclaw-status-guard 및 tradebot-last-guard 기록은 진행.
2. seekerclaw-status-guard — proceed / hold / kill_and_hold, tradebot-last-guard.json 기록.
3. hold 또는 kill_and_hold면 **트레이딩 팩 관점에서** 이번 주기는 스왑·신규 주문 없음(위쪽 기존 하트비트 항목은 문서에 정의된 대로).
4. 승인 대기 — `memory/tradebot-pending-scenario.json`이 없거나 유효한 승인 대기 초안이 없고 proceed면 seekerclaw-draft-and-approve(주기당 max_new_draft 준수).
5. 실행 — 명시적 승인 후 status-guard 재실행이 proceed일 때만 seekerclaw-execute-approved(주기당 max_live_swap_attempt 준수).
   - 승인 명령은 `/approve <scenario_id>`를 우선 사용한다.
   - `<scenario_id>`가 없으면 현재 pending 1건만 있을 때만 승인으로 해석한다.
   - pending이 2건 이상이면 "대상 불명확"으로 처리하고, ID를 포함한 재승인을 요청한다.
6. 복구 — 거절·한도·만료·RPC 이상 시 seekerclaw-ops-recovery, tradebot-state severity 갱신.
7. 스냅샷 — memory/tradebot-heartbeat-log.md에 시각·phase·decision 한 줄 append.

### 이번 주기 실행 금지(트레이딩 팩, 즉시)
severity == critical / entry_blocked == true / live_swap_blocked == true / 무승인 라이브 스왑

<!-- seekerclaw-tradebot-pack:end -->
