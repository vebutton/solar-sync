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

**Slider**:
The single continuous control surface in solar-sync's UI, conceptually 0–100%. 0 = strict solar-only (only charge from Net excess). 100 = force charge regardless of solar. Replaces the separate "mostly free" mode and manual force-charge override from early requirements drafts.
_Avoid_: mostly-free mode, force-charge button (both subsumed)

**Dedicated Breaker**:
Emporia's term, set inside the Emporia Android app, for the maximum amperage the EVSE will ever deliver. The hardware safety ceiling — solar-sync cannot exceed it by any means.
_Avoid_: amp cap (ambiguous — there are multiple amperage ceilings in this system)
