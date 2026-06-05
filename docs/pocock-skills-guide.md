# Matt Pocock Skills — Install & Usage Guide

A reusable recipe for installing Matt Pocock's engineering skills
(`github.com/mattpocock/skills`) into Claude Code, and a quick reference for
the skills `solar-sync` will use. Written so it can be lifted into other
projects.

## What this is

Matt Pocock publishes a set of opinionated Claude Code skills that implement
his 7-step engineering process: Idea → Spike → Prototype → PRD → Kanban →
Implementation → Polish. The skills are the prompts; you run them as slash
commands once installed.

Upstream: <https://github.com/mattpocock/skills>

Pocock's local-only step reference for this project lives at
`collateral/Pocock-7-step-process.md` (gitignored).

## Skills used in solar-sync

| Step          | Skill                       | When                                |
|---------------|-----------------------------|-------------------------------------|
| (install)     | `setup-matt-pocock-skills`  | One-time per repo, after install    |
| 1 Idea        | `grill-with-docs`           | Pre-viability — sharpen the idea    |
| 3 Prototype   | `prototype`                 | After viability is decided          |
| 4 PRD         | `to-prd`                    | Once requirements are firm          |
| 5 Kanban      | `to-issues`                 | Turn the PRD into issues            |

Other skills the plugin loads but solar-sync isn't planning to use yet:
`diagnose`, `tdd`, `triage`, `improve-codebase-architecture`, `zoom-out`,
`caveman`, `grill-me`, `handoff`, `write-a-skill`.

## Installation

Pocock's repo is a **single plugin**, not a marketplace, so the obvious
`claude plugin marketplace add mattpocock/skills` fails (it looks for
`.claude-plugin/marketplace.json` and finds only `plugin.json`). The working
recipe is to wrap it in a tiny personal marketplace.

### 1. Create a wrapper marketplace

```bash
mkdir -p ~/.claude/personal-marketplaces/pocock-wrapper/.claude-plugin
```

Write `~/.claude/personal-marketplaces/pocock-wrapper/.claude-plugin/marketplace.json`:

```json
{
  "$schema": "https://anthropic.com/claude-code/marketplace.schema.json",
  "name": "pocock-wrapper",
  "description": "Personal wrapper marketplace pointing at mattpocock/skills",
  "owner": { "name": "Vince Button" },
  "plugins": [
    {
      "name": "mattpocock-skills",
      "description": "Matt Pocock's engineering + productivity skills",
      "author": { "name": "Matt Pocock" },
      "source": {
        "source": "github",
        "repo": "mattpocock/skills"
      },
      "homepage": "https://github.com/mattpocock/skills"
    }
  ]
}
```

The `"source": "github"` shorthand clones via SSH (`git@github.com:`) and gives
a full working tree. **Prerequisite:** your GitHub SSH key must be set up and
`ssh -T git@github.com` must succeed (see Gotcha 1). If SSH to GitHub is not an
option on your machine, see Gotcha 3 for the `git-subdir` workaround.

### 2. Register and install

```bash
claude plugin marketplace add ~/.claude/personal-marketplaces/pocock-wrapper
claude plugin install mattpocock-skills@pocock-wrapper
```

### 3. Restart Claude Code

Plugin loading happens at startup. Skills won't appear as `/<name>` slash
commands until the next session.

### 4. Verify

```bash
claude plugin list
```

You should see `mattpocock-skills@pocock-wrapper` enabled.

## Install gotchas (learned the hard way)

These bit us on the solar-sync install. If you hit them again, the fixes are
known.

### Gotcha 1 — `"source": "github"` shorthand requires working GitHub SSH

The marketplace.json `"source": { "source": "github", "repo": "..." }` form
clones via SSH (`git@github.com:`). It fails on machines that don't have a
GitHub SSH key set up, or where SSH access is otherwise blocked. Symptom:
`Permission denied (publickey)` or a host-key error at install time.

**Fix:** set up an SSH key for GitHub and verify with `ssh -T git@github.com`
(should respond `Hi <user>! You've successfully authenticated...`). Once SSH
works, the `"github"` shorthand is the right form — it gives a full working
tree and avoids the sparse-checkout trap in Gotcha 3.

**Watch-out:** a global git config URL rewrite
(`url.git@github.com:.insteadOf https://github.com/`) will route every clone
through SSH whether you want it or not. Check with
`git config --global --get-regexp 'url\..*\.insteadof'` if anything else
behaves oddly.

**If SSH to GitHub really isn't an option:** fall back to `"source":
"git-subdir"` with an explicit `https://` URL — but read Gotcha 3 first, the
naive form silently installs *nothing*. Do **not** try `"source": "git"` —
it returns "This plugin uses a source type your Claude Code version does not
support."

### Gotcha 2 — stale GitHub host key in `~/.ssh/known_hosts`

GitHub rotated their RSA SSH host key in March 2023 after a key exposure
incident. If your `~/.ssh/known_hosts` predates that rotation, the install
fails with:

> `@ WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED! @`
> `Offending RSA key in ~/.ssh/known_hosts:NN`

The fingerprint the server actually sends — `SHA256:uNiVztksCsDhcc0u9e8Buj-`
`QXVUpKZIDTMczCvj3tD2s` — is GitHub's current published RSA fingerprint
(<https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/githubs-ssh-key-fingerprints>),
so it's not a real MITM, just a stale entry.

**Fix:**

```bash
ssh-keygen -R github.com
ssh-keyscan -H github.com >> ~/.ssh/known_hosts
```

Switching to the `git-subdir` HTTPS form (see Gotcha 3 for the working
variant) also sidesteps this entirely.

### Gotcha 3 — `"source": "git-subdir"` with `"path": "."` silently sparse-empties the install

This is what bit `solar-sync` on the first install. The marketplace.json
form

```json
"source": {
  "source": "git-subdir",
  "url": "https://github.com/mattpocock/skills.git",
  "path": ".",
  "ref": "main"
}
```

…clones the repo into `~/.claude/plugins/cache/<marketplace>/<plugin>/<sha>/`
under sparse-checkout with the cone pattern `.`, which means **include only
the top-level files but no subdirectories**. The plugin's `SKILL.md` files
all live under `skills/`, so none of them get materialized and **no slash
commands appear** — but `claude plugin list` still reports the plugin as
installed, and `settings.json`'s `enabledPlugins` shows it as enabled. Two
days went by before anyone noticed.

**Symptoms (read in this order):**

1. `/<skill-name>` returns "Unknown command".
2. `~/.claude/settings.json` `enabledPlugins` still shows the plugin as `true`.
3. `~/.claude/plugins/installed_plugins.json` shows it installed with a real
   sha and `installPath`.
4. `ls <installPath>` shows only `.git/` and `.in_use/` — no SKILL.md, no
   `.claude-plugin/`, no `scripts/`.
5. `git -C <installPath> status` says `You are in a sparse checkout with 0%
   of tracked files present`.
6. `cat <installPath>/.git/info/sparse-checkout` shows `--cone\n.\n`.

**Live fix on an existing broken install** (does not require reinstall):

```bash
PLUG="$(jq -r '.plugins["mattpocock-skills@pocock-wrapper"][0].installPath' \
  ~/.claude/plugins/installed_plugins.json)"
git -C "$PLUG" sparse-checkout disable
```

Then restart Claude Code so the plugin loader re-scans the directory.

**Permanent fix:** switch to `"source": "github"` (Gotcha 1) once SSH to
GitHub works. If you genuinely must stay on HTTPS, omit `"path"` entirely or
use a non-`.` value — but verify the install materialised real files before
trusting it. Always spot-check:

```bash
ls ~/.claude/plugins/cache/<marketplace>/<plugin>/<sha>/.claude-plugin/
# should list plugin.json
```

## Safety audit

**Re-audited 2026-06-05** against the materialized files at
`~/.claude/plugins/cache/pocock-wrapper/mattpocock-skills/.../`
(commit `aaf2453fbdfe`). An earlier audit dated 2026-06-03 turned out to be
done while the install was sparse-checkout-empty (Gotcha 3) — its
conclusions were directionally right but a couple of details were
imprecise. This section reflects the verified state.

- **No script in the plugin executes a network call.** No `curl`, `wget`,
  `subprocess`, `os.system`, or runtime `fetch()` anywhere in any `.sh`,
  `.py`, `.js`, `.ts`, or `.json` file.
- **No telemetry, analytics, or tracking SDKs.** (False positives that look
  like "tracking" are CSS class names and triage-label strings.)
- **4 shell scripts total**, in two locations:
  - `scripts/link-skills.sh` and `scripts/list-skills.sh` at the repo root —
    local symlink/find utilities. `link-skills.sh` has a defensive guard
    against self-linking into its own repo.
  - `skills/engineering/diagnose/scripts/hitl-loop.template.sh` — an
    interactive `read`-driven template. No network.
  - `skills/misc/git-guardrails-claude-code/scripts/block-dangerous-git.sh`
    — defensive PreToolUse hook that exits 2 on matching dangerous git
    patterns. **Not active by default** — see next point.
- **Only 14 skills are loaded** by the plugin manifest at
  `.claude-plugin/plugin.json`: 10 under `skills/engineering/` and 4 under
  `skills/productivity/`. The repo also contains `skills/misc/`,
  `skills/personal/`, `skills/in-progress/`, and `skills/deprecated/` —
  those are present on disk but **not registered as skills**. Notably the
  `git-guardrails-claude-code` skill (which contains `block-dangerous-git.sh`)
  lives under `misc/` and won't show up as a slash command unless you
  explicitly opt in by invoking it some other way.
- **Setup skill (`setup-matt-pocock-skills`)** is prompt-driven, not a
  script. It edits **your own** `CLAUDE.md` or `AGENTS.md` (whichever
  exists, never creates the other) by adding or updating an `## Agent skills`
  block, and writes 3 seed files under `docs/agents/`. Per the SKILL.md:
  *"Don't overwrite user edits to the surrounding sections."* It shows you
  a draft and asks for confirmation before writing. Also carries
  `disable-model-invocation: true` in its frontmatter, so the model won't
  auto-fire it — you must explicitly run `/setup-matt-pocock-skills`.
- **External URLs in markdown are documentation/branding only**, never
  executed by the plugin:
  - `cdn.tailwindcss.com` and `cdn.jsdelivr.net/npm/mermaid` — loaded only
    if you opt in to the `improve-codebase-architecture` HTML report and
    open it in a browser.
  - `cloudinary.com` logo images, `aihero.dev` newsletter link, Amazon book
    references, GitLab CLI docs — all README content.
  - `skills.sh` badge in the README quickstart promotes
    `npx skills@latest add mattpocock/skills` as an alternative installer.
    **We are not using that path** — the wrapper marketplace bypasses it
    entirely. If you ever copy the quickstart instead, audit `skills.sh`
    separately.

**Caveats / what this audit does NOT cover:**

- Skill *prompts* can still instruct Claude to run arbitrary code if you
  invoke them. The audit confirms the plugin itself has no autonomous
  network/exec behavior; it does not confirm each skill is safe to run in
  every context. Read each `SKILL.md` before first use.
- Audit is pinned to commit `aaf2453fbdfe`. If you update the plugin,
  re-run the spot checks (especially the network/exec greps from
  `scripts/list-skills.sh` and the manifest).

## Quick usage reference

After install + restart, the relevant slash commands are:

- `/setup-matt-pocock-skills` — one-time per repo. Scaffolds the
  `## Agent skills` block in your `CLAUDE.md` plus `docs/agents/{issue-`
  `tracker,triage-labels,domain}.md`. Walks you through 3 decisions: issue
  tracker (GitHub / GitLab / local-markdown / other), triage label
  vocabulary, single- vs multi-context domain docs.
- `/grill-with-docs` — Step 1. Interrogates an idea against linked docs to
  sharpen it before you commit to a build path. Good fit for pre-viability
  work like solar-sync's "do these vendors even expose APIs?" question.
- `/prototype` — Step 3. Builds a throwaway proof-of-concept.
- `/to-prd` — Step 4. Turns a grilled idea into a PRD.
- `/to-issues` — Step 5. Splits a PRD into issues.

Read each skill's `SKILL.md` (under `~/.claude/plugins/.../skills/`) for the
full contract before relying on it.

## Reusing this guide

To install Pocock's skills on another project / machine: copy this guide,
run the steps in **Installation**, re-run the **Safety audit** spot-checks,
and you're done. The wrapper marketplace is user-scope so it persists
across projects on the same machine — you only need to do the install once
per machine, not once per repo.
