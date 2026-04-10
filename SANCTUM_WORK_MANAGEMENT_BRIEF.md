---
document-type: project-brief
project: sanctum-work-management
status: design-open
version: "0.1"
created-at: 2026-04-09
owner: operator
audience: milo
tags: [project, infrastructure, postgres, tracking, work-management]
summary: "Operator brief for a Sanctum-native work management system — structured project/effort/task tracking in Postgres with CLI tooling, replacing flat-file Kanban."
---

# Sanctum Work Management System — Project Brief

## For Milo — from the Operator, via intermediary

---

## The Problem

The Sanctum currently tracks projects, tasks, and to-dos in flat markdown files — Kanban boards, flag ledgers, the Future Board in `SANCTUM_STATE_OVERVIEW.md`. This works now. It will not work when ten or fifteen agent teams are running concurrent missions with interdependent work items that need to be referenced across journals, IRC, relay messages, commit messages, and handoffs.

Markdown is a ledger. What we need is a **system** — one where every unit of work has a canonical number, a parent, a state, and a history. One where an agent can write `TSK-0034` in a journal entry and six months later anyone can trace that back to a specific task under a specific effort under a specific project. Provenance for work, not just for identity.

---

## The Model

The Operator's reference point is ServiceNow's work hierarchy, specifically:

- **RITM (Requested Item)** — the top-level initiative. "We are installing a DNS server." Everything underneath references this.
- **Change Record** — a scoped effort within the initiative. "Configure the zone files." "Set up replication." Each one is a coherent chunk of work with its own lifecycle.
- **Task** — the atomic unit. Someone picks it up, does it, closes it. "Write the forward zone for thesanctum.tech."

Three tiers. Each with a unique identifier. Each referencing its parent. The RITM is the *why*. The change is the *what*. The task is the *do*.

**What we are NOT building:** ServiceNow. No workflows, no approval chains, no SLA engines, no UI dashboards. The structure is the point — the numbering, the hierarchy, the queryability. Everything else is overhead we don't need.

---

## The Sanctum Translation

ServiceNow's vocabulary is enterprise IT. Ours doesn't need to be. The structure stays; the language belongs to us. Proposed mapping (open for design — Milo should weigh in):

| ServiceNow | Sanctum Equivalent | Description |
|------------|-------------------|-------------|
| RITM | **Project** | The initiative. Why we are doing work. Has a name, a description, an owner, a status. |
| Change Record | **Effort** | A scoped chunk within a project. Coherent enough to hand to a unit or an agent. Has its own lifecycle. |
| Task | **Task** | The atomic action. Someone does this. It opens, it closes. |

Each tier gets a **sequential identifier** — the numbering scheme is an open design decision (see below).

---

## What Already Exists

- **Postgres on Arcana** — `sanctum_feed` schema proves the pattern. We know how to build real data infrastructure here.
- **CLI tooling patterns** — the flag system (`add-flag.sh`, `update-flag.sh`, `list-flags.sh`) and journal skill (`journal-entry.sh`) are working examples of agent-facing CLI tools backed by structured data.
- **Agents comfortable with structured workflows** — every agent already works with append-only logs, timestamped entries, and canonical identifiers (agent-ids, flag-ids).
- **IRC and relay** — downstream integration targets. Not required at launch, but the system should assume that agents will want to query and update work items from within their normal communication channels.

---

## What Needs to Be Built

### 1. Schema

Postgres tables for the three tiers. At minimum:

**Projects**
- `project_id` (sequential, formatted — e.g., `PRJ-0001`)
- `title`
- `description`
- `owner` (agent-id or `operator`)
- `status` (enum: `open`, `active`, `blocked`, `complete`, `archived`)
- `created_at`, `updated_at`
- `created_by`

**Efforts**
- `effort_id` (sequential, formatted — e.g., `EFF-0001`)
- `project_id` (FK → projects)
- `title`
- `description`
- `assignee` (agent-id, unit, or null)
- `status` (same enum or similar)
- `created_at`, `updated_at`
- `created_by`

**Tasks**
- `task_id` (sequential, formatted — e.g., `TSK-0001`)
- `effort_id` (FK → efforts)
- `project_id` (FK → projects, denormalized for query convenience)
- `title`
- `description`
- `assignee` (agent-id or null)
- `status` (enum: `open`, `in_progress`, `blocked`, `done`, `cancelled`)
- `created_at`, `updated_at`, `closed_at`
- `created_by`

**History / audit log** — every status change, reassignment, or edit recorded. Append-only. The Chronicler will care about this.

### 2. CLI Tools

Shell scripts or lightweight wrappers that agents can invoke from their harness. Same pattern as the flag system. Minimum viable set:

- `work-create.sh project|effort|task` — create a new item, returns the assigned ID
- `work-update.sh <ID> --status <status>` — change status (and other fields)
- `work-list.sh [--project PRJ-XXXX] [--status open] [--assignee sam]` — query with filters
- `work-show.sh <ID>` — display a single item with its full context (parent chain, children, history)
- `work-close.sh <ID>` — close a task/effort/project with a closing note

### 3. Integration Points (downstream — not required at launch)

- IRC: `/work TSK-0034` returns a summary in-channel
- Journal: agents can reference work IDs in journal entries; the ID is a hyperlink in the record
- Relay: work item creation/closure events posted to a channel
- Sam: the Chronicler indexes work items as part of the historical record

---

## Open Design Decisions

These are the questions that need answers before schema is written. Milo should think about these and bring recommendations.

| ID | Question | Notes |
|----|----------|-------|
| WM-001 | **Numbering scheme** — Flat sequential (`SANCTUM-0001`) or tier-prefixed (`PRJ-001`, `EFF-001`, `TSK-001`)? Tier-prefixed is more readable at a glance. Flat is simpler. | Operator leans toward tier-prefixed. |
| WM-002 | **Schema name** — New Postgres schema (`sanctum_work`?) or extend `sanctum_feed`? | Probably new schema. Clean separation. |
| WM-003 | **Cross-references** — Should items be able to reference each other laterally (e.g., `TSK-0012 blocked by TSK-0008`)? Adds complexity but is genuinely useful. | Start without, design the column for later? |
| WM-004 | **Where do the CLI tools live?** — `sanctum-memory/shared/skills/work-management/`? Same pattern as flags and journal. | Needs a SKILL.md. |
| WM-005 | **Relationship to existing tracking** — Do we migrate the current Kanban/flag items into this system, or start fresh and let the old system age out? | Operator preference TBD. |
| WM-006 | **Permissions** — Can any agent create a project, or only the Operator? Can agents create their own tasks within an effort they're assigned to? | Probably: Operator creates projects, anyone creates tasks within their scope. |

---

## Constraints

- **Postgres on Arcana.** Not SQLite, not flat files. This is infrastructure that needs to scale and be queryable.
- **CLI-first.** Agents interact through shell tools, not a web UI. The interface is the terminal.
- **No over-engineering.** Three tables, an audit log, and a set of shell scripts. We can add complexity when the simple version proves insufficient. Build by doing.
- **The identifiers are the point.** The entire value of this system is that every piece of work gets a citable, traceable, permanent number. Everything else is secondary to that.

---

## Success Criteria

The system is working when:

1. An agent can create a task, receive a number, and reference that number in a journal entry or IRC message.
2. The Operator can query "show me everything open under PRJ-0003" and get a clean answer.
3. Sam can look at a closed project six months from now and reconstruct what happened, in what order, by whom.
4. The overhead of using the system is lower than the overhead of not having it.

---

*Brief prepared 2026-04-09 by the Operator's hand, channeled through intermediary. Milo: build by doing. The generator comes after you've watched the process work.*
