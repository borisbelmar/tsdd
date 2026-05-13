# TSDD Implementation Examples

Real-world case studies showing how TSDD applies across different team sizes and contexts.
Each example follows a fictional product through its lifecycle — from proposal to launch.

## Case Studies

| Scenario | Product | Guide |
|---|---|---|
| **Solo developer, personal project** | `logtail` — a CLI log viewer | [solo-developer-personal.md](solo-developer-personal.md) |
| **Solo developer, client with backlog** | AutoParts Store — auto parts e-commerce + booking | [solo-developer-client.md](solo-developer-client.md) |
| **Small team (2-3 devs)** | `Reserva` — SaaS appointment booking platform | [small-team.md](small-team.md) |
| **Large team (7-8 devs)** | `MercadoLocal` — multi-vendor marketplace | [large-team.md](large-team.md) |

## What Changes, What Doesn't

The **stages** are the same. What changes is the **ceremony level**:

| | Solo Personal | Solo + Client | Small Team | Large Team |
|---|---|---|---|---|
| Proposal | When fuzzy | Always for new areas | Always | Always |
| User Stories | Never | For client alignment | Optional | For stakeholders |
| Specs | Always | Always | Always | Always |
| Spec index | Todo list | Progress tracker | Coordination | Single source of truth |
| Artifacts | Rare | When client needs them | For decisions | For everyone |
| MR | Commit is enough | Commit + spec reference | Full MR | Full MR + artifacts |

## Skills Used

All case studies assume these skills are available:

| Skill | Role |
|---|---|
| `tsdd-init` | Project setup, context discovery, AGENTS.md |
| `proposal-writer` | Structured proposals for fuzzy problem spaces |
| `spec-writer` | Writing and maintaining executable specs |
| `spec-implement` | Implementing against a spec end-to-end |
| `agents-writer` | Creating and editing AGENTS.md |
