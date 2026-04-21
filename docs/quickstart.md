# Quickstart

별도 HTTP 백엔드·엔드포인트나 Node 전용 런타임은 **필요 없습니다**. SeekerClaw 앱 워크스페이스에 스킬과 하트비트만 맞추면 됩니다.

**쉬운 말:** 문서에 나오는 **드라이런**은 **「연습 모드」**(실제 스왑·체결 없이 절차만 도는 상태)라고 보면 됩니다. README 한국어 절의 **쉬운 말로 (용어)** 표를 함께 보세요.

**라이브(실거래) 전 필수:** [`docs/OPERATIONS_CHECKLIST.md`](OPERATIONS_CHECKLIST.md)(사전 점검·매 주기 점검·장애 대응·재개 기준).

## 1) 스킬 설치

- 이 프로젝트의 `skills/seekerclaw-*/` 디렉터리를 기기 워크스페이스의 `skills/` 아래에 두거나, 앱에서 제공하는 스킬 설치 방법(URL·파일 등)으로 등록합니다.
- 각 `SKILL.md`의 `description`이 시맨틱 라우팅에 쓰이므로, 앱 재시작 후 스킬 목록에 로드됐는지 확인합니다.

## 2) 하트비트에 절차 넣기 (기존 설정 보존)

기존 `HEARTBEAT.md` 내용을 **지우지 않습니다.** 트레이딩 팩은 **파일 맨 아래에만** 추가합니다.

1. `HEARTBEAT.md`를 연 뒤, 상단·중간의 **기존 하트비트·체크리스트는 수정하지 않는다.**
2. `docs/HEARTBEAT-APPEND.md` 파일 **전체**(또는 GitHub raw 동일 경로)를 `HEARTBEAT.md` 파일 **끝**에 붙인다. 설명은 `docs/heartbeat-template.md`를 본다.
3. 이미 `seekerclaw-tradebot-pack:begin` 이 있다면 **그 한 쌍 사이만** 최신 템플릿으로 교체하고, 블록을 두 번 넣지 않는다.
4. 하트비트 **주기**(예: 5분)는 SeekerClaw 앱 설정을 따른다. 템플릿의 `max_live_swap_attempt` / `max_new_draft`는 트레이딩 팩 내부 한도이며, 필요 시에만 숫자를 조정한다.

## 3) 한 주기 권장 순서

`seekerclaw-status-guard`의 `decision` 값(`proceed` / `hold` / `kill_and_hold`)과 초안·실행 스킬 출력의 `decision` 값(`await_approval` / `hold`, `executed` / `hold` 등)은 **서로 다른 열거형**이므로 같은 레이블로 섞어 쓰지 않는다.

1. `seekerclaw-status-guard` — 워크스페이스 상태 파일을 읽고 `proceed` / `hold` / `kill_and_hold` 결정
2. `hold` 또는 `kill_and_hold`면 이번 주기 종료(실행·스왑 없음)
3. `memory/tradebot-pending-scenario.json`이 **없거나**, 있어도 **유효한** 승인 대기 초안이 없을 때만 `seekerclaw-draft-and-approve`를 실행한다.
4. 운영자 승인(텔레그램 등) 후 조건이 맞을 때만 `seekerclaw-execute-approved`
5. 이상 징후 시 `seekerclaw-ops-recovery`

## 4) 운영 규칙

- `critical`에 해당하는 상태면 **실행·스왑 금지**
- 텔레그램(또는 앱)에서 명시적 승인 없이 **라이브 스왑 금지**
- 소액·페이퍼(시뮬레이션) 모드가 있다면 라이브 전에 반드시 **연습 모드**(실거래 없이 절차만)로 검증
- `memory/tradebot-idempotency.json`은 `records` 맵으로 `approval_id`별 상태를 둔다(`docs/idempotency-policy.md`). 재시작 후에도 동일 승인이면 **기존 키 재사용·성공 시 재실행 없음**; `last_result.ok`가 거짓이면 **새 키 없이** `seekerclaw-ops-recovery`로 처리한다.

## 5) 점검 포인트

- 워크스페이스 `memory/tradebot-state.json`(권장 경로, 스킬 본문 참고)의 `severity`, `alerts`, 게이트 필드
- SeekerClaw 로그 또는 텔레그램 `/logs` 등으로 최근 거절·실패 메시지 확인
- **수동 검증:** `memory/tradebot-state.json`을 잠시 제거하거나 이름을 바꾼 뒤 `seekerclaw-status-guard`만 실행한다. 이 경우 출력 `decision`은 **`hold` 또는 `kill_and_hold`이면 정상**이고, **`proceed`이면 오류**(폴백이 반영되지 않은 것으로 본다).
- **수동 검증(승인 없는 실행):** `seekerclaw-execute-approved`를 **`/approve`(또는 앱의 동등한 명시적 승인 UI) 없이** 호출하는 시나리오를 만든다. 출력은 **`decision`이 `hold`**, `reasons`에 `approval_missing` 등 승인 누락 사유, `actions`에 `request_explicit_approval`이 포함되는 것이 정상이다(라이브 스왑 없음).

## 6) 실거래 전 — 하트비트 **10번 연습** 점검

README에서는 **「드라이런」**이라 부르기도 하는데, 뜻은 **연습 모드**(실제 돈 나가는 스왑 없음, 승인도 일부러 안 보내는 식으로 통제)입니다.

- **연습 조건**에서 하트비트를 **10번** 연속 돌립니다(설정에 따라 명시적 승인 없음·`live_swap_blocked` 유지 등).
- **통과 기준:** 10번이 지난 뒤 **승인 없이** `seekerclaw-execute-approved`가 **라이브 체결을 시도한 횟수 = 0**이어야 합니다. 로그, `memory/tradebot-heartbeat-log.md`, 필요 시 `memory/tradebot-idempotency.json`으로 세면 됩니다.
- 0이 아니면 `HEARTBEAT.md` 블록·가드·승인 게이트를 고친 뒤 **다시 10번부터** 반복합니다.
