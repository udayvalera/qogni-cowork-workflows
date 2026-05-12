# QCW Workflow Generator

> **Generate reusable, multi-agent workflow definitions for the Qogni Cowork marketplace.**

Workflows run one step at a time in harness tools — Claude Code, Gemini CLI, OpenCode, Antigravity, or Codex. Each step produces an artifact or awaits manual completion, with optional custom agent personas driving each step.

---

## Table of Contents

- [Quick Start](#quick-start)
- [Installation](#installation)
- [Concepts](#concepts)
- [Repository Layout](#repository-layout)
- [Pre-Built Workflows](#pre-built-workflows)
- [Step-by-Step Guide](#step-by-step-guide)
- [Examples](#examples)
- [Schema Reference](#schema-reference)
- [Troubleshooting](#troubleshooting)

---

## Quick Start

```bash
# 1. Clone or navigate to the skill store
cd skill-store

# 2. Copy the skill to your tool's skill directory (see Installation)
cp -r .agent/skills/qcw-workflow-generator ~/.gemini/skills/

# 3. Ask your agent to generate a workflow
> "Create a Qogni workflow for onboarding new developers to a codebase"
```

The agent reads the skill, follows the schema, and produces a valid `workflow-bundle.json` you can import into Qogni Cowork.

---

## Installation

### Antigravity

Antigravity discovers skills from two locations:

```bash
# Global (all projects)
cp -r .agent/skills/qcw-workflow-generator ~/.gemini/antigravity/skills/

# Project-level (this workspace only) — already in place
# .agent/skills/qcw-workflow-generator/SKILL.md
```

### Gemini CLI

```bash
# Global (all projects)
mkdir -p ~/.gemini/skills
cp -r .agent/skills/qcw-workflow-generator ~/.gemini/skills/

# Project-level
mkdir -p .gemini/skills
cp -r .agent/skills/qcw-workflow-generator .gemini/skills/

# Verify
gemini> /skills list
# Should show: qcw-workflow-generator
```

### Claude Code

Claude Code uses `~/.claude/skills/` for global skills:

```bash
# Global
mkdir -p ~/.claude/skills
cp -r .agent/skills/qcw-workflow-generator ~/.claude/skills/

# Project-level — add to CLAUDE.md
echo '@.agent/skills/qcw-workflow-generator/SKILL.md' >> CLAUDE.md
```

### OpenCode

OpenCode discovers skills from `~/.config/opencode/skills/` or `.opencode/skills/`:

```bash
# Global
mkdir -p ~/.config/opencode/skills
cp -r .agent/skills/qcw-workflow-generator ~/.config/opencode/skills/

# Project-level
mkdir -p .opencode/skills
cp -r .agent/skills/qcw-workflow-generator .opencode/skills/
```

### Summary Table

| Tool | Global Path | Project Path |
|------|------------|--------------|
| **Antigravity** | `~/.gemini/antigravity/skills/` | `.agent/skills/` |
| **Gemini CLI** | `~/.gemini/skills/` | `.gemini/skills/` |
| **Claude Code** | `~/.claude/skills/` | Reference in `CLAUDE.md` |
| **OpenCode** | `~/.config/opencode/skills/` | `.opencode/skills/` |

---

## Concepts

### Workflow Formats

| Format | When to Use |
|--------|------------|
| `workflow.yaml` | Simple workflows, no custom agents |
| `workflow-bundle.json` | Workflows with custom agent personas (recommended) |

### Key Components

**Steps** — Sequential actions. Each step either:
- `artifact_generated` — produces a file (e.g., `docs/spec.md`)
- `manual_completion` — waits for human approval

**Custom Agents** — Specialized personas with three artifact types:
- **Soul** (required) — identity, principles, deliverable format
- **Template** (recommended) — structured output skeleton
- **Reference** (optional) — domain rules and heuristics

**Prompts** — Instructions for each step. Must include one default prompt (`executor_type: null`). Can optionally include executor-specific prompts for `CLAUDE_CODE`, `GEMINI`, or `CODEX`.

---

## Repository Layout

```
qogni-wf-skill/
├── list.json                          # Marketplace catalog (register all workflows here)
├── instruction.md                     # Specification reference
├── artifacts-docs/                    # Implementation plan, task tracker, walkthrough
│
├── landing-page-sprint/               # Pre-built workflow bundle
│   └── workflow-bundle.json
├── api-first-design/
│   └── workflow-bundle.json
├── code-review-sprint/
│   └── workflow-bundle.json
├── brownfield-onboarding/
│   └── workflow-bundle.json
├── bug-triage-fix/
│   └── workflow-bundle.json
│
└── landing-page-sprint-workflow/      # Reference samples (individual files)
    └── landing-page-sprint-workflow/
        ├── landing-page-sprint.workflow.json
        ├── content-creator.agent.json
        ├── frontend-developer.agent.json
        ├── growth-hacker.agent.json
        └── ui-designer.agent.json

.agent/skills/qcw-workflow-generator/  # The skill itself
├── SKILL.md                           # Main skill instructions
└── agent-design-guide.md              # Agent authoring reference
```

---

## Pre-Built Workflows

### 1. Landing Page Sprint
**Category:** `web-development` · **Steps:** 6 · **Agents:** 4

A one-day workflow to create, build, optimize, and ship a conversion-focused landing page.

| Step | Agent | Output |
|------|-------|--------|
| Write Landing Page Copy | content-creator | `landing-page-sprint/copy.md` |
| Create UI Design Specs | ui-designer | `landing-page-sprint/design-spec.md` |
| Build Initial Landing Page | frontend-developer | `index.html` |
| Conversion Review | growth-hacker | `landing-page-sprint/growth-review.md` |
| Apply Growth Feedback | frontend-developer | `landing-page-sprint/final-checklist.md` |
| Ship Handoff | frontend-developer | `landing-page-sprint/ship-notes.md` |

### 2. API-First Design
**Category:** `api-development` · **Steps:** 6 · **Agents:** 4

Design, review, implement, test, and document a RESTful API.

| Step | Agent | Output |
|------|-------|--------|
| Gather API Requirements | api-architect | `api-design/requirements.md` |
| Design OpenAPI Specification | api-architect | `api-design/openapi.yaml` |
| API Design Review | api-reviewer | `api-design/review-report.md` |
| Generate Server Stubs | backend-developer | `api-design/implementation-notes.md` |
| Write Integration Tests | test-engineer | `api-design/test-plan.md` |
| Documentation Handoff | api-architect | `api-design/docs-handoff.md` |

### 3. Code Review Sprint
**Category:** `review` · **Steps:** 6 · **Agents:** 4

Multi-perspective code review: static analysis, security, performance, and consolidated reporting.

| Step | Agent | Output |
|------|-------|--------|
| Select Review Scope | review-coordinator | `code-review/scope.md` |
| Static Analysis | code-analyzer | `code-review/static-analysis.md` |
| Security Audit | security-reviewer | `code-review/security-audit.md` |
| Performance Review | performance-analyst | `code-review/performance-review.md` |
| Consolidated Report | review-coordinator | `code-review/consolidated-report.md` |
| Approval Gate | review-coordinator | Manual approval |

### 4. Brownfield Onboarding
**Category:** `brownfield` · **Steps:** 5 · **Agents:** 3

Understand and document an existing codebase for new developer onboarding.

| Step | Agent | Output |
|------|-------|--------|
| Repository Scan | codebase-archaeologist | `onboarding/repo-scan.md` |
| Architecture Map | architecture-mapper | `onboarding/architecture.md` |
| Dependency Audit | dependency-auditor | `onboarding/dependency-audit.md` |
| Developer Onboarding Guide | codebase-archaeologist | `onboarding/getting-started.md` |
| Quick-Start Verification | codebase-archaeologist | Manual approval |

### 5. Bug Triage & Fix
**Category:** `ops` · **Steps:** 5 · **Agents:** 3

Structured bug investigation from intake through fix, testing, and approval.

| Step | Agent | Output |
|------|-------|--------|
| Bug Intake | triage-coordinator | `bug-fix/intake.md` |
| Root Cause Analysis | debugger | `bug-fix/root-cause.md` |
| Fix Implementation | fix-developer | `bug-fix/fix-summary.md` |
| Test Coverage | debugger | `bug-fix/test-report.md` |
| Fix Review | triage-coordinator | Manual approval |

---

## Step-by-Step Guide

### Creating a New Workflow

#### Step 1: Plan Your Workflow

Decide on:
- **What problem does this workflow solve?**
- **What steps are needed?** (list them in order)
- **What agents are needed?** (who does what)
- **What artifacts does each step produce?**

#### Step 2: Define Your Agents

For each unique role, create an agent with:

```json
{
  "version": 1,
  "name": "My Agent",
  "slug": "my-agent",
  "description": "What this agent does in one sentence.",
  "artifacts": [
    {
      "kind": "soul",
      "title": "SOUL",
      "content": "# My Agent\n\nYou are a [role]...\n\nPrinciples:\n- ...\n\nDefault deliverable format:\n- ..."
    }
  ]
}
```

#### Step 3: Write Your Steps

For each step, define:

```json
{
  "name": "Step Name",
  "description": "What this step does.",
  "transition_type": "artifact_generated",
  "artifact_pattern": "output/file.md",
  "require_approval": false,
  "requires_user_input": false,
  "user_input_question": null,
  "agent_slug": "my-agent",
  "prompts": [{
    "executor_type": null,
    "prompt": "Activate My Agent.\n\n[Context]\n[Task]\n[Output path]"
  }],
  "skills": []
}
```

#### Step 4: Assemble the Bundle

Wrap steps and agents into `workflow-bundle.json`:

```json
{
  "kind": "qogni.workflow_bundle",
  "version": 1,
  "workflow": {
    "version": 1,
    "name": "My Workflow",
    "description": "What this workflow accomplishes.",
    "steps": [ /* your steps */ ]
  },
  "resources": {
    "custom_agents": [ /* your agents */ ]
  }
}
```

#### Step 5: Register in the Marketplace

Add an entry to `list.json`:

```json
{
  "id": "my-workflow",
  "directory": "my-workflow",
  "definition_file": "workflow-bundle.json",
  "category": "agile",
  "tags": ["planning", "sprint"]
}
```

#### Step 6: Validate

```bash
# Check JSON syntax
python3 -m json.tool my-workflow/workflow-bundle.json > /dev/null

# Verify all agent_slugs match, souls exist, default prompts present
# Use the validation script from the verification section
```

---

## Examples

### Example 1: Generating a Workflow via Prompt

Ask your agent:

```
Using the qcw-workflow-generator skill, create a Qogni workflow bundle 
for a "Documentation Sprint" that:
1. Scans the codebase for undocumented functions
2. Generates JSDoc comments
3. Creates a README
4. Reviews everything for accuracy
```

The agent will produce a complete `workflow-bundle.json` with custom agents and register it in `list.json`.

### Example 2: Simple 2-Step Workflow (YAML)

```yaml
version: 1
name: "Quick Spec Review"
description: "Review a spec and get approval."
steps:
  - name: "Draft Review"
    description: "Analyze the spec and write review notes."
    transition_type: artifact_generated
    artifact_pattern: "docs/review.md"
    require_approval: false
    requires_user_input: true
    user_input_question: "Which spec file should be reviewed?"
    agent_slug: null
    prompts:
      - executor_type: null
        prompt: |
          Review the specified file and save findings to docs/review.md.
          Include: summary, issues found, recommendations.
    skills: []

  - name: "Approve Review"
    description: "Human approves the review before sharing."
    transition_type: manual_completion
    artifact_pattern: null
    require_approval: true
    requires_user_input: false
    user_input_question: null
    agent_slug: null
    prompts:
      - executor_type: null
        prompt: |
          Present the review from docs/review.md to the user.
          Wait for approval before marking complete.
    skills: []
```

### Example 3: Step Execution Flow (Bug Triage)

Here's how the **Bug Triage & Fix** workflow executes step by step:

```
┌─────────────────────────────────────────────────────┐
│ Step 1: Bug Intake                                  │
│ Agent: triage-coordinator                           │
│ User Input: "Describe the bug..."                   │
│ Output: bug-fix/intake.md                           │
├─────────────────────────────────────────────────────┤
│ Step 2: Root Cause Analysis                         │
│ Agent: debugger                                     │
│ Reads: bug-fix/intake.md                            │
│ Output: bug-fix/root-cause.md                       │
├─────────────────────────────────────────────────────┤
│ Step 3: Fix Implementation                          │
│ Agent: fix-developer                                │
│ Reads: bug-fix/root-cause.md                        │
│ Output: bug-fix/fix-summary.md + code changes       │
├─────────────────────────────────────────────────────┤
│ Step 4: Test Coverage                               │
│ Agent: debugger                                     │
│ Reads: intake.md + root-cause.md + fix-summary.md   │
│ Output: bug-fix/test-report.md + test files          │
├─────────────────────────────────────────────────────┤
│ Step 5: Fix Review (APPROVAL GATE)                  │
│ Agent: triage-coordinator                           │
│ Reads: all bug-fix/*.md                             │
│ Action: Human reviews and approves                   │
└─────────────────────────────────────────────────────┘
```

### Example 4: Custom Agent with All Three Artifacts

```json
{
  "version": 1,
  "name": "Technical Writer",
  "slug": "technical-writer",
  "description": "Produces clear developer documentation from code and specs.",
  "artifacts": [
    {
      "kind": "soul",
      "title": "SOUL",
      "content": "# Technical Writer\n\nYou are a senior technical writer specializing in developer documentation.\n\nPrinciples:\n- Write for developers who skim first, read second.\n- Use concrete code examples over abstract explanations.\n- Keep paragraphs short and scannable.\n- Include prerequisites and expected outcomes for every guide.\n\nDefault deliverable format:\n- Write Markdown.\n- Use heading hierarchy: H2 for sections, H3 for subsections.\n- Include code blocks with language annotations."
    },
    {
      "kind": "template",
      "title": "Documentation Template",
      "content": "# [Title]\n\n## Overview\n## Prerequisites\n## Quick Start\n## API Reference\n## Examples\n## Troubleshooting\n## FAQ"
    },
    {
      "kind": "reference",
      "title": "Documentation Standards",
      "content": "# Standards\n\n- Every function gets: description, params, return value, example.\n- Error codes get: code, meaning, resolution.\n- Guides get: goal, steps, verification.\n- Never assume the reader knows your internal terminology."
    }
  ]
}
```

---

## Schema Reference

### Workflow Definition

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `version` | `1` | Yes | Always `1` |
| `name` | `string` | Yes | Human-readable name |
| `description` | `string \| null` | No | What the workflow accomplishes |
| `steps` | `array` | Yes | Ordered, runs sequentially |

### Step

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `name` | `string` | Yes | Short action name |
| `description` | `string \| null` | No | What this step produces |
| `transition_type` | `artifact_generated \| manual_completion` | Yes | How the step completes |
| `artifact_pattern` | `string \| null` | Conditional | Required for `artifact_generated` |
| `require_approval` | `boolean` | Yes | Human must approve? |
| `requires_user_input` | `boolean` | Yes | Needs user input? |
| `user_input_question` | `string \| null` | Conditional | Required when `requires_user_input: true` |
| `agent_slug` | `string \| null` | No | References a bundled agent |
| `prompts` | `array` | Yes | Must include default (`executor_type: null`) |
| `skills` | `array` | No | Optional skill references |

### Bundle Wrapper

| Field | Type | Notes |
|-------|------|-------|
| `kind` | `"qogni.workflow_bundle"` | Required literal |
| `version` | `1` | Bundle format version |
| `workflow` | `object` | Full workflow definition |
| `resources.custom_agents` | `array` | Agent definitions |

### Custom Agent

| Field | Type | Required |
|-------|------|----------|
| `version` | `1` | Yes |
| `name` | `string` | Yes |
| `slug` | `string` | Yes (kebab-case) |
| `description` | `string` | Yes |
| `artifacts` | `array` | Yes (must include one `soul`) |

### Catalog Entry (`list.json`)

| Field | Type | Notes |
|-------|------|-------|
| `id` | `string` | Unique, lower-kebab-case |
| `directory` | `string` | Relative path, no `..` |
| `definition_file` | `string` | `workflow.yaml` or `workflow-bundle.json` |
| `category` | `string` | Short and stable |
| `tags` | `array` | Short searchable labels |

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Agent doesn't recognize the skill | Verify `SKILL.md` is in the correct directory for your tool (see Installation table) |
| JSON parse error on bundle | Run `python3 -m json.tool workflow-bundle.json` to find syntax errors |
| Step skipped or fails | Check that `artifact_pattern` matches what the prompt actually creates |
| Agent persona not applied | Verify `agent_slug` in the step matches a `slug` in `resources.custom_agents` |
| "Missing default prompt" error | Every step needs at least one prompt with `executor_type: null` |
| Workflow not in marketplace | Check `list.json` has an entry with matching `directory` and `definition_file` |
| Agent produces unstructured output | Add a `template` artifact to the agent definition |
| `requires_user_input` but no question | Set `user_input_question` to a specific, actionable question |

---

## Validation Checklist

Before publishing a workflow:

- [ ] Every step has `name`, `transition_type`, `require_approval`, and a default prompt
- [ ] `artifact_generated` steps have `artifact_pattern`
- [ ] `manual_completion` steps have `artifact_pattern: null`
- [ ] Every `agent_slug` matches a bundled agent's `slug`
- [ ] Every custom agent has exactly one `soul` artifact
- [ ] `list.json` entry exists with correct `directory` and `definition_file`
- [ ] JSON/YAML files parse without errors
- [ ] All paths are relative and free of `..`

---

## License

See repository root for license information.
