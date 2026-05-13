# Case Study: Solo Developer — Personal Project

> **Product:** `logtail` — a CLI tool that tails, filters, and colorizes log files in real-time.
> **Developer:** You, building something you actually need for your own workflow.
> **Stack:** Go, Cobra CLI, no external services.

## The Situation

You're tired of `tail -f | grep | sed` chains. You want a single CLI that can:
- Tail multiple log files simultaneously
- Filter by level, source, or regex patterns
- Colorize output based on configurable rules
- Save filter presets for reuse

No stakeholders. No deadlines. Just a tool you want to build right.

## What TSDD Looks Like Here

You skip User Stories — you're the only stakeholder. You skip heavy proposals — the problem is clear. You go straight to specs.

### Feature: Filter Presets

You want to save and load filter configurations so you don't retype them every session.

**Step 1: Write the spec**

You invoke `spec-writer` and describe what you want:

> "I want to save filter presets — name, patterns, colors — and load them by name. Store them in a config file."

The skill produces a spec at `docs/specs/004-filter-presets.md`:

```markdown
# Spec 004: Filter Presets

## Context
Users need to save and reuse filter configurations across sessions.

## Scope
- Save a preset with: name, level filters, source filters, regex patterns, color rules
- Load a preset by name via `--preset <name>` flag
- List available presets via `presets list` subcommand
- Delete a preset via `presets delete <name>`
- Store presets in `~/.config/logtail/presets.yaml`

## Out of scope
- Sharing presets between machines
- Preset versioning or migration
- Import/export presets

## Files to modify
- `cmd/presets.go` — new subcommand
- `internal/config/presets.go` — new preset storage logic
- `internal/config/config.go` — extend with preset loading

## Acceptance criteria
- [ ] `logtail --preset prod` applies saved filters and colors
- [ ] `logtail presets list` shows all saved presets with names
- [ ] `logtail presets save prod --level error --source api` creates a preset
- [ ] `logtail presets delete prod` removes the preset file entry
- [ ] Presets persist across CLI invocations
- [ ] Invalid preset name returns a clear error message
```

**Step 2: Implement**

You invoke `spec-implement`. It reads the spec, reads the existing code, plans the changes, and executes. You review the output, confirm it matches what you wanted.

**Step 3: Commit**

```
feat(cli): add filter presets save, load, list, and delete

Spec: docs/specs/004-filter-presets.md
```

### Bug Fix: Regex Panic on Invalid Pattern

You discover that entering an invalid regex like `[unclosed` causes a panic instead of a graceful error.

**Step 1: Write the spec (yes, even for bugs)**

```markdown
# Spec 005: Fix regex panic on invalid pattern

## Context
Entering an invalid regex pattern causes a panic in the filter parser
instead of returning a user-friendly error.

## Scope
- Validate regex patterns before compilation
- Return error message: "invalid regex pattern: <reason>"
- Add test case for unclosed brackets, invalid escape sequences

## Files to modify
- `internal/filter/regex.go` — add validation before compile
- `internal/filter/regex_test.go` — add panic-recovery tests

## Acceptance criteria
- [ ] `logtail --regex '[unclosed'` prints error and exits gracefully
- [ ] No panics for any invalid regex input
- [ ] Error message includes the specific regex problem
```

**Step 2: Implement, commit.**

```
fix(filter): validate regex patterns before compilation to prevent panic

Spec: docs/specs/005-fix-regex-panic.md
```

### Enhancement: Support JSON Log Parsing

You realize many of your logs are JSON and you want structured filtering.

**Step 1: Proposal first (the problem space is bigger)**

You invoke `proposal-writer` because this isn't just a filter change — it's a new parsing mode. The proposal explores:

- Should JSON be auto-detected or explicit via `--format json`?
- How do filters work on JSON fields? (`--json-field level error`)
- What about nested JSON?
- Performance implications of JSON parsing on large log streams

The proposal at `docs/proposal/json-log-support.md` produces a planned specs list:

```markdown
## Planned specs

1. `006-json-log-parser.md` — core JSON parsing engine
2. `007-json-field-filters.md` — filter by JSON field paths
3. `008-auto-format-detection.md` — detect JSON vs plain text automatically
```

You write them one at a time with `spec-writer`, implement with `spec-implement`.

## Your Spec Index

Over time, `docs/specs/README.md` becomes your project roadmap:

```markdown
# Specs Index

| ID   | Title                  | State     | Layer |
|------|------------------------|-----------|-------|
| 001  | Multi-file tailing     | done      | CLI   |
| 002  | Level-based filtering  | done      | CLI   |
| 003  | Color output rules     | done      | CLI   |
| 004  | Filter presets         | done      | CLI   |
| 005  | Fix regex panic        | done      | CLI   |
| 006  | JSON log parser        | in-progress| CLI  |
| 007  | JSON field filters     | draft     | CLI   |
| 008  | Auto format detection  | draft     | CLI   |
```

## What You Actually Use

| Stage | When |
|---|---|
| `spec-writer` | Every feature, every bug fix |
| `spec-implement` | After every spec |
| `proposal-writer` | When the problem space is fuzzy (like JSON support) |
| `tsdd-init` | Once, at project start |
| `agents-writer` | Once, to set up AGENTS.md |

Your AGENTS.md is 60 lines. Your specs are half a page each. Your spec index is your todo list. Six months from now, you open a spec and remember exactly why you built something the way you did.
