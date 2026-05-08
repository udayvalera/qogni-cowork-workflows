# Workflow Marketplace Repository Instructions

This repository is the source of truth for workflow marketplace content.
It should contain only workflow catalog metadata, workflow definitions, and
optional workflow bundles with custom agents.

## Repository Layout

```text
list.json
instruction.md
my-workflow/
  workflow.yaml
custom-agent-workflow/
  workflow-bundle.json
```

Keep `list.json` at the repository root. Each workflow must live in its own
directory next to `list.json`.

## Catalog Format

`list.json` must be valid JSON with this shape:

```json
{
  "version": 1,
  "workflows": [
    {
      "id": "my-workflow",
      "directory": "my-workflow",
      "definition_file": "workflow.yaml",
      "category": "agile",
      "tags": ["planning", "greenfield"]
    }
  ]
}
```

Catalog rules:

- `version` must be `1`.
- `id` must be stable, unique, and lower-kebab-case.
- `directory` must be a safe relative path. Do not use absolute paths or `..`.
- `definition_file` is optional and defaults to `workflow.yaml`.
- Use `workflow.yaml` for normal workflows.
- Use `workflow-bundle.json` when the workflow includes custom agents.
- `category` should be short and stable, such as `agile`, `brownfield`, `review`, or `ops`.
- `tags` should be short searchable labels.

## Simple Workflow Definition

Create `workflow.yaml` in the workflow directory:

```yaml
version: 1
name: "Spec Review Workflow"
description: "Creates a review note and waits for manual completion."
workflow_initialization_script: "mkdir -p workflow-state"
workflow_environment_setup_script: "pnpm install --frozen-lockfile"
workflow_cleanup_script: "rm -rf workflow-state/tmp"
steps:
  - name: "Draft Review"
    description: "Create a spec review note in docs/review.md"
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

  - name: "Manual Check"
    description: "Wait for the user to approve the review."
    transition_type: manual_completion
    artifact_pattern: null
    require_approval: true
    requires_user_input: false
    user_input_question: null
    agent_slug: null
    prompts:
      - executor_type: null
        prompt: |
          Review docs/review.md for completeness and mark the step complete when ready.
    skills: []
```

Workflow rules:

- `version` must be `1`.
- `name` is required and becomes the imported workflow name.
- `description` can be a string or `null`.
- `workflow_initialization_script` is optional. This is the startup script and runs once before step 1 (after dependencies are materialized).
- `workflow_environment_setup_script` is optional. This is the per-step script and runs before every step.
- `workflow_cleanup_script` is optional. This is the workflow clean script and runs once during promote-to-task, after the workflow has completed and before task creation.
- Script fields are Bash snippets. Empty strings are normalized to `null`.
- Script failures are blocking: startup/per-step failures fail the workflow execution, and workflow clean failures fail promotion.
- `steps` run in order.
- Every step requires `name`, `transition_type`, `require_approval`, and at least one default prompt with `executor_type: null`.
- `transition_type: artifact_generated` requires `artifact_pattern`.
- `transition_type: manual_completion` should use `artifact_pattern: null`.
- If `requires_user_input` is `true`, `user_input_question` must be a non-empty question.
- Prompt `executor_type` can be `null` for default or executor-specific strings such as `CODEX`, `CLAUDE_CODE`, or `GEMINI`.
- Duplicate prompt targets in the same step are invalid.
- `skills` are optional metadata and are deduplicated by `skill_name`.

## Workflow Bundle With Custom Agents

Use `workflow-bundle.json` when a marketplace workflow must install custom
agents alongside the workflow.

```json
{
  "kind": "qogni.workflow_bundle",
  "version": 1,
  "workflow": {
    "version": 1,
    "name": "Custom Review Workflow",
    "description": "Runs a review step with a bundled custom agent.",
    "workflow_initialization_script": "mkdir -p workflow-state",
    "workflow_environment_setup_script": "pnpm install --frozen-lockfile",
    "workflow_cleanup_script": "rm -rf workflow-state/tmp",
    "steps": [
      {
        "name": "Custom Review",
        "description": "Review the repository with the bundled reviewer.",
        "transition_type": "manual_completion",
        "artifact_pattern": null,
        "require_approval": true,
        "requires_user_input": false,
        "user_input_question": null,
        "agent_slug": "marketplace-reviewer",
        "prompts": [
          {
            "executor_type": null,
            "prompt": "Use the custom reviewer persona and summarize risks before completion."
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
        "name": "Marketplace Reviewer",
        "slug": "marketplace-reviewer",
        "description": "Reviews implementation plans and code changes.",
        "artifacts": [
          {
            "kind": "soul",
            "title": "SOUL",
            "content": "You are Marketplace Reviewer. Focus on correctness, regressions, missing tests, and rollout risk."
          },
          {
            "kind": "template",
            "title": "Review Checklist",
            "content": "Check behavior, tests, migrations, security, and user-facing copy."
          }
        ]
      }
    ]
  }
}
```

Bundle rules:

- `kind` must be `qogni.workflow_bundle`.
- Bundle `version` must be `1`.
- `workflow` must follow the same workflow definition rules as `workflow.yaml`.
- `resources.custom_agents` can be empty, but use a plain `workflow.yaml` if no custom agents are needed.
- Each custom agent must have `version`, `name`, `slug`, and exactly one artifact with `"kind": "soul"`.
- Additional agent artifacts can use `"kind": "template"` or `"kind": "reference"`.
- Workflow steps can reference bundled agents by setting `agent_slug` to the agent `slug`.
- Keep bundled agent slugs stable because workflow steps refer to them by `agent_slug`.

## Completion Checklist

Before adding or updating a marketplace workflow:

- Add or update one entry in `list.json`.
- Create the matching workflow directory.
- Add exactly one definition file named by `definition_file`.
- Ensure every step has a default prompt with `executor_type: null`.
- If using startup/per-step/workflow clean scripts, keep them idempotent and safe to re-run.
- Ensure artifact-generated steps produce the path in `artifact_pattern`.
- Ensure bundled custom agents each include exactly one `soul` artifact.
- Keep all catalog paths relative and free of `..`.
- Prefer concise categories and tags so the marketplace remains searchable.
- Validate JSON files with a JSON parser and YAML files with a YAML parser before publishing.
