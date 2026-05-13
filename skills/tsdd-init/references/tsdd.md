# TSDD — Thin Spec Driven Development

> *Clear enough for an agent to execute. Lean enough for a human to maintain.*

## Core Philosophy

TSDD is a **methodology**, not a framework. It's a compass — a set of principles and stages that guide a development team from idea to repository. Each team implements it the way it makes sense: with AI agents, with plain Markdown files, with sticky notes. The tooling is irrelevant. The thinking is what matters.

### The Core Premise

> **A Spec exists to give an agent — human or AI — everything it needs to execute a task correctly, without ambiguity, and without asking twice.**

TSDD borrows from Lean Thinking — eliminate waste, deliver value early, decide as late as responsible. A spec full of business verbosity is waste. A spec full of implementation trivia is also waste. A decision that nobody recorded is a future incident.

## Prerequisites

TSDD works best when two engineering disciplines are in place:

### Context Engineering

Every agent operates on two distinct layers:

**Project Context** — stable, always-present knowledge: architecture, tech stack, coding conventions, folder structure, dependencies, team boundaries. Lives in `AGENTS.md`, `CLAUDE.md`, `.cursorrules`, or equivalent. Written once, maintained continuously. The agent carries it into every task.

**Task Context (the Spec)** — specific, temporary context for a single task or feature. Scoped, task-bound, expires when the cycle closes.

> **A Spec never restates the Project Context.** If a Spec explains the stack, the framework, or the team's conventions, it's carrying fat that belongs elsewhere.

Beyond these two layers, a mature context setup may include:

| Tool | Purpose |
|---|---|
| MCP servers | Give agents access to external systems (repos, docs, databases, APIs) |
| Skills | Reusable, scoped behaviors the agent can invoke |
| RAG | Retrieve relevant knowledge from large documentation or codebases |
| Memory banks | Persist decisions, patterns, and learned context across sessions |
| Artifacts | Outputs from previous TSDD cycles that inform the next |

> **Context is infrastructure. Treat it like code.** Version it, maintain it, review it.

### Harness Engineering

The harness is the wiring that makes context usable. It defines:

- **What context gets injected** — and when, for which type of task
- **Which tools are available** — MCPs, Skills, capabilities scoped per agent or per stage
- **Agent boundaries** — what the agent can do autonomously, what requires approval, what's off-limits

A practical harness uses a three-tier boundary model:

| Tier | Meaning | Example |
|---|---|---|
| ✅ Always | Execute without asking | Run tests before committing |
| ⚠️ Ask first | Pause and confirm | Add a new dependency, change a schema |
| 🚫 Never | Hard stop | Commit secrets, modify CI config, touch vendor files |

The harness is not part of the Spec. It's part of the agent's operating environment — defined once at the project level, not per task.

## Core Principles

1. **Human in the loop — always.** The developer is not an approver at the end. They are a collaborator at every stage. The agent executes; the human directs, refines, and validates.

2. **The spec serves the team, not the process.** No spec artifact should exist solely to satisfy a methodology. If it doesn't help the team build better or faster, drop it.

3. **Decide as late as responsible.** Don't document decisions before you have enough information to make them well. The Proposal explores. The Spec decides. Artifacts record.

4. **The loop closes at the repository.** A workflow that ends before the commit is incomplete. Code, decisions, and history belong together.

5. **Modularity over prescription.** Not every feature needs User Stories. Not every task needs an ADR. TSDD stages are building blocks — use the ones that fit the context.

6. **Artifacts reflect reality.** Generated artifacts (diagrams, ADRs, specs) should describe what was actually built, not what was imagined at the start.

7. **Context is infrastructure.** A clean, well-maintained Project Context and a well-engineered harness are prerequisites, not afterthoughts.

## The Workflow

```
         [Human + Agent]
               │
            Proposal
               │
               ├── Does it involve a team or business stakeholders?
               │          │
               │        YES → [Human + Agent] User Stories
               │          │
               │         NO ──────────────────────┐
               │                                  │
               └──────────────────────────────────┘
                                  │
                          [Human + Agent]
                              Spec
                         (iterate until
                          unambiguous)
                                  │
                          [Human + Agent]
                           Implementation
                          (Spec is alive —
                        redirect, complete,
                          document inline)
                                  │
                          [Human + Agent]
                             Artifacts
                     (generate, review, iterate)
                                  │
                            [Human]
                             Commit
                                  │
                            [Human]
                           Merge Request
```

## The Stages

### 1. Proposal

The starting point. A **hybrid artifact** — mixes business intent with technical thinking from day one.

A Proposal should answer:
- What problem are we solving, and for whom?
- What are the main technical constraints or decisions pending?
- What does the system look like at a high level? (use diagrams — Mermaid or equivalent)

A Proposal is **not** a requirements document. It's a structured brainstorm. It ends when the team has enough shared understanding to move forward.

**Output:** A short document with context, a rough architecture diagram, and a list of open decisions.

### 2. User Stories *(optional)*

Use when:
- A product team or business stakeholders are involved
- The work is large enough to benefit from decomposition
- Multiple developers will work in parallel

Skip when:
- The task is purely technical
- The team already shares full context
- It's a solo developer with a clear goal

Structure:
```
As a [role], I want [capability], so that [outcome].
Acceptance criteria: [testable conditions]
```

No ceremony. No Gherkin required unless the team finds value in it.

### 3. Spec

The **central artifact** of TSDD. The contract between intent and execution — written for the agent (human or AI) that will implement it.

A well-written Spec is:
- **Technically precise** — defines what needs to be built at the implementation level: data structures, endpoints, behaviors, constraints, edge cases
- **Unambiguous** — an agent reading it should not need to infer, assume, or ask clarifying questions
- **Bounded** — defines explicit scope and explicit exclusions
- **Free of noise** — no business justification essays, no stakeholder language, no implementation trivia

A Spec should define:
- **Context** — what problem this solves and where it fits in the system (one paragraph max)
- **Scope** — what will be built, expressed technically
- **Out of scope** — explicit exclusions to prevent drift
- **Technical requirements** — data models, API contracts, business rules, constraints, error handling
- **Acceptance criteria** — testable conditions that define "done"
- **Open questions** — unresolved decisions that need to be made during implementation

> **A Spec is not thin because it's short. It's thin because it carries no fat.**

A Spec is **alive**. It is updated as implementation reveals new information. The final state reflects what was actually built — not what was originally imagined.

> **A Spec assumes the Project Context. It never restates it.**

**What a Spec is not:**
- A PRD or business requirements document
- A design document with exhaustive diagrams
- A conversation summary
- A placeholder to be filled in later

### 4. Implementation

Build against the Spec. When reality diverges from the Spec, update the Spec — don't silently abandon it.

One rule: **keep the Spec honest**.

### 5. Artifacts

Once implementation is complete (or at meaningful checkpoints), generate artifacts that serve the team and the future.

Artifacts are **contextual** — use what the situation calls for:

| Situation | Artifact |
|---|---|
| An architectural decision was made | ADR (Architecture Decision Record) |
| A new service or module was created | Architecture diagram |
| An API was defined or changed | API contract (OpenAPI, Bruno collection) |
| A complex flow was implemented | Sequence or flow diagram |
| A security or compliance decision | Decision log |

No situation requires all of these. One may require none.

### 6. Commit

Write a commit message that connects to the Spec and any generated artifacts.

Follow [Conventional Commits](https://www.conventionalcommits.org/) or your team's convention.

```
feat(auth): implement JWT refresh token rotation

Spec: docs/specs/auth-refresh.md
ADR: docs/adr/0012-jwt-rotation-strategy.md
```

### 7. Merge Request

The MR description is not a summary of the diff. It's a summary of the **intent**, the **decisions made**, and the **artifacts generated**.

A TSDD MR description references:
- The Spec (or User Story) it implements
- Any artifacts generated during the process
- Open questions or follow-up tasks

## What TSDD Is Not

- **Not a replacement for Agile.** Compatible with Scrum, Kanban, or any other framework. Operates at the task and feature level, not the process level.
- **Not a tool.** No CLI to install, no platform to sign up for, no templates required. Any text editor and a Git repository are sufficient.
- **Not waterfall.** The Spec is alive. Artifacts are generated post-implementation. Decisions are made as late as responsible.
- **Designed for agent-driven development, not dependent on it.** Works with human developers too. The discipline is the same.

## Comparison

| | Vibe Coding | BMAD / Spec Kit | **TSDD** |
|---|---|---|---|
| Spec overhead | None | High | Precise, no fat |
| Tooling required | None | Yes | None |
| Agent-ready Spec | No | Yes | Yes |
| Human in the loop | Reactive | Approval gates | Active collaborator |
| Context engineering | Ignored | Bundled in tool | Explicit prerequisite |
| Harness engineering | None | Partial | Explicit prerequisite |
| Closes to repo | No | Partial | Yes |
| Works without AI | Yes | No | Yes |
| Team-size fit | Solo | Medium–Large | Any |
| Artifacts | None | Pre-implementation | Post-implementation |
| Philosophy | Explore | Prescribe | Orient |
