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
hysteresis, no custom thresholds). The user wants configurable thresholds and
a "mostly free" mode that still charges when excess is *most* but not *all* of
the load.

**Major pivot, 2026-06-28.** The end user built the integration plane himself
between 2026-06-16 and 2026-06-28 while Vince was heads-down on other work.
He dropped Base Power as the data source, surfaced his **Enphase Envoy** local
API instead (already on his LAN), wired in the consumption current taps
Enphase had shipped him originally, and stood up **Home Assistant on a
Raspberry Pi 5** as the control plane. HA can now read `Current net power
consumption` (signed: + import / − export) and start/stop/throttle the
Emporia EVSE via a community integration's charger-entities branch. The
viability questions about Emporia and Base APIs are largely settled — not by
research, but by the end user changing the question. **What's left is the
decision logic** (mode picker + titration loop), not the integration plane.

- **Full requirements & open questions:** [docs/requirements.md](docs/requirements.md)
- **The pivot, in detail:** [docs/session-history/2026-06-28-end-user-pivot-to-ha.md](docs/session-history/2026-06-28-end-user-pivot-to-ha.md)

## Input / Output
- **Input (read):** `Current net power consumption` from the Enphase Envoy
  local API (already on the end user's LAN). Single signed signal: positive
  = importing from grid, negative = exporting. Solar production is also
  available from the same Envoy if needed.
- **Output (write):** EV charger commands to the Emporia Classic Level 2 EVSE
  via the Home Assistant community Emporia integration — start/pause and
  max-amps (6–48A, the user's practical range is 6–11A).
- **Decision logic:** mode-picker on top of a titration loop. Modes range
  from OFF / Solar Only / mixed (configurable import budget) / fixed-rate
  override. Loop: titrate amps so net export → 0 (or → a target import for
  hybrid modes), with hysteresis + per-step dwell time.

## Interface
**HA dashboard on the Pi, served over the LAN.** The end user hits it from
his Pixel browser — same UX shape as the original "web app" preference, now
provided by HA's web UI rather than something we build. The 2026-06-06
4-mode pencil tree is still the canonical mental model; the 2026-06-28
proposal expands it to 7 modes (see session log).

## Key Integrations
- Claude API / Claude Code (development).
- **Enphase Envoy** — local published API on the end user's LAN. Source of
  net power consumption + solar production. Replaces Base Power as the data
  source.
- **Emporia Energy** EVSE — controlled via the Home Assistant community
  Emporia integration's charger-entities PR branch (not upstream stable).
  Cloud-mediated.
- **Home Assistant** on Raspberry Pi 5 — the control plane. Runs the
  automation (mode picker + titration loop).
- **Base Power** — dropped as a data source. Still relevant as the household
  battery / grid abstraction for understanding flows; not consumed by the
  software.

## Tech / Tooling
**Stack is now settled.** Home Assistant OS on a Raspberry Pi 5. The
automation runtime is HA-native: YAML for triggers and entity glue, and
**`pyscript` or `AppDaemon`** for the titration control loop (real Python,
testable — pure YAML gets ugly fast for threshold + hysteresis + dwell-time
logic). UI automation (Appium / ADB) is no longer on the table.

**Dev process:** follow **Matt Pocock's 7-step process** — see
[collateral/Pocock-7-step-process.md](collateral/Pocock-7-step-process.md) (local
only, gitignored). Pocock's skills plugin (`mattpocock-skills`) is installed at
user scope; install recipe + gotchas + per-skill notes live in
[docs/pocock-skills-guide.md](docs/pocock-skills-guide.md). Now that the
stack is real, the natural next Pocock step is **`/to-prd`** (the integration
unknowns that blocked the PRD are resolved).

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
Integration plane shipped (by the end user, on his hardware). Decision-logic
phase begins next.

- [x] Pocock skills installed + verified; per-repo config scaffolded
      (`docs/agents/*.md`, `CONTEXT.md` seeded)
- [x] Friday 2026-06-05 brainstorm with the end user — agenda + meeting
      complete; 8 RQs captured; `CONTEXT.md` and `docs/requirements.md`
      §7 updated with the resolutions
- [x] **Viability Q1 (Emporia EVSE API)** — resolved 2026-06-26 by the
      end user. Community Emporia integration's charger-entities PR
      branch loaded in HA; start/stop/throttle confirmed working
- [x] **Viability Q4 (Vue local API)** — moot 2026-06-16. End user
      pivoted off Vue entirely to Enphase consumption taps (already
      in his possession). No (re-)purchase needed
- [x] **Stack decision** — resolved 2026-06-21. HA on a Raspberry Pi 5.
      Not STM32, not Android-on-Pixel
- [x] **Integration plane** — built by the end user 2026-06-16 → 2026-06-28
      (data source: Enphase Envoy local API; control: HA + community
      Emporia integration)
- [ ] **Confirm mode-list typo** — mode 3 likely should be "75% Solar
      Charge", not "50%". Verify with end user before building
- [ ] **Decision logic / titration loop** — `pyscript` or `AppDaemon` for
      the control law; YAML for trigger glue. Hysteresis + 30–60 s dwell
      per amp step. This is the actual remaining work
- [ ] **Mode picker UX** — HA dashboard surface; map the 2026-06-28
      7-mode list back onto the 2026-06-06 4-mode pencil tree to keep
      the mental model coherent
- [ ] **PRD (`to-prd`)** — now unblocked; integration unknowns are gone
- [ ] **Viability Q5 (Smartcar / car-side API)** — still orthogonal to
      v1 critical path; deferred enhancement
- [ ] **EVSE cold-start-without-WAN fragility** — does HA's path handle
      it? Test before declaring done
- [ ] **Repo posture** — decide whether `solar-sync` becomes the home
      for the end user's HA automations (versioned + tested) or whether
      we collaborate inline on his Pi and archive the repo
- [ ] First end-to-end "decide and act" loop running against real user data

## Session State
**Glossary:** HA = **Home Assistant** (open-source home-automation
platform; runs on the end user's Raspberry Pi 5). Pi = Raspberry Pi 5.
EVSE = the Emporia Classic Level 2 EV charger. Envoy = Enphase's solar
controller / gateway box (the end user's local-API source).

**Last updated:** 2026-07-02 — Debug-support round on the end user's first
HA `templates.yaml`. The end user sent his "Kind of working" email
2026-06-29 with the free-solar case working (charging 10–11 A,
pausing on AC) but occasional spurious start/stop, and asked for a
printf equivalent in HA. Vince analyzed the YAML, wrote off block
#6 as self-correcting math, and shipped two upstream hypotheses on
step #2 (H1: entity-name typo; H2: kW-vs-W unit mismatch), plus a
one-liner template-sensor pattern for printf-style visibility.
The end user responded 2026-06-30: printf answered by choosing HA logs
(via `logger.log` from an automation, gated to plug-in hours); H1
**falsified** (HA actually names the entity `emphoria_...` — his
AI matched the environment, captured as a standing decision); H2
left latent (he did not mention checking the unit); a separate
consumption-chart worry self-resolved once he noticed the 24-hour
timescale. No open ask back. He was heading out for a Tuesday
ride.
**Session log:** [docs/session-history/2026-07-02-kind-of-working-debug.md](docs/session-history/2026-07-02-kind-of-working-debug.md)

**Open / next steps:**
- **H2 (unit-of-measurement check) latent.** Do not re-raise
  unsolicited — the end user's message closed the loop. If he reports the
  flap continuing, next move is to confirm the sensor's
  `unit_of_measurement` in Developer Tools → States.
- **Await the end user's v1 HA YAML** (carried over from the prior session).
  When it lands, verify two interpretive assumptions: (a) "the
  ladder" he committed to includes the 0% Solar rung (he said
  "on/off switch and the ladder" without enumerating rungs), and
  (b) the mode-3 label correction ("75% Solar," not "50% Solar")
  was tacitly accepted.
- **Update `CONTEXT.md` *Decision mode* term** to reference the
  Smart Ladder taxonomy now that the end user has accepted it. The
  current entry references the 2026-06-06 4-mode pencil tree only.
- **Consider an ADR** for the Smart Ladder + Specials paradigm
  split. The end user accepted the structural call; future readers
  will want to know why Hybrid was lifted out of the ladder.
- **YAML expertise stays in reserve.** Don't send the end user
  unsolicited YAML — he knows HA. Contribute UX thinking and
  debugging support; if he asks for code, we have a rough HA YAML
  structure ready (`input_select` + `automation` dispatch +
  parameterized `script` for the titration loop).
- **Run `/to-prd`** — integration unknowns are gone; decision logic
  is well-defined enough to PRD.
- **Decide repo posture.** Either solar-sync becomes a versioned
  home for the end user's HA automations (with tests for the
  control law) or we collaborate inline on his Pi and archive.
- **EVSE cold-start-without-WAN fragility** test (observed twice
  pre-pivot). Does HA's mediation handle it, or does the charger
  losing its cloud session at cold-start still break the API call?
- **Smartcar (Q5)** desk-research remains deferred — orthogonal,
  not v1 critical.

**Standing decisions:**
- Project type: **app** (no agent persona, no `prompts/` directory).
- Project name: `solar-sync`.
- Pocock skills plugin: user scope, via personal wrapper marketplace.
- `marketplace.json` source type: `"source": "github"` shorthand.
- Issue tracker: **GitHub Issues** at `vebutton/solar-sync` via `gh` CLI.
- Triage labels: **canonical defaults** (no remapping).
- Domain layout: **single-context** (`CONTEXT.md` + `docs/adr/` at root).
- Brainstorm artifacts (agenda, notes) live in `collateral/`
  (gitignored), not `docs/` — PII redaction per `CLAUDE.local.md`.
- Session State pattern: per-session narratives in
  `docs/session-history/`; this section holds only standing state +
  the latest session-log pointer.
- **Data source = Enphase Envoy local API** (not Base Power). Single
  signed signal: `Current net power consumption` (+ = import, − =
  export). Locked in 2026-06-28 by the end user's pivot.
- **Emporia power sensor is named `sensor.emphoria_power_minute_average`**
  in the end user's HA instance — with the extra "h." Not a typo. HA lists it
  that way natively (the integration named the entity, not us). Any
  code we discuss with him uses that spelling as-is. Confirmed by the
  end user 2026-06-30.
- **Control plane = Home Assistant on a Raspberry Pi 5**, charger via
  the community Emporia integration's charger-entities PR branch
  (not upstream stable). Locked in 2026-06-26.
- **Automation runtime = `pyscript` or `AppDaemon`** for the control
  loop, YAML for trigger glue. UI automation (Appium / ADB) is no
  longer on the table.
- **"Excess solar"** resolves to **Net excess** (`Production − Usage`);
  with Enphase it's just `−1 × (Current net power consumption)`
  when that value is negative. Base battery is abstracted as part of
  "the grid" (RQ1).
- **Algorithm philosophy:** titrate EV amps so net export → 0; enable
  at ≥ 0.1 kW Net excess; respect the amp-ceiling hierarchy (RQ2 / RQ6).
- **UX direction.** The end user's 4-mode decision tree
  (`docs/requirements.md` §4.0; `CONTEXT.md → Decision mode`) remains
  the canonical mental model. The 2026-06-28 7-mode list expands it
  and the **Smart Ladder + Specials** redesign (proposed and accepted
  2026-06-28) is the v1 UX taxonomy: a five-rung % Solar slider
  (100/75/50/25/0) plus separate buttons for OFF, Hybrid, and the
  12 A / 24 A / 36 A overrides. **v1 ships OFF + the ladder only**;
  Hybrid and the fixed-rate overrides are deferred (Hybrid may be
  dropped entirely per the end user). The threshold is "**minimum
  excess solar to charge**" — an ongoing test against Net excess,
  not a one-time start gate. HA dashboard is the surface, accessed
  from the end user's Pixel browser. See `docs/mode-cheatsheet.md`
  for the full plain-English reference.
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
