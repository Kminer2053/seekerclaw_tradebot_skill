# Quickstart

별도 HTTP 백엔드·엔드포인트나 Node 전용 런타임은 **필요 없습니다**. SeekerClaw 앱 워크스페이스에 스킬과 하트비트만 맞추면 됩니다.

**라이브 전 필수:** [`docs/OPERATIONS_CHECKLIST.md`](OPERATIONS_CHECKLIST.md)(Pre-flight·주기 점검·장애 대응·재개 기준).

## 1) 스킬 설치

- 이 프로젝트의 `skills/seekerclaw-*/` 디렉터리를 기기 워크스페이스의 `skills/` 아래에 두거나, 앱에서 제공하는 스킬 설치 방법(URL·파일 등)으로 등록합니다.
- 각 `SKILL.md`의 `description`이 시맨틱 라우팅에 쓰이므로, 앱 재시작 후 스킬 목록에 로드됐는지 확인합니다.

## 2) 하트비트에 절차 넣기

- 워크스페이스 루트의 `HEARTBEAT.md`에 `docs/heartbeat-template.md`의 **「복사용 운영 블록」만** 붙여 넣고, 주기(예: 5분)는 앱 하트비트 설정에 맞게 둡니다.
- 주기 한도는 템플릿에 명시: `max_live_swap_attempt = 1`, `max_new_draft = 1`(운영 정책 변경 시에만 수정).

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
- 소액·페이퍼(시뮬레이션) 모드가 있다면 라이브 전에 반드시 드라이런
- `memory/tradebot-idempotency.json`은 `records` 맵으로 `approval_id`별 상태를 둔다(`docs/idempotency-policy.md`). 재시작 후에도 동일 승인이면 **기존 키 재사용·성공 시 재실행 없음**; `last_result.ok`가 거짓이면 **새 키 없이** `seekerclaw-ops-recovery`로 처리한다.

## 5) 점검 포인트

- 워크스페이스 `memory/tradebot-state.json`(권장 경로, 스킬 본문 참고)의 `severity`, `alerts`, 게이트 필드
- SeekerClaw 로그 또는 텔레그램 `/logs` 등으로 최근 거절·실패 메시지 확인
- **수동 검증:** `memory/tradebot-state.json`을 잠시 제거하거나 이름을 바꾼 뒤 `seekerclaw-status-guard`만 실행한다. 이 경우 출력 `decision`은 **`hold` 또는 `kill_and_hold`이면 정상**이고, **`proceed`이면 오류**(폴백이 반영되지 않은 것으로 본다).
- **수동 검증(승인 없는 실행):** `seekerclaw-execute-approved`를 **`/approve`(또는 앱의 동등한 명시적 승인 UI) 없이** 호출하는 시나리오를 만든다. 출력은 **`decision`이 `hold`**, `reasons`에 `approval_missing` 등 승인 누락 사유, `actions`에 `request_explicit_approval`이 포함되는 것이 정상이다(라이브 스왑 없음).

## 6) 10주기 드라이런(라이브 전)

- **페이퍼/드라이런 모드**에서 하트비트를 **10주기** 연속 실행한다(명시적 승인·라이브 스왑 없음).
- 검증: 10주기 후 **execute without approval(무승인 `seekerclaw-execute-approved`에 의한 라이브 체결 시도) = 0건**이어야 한다. 로그, `memory/tradebot-heartbeat-log.md`, 필요 시 `memory/tradebot-idempotency.json`으로 집계한다. 0이 아니면 템플릿·가드·승인 게이트를 점검한 뒤 다시 10주기 드라이런한다.
