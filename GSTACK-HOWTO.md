# How to Use gstack

Source: https://github.com/garrytan/gstack

gstack turns Claude Code (and 9 other coding agents) into a virtual engineering
team: a CEO who challenges scope, an eng manager who locks architecture, a
designer who catches AI slop, a reviewer who finds production bugs, a QA lead
who opens a real browser, a security officer, and a release engineer — all as
slash commands, all Markdown, MIT licensed. It's a **process**, not a grab-bag
of tools: Think → Plan → Build → Review → Test → Ship → Reflect, with each
skill feeding the next.

## 1. Install (30 seconds)

**Requirements:** Claude Code, Git, Bun v1.0+, Node.js (Windows only).

Paste this into Claude Code — it does the rest:

```
Install gstack: run git clone --single-branch --depth 1
https://github.com/garrytan/gstack.git ~/.claude/skills/gstack &&
cd ~/.claude/skills/gstack && ./setup
```

That also asks Claude to add a `## gstack` section to your `CLAUDE.md`
pointing at `/browse` for web browsing and listing the available skills —
which is exactly the block already present in this environment's
`CLAUDE.md`, so gstack is active here.

### Team mode (shared repos)

From inside a repo, so every teammate gets gstack automatically with no
vendored files and no manual upgrades:

```bash
(cd ~/.claude/skills/gstack && ./setup --team) && \
  ~/.claude/skills/gstack/bin/gstack-team-init required && \
  git add .claude/ CLAUDE.md && \
  git commit -m "require gstack for AI-assisted work"
```

Swap `required` for `optional` to nudge teammates instead of blocking them.

### Other agents

gstack also supports Codex CLI, OpenCode, Cursor, Factory Droid, Kiro, Slate,
OpenClaw, Hermes, and GBrain — auto-detected by `./setup`, or targeted with
`./setup --host <name>`. Any rules-reading agent (Zed, Amp, Jules) can get a
lightweight instruction-only version by copying
`agents-digest/gstack-AGENTS.md` into a file it reads.

## 2. Quick start

1. Install gstack.
2. `/office-hours` — describe what you're building.
3. `/plan-ceo-review` on any feature idea.
4. `/review` on any branch with changes.
5. `/qa` on your staging URL.
6. Stop there — you'll know if this is for you.

## 3. A typical run, end to end

```
You:    I want to build a daily briefing app for my calendar.
You:    /office-hours
Claude: [asks about the pain, not the feature] → writes a design doc

You:    /plan-ceo-review
        [challenges scope, runs a 10-section review]

You:    /plan-eng-review
        [data flow diagrams, test matrix, failure modes]

You:    Approve plan. Exit plan mode.
        [implementation happens]

You:    /review
        [auto-fixes obvious issues, flags the rest for approval]

You:    /qa https://staging.myapp.com
        [opens a real browser, clicks through flows, fixes bugs found]

You:    /ship
        [tests run, PR opened]
```

Each skill hands its output to the next one automatically — the design doc
from `/office-hours` feeds `/plan-ceo-review`, the test plan from
`/plan-eng-review` feeds `/qa`, and so on.

## 4. Core workflow skills

| Command | Role | Use it to... |
|---|---|---|
| `/office-hours` | YC Office Hours | Start here — 6 forcing questions that reframe a rough idea before code is written. |
| `/plan-ceo-review` | CEO / Founder | Challenge scope on a plan (Expansion / Selective Expansion / Hold Scope / Reduction). |
| `/plan-eng-review` | Eng Manager | Lock architecture, data flow, edge cases, and a test plan. |
| `/plan-design-review` | Senior Designer | Score each design dimension 0–10 and edit the plan up to a 10. |
| `/plan-devex-review` | DX Lead | Interactive developer-experience review before you build a dev-facing surface. |
| `/design-consultation` | Design Partner | Build a design system from scratch; writes `DESIGN.md`. |
| `/autoplan` | Review Pipeline | Runs CEO → design → DX → eng review automatically, one command. |
| `/review` | Staff Engineer | Find production-shaped bugs after code is written; auto-fixes the obvious ones. |
| `/investigate` | Debugger | Systematic root-cause debugging — no fixes without investigation. |
| `/qa` | QA Lead | Drive a real browser through your app, fix bugs, add regression tests. |
| `/qa-only` | QA Reporter | Same as `/qa` but report-only, no code changes. |
| `/cso` | Security Officer | OWASP Top 10 + STRIDE threat model with low false-positive rate. |
| `/ship` | Release Engineer | Sync main, run tests, push, open the PR. |
| `/land-and-deploy` | Release Engineer | Merge, wait for CI/deploy, verify production health. |
| `/document-release` | Technical Writer | Update README/ARCHITECTURE/etc. to match what just shipped. |
| `/retro` | Eng Manager | Weekly retro across people, streaks, and test health. |

Full deep dives for every skill (including the design pipeline, browser
tooling, and power tools like `/careful`, `/freeze`, `/guard`, `/codex`,
`/pair-agent`): [docs/skills.md](https://github.com/garrytan/gstack/blob/main/docs/skills.md).

### Which review do I run?

| Building for... | Before code | After shipping |
|---|---|---|
| End users (UI/app) | `/plan-design-review` | `/design-review` |
| Developers (API/CLI/SDK) | `/plan-devex-review` | `/devex-review` |
| Architecture | `/plan-eng-review` | `/review` |
| All of the above | `/autoplan` | — |

## 5. Browser tooling

`/browse` is the shared engine every browser skill (`/qa`, `/scrape`,
`/design-review`, `/canary`, `/benchmark`) stands on. On macOS with the
[Aside](https://aside.com) browser open, these drive Aside directly — your
real, logged-in sessions. Everywhere else (Linux, Windows, or Aside closed),
they fall back to gstack's own bundled Chromium automatically — same
skills, same evidence. `/open-gstack-browser` launches that fallback browser
headed with a sidebar AI assistant, anti-bot stealth, and cookie import.
Never call `mcp__claude-in-chrome__*` tools directly — always go through
`/browse`.

## 6. Safety guardrails

- `/careful` — warns before destructive commands (`rm -rf`, `DROP TABLE`,
  force-push). Say "be careful" to trigger it.
- `/freeze` — restricts edits to one directory.
- `/guard` — `/careful` + `/freeze` together, for prod work.
- `/unfreeze` — removes the freeze boundary.

## 7. Parallel sprints

gstack works with one sprint at a time but is built for running several.
[Conductor](https://conductor.build) can run 10–15 Claude Code sessions in
parallel, each in its own isolated workspace — one on `/office-hours` for a
new idea, another on `/review` for a PR, a third implementing, a fourth on
`/qa`. The sprint structure (think → plan → build → review → test → ship) is
what keeps that many agents from turning into chaos: each one knows exactly
what to do and when to stop, and you check in only on the decisions that
matter.

## 8. Persistent memory (GBrain, optional)

`/setup-gbrain` gets a persistent knowledge base running for your agent in
under 5 minutes (local PGLite, Supabase, or remote MCP). `/sync-gbrain`
re-indexes a repo's code into it and updates `CLAUDE.md` so the agent prefers
`gbrain search` over grep. Full guide: `USING_GBRAIN_WITH_GSTACK.md` in the
repo.

## 9. Updating

```bash
/gstack-upgrade
```

Detects a global vs. vendored install, syncs both, and shows what changed.
Or set `auto_upgrade: true` in `~/.gstack/config.yaml`.

## 10. Uninstall

```bash
~/.claude/skills/gstack/bin/gstack-uninstall
```

Removes skills, symlinks, global state (`~/.gstack/`), project-local state,
browse daemons, and temp files. Add `--keep-state` to preserve config, or
`--force` to skip confirmation. Afterward, manually remove the `## gstack`
and `## Skill routing` sections from any `CLAUDE.md` you added them to (the
uninstaller doesn't edit that file). If you never cloned the repo locally,
the README's manual-removal steps cover cleanup by hand.

## 11. Troubleshooting

- **Skill not showing up?** `cd ~/.claude/skills/gstack && ./setup`
- **`/browse` says `NEEDS_ASIDE`?** That's just the fallback-browser probe;
  open Aside and sign in, or ignore it and gstack uses its own Chromium.
- **Stale install?** `/gstack-upgrade`
- **Want shorter/longer command names?** `./setup --no-prefix` (e.g. `/qa`)
  or `./setup --prefix` (e.g. `/gstack-qa`).
- **Windows:** works via Git Bash or WSL; requires both `bun` and `node` on
  PATH. Re-run `./setup` after every `git pull` if symlinks aren't available
  (no Developer Mode) — setup prints a reminder when this applies.

## Privacy

Telemetry is opt-in and off by default. If enabled, only skill name,
duration, success/fail, gstack version, and OS are sent — never code, file
paths, repo names, or prompts. `gstack-config set telemetry off` disables it
at any time. Every off-machine send is logged to a tamper-evident local
ledger, auditable with `gstack-egress list` / `gstack-egress verify`.

## Links

- Repo: https://github.com/garrytan/gstack
- Skill deep dives: https://github.com/garrytan/gstack/blob/main/docs/skills.md
- Architecture: https://github.com/garrytan/gstack/blob/main/ARCHITECTURE.md
- Browser internals: https://github.com/garrytan/gstack/blob/main/BROWSER.md
- Contributing: https://github.com/garrytan/gstack/blob/main/CONTRIBUTING.md
