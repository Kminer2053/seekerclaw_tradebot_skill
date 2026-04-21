<div align="center">

# SeekerClaw Tradebot Skills

**Approval-first trading orchestration as SeekerClaw skills.**  
Runs on-device with SeekerClaw. No separate trading backend; state stays in workspace files.

[![SeekerClaw](https://img.shields.io/badge/SeekerClaw-upstream-111827?style=flat-square&logo=github)](https://github.com/sepivip/SeekerClaw)
[![Skill format](https://img.shields.io/badge/Skill%20format-SKILL--FORMAT.md-0F766E?style=flat-square)](https://github.com/sepivip/SeekerClaw/blob/main/SKILL-FORMAT.md)
[![License](https://img.shields.io/badge/License-MIT-blue?style=flat-square)](LICENSE)

**Repo:** [github.com/Kminer2053/seekerclaw_tradebot_skill](https://github.com/Kminer2053/seekerclaw_tradebot_skill)

[English](#english) · [한국어](#한국어-korean)

</div>

---

## English

### Who this is for

You already run [SeekerClaw](https://github.com/sepivip/SeekerClaw) on Android, use Telegram with your agent, and want a **documented, approval-gated** loop for trading-related steps **without** standing up a separate backend. If that is not you, start with SeekerClaw’s own setup first.

### What you get

| Deliverable | Role |
|-------------|------|
| Four skills under `skills/seekerclaw-*` | Guard, draft + approval request, execute after approval, recovery |
| `docs/HEARTBEAT-APPEND.md` | **Paste this entire file** at the end of `HEARTBEAT.md` (single source for the pack block) |
| `docs/heartbeat-template.md` | Human guide + fork/mirror URL notes for heartbeat |
| `docs/quickstart.md`, `docs/OPERATIONS_CHECKLIST.md` | Operator checklist and dry-run steps |

### Prerequisites (before install)

- SeekerClaw installed and onboarded (model API key, Telegram bot, workspace writable).
- You understand that **live swaps can lose funds**; use simulation or paper mode first if your build supports it.
- Optional but recommended: read [SeekerClaw `PROJECT.md` — Skills](https://github.com/sepivip/SeekerClaw/blob/main/PROJECT.md) (skill install, import, `/skills`).

---

### Installation

Pick **one** path. After any path, confirm all four skills appear under the app **Skills** list (or via `/skills` if your build exposes it).

#### Method A — In-app **Import skill** (ZIP, recommended for most users)

SeekerClaw can import skills from a **ZIP** (and from single `.md` files); see upstream notes on import and size limits.

1. On a PC, open **[Download ZIP](https://github.com/Kminer2053/seekerclaw_tradebot_skill/archive/refs/heads/main.zip)** of this repo and extract it.
2. Enter the extracted folder `seekerclaw_tradebot_skill-main/skills/`.
3. Select **all four** folders (`seekerclaw-status-guard`, `seekerclaw-draft-and-approve`, `seekerclaw-execute-approved`, `seekerclaw-ops-recovery`) and create **`tradebot-skills.zip`** containing those folders at the **top level** (each folder must still contain `SKILL.md`).
4. Copy `tradebot-skills.zip` to your phone (USB, cloud, etc.).
5. In SeekerClaw, open the **Skills** screen → **Import** (wording may be **Import skill** / **Import from file** depending on version).
6. Choose `tradebot-skills.zip` and complete the import. If the app imports **one skill per ZIP**, repeat with **one folder zipped at a time** (each ZIP contains a single `seekerclaw-*/SKILL.md` tree).

#### Method B — In-app import **one `SKILL.md` at a time**

If your build only accepts a single markdown file per import:

1. On the phone or PC, download each file from **raw GitHub** (save as `.md` if needed):

   - [seekerclaw-status-guard/SKILL.md](https://raw.githubusercontent.com/Kminer2053/seekerclaw_tradebot_skill/main/skills/seekerclaw-status-guard/SKILL.md)
   - [seekerclaw-draft-and-approve/SKILL.md](https://raw.githubusercontent.com/Kminer2053/seekerclaw_tradebot_skill/main/skills/seekerclaw-draft-and-approve/SKILL.md)
   - [seekerclaw-execute-approved/SKILL.md](https://raw.githubusercontent.com/Kminer2053/seekerclaw_tradebot_skill/main/skills/seekerclaw-execute-approved/SKILL.md)
   - [seekerclaw-ops-recovery/SKILL.md](https://raw.githubusercontent.com/Kminer2053/seekerclaw_tradebot_skill/main/skills/seekerclaw-ops-recovery/SKILL.md)

2. Import **each** file from the Skills screen. The app should place it under workspace `skills/` with the correct skill name from YAML frontmatter.

#### Method C — Agent-assisted install (`skill_install` URL)

SeekerClaw exposes a **`skill_install`** tool that can install from a **URL** (see [PROJECT.md](https://github.com/sepivip/SeekerClaw/blob/main/PROJECT.md)). Tell your agent explicitly, in one message:

```text
Install these four skills from URL, one after another, and confirm each with /skills or the Skills UI:
1. https://raw.githubusercontent.com/Kminer2053/seekerclaw_tradebot_skill/main/skills/seekerclaw-status-guard/SKILL.md
2. https://raw.githubusercontent.com/Kminer2053/seekerclaw_tradebot_skill/main/skills/seekerclaw-draft-and-approve/SKILL.md
3. https://raw.githubusercontent.com/Kminer2053/seekerclaw_tradebot_skill/main/skills/seekerclaw-execute-approved/SKILL.md
4. https://raw.githubusercontent.com/Kminer2053/seekerclaw_tradebot_skill/main/skills/seekerclaw-ops-recovery/SKILL.md
```

If a step fails, use Method A or B instead (network or tool policy may block raw URLs).

#### Method D — Manual copy (advanced)

If you have direct access to the OpenClaw/SeekerClaw **workspace** on disk (developer workflows):

1. Copy the four directories from this repo’s `skills/` into the workspace `skills/` directory.
2. Restart the agent or trigger a skill reload per SeekerClaw docs.

---

### After installing skills: `HEARTBEAT.md` (non-destructive)

1. Open workspace **`HEARTBEAT.md`**. **Do not delete** existing sections.
2. Open [docs/HEARTBEAT-APPEND.md](docs/HEARTBEAT-APPEND.md) (single source of truth for the snippet).
3. Copy the **entire file** and paste at the **very end** of `HEARTBEAT.md`. If `seekerclaw-tradebot-pack` markers already exist, replace **only** the content between them once.
4. Raw URL for agents or `curl`:  
   `https://raw.githubusercontent.com/Kminer2053/seekerclaw_tradebot_skill/main/docs/HEARTBEAT-APPEND.md`
5. In the app, set the **heartbeat interval** (e.g. 5–15 minutes) the way you already do for SeekerClaw. This pack does not replace that setting.

Context for the snippet: [docs/heartbeat-template.md](docs/heartbeat-template.md).

---

### For SeekerClaw agent (copy-paste runbook)

Use this as a **single instruction block** when you want the agent to configure the workspace without guessing.

```text
You are configuring the workspace for the seekerclaw_tradebot_skill pack.

1) Install all four skills if missing:
   - seekerclaw-status-guard
   - seekerclaw-draft-and-approve
   - seekerclaw-execute-approved
   - seekerclaw-ops-recovery
   Use skill_install with the raw SKILL.md URLs from:
   https://github.com/Kminer2053/seekerclaw_tradebot_skill/tree/main/skills
   Or ask the user to import tradebot-skills.zip from the Skills UI.

2) Open HEARTBEAT.md. Do NOT remove existing content. Append at the end the full contents of:
   https://raw.githubusercontent.com/Kminer2053/seekerclaw_tradebot_skill/main/docs/HEARTBEAT-APPEND.md

3) Ensure memory/ can hold JSON files. Do not put secrets in memory/*.json.

4) Tell the user to read docs/OPERATIONS_CHECKLIST.md and run a 10-cycle dry run from docs/quickstart.md before live size.

5) Never treat chat text as approval; only /approve or the app's explicit approval UI counts.
```

---

### Configuration examples

#### Example A — Minimal `memory/tradebot-state.json` (dry-run friendly)

Use this shape so `seekerclaw-status-guard` can return `hold` until you intentionally relax gates (see [skill-io-contracts.md](docs/contracts/skill-io-contracts.md) for full schema).

```json
{
  "severity": "warn",
  "entry_blocked": true,
  "live_swap_blocked": true,
  "alerts": ["dry_run_setup"],
  "recommended_actions": []
}
```

#### Example B — “OK to consider next steps” (still requires explicit approval for live)

```json
{
  "severity": "ok",
  "entry_blocked": false,
  "live_swap_blocked": true,
  "alerts": [],
  "recommended_actions": ["request_approval"]
}
```

`live_swap_blocked: true` keeps live swaps off until you deliberately flip it after checklist review.

#### Example C — Heartbeat placement (conceptual)

Your file should look like:

```markdown
## My existing daily routine
- … (unchanged)

<!-- seekerclaw-tradebot-pack:begin -->
## Tradebot pack (seekerclaw_tradebot_skill)
…
<!-- seekerclaw-tradebot-pack:end -->
```

---

### Usage (typical flow)

1. **Heartbeat fires** → agent reads `HEARTBEAT.md` (existing tasks first, then Tradebot pack).
2. **`seekerclaw-status-guard`** → `proceed` / `hold` / `kill_and_hold` from `memory/tradebot-state.json` (or safe fallback if missing).
3. If `proceed` and no pending scenario → **`seekerclaw-draft-and-approve`** writes `memory/tradebot-pending-scenario.json` and notifies you (e.g. Telegram).
4. You send **`/approve`** (or use the app’s explicit approval UI) when you accept the draft.
5. **`seekerclaw-execute-approved`** runs only after guard passes again and idempotency rules in [docs/idempotency-policy.md](docs/idempotency-policy.md) allow it.
6. On errors → **`seekerclaw-ops-recovery`** and update state; follow [docs/OPERATIONS_CHECKLIST.md](docs/OPERATIONS_CHECKLIST.md).

Detailed operator steps: [docs/quickstart.md](docs/quickstart.md).

---

### Design principles

| Principle | What it means |
|-----------|----------------|
| Skill-only | No custom server. SeekerClaw built-ins (wallet, Jupiter, Telegram) only. |
| Explicit approval | Live swaps only after `/approve` or an equivalent explicit UI in the app. |
| Guard first | `seekerclaw-status-guard` runs before drafts or execution. |
| Idempotent execution | One stable key per `approval_id` in `memory/tradebot-idempotency.json`. |
| Non-destructive heartbeat | Append one marked section to the bottom of `HEARTBEAT.md`. |

### Repository layout

```
skills/seekerclaw-*/SKILL.md
docs/quickstart.md
docs/heartbeat-template.md
docs/OPERATIONS_CHECKLIST.md
docs/idempotency-policy.md
docs/contracts/skill-io-contracts.md
```

### Documentation map

| Doc | Purpose |
|-----|---------|
| [docs/quickstart.md](docs/quickstart.md) | Install, heartbeat, dry-run |
| [docs/OPERATIONS_CHECKLIST.md](docs/OPERATIONS_CHECKLIST.md) | Pre-flight, per-cycle, incidents, resume |
| [docs/HEARTBEAT-APPEND.md](docs/HEARTBEAT-APPEND.md) | **Append-only** text for `HEARTBEAT.md` (use this file when copying) |
| [docs/heartbeat-template.md](docs/heartbeat-template.md) | Human guide + automation notes for heartbeat |
| [docs/idempotency-policy.md](docs/idempotency-policy.md) | Idempotency file rules |
| [docs/contracts/skill-io-contracts.md](docs/contracts/skill-io-contracts.md) | JSON response shapes |

### Safety

SeekerClaw can move real funds. This pack is **not** financial advice. Start small, verify behavior in simulation or paper mode if your build supports it, and never treat chat text as approval unless it goes through the app’s explicit approval path.

### License

[MIT](LICENSE)

### Upstream

Skill format: [SKILL-FORMAT.md](https://github.com/sepivip/SeekerClaw/blob/main/SKILL-FORMAT.md).

---

## 한국어 (Korean)

### 이런 분께 권합니다

이미 [SeekerClaw](https://github.com/sepivip/SeekerClaw)를 안드로이드에서 돌리고, 텔레그램으로 에이전트와 대화하며, **별도 서버 없이** 승인 절차를 밟는 트레이딩 관련 자동화를 쓰고 싶은 경우입니다. SeekerClaw 자체 설정이 아직이면 먼저 공식 온보딩을 마치세요.

### 무엇이 포함되나요

| 구성요소 | 역할 |
|----------|------|
| `skills/seekerclaw-*` 네 스킬 | 가드, 초안·승인 요청, 승인 후 실행, 복구 |
| `docs/HEARTBEAT-APPEND.md` | `HEARTBEAT.md` **맨 아래에 통째로 붙이는** 트레이딩 팩 본문(단일 원본) |
| `docs/heartbeat-template.md` | 사람용 안내·포크 시 raw URL 교체 방법 |
| `docs/quickstart.md`, `docs/OPERATIONS_CHECKLIST.md` | 설치·드라이런·라이브 점검 |

### 설치 전 확인

- SeekerClaw 설치 및 기본 설정 완료(API 키, 텔레그램 봇, 워크스페이스 쓰기 가능).
- **라이브 스왑은 손실 가능**함을 이해했는지. 가능하면 시뮬레이션·페이퍼 모드로 검증.
- 권장: [SeekerClaw `PROJECT.md` 스킬 절](https://github.com/sepivip/SeekerClaw/blob/main/PROJECT.md)(설치, 가져오기, `/skills`).

---

### 설치 방법 (하나만 선택)

설치 후 앱 **스킬** 목록(또는 빌드에 따라 `/skills`)에 네 스킬이 모두 보이는지 확인합니다.

#### 방법 A — 앱에서 **스킬 가져오기**(ZIP, 일반 사용자 추천)

1. PC에서 저장소 **[ZIP 다운로드](https://github.com/Kminer2053/seekerclaw_tradebot_skill/archive/refs/heads/main.zip)** 후 압축 해제.
2. `seekerclaw_tradebot_skill-main/skills/` 폴더로 들어갑니다.
3. 다음 **네 폴더 전체**를 선택해 **`tradebot-skills.zip`**을 만듭니다. ZIP **안 최상위**에 네 폴더가 있고, 각 폴더에 `SKILL.md`가 있어야 합니다.  
   - `seekerclaw-status-guard`  
   - `seekerclaw-draft-and-approve`  
   - `seekerclaw-execute-approved`  
   - `seekerclaw-ops-recovery`
4. ZIP을 휴대폰으로 옮깁니다.
5. SeekerClaw 앱 → **스킬** 화면 → **가져오기 / Import**(버전에 따라 문구 상이).
6. `tradebot-skills.zip` 선택 후 완료. 앱이 **스킬당 ZIP 하나**만 받는 경우, 폴더를 **하나씩** ZIP으로 만들어 네 번 가져오기 합니다.

#### 방법 B — **`SKILL.md` 파일을 하나씩** 가져오기

1. 아래 **raw** 주소에서 각 파일을 저장합니다(필요 시 `.md`로 저장).

   - [seekerclaw-status-guard/SKILL.md](https://raw.githubusercontent.com/Kminer2053/seekerclaw_tradebot_skill/main/skills/seekerclaw-status-guard/SKILL.md)
   - [seekerclaw-draft-and-approve/SKILL.md](https://raw.githubusercontent.com/Kminer2053/seekerclaw_tradebot_skill/main/skills/seekerclaw-draft-and-approve/SKILL.md)
   - [seekerclaw-execute-approved/SKILL.md](https://raw.githubusercontent.com/Kminer2053/seekerclaw_tradebot_skill/main/skills/seekerclaw-execute-approved/SKILL.md)
   - [seekerclaw-ops-recovery/SKILL.md](https://raw.githubusercontent.com/Kminer2053/seekerclaw_tradebot_skill/main/skills/seekerclaw-ops-recovery/SKILL.md)

2. 스킬 화면에서 파일마다 **가져오기**를 반복합니다.

#### 방법 C — 에이전트에게 **`skill_install` URL**로 설치 시키기

SeekerClaw는 URL로 스킬을 설치하는 **`skill_install`** 도구를 제공합니다([PROJECT.md](https://github.com/sepivip/SeekerClaw/blob/main/PROJECT.md)). 에이전트에게 아래를 **한 번에** 붙여 넣습니다.

```text
아래 네 개 SKILL.md URL을 순서대로 skill_install로 설치하고, /skills 또는 스킬 화면으로 로드 여부를 확인해 줘.
1. https://raw.githubusercontent.com/Kminer2053/seekerclaw_tradebot_skill/main/skills/seekerclaw-status-guard/SKILL.md
2. https://raw.githubusercontent.com/Kminer2053/seekerclaw_tradebot_skill/main/skills/seekerclaw-draft-and-approve/SKILL.md
3. https://raw.githubusercontent.com/Kminer2053/seekerclaw_tradebot_skill/main/skills/seekerclaw-execute-approved/SKILL.md
4. https://raw.githubusercontent.com/Kminer2053/seekerclaw_tradebot_skill/main/skills/seekerclaw-ops-recovery/SKILL.md
```

실패하면 방법 A 또는 B를 사용합니다(네트워크·정책에 따라 URL 설치가 막힐 수 있음).

#### 방법 D — 수동 복사(고급)

워크스페이스 `skills/` 디렉터리에 직접 접근 가능하면, 이 저장소의 `skills/` 아래 네 폴더를 그대로 복사한 뒤 에이전트를 재시작하거나 스킬 리로드합니다.

---

### 스킬 설치 후: `HEARTBEAT.md`(비파괴)

1. 워크스페이스 **`HEARTBEAT.md`**를 엽니다. **기존 내용은 삭제하지 않습니다.**
2. [docs/HEARTBEAT-APPEND.md](docs/HEARTBEAT-APPEND.md) 파일 **전체**를 복사합니다(에이전트·자동화는 raw URL 사용).  
   `https://raw.githubusercontent.com/Kminer2053/seekerclaw_tradebot_skill/main/docs/HEARTBEAT-APPEND.md`
3. `HEARTBEAT.md` **맨 아래**에 붙입니다. 이미 `seekerclaw-tradebot-pack` 주석이 있으면 **그 사이만** 최신으로 교체합니다.
4. 하트비트 **주기**(예: 5~15분)는 SeekerClaw 앱에서 기존과 같이 설정합니다. 배경 설명은 [docs/heartbeat-template.md](docs/heartbeat-template.md).

---

### SeekerClaw 에이전트용 실행 스크립트(복사용)

에이전트가 추측하지 않고 그대로 따라 할 수 있게 한 블록입니다.

```text
seekerclaw_tradebot_skill 패키지로 워크스페이스를 구성한다.

1) 네 스킬이 없으면 설치한다:
   seekerclaw-status-guard, seekerclaw-draft-and-approve, seekerclaw-execute-approved, seekerclaw-ops-recovery
   skill_install로 아래 raw URL을 순서대로 사용하거나, 사용자에게 Skills UI에서 tradebot-skills.zip 가져오기를 요청한다.
   https://github.com/Kminer2053/seekerclaw_tradebot_skill/tree/main/skills

2) HEARTBEAT.md는 기존 내용을 유지하고, 파일 끝에 아래 raw 전체를 붙인다:
   https://raw.githubusercontent.com/Kminer2053/seekerclaw_tradebot_skill/main/docs/HEARTBEAT-APPEND.md

3) memory/ 아래 JSON에는 비밀을 넣지 않는다.

4) 사용자에게 docs/OPERATIONS_CHECKLIST.md와 docs/quickstart.md 10주기 드라이런을 라이브 전에 수행하라고 안내한다.

5) 채팅 일반 동의는 승인이 아니다. /approve 또는 앱의 명시적 승인 UI만 인정한다.
```

---

### 설정 예시

#### 예시 A — 드라이런용 `memory/tradebot-state.json` 최소 형태

전체 필드·폴백은 [docs/contracts/skill-io-contracts.md](docs/contracts/skill-io-contracts.md)를 참고합니다.

```json
{
  "severity": "warn",
  "entry_blocked": true,
  "live_swap_blocked": true,
  "alerts": ["dry_run_setup"],
  "recommended_actions": []
}
```

#### 예시 B — 다음 단계 검토 가능(라이브는 여전히 게이트로 끄기)

```json
{
  "severity": "ok",
  "entry_blocked": false,
  "live_swap_blocked": true,
  "alerts": [],
  "recommended_actions": ["request_approval"]
}
```

#### 예시 C — `HEARTBEAT.md` 배치(개념)

```markdown
## 기존 일일 루틴
- … (변경 없음)

<!-- seekerclaw-tradebot-pack:begin -->
## Tradebot pack (seekerclaw_tradebot_skill)
…
<!-- seekerclaw-tradebot-pack:end -->
```

---

### 사용 흐름(요약)

1. 하트비트 주기마다 `HEARTBEAT.md` 실행(위쪽 기존 항목 → 아래 트레이딩 팩).
2. `seekerclaw-status-guard`로 `proceed` / `hold` / `kill_and_hold` 판단.
3. `proceed`이고 대기 초안 없으면 `seekerclaw-draft-and-approve`가 `memory/tradebot-pending-scenario.json` 등을 갱신·알림.
4. 운영자가 **`/approve`**(또는 동등 UI)로 명시적 승인.
5. 가드 재통과 후에만 `seekerclaw-execute-approved`·멱등 규칙 적용([docs/idempotency-policy.md](docs/idempotency-policy.md)).
6. 오류 시 `seekerclaw-ops-recovery` 및 [docs/OPERATIONS_CHECKLIST.md](docs/OPERATIONS_CHECKLIST.md).

자세한 단계는 [docs/quickstart.md](docs/quickstart.md)를 따릅니다.

---

### 설계 원칙·문서 맵·안전·라이선스·업스트림

영문 절의 **Design principles**, **Documentation map**, **Safety**, **License**, **Upstream**과 동일합니다. 스킬 형식은 [SKILL-FORMAT.md](https://github.com/sepivip/SeekerClaw/blob/main/SKILL-FORMAT.md)를 따릅니다.
