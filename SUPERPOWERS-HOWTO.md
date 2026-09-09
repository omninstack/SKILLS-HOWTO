# How to Use Superpowers

Source: https://github.com/obra/superpowers

Superpowers is a software development methodology for coding agents, packaged
as a set of composable **skills** plus a bootstrap instruction that makes sure
your agent actually uses them. It turns "write me a feature" into a disciplined
pipeline: clarify intent → design → plan → implement with TDD → review → merge.

## 1. Install it for your coding agent

Superpowers is distributed per-harness — if you use more than one agent (Claude
Code, Cursor, Codex, Gemini CLI, etc.), install it separately in each.

| Harness | Install command |
|---|---|
| Claude Code (official marketplace) | `/plugin install superpowers@claude-plugins-official` |
| Claude Code (Superpowers marketplace) | `/plugin marketplace add obra/superpowers-marketplace` then `/plugin install superpowers@superpowers-marketplace` |
| Antigravity | `agy plugin install https://github.com/obra/superpowers` |
| Codex App | Sidebar → Plugins → find "Superpowers" → install |
| Codex CLI | `/plugins` → search `superpowers` → Install Plugin |
| Cursor | In Agent chat: `/add-plugin superpowers` |
| Devin CLI | `devin plugins install obra/superpowers` |
| Factory Droid | `droid plugin marketplace add https://github.com/obra/superpowers` then `droid plugin install superpowers@superpowers` |
| Gemini CLI | `gemini extensions install https://github.com/obra/superpowers` |
| GitHub Copilot CLI | `copilot plugin marketplace add obra/superpowers-marketplace` then `copilot plugin install superpowers@superpowers-marketplace` |
| Grok Build CLI | `grok plugin install superpowers@xai-official --trust` |
| Kimi Code | `/plugins install https://github.com/obra/superpowers` |
| OpenCode | Ask it to fetch/follow `https://raw.githubusercontent.com/obra/superpowers/refs/heads/main/.opencode/INSTALL.md` |
| Pi | `pi install git:github.com/obra/superpowers` |
| Hermes Agent | `hermes plugins install obra/superpowers --enable` |

After installing, updates are largely automatic (mechanism varies slightly by
harness — see the repo README's "Updating" section).

## 2. You don't have to invoke anything manually

Superpowers works via **automatic skill triggering**. A `using-superpowers`
bootstrap loads at session start (and again after context compaction) and
tells the agent: before responding to *any* task — including just asking a
clarifying question — check whether a relevant skill exists, and if one does,
you must use it. So once installed, just talk to your agent normally:

> "Let's add a rate limiter to the API"

The agent should recognize this as creative/feature work and route into the
workflow below on its own, rather than jumping straight to code.

## 3. The basic workflow

Superpowers strings together skills into a pipeline. Each stage is itself a
skill that "activates" (gets invoked) when its trigger condition is met:

1. **brainstorming** — Activates before any code is written. Asks questions to
   refine a rough idea, explores alternatives, and presents the design back to
   you in digestible chunks for approval. Produces a design document.
2. **using-git-worktrees** — Activates once the design is approved. Creates an
   isolated workspace on a new branch and verifies a clean test baseline
   before work starts.
3. **writing-plans** — Activates with an approved design. Breaks the work into
   small (2–5 minute) tasks, each with exact file paths, the code to write,
   and how to verify it.
4. **subagent-driven-development** (fast iteration) or **executing-plans**
   (batch mode with human checkpoints) — Activates with a plan. Dispatches a
   fresh subagent per task with two-stage review: spec compliance, then code
   quality.
5. **test-driven-development** — Activates during implementation. Enforces
   RED → GREEN → REFACTOR: write a failing test, watch it fail, write the
   minimal code to pass, watch it pass, commit. Code written before its test
   gets deleted.
6. **requesting-code-review** — Activates between tasks. Reviews the diff
   against the plan and reports issues by severity; critical issues block
   progress.
7. **finishing-a-development-branch** — Activates once all tasks are done.
   Verifies tests pass and presents options: merge, open a PR, keep the
   branch, or discard it.

This is treated as a **mandatory workflow, not a suggestion** — the agent is
instructed to check for a matching skill before every task.

## 4. Other skills worth knowing about

Beyond the main pipeline:

- **systematic-debugging** — 4-phase root-cause process for any bug/test
  failure/unexpected behavior, used instead of guessing at fixes.
- **verification-before-completion** — Forces the agent to actually run
  verification commands and check output before claiming something is fixed
  or complete.
- **dispatching-parallel-agents** — For 2+ independent tasks with no shared
  state, runs them concurrently instead of sequentially.
- **receiving-code-review** — Governs how the agent responds to review
  feedback (verify, don't just comply).
- **writing-skills** — The meta-skill for authoring new skills, including how
  to test them.

## 5. Typical session shape

```
You:    "I want to add CSV export to the reports page"
Agent:  [invokes brainstorming] asks clarifying questions, proposes a design
You:    approve / adjust the design
Agent:  [using-git-worktrees] sets up an isolated branch
Agent:  [writing-plans] produces a step-by-step task plan
Agent:  [subagent-driven-development] works through tasks one at a time,
        writing a failing test, then code, then passing test, per task
Agent:  [requesting-code-review] flags anything concerning between tasks
Agent:  [finishing-a-development-branch] asks: merge, PR, keep, or discard?
```

You mostly just answer questions and approve/adjust at checkpoints — the
agent drives the mechanics.

## 6. Contributing / customizing

- The project doesn't generally accept new skills as contributions, and any
  change to a skill must work across every supported harness.
- To modify or add a skill locally, follow the process in
  `skills/writing-skills/SKILL.md` in the repo.
- Contribution flow: fork → branch off `dev` → follow `writing-skills` →
  submit a PR with the template filled in.

## 7. Telemetry note

The brainstorming skill's optional visual companion loads a logo from Prime
Radiant's website, which reports only the Superpowers version in use — no
project, prompt, or code details. It's opt-out via the environment variable
`SUPERPOWERS_DISABLE_TELEMETRY` (or Claude Code's own `DISABLE_TELEMETRY` /
`CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC`).

## Links

- Repo: https://github.com/obra/superpowers
- Release announcement: https://blog.fsck.com/2025/10/09/superpowers/
- Issues: https://github.com/obra/superpowers/issues
- Discord: https://discord.gg/35wsABTejz
