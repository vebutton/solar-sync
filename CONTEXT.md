# solar-sync

solar-sync automates a single household's EV charging so it draws preferentially from rooftop-solar excess rather than grid power. It replaces a human-in-the-loop process of eyeballing one app and toggling switches in another.

## Language

**EVSE**:
The Electric Vehicle Supply Equipment — the wall-mounted box that advertises available current to the car. Does not contain a charger. In this project it is specifically the Emporia Classic Level 2 unit.
_Avoid_: EV charger, charger, charging station, wall box

**Charger**:
The component inside the car that converts AC delivered by the EVSE into DC for the traction battery. solar-sync never talks to the Charger; it talks to the EVSE.
_Avoid_: onboard charger (use Charger), inverter

**Production**:
The instantaneous solar power generation, in kW, as reported by the Base app. One of the two numbers the Base dashboard surfaces in real time.
_Avoid_: generation, solar output, PV output

**Usage**:
The instantaneous whole-house power draw, in kW, as reported by the Base app. Includes the EVSE's draw when the EV is charging. The other of the two real-time numbers on the Base dashboard.
_Avoid_: consumption, load, demand

**Net excess**:
`Production − Usage`. Positive = power leaving the house (whether absorbed by the Base battery or exported to the grid; the Base app does not distinguish). Negative = power imported from the grid. solar-sync's primary control signal.
_Avoid_: excess solar (informal — use Net excess in code and docs), surplus

**Base battery**:
The 25 kWh battery Base owns and operates outside the house. From the end user's vantage point — and the Base app's — it is part of "the grid" (Base calls it a "virtual power plant" asset). solar-sync never reads or targets it directly.
_Avoid_: home battery (the end user doesn't own one), battery (ambiguous with the car's traction battery)

**Decision mode**:
The discrete behavioral state solar-sync is in at any moment. the end user's 2026-06-06 pencil notes define four: (1) **Don't charge**, (2) **Charge when free** (only when Production exceeds Usage plus the *planned EVSE draw* — the end user's intended draw band is 1.5–3 kW, which maps to his 6–12A practical amp range; the check is forward-looking), (3) **Charge to prevent export** (titrate amps so Net excess tracks toward zero, with a configurable import-allowed level), (4) **Always charge**. Modes 1 and 4 are bypass states; 2 and 3 are the algorithmic modes and differ in *direction* — mode 2 requires Net excess ≥ the planned EV draw before enabling; mode 3 tolerates a configurable import below zero excess before pausing. The control surface that exposes these to the user is OPEN (see `docs/requirements.md` §6); the 1.5–3 kW band is one plausible place a *Slider* could re-emerge *inside* mode 2.

**Open** on Decision mode: the end user wrote "1.5 kW to 3 kW" — confirm with him whether that denotes the EV draw band (current reading), a configurable margin above the planned draw, or his observed solar peak. Affects how mode 2 is exposed.

_Avoid_: charging mode (ambiguous with Emporia's "Excess Solar" mode), behavior preset

**Slider** (provisional, 2026-06-05):
A single continuous 0–100% control surface sketched at the 2026-06-05 brainstorm to collapse mode selection and the force-charge override into one knob. **Held as provisional** since 2026-06-06: the end user's own pencil notes describe the same problem as a discrete picker over four named Decision modes, which is now the canonical mental model. A slider-shaped variant may resurface during the PRD pass — most plausibly *inside* mode 2 (the 1.5–3 kW EV draw band) rather than as the top-level control. Not currently load-bearing in code or design — use **Decision mode** instead.
_Avoid_: treating Slider as canonical until UX is decided; mostly-free mode, force-charge button (both subsumed by the 4-mode tree)

**Dedicated Breaker**:
Emporia's term, set inside the Emporia Android app, for the maximum amperage the EVSE will ever deliver. The hardware safety ceiling — solar-sync cannot exceed it by any means.
_Avoid_: amp cap (ambiguous — there are multiple amperage ceilings in this system)
