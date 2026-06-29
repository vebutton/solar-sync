# Decision-mode cheatsheet

> Personal refresher for Vince. The Smart Ladder + Specials design
> proposed to the end user 2026-06-28 and substantively accepted the
> same evening (with one terminology fix and a scope cut to v1).
> If you're returning to this after a gap, read top to bottom — it
> takes about three minutes.
>
> **Status (2026-06-28):**
> - **v1 scope:** OFF + Smart Ladder only. End user voluntarily
>   deferred Hybrid and the fixed-amp overrides; he may never build
>   Hybrid at all ("use case is a bit unclear").
> - **All modes** are documented here anyway, since the design lives
>   as one family and the deferred ones may resurface.
> - **End user's terminology fix:** what looked like "min solar to
>   start" is really "min *excess* solar needed to *charge*." It is
>   an ongoing test (start *and* continue) measured against excess
>   solar (production minus other house loads), not gross production.

## What each mode does

| Mode | What it does |
|---|---|
| **OFF** | Turn the charger off and step back. Nothing further until you change modes. |
| **100% Solar** | Charge only when excess solar covers the 1.5 kW minimum on its own. Above the minimum, ride solar — never pull a watt from grid. Pause if excess drops below 1.5 kW. |
| **75% Solar** | Charge as long as there's at least 1.1 kW of excess solar (grid covers the remaining 0.4 kW). Above the minimum, ride solar — no extra grid. Pause if excess drops below 1.1 kW. |
| **50% Solar** | Charge as long as there's at least 0.7 kW excess solar (grid covers 0.8 kW). Above the minimum, solar-only. Pause if excess drops below 0.7 kW. |
| **25% Solar** | Charge as long as there's at least 0.3 kW excess solar (grid covers 1.2 kW). Above the minimum, solar-only. Pause if excess drops below 0.3 kW. |
| **0% Solar** | Charge whenever plugged in — grid covers the entire 1.5 kW minimum if needed. As sun rises, the loop automatically reduces grid use and rides solar above the minimum. Doesn't pause on low sun. |
| **Hybrid** *(deferred)* | Always pull 1.2 kW from grid, plus whatever excess solar is available on top. With 2.8 kW excess solar, charges at 4.0 kW. Doesn't pause on low sun — grid keeps it going. |
| **12 A** *(deferred)* | Override: lock the charger at 12 A (~2.9 kW) regardless of solar. |
| **24 A** *(deferred)* | Override: lock the charger at 24 A (~5.8 kW) regardless of solar. |
| **36 A** *(deferred)* | Override: lock the charger at 36 A (~8.6 kW) regardless of solar. |

## Two things to notice as you read down

**The ladder is one rule with one knob.** Every Smart Ladder rung
asks the same question every tick: "is there enough excess solar to
keep charging within my grid budget?" The only thing that varies
between rungs is the budget. Above the 6 A floor, all five rungs do
the same thing — back off grid when sun is plentiful. The
differences only show up at the boundary, when sun is marginal.

**0% Solar and Hybrid never pause for low sun.** They are the two
"always charging" modes — but for different reasons. 0% Solar
accepts up to the 1.5 kW minimum from grid and rides any solar
above it. Hybrid holds 1.2 kW of grid as a constant baseline and
treats solar as bonus on top. 0% is "I'll plug in overnight and the
car charges, period." Hybrid is "I need a fast charge today and I'm
willing to pay for it."

## One subtlety on 0% Solar / overnight

At night with no sun and other house load active (fridge, AC, etc.),
0% Solar still starts the EV at the 6 A floor. The EV's share of
grid draw at the floor is 1.5 kW — within the 0%-rung budget. But
the *house* may also be importing 0.5–1 kW for everything else.
Total grid bill includes both. That is expected behavior, but worth
knowing: 0% Solar does not mean "no extra grid above what the house
is already pulling" — it means "the EV can draw up to its 1.5 kW
minimum from grid." If a "stop importing more than the house
already was" mode is wanted later, that is a different rule and a
different knob.

## Why Hybrid is a separate paradigm

The end user's original 7-mode list put Hybrid at the bottom of a
"% Solar" ladder, but it does not follow the ladder rule. Modes 2–4
say "minimize grid use; ride solar above the minimum." Mode 5
(Hybrid) says "hold a constant 1.2 kW grid; use solar as bonus."
With 2.8 kW of excess solar, modes 2/3/4 all charge at 2.8 kW (no
grid above the minimum); Hybrid charges at 4.0 kW (1.2 grid + 2.8
solar). The label "25% Solar Charge" on the original mode 5 hid the
difference. Splitting Hybrid out of the ladder is what made the
ladder clean enough to be a single slider.

## Where the design lives

- **Session log (the email exchange):**
  `docs/session-history/2026-06-28-smart-ladder-email.md`
- **Domain term (canonical):** `CONTEXT.md → Decision mode`
- **Original 4-mode pencil tree (the seed):**
  `docs/requirements.md` §4.0 and the
  `2026-06-06-end-user-pencil-notes.md` session log
- **Standing decisions and current Open/Next:** `CLAUDE.md`
- **Status going forward:** end user is writing the YAML himself
  (HA, with Gemini as assistant). solar-sync's role is design input
  and debugging partner, not code delivery.
