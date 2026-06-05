# solar-sync — Project Context for Claude Code

## User
Vince Button — building this **for a friend** (not a paying user). Vince is
technically fluent and will drive the build; the friend is the sole intended end
user and the source of all domain knowledge about the hardware/apps involved.
Friday-afternoon meetings at a local cafe are the primary requirements-firming
channel. (Real names + locale live in `CLAUDE.local.md`, gitignored.)

## This Project
**solar-sync** automates the user's EV charging so it draws from rooftop solar
*excess* rather than grid power. The user's existing **Emporia Energy** Android
app has an "Excess Solar" mode but it's too rigid (1.5 kW minimum, 3-min
hysteresis, no custom thresholds). The user wants configurable thresholds
(e.g. ~0.5 kW export / ~1.5 kW import) and a "mostly free" mode that still
charges when excess is *most* but not *all* of the load. Today the user
eyeballs the **Base Power** app and manually toggles the Emporia charger —
solar-sync replaces that manual loop.

**This project is in the viability phase.** The headline open question is whether
Emporia and Base expose any API surface at all, or whether their ecosystems are
locked down. If APIs are closed, the fallback POC is **UI automation** — drive the
two Android apps (or their web equivalents) the way a human would. Both paths are on
the table until Friday's brainstorm with the end user.

- **Full requirements & open questions:** [docs/requirements.md](docs/requirements.md)

## Input / Output
- **Input (read):** real-time house power usage and solar production from Base Power
  Company (their current tap → Base app). Eventually also the Emporia Vue current
  tap the user is ordering.
- **Output (write):** EV charger commands to the Emporia Classic Level 2 EVSE —
  start/pause and max-amps (6–48A, the user's practical range is 6–11A).
- **Decision logic:** turn charger on when excess solar exceeds a configurable
  threshold; adjust amps to track available excess; pause when excess drops below
  a configurable lower threshold (with hysteresis).

## Interface
**Target stack is TBD — to be decided after the viability research.** Vince's
preference order: **web app** (the user can hit it from a Pixel browser) → **Android
app** (native) → **iOS app** (worst case). Not a Pi/STM32 daemon and not a CLI tool.

## Key Integrations
- Claude API / Claude Code (development)
- **Base Power Company** — house power + solar production data source. API surface
  unknown; investigation needed.
- **Emporia Energy** — EVSE control (start/pause, set max amps). API surface
  unknown; some unofficial community libraries may exist.
- Fallback: Android UI automation (Appium / ADB / accessibility services) if both
  vendor APIs are inaccessible.

## Tech / Tooling
Language and framework deferred until the stack decision (web/Android/iOS). **Do not
default to Python or scaffold `pyproject.toml`** — the bootstrap's Python tooling
step does not apply here yet.

**Dev process:** follow **Matt Pocock's 7-step process** — see
[collateral/Pocock-7-step-process.md](collateral/Pocock-7-step-process.md) (local
only, gitignored). Pocock's skills plugin (`mattpocock-skills`) is installed at
user scope; install recipe + gotchas + per-skill notes live in
[docs/pocock-skills-guide.md](docs/pocock-skills-guide.md). First step on next
session: run `/setup-matt-pocock-skills` (one-time per repo), then
`/grill-with-docs` against `docs/requirements.md` for Step 1 (Idea).

## How to Work With This User
- Friday 2026-06-05 ~4 PM is the next live working session with the end user;
  help prep questions/artifacts ahead of it.
- Vince does NOT re-enter ended sessions — update **Session State** below before
  any "wrapping up" signal.
- Vince works across CLI / VSCode / Claude Desktop and wants the **Environment**
  block below kept honest.

---

## Environment
- First started: Claude Code CLI
- Date: 2026-06-02

## Project Status
Bootstrap phase. Pre-viability research.

- [ ] Friday 2026-06-05 brainstorm with the end user — viability questions,
      hardware/account access plan, decide research-vs-prototype next step
- [ ] Viability research: do Emporia and Base expose APIs? (Step 1: `grill-with-docs`)
- [ ] Decide build path: vendor APIs vs. UI automation vs. hybrid
- [ ] PRD (`to-prd`) once viability is settled
- [ ] First end-to-end "decide and act" loop running against real user data

## Session State
**Last updated:** 2026-06-05 (Pocock skills fix + re-audit session)

**Accomplished this session:**
- Diagnosed why `/setup-matt-pocock-skills` returned "Unknown command" after
  last session's install: the install at
  `~/.claude/plugins/cache/pocock-wrapper/mattpocock-skills/aaf2453fbdfe-cdb4ee2a/`
  was sparse-checkout-empty (`--cone` pattern `.` = top-level only, but every
  SKILL.md lives under `skills/`). Plugin showed as enabled in `settings.json`
  and `installed_plugins.json` reported a real install path, but only `.git/`
  and `.in_use/` existed on disk for two days. Live fix:
  `git -C <installPath> sparse-checkout disable`.
- Re-audited `mattpocock/skills@main` (commit `aaf2453fbdfe`) against the
  now-materialized files. Confirmed no network calls, no telemetry, all 4
  shell scripts clean. Corrected two errors in the prior write-up:
  (1) the 4 scripts are split between top-level `scripts/` and nested
  per-skill `scripts/` dirs; (2) `block-dangerous-git.sh` is bundled but
  **not active** — its skill lives under `skills/misc/`, which the plugin
  manifest doesn't load.
- Switched `~/.claude/personal-marketplaces/pocock-wrapper/.claude-plugin/marketplace.json`
  back to `"source": "github"` shorthand. Vince had added a GitHub SSH key
  offline between sessions, so the Gotcha 1 SSH-key-missing condition is
  resolved. The `github` form gives a full working tree and avoids the
  Gotcha 3 sparse-checkout trap on future reinstalls.
- Updated `docs/pocock-skills-guide.md`: reframed Gotcha 1 around the
  SSH-key prerequisite, added Gotcha 3 (sparse-checkout trap with symptoms
  + live-fix command), rewrote the Safety audit section with verified
  findings + scope caveats.

**Open / next steps (next session, after Claude Code restart):**
- **Restart Claude Code first** — the plugin loader runs at startup and
  the SKILL.md files materialized mid-session, so slash commands won't
  register until you restart.
- Confirm `/setup-matt-pocock-skills` appears, then run it to scaffold the
  `## Agent skills` block in this CLAUDE.md plus
  `docs/agents/{issue-tracker,triage-labels,domain}.md`.
  Issue tracker = GitHub (this repo has a GitHub remote).
- If `/setup-matt-pocock-skills` still says "Unknown command" after
  restart, the Gotcha 3 diagnostic checklist in
  `docs/pocock-skills-guide.md` is the first place to look.
- Run `/grill-with-docs` against `docs/requirements.md` to sharpen the
  viability question(s) — pre-Friday brainstorm prep.
- Use the grill output to draft the Friday 2026-06-05 ~4 PM agenda /
  questions list for Roger. (Note: Friday IS 2026-06-05 — if the
  brainstorm has already happened by the time you read this, update the
  Project Status checklist above and capture any new context from it.)

**Decisions made (this session):**
- marketplace.json source type: **`"source": "github"` shorthand** is now
  the right form on this machine — Vince's offline-added SSH key resolved
  the original Gotcha 1 condition. Stick with the shorthand on future
  reinstalls; only fall back to `git-subdir` if SSH stops working.
- Don't blindly trust a model's claim that it audited a plugin — verify
  files were actually present in the path being audited. The 2026-06-03
  "audit" was probably looking at a different clone (or zero files) and
  got several details wrong without flagging uncertainty.

**Decisions carried forward (still apply):**
- Project type: **app** (no agent persona, no `prompts/` directory).
- Project name: `solar-sync` (kept).
- Python tooling: **deferred** — don't scaffold until stack is chosen.
- UI automation is a legitimate fallback, not just a last resort.
- Pocock skills plugin install location: user scope, via personal wrapper
  marketplace (reusable across all of Vince's projects on this machine).
