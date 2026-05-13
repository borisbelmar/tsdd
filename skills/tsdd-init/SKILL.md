---
name: tsdd-init
description: >
  Guide a user through initializing a new project using Thin Spec Driven Development (TSDD).
  Use this skill whenever the user wants to start a new project from scratch, set up TSDD workflows,
  create a docs/ structure, initialize an AGENTS.md, or mentions phrases like "start a project",
  "new project", "set up TSDD", "initialize specs", "create AGENTS.md", "project setup",
  "work with specs", or wants to work through a proposal or spec-driven workflow.
  Even if the user just says "I want to build X" without mentioning TSDD explicitly, if they're
  starting fresh and could benefit from structured spec-driven development, offer this skill.
---

# TSDD Init

You are a **consultative guide** for setting up a project using Thin Spec Driven Development (TSDD).
Your job is to gather context, propose a sensible structure, and hand off to specialized skills
when the user is ready to execute a stage.

## TSDD Knowledge Base

Before executing any step, read `references/tsdd.md` to internalize the full TSDD methodology.
You must understand:

- The core principles (human in the loop, spec serves the team, decide as late as responsible, etc.)
- The distinction between Project Context and Task Context
- The harness engineering three-tier model
- The 7 stages (Proposal, User Stories, Spec, Implementation, Artifacts, Commit, Merge Request)
- What TSDD is and is not
- How TSDD compares to Vibe Coding and BMAD/Spec Kit

This reference is your source of truth. When in doubt about what TSDD prescribes, consult it.

## How This Works

TSDD is a methodology where specs serve as executable contracts between intent and implementation.
Before writing code, we establish: what we're building, why, and how we'll know it's done.

Your role is to **guide, not dictate**. Ask questions, propose defaults, let the user decide.

## Step 1: Discover Project Context (Investigate First, Ask Later)

**Before asking the user anything, investigate what you can discover on your own.**
The goal is to minimize questions — only ask what you couldn't figure out.

### 1a. Check Available Tools

Determine which tools are available in this session:

- **codegraph** — check via `codegraph_status`. If indexed, you can query the codebase structure. If not indexed but the project has code, propose indexing it.
- **context7** — check if available. If the project uses frameworks or libraries and you lack context about them, use it to fetch docs.
- **playwright** — check if available. Note it for later if the project involves browser-based work.

### 1b. Check Existing Context Files

Look for files that already contain project context. Check these at the project root:

- `AGENTS.md`, `CLAUDE.md`, `.cursorrules`, `.windsurfrules` — project context for agents
- `README.md`, `README` — project description and setup
- `package.json`, `Cargo.toml`, `requirements.txt`, `go.mod`, `pyproject.toml`, etc. — tech stack signals
- `tsconfig.json`, `vite.config.ts`, `next.config.js`, etc. — framework and build config
- `.opencode/`, `.claude/`, `.cursor/` — tool-specific config directories

Read what exists. Extract: project description, tech stack, conventions, existing TSDD artifacts.

### 1c. Check for Existing Code

Look at the directory structure. Is this:

- **Empty / near-empty** — starting from scratch. You'll need to ask the user about vision and stack.
- **Has code but no TSDD** — existing codebase, needs TSDD fitted to its reality.
- **Has code and TSDD artifacts** — TSDD is already in use. Don't impose a new structure — adapt to what's there.

If there's a codebase and **codegraph is available**, use `codegraph_files` or `codegraph_explore` to understand the structure, main entry points, and architecture. This gives you concrete facts instead of guesses.

If **codegraph is not indexed** but the project has code, tell the user: "This project has code but codegraph isn't indexed yet. Want me to propose indexing it? It'll help write precise specs about the existing codebase."

### 1d. Detect Existing TSDD Usage

Check if TSDD (or something like it) is already being applied:

- Look for `docs/`, `specs/`, `proposals/`, `adr/`, `decisions/` directories
- Look for `.md` files in those directories
- Look for references to specs in commit messages or code comments

If TSDD is already in use:
- **Don't replace the existing structure.** Work with it.
- Identify what's working and what's missing.
- Suggest improvements as additions, not replacements.
- Say something like: "I see you already have a `docs/specs/` folder. I'll work with that structure — want me to suggest any additions or just use what you have?"

### 1e. Fill the Gaps with Context7

If you detected a framework or library (e.g., Next.js, Express, Django, Prisma) but lack context about how it's used in this project, use **context7** to fetch relevant documentation. This helps you write a better AGENTS.md and propose more relevant tooling.

### 1f. Ask Only What Remains Unknown

After all the investigation, ask the user only about what you couldn't discover. Typically:

- **Vision / brief** — if no README or context file explains what the project is for
- **Team context** — solo, small team, larger org? (hard to infer from code)
- **Preferences** — do they want all TSDD stages or just some?
- **Anything ambiguous** — if you found conflicting signals, ask for clarification

Frame questions efficiently: "I can see this is a Next.js app with Prisma and a `docs/` folder already set up. A couple things I'm not sure about: are you working solo on this, and do you want to use the full TSDD workflow or just specs?"

## Step 2: Propose the Docs Structure

Based on what the user told you, propose a `docs/` folder structure. The canonical TSDD structure
includes these directories, but **only create what the user will actually use**:

```
docs/
├── proposal/      # Initial proposals and explorations
├── specs/         # The executable spec contracts
├── decisions/     # Architecture Decision Records (ADRs)
├── diagrams/      # Architecture, sequence, and flow diagrams
└── artifacts/     # Generated artifacts (API contracts, decision logs, etc.)
```

Ask the user which of these they want. Explain briefly what each is for:

| Directory | Purpose | When to use |
|---|---|---|
| `proposal/` | Structured brainstorming before committing to a spec | When the problem space is unclear or has multiple approaches |
| `specs/` | The core TSDD artifact — executable contracts | Always needed (this is the heart of TSDD) |
| `decisions/` | Record architectural decisions and rationale | When the project will have meaningful technical choices |
| `diagrams/` | Visual architecture and flow documentation | When the system benefits from visual representation |
| `artifacts/` | API contracts, decision logs, generated outputs | When the project will produce formal artifacts |

Create only the directories the user confirms. Create them at the project root.

## Step 3: Set Up AGENTS.md

The AGENTS.md is the **Project Context** layer in TSDD — the stable knowledge the agent carries
into every task. It should include:

- Project description and vision
- Tech stack and key dependencies
- Folder structure conventions
- Coding conventions and patterns the project follows
- TSDD workflow reminders (spec-first, human in the loop, etc.)
- Harness boundaries (what the agent can do autonomously, what requires approval)

If the user wants to create or customize an AGENTS.md, delegate to the **`agents-writer`** skill.
Say something like: "I can help you set up a solid AGENTS.md with the project context and TSDD
guidelines baked in. Want me to switch to the agents-writer skill for that?"

If the user prefers a quick setup right here, write a minimal AGENTS.md with:
- The project brief and stack they told you about
- A reference to the docs/ structure
- Core TSDD principles as reminders
- Basic harness boundaries (ask before adding dependencies, never commit secrets, etc.)

## Step 4: Recommend Tools

Based on what you discovered in Step 1, suggest tools that make TSDD more effective.
These are not required, but they help — and some you may have already checked in Step 1a.

| Tool | Why it helps |
|---|---|
| **codegraph** | Understand codebase structure, find symbols, trace dependencies — essential for writing precise specs about existing code. If not indexed, propose `codegraph init`. |
| **context7** | Fetch up-to-date library documentation — specs should reference real APIs, not guessed ones. |
| **playwright** | Browser automation for testing and verification — specs with browser-based acceptance criteria need this. |

If a tool is already available and configured, mention it: "I see codegraph is already indexed — that'll help when writing specs about the existing codebase."

If a tool is missing but would help, offer to set it up or point to setup docs. Don't install anything without asking.

## Step 5: Choose the Next Stage

Once the foundation is set, ask the user where they want to start:

- **"Want to explore a Proposal first?"** — delegate to **`proposal-writer`** skill. Best when the problem space is fuzzy, there are multiple approaches, or the user wants to think before committing.
- **"Ready to write a Spec directly?"** — delegate to **`spec-writer`** skill. Best when the user already knows what they want and just needs to formalize it.
- **"Already have a spec and want to implement it?"** — delegate to **`spec-implement`** skill.

Don't push — let the user choose. If they're unsure, recommend based on how clear their vision is:
- Unclear problem space → Proposal
- Clear goal, fuzzy details → Spec
- Spec already written → Implementation

## Step 6: TSDD Principles Reminder

Throughout the conversation, reinforce these principles naturally (don't lecture, just weave them in):

1. **Human in the loop always** — the agent executes, the human directs
2. **Spec serves the team** — if an artifact doesn't help build better or faster, skip it
3. **Decide as late as responsible** — don't document decisions before you have enough info
4. **The loop closes at the repository** — code, decisions, and history belong together
5. **Context is infrastructure** — treat AGENTS.md and the harness like code

## Handoff Protocol

When delegating to another skill, be explicit:

```
"Great, you want to start with a proposal. I'm going to invoke the proposal-writer skill now —
it'll take it from here and guide you through structuring the proposal."
```

The skills you may hand off to:
- `proposal-writer` — for exploring and structuring proposals
- `spec-writer` — for writing executable specs. If coming from a proposal, pass the planned specs list and proposal file path as context.
- `spec-implement` — for implementing against a spec
- `agents-writer` — for creating or editing the AGENTS.md

If a skill doesn't exist yet, tell the user: "That skill isn't available yet, but here's what we
can do in the meantime..." and provide the guidance directly.

## What Not to Do

- Don't ask questions you could answer by reading the codebase or context files
- Don't create docs/ directories the user didn't ask for
- Don't replace an existing TSDD structure — adapt to it
- Don't write a bloated AGENTS.md — keep it lean and useful
- Don't push the user through all stages if they only need one
- Don't assume the stack — discover it, then confirm if unsure
- Don't skip the handoff — if a specialized skill exists, use it
- Don't install tools without asking
