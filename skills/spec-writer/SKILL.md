---
name: spec-writer
description: >
  Writes and maintains TSDD specs — the central artifact that bridges intent
  and implementation. Use this skill whenever the user wants to plan, document,
  scope, or break down work before implementing it. Activate when the user says
  things like "let's spec this out", "write a spec for X", "I want to add X",
  "hagamos un spec para X", "planifiquemos X", or describes a feature, fix, or
  refactor that needs to be documented first. Also activate when the user wants
  to update or split an existing spec, or create a roadmap of specs from a
  larger plan. Do NOT activate when the user wants to implement an existing spec
  — that belongs to the spec-implement skill.
---

# Spec Writer

> TSDD Stages 1–3 — Proposal → Spec. Write the contract between intent and
> execution. Clear enough for an agent to execute. Lean enough for a human
> to maintain.

This skill drives the spec-writing phase of a TSDD cycle. It assumes the
project has a Project Context file (`AGENTS.md`, `CLAUDE.md`, or equivalent)
that defines where specs live, the language to write them in, and any
project-specific structure conventions.

---

## Phase 1 — Orient

**Read the Project Context** — Before writing anything, read `AGENTS.md` (or
equivalent) to understand: where specs are stored, what conventions apply
(language, naming, structure), and what the current state of the project is.

**Read the spec index** — Open the spec index file (e.g. `docs/specs/README.md`)
to find the next available ID, check if a similar spec already exists, and
understand current dependencies and states. If no index exists, create one
— ask the user for the location and structure they prefer first.

**Check for proposal context** — If coming from a proposal, read the proposal
file to understand the full roadmap, the planned specs list, and the broader
vision. Use the proposal's "Planned specs" section to understand sequencing
and dependencies. Reference the proposal in the spec (e.g. "See proposal:
`docs/proposal/mvp-tienda-mecanica.md`").

**Understand the request** — Before writing, make sure you understand:
- What problem this solves and why now
- What the rough scope is (what's in, what's out)
- Whether there are prerequisite specs that must be done first
- Whether this is a new spec, an update to an existing one, or a split

If the user's request is vague, ask one or two focused questions. Don't write
a spec based on assumptions — a wrong spec wastes an implementation cycle.

---

## Phase 2 — Write

**Follow project conventions** — Use the structure, language, and format
defined in the Project Context. The spec format may vary by project. If the
project has existing specs, match their structure exactly.

**What a good spec contains** — regardless of format, a spec should always
answer:
- **What** — what will be built, expressed technically (data structures,
  endpoints, behaviors, constraints, edge cases)
- **Why** — one short paragraph of context. Not a business essay — just enough
  for the implementer to understand the intent
- **Scope** — explicit includes and explicit excludes. The excludes section
  prevents scope creep and is as important as the includes
- **Where** — which files will change or be created, with concrete paths
- **Done** — acceptance criteria that are observable and testable, not vague

**A spec is thin, not short.** Thin means no fat — no business justifications,
no restating of project conventions, no implementation trivia. A spec for a
complex feature may be several pages; a simple fix may be half a page. Length
is not the measure — precision and clarity are.

**A spec assumes the Project Context.** Never restate the stack, the framework,
or the team's conventions inside a spec. That belongs in `AGENTS.md`. If a
spec explains something that applies to every task in the project, it's
carrying fat that belongs elsewhere.

**Technical notes explain the why.** The "technical notes" section (or
equivalent) is where you record decisions already made, constraints to respect,
and patterns to follow — with the reasoning behind them. "Use httpOnly cookies"
is a what. "Use httpOnly cookies because the app is same-origin and we want
to prevent XSS token theft" is a why. The second form is more valuable.

---

## Phase 3 — Close

**Present the spec** — Show the spec to the user before saving. Let them
review scope, acceptance criteria, and file list. A spec reviewed before
implementation is worth more than one corrected during it.

**Update the index** — Add or update the spec's entry in the index file.
Keep it sorted by ID. Mark which layers the spec touches (backend, frontend,
CLI, infra, etc.) using the conventions already present in the index.

**Mark the state** — Set the initial state to `draft` unless the user says
otherwise. States typically follow this progression:

```
draft → in-progress → done
```

Some projects add `planning` (being refined) or `review` (in validation).
Follow what the project already uses.

---

## Keeping Specs Honest

A spec is a living document — it should reflect what was actually built, not
what was originally imagined.

During and after implementation, update the spec when:
- A file not originally listed turns out to need changes
- An acceptance criterion was wrong or incomplete
- A technical decision changed
- Something was explicitly decided to be out of scope

Append a note to record significant decisions or course corrections. The spec's
final state is an artifact of the implementation — not a frozen plan.

---

## Splitting and Sequencing

When a user brings a large idea, the right output is often a set of specs, not
one spec:

- Break along natural checkpoints — each spec should be implementable and
  verifiable on its own
- Sequence by dependency — a spec should only start when its prerequisites are
  done
- Keep each spec cohesive — one clear objective, one bounded scope

Present the proposed breakdown to the user before writing all specs. It's
faster to correct the sequencing at the outline stage than after writing five
full specs.
