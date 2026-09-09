# How to Use GSD Core

Source: https://github.com/open-gsd/gsd-core

**"Git. Ship. Done."** GSD Core is a lightweight meta-prompting, context-
engineering, and spec-driven development system for AI coding agents (Claude
Code, OpenCode, Antigravity CLI, Kimi CLI, Kilo, Codex, Copilot, Cursor,
Windsurf, and more). It solves **context rot** — the quality degradation that
builds up as an agent's context window fills — by pushing all heavy research,
planning, and execution work into fresh-context subagents while your main
session stays lean.

## 1. Install

Requires Node.js.

```bash
npx @opengsd/gsd-core@latest
```

The installer prompts for your runtime (Claude Code, OpenCode, Antigravity
CLI, Kimi CLI, Kilo, Codex, Copilot, Cursor, Windsurf, ...) and whether to
install globally or locally. **Always use the installer** — it handles
cross-runtime compatibility; don't copy files out of `agents/` or `commands/`
by hand. Runtimes without a native installer path are covered in
[Install on your runtime](https://github.com/open-gsd/gsd-core/blob/main/docs/how-to/install-on-your-runtime.md).

### Command syntax by runtime

- Claude Code / Copilot / OpenCode / Kilo: `/gsd-command-name [args]`
- Codex: `$gsd-command-name [args]`

Same commands, runtime-specific spelling — the installer writes the correct
form for whichever agent you picked.

## 2. Start a project

```bash
/gsd-new-project     # greenfield — new project from scratch
/gsd-onboard         # brownfield — bring GSD into an existing codebase
```

`/gsd-new-project` does deep interactive context-gathering (or
`--auto @prd.md` to extract from an existing doc) and produces `PROJECT.md`,
`REQUIREMENTS.md`, `ROADMAP.md`, `STATE.md`, and `CLAUDE.md`.

`/gsd-onboard` checks repo state and routes you through codebase mapping,
optional docs ingestion, and project initialization, ending with an
onboarding summary. Use `--fast` for a lightweight first pass.

## 3. The phase loop — the core mental model

All work moves through the same five-step cycle, one **phase** at a time:

```
Discuss → (UI design, optional) → Plan → Execute → Verify → Ship
```

| Step | Command | Why it exists |
|---|---|---|
| Discuss | `/gsd-discuss-phase` | Captures implementation decisions (libraries, error handling, edge-case behavior) *before* planning, so the planner isn't guessing. Writes `CONTEXT.md`. |
| UI design (optional) | `/gsd-ui-phase` | For visually complex phases — writes a `UI-SPEC.md` design contract before code is written. |
| Plan | `/gsd-plan-phase` | Fresh-context subagents research the ecosystem (`RESEARCH.md`), decompose the work into `PLAN.md` files ordered into dependency waves, and a plan-checker verifies completeness before execution starts. |
| Execute | `/gsd-execute-phase` | Runs plans in parallel waves; each executor gets a clean 200k-token context loaded with only what its plan needs, and commits atomically per task. |
| Verify | (part of the phase pipeline) | Checks what was built against the phase goal, requirement coverage, and the decisions in `CONTEXT.md`; produces `VERIFICATION.md` and fix plans if something's off. |
| Ship | `/gsd-ship` | Creates the PR, archives phase artifacts, marks the phase complete in `STATE.md`. |

A **milestone** is a version cycle — a releasable increment with its own
requirements. A **phase** is one bounded unit of work inside it. Good phase
scope: statable in one sentence, boundable research, parallelizable into a
handful of plans, and independently verifiable (e.g. "Add HMAC-SHA256
signature validation middleware"). "Build the authentication system" is
usually too broad — split it. Something as small as a typo fix is *below*
the threshold where the loop is worth it — use `/gsd-quick` instead.

Everything the loop produces lives in `.planning/` and survives across
sessions and context resets — `STATE.md` is the navigation layer that tells
any agent (or you) exactly where the project currently sits.

## 4. A typical phase, end to end

```bash
/gsd-spec-phase 3        # clarify WHAT phase 3 delivers (optional, ambiguity-scored)
/gsd-discuss-phase 3     # decide HOW — writes CONTEXT.md
/gsd-plan-phase 3        # research + decompose — writes PLAN.md files
/gsd-execute-phase 3     # parallel-wave execution, fresh context per executor
# verification runs as part of the pipeline, producing VERIFICATION.md
/gsd-ship                # PR, archive, mark phase complete
```

Or just run `/gsd-next` any time — it detects project state and routes you
to the right next action without you tracking where you are manually.

## 5. Namespace routers (v1.40+)

Six routers act as low-cost entry points that route to the full ~86-skill
surface — every concrete command is still directly invocable if you know its
name:

| Router | Routes to |
|---|---|
| `/gsd-workflow` | Phase pipeline: discuss / plan / execute / verify / phase / progress / next |
| `/gsd-project` | Project lifecycle: milestones, audits, summary |
| `/gsd-quality` | Quality gates: code review, debug, audit, security, eval, ui |
| `/gsd-context` | Codebase intelligence: map, graphify, docs, learnings |
| `/gsd-manage` | Management: config, workspace, workstreams, thread, update, ship, inbox |
| `/gsd-ideate` | Exploration & capture: explore, sketch, spike, spec, capture |

## 6. Useful commands outside the main loop

| Command | Use it for |
|---|---|
| `/gsd-quick` | A trivial task with GSD's atomic-commit/state-tracking guarantees, skipping the full loop. |
| `/gsd-fast` | Execute a trivial task inline — no subagents, no planning overhead. |
| `/gsd-autonomous` | Run all remaining phases end to end (discuss → plan → execute per phase) without stopping. |
| `/gsd-progress` / `/gsd-stats` | Fast, low-effort reads of where the project stands. |
| `/gsd-workspace --new --name X --repos a,b` | Isolated multi-repo (or same-repo worktree) workspace with its own `.planning/`. |
| `/gsd-map-codebase` | Analyze an existing codebase into `.planning/codebase/` docs. |
| `/gsd-code-review` | Review changed files for bugs, security issues, and quality problems. |
| `/gsd-secure-phase` | Retroactively verify threat mitigations for a completed phase. |
| `/gsd-debug` | Systematic debugging with state that persists across context resets. |
| `/gsd-undo` | Safe git revert of a phase or plan's commits, dependency-aware. |
| `/gsd-workstreams` | Manage parallel workstreams — list, create, switch, status, complete. |
| `/gsd-pause-work` / `/gsd-resume-work` | Hand off mid-phase and pick it back up with full context restored. |

Full syntax, flags, and examples for every command:
[docs/COMMANDS.md](https://github.com/open-gsd/gsd-core/blob/main/docs/COMMANDS.md).

## 7. Why the loop, not just prompting

Most AI-coding setups degrade at scale for three reasons GSD Core targets
directly:

- **Context bloat** silently degrades output quality → heavy work runs in
  fresh-context subagents instead of your accumulating main session.
- **No shared memory between sessions** → structured artifacts (`STATE.md`,
  `CONTEXT.md`, `PLAN.md`, `VERIFICATION.md`) persist in `.planning/` and
  survive session boundaries and restarts.
- **Nothing verifies the code actually works** → the Verify step checks
  requirement coverage and decision coverage, not just "did execution error
  out."

See [Context engineering](https://github.com/open-gsd/gsd-core/blob/main/docs/explanation/context-engineering.md)
and [The phase loop](https://github.com/open-gsd/gsd-core/blob/main/docs/explanation/the-phase-loop.md)
for the full reasoning.

## 8. Learning paths

- **New project, first time using GSD:**
  [Your first project](https://github.com/open-gsd/gsd-core/blob/main/docs/tutorials/your-first-project.md)
- **Existing codebase:**
  [Onboarding an existing codebase](https://github.com/open-gsd/gsd-core/blob/main/docs/tutorials/onboarding-an-existing-codebase.md)
- **Task-focused how-tos:** install, plan a phase, verify and ship, recover
  and troubleshoot — [docs/README.md](https://github.com/open-gsd/gsd-core/blob/main/docs/README.md#how-to-guides)
- **Reference:** [Commands](https://github.com/open-gsd/gsd-core/blob/main/docs/COMMANDS.md) ·
  [Configuration](https://github.com/open-gsd/gsd-core/blob/main/docs/CONFIGURATION.md) ·
  [CLI tools](https://github.com/open-gsd/gsd-core/blob/main/docs/CLI-TOOLS.md)

## 9. Troubleshooting

See [Recover and troubleshoot](https://github.com/open-gsd/gsd-core/blob/main/docs/how-to/recover-and-troubleshoot.md)
for stuck phases, corrupted `.planning/` state, and similar recovery steps.

## Links

- Repo: https://github.com/open-gsd/gsd-core
- Docs index: https://github.com/open-gsd/gsd-core/blob/main/docs/README.md
- Architecture: https://github.com/open-gsd/gsd-core/blob/main/docs/ARCHITECTURE.md
- Discord: https://discord.gg/mYgfVNfA2r
- License: MIT
