# SeekerClaw 스킬 전용 실행 계획

## 목표

별도 `bot-core` 없이 SeekerClaw 기본 기능만으로 다음을 재현합니다.

- 실행 전 안전 게이트(`severity`, 진입/라이브 스왑 가능 여부)
- 시나리오 초안·승인 대기·무승인 실행 금지
- 온디바이스 idempotency 파일로 이중 체결 방지
- 실패 시 표준화된 복구 절차

---

## Phase 1: 계약 고정

- [x] 네 스킬의 구조화 출력 스키마(`docs/contracts/skill-io-contracts.md`)
- [x] 하트비트 순서·속도 제한(`docs/heartbeat-template.md`)
- [x] idempotency 파일 규약(`docs/idempotency-policy.md`)

---

## Phase 2: 스킬 본문

- [x] `skills/*/SKILL.md` — 워크스페이스·내장 지갑·텔레그램 승인 기준으로 서술
- [ ] 실제 기기에서 트리거 문구(`description`) 튜닝

---

## Phase 3: 하트비트 연동

- [ ] 사용자 `HEARTBEAT.md`에 템플릿 반영
- [ ] `memory/tradebot-*.json` 생성·갱신이 주기마다 일어나는지 확인

---

## Phase 4: 검증 시나리오

- [ ] 정상: 초안 → 승인 → 단일 실행
- [ ] 게이트: `hold`일 때 실행 0회
- [ ] idempotency: 동일 승인 재시도 시 스왑 1회만
- [ ] 복구: 한도·거절 후 `ops-recovery`로 안전 정지

---

## Phase 5: 운영 문서

- [ ] `docs/OPERATIONS_CHECKLIST.md`(선택) 추가
- [ ] `docs/quickstart.md`와 동기화

---

## Done Definition

- [ ] `critical` / `hold`에서 라이브 스왑 시도 0건
- [ ] 무승인 라이브 스왑 0건
- [ ] 운영자가 상태 파일·텔레그램만으로 조치 판단 가능
- [ ] 소액/시뮬레이션 드라이런 후 라이브 전환
