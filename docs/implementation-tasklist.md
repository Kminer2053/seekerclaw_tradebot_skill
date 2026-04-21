# Implementation Tasklist

## 범위

이 저장소는 **SeekerClaw 마크다운 스킬 + 문서**만 다룹니다. `bot-core` HTTP 연동, 별도 Node 하트비트 런타임, `pnpm`/`tsx` 실행기는 **범위에서 제외**했습니다.

## Task 1: 스킬 패키지 정리

- [x] `skills/` 네 스킬에서 `BOT_*` 환경 변수·`/v1/*` API 의존 제거
- [x] 워크스페이스 파일·내장 지갑·텔레그램 승인 흐름 기준으로 본문 수정
- [ ] 실제 기기에서 스킬 로드 및 시맨틱 트리거 문구 미세 조정(운영자)

완료 조건: 앱 스킬 화면에 네 스킬이 오류 없이 표시되고, 설명이 의도대로 노출됨.

## Task 2: 상태 파일 규약

- [ ] 워크스페이스에 `memory/tradebot-state.json` 생성 절차를 하트비트 템플릿에 맞춤
- [ ] `status-guard` 스킬의 필드 정의(`severity`, `entry_blocked`, `live_swap_blocked` 등)와 실제 기록 형식 일치

완료 조건: 하트비트 한 주기마다 파일이 갱신되거나, 없을 때 스킬이 안전하게 `hold`로 끝남.

## Task 3: 중복 실행 방지(온디바이스)

- [ ] `docs/idempotency-policy.md`에 따라 `memory/tradebot-idempotency.json`(또는 동등 파일) 운용
- [ ] 동일 `approval_id`에 대해 한 주기·재시작 후에도 동일 실행 키 재사용 확인

완료 조건: 앱 재시작 후에도 같은 승인 건에 대해 이중 체결 시도가 없음.

## Task 4: HEARTBEAT.md 운영 반영

- [x] `docs/heartbeat-template.md` 초안
- [ ] 사용자 워크스페이스 `HEARTBEAT.md`에 붙여 넣고 주기·모델·도구 제한을 본인 환경에 맞게 조정

완료 조건: 하트비트가 스킬 순서대로만 동작하고, 라이브 스왑은 승인 후에만 발생.

## Task 5: 검증 시나리오

- [ ] 정상: 드래프트 → 승인 → 단일 실행
- [ ] 게이트: `hold`일 때 실행 0회
- [ ] 복구: 거절/한도/만료 메시지 후 `ops-recovery` 절차로 정지 또는 재개

## Task 6: 문서

- [x] `README.md`, `docs/quickstart.md` — bot-core 미사용 명시
- [ ] (선택) 운영 체크리스트 `docs/OPERATIONS_CHECKLIST.md` 신규 작성

## 최종 완료 기준

- [ ] `critical` 또는 `hold`에서 라이브 스왑 시도 0건
- [ ] 무승인 라이브 스왑 0건
- [ ] 하트비트·스킬 출력이 구조화되어 로그/메모리에 남음
