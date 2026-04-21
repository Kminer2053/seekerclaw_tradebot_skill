# Idempotency Policy (온디바이스)

**워크스페이스 파일**로 동일 승인 건의 중복 체결을 방지합니다.

## 목적

- 하트비트가 같은 승인에 대해 두 번 스왑을 시도하지 않음
- 앱 재시작·재시도 후에도 동일 승인 ID에는 동일 키 유지

## 키 규칙

권장 포맷:

```text
exec-approved:{approval_id}:{date_utc}
```

`date_utc`는 UTC 기준 `YYYY-MM-DD`.

## 저장 파일

권장 경로: `memory/tradebot-idempotency.json`

최상위 `records` 맵의 키는 `approval_id`와 동일하게 둔다.

```json
{
  "records": {
    "ap-1234": {
      "idempotency_key": "exec-approved:ap-1234:2026-04-22",
      "last_result": {
        "ok": true,
        "tx_signature": "sig1"
      }
    }
  }
}
```

## 재시작 동작

앱·프로세스를 재시작한 뒤에도 **동일 `approval_id`**로 들어오면, 파일에 이미 있는 레코드를 읽어 **기존 `idempotency_key`를 그대로 재사용**한다. 새 키를 만들지 않는다. `last_result.ok === true`이면 이번 주기에서 **라이브 스왑을 다시 실행하지 않는다**(이미 완료로 간주).

## 절차

1. 실행 직전 `approval_id` 확인
2. `records[approval_id]` 조회
3. 없으면 키 생성 후 `records`에 해당 항목을 추가하고 스왑 시도
4. 있고 `last_result.ok === true`이면 **재실행하지 않음**(idempotent 완료)
5. 있고 `last_result.ok === false`이면 **새 idempotency 키를 발급하지 않음**. 수동 복구·정리 후 `seekerclaw-ops-recovery` 절차로 넘긴다.

## 금지

- 다른 승인 ID에 이전 키 재사용
- 매 주기·매 루프마다 임의 문자열로 키를 새로 만들거나 갱신하는 것
