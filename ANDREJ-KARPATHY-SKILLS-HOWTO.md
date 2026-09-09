# How to Use Andrej Karpathy Skills

Source: https://github.com/multica-ai/andrej-karpathy-skills
(a mirror/fork of [forrestchang/andrej-karpathy-skills](https://github.com/forrestchang/andrej-karpathy-skills))

A single `CLAUDE.md` file (also shipped as a Claude Code skill and a Cursor
rule) that improves agent coding behavior, derived from
[Andrej Karpathy's observations](https://x.com/karpathy/status/2015883857489522876)
on common LLM coding pitfalls. It's already in this environment as the
`andrej-karpathy-skills:karpathy-guidelines` skill.

## The problem it addresses

From Karpathy's post:

> "The models make wrong assumptions on your behalf and just run along with
> them without checking... They really like to overcomplicate code and APIs,
> bloat abstractions... They still sometimes change/remove comments and code
> they don't sufficiently understand as side effects, even if orthogonal to
> the task."

## The four principles

| Principle | Addresses |
|---|---|
| **Think Before Coding** | Wrong assumptions, hidden confusion, missing tradeoffs |
| **Simplicity First** | Overcomplication, bloated abstractions |
| **Surgical Changes** | Orthogonal edits, touching code you shouldn't |
| **Goal-Driven Execution** | Leverage through tests-first, verifiable success criteria |

### 1. Think Before Coding

Don't assume, don't hide confusion, surface tradeoffs.

- State assumptions explicitly — ask rather than silently guess.
- Present multiple interpretations when a request is ambiguous.
- Push back when a simpler approach exists.
- Stop and name what's unclear when genuinely confused.

### 2. Simplicity First

Minimum code that solves the problem, nothing speculative.

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility"/"configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If 200 lines could be 50, rewrite it.

**The test:** would a senior engineer call this overcomplicated? If yes, simplify.

### 3. Surgical Changes

Touch only what you must; clean up only your own mess.

- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style even if you'd do it differently.
- Mention unrelated dead code you notice — don't delete it.
- Do remove imports/variables/functions your *own* change made unused; don't
  remove pre-existing dead code unless asked.

**The test:** every changed line should trace directly to the user's request.

### 4. Goal-Driven Execution

Define success criteria, loop until verified — transform imperative tasks
into verifiable goals:

| Instead of... | Transform to... |
|---|---|
| "Add validation" | "Write tests for invalid inputs, then make them pass" |
| "Fix the bug" | "Write a test that reproduces it, then make it pass" |
| "Refactor X" | "Ensure tests pass before and after" |

For multi-step tasks, state a brief plan:

```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let the agent loop independently; weak criteria
("make it work") force constant clarification.

## Install

**Option A — Claude Code plugin (recommended, applies across all projects):**

```
/plugin marketplace add forrestchang/andrej-karpathy-skills
/plugin install andrej-karpathy-skills@karpathy-skills
```

(This is the canonical marketplace the multica-ai mirror also publishes
through — same plugin, same `karpathy-guidelines` skill.)

**Option B — CLAUDE.md, per project:**

New project:
```bash
curl -o CLAUDE.md https://raw.githubusercontent.com/multica-ai/andrej-karpathy-skills/main/CLAUDE.md
```

Existing project (append):
```bash
echo "" >> CLAUDE.md
curl https://raw.githubusercontent.com/multica-ai/andrej-karpathy-skills/main/CLAUDE.md >> CLAUDE.md
```

**Using with Cursor:** the repo ships a Cursor project rule at
`.cursor/rules/karpathy-guidelines.mdc`, applying the same guidelines when
the project is opened in Cursor. See `CURSOR.md` in the repo for setup and
how it relates to the Claude Code version.

## How it actually gets used

Once installed, you don't invoke anything explicitly — it's behavioral
guidance that shapes how the agent responds to every coding request. In
Claude Code specifically it surfaces as the `karpathy-guidelines` skill,
triggered "when writing, reviewing, or refactoring code to avoid
overcomplication, make surgical changes, surface assumptions, and define
verifiable success criteria."

Example of the shift it's meant to produce — request: *"Add a feature to
export user data."*

- **Without it:** the agent silently assumes scope (export all users?),
  format, fields, and file location, then writes a few dozen lines that may
  not match what you meant.
- **With it:** the agent asks about scope (privacy implications), format
  (download vs. background job vs. API), which fields, and expected volume
  — before writing anything — and proposes the simplest approach first.

More worked examples (assumption-surfacing, over-abstraction avoided,
surgical vs. drive-by diffs, imperative-to-declarative goal framing) are in
[EXAMPLES.md](https://github.com/multica-ai/andrej-karpathy-skills/blob/main/EXAMPLES.md).

## Customization

Designed to merge with project-specific instructions — append to your
existing `CLAUDE.md`, or add a section like:

```markdown
## Project-Specific Guidelines

- Use TypeScript strict mode
- All API endpoints must have tests
- Follow the existing error handling patterns in `src/utils/errors.ts`
```

## Tradeoff to know about

These guidelines bias toward **caution over speed**. For trivial tasks
(typo fixes, obvious one-liners), use judgment — not every change needs the
full rigor. The goal is reducing costly mistakes on non-trivial work, not
slowing down simple ones.

## How to know it's working

- Fewer unnecessary changes in diffs — only requested changes appear.
- Fewer rewrites due to overcomplication — code is simple the first time.
- Clarifying questions come before implementation, not after mistakes.
- Clean, minimal PRs — no drive-by refactoring or "improvements."

## Links

- Repo (mirror): https://github.com/multica-ai/andrej-karpathy-skills
- Original: https://github.com/forrestchang/andrej-karpathy-skills
- Source observation thread: https://x.com/karpathy/status/2015883857489522876
- License: MIT
