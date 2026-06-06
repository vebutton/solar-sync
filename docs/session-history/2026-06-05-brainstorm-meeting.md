# 2026-06-05 — Friday brainstorm capture (end-user meeting)

> Live capture of the Friday 2026-06-05 ~4 PM brainstorm. Detailed
> RQ-by-RQ responses live in
> `collateral/2026-06-05-brainstorm-notes.md` (gitignored for PII).
> This file is the public-repo summary of what we learned and what's
> next.

## Accomplished

- Walked all 8 RQs from the Friday brainstorm agenda (gitignored in
  `collateral/`).
- Captured RQ-by-RQ responses in
  `collateral/2026-06-05-brainstorm-notes.md` (gitignored).
- Updated `CONTEXT.md` with resolved domain terms (Production, Usage,
  Net excess, Base battery, Slider, Dedicated Breaker).
- Re-ranked `docs/requirements.md` §7 viability Q1–Q4; added
  access-constraint and cold-start-fragility subsections.

## Major design clarifications

- **"Excess solar"** resolves to **Net excess** = `Production − Usage`.
  The Base battery is abstracted as part of "the grid" from the end
  user's vantage point (Base calls it a "virtual power plant" asset).
  The two-definition ambiguity in earlier requirements collapses.
- **Algorithm spec.** Titrate EV charging amps so net grid export → 0
  (maximize self-consumption). Enable charging at ≥ 0.1 kW Net excess.
  Disable when Net excess would go negative — *unless* the Slider is
  pulled toward override.
- **Slider UX.** A single continuous 0–100% control replaces the
  separate F4 ("mostly free" mode) and F5 (manual override) from
  requirements.md. 0 = strict solar-only; 100 = force charge
  regardless of solar. The end user explicitly preferred this shape.
- **EV-SOC-driven curve** (end user's mental model for slider use):
  | SOC  | Behavior                                          |
  |------|---------------------------------------------------|
  | 20%  | Force charge — accept 100% grid                   |
  | 50%  | Charge with up to ~30% grid (70% solar OK)        |
  | 70%+ | Wait for 100% solar (unless trip or known bad wx) |
- **Amp ceiling hierarchy** (configurable, not hardcoded):
  | Ceiling                  | Value | Source                       |
  |--------------------------|-------|------------------------------|
  | Dedicated Breaker (hard) | 48A   | Emporia app, hardware safety |
  | Override / trip max      | 40A   | End-user preference          |
  | Auto / solar-tracking    | 12A   | End-user preference          |
  | Solar-tracking floor     | 6A    | EVSE minimum                 |
- **Car is the durable safety net.** The Cadillac Lyriq's own SOC cap
  stops the charging session when reached; solar-sync does not need a
  force-pause primitive. "Stop" = amps to 0 or EVSE pause.

## Viability shifts

- **Q1 (Emporia EVSE API)** — promoted to top: the end user pointed
  at an existing HA community Emporia integration. Desk-research that
  first.
- **Q4 (Vue local API)** — promoted to second; the end user's Vue
  *re-*purchase decision is gated on whether we can confirm it
  meaningfully helps (he returned a Vue once already).
- **Q5 access constraints** — AI off the credential path; mitmproxy
  is last-resort, snooping-only, and OOB-analysis-only; a spare phone
  is required before any invasive work.
- **Stack tension** — the end user explicitly prefers a "dumb device"
  (STM32) over anything that needs OS maintenance, but the leading
  Q1 path (HA integration) is Linux/Python. Options under
  consideration: Pi + auto-updating OS, Android app on the daily
  driver, STM32 + custom Emporia client, hybrid.

## Other notable inputs

- **Primary value prop = summer AC-frustration relief.** kWh savings
  is the secondary framing; the headline pain is the manual loop
  becoming intolerable when the AC cycles often. Lead with frustration
  in any PRD / pitch.
- **No deadline, no hours commitment** from the end user — relaxed
  pace is appropriate.
- **Real-world anchors** for algorithm work: 0.4 kW baseline house
  load, ~2.5 kW sweet-spot Net excess, ~3 mi/kWh Lyriq efficiency,
  ~1 minute EVSE start-up hysteresis after a "begin charging" command.
- **EVSE cold-start without WAN is a known fragility** (observed
  twice). Open research item.
- **Identity redaction stays as-is** on the public repo (RQ8).

## Decisions captured

- Domain language additions to `CONTEXT.md`: Production, Usage, Net
  excess, Base battery, Slider, Dedicated Breaker.
- Algorithm philosophy: titrate to Net excess ≈ 0; respect amp
  ceilings; enable at 0.1 kW excess; rely on the car's SOC cap for
  "stop."
- UX direction: Slider replaces F4 + F5 (still subject to a design
  pass; not yet ADR-locked).
- Viability priority order: Q1 → Q4 → Q2 → Q3.
- Project narrative: summer frustration relief is the headline; kWh
  savings is secondary.
- PII redaction remains in force; `CLAUDE.local.md` identity mapping
  unchanged.
