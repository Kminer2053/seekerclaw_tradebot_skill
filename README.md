<div align="center">

# SeekerClaw Tradebot Skills

**Your trades, your approval. Every time.**

An add-on skill pack for [SeekerClaw](https://github.com/sepivip/SeekerClaw) that makes sure no trade happens without your explicit OK.

[![SeekerClaw](https://img.shields.io/badge/SeekerClaw-upstream-111827?style=flat-square&logo=github)](https://github.com/sepivip/SeekerClaw)
[![Skill format](https://img.shields.io/badge/Skill%20format-SKILL--FORMAT.md-0F766E?style=flat-square)](https://github.com/sepivip/SeekerClaw/blob/main/SKILL-FORMAT.md)
[![License](https://img.shields.io/badge/License-MIT-blue?style=flat-square)](LICENSE)

**Repo:** [github.com/Kminer2053/seekerclaw_tradebot_skill](https://github.com/Kminer2053/seekerclaw_tradebot_skill)

[English](#english) · [한국어](#한국어)

</div>

---

## English

### What is this?

This pack adds **four skills** to your SeekerClaw app. Together they create a simple loop:

```
  ┌─────────────────────────────────────────────────────┐
  │                   Every heartbeat                    │
  │                                                     │
  │  1. Guard    ──▶  "Is it safe to trade right now?"  │
  │  2. Draft    ──▶  Write a trade plan, ping you      │
  │  3. Approve  ──▶  YOU send /approve (or ignore)     │
  │  4. Execute  ──▶  Run the trade (only if approved)  │
  │  5. Recover  ──▶  Handle errors, stay safe          │
  │                                                     │
  └─────────────────────────────────────────────────────┘
```

**No separate server needed.** Everything runs inside SeekerClaw using workspace files and Telegram.

### Key rule

> **Nothing trades without your `/approve`.** Typing "yes" or "do it" in chat does NOT count. Only the `/approve <draft_scenario_id>` command (or the app's official approval button) triggers a real trade.

### Quick glossary

| Word you'll see | What it means |
|-----------------|---------------|
| **Heartbeat** | A timer (e.g. every 5 minutes) that wakes the agent to run its checklist. |
| **Guard** | The first check: "Is it safe to continue?" Reads the state file and decides yes/no. |
| **Draft** | A trade plan the agent writes and sends to your Telegram for review. |
| **Live swap** | A real on-chain trade that moves actual funds. Only happens after your approval. |
| **Practice mode** | Running the full loop with live swaps blocked, so you can watch without risk. |

### What's in the box

| Item | Purpose |
|------|---------|
| 4 skills in `skills/seekerclaw-*` | Guard, Draft + Approve, Execute, Recovery |
| `docs/HEARTBEAT-APPEND.md` | Text block you paste into your `HEARTBEAT.md` |
| `docs/quickstart.md` | Step-by-step setup and 10-cycle practice check |
| `docs/OPERATIONS_CHECKLIST.md` | Pre-flight checks, incident response, resume criteria |

### A week in the life (example)

**Monday (one-time setup)**
Download this repo, import the four skills into SeekerClaw, paste the heartbeat block, and create your state file with `live_swap_blocked: true`. Set the heartbeat to every 10 minutes.

**Tuesday (practice mode)**
The agent runs the guard every 10 minutes. Most cycles return "hold" because gates are tight. No real trades happen. You check the heartbeat log to confirm the loop is alive.

**Wednesday (a draft arrives)**
Telegram: *"Trade plan: swap 0.01 SOL to USDC. Reply /approve scenario-abc123 to proceed."*
You don't like it, so you do nothing. The draft expires.

**Thursday (you approve)**
A new draft looks good. You send `/approve scenario-abc123` in Telegram. Next heartbeat: the guard passes, the execute skill runs the swap once.

**Friday (emergency stop)**
Something feels off. You edit the state file: set `severity` to `critical`. The guard immediately blocks all trades until you fix it.

### Before you start

- SeekerClaw is installed and working (API key, Telegram bot, workspace access).
- You understand that **live swaps can lose real money**. Start with practice mode.
- Recommended: read [SeekerClaw PROJECT.md](https://github.com/sepivip/SeekerClaw/blob/main/PROJECT.md) for skill basics.

---

### Installation

Pick **one** method. After installing, check that all four skills appear in the app's Skills list.

#### Method A — ZIP import (recommended)

1. [Download ZIP](https://github.com/Kminer2053/seekerclaw_tradebot_skill/archive/refs/heads/main.zip) and extract it.
2. Go to `seekerclaw_tradebot_skill-main/skills/`.
3. Select these four folders and zip them into `tradebot-skills.zip`:
   - `seekerclaw-status-guard`
   - `seekerclaw-draft-and-approve`
   - `seekerclaw-execute-approved`
   - `seekerclaw-ops-recovery`
4. Transfer the ZIP to your phone.
5. In SeekerClaw: **Skills** → **Import** → select `tradebot-skills.zip`.

> If the app only accepts one skill per ZIP, zip each folder separately and import four times.

#### Method B — Import one file at a time

Download each SKILL.md and import it from the Skills screen:

- [seekerclaw-status-guard](https://raw.githubusercontent.com/Kminer2053/seekerclaw_tradebot_skill/main/skills/seekerclaw-status-guard/SKILL.md)
- [seekerclaw-draft-and-approve](https://raw.githubusercontent.com/Kminer2053/seekerclaw_tradebot_skill/main/skills/seekerclaw-draft-and-approve/SKILL.md)
- [seekerclaw-execute-approved](https://raw.githubusercontent.com/Kminer2053/seekerclaw_tradebot_skill/main/skills/seekerclaw-execute-approved/SKILL.md)
- [seekerclaw-ops-recovery](https://raw.githubusercontent.com/Kminer2053/seekerclaw_tradebot_skill/main/skills/seekerclaw-ops-recovery/SKILL.md)

#### Method C — Tell your agent to install

Paste this into your SeekerClaw chat:

```text
Install these four skills from URL, one after another, and confirm each with /skills:
1. https://raw.githubusercontent.com/Kminer2053/seekerclaw_tradebot_skill/main/skills/seekerclaw-status-guard/SKILL.md
2. https://raw.githubusercontent.com/Kminer2053/seekerclaw_tradebot_skill/main/skills/seekerclaw-draft-and-approve/SKILL.md
3. https://raw.githubusercontent.com/Kminer2053/seekerclaw_tradebot_skill/main/skills/seekerclaw-execute-approved/SKILL.md
4. https://raw.githubusercontent.com/Kminer2053/seekerclaw_tradebot_skill/main/skills/seekerclaw-ops-recovery/SKILL.md
```

#### Method D — Manual copy (advanced)

Copy the four folders from `skills/` into your SeekerClaw workspace's `skills/` directory. Restart the agent.

---

### Setting up the heartbeat

After installing skills, you need to tell the agent what to do each cycle:

1. Open `HEARTBEAT.md` in your workspace. **Don't delete anything already there.**
2. Open [docs/HEARTBEAT-APPEND.md](docs/HEARTBEAT-APPEND.md) and copy the **entire file**.
3. Paste it at the **very bottom** of `HEARTBEAT.md`.
4. Set the heartbeat interval in the app (e.g. every 5-15 minutes).

> If `seekerclaw-tradebot-pack` markers already exist, replace only the content between them.

---

### Configuration

#### State file — `memory/tradebot-state.json`

This file controls what the guard allows. Start with practice mode:

```json
{
  "severity": "warn",
  "entry_blocked": true,
  "live_swap_blocked": true,
  "alerts": ["practice_mode_setup"],
  "recommended_actions": []
}
```

When you're ready to let the agent draft plans (but still block live trades):

```json
{
  "severity": "ok",
  "entry_blocked": false,
  "live_swap_blocked": true,
  "alerts": [],
  "recommended_actions": ["request_approval"]
}
```

#### Notification language — `memory/tradebot-operator-preferences.json`

Control what language Telegram notifications arrive in:

```json
{
  "notification_language": "ko",
  "fallback_language": "en"
}
```

Default is Korean (`ko`) if this file doesn't exist.

---

### How it works

```
  Heartbeat timer fires
        │
        ▼
  ┌─────────────┐     ┌──────────┐
  │ Status Guard │────▶│ hold?    │──▶ Stop this cycle.
  └─────────────┘     │ proceed? │
                      └────┬─────┘
                           ▼
                 ┌──────────────────┐
                 │ Draft & Approve  │──▶ Save plan, ping Telegram
                 └──────────────────┘
                           │
              You send /approve <id>
                           │
                           ▼
                 ┌──────────────────┐
                 │ Execute Approved │──▶ Run the swap ONCE
                 └──────────────────┘
                           │
                    Error? ─┤
                           ▼
                 ┌──────────────────┐
                 │  Ops Recovery    │──▶ Log error, adjust state
                 └──────────────────┘
```

### Safety principles

| Principle | What it means |
|-----------|---------------|
| Approval first | No trade without `/approve <draft_scenario_id>`. Chat messages don't count. |
| Guard before everything | The guard runs before any draft or execution. |
| One trade per approval | Each approval can only trigger one swap (idempotency). |
| No separate server | Everything stays in SeekerClaw workspace files. |
| Non-destructive setup | You only append to `HEARTBEAT.md`, never overwrite. |

### Repository layout

```
skills/
  seekerclaw-status-guard/SKILL.md      — Safety check
  seekerclaw-draft-and-approve/SKILL.md — Write plan + request approval
  seekerclaw-execute-approved/SKILL.md  — Run approved trade
  seekerclaw-ops-recovery/SKILL.md      — Handle errors

docs/
  quickstart.md              — Setup guide + practice check
  HEARTBEAT-APPEND.md        — Text to paste into HEARTBEAT.md
  heartbeat-template.md      — Detailed heartbeat explanation
  OPERATIONS_CHECKLIST.md    — Operations checklist
  idempotency-policy.md      — Duplicate prevention rules
  contracts/
    skill-io-contracts.md    — Skill input/output formats
```

### Documentation guide

| Document | What it covers |
|----------|----------------|
| [quickstart.md](docs/quickstart.md) | Installation, heartbeat setup, 10-cycle practice check |
| [OPERATIONS_CHECKLIST.md](docs/OPERATIONS_CHECKLIST.md) | Pre-flight checks, per-cycle checks, incident response |
| [HEARTBEAT-APPEND.md](docs/HEARTBEAT-APPEND.md) | The exact text to paste into your heartbeat file |
| [heartbeat-template.md](docs/heartbeat-template.md) | Background explanation for the heartbeat block |
| [idempotency-policy.md](docs/idempotency-policy.md) | How duplicate trades are prevented |
| [skill-io-contracts.md](docs/contracts/skill-io-contracts.md) | JSON formats for skill inputs and outputs |

### Agent runbook (copy-paste)

Give this to your agent to auto-configure the workspace:

```text
You are configuring the workspace for the seekerclaw_tradebot_skill pack.

1) Install all four skills if missing:
   seekerclaw-status-guard, seekerclaw-draft-and-approve,
   seekerclaw-execute-approved, seekerclaw-ops-recovery
   Use skill_install with raw SKILL.md URLs from:
   https://github.com/Kminer2053/seekerclaw_tradebot_skill/tree/main/skills

2) Append the full contents of this file to the end of HEARTBEAT.md (keep existing content):
   https://raw.githubusercontent.com/Kminer2053/seekerclaw_tradebot_skill/main/docs/HEARTBEAT-APPEND.md

3) Do not store secrets in memory/*.json.

4) Before going live, tell the user to complete the 10-cycle practice check
   from docs/quickstart.md and review docs/OPERATIONS_CHECKLIST.md.

5) Only /approve <draft_scenario_id> or the app's explicit approval UI counts
   as approval. Never treat chat messages as approval.
```

### Safety warning

SeekerClaw can move real funds. This pack is **not** financial advice. Always start with practice mode, verify with small amounts, and never treat casual chat as trade approval.

### License

[MIT](LICENSE)

### Upstream

Skill format: [SKILL-FORMAT.md](https://github.com/sepivip/SeekerClaw/blob/main/SKILL-FORMAT.md)

---

## 한국어

### 이게 뭔가요?

이 패키지는 [SeekerClaw](https://github.com/sepivip/SeekerClaw) 앱에 **스킬 4개**를 추가합니다. 이 스킬들이 함께 돌아가면서 아래와 같은 흐름을 만들어 줍니다:

```
  ┌──────────────────────────────────────────────────┐
  │              하트비트가 돌 때마다                    │
  │                                                  │
  │  1. 가드     ──▶  "지금 거래해도 안전한가?"         │
  │  2. 초안     ──▶  거래 계획을 쓰고 텔레그램 알림     │
  │  3. 승인     ──▶  내가 /approve 보내기 (또는 무시)  │
  │  4. 실행     ──▶  승인된 거래만 1회 실행             │
  │  5. 복구     ──▶  오류 처리, 안전하게 정지           │
  │                                                  │
  └──────────────────────────────────────────────────┘
```

**별도 서버가 필요 없습니다.** SeekerClaw 안에서 워크스페이스 파일과 텔레그램만으로 돌아갑니다.

### 핵심 규칙

> **내가 `/approve`를 보내기 전에는 어떤 거래도 실행되지 않습니다.** "해줘", "좋아" 같은 일반 채팅은 승인으로 안 셉니다. 반드시 `/approve <draft_scenario_id>` 명령(또는 앱의 공식 승인 버튼)을 써야 합니다.

### 쉬운 용어 정리

| 문서에 나오는 말 | 뜻 |
|-----------------|-----|
| **하트비트** | 정해진 간격(예: 5분)마다 에이전트가 점검 절차를 한 바퀴 도는 것. |
| **가드** | 첫 번째 점검: "지금 진행해도 되나?" 상태 파일을 읽고 판단합니다. |
| **초안** | 에이전트가 만든 거래 계획. 텔레그램으로 보내서 승인을 기다립니다. |
| **라이브 스왑** | 실제 자산이 움직이는 거래. 승인 없이는 절대 실행되면 안 됩니다. |
| **연습 모드** | 라이브 스왑을 막아 둔 채로 전체 절차만 돌려 보는 상태. 돈은 안 나갑니다. |

### 들어있는 것

| 구성 요소 | 역할 |
|-----------|------|
| `skills/seekerclaw-*` 스킬 4개 | 가드 / 초안 작성 + 승인 요청 / 실행 / 오류 복구 |
| `docs/HEARTBEAT-APPEND.md` | `HEARTBEAT.md`에 그대로 붙여 넣는 블록 |
| `docs/quickstart.md` | 설치 순서 + 10회 연습 점검 |
| `docs/OPERATIONS_CHECKLIST.md` | 운영 전 점검, 장애 대응, 재개 기준 |

### 실사용 예시 — 민수 씨의 첫째 주

**월요일 (한 번만 하는 설정)**
ZIP을 받아서 스킬 4개를 가져오고, `HEARTBEAT.md` 맨 아래에 하트비트 블록을 붙입니다. 상태 파일은 `live_swap_blocked: true`로 만들어 놓습니다. 하트비트는 10분 간격.

**화요일 (연습 모드)**
10분마다 가드가 돌지만, 설정이 보수적이라 대부분 "보류(hold)" 입니다. 텔레그램에 초안이 안 올 수도 있습니다. 하트비트 로그에 줄이 쌓이면 "절차는 도는데 돈은 안 나감" 상태가 맞습니다.

**수요일 (초안이 왔지만 거절)**
텔레그램: *"USDC 2만 원어치를 SOL로 바꿀 초안입니다. `/approve scenario-abc123`을 보내면 진행합니다."*
마음에 안 들어서 **아무것도 안 합니다.** 초안은 자동으로 넘어갑니다.

**목요일 (승인)**
새 초안이 괜찮습니다. 텔레그램에 `/approve scenario-abc123`을 보냅니다. 다음 하트비트에서 가드 통과 → 스왑 1회 실행.

**금요일 (긴급 멈춤)**
뭔가 이상합니다. 상태 파일에서 `severity`를 `critical`로 바꿉니다. 가드가 즉시 모든 거래를 막습니다.

### 시작하기 전에

- SeekerClaw가 설치되어 있고 API 키, 텔레그램 봇, 워크스페이스가 준비되어 있어야 합니다.
- **라이브 스왑은 실제 자산 손실로 이어질 수 있습니다.** 반드시 연습 모드부터 시작하세요.
- 권장: [SeekerClaw PROJECT.md](https://github.com/sepivip/SeekerClaw/blob/main/PROJECT.md)에서 스킬 설치 방법을 미리 확인하세요.

---

### 설치 방법

네 가지 중 하나만 선택하세요. 설치 후 앱의 스킬 목록에 4개가 모두 나타나는지 확인합니다.

#### 방법 A — ZIP으로 한 번에 (추천)

1. [ZIP 다운로드](https://github.com/Kminer2053/seekerclaw_tradebot_skill/archive/refs/heads/main.zip) 후 압축을 풉니다.
2. `seekerclaw_tradebot_skill-main/skills/` 폴더로 이동합니다.
3. 아래 4개 폴더를 선택해서 `tradebot-skills.zip`으로 압축합니다:
   - `seekerclaw-status-guard`
   - `seekerclaw-draft-and-approve`
   - `seekerclaw-execute-approved`
   - `seekerclaw-ops-recovery`
4. ZIP을 폰으로 옮긴 뒤, SeekerClaw 앱 → **스킬** → **가져오기**에서 선택합니다.

> 앱이 ZIP당 스킬 1개만 받는다면, 폴더를 하나씩 따로 압축해서 4번 가져오기 하세요.

#### 방법 B — 파일 하나씩 가져오기

아래 링크에서 파일을 저장한 뒤 스킬 화면에서 가져오기:

- [seekerclaw-status-guard](https://raw.githubusercontent.com/Kminer2053/seekerclaw_tradebot_skill/main/skills/seekerclaw-status-guard/SKILL.md)
- [seekerclaw-draft-and-approve](https://raw.githubusercontent.com/Kminer2053/seekerclaw_tradebot_skill/main/skills/seekerclaw-draft-and-approve/SKILL.md)
- [seekerclaw-execute-approved](https://raw.githubusercontent.com/Kminer2053/seekerclaw_tradebot_skill/main/skills/seekerclaw-execute-approved/SKILL.md)
- [seekerclaw-ops-recovery](https://raw.githubusercontent.com/Kminer2053/seekerclaw_tradebot_skill/main/skills/seekerclaw-ops-recovery/SKILL.md)

#### 방법 C — 에이전트한테 설치 시키기

아래 텍스트를 SeekerClaw 채팅에 통째로 붙여 넣으세요:

```text
아래 4개 SKILL.md URL을 순서대로 skill_install로 설치해 줘.
각 설치 후 /skills에서 확인해 줘.

1. https://raw.githubusercontent.com/Kminer2053/seekerclaw_tradebot_skill/main/skills/seekerclaw-status-guard/SKILL.md
2. https://raw.githubusercontent.com/Kminer2053/seekerclaw_tradebot_skill/main/skills/seekerclaw-draft-and-approve/SKILL.md
3. https://raw.githubusercontent.com/Kminer2053/seekerclaw_tradebot_skill/main/skills/seekerclaw-execute-approved/SKILL.md
4. https://raw.githubusercontent.com/Kminer2053/seekerclaw_tradebot_skill/main/skills/seekerclaw-ops-recovery/SKILL.md
```

#### 방법 D — 직접 복사 (개발자용)

`skills/` 안의 4개 폴더를 워크스페이스의 `skills/`에 복사하고 에이전트를 재시작합니다.

---

### 하트비트 설정

스킬 설치 후, 매 주기마다 에이전트가 할 일을 알려줘야 합니다:

1. 워크스페이스의 `HEARTBEAT.md`를 엽니다. **기존 내용은 지우지 마세요.**
2. [docs/HEARTBEAT-APPEND.md](docs/HEARTBEAT-APPEND.md)를 열어서 **전체를 복사**합니다.
3. `HEARTBEAT.md` **맨 아래**에 붙여 넣습니다.
4. 앱에서 하트비트 주기를 설정합니다 (예: 5~15분).

> 이미 `seekerclaw-tradebot-pack` 마커가 있다면 그 사이 내용만 교체하세요.

---

### 설정

#### 상태 파일 — `memory/tradebot-state.json`

이 파일로 가드의 동작을 제어합니다. 처음에는 연습 모드로 시작하세요:

```json
{
  "severity": "warn",
  "entry_blocked": true,
  "live_swap_blocked": true,
  "alerts": ["practice_mode_setup"],
  "recommended_actions": []
}
```

초안 작성은 허용하되 실거래는 막으려면:

```json
{
  "severity": "ok",
  "entry_blocked": false,
  "live_swap_blocked": true,
  "alerts": [],
  "recommended_actions": ["request_approval"]
}
```

#### 알림 언어 — `memory/tradebot-operator-preferences.json`

텔레그램 알림을 어떤 언어로 받을지 설정합니다:

```json
{
  "notification_language": "ko",
  "fallback_language": "en"
}
```

파일이 없으면 기본값은 한국어(`ko`)입니다.

---

### 작동 흐름

```
  하트비트 타이머 작동
        │
        ▼
  ┌─────────────┐     ┌──────────┐
  │  상태 가드   │────▶│ 보류?    │──▶ 이번 주기 종료
  └─────────────┘     │ 진행?    │
                      └────┬─────┘
                           ▼
                 ┌──────────────────┐
                 │ 초안 작성 + 승인  │──▶ 계획 저장, 텔레그램 알림
                 └──────────────────┘
                           │
              /approve <id> 를 보냄
                           │
                           ▼
                 ┌──────────────────┐
                 │  승인 후 실행     │──▶ 스왑 1회 실행
                 └──────────────────┘
                           │
                    오류? ──┤
                           ▼
                 ┌──────────────────┐
                 │   오류 복구      │──▶ 오류 기록, 상태 조정
                 └──────────────────┘
```

### 설계 원칙

| 원칙 | 의미 |
|------|------|
| 승인 우선 | `/approve <draft_scenario_id>` 없이는 거래 없음. 채팅 메시지는 승인이 아님. |
| 가드 먼저 | 모든 초안 작성과 실행 전에 가드가 먼저 돌아감. |
| 한 승인 = 한 거래 | 같은 승인으로 두 번 실행되지 않음 (멱등성). |
| 서버 불필요 | 모든 것이 SeekerClaw 워크스페이스 파일 안에서 처리됨. |
| 비파괴적 설치 | `HEARTBEAT.md`에 내용을 추가만 하고, 기존 내용을 덮어쓰지 않음. |

### 저장소 구조

```
skills/
  seekerclaw-status-guard/SKILL.md      — 안전 점검
  seekerclaw-draft-and-approve/SKILL.md — 거래 계획 작성 + 승인 요청
  seekerclaw-execute-approved/SKILL.md  — 승인된 거래 실행
  seekerclaw-ops-recovery/SKILL.md      — 오류 복구

docs/
  quickstart.md              — 설치 가이드 + 연습 점검
  HEARTBEAT-APPEND.md        — HEARTBEAT.md에 붙일 블록
  heartbeat-template.md      — 하트비트 상세 설명
  OPERATIONS_CHECKLIST.md    — 운영 체크리스트
  idempotency-policy.md      — 중복 실행 방지 규칙
  contracts/
    skill-io-contracts.md    — 스킬 입출력 형식
```

### 문서 안내

| 문서 | 다루는 내용 |
|------|------------|
| [quickstart.md](docs/quickstart.md) | 설치, 하트비트 설정, 10회 연습 점검 |
| [OPERATIONS_CHECKLIST.md](docs/OPERATIONS_CHECKLIST.md) | 운영 전 점검, 주기별 점검, 장애 대응 |
| [HEARTBEAT-APPEND.md](docs/HEARTBEAT-APPEND.md) | 하트비트 파일에 붙일 텍스트 |
| [heartbeat-template.md](docs/heartbeat-template.md) | 하트비트 블록의 상세 배경 설명 |
| [idempotency-policy.md](docs/idempotency-policy.md) | 중복 거래 방지 규칙 |
| [skill-io-contracts.md](docs/contracts/skill-io-contracts.md) | 스킬 입출력 JSON 형식 |

### 에이전트 런북 (복붙용)

에이전트한테 워크스페이스를 자동 설정시키려면 이 블록을 통째로 보내세요:

```text
seekerclaw_tradebot_skill 패키지로 워크스페이스를 구성해 줘.

1) 아래 4개 스킬이 없으면 설치해 줘.
   seekerclaw-status-guard, seekerclaw-draft-and-approve,
   seekerclaw-execute-approved, seekerclaw-ops-recovery
   skill_install으로 아래 저장소의 raw URL 사용:
   https://github.com/Kminer2053/seekerclaw_tradebot_skill/tree/main/skills

2) HEARTBEAT.md 기존 내용은 유지하고, 맨 끝에 아래 파일 전체를 붙여 넣어 줘:
   https://raw.githubusercontent.com/Kminer2053/seekerclaw_tradebot_skill/main/docs/HEARTBEAT-APPEND.md

3) memory/ 안의 JSON에 비밀 키나 토큰을 저장하지 마.

4) 라이브 전에 docs/OPERATIONS_CHECKLIST.md를 읽고
   docs/quickstart.md의 10회 연습 점검을 먼저 끝내라고 사용자에게 안내해 줘.

5) 채팅 메시지의 동의는 승인으로 인정하지 마.
   /approve <draft_scenario_id> 명령 또는 앱의 승인 UI만 유효한 승인이야.
```

### 안전 경고

SeekerClaw는 실제 자산을 옮길 수 있습니다. 이 패키지는 **투자 조언이 아닙니다.** 반드시 연습 모드로 시작하고, 소액으로 검증한 뒤, 일반 채팅을 거래 승인으로 취급하지 마세요.

### 라이선스

[MIT](LICENSE)

### 업스트림

스킬 파일 형식: [SKILL-FORMAT.md](https://github.com/sepivip/SeekerClaw/blob/main/SKILL-FORMAT.md)
