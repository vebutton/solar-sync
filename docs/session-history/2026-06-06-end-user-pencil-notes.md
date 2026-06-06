# 2026-06-06 — End-user pencil-note decision tree (async drop)

> Async drop from the end user between the 2026-06-05 brainstorm and
> the next live session. Vince forwarded 4 phone screenshots plus the
> end user's transcribed pencil notes on how he wants charging
> decisions made. The 2026-06-05 brainstorm produced a Slider sketch;
> today's drop displaced it with a 4-mode decision tree as the
> canonical mental model.

## Source material

Lives in `collateral/20260605/` (gitignored — phone screenshots carry
the end user's identifiers and live data):

| File                              | What it shows                                                                                              |
|-----------------------------------|------------------------------------------------------------------------------------------------------------|
| `Screenshot_20260605-161802.png`  | Base app *Live* view (Usage + Solar production cards, 15-min trend). The view the end user checks most when deciding whether to charge. |
| `Screenshot_20260605-162701.png`  | Base app *Daily overview* with backup-duration estimate. The duration swings wildly (0–125 hr) with instantaneous draw — supports treating the Base battery as opaque "grid." |
| `Screenshot_20260605-170140.png`  | Cadillac Lyriq Charge Management — 54% SOC, target 65% set in the car app. Confirms the car-side SOC cap as the durable stop signal. |
| `Screenshot_20260605-171400.png`  | Battery-degradation curves vs SOC at 0 °C / 20 °C / 40 °C (Arrhenius / SEI growth). Physics rationale for the "SOC-capped + solar-only" strategy: high SOC × heat is the worst combination. |

The end user's pencil notes, transcribed by Vince:

1. **Don't charge** — off.
2. **Charge when free** — `solar > usage by 1.5 kW to 3 kW`. Clarified
   as: charge when `usage + planned EVSE draw < solar input`.
3. **Charge to prevent export** — don't send to the grid if it can
   charge the car instead; configurable level.
4. **Always charge** — on regardless.

He also pointed at `magico13/ha-emporia-vue`
(https://github.com/magico13/ha-emporia-vue) as the Home Assistant
community Emporia integration to evaluate for Viability Q1.

## Accomplished

- Inserted **§4.0 — Decision model (the end user's 4-mode tree)** at
  the top of `docs/requirements.md` §4. Reframed F3 (per-mode
  thresholds), F4 (mode selection — subsumes the prior "mostly free"
  framing), and F5 (modes 1 + 4 as the bypass states).
- Added **§6 UX surface — OPEN** to `docs/requirements.md`:
  mode-picker vs single-Slider vs both. Leans mode-picker; defers the
  final call to the end user's reaction once viability is in.
- Added the **Decision mode** term to `CONTEXT.md`. Demoted the
  **Slider** term to *provisional, 2026-06-05* with an explicit note
  that a slider may re-emerge *inside* mode 2 (the 1.5–3 kW EV-draw
  band) rather than as the top-level control.
- Rewrote the **UX direction** standing decision in `CLAUDE.md` to
  reflect the provisional Slider status. Bumped *Last updated* to
  2026-06-06. Updated the Q1 next-step bullet to carry the
  `magico13/ha-emporia-vue` URL with a "name says Vue — confirm
  charger entities exist" caveat.

## Major clarification: 1.5–3 kW is the EV draw band, not headroom

First-pass framing of the end user's mode-2 numbers had "1.5–3 kW of
headroom" as a configurable margin above the planned EV draw. Vince
caught this on review: **1.5–3 kW maps cleanly to 6–12A at 240V**,
which is exactly the end user's stated practical amp band (6–11A in
`docs/requirements.md` §3) and the 6A floor / 12A auto-mode rungs in
the amp-ceiling hierarchy (`CLAUDE.md` standing decisions). So the
figure is the *planned EVSE draw* the user dials in, not a margin
above it.

Mode 2 then reads: `Production > Usage + planned EV draw`, with the
planned draw configurable in roughly the 1.5–3 kW window. This also
re-opened the slider question — a slider *could* re-emerge in mode 2
as the in-mode control for the planned draw, now captured in both
`CONTEXT.md` (Slider term) and `docs/requirements.md` §6.

**Open:** confirm with the end user whether "1.5 kW to 3 kW" denotes
the EV draw band (current reading), a configurable margin above the
draw, or his observed solar peak. Flagged in `CONTEXT.md` and §4.0 of
`docs/requirements.md`.

## Decisions captured

- **4-mode tree is canonical.** The end user described the modes
  verbatim, unprompted — strong evidence they match his mental model
  better than the slider abstraction does.
- **Slider is provisional, not killed.** If technical viability lands
  and the end user reacts well to a slider variant, it may resurface
  — most plausibly *inside* mode 2 rather than at the top level. The
  `Slider` term in `CONTEXT.md` is annotated accordingly.
- **No ADR yet.** Both the 4-mode tree adoption and the Slider
  demotion are reversible until the PRD pass. Neither meets the
  hard-to-reverse + surprising bar for an ADR.

## Process note: PII redaction caught and fixed

First-pass edits to `CLAUDE.md`, `CONTEXT.md`, and
`docs/requirements.md` used the end user's first name (forwarded by
Vince in conversation) verbatim. `CLAUDE.local.md` establishes the
redaction convention — "the end user" / "the user" / "a friend" in
all committed files — and this session caught the leak on
`grep -rn` review before commit. Fixed via a single bulk substitution
per file. Lesson queued as auto-memory: when a `CLAUDE.local.md`
identity mapping exists, never write the real name into committed
files even when the conversation uses it freely.

## Open / next

- **Viability Q1.** Concrete moving piece: desk-research
  `magico13/ha-emporia-vue`. Confirm whether the integration surfaces
  charger entities (start/pause + set amps) for the end user's
  Emporia Classic L2 EVSE, or whether only the upstream `PyEmVue`
  library exposes that.
- **Confirm 1.5–3 kW interpretation** at the next live session
  (draw band vs margin vs solar peak).
- **PRD pass eventually** — defer until Q1 (and ideally Q4) verdicts
  are in.
