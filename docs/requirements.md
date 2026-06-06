# solar-sync — Requirements

> This is the working spec for solar-sync. It is intentionally **incomplete** —
> the project is in the viability phase. Sections marked **OPEN** must be answered
> (mostly with the user, on Friday 2026-06-05) before we can commit to a build path.

---

## 1. End user & goal

Single end user: a friend of Vince's, in a North Texas suburb. Rooftop solar without an owned home
battery. Goal: charge his EV preferentially from solar **excess** (power that would
otherwise be exported to the grid at ~4–5¢/kWh and bought back later at ~15¢/kWh).

the user's quantified upside is ~$10/month. The motivation is *the principle of the
thing* plus interest in the engineering — not financial necessity. This shapes
priorities: **fidelity to the user's mental model matters more than maximum kWh
savings**. The app should reproduce what he does manually today, but automatically.

## 2. The manual loop being replaced

What the user does today:
1. Open the **Base Power** Android app, look at live solar production vs. house
   usage.
2. Decide whether there is "enough more" solar than usage to make charging
   worthwhile (his threshold is judgmental, not numeric).
3. Open the **Emporia Energy** Android app, start the charger and pick amps
   (typically 6–11A).
4. Watch/listen for the dryer or A/C kicking on, or clouds rolling in — manually
   pause or drop the amps.
5. Bias up over time: if it's been cloudy for days, accept ≥50% solar coverage
   rather than holding out for ~100%. If a long trip is coming, just charge.

solar-sync automates steps 2–4 with configurable thresholds and a "mostly free"
mode that captures the step-5 bias logic.

## 3. Hardware & vendor surface

### Already in place
| Component | Notes |
|---|---|
| **Base Power Company** current tap on main breakers | Measures whole-house usage + solar output; data visible in **Base Power** app |
| **Base Power Company** 25 kWh battery | Installed outside the house, owned/traded by Base. Base guarantees ≥20% reserve for outages. Enables solar to work in outages. **Not** a battery solar-sync can directly target. |
| **Emporia Classic Level 2 EVSE** | 6–48A capable; the user's practical range 6–11A. Controlled today via Emporia Energy Android app (pause/resume, amps slider, schedule). |
| **Android phone: Pixel 9a, Android 16** (the user's) | Last update 2026-04-05. Relevant if we go Android-app or UI-automation routes. |

### On order
- **Emporia Vue** current tap on main breakers — the user is ordering this. Emporia's
  own "Excess Solar" mode requires it. Once installed, gives Emporia the same
  whole-house data Base already has. **Open question:** does Vue expose data
  independently of Emporia's cloud?

### Why Emporia's built-in "Excess Solar" isn't enough (the user's words)
- Hard floor of 1.5 kW excess required to activate.
- 3-minute hysteresis required.
- No configurable import/export thresholds (the user wants ~0.5 kW export to
  enable / ~1.5 kW import to disable, or similar).
- No Home Assistant integration today (rumored future paid subscription).

## 4. Functional requirements

### MUST
- **F1.** Read live (or near-live) whole-house power and solar production data.
- **F2.** Issue start, pause, and "set max amps" commands to the Emporia EVSE.
- **F3.** Configurable thresholds — at minimum: enable-charging threshold,
  disable-charging threshold, hysteresis time.
- **F4.** "Mostly free" mode — accept a configurable fraction of solar coverage
  (e.g. 50%, 80%) rather than 100%.
- **F5.** Manual override — the user must be able to force-charge (long-trip case)
  or force-pause without uninstalling the app.

### SHOULD
- **S1.** Continuously adjust amps to track available excess (not just on/off).
- **S2.** A "what is solar-sync doing right now and why" status view.
- **S3.** Daily/weekly summary of how much charging was solar-funded vs. grid.

### COULD (deferred)
- **C1.** Multi-user / installable for someone besides the user.
- **C2.** Integration with weather forecast to anticipate cloud cover.
- **C3.** Battery-aware logic (only relevant if the user ever adds his own battery
  + hybrid inverter; currently uneconomic per his ROI math).

### WON'T (out of scope, at least for v1)
- Anything touching the Base-owned battery directly.
- Anything affecting the house's grid-export contract with Base.
- Generalization beyond the user's specific hardware combo.

## 5. Non-functional & operational

- **Latency:** seconds-to-minutes is fine. This isn't a real-time control loop;
  the user's manual loop runs on the order of minutes.
- **Reliability:** if solar-sync crashes, the Emporia charger should fall back to
  its last setting (or to off), not to an unsafe state. Validate this assumption.
- **Privacy:** all data is the user's; no third-party telemetry.
- **Hosting:** TBD with stack decision. Web app implies a server (where?). Android
  app runs on the user's phone. UI-automation may need a phone-as-server or a tethered
  workstation.

## 6. Interface / stack — **OPEN**

Vince's preference order: **web app → Android app → iOS app**. Not a Pi/STM32
daemon, not a CLI tool. Final decision blocked on viability research — UI
automation, for example, almost certainly forces Android (or a controlled
Android emulator).

## 7. Viability research — **the headline OPEN question**

Before any meaningful build, we need to answer:

### Post-meeting status (2026-06-05)

After the 2026-06-05 brainstorm with the end user (notes in
`collateral/2026-06-05-brainstorm-notes.md`, gitignored), the priority
order changes:

1. **Q1 (Emporia EVSE API) — top priority.** The end user pointed to
   an existing Home Assistant community Emporia integration (likely a
   HACS project). If that proves out against his specific EVSE, the
   vendor-API path is the leading design.
2. **Q4 (Vue local API) — second priority, purchase-gated.** The end
   user is willing to (re-)buy a Vue current tap *if it meaningfully
   helps solar-sync*. We owe him a yes/no/maybe verdict before he
   commits. If Vue publishes data locally, it can replace Base as
   solar-sync's input signal — collapsing the architecture to a single
   vendor (Emporia) for both input and output.
3. **Q2 (Base API) — deprioritized but live.** If Q4 succeeds, Q2 may
   become moot. If Q4 fails, Q2 escalates and likely requires the
   mitmproxy path (with the constraints below).
4. **Q3 (UI automation) — deferred indefinitely.** Tightened access
   constraints from the meeting (see below) make this much harder than
   originally framed.

### Access constraints (from the 2026-06-05 meeting)

- **AI is off the credential path.** Anything that touches the end
  user's accounts is done either in-person side-by-side, or via small
  auditable scripts he can verify with an independent AI (Bing Chat /
  Copilot) before running. solar-sync's credential-handling code must
  stay small enough to survive that audit.
- **Daily-driver Pixel is off-limits for dev mode.** Any invasive
  experimentation (ADB, traffic snooping, UI automation prototyping)
  requires a **spare phone** — identifying that phone is a precondition.
- **mitmproxy is last-resort.** Snooping only (no injection); AI off
  the live-capture path; analysis happens out-of-band on saved files.

### EVSE cold-start fragility (from the 2026-06-05 meeting)

The end user has observed twice that the Emporia EVSE struggles to
start charging after a cold-start without WAN connectivity. Worth a
research item: does the HA Emporia integration handle this case, or
is power-cycling required?

---

### Q1. Does **Emporia** expose a controllable API for the Classic EVSE?
- Official API? Documented? Stable?
- Unofficial community libraries (HACS / Home Assistant integrations / GitHub)?
- What auth model — cloud account credentials, local-network access, OAuth?
- Rate limits / known abuse-detection?
- Emporia themselves told the user "Home Assistant integration is a recurring
  feature request, possibly future paid subscription" — so today: no official
  HA integration.

### Q2. Does **Base Power Company** expose a data API for the house tap?
- Customer-facing API? Partner API? Any documented surface at all?
- Could the **Base Power** Android app be reverse-engineered (mitmproxy on a
  rooted device / certificate-pinning bypass)?
- the user has the data already; what we need is *programmatic* access.

### Q3. Fallback — **UI automation**
- Drive the **Emporia Energy** + **Base Power** Android apps as a human would:
  Appium / `adb` / Android accessibility services / a UI-test harness.
- Feasibility on a Pixel 9a / Android 16. Does either app fight automation
  (root detection, integrity checks, anti-tampering)?
- Could the web equivalents (if any) be driven via Playwright instead — gentler
  than Android automation?
- "Gated/restrictive IoT ecosystem" concern flagged by Vince — sanity-check on
  Friday whether the user's home network or Base's setup blocks any of this.

### Q4. The Emporia **Vue** current tap (on order)
- Does Vue publish its readings *independently* of Emporia's cloud (e.g. a local
  MQTT stream, a published-by-the-device API)?
- If yes, that's a much cleaner data source than driving the Base app.

### Output of viability research
A short written verdict per question (yes / no / probably-with-X) plus the
recommended build path. This is **Step 1 (Idea) of Pocock's 7-step process** —
use the `grill-with-docs` skill once installed.

## 8. References

- **the user's reference implementation:**
  https://beenebrothers.com/2025/08/17/sunpower-sunvault-ac-coupled/
  Different stack (Raspberry Pi, SunPower SunVault) but functionally similar —
  monitors house power, dynamically toggles charging and adjusts rate. Worth
  reading shape-of-the-solution before designing solar-sync.
- **Emporia Classic Level 2 EV Charger:**
  https://shop.emporiaenergy.com/products/emporia-ev-charger
- **Base Power Company:** basepowercompany.com (vendor of the house tap + battery)
- **Matt Pocock's 7-step dev process:**
  see `collateral/Pocock-7-step-process.md` (local, gitignored) and
  https://github.com/mattpocock/skills

## 9. Glossary (the user's pedantic correction, worth honoring)

- **EVSE** (Electric Vehicle Supply Equipment) — the box on the wall (what
  Emporia sells). It *advertises* available current to the car; it does not
  contain the actual charger.
- **Charger** — lives inside the car. Converts AC from the EVSE into DC for
  the battery.
- "EV charger" in casual use (and in Emporia's marketing) means the EVSE.
  solar-sync will use **EVSE** in code and docs to avoid the user's eye-roll.
