# How to Use Harness Skills

Source: https://github.com/harness/harness-skills

Harness Skills is a set of Claude Code (and other AI coding tool) skills for
the [Harness.io](https://harness.io) CI/CD platform: generate pipeline YAML,
manage resources, debug failures, analyze costs, and more, from natural
language. It's built as a **workflow system**, not a folder of prompts —
shared top-level instructions (`AGENTS.md`, imported by `CLAUDE.md` for
Claude Code, plus `.github/copilot-instructions.md`) establish common
behavior, while individual skills specialize in creation, debugging,
governance, and reporting.

## 1. Prerequisite: Harness MCP v2 Server

Most skills depend on the [Harness MCP v2 server](https://github.com/harness/mcp-server)
for Harness API access. Set it up before relying on any MCP-powered skill.

## 2. Setup by tool

### Claude Code

```bash
git clone https://github.com/harness/harness-skills.git
cd harness-skills
claude
```

Skills auto-discover from `skills/*/SKILL.md`; project rules load via
`CLAUDE.md → @AGENTS.md`. Add the MCP server to `~/.claude/settings.json`:

```json
{
  "mcpServers": {
    "harness-mcp-v2": {
      "command": "npx",
      "args": ["-y", "harness-mcp-v2"],
      "env": { "HARNESS_API_KEY": "<your-api-key>" }
    }
  }
}
```

Then invoke a skill by name:

```
/create-pipeline
Create a CI pipeline for a Node.js app that builds, runs tests,
and pushes a Docker image to ECR
```

### Cursor

1. Clone the repo into your project (or as a reference workspace).
2. `.cursor/rules/harness.mdc` is auto-loaded as a project rule.
3. Configure the MCP server in `~/.cursor/mcp.json` (same JSON shape as above).
4. Reference a skill directly with `@file`:
   `@harness-skills/skills/create-pipeline/SKILL.md`

### OpenAI Codex

1. Clone the repo; `AGENTS.md` at the root is read automatically as system
   instructions.
2. Configure the MCP server in your Codex MCP config.
3. Reference a skill file as context: *"Using the instructions in
   harness-skills/skills/debug-pipeline/SKILL.md, diagnose why my deploy
   pipeline failed."*

### GitHub Copilot

1. Clone the repo; `.github/copilot-instructions.md` is read automatically
   on both GitHub.com and VS Code.
2. In VS Code, configure the MCP server in `.vscode/mcp.json`.
3. Reference a skill with `#file:harness-skills/skills/create-pipeline/SKILL.md`
   in Copilot Chat, or attach skill files as knowledge-base references in
   org settings for GitHub.com Copilot.

### Windsurf / other AI editors

Skills are plain Markdown with YAML frontmatter, so any tool that supports
(1) system instructions via `AGENTS.md`, (2) MCP servers, and (3) file
context can use them — just point it at the relevant `skills/*/SKILL.md`.

## 3. Operating model — how skills behave

Every skill that touches Harness resources follows the same control flow:

1. **Establish scope first** — confirm account/org/project context before
   listing, creating, updating, or deleting anything.
2. **Verify dependencies before generating dependents** — never reference a
   connector, secret, environment, infrastructure, or template that hasn't
   been confirmed to exist.
3. **Discover schema before writing payloads** — use `harness_describe` and
   API validation feedback instead of guessing field names or shape.

These are documented as repo-level playbooks:
[scope-establishment.md](https://github.com/harness/harness-skills/blob/main/references/scope-establishment.md),
[dependency-check-playbook.md](https://github.com/harness/harness-skills/blob/main/references/dependency-check-playbook.md),
[schema-validation-loop.md](https://github.com/harness/harness-skills/blob/main/references/schema-validation-loop.md).

## 4. Workflow modes

| Mode | Representative skills | Use when |
|---|---|---|
| Create and scaffold | `/create-pipeline`, `/create-service`, `/create-connector`, `/create-template` | Defining or generating new Harness resources. |
| Run and debug | `/run-pipeline`, `/debug-pipeline`, `/migrate-pipeline`, `/manage-delegates` | Resources exist; you need to execute, diagnose, or repair. |
| Govern and secure | `/manage-roles`, `/manage-users`, `/create-policy`, `/security-report`, `/audit-report` | RBAC, policy, compliance, security with blast-radius awareness. |
| Analyze and report | `/dora-metrics`, `/analyze-costs`, `/scorecard-review`, `/template-usage` | Structured reports, summaries, recommendations, adoption analysis. |

## 5. End-to-end workflows

**New microservice setup** (run in order):

```
/create-connector → /create-secret → /create-service →
/create-environment → /create-infrastructure → /create-pipeline → /create-trigger
```

**Debug a failed deployment:**

```
/run-pipeline          # identify the latest execution / reproduce
/debug-pipeline         # classify the failure, inspect root cause
/template-usage         # check if a shared template propagated the issue
/manage-delegates        # check delegate capacity/connectivity if that's the cause
```

## 6. Skill catalog

**Pipeline & template creation**

| Skill | Description |
|---|---|
| `/create-pipeline` | Generate v0 Pipeline YAML (CI, CD, approvals, matrix strategies) |
| `/create-pipeline-v1` | Generate v1 simplified Pipeline YAML — *alpha* |
| `/create-template` | Reusable Step, Stage, Pipeline, or StepGroup templates |
| `/create-trigger` | Webhook, scheduled, and artifact triggers |
| `/create-agent-template` | AI-powered agent templates — *alpha* |

**Resource management**

| Skill | Description |
|---|---|
| `/create-service` | Service definitions (K8s, Helm, ECS, Lambda) |
| `/create-environment` | Environment definitions with overrides |
| `/create-infrastructure` | Infrastructure definitions |
| `/create-connector` | Connectors (Git, cloud, registries, clusters) |
| `/create-secret` | Secrets (text, file, SSH, WinRM) |

**Database operations (MCP)**

| Skill | Description |
|---|---|
| `/dbops-changeset` | Generate/refine/review/execute Liquibase changesets via Harness DBOPS |

**Access control & feature flags (MCP)**

| Skill | Description |
|---|---|
| `/manage-users` | Users, user groups, service accounts |
| `/manage-roles` | Role assignments and RBAC |
| `/manage-feature-flags` | Create, list, toggle, delete feature flags |

**Operations & debugging (MCP)**

| Skill | Description |
|---|---|
| `/run-pipeline` | Execute pipelines, monitor progress, handle approvals |
| `/debug-pipeline` | Analyze execution failures, diagnose root causes |
| `/migrate-pipeline` | Convert pipelines v0 → v1 |
| `/optimize-pipeline` | Speed: parallel testing, caching, monorepo builds |
| `/deployment-readiness` | Pre-deployment checks, environment drift, canary decisions |
| `/incident-response` | Deployment-incident correlation, blast radius, postmortems |
| `/pr-analysis` | PR pipeline impact, security review, PR-to-production tracking |
| `/template-usage` | Track template dependencies and adoption |
| `/manage-delegates` | Monitor delegate health, manage tokens |

**Security**

| Skill | Description |
|---|---|
| `/configure-repo-scan` | SAST/SCA scans on repos |
| `/configure-secret-scan` | Detect exposed secrets/credentials |
| `/configure-container-scan` | Scan container images for vulnerabilities |
| `/configure-dast-scan` | DAST on running applications |
| `/exempt-vuln` | Vulnerability waivers (target/project/pipeline scope) |
| `/approve-exempt` | Approve pending STO exemptions, elevate scope if needed |
| `/generate-slsa` / `/enforce-slsa` | Generate/verify SLSA provenance, enforce OPA policies |
| `/create-sbom` / `/enforce-sbom` | Generate/verify SBOM attestations, enforce OPA policies |
| `/sign-artifact` / `/verify-sign` | Sign and verify artifact integrity/authenticity |

**Platform intelligence (MCP)**

| Skill | Description |
|---|---|
| `/analyze-costs` | Cloud cost analysis and optimization (CCM) |
| `/security-report` | Vulnerability reports, SBOMs, compliance (SCS/STO) |
| `/dora-metrics` | DORA metrics and engineering performance (SEI) |
| `/sei-analytics` | Sprint analytics, investment allocation, capacity forecasting |
| `/manage-slos` | SLO definition, error budgets, incident detection, runbooks (SRM) |
| `/ai-operations` | Predictive failure analysis, alert correlation (AIDA) |
| `/gitops-status` | GitOps application health and sync status |
| `/chaos-experiment` / `/chaos-dr-test` | Chaos experiments and DR test pipelines |
| `/scorecard-review` | Service maturity scorecards (IDP) |
| `/manage-idp` | Service catalog, self-service workflows, onboarding (IDP) |
| `/manage-iacm` | Terraform workspaces, drift detection, cost estimation (IaCM) |
| `/manage-cde` | Cloud development environments and workspace templates |
| `/manage-artifacts` | Artifact registry, scanning, replication (AR) |
| `/manage-supply-chain` | SBOM generation, signing, supply chain policies (SSCA) |
| `/audit-report` | Audit trails and compliance reports |
| `/create-policy` | OPA governance policies for supply chain security |

## 7. Repo structure

```
harness-skills/
├── skills/<name>/SKILL.md + references/   # skill definitions
├── references/                            # shared repo-level playbooks
├── templates/                             # shared output templates (e.g. operation-summary.md)
├── scripts/validate-skills.sh             # structural validation
├── examples/                              # v0/v1 pipeline, template, trigger, service... examples
├── .cursor/rules/harness.mdc              # auto-loaded by Cursor
├── .github/copilot-instructions.md
├── AGENTS.md                              # canonical project rules
└── CLAUDE.md                              # imports AGENTS.md for Claude Code
```

Use root-level `references/`/`templates/` for behavior that should stay
consistent across many skills; use per-skill `references/`/`templates/` for
domain-specific content that shouldn't be imported broadly.

## 8. Skill anatomy (for writing your own)

Each skill is a directory under `skills/` with a `SKILL.md`:

```yaml
---
name: my-skill
description: >-
  What it does, when to use it, when not to, and trigger phrases.
  Keep under 1024 characters.
metadata:
  author: Harness
  version: 1.0.0
  mcp-server: harness-mcp-v2
license: Apache-2.0
compatibility: Requires Harness MCP v2 server (harness-mcp-v2)
---

# My Skill
One or two sentences on operating mode and expected outcome.

## Instructions
Phase-based steps for Claude to follow.

## Examples
Invocation examples and brief worked examples for complex skills.

## Performance Notes
Validation checks, tradeoffs, speed/accuracy guidance.

## Troubleshooting
Common errors, recovery steps, expected fallbacks.
```

For complex skills, put extra detail in a `references/` or `templates/`
subfolder rather than bloating `SKILL.md`. See
[CONTRIBUTING.md](https://github.com/harness/harness-skills/blob/main/CONTRIBUTING.md)
for authoring standards and validation rules.

## 9. MCP tools reference

MCP-powered skills use the Harness MCP v2 server's 10 generic tools,
dispatched by `resource_type`:

| Tool | Purpose |
|---|---|
| `harness_list` | List resources |
| `harness_get` | Get resource details |
| `harness_create` | Create a resource |
| `harness_update` | Update a resource |
| `harness_delete` | Delete a resource |
| `harness_execute` | Execute an action |
| `harness_search` | Search across resources |
| `harness_describe` | Get resource schema |
| `harness_diagnose` | Diagnose issues |
| `harness_status` | Check system status |

## Links

- Repo: https://github.com/harness/harness-skills
- Harness MCP v2 server: https://github.com/harness/mcp-server
- v0 Pipeline/Template/Trigger schema: https://github.com/harness/harness-schema/tree/main/v0
- v1 Pipeline spec: https://github.com/thisrohangupta/spec
- License: Apache 2.0
