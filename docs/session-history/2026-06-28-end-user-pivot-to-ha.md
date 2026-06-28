# 2026-06-28 — End user pivots to Home Assistant on a Pi (async drops)

> Three async emails from the end user across 2026-06-16 → 2026-06-28
> (saved in `collateral/20260628/`, gitignored). Vince was heads-down
> on other work for most of that window; the end user moved the project
> forward unilaterally and we caught up at the end of the month.
> Net effect: the integration plane that solar-sync was scoped to
> build is now **already built by the end user**, on his hardware.

## Source material

`collateral/20260628/`:

| File                              | Date         | What it shows                                                                                              |
|-----------------------------------|--------------|------------------------------------------------------------------------------------------------------------|
| `Todos.eml`                       | 2026-06-16   | End user proposes the architecture pivot: drop Base Power data source in favor of his Enphase controller (local API, already on his LAN). Frames two paths — HA on a Pi vs. roll-your-own. Vince's reply (2026-06-21): "go path #1, HA on a Pi." |
| `Progress report.eml`             | 2026-06-26   | Pi arrived. HA installed in ~1 hr, Enphase auto-discovered. ~3 hrs on the case fan. ~6–7 hrs the next day fighting Gemini through the community Emporia HA integration to load the right branch with charger-specific entities. Outcome: HA can start/stop the charger and vary its output. |
| `I now have consumption data.eml` | 2026-06-28   | Consumption current taps wired into the Enphase controller. HA now sees `Current net power consumption` (signed: + = import, − = export). All inputs and outputs are in HA. Proposes a mode-picker UX with 7 modes. |

## What the end user actually built (without us)

- **Data plane.** Enphase Envoy (local published API, already on LAN)
  + Enphase consumption CTs (shipped with original solar install,
  never wired in until now). One signal does the job:
  *Current net power consumption*. No Base Power dependency.
- **Control plane.** Home Assistant on a Raspberry Pi 5, with a
  community Emporia integration (specific PR branch with charger
  entities, not the upstream stable). Can start/stop the EVSE and
  set its output amps from HA.
- **Hardware install.** ~70–90 ft attic run from the panel to the
  shed where the Enphase controller lives, using ethernet cable to
  extend the CT leads (Enphase supports up to 150 ft on consumption
  taps). Done.

**What's left:** the *decision logic* — the YAML/Python automation
that reads net consumption and dials EVSE amps. The 6 hours he lost
to Gemini were on integration setup, not on the algorithm; the
algorithm hasn't been written.

## End user's proposed UX (from 2026-06-28)

A slider that selects a mode + a Go/Save button to commit. Modes,
verbatim (typos preserved):

1. **OFF** — turn the charger off and stop managing it.
2. **Solar Only** — never import; titrate amps to track excess.
3. **"50%" Solar Charge** — allow up to **0.4 kW** import to hit the
   6A / 1.5 kW minimum charge rate; don't exceed excess solar above
   the minimum.
4. **"50%" Solar Charge** — same shape, allow up to **0.8 kW** import.
5. **25% Solar Charge** — allow up to **1.2 kW** import *and target
   1.2 kW import* even when above the minimum (different control law:
   hold import constant rather than `export → 0`).
6. **Charge at 12 A**
7. **Charge at 24 A**
8. **Charge at 36 A**

## Open questions on the mode list

- **Label slip.** Two modes are labeled "50% Solar Charge" with
  different kW caps (0.4 and 0.8). Given the third is "25% Solar
  Charge" at 1.2 kW, mode 3 is almost certainly meant to be
  **"75% Solar Charge"** (0.4 kW grid ≈ 27% of the 1.5 kW min).
  Confirm with the end user before hardcoding labels.
- **Mode 5 is a different control law.** Modes 2–4 minimize import
  subject to a floor; mode 5 *targets* import. Worth a clearer name
  ("Hybrid 1.2 kW grid + solar topping") so future-end-user doesn't
  trip on it.
- **Mapping to the 4-mode pencil tree.** The new list is the same
  family as 2026-06-06 (Cheap / Mixed / Solar / Override), just
  exploded — Solar Only = mode 3 (Solar); modes 3–5 = three variants
  of Mixed; modes 6–8 = Override-On; OFF = Override-Off. Top-level
  shape unchanged.

## Effect on viability questions

- **Q1 (Emporia HA integration).** Effectively resolved — the end
  user got it working on the charger-specific PR branch. Caveat: not
  the upstream stable; will need to track that branch's status if
  he wants long-term updates. The `magico13/ha-emporia-vue` lead
  from 2026-06-06 was useful but he ended up on the broader
  community integration with the right charger-entities branch.
- **Q4 (Vue local API).** Moot — he pivoted off Vue to Enphase
  consumption taps. No (re-)purchase decision needed.
- **Q5 (Smartcar).** Still orthogonal and deferred.
- **Stack decision.** Resolved: HA on Pi 5. Not STM32, not Android
  on Pixel.

## Decisions captured

- **Data source = Enphase Envoy local API** (not Base Power).
  Single signal: `Current net power consumption` (signed).
- **Control plane = Home Assistant on a Raspberry Pi 5**, charger
  via the community Emporia integration's charger-entities branch.
- **solar-sync the codebase pivots to the decision logic.** The
  integration plane is no longer ours to build. The repo's job is
  now the mode picker + titration loop, versioned alongside the
  end user's HA install.
- **UX is HA-native.** The mode picker / slider lives in HA's
  dashboard, accessed from the end user's Pixel browser. The "web
  app" / "Android app" / "iOS app" preference order in the old
  CLAUDE.md is moot — HA's web UI is the web app.

## Open / next

- **Confirm the mode-list typo** with the end user (mode 3 = 75%
  Solar, not 50%).
- **Draft the automation.** Recommend `pyscript` or `AppDaemon`
  add-on for the control loop (real Python, testable). Pure YAML
  for trigger glue. Hysteresis + dwell-time (30–60 s per amp step)
  matter more than thresholds.
- **EVSE cold-start-without-WAN fragility.** Still on the books.
  HA mediates the API call but if the charger loses its cloud
  session at cold-start, HA's call still fails. Worth a test
  before declaring done.
- **Repo posture.** Decide whether solar-sync the repo becomes
  the home for the end user's HA automations (versioned, with
  tests for the control law) or whether we just collaborate
  inline on his Pi and archive the repo. Vince's call.

## Process note

This is the first session driven entirely by async drops with no
live working session in between. The 2026-06-05 Friday cadence
held for two weeks and then broke — Vince got busy and the end
user took the project further than expected on his own. The
session-history file pattern handled the gap well: we picked up
from the standing-decisions section + the three emails without
needing live context.
