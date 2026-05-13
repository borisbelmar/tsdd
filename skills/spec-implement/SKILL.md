---
name: spec-implement
description: >
  Implements a TSDD spec end-to-end: reads the spec, understands the affected
  files, plans and applies the changes, runs validation, updates the spec state,
  and closes the cycle. Use this skill whenever the user wants to implement a
  spec, work on a numbered spec, or says things like "implement spec X",
  "let's do spec 021", "apply the changes from spec N", "continue with the next
  spec", or references a spec by number, name, or topic. Also activate when the
  user pastes a spec path or says something like "let's implement this".
  Do NOT activate when the user is only writing or updating a spec without
  implementing it — that belongs to the spec-writer skill.
---

# Spec Implementation

> TSDD Stage 4 — Implementation. Build against the spec. When reality diverges
> from the spec, update the spec — don't silently abandon it.

This skill drives the implementation phase of a TSDD cycle. It assumes the
project has a spec directory (typically `docs/specs/`) and a Project Context
file (`AGENTS.md`, `CLAUDE.md`, or equivalent) that defines the stack,
conventions, and validation commands.

---

## Phase 1 — Read and Understand

Before touching any code, build a complete picture of what needs to change and why.

**1. Locate the spec** — Read the spec index (e.g. `docs/specs/README.md`) to
find the spec file. Then read the spec itself in full: objective, scope,
expected file changes, acceptance criteria, and any stated prerequisites.

**2. Check the spec state** — If the spec is already `done`, tell the user and
ask if they want to reopen it. If prerequisites are listed, check that they are
also done before proceeding.

**3. Read all files that will change** — Open every file listed in the spec
under "files to modify" and "files to create". Also read adjacent files that
provide context: shared types, route registrations, index files, neighboring
modules. Understanding the existing code is what separates a clean
implementation from a noisy one.

**4. Check for partial work** — Scan quickly to see if any spec changes have
already been applied. Note this to the user so you don't duplicate work.

**5. Read the Project Context** — Read `AGENTS.md` (or the equivalent context
file) to understand the stack, conventions, build order, and validation
commands. The spec never restates this — it lives here.

---

## Phase 2 — Plan

**Present the plan** — For each change, describe: which file, what the current
relevant code looks like, and what you will change. Group related edits. Be
specific enough that the user can spot misunderstandings before a single line
is written.

**Surface ambiguities** — If anything in the spec is unclear, or if you see a
conflict between the spec and the existing code, ask before assuming. Specs
are living documents; a clarifying question now is cheaper than a wrong
implementation later.

**Update the spec when clarifications arrive** — If the user corrects your
understanding, update the spec immediately (scope, file list, acceptance
criteria, technical notes) and add a note recording the decision. This keeps
the spec honest as the single source of truth.

---

## Phase 3 — Implement

**Apply the changes** — Make every edit from the plan. Apply independent edits
in parallel where the tooling allows.

**Keep the spec honest** — If implementation reveals that a file not listed in
the spec needs to change, update the spec's file list. If an acceptance
criterion turns out to be wrong, fix it. The final state of the spec should
reflect what was actually built.

**Run validation** — Use the commands defined in the Project Context
(`AGENTS.md` or equivalent). If the project defines a build order (e.g.
shared types before consuming packages), follow it. If a check fails, read
the error, fix it, and re-run. Don't skip a failing check — the spec isn't
done until all checks pass.

**Mark acceptance criteria** — Check off every criterion in the spec as you
verify it.

---

## Phase 4 — Close

**Present results** — Show a concise summary: files changed, what each change
does, and validation results. Don't close the spec yet — wait for the user.

**Wait for user review** — NEVER mark a spec as `done` until the user explicitly
reviews and confirms. Present the results clearly and ask for review. The spec
state stays as-is until the user says it's done.

**Do NOT advance** — After presenting results, do NOT start the next spec or
suggest moving forward without being explicitly told. Wait for the user to
decide the next step.

**Close the spec** — Only when the user confirms:
- Set the spec state to `done`
- Update the spec index if one exists

**Offer to commit** — Ask the user if they want to commit now. Follow the
commit convention defined in the Project Context (e.g. Conventional Commits
or a team-specific format). A TSDD commit connects to the spec and any
artifacts generated:

```
feat(scope): short description

Spec: docs/specs/NNN-spec-name.md
```

Do not commit without being explicitly asked.

---

## Principles

These reflect TSDD's implementation stage. Keep them in mind throughout.

- **The spec is alive.** Update it as you learn. A spec that reflects what was
  actually built is more valuable than one that describes the original plan.

- **The spec assumes the Project Context.** Never restate the stack, the
  framework, or the conventions in a spec — they belong in `AGENTS.md`. If the
  spec explains something that applies to every task, it's carrying fat.

- **Ask rather than assume.** If the spec is ambiguous, a short question is
  faster than a wrong implementation. This is expected and healthy.

- **Don't gold-plate.** Implement exactly what the spec says. Spotted an
  improvement outside the spec's scope? Mention it to the user instead of
  silently doing it — it may deserve its own spec.

- **The loop closes at the repository.** A cycle that ends before the commit
  is incomplete. Code, decisions, and history belong together.
