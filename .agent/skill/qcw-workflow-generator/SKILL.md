---
name: qcw-workflow-generator
description: Use when authoring, generating, or modifying Qogni Cowork workflow definitions and workflow bundles with custom agents for the marketplace
---

# QCW Workflow Generator

## Overview

Generate Qogni Cowork workflow definitions that run one step at a time in harness tools (Claude Code, Gemini CLI, Codex). Workflows are reusable, version-controlled multi-step processes with optional custom agents.

**Core principle:** Each workflow step produces an artifact or awaits manual completion. Steps run sequentially — each step can reference outputs from earlier steps.

## When to Use

- Creating a new multi-step workflow for the Qogni Cowork marketplace
- Generating `workflow.yaml` for simple workflows without custom agents
- Generating `workflow-bundle.json` for workflows that need custom agent personas
- Adding or updating entries in `list.json`
- Converting an ad-hoc multi-step process into a reusable workflow definition

## Workflow Formats

| Format | Use When |
|--------|----------|
| `workflow.yaml` | No custom agents needed. Steps use default executor. |
| `workflow-bundle.json` | Steps need custom agent personas (soul, templates, references). |

**Prefer `workflow-bundle.json`** when steps benefit from specialized agent roles. Most marketplace workflows should use bundles.

## Canonical Schema

### Workflow Definition

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `version` | `1` | Yes | Always `1`. |
| `name` | `string` | Yes | Human-readable workflow name. |
| `description` | `string \| null` | No | What the workflow accomplishes. |
| `steps` | `array` | Yes | Ordered list. Runs sequentially. |

### Step Definition

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `name` | `string` | Yes | Short action name. |
| `description` | `string \| null` | No | What this step produces. |
| `transition_type` | `artifact_generated \| manual_completion` | Yes | How the step completes. |
| `artifact_pattern` | `string \| null` | Conditional | Required when `transition_type` is `artifact_generated`. Path to the output file. |
| `require_approval` | `boolean` | Yes | Whether a human must approve before proceeding. |
| `requires_user_input` | `boolean` | Yes | Whether the step needs user input before running. |
| `user_input_question` | `string \| null` | Conditional | Required when `requires_user_input` is `true`. |
| `agent_slug` | `string \| null` | No | References a bundled custom agent by slug. |
| `prompts` | `array` | Yes | Must include one default prompt (`executor_type: null`). |
| `skills` | `array` | No | Optional skill references. Deduplicated by `skill_name`. |

### Prompt Object

| Field | Type | Notes |
|-------|------|-------|
| `executor_type` | `string \| null` | `null` for default. `CLAUDE_CODE`, `GEMINI`, or `CODEX` for executor-specific. |
| `prompt` | `string` | The instruction for this step. |

### Bundle Wrapper (`workflow-bundle.json`)

| Field | Type | Notes |
|-------|------|-------|
| `kind` | `"qogni.workflow_bundle"` | Required literal. |
| `version` | `1` | Bundle format version. |
| `workflow` | `object` | Full workflow definition (same schema as `workflow.yaml`). |
| `resources.custom_agents` | `array` | Agent definitions referenced by steps via `agent_slug`. |

### Custom Agent Definition

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `version` | `1` | Yes | Always `1`. |
| `name` | `string` | Yes | Human-readable agent name. |
| `slug` | `string` | Yes | Stable kebab-case identifier. Steps reference this. |
| `description` | `string` | Yes | What the agent does. |
| `artifacts` | `array` | Yes | Must include exactly one `soul` artifact. |

### Agent Artifact Kinds

| Kind | Purpose | Required |
|------|---------|----------|
| `soul` | Agent persona, principles, and deliverable format. | Exactly one per agent. |
| `template` | Structured deliverable template the agent fills in. | Optional but recommended. |
| `reference` | Domain rules, heuristics, and constraints. | Optional but recommended. |

## Authoring Guide

### Step Design

**Choose `artifact_generated`** when the step produces a file:
- Set `artifact_pattern` to the output path (e.g., `docs/spec.md`, `index.html`)
- Later steps can reference this file in their prompts

**Choose `manual_completion`** for human gates:
- Code reviews, deployment approvals, manual testing checkpoints
- Set `require_approval: true` for approval gates
- Set `artifact_pattern: null`

**Use `requires_user_input`** when a step needs context only the user can provide:
- Set `user_input_question` to a clear, specific question
- Example: `"What is the target audience for this API?"`

### Prompt Writing

Follow the **"Activate [Agent]"** pattern from proven workflows:

```
Activate [Agent Name].

[Context: what the agent needs to know]
[Input: files from prior steps to read]
[Task: specific deliverable to produce]
[Constraints: format, scope, guardrails]
[Output: exact file path to create]
```

**Key rules:**
- Reference prior step outputs by file path (e.g., `Read docs/spec.md`)
- Be specific about deliverable format and sections
- Include a fallback instruction if prior artifacts are missing
- Keep prompts self-contained — agents don't share context between steps

### Agent Design

See `agent-design-guide.md` for the full reference. Quick summary:

**Soul artifact** — the agent's identity:
- Role statement ("You are a senior...")
- 4–6 principles the agent follows
- Default deliverable format

**Template artifact** — structured output:
- Markdown with headings and fill-in sections
- Matches what the step prompt asks for

**Reference artifact** — domain knowledge:
- Heuristics, constraints, evaluation criteria
- Keeps the soul concise while providing depth

### Marketplace Registration

Add entries to `list.json` next to the workflow directories:

```json
{
  "id": "my-workflow",
  "directory": "my-workflow",
  "definition_file": "workflow-bundle.json",
  "category": "agile",
  "tags": ["planning", "sprint"]
}
```

**Rules:**
- `id`: stable, unique, lower-kebab-case
- `directory`: safe relative path, no `..`
- `category`: short and stable (`agile`, `web-development`, `review`, `ops`, `api-development`, `brownfield`)
- `tags`: short searchable labels

## Minimal Examples

### Simple Workflow (YAML)

```yaml
version: 1
name: "Quick Spec Review"
description: "Review a spec and produce notes."
steps:
  - name: "Draft Review"
    description: "Review the spec and write notes."
    transition_type: artifact_generated
    artifact_pattern: "docs/review.md"
    require_approval: false
    requires_user_input: false
    user_input_question: null
    agent_slug: null
    prompts:
      - executor_type: null
        prompt: |
          Review the current specification and save your notes to docs/review.md.
    skills: []
```

### Bundle With Custom Agent (JSON)

```json
{
  "kind": "qogni.workflow_bundle",
  "version": 1,
  "workflow": {
    "version": 1,
    "name": "Guided Code Review",
    "description": "A structured code review with a custom reviewer agent.",
    "steps": [
      {
        "name": "Code Review",
        "description": "Analyze the codebase and produce a review report.",
        "transition_type": "artifact_generated",
        "artifact_pattern": "review/report.md",
        "require_approval": false,
        "requires_user_input": true,
        "user_input_question": "Which files or directories should be reviewed?",
        "agent_slug": "code-reviewer",
        "prompts": [
          {
            "executor_type": null,
            "prompt": "Activate Code Reviewer.\n\nReview the specified files. Create review/report.md with findings."
          }
        ],
        "skills": []
      }
    ]
  },
  "resources": {
    "custom_agents": [
      {
        "version": 1,
        "name": "Code Reviewer",
        "slug": "code-reviewer",
        "description": "Reviews code for correctness, maintainability, and security.",
        "artifacts": [
          {
            "kind": "soul",
            "title": "SOUL",
            "content": "# Code Reviewer\n\nYou are a senior code reviewer..."
          }
        ]
      }
    ]
  }
}
```

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Missing default prompt (`executor_type: null`) | Every step must have at least one prompt with `executor_type: null`. |
| `agent_slug` references non-existent agent | Every `agent_slug` must match a `slug` in `resources.custom_agents`. |
| Agent missing `soul` artifact | Every custom agent must have exactly one artifact with `"kind": "soul"`. |
| `artifact_generated` without `artifact_pattern` | Set `artifact_pattern` to the expected output file path. |
| `requires_user_input: true` without question | Set `user_input_question` to a specific, actionable question. |
| Prompts don't reference prior outputs | Later steps should reference files created by earlier steps by path. |
| Duplicate `executor_type` in same step | Each step can have at most one prompt per executor type. |

## Completion Checklist

Before publishing a workflow to the marketplace:

- [ ] Every step has `name`, `transition_type`, `require_approval`, and a default prompt
- [ ] `artifact_generated` steps have `artifact_pattern`
- [ ] Every `agent_slug` matches a bundled agent's `slug`
- [ ] Every custom agent has exactly one `soul` artifact
- [ ] `list.json` entry exists with correct `directory` and `definition_file`
- [ ] JSON files parse without errors
- [ ] YAML files parse without errors
- [ ] All paths are relative and free of `..`
