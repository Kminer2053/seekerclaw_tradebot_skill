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

**원문은 [HEARTBEAT-APPEND.md](HEARTBEAT-APPEND.md) 한 파일에만 둡니다.** 내용이 바뀌어도 그 파일과 동기화하면 됩니다.

- **사람:** `HEARTBEAT-APPEND.md`를 연 뒤 **전체**를 복사해 `HEARTBEAT.md` **하단**에 붙인다.
- **에이전트·자동화:** 아래 raw URL 본문 전체를 가져와 동일하게 append 한다.  
  `https://raw.githubusercontent.com/Kminer2053/seekerclaw_tradebot_skill/main/docs/HEARTBEAT-APPEND.md`  
  (포크·미러를 쓰는 경우 해당 저장소의 `main` 기준 경로로 바꾼다.)

---

## 설명·참고 (복사 불필요)

### 스키마·증거 매핑

정상·누락·손상 폴백 필드와 `evidence`는 `docs/contracts/skill-io-contracts.md` **「1) seekerclaw-status-guard」** 상태 파일 스키마를 본다.

### 표준 실패 모드 보강

| 모드 | 의미 |
|------|------|
| `state_file_missing` | `memory/tradebot-state.json` 부재·파싱 실패. 체크리스트 1·2에 따라 hold/kill 쪽으로 수렴; 라이브 경로 진입 전에 가드가 막아야 한다. |
| `approval_missing` | 텔레그램 `/approve` 등 명시적 승인 없이 실행 스킬을 타지 않음. `seekerclaw-execute-approved`는 hold·`request_explicit_approval` 쪽이 정상. |
| `rpc_error_burst` | 동일 RPC 오류 연속 3주기 시 `severity: critical`·라이브 스왑 중단·다음 가드 `kill_and_hold` 유도. 백오프 적용. 상세는 `docs/OPERATIONS_CHECKLIST.md`. |

### 실패 처리(연속 오류)

- 동일 오류 연속 3주기면 `severity: critical`, 라이브 스왑 중단; 다음 **seekerclaw-status-guard**는 **kill_and_hold**가 되도록 한다.
- 운영자 확인 전까지 **hold** 또는 **kill_and_hold** 유지.

### 실행 금지(상세)

- `severity == critical`
- `entry_blocked == true`
- 라이브 경로에서 `live_swap_blocked == true`
- 무승인 상태에서의 라이브 스왑

본 패키지는 **스킬 + 워크스페이스 메모리 파일**만 사용한다. 흐름 개요는 `docs/heartbeat-flow.md`.
