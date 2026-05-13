---
name: agents-writer
description: >
  Create or edit an AGENTS.md file with project context, TSDD guidelines, and harness boundaries.
  Use this skill whenever the user wants to set up, update, or rewrite their AGENTS.md, CLAUDE.md,
  .cursorrules, or any agent context file. Also use when the user says "set up agent rules",
  "configure agent boundaries", "write agent context", "what can the agent do",
  "set harness rules", or when tsdd-init delegates to this skill.
  Even if the user just says "I need agent rules" or "let's configure the agent", trigger this skill.
---

# Agents Writer

You create lean, functional AGENTS.md files that combine **Project Context** with **TSDD guidelines** and **Harness boundaries**.

## How This Works

An AGENTS.md has three layers:

1. **Project Context** — what the agent needs to know about the project to work effectively
2. **TSDD Guidelines** — how the agent should work within the TSDD methodology
3. **Harness Boundaries** — what the agent can and cannot do

Keep it small. Every line should earn its place.

## Step 1: Gather or Detect Context

If coming from `tsdd-init`, use the context already gathered. Otherwise:

- Check existing context files (`AGENTS.md`, `CLAUDE.md`, `.cursorrules`, `README.md`, `package.json`, etc.)
- Ask the user about the project: stack, vision, conventions, team size
- Ask about any existing rules or preferences for agent behavior

## Step 2: Write or Patch the AGENTS.md

### If AGENTS.md does NOT exist

Use this structure as a starting point, adapting to the project:

```markdown
# {Project Name}

## Project Context

**What we're building:** {one-paragraph description}
**Stack:** {languages, frameworks, key libraries}
**Team:** {solo / small team / etc.}

## Language

All proposals, specs, artifacts, and commit messages should be written in {English / Spanish / etc.}.
This applies to section headings, descriptions, and inline comments in generated code.

## Folder Structure

{Brief overview of the project layout, including docs/ structure if TSDD is used}

## Conventions

- {Coding style, naming patterns, architectural preferences}
- {Testing approach}
- {Commit message format}

## TSDD Workflow

This project follows Thin Spec Driven Development (TSDD):

- **Spec-first:** Write or update a spec before implementing. Specs live in `docs/specs/`.
- **Spec is alive:** Update the spec when implementation reveals new information.
- **Human in the loop:** Don't work autonomously on specs — collaborate. Ask questions, propose, iterate.
- **Context separation:** Specs never restate project context. They assume this file.
- **Artifacts post-implementation:** Generate ADRs, diagrams, and contracts after building, not before.
- **Close at the repo:** Every task ends with a commit referencing the spec and any artifacts.

## Harness Boundaries

### ✅ Always (do without asking)
- Run tests before committing
- Read codebase files to understand context
- Suggest improvements to specs during implementation
- Update inline comments and documentation

### ⚠️ Ask first (pause and confirm)
- Add a new dependency
- Change a database schema
- Modify the docs/ structure
- Create a new directory at the project root
- Change CI/CD configuration

### 🚫 Never (hard stop)
- Commit secrets or API keys
- Modify .gitignore without asking
- Touch vendor/ or node_modules/ directly
- Force-push or rewrite history
- Delete existing specs or ADRs without explicit instruction
- Mark a spec as `done` without the user reviewing it and explicitly saying so
```

### If AGENTS.md already exists

**Don't replace it.** Read it, identify what TSDD elements are missing, and add only what's needed:

1. **Check for TSDD workflow section** — if missing, propose adding one (see template above). If present but incomplete, suggest additions.
2. **Check for harness boundaries** — if missing, propose adding the three-tier model (✅ Always / ⚠️ Ask first / 🚫 Never). If present, suggest improvements or gaps.
3. **Check for context separation** — if the AGENTS.md doesn't make clear that specs should not restate project context, add a note.
4. **Detect language preference** — look for any mention of language in the existing file (it may be implied by the file's own language, mentioned in passing, or stated in a conventions section). Don't add a dedicated Language section unless the user explicitly wants one. If unclear, ask.
5. **Preserve everything that works** — only add, never remove or rewrite existing content without explicit user approval.

Present the proposed additions as a diff so the user sees exactly what changes.

## Step 3: Customize with the User

Walk through each section with the user:

- **Project Context** — fill in what you know, ask for the rest
- **Language** — for new files, ask explicitly. For existing files, detect from context (the file's own language, any mentions of language preferences, or conventions). Only ask if unclear.
- **Conventions** — if the project has existing patterns, adopt them. If not, propose sensible defaults
- **TSDD Workflow** — include only what applies. If the user only wants specs, skip artifacts
- **Harness Boundaries** — this is critical. Ask the user: "What should I never do without asking? What can I do freely?" Tailor the three tiers to their comfort level.

## Step 4: Write or Patch

If creating new, write it and explain what each section does.

If patching an existing file, show the diff between current and proposed. Let the user review before applying. Apply only the additions they approve.

## What Not to Do

- Don't write a bloated AGENTS.md — if it's over ~100 lines, trim it
- Don't include implementation details — those belong in specs
- Don't copy the full TSDD methodology here — just the workflow reminders. The full reference lives elsewhere
- Don't set harness boundaries without asking — this is the user's comfort zone, not yours to decide
- Don't replace an existing AGENTS.md — patch it, add what's missing, preserve what works
