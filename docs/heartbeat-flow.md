# Heartbeat Flow

별도 서버 API 없이, SeekerClaw 하트비트가 **스킬 순서**와 **워크스페이스 상태 파일**로 분기합니다. 절차 문구는 `docs/heartbeat-template.md`의 **Tradebot pack** 섹션과 맞춥니다. `HEARTBEAT.md`에는 기존 업무를 유지한 채 그 **아래**에 해당 섹션만 붙인다.

## 주기당 한도(템플릿과 동일)

- `max_live_swap_attempt = 1`
- `max_new_draft = 1`

## 기본 루프

1. `seekerclaw-status-guard` — `memory/tradebot-state.json` 등을 읽고 `proceed` / `hold` / `kill_and_hold` 결정 (`state_file_missing` 등은 템플릿 표준 실패 모드 참고)
2. `hold` 또는 `kill_and_hold`면 **이번 주기 종료**(스왑·체결 없음)
3. `proceed`일 때:
   - 승인 대기 시나리오가 없으면 `seekerclaw-draft-and-approve`로 초안 기록 및 운영자 알림(**max_new_draft**)
  - 명시적 승인이 있으면 `seekerclaw-status-guard`를 다시 실행해 `proceed`일 때만 `seekerclaw-execute-approved`로 실행 시도(**max_live_swap_attempt**). 무승인이면 `approval_missing` 처리로 실행 없음
4. 실패·경보 시 `seekerclaw-ops-recovery` (`rpc_error_burst` 등은 `docs/OPERATIONS_CHECKLIST.md`)
5. 스냅샷을 메모리 파일에 남기고 다음 주기 대기

## 실행 조건(요약)

- `severity !== critical`
- 진입 허용 플래그(스킬에서 정의한 `entry_blocked === false`)
- 라이브 스왑 시 `live_swap_blocked === false` 및 일일 한도·리스크 규칙 통과(`docs/contracts/skill-io-contracts.md`와 각 스킬 출력 계약 동시 확인)

## 재시도·백오프

- 네트워크·RPC 일시 실패: 다음 주기까지 대기, 연속 실패 시 백오프(템플릿 참고)
- 동일 주기 내 **중복 실행 금지**(idempotency 파일 참고)

## 중복 방지

- 동일 `approval_id`에는 동일 실행 키 재사용(`docs/idempotency-policy.md`)
