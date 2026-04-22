---
name: seekerclaw-execute-approved
description: "운영자가 명시적으로 승인한 시나리오만, 상태 가드를 다시 통과한 뒤 SeekerClaw 내장 지갑·Jupiter 스왑으로 실행할 때 사용한다."
version: "0.2.0"
emoji: "⚡"
requires:
  bins: []
  env: []
---

# seekerclaw-execute-approved

## 목적

**승인된** 시나리오에 대해 단일 실행을 수행한다. 스킬·워크스페이스 메모리와 SeekerClaw 내장 경로만 사용한다.

## 선행 조건

- **승인 원천 고정:** 유효한 실행은 텔레그램 **`/approve`**(또는 앱이 제공하는 **동등한 명시적 승인 UI**—전용 승인 버튼·확인 화면 등)에서만 낸 승인만 인정한다. 일반 채팅·암시적 동의·문맥상의 “OK”는 승인이 **아니다**.
- **승인 대상 식별:** 승인 시 `/approve <scenario_id>`를 우선 사용한다.
  - `<scenario_id>`가 있으면 해당 ID와 `memory/tradebot-pending-scenario.json`의 pending 항목을 매칭해 승인한다.
  - `<scenario_id>`가 없으면 pending이 정확히 1건일 때만 승인으로 인정한다.
  - pending이 0건이면 `decision=hold`, `reasons`에 `approval_missing` 포함.
  - pending이 2건 이상이면 `decision=hold`, `reasons`에 `approval_target_ambiguous` 포함.
- **멱등 경로(필수):** 라이브 스왑 전에 `approval_id`(또는 `docs/idempotency-policy.md` 등 정책이 정한 **동등한 승인·정책 식별자**)를 **반드시** 확보하고 기록한다. 식별자를 안전하게 고정·기록하기 전에는 스왑을 시작하지 않는다.
- 실행 직전에 `seekerclaw-status-guard`를 **반드시** 다시 실행해 `decision=proceed`
- `memory/tradebot-idempotency.json`의 `records[approval_id]`를 확인한다. 구조는 `docs/idempotency-policy.md`를 따른다. 이미 `last_result.ok === true`이면 **즉시 종료**(라이브 재실행 없음). **앱 재시작 후에도** 동일 `approval_id`면 파일에 있는 **기존 `idempotency_key`만** 쓰고 새 키를 만들지 않는다. `last_result.ok === false`이면 **새 키를 발급하지 않고** `seekerclaw-ops-recovery`로 넘긴다.
- `entry_blocked`, `live_swap_blocked`가 허용 상태

## 기본 절차

1. `docs/idempotency-policy.md`에 따라 idempotency 키를 확보한다.
2. SeekerClaw가 제공하는 **Solana / Jupiter 스왑**(또는 동등 내장 도구)으로 트랜잭션을 구성·전송한다. 페이퍼/시뮬레이션 모드가 있으면 라이브 전에 먼저 사용한다.
3. 결과 서명(`tx_signature` 등)을 idempotency 파일과 `memory/tradebot-state.json`에 반영한다.
4. 실패 시 오류 메시지를 구조화해 `seekerclaw-ops-recovery`로 넘길 수 있게 남긴다.

## 출력 계약

```json
{
  "decision": "executed | hold",
  "reasons": ["..."],
  "actions": ["verify_wallet_tx", "append_memory_log"],
  "evidence": {
    "approval_id": "...",
    "idempotency_key_used": true,
    "tx_signature": "...",
    "simulation_only": false
  }
}
```

### 승인 누락 시(출력 계약 예시)

`/approve`(또는 동등 UI)로 확보된 승인이 없으면 실행하지 않고 아래 형태로 **보류**한다.

```json
{
  "decision": "hold",
  "reasons": ["approval_missing"],
  "actions": ["request_explicit_approval"]
}
```

### 승인 대상 불명확 시(출력 계약 예시)

pending이 여러 건인데 `/approve`에 `scenario_id`가 없으면 실행하지 않고 아래 형태로 **보류**한다.

```json
{
  "decision": "hold",
  "reasons": ["approval_target_ambiguous"],
  "actions": ["request_explicit_approval_with_scenario_id"]
}
```

## 금지

- **`/approve` 또는 동등 명시적 승인 UI 외**의 경로로 간주되는 “승인”에 기반한 라이브 스왑
- 무승인 라이브 스왑
- 가드 없이 실행
- 동일 주기·동일 승인에 대한 두 번째 라이브 시도(idempotency 위반)
- 매 주기·매 하트비트 루프마다 **임의·새** idempotency 키를 생성하는 행위(파일·정책에 따라 **한 승인당 하나의 키**만 유지)
