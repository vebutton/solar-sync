# 2026-07-02 — Debug support for the end user's first HA templates.yaml

> Session spans 2026-06-29 (email in) → 2026-07-02 (session close). The end
> user sent his first working HA template sensor with a reported bug and
> asked for a "printf" equivalent. Vince responded with two hypotheses; the
> end user came back with resolutions and a self-diagnosis of a separate
> scare.

## What we did

1. **Read the end user's "Kind of working" email** (`collateral/_Kind of_ working.eml`,
   2026-06-29). His HA template sensor `sensor.solar_ev_target_amps` was
   working for the free-solar scenario (charging at 10–11 A, pausing when
   AC kicked in), but he saw occasional spurious start/stop cycles and hit
   his Gemini free limit mid-fix. He pinpointed block #6 (the "Granular
   Cushion Floor Rule") as the suspected bug and asked whether HA has a
   printf equivalent so he could watch intermediate variables tick.

2. **Analyzed the YAML.** Block #6 math actually self-corrects: step #3
   (`solar_export_w = -net_w + charger_w`) adds the charger's own draw
   back into the available-solar figure, so the target stays stable
   across the charge/pause transition as long as `charger_w` is real and
   in watts. The likely bug lived upstream at step #2
   (`sensor.emphoria_power_minute_average`), not at #6.

3. **Formed two hypotheses:**
   - **H1 (entity name typo):** "emphoria" instead of "emporia," making
     `states(...) | float(0)` fall through to 0.
   - **H2 (unit mismatch):** sensor reports kW but the code treats it as
     W, under-counting by 1000×.

   Either failure mode causes the charger's draw to never effectively
   cancel out. The controller then flaps between "start at 6 A" (while
   paused) and "pause" (right after starting) every 2 minutes, matching
   the end user's start/stop symptom.

4. **Drafted the reply email.** Two-part structure: the debug template-
   sensor pattern for printf-style visibility, and the two hypotheses to
   verify in HA Developer Tools → States. Vince edited before sending
   (added opener "Great work. Two thoughts" and a PS on the $3 AI
   subscription; condensed the body).

5. **Verified the "Developer Tools → States" UI path before sending.**
   Vince caught the AI-suggested UI reference and asked for confirmation
   before relaying it to the end user. Confirmed it's the standard HA web-UI
   path (sidebar → Developer tools → States tab), admin-role only, and
   collapses under the hamburger on phone-sized screens.

6. **the end user responded 2026-06-30**
   (`collateral/20260630_Kind of_ working.eml`) with:
   - **Printf resolution.** He is going with HA logs (via the
     `logger.log` action from an automation) rather than debug template
     sensors. Persistent, filterable, and he can gate the logging window
     to the ~4–6 hours per day the car is plugged in.
   - **H1 falsified.** "emphoria" is not a typo. HA lists the entity
     that way natively (his AI was matching the real environment). Any
     future code we look at should assume that spelling is authoritative.
     Captured as a standing decision in `CLAUDE.md`.
   - **H2 unresolved.** He did not mention checking the sensor's
     `unit_of_measurement`. If the flap continues we will hear about
     it; if it stops, H2 was moot.
   - **Consumption chart worry, self-resolved.** He briefly worried the
     metering was noisy after seeing 24-hour spikes, then noticed the
     chart timescale and identified the spikes as AC + dryer cycles
     (with an inverted stretch 1–3 PM when solar-sync was holding net
     usage near zero, which is exactly the intended behavior).

## Decisions captured (see `CLAUDE.md → Standing decisions`)

- **The "emphoria" spelling is authoritative** in the end user's HA instance
  (`sensor.emphoria_power_minute_average`). Not a typo to fix. Any code
  we discuss with him uses that spelling as-is (confirmed 2026-06-30).

## Open / next steps

- **H2 (unit check) latent.** Do not re-raise unsolicited — the end user's
  message closed the loop. If he reports the flap continuing, next move
  is to confirm the sensor's `unit_of_measurement` in Developer Tools →
  States.
- **Wait for the end user's next update.** He was heading out for a Tuesday
  ride after his reply. If the ladder v1 lands stably, next natural
  work item on our side is `/to-prd`.

## Reusable takeaways

- **HA "printf" pattern, two clean options.**
  (a) Expose each intermediate as a one-liner template sensor and watch
      them in **Developer Tools → States** (or drop them on the dashboard).
      Real-time and self-updating.
  (b) Log from an automation via the `logger.log` action. Persistent
      and filterable, can be windowed to periods that matter.

  The end user chose (b). Both belong in the toolkit.

- **Verify the UI path before relaying it.** Vince's instinct to ask
  "are you sure of Developer Tools → States" before sending was right.
  Any time a reply hinges on a live-app path a friend will follow
  blindly, confirm it first.

- **Bugs live upstream more often than the reporter thinks.** the end user
  pointed at block #6 (his most recent AI edit); the actual candidate
  was block #2 (sensor availability + units). Reading the whole
  pipeline before accepting the reporter's localization is worth the
  minute it costs.
