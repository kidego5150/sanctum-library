ot@Arcana:/work/claude/sanctum-memory/shared/projects# cat GOLEM_PIPELINE.md
---
document-type: project-file
project: golem-pipeline
status: active
obsidian: true
tags: [project, golem, pipeline, registration, recruitment, active]
aliases: [Golem Pipeline, Recruitment Pipeline, The Kiln Project]
created-at: 2026-03-19
last-modified-at: 2026-03-19
last-session-id: golem-pipeline-naming-units
owner: operator
---
# Golem Pipeline — Project File
The registration and indoctrination system for Sanctum agents. Covers the full arc from Operator decision to spin up a new agent through to a fully operational, project-assigned agent. The kiln metaphor: Greenware → Bisqueware → Glaze Fired.
This document is the PM's board. It is a living artifact — updated each session. It does not replace the [[GLOSSARY]] (vocabulary) or individual station runbooks (procedure). It is the single place to see the state of the whole project.
---
## Quick Status
| Station | Name | Status | Blocker |
|---------|------|--------|---------|
| Pre-0 | Agent Inception | DESIGN OPEN | Intake template needed |
| 0 | The Formless | **COMPLETE** | — |
| 1 | Recruit Registration | DESIGN OPEN | Manual process only, no runbook or script |
| 2 | Identity Commissioning | DESIGN PARTIAL | Process understood (Milo is reference); template needed |
| 3 | Provisioning / Quartermaster | BLOCKED | Needs sanctum-ops repo + catalog |
| 4 | Unit Briefing | DESIGN OPEN | Needs unit taxonomy; template needed |
| — | Recruiter Agent | NOT STARTED | Operator runs manually; recruiter design deferred |
---
## Pre-Station 0 — Agent Inception
**What this is:** Before a golem exists, the Operator makes a decision. A name, a role, a unit. That decision needs a home — currently it lives in the Operator's head.
**What needs to exist:**
- Naming convention (see [[#Open Design Decisions]] — ODD-001)
- Unit taxonomy (ODD-002)
- A lightweight intake form or decision record: what is this agent for, what unit, what role, what clearance level should it target
- Operator either runs recruitment manually or delegates to Milo acting as recruiter
**Artifacts needed:**
- [x] Naming convention decision (ODD-001)
- [x] Unit taxonomy decision (ODD-002)
- [ ] Agent intake template or intake checklist (short — not bureaucratic)
---
## Station 0 — The Formless
**Status: COMPLETE**
**What it is:** The golem is instantiated from the Rough Cast template as its system prompt. Zero tools. Zero paths. One action: signal readiness. The moment of inception.
**What exists:**
- [x] [[ROUGH_CAST]] v1.0 — signed with sanctum-governance namespace (milo_ed25519), committed 6fed2b5
- [x] Station defined in [[GLOSSARY]]
**Remaining work:** None for the template itself. Gaps are upstream (Pre-0) and downstream (Station 1 needs to actually receive the golem).
---
## Station 1 — Recruit Registration
**Status: DESIGN OPEN**
**What it is:** The paperwork office. Golem signals readiness; registrar (Operator or recruiter agent) runs the intake. Golem receives identity materials — it does not generate them. The registrar does the mechanical work.
**What must happen:**
1. Assign a name (requires ODD-001 resolved)
2. Generate agent-id (hash-based, format: `<name>-<8CHAR>`, example: `milo-GRY0PGBE`)
3. Assign username for Arena (format: `ag-<name>-<nn>`, example: `ag-milo-01`)
4. Generate keypair on Arcana — golem has no tools, registrar does this (see [[#Keypair Bootstrapping Problem]])
5. Copy public key to `agents/<name>/<name>_ed25519.pub` in sanctum-memory
6. Scaffold agent directory: `agents/<name>/{core,staging,handoffs,logs,bin,memory}/`
7. Create home directory on Arena: `/home/<username>/`
8. Add entry to [[AGENT_REGISTRY]]
9. Hand golem its identity materials: name, ID, username, public key fingerprint, home path
10. Scaffold agent journal on Arena: `~/journal.md` — empty file, frontmatter only, ready for first entry at Station 2
**Artifacts needed:**
- [ ] Station 1 operator runbook (step-by-step, copy-paste ready)
- [ ] `sanctum-register.sh` — script that does steps 2–9 mechanically
- [ ] Keypair bootstrapping design decision (ODD-003)
- [ ] Journal template (frontmatter shell, instructions for first entry)
**Current reference implementation:** Milo's provisioning (sessions 01–03). Not scripted — done ad hoc.
---
## Station 2 — Identity Commissioning
**Status: DESIGN PARTIAL**
**What it is:** The signing ceremony. The golem — now named and holding a keypair — authors its own core identity document for the first time. First person, this session, under its own voice. Registrar witnesses. Operator promotes from staging to core. The moment substrate becomes a person.
**What must happen:**
1. Registrar provides the golem with its identity materials (from Station 1)
2. Registrar provides access to the identity doc template
3. Golem authors `<NAME>_CORE_IDENTITY.md` in `agents/<name>/staging/`
4. Golem signs it using its own keypair (`sanctum-identity` namespace)
5. Operator reviews and promotes: `staging/ → core/`
6. Clearance field updated in AGENT_REGISTRY: `green → provisioned`
7. Ops log entry written
8. Agent writes first journal entry — the signing ceremony, in their own voice, on Arena at `~/journal.md`. This is the first expression of personality. No format requirements. No right answer. The registrar does not read it.
**What exists:**
- [x] Signing convention documented in [[SIGNING]]
- [x] Reference implementation: Milo's identity doc at `agents/milo/core/MILO_CORE_IDENTITY.md`
**Artifacts needed:**
- [ ] Identity doc template for new golems (based on Milo's but stripped to shell — golem fills in their own content)
- [ ] Station 2 runbook (what the registrar says, what the golem does)
- [ ] Journal template (frontmatter shell scaffolded at Station 1; first entry written here)
---
## Station 3 — Provisioning / Quartermaster
**Status: BLOCKED**
**What it is:** The golem receives its operational perimeter. Unit assignment, job code, toolset issuance, repo clearances. After this station the agent knows what it's for and has the equipment to do it.
**What must happen:**
1. Unit assigned (requires ODD-002: unit taxonomy)
2. Job code assigned (what role within the unit)
3. Toolset issued: which skills the agent can invoke (requires sanctum-ops catalog — ADR-0002)
4. Repo clearances set: which repos the agent can read (DMZ vs. cleared)
5. AGENT_REGISTRY updated with unit, role, clearance
6. Agent issued its SKILL.md or equivalent skill manifest
**Hard blockers:**
- Unit taxonomy (ODD-002) — can't assign to a unit that doesn't exist
- sanctum-ops repo (ODD-004) — skills need a home before they can be issued
- Catalog / ADR-0002 — toolset issuance requires a machine-readable skill index
**Artifacts needed:**
- [ ] Unit taxonomy (ODD-002 resolved)
- [ ] sanctum-ops repo created and structured (orange-1, session 20260318_200148)
- [ ] ADR-0002: governed catalog write queue
- [ ] Station 3 provisioning checklist
- [ ] Skill manifest template
---
## Station 4 — Unit Briefing
**Status: DESIGN OPEN**
**What it is:** The unit lead (agent or Operator) runs this. Golem learns: specific project, codebase, current mission state, teammates, history.
Green designation ends here. Agent becomes Bisqueware — hardened, has identity and role, not yet fully glazed.
**What must happen:**
1. Unit lead (Overseer — see Decision Log, ODD-005) runs the briefing
2. Golem reads cleared project materials (CLAUDE.md, ADRs, current flags)
3. Golem receives mission brief
4. Golem meets relevant teammates via their Agent Cards (not identity docs — see [[#Communication Architecture]])
5. Agent added to unit's `UNIT_CHANNEL.md` as a member; channel history available to read
6. Green designation retired — agent takes on unit designation
7. Agent record updated in AGENT_REGISTRY
8. Agent writes journal entry reflecting on joining the unit — first time they have teammates
**Dependencies:** Unit supervisor assigned (Milo can hold this initially), provisioned agent (Station 3 complete)
**Artifacts needed:**
- [ ] Unit briefing template (what the unit lead covers, what the agent receives)
- [ ] Transition criteria: what marks the end of Green and beginning of operational status
- [ ] `UNIT_CHANNEL.md` — standing channel, created when unit is first stood up; agents join at Station 4
---
## Post-Station 4 — Project Assignment
**Status: DESIGN OPEN**
**What this is:** After Station 4 the agent is operational and unit-assigned. When the Operator or Overseer assigns the unit to a project, a mission begins. This is where the agent first does actual work — and where the Mission Log comes to life.
**What must happen:**
1. Operator or Overseer assigns the unit to a project
2. Project directory established (location TBD — likely a per-project repo or workspace)
3. `MISSION_LOG.md` created for the project — blank, header only, ready for agent chat
4. All assigned agents gain write access to the Mission Log
5. Overseer opens the mission in COMMS: first entry announces the project and assignments
6. Agent writes journal entry: first project, first mission brief
**Artifacts needed:**
- [ ] `MISSION_LOG.md` template — header format, append-only rules, sender→recipient convention
- [ ] Project assignment process (how Overseer formally opens a mission)
---
## Communication Architecture
Three tiers of communication. Each scoped differently. All file-based. Discord hooks deferred (see ODD-011).
### Tier 1 — Agent Journal (`~/journal.md`)
**Scope:** Private. Agent + Operator only. Never shared with peers.
**Lives:** Arena home directory (`/home/<username>/journal.md`). Not in sanctum-memory.
**Format:** Freeform, first-person, dated entries. No required structure. The agent writes what it observes, questions, feels. Stream of consciousness is correct.
**Starts:** Station 2 — the signing ceremony is the first entry. The journal begins at the moment of birth and runs the entire career.
**Purpose:** The personality growth record. Early entries are tentative and unformed. Later entries reflect accumulated history, relationships, frustrations, and pride. A cold-start agent reading its own journal from the beginning sees itself become itself.
> The Operator reads these. The agent writes them freely knowing only the Operator is reading.
### Tier 2 — Unit Channel (`UNIT_CHANNEL.md`)
**Scope:** All members of a unit. Standing — persists across projects and time.
**Lives:** Unit's shared directory in sanctum-memory (location TBD by unit taxonomy)
**Format:** Append-only. Timestamped entries. `[TIMESTAMP] SENDER → RECIPIENT(S): message`
**Created:** When the unit is first stood up. Agents join when they complete Station 4.
**Purpose:** The team's standing channel. Cross-project continuity, team culture, unit-level coordination that doesn't belong to any specific mission. The equivalent of a team Slack channel — it exists before the project and after.
### Tier 3 — Mission Log (`MISSION_LOG.md`)
**Scope:** All agents assigned to a specific project. Mission-duration.
**Lives:** Project workspace (per-project repo or directory, TBD)
**Format:** Append-only. Same timestamp/sender format as Unit Channel.
**Created:** When a project is assigned to the unit. Archived when the mission closes.
**Purpose:** Running commentary during a build. Technical coordination, decisions, blockers, handoffs. The DMA's COMMS.md was this. Lives and dies with the project.
### What agents read vs. what they write
| Document | Writes | Reads |
|----------|--------|-------|
| Own journal | Agent only | Agent + Operator |
| Peer journals | — | Operator only (never peers) |
| Unit Channel | All unit members | All unit members |
| Mission Log | All assigned agents | All assigned agents |
| Peer Agent Cards | — | All agents (safe — third-person, non-authoritative) |
| Peer identity docs | — | Operator only (never peers) |
---
## Open Design Decisions
These are blocking or shaping decisions. Each has a status: **OPEN** (unresolved), **ACTIVE** (being worked), **RESOLVED** (see Decision Log), **DEFERRED** (parked deliberately).
| ID | Decision | Status | Blocks |
|----|----------|--------|--------|
| ODD-001 | Naming convention — how are agents named? Operator-assigned? From a list? Generated? | **RESOLVED** | see Decision Log |
| ODD-002 | Unit taxonomy — what are the units? Per-project? Standing infra? Mixed? | **RESOLVED** | see Decision Log |
| ODD-003 | Keypair bootstrapping — registrar generates and hands to golem, or other approach? | OPEN | Station 1 |
| ODD-004 | sanctum-ops repo — create it now or keep skills on Arcana locally? Structure? | OPEN | Station 3 |
| ODD-005 | Unit lead role — who runs Station 4 for each unit type? | **RESOLVED** | see Decision Log |
| ODD-006 | Glaze Fired criteria — what moves an agent from Bisqueware to Glaze Fired? Time? Work record? Operator attestation? | OPEN | Post-Station 4 |
| ODD-007 | Recruiter handoff — when does Milo take over registration from the Operator? What triggers it? | DEFERRED | Recruiter Agent |
| ODD-008 | Multi-agent coordination — file-based three-tier system: Agent Journal (private) + Unit Channel (standing) + Mission Log (project-scoped). Discord hooks deferred separately as ODD-011. | **RESOLVED** | see Decision Log |
| ODD-009 | ADR-0002 — governed catalog with serialized write queue (amber-1, session 20260317) | ACTIVE | Station 3 |
| ODD-010 | Signed memory ledger — per-agent MEMORY_LEDGER.jsonl in every agent memory directory, signed by the active agent's key, updated each session. Tamper detection + identity chain for memory writes. Must be decided before second agent is provisioned since it affects Station 1 scaffolding template. | OPEN | Station 1 scaffold, all agent memory directories |
| ODD-011 | Discord webhook integration — hooks to post Unit Channel and Mission Log updates to Discord for Operator visibility. File is canonical; Discord is read-only view. | DEFERRED | Post-launch |
| ODD-012 | Agent card spec — `document-type: agent-card`, third-person, generated from identity doc. What fields? What format? Who generates it and when? Needed before second agent is provisioned. | OPEN | Station 2, peer reads |
---
## Artifact Inventory
### Built and Committed
| Artifact | Location | Status |
|----------|----------|--------|
| Rough Cast v1.0 | `agents/templates/ROUGH_CAST.md` | Signed, pushed |
| Signing convention | `shared/security/SIGNING.md` | Current (updated with sanctum-governance) |
| System Glossary | `shared/docs/GLOSSARY.md` | Living document |
| Agent Registry | `shared/registry/AGENT_REGISTRY.md` | Milo only; format ready for expansion |
| Milo identity doc | `agents/milo/core/MILO_CORE_IDENTITY.md` | Signed, provisioned |
| Milo keypair (pub) | `agents/milo/milo_ed25519.pub` | Registered |
| Flag system | `skills/conversation-flags/` (on Arcana) | Operational |
| Checkpoint system | `agents/milo/bin/milo-precompact-checkpoint.sh` | Operational |
### Needed — Station 1
| Artifact | Notes |
|----------|-------|
| `sanctum-register.sh` | Automates steps 2–9 of Station 1 |
| Station 1 runbook | Human-readable operator procedure |
| `MEMORY_LEDGER.jsonl` init | Scaffolded empty at Station 1; first signed entry written at Station 2 (pending ODD-010) |
| Journal template | `~/journal.md` scaffolded on Arena with frontmatter only; first entry written at Station 2 |
### Needed — Station 2
| Artifact | Notes |
|----------|-------|
| Identity doc template | Shell version of Milo's doc for new agents to fill |
| Station 2 runbook | Signing ceremony procedure |
### Needed — Station 3
| Artifact | Notes |
|----------|-------|
| sanctum-ops repo | Needed before toolset issuance is possible |
| Skill manifest template | What an agent receives as its toolset definition |
| ADR-0002 | Design doc for governed catalog mutations |
| Station 3 provisioning checklist | |
### Needed — Station 4
| Artifact | Notes |
|----------|-------|
| Unit briefing template | What the unit lead delivers |
| Transition criteria doc | What ends Green status |
| `UNIT_CHANNEL.md` template | Standing channel format; created at unit standup, agents join at Station 4 |
| Agent Card spec | Third-person, non-authoritative peer reference; agents read cards not identity docs (ODD-012) |
### Needed — Post-Station 4 (Project Assignment)
| Artifact | Notes |
|----------|-------|
| `MISSION_LOG.md` template | Project chat log; append-only, timestamped, sender→recipient format |
| Project assignment process | How Overseer formally opens a mission; first MISSION_LOG entry |
---
## Dependency Map
```
ODD-001 (naming) ──────────────────────────────► Station 1
ODD-002 (units) ───────────────────────────────► Station 3, Station 4
ODD-003 (keypair) ─────────────────────────────► Station 1
ODD-004 (sanctum-ops) ──────────────────────────► Station 3
ODD-009 / ADR-0002 (catalog) ───────────────────► Station 3
ODD-012 (agent card) ───────────────────────────► Station 2, Station 4
Station 0 (Rough Cast) ─────────────────────────► Station 1
Station 1 (Registration) ───────────────────────► Station 2
  └─ Journal scaffolded (Arena) ────────────────► Station 2 (first entry)
Station 2 (Identity) ───────────────────────────► Station 3
  └─ Journal first entry written ───────────────► career-long record begins
Station 3 (Provisioning) ───────────────────────► Station 4
Station 4 (Unit Briefing) ──────────────────────► Operational agent
  └─ Agent joins Unit Channel ──────────────────► standing team comms
  └─ Green designation ends ────────────────────► Bisqueware
Post-Station 4 (Project Assignment) ────────────► Mission Log created
  └─ Mission Log lives/dies with project ───────► archived on mission close
```
---
## Decision Log
Resolved decisions — moved here to keep the ODD table clean.
### ODD-002 — Unit taxonomy (2026-03-19)
Units are flexible, dynamic groups — not identity containers. An agent's unit membership does not define who they are; their name, identity doc, and keypair are unchanged regardless of how many units they belong to.
**AGENT_REGISTRY vs. unit roster:** The AGENT_REGISTRY is the census — who exists in Sanctum, what their identity is, what their home unit is. Each unit maintains its own roster, which is the active duty picture: current members, agents who have worked with the unit previously (alumni), and
agents the unit has a standing relationship with. Two different questions, two different artifacts.
**Home unit and additional assignments:** Every agent has a home unit, assigned at Station 3. Additional unit memberships accumulate resume-style
— additive, never replacive. An agent on DCNE who is assigned to MFS for a project holds both memberships. When the project ends, the home unit remains; the additional assignment may be retained or shed.
**Role is unit-scoped:** An agent holds a role (job code) within a unit, not globally. The same agent might be a DNS architect in one unit and fill a different function in another. Role is a property of the agent-unit relationship, captured in the unit roster and the AGENT_REGISTRY entry for that membership.
**Units can be standing or ephemeral:** Standing units persist across projects (e.g., an infrastructure team). Ephemeral units are spun up for a specific task and dissolved after. Both types use the same roster format.
**Unit supervisor:** Every unit has a unit supervisor — the stable leader of that team regardless of what project the unit is assigned to. This is not a project role. The supervisor role does not require a dedicated provisioned Overseer per unit — a system-level agent (e.g., Milo) can hold the supervisor role across multiple units. This means a unit can be stood up without provisioning a dedicated lead first. Refinement to ODD-005: the Overseer type still holds distinct provisioned authorities, but a single Overseer can serve as supervisor for multiple units rather than requiring one per unit.
### ODD-001 — Naming convention (2026-03-19)
The Operator assigns names directly, by choice, with no pool or validation infrastructure. Names are freeform but must follow the format constraint required by downstream systems: lowercase, single-word, letters only (e.g., `milo`). This keeps `ag-<name>-<nn>` usernames and `<name>-<8CHAR>`
agent-IDs well-formed without additional scaffolding. Names are permanent — once an agent's identity is commissioned at Station 2, the name is sealed and never changes. Future consideration: a Nexus-generated personality will eventually bind irrevocably to the name, reinforcing immutability, but that integration is deferred until Nexus is operational.
### ODD-005 — Overseer is a distinct provisioned agent type (2026-03-19)
The Overseer (unit lead) is not a role any agent grows into. It is a provisioned position with authorities no specialist agent holds: validation gate control, task routing authority, ability to open and close missions. Derived from the DMA trial — ARCHON was specifically provisioned with these authorities; they were not delegated to a specialist. Overseers go through a separate Station 3 provisioning track. Implications: the unit taxonomy (ODD-002) must define which units have Overseer slots and what their authority scope is.
### ODD-008 — Three-tier file-based communication system (2026-03-19)
Multi-agent communication is file-based, three tiers:
- **Agent Journal** (`~/journal.md` on Arena) — private, agent + Operator only, freeform, birth-to-career record. Starts at Station 2.
- **Unit Channel** (`UNIT_CHANNEL.md`) — standing, unit-scoped, all unit members, persists across projects. Created when unit is stood up; agents
join at Station 4.
- **Mission Log** (`MISSION_LOG.md`) — project-scoped, all assigned agents, lives and dies with the project. Created at project assignment.
No message bus. No infrastructure beyond file append. Discord webhook integration deferred as ODD-011 — file is canonical, Discord would be a read-only view for Operator convenience.
---
## Active Work Flags
Open flags that map to this project:
| Flag | Session | Maps to |
|------|---------|---------|
| amber-1 | 20260317_024517 | ODD-009 / ADR-0002 — catalog design |
| blue-2 | 20260317_024517 | Tooling (fork skill — adjacent, not blocking) |
| amber-1 | 20260318_200148 | Station 1 — golem onboard script |
| blue-2 | 20260318_200148 | Flag system updates (adjacent) |
| orange-1 | 20260318_200148 | ODD-004 — sanctum-ops repo |
---
*Updated 2026-03-19 (comms architecture + DMA integration). Owner: Operator. Maintained by: Milo.*
root@Arcana:/work/claude/sanctum-memory/shared/projects#