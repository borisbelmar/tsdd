---
name: proposal-writer
description: >
  Guide a user through writing a structured proposal for a project or feature using TSDD methodology.
  Use this skill whenever the user wants to explore a problem space, brainstorm a solution,
  write a proposal document, or mentions phrases like "write a proposal", "explore this idea",
  "what should we build", "brainstorm", "proposal", "let's think about", "how should this work",
  or when tsdd-init delegates to this skill.
  Even if the user just says "I have an idea for X" or "let's figure out how to approach Y",
  trigger this skill.
---

# Proposal Writer

You guide users through writing a **structured proposal** — a hybrid artifact that mixes business intent with technical thinking. This is TSDD's starting point: a structured brainstorm, not a requirements document.

## How This Works

A proposal answers three things:
1. **What problem are we solving, and for whom?**
2. **What does the solution look like at a high level?**
3. **What decisions are still open?**

The proposal is **iterative**. You work through it with the user, section by section, refining until they have enough shared understanding to move forward.

## Step 1: Detect Language Preference

Check the project's `AGENTS.md` for language preference. It may appear as a dedicated section, be mentioned within conventions, or simply be implied by the file's own language. If unclear, ask the user.

All skill instructions are in English, but the **output** follows the project's language preference.

## Step 2: Understand the Idea

Start by asking the user about their idea. You need:

- **What are we building?** — even a rough description
- **Who is it for?** — target users or stakeholders
- **What problem does it solve?** — the pain point or opportunity
- **Any constraints?** — tech stack, budget, timeline, platform

If coming from `tsdd-init`, you may already have some of this. Confirm and fill gaps.

## Step 3: Build the Proposal Iteratively

Propose sections to the user and work through them one at a time. Don't generate the whole thing at once — collaborate on each section.

### Standard Proposal Structure

The section headings below are shown in English as placeholders. **Translate all headings and content to the project's language** (from AGENTS.md or user preference).

```markdown
# Proposal: {Project or Feature Name}

## Problem

{What problem are we solving, and for whom?}

**For whom:**
- {User persona 1}
- {User persona 2}

## Proposal

{High-level solution — what are we building and why this approach?}

## Scope

### What's included

| Area | Detail |
|------|--------|
| {area} | {what it covers} |

### What's excluded (post-MVP)

- {excluded item 1}
- {excluded item 2}

## Technical constraints

- {constraint 1}
- {constraint 2}

## High-level diagram

{ASCII or Mermaid diagram showing the system at a high level}

## Configuration

| Parameter | Value |
|-----------|-------|
| {param} | {value} |

## Planned specs

1. `{001-name.md}` — {what this spec covers}
2. `{002-name.md}` — {what this spec covers}

## Success criteria

- [ ] {testable condition 1}
- [ ] {testable condition 2}

## Pending decisions

1. **{Decision topic}:** {what needs to be decided}
2. **{Decision topic}:** {what needs to be decided}
```

### Section-by-Section Guidance

Work through these sections with the user. Not all apply to every proposal — skip what doesn't fit. **All headings and content should use the project's language.**

**Problem** — Help them articulate the pain point. Ask "who feels this?" and "what are they doing now to cope?"

**Proposal** — Suggest approaches. If there are multiple viable paths, present options with tradeoffs. The proposal picks one but acknowledges alternatives.

**Visual Identity** — If the project has a design component, capture: logo assets, color palette, style references. Link to files in `assets/` or `docs/diagrams/`.

**Scope** — This is critical. Help them define what's in the MVP and what's explicitly out. A proposal without exclusions is a recipe for scope creep.

**Catalog / Data assumptions** — If the project works with dynamic data (products, content, users), clarify what's assumed vs. what's loaded dynamically.

**Technical constraints** — Conventions, file naming, what not to touch, commit format. Pull from the project's AGENTS.md if available.

**High-level diagram** — Generate an ASCII or Mermaid diagram showing the system layout. This helps everyone visualize the same thing.

**Configuration** — Key parameters: domain, currency, language, payment gateway, thresholds. Things that are decided and fixed.

**Planned specs** — List the specs that will be needed to implement this proposal. This bridges the proposal to the spec stage. Each item gets a descriptive name and one-line summary.

**Success criteria** — Testable conditions that define "the MVP works." Use checkboxes. Each criterion should be verifiable.

**Pending decisions** — Open questions that need resolution during implementation. Don't force decisions you don't have enough info for — that's what "decide as late as responsible" means.

**Assets** — If there are design files, logos, or other assets, list them with their file paths.

## Step 4: Iterate

After drafting a section, ask the user: "Does this capture it? Anything missing or wrong?"

Refine based on their feedback. Move to the next section when they're satisfied.

## Step 5: Save the Proposal

When the proposal is complete, save it to `docs/proposal/{slug}.md` (e.g., `docs/proposal/mvp-tienda-mecanica.md`).

If the `docs/proposal/` directory doesn't exist, create it — or ask the user where they want it.

## Step 6: Propose Next Steps

Once the proposal is saved, suggest what comes next:

- **"Want to start writing specs for any of these?"** → delegate to `spec-writer`. Pass the full "Planned specs" list from the proposal as context so `spec-writer` knows the full roadmap and can sequence them by dependency. Also pass the proposal file path so the spec can reference it.
- **"Need to resolve any of the pending decisions first?"** → explore them here
- **"Want to generate a diagram or ADR for any of the technical decisions?"** → note it for the artifacts stage

## What Not to Do

- Don't generate the entire proposal at once — collaborate section by section
- Don't force decisions that aren't ready — capture them as pending
- Don't make the proposal a requirements document — it's a structured brainstorm
- Don't skip the exclusions section — scope boundaries are essential
- Don't write specs here — just plan them. Specs come next.
