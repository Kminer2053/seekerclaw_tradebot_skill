# Heartbeat Execution Template

## 기존 하트비트와 함께 쓰기 (중요)

SeekerClaw 워크스페이스의 `HEARTBEAT.md`에는 **이미 다른 주기 업무**가 있을 수 있습니다. 아래 절차는 그 내용을 **삭제하거나 덮어쓰지 않습니다.**

1. 워크스페이스에서 `HEARTBEAT.md`를 연다.
2. **위쪽에 있는 기존 섹션·체크리스트는 그대로 둔다.**
3. 파일 **맨 아래**에 빈 줄을 하나 넣은 뒤, `<!-- seekerclaw-tradebot-pack:begin -->` 부터 `<!-- seekerclaw-tradebot-pack:end -->` **포함**까지 통째로 붙여 넣는다. (주석 줄도 그대로 두면, 나중에 이 구간만 찾아서 수정·삭제하기 쉽다.)
4. 이미 동일한 begin/end 주석이 있으면 **그 사이만** 이 저장소 최신본으로 갈아끼운다. 중복 블록을 만들지 않는다.
5. 선택: 붙이기 전에 `HEARTBEAT.md`를 한 번 복사해 백업해 둔다.

트레이딩 팩은 **하트비트 한 주기 안에서** 기존 항목을 실행한 **뒤** 이어서 돌리거나, 문서 순서대로 “기존 작업 → 아래 Tradebot pack” 순으로 운영자가 읽게 배치하면 된다. SeekerClaw가 한 번에 전체를 프롬프트로 넣는다면, **기존 내용이 먼저 오고** 본 패키지 블록이 **뒤에 오도록** 유지하는 것이 안전하다.

스키마·폴백·장애 매핑은 아래 **「설명·참고」**를 본다.

---

## 복사해 넣을 블록 (전체 선택)

아래 코드 블록 **전체**를 `HEARTBEAT.md` **하단**에 붙인다.

```markdown
<!-- seekerclaw-tradebot-pack:begin -->

## Tradebot pack (seekerclaw_tradebot_skill)

위에 있는 기존 하트비트 절차는 그대로 두고, **이 섹션만** 본 저장소의 승인 기반 트레이딩 루프용이다. 앱 하트비트 주기(예: 5분)는 기기 설정을 따른다.

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
6. 복구 — 거절·한도·만료·RPC 이상 시 seekerclaw-ops-recovery, tradebot-state severity 갱신.
7. 스냅샷 — memory/tradebot-heartbeat-log.md에 시각·phase·decision 한 줄 append.

### 이번 주기 실행 금지(트레이딩 팩, 즉시)
severity == critical / entry_blocked == true / live_swap_blocked == true / 무승인 라이브 스왑

<!-- seekerclaw-tradebot-pack:end -->
```

---

## 설명·참고 (복사 불필요)

### 스키마·증거 매핑

정상·누락·손상 폴백 필드와 `evidence`는 `docs/contracts/skill-io-contracts.md` **「1) seekerclaw-status-guard」** 상태 파일 스키마를 본다.

### 표준 실패 모드 보강

| 모드 | 의미 |
|------|------|
| `state_file_missing` | `memory/tradebot-state.json` 부재·파싱 실패. 체크리스트 1·2에 따라 hold/kill 쪽으로 수렴; 라이브 경로 진입 전에 가드가 막아야 한다. |
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

본 패키지는 **스킬 + 워크스페이스 메모리 파일**만 사용한다. 흐름 개요는 `docs/heartbeat-flow.md`.
