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

### 4.0 Decision model — the end user's 4-mode tree (2026-06-06)

the end user's own pencil notes (dropped between the 2026-06-05 brainstorm and
the 2026-06-06 follow-up) give the canonical mental model for what
solar-sync decides moment-to-moment. The four modes are *what the end
user wants to choose between*; F1–F5 below implement them.

| # | Mode                         | Behavior                                                                                                                                                                                                                       |
|---|------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 1 | **Don't charge**             | EVSE off regardless of solar state.                                                                                                                                                                                            |
| 2 | **Charge when free**         | Charge only when `Production > Usage + planned EVSE draw`. the end user's intended EV draw band is **1.5–3 kW**, which maps to his 6–12A practical amp range — whether the 1.5–3 kW figure denotes draw, margin, or his solar peak still needs confirming with him. The check is **forward-looking** — it must account for the load the EVSE itself will add, not just instantaneous Usage. |
| 3 | **Charge to prevent export** | Titrate amps so Net excess (`Production − Usage`) tracks toward zero — i.e. consume excess locally rather than export it. Configurable import-allowed level for when production momentarily dips.                              |
| 4 | **Always charge**            | EVSE on regardless of solar state. Force-charge (e.g. long-trip prep).                                                                                                                                                         |

Modes 1 and 4 are bypass states. Modes 2 and 3 are the algorithmic
modes; they differ in *direction*: mode 2 requires Net excess ≥ the
planned EV draw before enabling (solar must already cover the load the
EVSE is about to add); mode 3 tolerates a configurable level of import
below zero excess before pausing.

> **Tension with the Slider direction.** A standing decision in
> `CONTEXT.md` (term: *Slider*) and `CLAUDE.md` collapses all
> configurable behavior into a single 0–100% control. the end user's pencil
> notes instead frame the problem as a **discrete mode picker with
> per-mode settings**. Reconciling these is an OPEN UX question — see
> §6.

### MUST
- **F1.** Read live (or near-live) whole-house power and solar production data.
- **F2.** Issue start, pause, and "set max amps" commands to the Emporia EVSE.
- **F3.** Configurable thresholds for the algorithmic modes (see §4.0):
  the *headroom required* in mode 2 (Charge when free), the
  *import-allowed level* in mode 3 (Charge to prevent export), and the
  hysteresis time both share. Mode 2's check is forward-looking — it
  must include the EVSE's own planned draw, not just instantaneous Usage.
- **F4.** Mode selection — the end user must be able to choose between
  the four modes in §4.0, and within each algorithmic mode set its
  threshold. Subsumes the earlier "mostly free" framing. (UX surface
  for this is OPEN — see §6.)
- **F5.** Manual override = modes 1 (Don't charge) and 4 (Always
  charge). These bypass states must be reachable in one or two taps
  without uninstalling or reconfiguring the app.

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

### Platform choice

Vince's preference order: **web app → Android app → iOS app**. Not a Pi/STM32
daemon, not a CLI tool. Final decision blocked on viability research — UI
automation, for example, almost certainly forces Android (or a controlled
Android emulator).

### UX surface — **OPEN** (added 2026-06-06)

the end user's 4-mode tree (§4.0) and the prior single-Slider direction
(`CONTEXT.md → Slider`) are two different mental models for the same
control problem. Decide before the PRD pass:

- **Mode picker + per-mode settings.** Honors the end user's mental model
  directly: a four-way picker; modes 2 and 3 each reveal one numeric
  control. Maps 1:1 to what he drew on paper.
- **Single slider** (the standing decision from 2026-06-05). One
  continuous 0–100% knob; the mode is *inferred* from where the user
  lands. Visually clean; obscures the *kind* of decision being made
  and asks the user to translate from his own mental model into ours.
- **Both** (modes as preset positions on the slider, with fine-tune).
  Possibly the worst of both worlds — every UI element costs
  explanation.

Lean toward the mode picker: the end user described the four modes verbatim,
unprompted, in his own pencil notes. That is strong evidence that the
modes match his mental model better than a slider does. Defer the
final call to his reaction.

## 7. Viability research — **the headline OPEN question**

Before any meaningful build, we need to answer:

### Post-meeting status (updated 2026-06-06)

After the 2026-06-05 brainstorm with the end user (notes in
`collateral/2026-06-05-brainstorm-notes.md`, gitignored) and the
2026-06-06 async drop, the priority order is:

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
5. **Q5 (Smartcar / car-side API) — added 2026-06-06, orthogonal to
   the v1 critical path.** The end user pointed at
   `smartcar.com/pricing` as an option for reading live SOC (and
   possibly writing the charge limit) from the Cadillac Lyriq.
   Useful enhancement but does not displace the Emporia path — it
   talks to the *car*, not the EVSE. Slot as deferred enhancement;
   revisit when v1 viability (Q1, Q4) is settled.

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

### Q5. **Smartcar** or any other car-side API (added 2026-06-06)

URL the end user pointed at: https://smartcar.com/pricing.

Smartcar is a third-party broker wrapping multiple OEM connected-car
APIs behind a single OAuth surface. If usable for the end user's
Cadillac Lyriq it would let solar-sync *read* live SOC (a signal
currently visible only to the human in the Cadillac app) and possibly
*write* the charge limit, start, and stop on the car side. Today the
durable stop in solar-sync is the target-SOC slider the end user
maintains by hand in the Cadillac app — see standing decision in
`CLAUDE.md`. Reading SOC live would let solar-sync stop *dynamically*
and would unlock the SOC-driven curve sketched at the 2026-06-05
brainstorm (20% → force / 50% → 70% solar OK / 70%+ → wait for sun).

**Pricing as of 2026-06-06 (per smartcar.com/pricing):**

| Tier   | Floor cost                              | Vehicles  | Commands/mo | Notes                                                                |
|--------|-----------------------------------------|-----------|-------------|----------------------------------------------------------------------|
| Free   | $0                                      | 1         | **0**       | "Limited vehicle signals." Read-only — no `commands` (start/stop/set).|
| Build  | "Starting at $1.99 / vehicle / app / mo"| up to 100 | 100         | Build Basic / Advanced / Premium variants (depth unknown).            |
| Custom | Negotiated                              | 500 min   | Tailored    | Enterprise; not in scope for a single-vehicle hobby project.          |

For the end user's case (1 VIN, low command volume), the **Build
floor is ~$1.99/mo** — roughly 20% of the project's ~$10/mo upside
per §1, *if* the floor really is the all-in number for the
sub-features we need. Acceptable. The Free tier's **0 commands/mo**
means any *write* (start, stop, set-limit) requires Build.

**Still open before Smartcar becomes a build path:**

- **Q5.a — OEM coverage.** Is the Cadillac Lyriq supported, and at
  what feature depth (signals only / signals + commands)? The
  pricing page does not list OEMs — needs the "Compatible brands"
  page and possibly OEM-specific docs.
- **Q5.b — Endpoint-to-tier mapping.** Specifically: is the battery
  state (SOC) endpoint included in Free-tier "limited signals," or
  gated to Build? Same question for `charging.start/stop` and
  `set charge limit`.
- **Q5.c — Credential-path fit.** Smartcar's OAuth puts *the end
  user* in the consent flow — that aligns with the **RQ5 "AI off
  the credential path"** constraint. But every command call carries
  a token; solar-sync's command-issuing code must stay small enough
  to audit per RQ5.
- **Q5.d — Reliability lattice.** Adding Smartcar makes solar-sync
  depend on three vendor surfaces (Base + Emporia + Smartcar) plus
  GM's connected-car backend behind Smartcar. Each link is a
  potential outage. Define explicitly: when Smartcar is unreachable
  but Emporia and Base are fine, solar-sync falls back to amp
  titration with the human-set SOC cap as today.

**Two architectures Smartcar enables:**

1. **Read-only (Free or Build).** solar-sync reads live SOC;
   control stays purely Emporia-side. The Cadillac app still owns
   the durable stop. Cheap and small.
2. **Read + write (Build tier, ~$1.99/mo).** solar-sync reads SOC
   *and* writes the charge limit / start / stop. Displaces the
   Cadillac app on the car side. Lives squarely under the RQ5
   credential-audit lens.

Neither displaces the Emporia path — Smartcar talks to the car, not
the EVSE. The EVSE remains how solar-sync titrates amps in response
to Net excess.

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
