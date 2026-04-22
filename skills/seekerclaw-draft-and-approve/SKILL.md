---
name: seekerclaw-draft-and-approve
description: "상태 가드가 proceed일 때 새 트레이드 시나리오 초안을 워크스페이스에 기록하고 텔레그램 등으로 운영자 승인을 요청할 때 사용한다."
version: "0.2.0"
emoji: "📝"
requires:
  bins: []
  env: []
---

# seekerclaw-draft-and-approve

## 목적

다음 라이브 실행에 쓸 **시나리오 초안**을 남기고, **명시적 승인**을 받기 위한 대기 상태를 만든다.

## 선행 조건

- `seekerclaw-status-guard` 결과가 `decision=proceed`

## 중복 승인 대기 방지(필수)

`memory/tradebot-pending-scenario.json`이 **존재하고** 내용이 **유효한**(필수 필드·스키마를 만족하는) 승인 대기 시나리오로 판단되면, **새 초안을 쓰지 않는다**. 이번 턴 출력은 아래 계약을 따른다.

- `decision`: `hold`
- `actions`: `notify_operator_existing_pending` 포함(운영자에게 이미 대기 중인 시나리오가 있음을 알림)
- `reasons`: 중복 대기 방지 사유 명시(예: 기존 `draft_scenario_id` 또는 파일 경로)

## 기본 절차

1. 전략 파라미터(입출력 자산, 수량/비율, 슬리피지 한도, 시간 유효성)를 JSON으로 정리한다.
2. `draft_scenario_id`를 생성한다(예: 타임스탬프 + 짧은 랜덤).
3. `memory/tradebot-pending-scenario.json`에 저장한다.
4. 텔레그램(또는 앱 알림)으로 운영자에게 요약 전송: 무엇을, 왜, 어떤 리스크로 제안하는지.
   - 언어: `memory/tradebot-operator-preferences.json`의 `notification_language`를 우선 사용한다.
   - 파일이 없거나 값이 유효하지 않으면 한국어(`ko`)를 기본으로 사용한다.
   - `fallback_language`가 지정되어 있으면 보조 언어로 짧게 병기할 수 있다.
5. 승인 안내 문구에는 `draft_scenario_id`를 반드시 포함한다.
   - 권장 승인 명령: `/approve <draft_scenario_id>`
   - 단일 pending만 있을 때는 `/approve`도 허용할 수 있으나, 다중 pending 가능성을 피하려면 ID 포함 명령을 기본으로 안내한다.
6. 승인은 **텔레그램 `/approve`(또는 앱 동등 명시적 승인 UI)**로만 받는다. 채팅 속 암시적 동의는 승인으로 치지 않는다.

## 출력 계약

- **`decision=await_approval`:** 새 초안을 기록한 뒤 운영자에게 승인 요청을 보낼 때. `actions`에 **`notify_operator_for_approval`를 포함**한다(승인 요청 알림 전용 액션).
- **`decision=hold`:** 보류·중복 대기 등. `actions`는 사유에 맞게 두며, **중복 승인 대기**는 아래 「중복 대기 시 출력 예시」처럼 `notify_operator_existing_pending` 등을 쓴다. **`notify_operator_for_approval`는 `hold`에 쓰지 않는다.**

`await_approval`일 때의 형태:

```json
{
  "decision": "await_approval",
  "reasons": ["..."],
  "actions": ["notify_operator_for_approval"],
  "evidence": {
    "draft_scenario_id": "...",
    "recorded_to": "memory/tradebot-pending-scenario.json"
  }
}
```

## 보류(hold) 예시

- 상태 파일이 깨졌거나 필수 필드가 없음
- **유효한** `memory/tradebot-pending-scenario.json`이 이미 있음 → `notify_operator_existing_pending`
- 하트비트 주기당 드래프트 한도 초과

### 중복 대기 시 출력 예시

```json
{
  "decision": "hold",
  "reasons": ["existing_pending_scenario"],
  "actions": ["notify_operator_existing_pending"],
  "evidence": {
    "recorded_to": "memory/tradebot-pending-scenario.json",
    "draft_scenario_id": "..."
  }
}
```
