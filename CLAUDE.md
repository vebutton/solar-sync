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

## Agent skills

### Issue tracker

GitHub Issues at `vebutton/solar-sync`, via the `gh` CLI. See `docs/agents/issue-tracker.md`.

### Triage labels

Default canonical names: `needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`. `wontfix` already exists on the repo; the other four are created on first `triage` run. See `docs/agents/triage-labels.md`.

### Domain docs

Single-context: one `CONTEXT.md` + `docs/adr/` at the repo root (neither exists yet — `/grill-with-docs` creates them lazily). See `docs/agents/domain.md`.

## Project Status
Bootstrap phase. Viability research begins next.

- [x] Pocock skills installed + verified; per-repo config scaffolded
      (`docs/agents/*.md`, `CONTEXT.md` seeded)
- [x] Friday 2026-06-05 brainstorm with the end user — agenda + meeting
      complete; 8 RQs captured; `CONTEXT.md` and `docs/requirements.md`
      §7 updated with the resolutions
- [ ] **Viability Q1 (Emporia EVSE API)** — desk-research the HA
      community Emporia integration the end user pointed to; verify it
      controls his specific EVSE model
- [ ] **Viability Q4 (Vue local API)** — determine whether Vue
      publishes locally; deliver a yes/no/maybe verdict to the end
      user (his Vue (re-)purchase is gated on this)
- [ ] **Stack decision** — resolve the dumb-device-vs-Linux-stack
      tension (STM32 / Pi + auto-update / Android-on-Pixel / hybrid)
- [ ] Investigate EVSE cold-start-without-WAN fragility (observed twice)
- [ ] Identify a backup phone for any invasive experiments
      (ADB / UI automation / mitmproxy)
- [ ] Decide build path: vendor APIs vs. UI automation vs. hybrid
      (depends on Q1 + Q4 outcomes)
- [ ] PRD (`to-prd`) once viability is settled
- [ ] First end-to-end "decide and act" loop running against real user data

## Session State
**Last updated:** 2026-06-05 — Friday brainstorm captured; viability
research is the next active phase.
**Session log:** [docs/session-history/2026-06-05-brainstorm-meeting.md](docs/session-history/2026-06-05-brainstorm-meeting.md)
(per-session narratives live in `docs/session-history/` — load on demand
for accomplishments + per-session decision context).

**Open / next steps:**
- Desk-research the HA Emporia integration (Q1 lead). Verify it
  controls the end user's specific EVSE model.
- Desk-research Vue local API (Q4). Deliver a verdict to the end user
  so he can decide whether to (re-)buy.
- Resolve the stack tension between the end user's "dumb device"
  preference (STM32) and the Linux/Python needs of the HA path.
  Candidates: Pi + auto-updating OS, Android app on the Pixel,
  STM32 + custom Emporia client, hybrid.
- Investigate the EVSE cold-start-without-WAN fragility (observed
  twice). Does the HA integration handle it?
- Identify a spare phone before any ADB / UI-automation / mitmproxy
  experiment.

**Standing decisions:**
- Project type: **app** (no agent persona, no `prompts/` directory).
- Project name: `solar-sync`.
- Python tooling: **deferred** — don't scaffold until stack is chosen.
- UI automation is a legitimate fallback, not just a last resort.
- Pocock skills plugin: user scope, via personal wrapper marketplace.
- `marketplace.json` source type: `"source": "github"` shorthand.
- Issue tracker: **GitHub Issues** at `vebutton/solar-sync` via `gh` CLI.
- Triage labels: **canonical defaults** (no remapping).
- Domain layout: **single-context** (`CONTEXT.md` + `docs/adr/` at root).
- Brainstorm artifacts (agenda, notes) live in `collateral/`
  (gitignored), not `docs/` — PII redaction per `CLAUDE.local.md`.
- Session State pattern: per-session narratives in
  `docs/session-history/`; this section holds only standing state + the
  latest session-log pointer.
- **"Excess solar"** resolves to **Net excess** (`Production − Usage`);
  Base battery is abstracted as part of "the grid" (RQ1).
- **Algorithm philosophy:** titrate EV amps so net export → 0; enable
  at ≥ 0.1 kW Net excess; respect the amp-ceiling hierarchy (RQ2 / RQ6).
- **UX direction:** a single 0–100% Slider collapses F4 + F5 in
  `docs/requirements.md` (RQ3). Not yet design-locked or ADR-recorded;
  pending the PRD pass.
- **Amp ceiling hierarchy:** 6A floor / 12A auto-mode / 40A override /
  48A Dedicated Breaker (RQ6). All configurable, not hardcoded.
- **AI is off the credential path.** Account-touching code must be
  small enough for the end user to audit via independent AI (Bing
  Chat / Copilot) before running (RQ5).
- **Stop semantics:** rely on the car's SOC cap (Cadillac Lyriq app
  slider) as the durable stop signal — solar-sync's "stop" is just
  "amps → 0 or EVSE pause," no force-pause primitive needed (RQ4).
- **Project narrative leads with summer AC-frustration relief**; kWh
  savings is the secondary framing (RQ7).
- **PII redaction stays in force** (RQ8). `CLAUDE.local.md` identity
  mapping unchanged.
