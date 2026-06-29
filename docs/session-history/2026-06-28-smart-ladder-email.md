# 2026-06-28 — Smart Ladder UX proposal + outbound email to the end user

> Second half of 2026-06-28. After capturing the end user's HA pivot
> (see [2026-06-28-end-user-pivot-to-ha.md](2026-06-28-end-user-pivot-to-ha.md)
> and commit `45c2561`), the rest of the session went into reading
> the end user's proposed mode list, finding a cleaner UX taxonomy,
> drafting an email back to him, and capturing two new writing-style
> preferences as feedback memories.

## What we did

1. **Walked the end user's 7-mode list** in plain English. Caught
   what looked at first like a labeling typo (mode 3 = "50%" with a
   0.4 kW grid budget; should arithmetically be "75%").
2. **Caught a deeper structural issue.** Modes 2–4 share one control
   law ("use up to X kW grid to reach the 1.5 kW minimum; above the
   minimum, only solar adds amps"). Mode 5 uses a *different* law
   ("hold 1.2 kW grid constantly, add solar on top"). The "% Solar"
   label hides the paradigm break.
3. **Proposed the Smart Ladder + Specials split** (Vince's idea
   after seeing the mode-5 asymmetry):
   - **Smart Ladder** — five rungs (100% / 75% / 50% / 25% / 0%
     Solar) all using the same control law, varying only in start
     threshold. A genuine slider. Adds a **0% rung** (overnight /
     start-anytime mode) the end user hadn't included.
   - **Specials** — buttons, not slider positions: OFF, **Hybrid**
     (mode 5 renamed to match its actual behavior — 1.2 kW grid +
     solar on top), and the 12 A / 24 A / 36 A fixed-rate overrides.
4. **Drafted the email** through five revisions:
   - v1: full design memo with "your house, your call" personality
   - v2: stripped personality, kept tables (Vince's feedback: "fake
     personality")
   - v3: wrapped table cells with `<br>` for narrower render
   - v4: pure plain-text bullets (Vince: "I like the tables")
   - v5 (sent): HTML tables, copied to clipboard via osascript with
     `«class HTML»` so Gmail renders bordered tables on paste
5. **Vince sent the email.** Now awaiting the end user's response.
6. **Discussed YAML.** Vince asked what HA YAML would look like
   for the modes ("not familiar with YAML used in such a dynamic
   way"). Walked through `input_select` (mode picker), `automation`
   with `choose:` dispatch by mode, parameterized `script` for the
   titration loop, and how Jinja2 templates make YAML "dynamic" at
   runtime. **Held back from sending the YAML to the end user** —
   he likely knows HA better than the sketch shows, and sending
   reads as "doing his work" instead of offering UX input.

## Two new writing-style preferences captured

Both saved as feedback memories so future drafts apply them
automatically:

1. **[Don't fake Vince's voice in outgoing drafts](../../../.claude/projects/-Users-vebutton-Development-AI-Apps-solar-sync/memory/feedback_dont_fake_vinces_voice.md)**
   — drafts go out factual / neutral. Vince adds his own opener,
   closer, and asides. Specifically flagged: "your house, your
   call," "bear with me," "tell me where I'm wrong," bike-ride
   callbacks, opening compliments.
2. **[Use simple present, not progressive, in drafts](../../../.claude/projects/-Users-vebutton-Development-AI-Apps-solar-sync/memory/feedback_simple_present_in_drafts.md)**
   — "I read" not "I'm reading"; "I think" not "I'm thinking";
   verbs of cognition / perception / possession / state default to
   simple present. Progressive only for genuine in-the-moment
   action.

Also extended the **flag-stray-untracked-files** memory: PDFs are
exempt (Vince's reading copies, generated intentionally — don't
flag, don't gitignore, don't ask).

## Decisions captured

- **Smart Ladder + Specials is provisional**, not adopted.
  Awaiting the end user's reaction. If he accepts the design, mode
  IDs in the eventual automation will be the ladder's percent
  labels plus OFF / Hybrid / 12A / 24A / 36A. If he pushes back, we
  reconvene from his 7-mode list.
- **No ADR yet.** This is reversible until the end user weighs in
  and the YAML actually lands.
- **YAML expertise held in reserve.** Don't send the end user
  unsolicited YAML drafts. He has more HA experience than us;
  contribute UX thinking, not code.

## Open / next

- **Await the end user's response** to the Smart Ladder proposal.
  Possibilities:
  - He accepts → we offer the YAML sketch if useful, otherwise
    just stay available for debugging.
  - He accepts with tweaks → integrate his changes, update the
    canonical mode list in `CONTEXT.md` and `docs/requirements.md`.
  - He pushes back on the paradigm split → reconvene on whether
    mode 5's asymmetry was intentional and a different framing
    helps.
- **If accepted, update `CONTEXT.md`** — the *Decision mode* term
  there currently references the 2026-06-06 4-mode pencil tree.
  The Smart Ladder is the same family expanded; worth a small
  edit so the terminology stays current.
- **Consider an ADR** if the end user adopts Smart Ladder — the
  decision to split the paradigm into ladder + specials is a
  structural call that future readers will want to understand.
- **Cold-start-without-WAN fragility** test remains on the books.
- **Repo posture decision** still open (does solar-sync host the
  end user's HA automations, or do we archive?).
