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
only, gitignored). Install Pocock's skills (`setup-matt-pocock-skills`) when we
reach Step 1 (Idea → `grill-with-docs`). Don't install upfront.

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
**Last updated:** 2026-06-02 (bootstrap session)

**Accomplished this session:**
- Bootstrapped repo from template; read all collateral (4 emails, 2 pptx decks,
  Pocock skills reference).
- Distilled the user's setup, the Emporia gap, and the viability question into
  this CLAUDE.md and `docs/requirements.md`.
- Confirmed with Vince: app (not agent); stack TBD (web > Android > iOS);
  Pocock skills installed *later*, not now; `grill me` is deprecated, use
  `grill-with-docs`.

**Open / next steps:**
- Prep for Friday meeting with the end user: list of viability questions to bring.
- Install Pocock skills and run `grill-with-docs` against this project (Step 1
  of his process).

**Decisions made:**
- Project type: **app** (no agent persona, no `prompts/` directory).
- Project name: `solar-sync` (kept).
- Python tooling: **deferred** — don't scaffold until stack is chosen.
- UI automation is a legitimate fallback, not just a last resort.
