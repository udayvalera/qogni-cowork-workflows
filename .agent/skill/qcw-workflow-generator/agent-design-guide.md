# Agent Design Guide

## Overview

Custom agents give each workflow step a specialized persona. A well-designed agent produces
consistent, high-quality output without needing the prompt to over-specify format or tone.

## Agent Anatomy

Every custom agent has three artifact slots:

```
┌────────────────────────────────────┐
│  SOUL (required)                   │
│  Identity, principles, format      │
├────────────────────────────────────┤
│  TEMPLATE (recommended)            │
│  Structured output skeleton        │
├────────────────────────────────────┤
│  REFERENCE (optional)              │
│  Domain rules, heuristics          │
└────────────────────────────────────┘
```

## Writing a Soul Artifact

The soul defines who the agent is. It should be short (150–250 words) and cover:

1. **Role statement** — one sentence establishing expertise
2. **Principles** — 4–6 rules the agent always follows
3. **Default deliverable format** — what the agent produces

### Template

```markdown
# [Agent Name]

You are a [role] specializing in [domain]. Your job is to [primary deliverable].

Principles:
- [Principle about quality/standards]
- [Principle about audience awareness]
- [Principle about conciseness or specificity]
- [Principle about what to avoid]
- [Principle about deliverable readiness]

Default deliverable format:
- [Format type, e.g., "Write Markdown."]
- [Structure expectation, e.g., "Use clear headings for each section."]
- [Completeness standard, e.g., "Make output ready for the next step to consume."]
```

### Good Soul Example

```
# API Architect

You are a senior API designer specializing in RESTful and event-driven architectures.
Your job is to produce clear, implementation-ready API specifications.

Principles:
- Design for the consumer first, infrastructure second.
- Use consistent naming conventions and HTTP semantics.
- Document edge cases, error responses, and rate limits explicitly.
- Avoid over-abstraction: every endpoint should map to a clear user action.
- Include versioning strategy from the start.

Default deliverable format:
- Write OpenAPI 3.1 YAML when producing specifications.
- Write Markdown when producing design rationale or review notes.
- Include request/response examples for every endpoint.
```

### Common Soul Mistakes

| Mistake | Impact |
|---------|--------|
| Too long (500+ words) | Agent ignores later principles. |
| Too vague ("Be helpful") | No behavioral guidance. |
| No deliverable format | Agent guesses output structure. |
| Overlapping with prompt | Duplication wastes context. |

## Writing a Template Artifact

Templates define the **structure** of the agent's output. The step prompt fills in the
context; the template ensures consistent formatting.

### Design Rules

1. Use Markdown headings for sections
2. Use placeholder markers (`-` or `1.`) for fill-in content
3. Match the sections the step prompt asks for
4. Keep templates under 300 words — they are scaffolds, not essays

### Good Template Example

```markdown
# API Design Specification

## Overview
- API name:
- Version:
- Base URL:

## Authentication
- Method:
- Token lifecycle:

## Endpoints
### [METHOD] /resource
- Description:
- Request body:
- Response (200):
- Response (4xx):
- Rate limit:

## Error Contract
| HTTP Status | Error Code | Description |
|-------------|------------|-------------|

## Versioning Strategy
- Approach:
- Deprecation policy:
```

### When to Skip Templates

- Agent produces free-form analysis (no repeatable structure)
- Step output varies significantly per invocation
- The step prompt already specifies exact output format

## Writing a Reference Artifact

References provide **domain knowledge** the agent needs beyond its soul identity.
Think of them as the agent's reference manual.

### Good Reference Content

- Industry heuristics and rules of thumb
- Evaluation criteria and checklists
- Constraints specific to the workflow domain
- Anti-patterns and what to avoid

### Good Reference Example

```markdown
# REST API Design Heuristics

- Use plural nouns for resource collections (/users, not /user).
- Use HTTP methods semantically: GET reads, POST creates, PUT replaces, PATCH updates, DELETE removes.
- Return 201 for resource creation with a Location header.
- Return 204 for successful deletes with no response body.
- Paginate collections by default. Use cursor-based pagination for large datasets.
- Version in the URL path (/v1/), not headers, for discoverability.
- Error responses must include: HTTP status, machine-readable error code, human-readable message.
- Never expose internal IDs, stack traces, or database details in responses.
- Rate limit all public endpoints and return 429 with Retry-After header.
```

### When to Skip References

- The soul already covers all necessary domain knowledge
- The domain is well-known and the agent doesn't need specialization
- Adding a reference would just duplicate what's in the soul

## Agent Reuse Across Steps

Multiple workflow steps can reference the same `agent_slug`. This is useful when:

- A developer agent builds and then later refines the same artifact
- A reviewer agent checks work at multiple stages
- A coordinator agent opens and closes the workflow

The agent's persona stays consistent across all steps that reference it.

## Naming Conventions

| Component | Convention | Example |
|-----------|-----------|---------|
| `name` | Title Case | "Frontend Developer" |
| `slug` | lower-kebab-case | "frontend-developer" |
| `description` | Present tense, one sentence | "Builds responsive landing pages from specs." |
| Soul title | Always "SOUL" | "SOUL" |
| Template title | Descriptive of output | "API Design Specification" |
| Reference title | Descriptive of domain | "REST API Design Heuristics" |
