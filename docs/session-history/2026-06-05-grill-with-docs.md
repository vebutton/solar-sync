# 2026-06-05 — `/setup-matt-pocock-skills` + `/grill-with-docs`

> Session narrative archive. Load on demand — `CLAUDE.md` keeps only the
> standing state. See `docs/session-history/README.md` (if it exists) for
> the directory's purpose.

## Accomplished

- Ran `/setup-matt-pocock-skills` after the previous-session restart
  confirmed slash commands were live. Wrote the `## Agent skills` block
  in `CLAUDE.md` (between `## Environment` and `## Project Status`) and
  created `docs/agents/issue-tracker.md`, `docs/agents/triage-labels.md`,
  `docs/agents/domain.md` from the seed templates. Choices: **GitHub
  Issues** via `gh` CLI; **default triage labels** (1:1 mapping —
  `wontfix` already exists on the repo, the other four will be created
  on first `triage` run); **single-context** domain layout.
- Ran `/grill-with-docs` against `docs/requirements.md`. Created
  `CONTEXT.md` and seeded it with the resolved EVSE/Charger pair from
  requirements §9. No ADRs warranted (no decisions met the
  hard-to-reverse + surprising + real-trade-off bar).
- Drafted the Friday 2026-06-05 brainstorm agenda — 8 end-user-facing
  questions (RQ1–RQ8) covering excess-solar semantics (RQ1), threshold
  numbers
  (RQ2), "mostly free" mode (RQ3), manual override UX + the "maybe we
  don't need our own UI" probe (RQ4), account/network/mitmproxy consent
  (RQ5), EVSE fallback behavior (RQ6), engagement + Vue ETA (RQ7),
  public-repo redaction consent (RQ8). The agenda lives in `collateral/`
  (gitignored) to keep the end user's real name out of the public repo
  per `CLAUDE.local.md`. RQ1 sub-q 1 (what the Base app actually shows
  in real time) is flagged as the single highest-leverage answer — it
  cascades through F3, F4, and §7 priorities.
- Refactored Session State out of `CLAUDE.md` into per-session files
  under `docs/session-history/` (this directory). `CLAUDE.md` Session
  State now holds only the open steps + standing decisions + a pointer
  to the latest narrative.

## Decisions made this session

- Issue tracker = **GitHub Issues** at `vebutton/solar-sync` via `gh`
  CLI (matches the existing remote; default the Pocock skills were
  designed around).
- Triage labels = **canonical defaults** (`needs-triage`, `needs-info`,
  `ready-for-agent`, `ready-for-human`, `wontfix`). No remapping needed.
- Domain layout = **single-context** (`CONTEXT.md` + `docs/adr/` at
  root). `docs/adr/` not created yet — lazy per the format guide.
- Session shape for the grill: **build the Friday agenda** rather than
  resolve definitions now. Most foundational terms ("excess solar,"
  "mostly free") depend on end-user-side knowledge of the Base app +
  battery behavior, so resolving them without him would be guessing.
- Pre-Friday viability research: **none**. Agenda only. All viability
  desk research deferred until the end user's answers are in.
- Friday brainstorm agenda location: **`collateral/`** (gitignored),
  not `docs/`. PII redaction rule from `CLAUDE.local.md` is the driver.
- Session State pattern: **per-session narratives live in
  `docs/session-history/`** (this file is the first). `CLAUDE.md`
  Session State keeps only the standing state + a pointer.
