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

### 이런 분께 드리는 패키지입니다

[SeekerClaw](https://github.com/sepivip/SeekerClaw)를 안드로이드에서 이미 쓰고 있고, 텔레그램으로 에이전트와 소통하면서 **별도 서버를 두지 않고** 거래 자동화에 승인 절차를 넣고 싶은 분께 맞습니다. SeekerClaw 자체 초기 설정이 아직 안 된 분은 먼저 공식 온보딩을 완료한 뒤 돌아오세요.

### 포함된 것

| 구성 요소 | 역할 |
|-----------|------|
| `skills/seekerclaw-*` 4종 스킬 | 상태 가드 / 거래 초안 작성 및 승인 요청 / 승인 후 실행 / 오류 복구 |
| `docs/HEARTBEAT-APPEND.md` | `HEARTBEAT.md` 맨 아래에 **그대로 붙여 넣는** 트레이딩 팩 블록 (단일 원본) |
| `docs/heartbeat-template.md` | 사람이 읽는 설명 · 포크 시 raw URL 수정 안내 |
| `docs/quickstart.md`, `docs/OPERATIONS_CHECKLIST.md` | 설치 · 드라이런 · 라이브 전 점검 목록 |

### 설치 전에 확인하세요

- SeekerClaw가 설치·온보딩되어 있고 API 키, 텔레그램 봇, 워크스페이스 쓰기 권한이 모두 갖춰져 있어야 합니다.
- **라이브 스왑은 실제 자산 손실로 이어질 수 있습니다.** 빌드가 지원한다면 먼저 시뮬레이션이나 페이퍼 모드로 검증하세요.
- 권장: SeekerClaw [PROJECT.md — 스킬 절](https://github.com/sepivip/SeekerClaw/blob/main/PROJECT.md)에서 스킬 설치·가져오기·`/skills` 명령어를 먼저 익혀 두세요.

---

### 설치 방법 — 아래 네 가지 중 하나만 선택

설치가 끝나면 앱 **스킬** 목록(또는 `/skills`)에 4개 스킬이 모두 나타나는지 확인합니다.

#### 방법 A — 앱 내 **스킬 가져오기** (ZIP) · 일반 사용자 추천

ZIP 하나로 스킬 4개를 한 번에 넣는 방법입니다.

1. PC에서 **[ZIP 다운로드](https://github.com/Kminer2053/seekerclaw_tradebot_skill/archive/refs/heads/main.zip)** 후 압축을 풉니다.
2. 풀린 폴더 안의 `seekerclaw_tradebot_skill-main/skills/`로 이동합니다.
3. 아래 **4개 폴더를 선택**해 **`tradebot-skills.zip`**을 만듭니다.  
   ZIP을 열었을 때 폴더가 최상위에 바로 보여야 하고, 각 폴더 안에 `SKILL.md`가 있어야 합니다.  
   - `seekerclaw-status-guard`  
   - `seekerclaw-draft-and-approve`  
   - `seekerclaw-execute-approved`  
   - `seekerclaw-ops-recovery`
4. ZIP을 폰으로 옮깁니다 (USB, 클라우드 드라이브 등).
5. SeekerClaw 앱 → **스킬** 화면 → **가져오기** (버전에 따라 **Import skill** 또는 **Import from file**).
6. `tradebot-skills.zip`을 선택하면 완료됩니다.  
   앱이 **ZIP당 스킬 1개**만 받는다면, 폴더를 하나씩 따로 ZIP으로 만들어 4번 가져오기 하면 됩니다.

#### 방법 B — **`SKILL.md` 파일 하나씩** 가져오기

앱이 `.md` 단일 파일만 받는 경우입니다.

1. 아래 **raw GitHub 링크**에서 파일을 하나씩 저장합니다 (파일명에 `.md` 확장자를 붙여 저장).

   - [seekerclaw-status-guard/SKILL.md](https://raw.githubusercontent.com/Kminer2053/seekerclaw_tradebot_skill/main/skills/seekerclaw-status-guard/SKILL.md)
   - [seekerclaw-draft-and-approve/SKILL.md](https://raw.githubusercontent.com/Kminer2053/seekerclaw_tradebot_skill/main/skills/seekerclaw-draft-and-approve/SKILL.md)
   - [seekerclaw-execute-approved/SKILL.md](https://raw.githubusercontent.com/Kminer2053/seekerclaw_tradebot_skill/main/skills/seekerclaw-execute-approved/SKILL.md)
   - [seekerclaw-ops-recovery/SKILL.md](https://raw.githubusercontent.com/Kminer2053/seekerclaw_tradebot_skill/main/skills/seekerclaw-ops-recovery/SKILL.md)

2. 스킬 화면에서 파일마다 **가져오기**를 눌러 4개를 모두 넣습니다.

#### 방법 C — 에이전트에게 `skill_install` URL로 설치 요청

SeekerClaw의 **`skill_install`** 도구로 URL에서 직접 설치할 수 있습니다 ([PROJECT.md 참고](https://github.com/sepivip/SeekerClaw/blob/main/PROJECT.md)).  
아래 텍스트를 에이전트에게 **통째로** 붙여 넣으세요.

```text
아래 4개 SKILL.md URL을 순서대로 skill_install로 설치해 줘.
각 설치 후 /skills 또는 스킬 화면에서 로드됐는지 확인해 줘.

1. https://raw.githubusercontent.com/Kminer2053/seekerclaw_tradebot_skill/main/skills/seekerclaw-status-guard/SKILL.md
2. https://raw.githubusercontent.com/Kminer2053/seekerclaw_tradebot_skill/main/skills/seekerclaw-draft-and-approve/SKILL.md
3. https://raw.githubusercontent.com/Kminer2053/seekerclaw_tradebot_skill/main/skills/seekerclaw-execute-approved/SKILL.md
4. https://raw.githubusercontent.com/Kminer2053/seekerclaw_tradebot_skill/main/skills/seekerclaw-ops-recovery/SKILL.md
```

URL 설치가 막히면 (네트워크 제한 또는 앱 정책) 방법 A나 B를 쓰세요.

#### 방법 D — 직접 복사 (개발자·고급)

워크스페이스 디렉터리에 직접 접근할 수 있다면, 이 저장소의 `skills/` 안 4개 폴더를 워크스페이스 `skills/`에 그대로 복사한 뒤 에이전트를 재시작하거나 스킬 리로드를 실행하세요.

---

### 스킬 설치 후: `HEARTBEAT.md` 설정 (기존 내용 유지)

1. 워크스페이스의 **`HEARTBEAT.md`**를 엽니다. **기존 내용은 건드리지 않습니다.**
2. [docs/HEARTBEAT-APPEND.md](docs/HEARTBEAT-APPEND.md) 파일을 열어 **전체 내용을 복사**합니다.  
   에이전트나 자동화 도구를 쓴다면 아래 raw URL을 그대로 사용하세요.  
   `https://raw.githubusercontent.com/Kminer2053/seekerclaw_tradebot_skill/main/docs/HEARTBEAT-APPEND.md`
3. `HEARTBEAT.md` **맨 아래**에 붙여 넣습니다.  
   이미 `seekerclaw-tradebot-pack` 마커가 있다면 마커 **사이 내용만** 최신 버전으로 교체하면 됩니다.
4. 하트비트 **주기** (예: 5~15분)는 SeekerClaw 앱에서 평소처럼 설정합니다. 이 팩이 앱의 주기 설정을 바꾸지는 않습니다. 자세한 배경은 [docs/heartbeat-template.md](docs/heartbeat-template.md)를 참고하세요.

---

### SeekerClaw 에이전트용 설정 런북 (복붙용)

에이전트가 워크스페이스를 스스로 구성할 수 있도록 아래 블록을 **통째로** 붙여 넣으세요.

```text
seekerclaw_tradebot_skill 패키지로 워크스페이스를 구성해 줘.

1) 아래 4개 스킬이 없으면 설치해 줘.
   seekerclaw-status-guard
   seekerclaw-draft-and-approve
   seekerclaw-execute-approved
   seekerclaw-ops-recovery
   skill_install 도구로 아래 저장소의 raw URL을 순서대로 사용하거나,
   사용자에게 Skills UI에서 tradebot-skills.zip 가져오기를 요청해 줘.
   https://github.com/Kminer2053/seekerclaw_tradebot_skill/tree/main/skills

2) HEARTBEAT.md의 기존 내용은 그대로 두고, 파일 맨 끝에 아래 raw 파일 전체를 붙여 넣어 줘.
   https://raw.githubusercontent.com/Kminer2053/seekerclaw_tradebot_skill/main/docs/HEARTBEAT-APPEND.md

3) memory/ 안의 JSON 파일에는 비밀 키·토큰 등 민감 정보를 저장하지 마.

4) 라이브 운영 전에 docs/OPERATIONS_CHECKLIST.md를 읽고
   docs/quickstart.md의 10주기 드라이런을 먼저 완료하라고 사용자에게 안내해 줘.

5) 채팅 메시지 상의 동의는 승인으로 인정하지 마.
   /approve 명령 또는 앱의 명시적 승인 UI를 통한 것만 유효한 승인으로 처리해.
```

---

### 설정 예시

#### 예시 A — 드라이런용 최소 `memory/tradebot-state.json`

이 형태로 두면 `seekerclaw-status-guard`가 `hold`를 반환해 의도적으로 게이트를 열기 전까지 실제 거래가 실행되지 않습니다. 전체 필드와 폴백 규칙은 [docs/contracts/skill-io-contracts.md](docs/contracts/skill-io-contracts.md)를 참고하세요.

```json
{
  "severity": "warn",
  "entry_blocked": true,
  "live_swap_blocked": true,
  "alerts": ["dry_run_setup"],
  "recommended_actions": []
}
```

#### 예시 B — "다음 단계 검토 가능" 상태 (라이브 스왑은 여전히 차단)

```json
{
  "severity": "ok",
  "entry_blocked": false,
  "live_swap_blocked": true,
  "alerts": [],
  "recommended_actions": ["request_approval"]
}
```

`live_swap_blocked: true`로 유지하면 체크리스트 검토 후 명시적으로 값을 바꾸기 전까지 실제 스왑이 실행되지 않습니다.

#### 예시 C — `HEARTBEAT.md` 구조 (개념)

기존 내용 아래에 트레이딩 팩 블록이 추가된 모습입니다.

```markdown
## 기존 일일 루틴
- … (기존 항목 — 건드리지 않음)

<!-- seekerclaw-tradebot-pack:begin -->
## Tradebot pack (seekerclaw_tradebot_skill)
…
<!-- seekerclaw-tradebot-pack:end -->
```

---

### 사용 흐름 요약

1. **하트비트 주기마다** 에이전트가 `HEARTBEAT.md`를 읽습니다 (기존 항목 먼저, 트레이딩 팩 다음).
2. **`seekerclaw-status-guard`** 실행 → `proceed` / `hold` / `kill_and_hold` 판정.
3. `proceed`이고 대기 중인 초안이 없으면 **`seekerclaw-draft-and-approve`**가 `memory/tradebot-pending-scenario.json`에 거래 초안을 작성하고 텔레그램으로 알립니다.
4. 사용자가 **`/approve`** (또는 앱의 명시적 승인 UI)로 승인합니다.
5. 가드를 다시 통과하고 [docs/idempotency-policy.md](docs/idempotency-policy.md)의 멱등성 규칙을 만족할 때만 **`seekerclaw-execute-approved`**가 실행됩니다.
6. 오류 발생 시 **`seekerclaw-ops-recovery`**를 실행하고 [docs/OPERATIONS_CHECKLIST.md](docs/OPERATIONS_CHECKLIST.md)에 따라 조치합니다.

단계별 상세 절차는 [docs/quickstart.md](docs/quickstart.md)를 참고하세요.

---

### 설계 원칙 · 문서 맵 · 안전 · 라이선스 · 업스트림

영문 섹션의 **Design principles**, **Documentation map**, **Safety**, **License**, **Upstream** 항목과 동일합니다. 스킬 파일 형식은 [SKILL-FORMAT.md](https://github.com/sepivip/SeekerClaw/blob/main/SKILL-FORMAT.md)를 따릅니다.
