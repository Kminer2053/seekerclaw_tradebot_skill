<div align="center">

# SeekerClaw Tradebot Skills

**Approval-first trading orchestration as SeekerClaw skills.**  
Runs on-device with SeekerClaw. No separate trading backend; state stays in workspace files.

[![SeekerClaw](https://img.shields.io/badge/SeekerClaw-upstream-111827?style=flat-square&logo=github)](https://github.com/sepivip/SeekerClaw)
[![Skill format](https://img.shields.io/badge/Skill%20format-SKILL--FORMAT.md-0F766E?style=flat-square)](https://github.com/sepivip/SeekerClaw/blob/main/SKILL-FORMAT.md)
[![License](https://img.shields.io/badge/License-MIT-blue?style=flat-square)](LICENSE)

[English](#english) · [한국어](#한국어-korean)

</div>

---

## English

### What this is

A small **skill pack** plus **operator docs** for running a rule-based, **human-approved** trading loop inside [SeekerClaw](https://github.com/sepivip/SeekerClaw) on Android. State, gates, and idempotency live in workspace files under `memory/`. The agent follows `HEARTBEAT.md` and the four skills in `skills/`.

### Design principles

| Principle | What it means |
|-----------|----------------|
| Skill-only | No custom server. SeekerClaw built-ins (wallet, Jupiter, Telegram) only. |
| Explicit approval | Live swaps only after `/approve` or an equivalent explicit UI in the app. |
| Guard first | `seekerclaw-status-guard` runs before drafts or execution. |
| Idempotent execution | One stable key per `approval_id` in `memory/tradebot-idempotency.json`. |
| Non-destructive heartbeat | Append one marked section to the bottom of `HEARTBEAT.md`. Do not replace your existing heartbeat. |

### Repository layout

```
skills/
  seekerclaw-status-guard/     # Gate: proceed / hold / kill_and_hold
  seekerclaw-draft-and-approve/
  seekerclaw-execute-approved/
  seekerclaw-ops-recovery/
docs/
  quickstart.md                # Install and first run
  heartbeat-template.md        # Append-only block for HEARTBEAT.md (preserves existing tasks)
  heartbeat-flow.md
  OPERATIONS_CHECKLIST.md      # Pre-flight, per-cycle, incidents, resume
  idempotency-policy.md
  contracts/skill-io-contracts.md
```

### Quick start

1. Copy the `skills/seekerclaw-*` folders into your SeekerClaw workspace `skills/` (or install via your usual SeekerClaw skill flow).
2. Open your workspace `HEARTBEAT.md`, **keep all existing content**, then paste the full block from `docs/heartbeat-template.md` (including `<!-- seekerclaw-tradebot-pack:begin/end -->`) at the **end** of the file. If the markers already exist, replace only the content between them once.
3. Read `docs/quickstart.md` and `docs/OPERATIONS_CHECKLIST.md` before any live size.

### Documentation map

| Doc | Purpose |
|-----|---------|
| [docs/quickstart.md](docs/quickstart.md) | Short path from clone to first safe cycle |
| [docs/OPERATIONS_CHECKLIST.md](docs/OPERATIONS_CHECKLIST.md) | Live checklist: `hold`, `kill_and_hold`, `recover_and_resume` |
| [docs/heartbeat-template.md](docs/heartbeat-template.md) | Append-only trading checklist for `HEARTBEAT.md` |
| [docs/idempotency-policy.md](docs/idempotency-policy.md) | On-device idempotency file rules |
| [docs/contracts/skill-io-contracts.md](docs/contracts/skill-io-contracts.md) | Structured JSON shapes between skills |

### Safety

SeekerClaw can move real funds. This pack is **not** financial advice. Start small, verify behavior in simulation or paper mode if your build supports it, and never treat chat text as approval unless it goes through the app’s explicit approval path.

### License

[MIT](LICENSE)

### Upstream

Skill file shape and loading behavior follow SeekerClaw’s [SKILL-FORMAT.md](https://github.com/sepivip/SeekerClaw/blob/main/SKILL-FORMAT.md).

---

## 한국어 (Korean)

### 이 저장소는 무엇인가요?

[SeekerClaw](https://github.com/sepivip/SeekerClaw) 안드로이드 온디바이스 에이전트 안에서 동작하는 **규칙 기반·승인 기반** 트레이딩 루프를 위한 **스킬 묶음**과 **운영 문서**입니다. 별도 트레이딩용 백엔드 서버는 두지 않고, 상태·게이트·멱등성은 워크스페이스 `memory/` 파일로 관리합니다. `HEARTBEAT.md`에는 **기존 하트비트를 유지한 채 하단에만** 트레이딩 팩 구간을 추가하고, `skills/` 네 스킬이 그 절차를 수행합니다.

### 설계 원칙

| 원칙 | 의미 |
|------|------|
| 스킬 전용 | 커스텀 서버 없음. SeekerClaw 기본 기능(지갑, Jupiter, 텔레그램 등)만 사용. |
| 명시적 승인 | 라이브 스왑은 앱의 `/approve` 또는 **동등한 명시적 승인 UI** 이후에만. |
| 가드 우선 | 초안·실행 전에 `seekerclaw-status-guard` 실행. |
| 멱등 실행 | `approval_id`당 안정적인 키 하나(`memory/tradebot-idempotency.json`). |
| 하트비트 비파괴 설치 | `HEARTBEAT.md` **맨 아래**에만 `seekerclaw-tradebot-pack` 블록을 붙인다. 기존 섹션은 삭제하지 않는다. |

### 디렉터리 구조

```
skills/
  seekerclaw-status-guard/     # 게이트: proceed / hold / kill_and_hold
  seekerclaw-draft-and-approve/
  seekerclaw-execute-approved/
  seekerclaw-ops-recovery/
docs/
  quickstart.md                # 설치·첫 주기
  heartbeat-template.md        # HEARTBEAT.md 하단 추가용 블록(기존 설정 유지)
  heartbeat-flow.md
  OPERATIONS_CHECKLIST.md      # 라이브 전·주기·장애·재개
  idempotency-policy.md
  contracts/skill-io-contracts.md
```

### 빠른 시작

1. `skills/seekerclaw-*` 디렉터리를 SeekerClaw 워크스페이스의 `skills/`에 복사하거나, 평소 쓰는 스킬 설치 방법으로 등록합니다.
2. 워크스페이스 `HEARTBEAT.md`에서 **위쪽 기존 내용은 그대로 두고**, `docs/heartbeat-template.md`의 **전체 블록**(주석 `seekerclaw-tradebot-pack:begin/end` 포함)만 **파일 끝**에 붙입니다. 이미 같은 주석이 있으면 **그 사이만** 최신 템플릿으로 갱신합니다. 하트비트 주기는 앱 설정을 따릅니다.
3. 라이브 전에 `docs/quickstart.md`와 `docs/OPERATIONS_CHECKLIST.md`를 읽습니다.

### 문서 안내

| 문서 | 용도 |
|------|------|
| [docs/quickstart.md](docs/quickstart.md) | 저장소부터 첫 안전 주기까지 짧은 경로 |
| [docs/OPERATIONS_CHECKLIST.md](docs/OPERATIONS_CHECKLIST.md) | 라이브 점검표: `hold`, `kill_and_hold`, `recover_and_resume` |
| [docs/heartbeat-template.md](docs/heartbeat-template.md) | `HEARTBEAT.md` 하단 추가용 체크리스트(기존 설정 보존) |
| [docs/idempotency-policy.md](docs/idempotency-policy.md) | 온디바이스 멱등 파일 규칙 |
| [docs/contracts/skill-io-contracts.md](docs/contracts/skill-io-contracts.md) | 스킬 간 구조화 JSON 계약 |

### 안전 안내

SeekerClaw는 실제 자금을 움직일 수 있습니다. 본 패키지는 **투자 조언이 아닙니다.** 가능하면 시뮬레이션·페이퍼 모드로 검증하고, 채팅 속 일반 동의는 승인으로 치지 마세요. 앱이 정한 **명시적 승인 경로**만 인정합니다.

### 라이선스

[MIT](LICENSE)

### 업스트림

스킬 파일 형식은 SeekerClaw [SKILL-FORMAT.md](https://github.com/sepivip/SeekerClaw/blob/main/SKILL-FORMAT.md)를 따릅니다.
