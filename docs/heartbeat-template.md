# Heartbeat Execution Template

워크스페이스 루트 `HEARTBEAT.md`에는 **아래「복사용 운영 블록」만** 붙입니다. 앱 하트비트 주기(예: 5분)와 함께 동작합니다. 스키마·폴백·장애 매핑은「설명·참고」를 봅니다.

---

## 복사용 운영 블록

`<!-- seekerclaw-heartbeat-paste:start -->` 와 `<!-- seekerclaw-heartbeat-paste:end -->` **사이**만 선택해 `HEARTBEAT.md`에 붙입니다(HTML 주석 줄은 제외).

<!-- seekerclaw-heartbeat-paste:start -->

# Zero tradebot — heartbeat (skill-only)

## 주기당 한도 (명시)
max_live_swap_attempt = 1
max_new_draft = 1
텔레그램 동일 경고 요약: 주기당 1회

## 표준 실패 모드 (이름·대응 요약)
- state_file_missing — 상태 파일 없음/손상: 거래 실행 계열 금지, status-guard·가드 로깅은 진행, recovery·체크리스트 따름.
- approval_missing — 명시적 승인 없음: execute-approved 호출 금지·hold, 승인 요청만.
- rpc_error_burst — RPC 연속 실패: 백오프, 연속 3주기 등은 severity·kill_and_hold(OPERATIONS_CHECKLIST).

## 체크리스트
1. 상태 파일 읽기 — memory/tradebot-state.json 요약. 없거나 JSON 깨짐이면 계약 폴백과 동등; 스왑·체결·신규 주문 없음. seekerclaw-status-guard 및 tradebot-last-guard 기록은 진행.
2. seekerclaw-status-guard — proceed / hold / kill_and_hold, tradebot-last-guard.json 기록.
3. hold 또는 kill_and_hold면 이번 주기 종료(스왑·신규 주문 없음).
4. 승인 대기 — `memory/tradebot-pending-scenario.json`이 없거나 유효한 승인 대기 초안이 없고 proceed면 seekerclaw-draft-and-approve(주기당 max_new_draft 준수).
5. 실행 — 명시적 승인 후 status-guard 재실행이 proceed일 때만 seekerclaw-execute-approved(주기당 max_live_swap_attempt 준수).
6. 복구 — 거절·한도·만료·RPC 이상 시 seekerclaw-ops-recovery, tradebot-state severity 갱신.
7. 스냅샷 — memory/tradebot-heartbeat-log.md에 시각·phase·decision 한 줄 append.

## 이번 주기 실행 금지(즉시)
severity == critical / entry_blocked == true / live_swap_blocked == true / 무승인 라이브 스왑

<!-- seekerclaw-heartbeat-paste:end -->

---

## 설명·참고 (복사 불필요)

### 스키마·증거 매핑

정상·누락·손상 폴백 필드와 `evidence`는 `docs/contracts/skill-io-contracts.md` **「1) seekerclaw-status-guard」** 상태 파일 스키마를 본다.

### 표준 실패 모드 보강

| 모드 | 의미 |
|------|------|
| `state_file_missing` | `tradebot-state.json` 부재·파싱 실패. 템플릿 체크리스트 1·2에 따라 hold/kill 쪽으로 수렴; 라이브 경로 진입 전에 가드가 막아야 한다. |
| `approval_missing` | 텔레그램 `/approve` 등 명시적 승인 없이 실행 스킬을 타지 않음. `seekerclaw-execute-approved`는 hold·`request_explicit_approval` 쪽이 정상. |
| `rpc_error_burst` | 단기간 RPC 오류 반복. 주기 건너뛰기·백오프; 동일 오류 3주기 이상이면 `severity: critical`·라이브 스왑 중단·다음 가드 `kill_and_hold` 유도. 상세는 `docs/OPERATIONS_CHECKLIST.md`. |

### 실패 처리(연속 오류)

- 연속 3주기 이상 동일 오류면 `severity: critical`, 라이브 스왑 중단; 다음 **seekerclaw-status-guard**는 **kill_and_hold**가 되도록 한다.
- 운영자 확인 전까지 **hold** 또는 **kill_and_hold** 유지.

### 실행 금지(상세)

- `severity == critical`
- `entry_blocked == true`
- 라이브 경로에서 `live_swap_blocked == true`
- 무승인 상태에서의 라이브 스왑

아키텍처는 **스킬 + 워크스페이스 메모리 파일**만 사용한다(별도 백엔드 API 없음). 흐름 개요는 `docs/heartbeat-flow.md`.
