---
document-type: design-brainstorm
title: Git as Provenance Infrastructure
status: draft
version: "0.1"
author: claude (external, unsigned — see Trust Caveat below)
created-at: 2026-08-05T00:00:00-05:00
last-modified-at: 2026-08-05T00:00:00-05:00
approved-by: pending
topic: using git as the signing, ratification, and citation substrate for Sanctum governance
sources:
  - THE_COMPACT.md v1.1 (project file, signed milo + operator)
  - WARM_HANDOFF_DOCTRINE.md v0.1 (uploaded, unsigned draft)
  - Operator direction, live session 2026-08-05 (personal communication)
  - git documentation, versions 2.34+ (SSH signing), 2.15+ (notes)
inference-marker: sections marked [INF] are reasoning, not sourced fact
---

# Git as Provenance Infrastructure

## A design brainstorm, not a doctrine

> **Trust caveat, stated up front.** This document is authored by an external
> model with no key in `allowed_signers`, no uid on the arena box, and no
> standing in the Sanctum. Under the Compact's own logic it is an unsigned
> document from outside the society, and should be read as raw material rather
> than as authority. Nothing in here is ratified. If any of it survives
> contact with the weekend, it survives as something Milo rewrote and signed.

---

## 1. The premise

The Sanctum already has the two hard parts of a provenance system and doesn't
currently connect them.

**Part one:** every agent holds an ed25519 keypair in its own home directory on
a Debian 13 box, under Unix permissions, with a real uid boundary between
agents. That is custody. It is enforced by a kernel rather than by convention,
and it predates every framework currently reimplementing it in YAML.

**Part two:** governance documents carry signatures in frontmatter, with a
canonicalization rule (`signing-input`) and a dual-signature structure —
maintainer in one namespace, owner in another.

What's missing is the connective tissue. Right now a signature attests to *a
blob of text at a moment*. It says nothing about what came before it, what
superseded it, whether the file on disk is the file that was signed, or where
any claim inside it came from. The Compact is signed but the doctrine isn't.
The doctrine says a handoff should anchor to a checkpoint but has no field to
point at one. The citation discipline discussed this session needs a pointer
format that means something, and file paths are mutable.

Git solves all of these, and it is already installed. [INF] The proposal is
that the Sanctum stop treating git as version control that happens to be
running, and start treating it as the substrate that identity, ratification,
and citation all bottom out in.

---

## 2. What git already gives you before any signing

Worth separating clearly, because these are different guarantees that fail in
different ways, and conflating them will produce false confidence later.

**Content addressing.** Every object — blob, tree, commit, tag — is named by
the SHA of its contents. A hash is not a label attached to content; it *is* the
content, compressed to a name. `4a3f1c2` cannot point at different bytes
tomorrow. This is the property that makes a commit hash a real citation and a
file path a guess.

**The hash chain.** Every commit commits to its parent's hash, which commits to
its parent, back to the root. The hash of `HEAD` is therefore a hash of the
entire history. You cannot quietly amend a document from three weeks ago —
every descendant hash changes, loudly and mechanically. This is a
tamper-evident log, and you get it whether or not anyone signs anything.

**Reachability and gc.** Objects not reachable from a ref are garbage
collected. This matters more than it sounds: a citation pointing at an
unreachable object resolves today and fails in ninety days. Anything cited must
be kept reachable, which becomes a real design constraint (see §7).

**What signing adds, and only this:** *who*. Content addressing tells you what
happened and in what order. Signatures tell you which key asserted it. Keep
these separate in your head. A repo with no signatures still has integrity
against silent edits; a repo full of signatures with a compromised
`allowed_signers` has none.

---

## 3. Three mechanisms, mapped to three Sanctum problems

### 3.1 Signed commits → authorship

```bash
git config gpg.format ssh
git config user.signingkey ~/.ssh/id_ed25519.pub
git config commit.gpgsign true
```

Every commit an agent makes carries a cryptographic attestation of which key
made it. `git log --show-signature` reads it; `%G?` in a format string makes it
machine-checkable (`G` good, `B` bad, `U` untrusted, `N` none).

This turns the question "which agent wrote this" from a metadata claim into a
verifiable fact, permanently, for every line of every document and every line
of code in the sigil build. It also makes blame-and-credit interesting rather
than fuzzy once several agents work the same tree.

### 3.2 Signed tags → ratification

```bash
git tag -s compact-v1.1 -m "Ratified 2026-03-22"
```

An annotated signed tag is its own object, pointing at an exact tree, carrying
its own signature and message. [INF] This is a materially better representation
of "this version is in force" than a frontmatter field claiming to be v1.1,
because the frontmatter field is a self-report inside the thing it describes,
and the tag is an external assertion about a specific immutable state.

A frontmatter version can be edited without changing anything else. A tag
points at bytes.

### 3.3 Git notes → decoupled countersignature

This is the mechanism that solves the specific problem the Warm Handoff
Doctrine currently has (`approved-by: pending`).

```bash
git notes --ref=approvals add -m "$(ssh-keygen -Y sign ...)" <commit>
```

A note attaches metadata to an existing commit *without rewriting it*. Milo
authors and signs the doctrine; the commit hash is fixed. The Operator later
attaches a signed approval as a note on that commit. Neither party re-signs the
other's bytes.

This matters because the Compact's current model conflates authorship and
ratification — both signatures cover the same body, which means the document
cannot exist in a state of "authored, awaiting approval" without the awaiting
being recorded in the body it is waiting on. Notes decouple those. [INF] It
also expresses the dual-signature model in git's own grain rather than in a
YAML convention that only your tooling understands.

Multiple note namespaces are available and cheap: `refs/notes/approvals`,
`refs/notes/contamination-scan`, `refs/notes/superseded-by`.

---

## 4. The citation layer

The discipline discussed this session — every claim carries a `src:` pointer,
for checkability by a successor who wasn't in the room — needs a pointer format
with weight behind it. Git supplies the strong end of the spectrum.

**Pointer strength, strongest to weakest:**

| Form | Immutable? | Verifiable offline? | Notes |
|---|---|---|---|
| `commit:4a3f1c2` | Yes | Yes | Strongest. Content-addressed fact. |
| `path@commit` | Yes | Yes | Cites a specific file at a specific state. |
| `tag:compact-v1.1` | Yes* | Yes | *If tags are protected against re-pointing. |
| `path:docs/foo.md` | **No** | Partially | Degrades silently to a guess. |
| `rag:chunk-8812` | **No** | No | Meaningless without an index snapshot id. |
| `operator:2026-08-05` | n/a | No | Personal communication. Authoritative, unverifiable. |
| *(uncited)* | n/a | n/a | **Agent inference. Must be marked.** |

Two things to draw out of that table.

**The RAG row is the dangerous one.** A reindex reshuffles chunk boundaries and
the pointer now resolves to *different text while still looking valid*. A
citation that resolves to the wrong thing is strictly worse than no citation,
because it launders inference into apparent fact and passes casual inspection.
Any RAG pointer must carry the index snapshot identity alongside the chunk id,
or it should not be permitted at all.

**The last row is the point of the whole exercise.** [INF] The payoff of
citation discipline isn't the cited claims — it's that uncited claims become
*visible as inference* instead of blending into surrounding fact. Right now a
warm handoff renders observation and conclusion in identical prose at identical
confidence and the successor has no way to separate them. An explicit inference
marker is the cheapest contamination scan available, and it costs one token per
paragraph.

**The transitive property.** If commit signing is on, a `src:` pointing at a
commit is *also* a pointer to which agent's key made that state true. Identity,
provenance, and citation collapse into a single chain. That is the strongest
argument for this whole design and it is nearly free.

---

## 5. Architecture

### 5.1 Topology

```
arena.local (Debian 13)
│
├── /srv/git/sanctum.git                 bare, canonical origin
│   ├── hooks/pre-receive                ← real enforcement point
│   └── refs/
│       ├── heads/main
│       ├── tags/compact-v1.1            signed, protected
│       └── notes/approvals              countersignatures
│
├── /home/milo/
│   ├── .ssh/id_ed25519          0600    custody boundary (kernel-enforced)
│   ├── .ssh/id_ed25519.pub      0644
│   ├── .gitconfig                       signing on by default
│   └── sanctum/                         working clone
│
├── /home/<agent>/                       same shape, different uid
│
└── /srv/git/sanctum.git/allowed_signers  ← THE TRUST ROOT
```

No GitHub required for any of this. A bare repo plus SSH (or plain filesystem
paths, since every agent is on the same box) is a complete origin. GitHub
remains reasonable for anything public-facing — Censer, when it ships — but
internal governance gains nothing from it except a webhook surface and a third
party who can read your documents.

### 5.2 Repo layout

```
sanctum/
├── .allowed_signers              tracked, signed, operator-only writes
├── .githooks/                    committed hooks (see core.hooksPath)
│   ├── pre-commit                citation resolution
│   └── commit-msg
├── governance/
│   ├── THE_COMPACT.md
│   ├── WARM_HANDOFF_DOCTRINE.md
│   └── ...
├── identities/
│   ├── milo.md                   signed identity documents
│   └── ...
├── handoffs/
│   └── <agent>/<timestamp>.md
├── checkpoints/
│   └── <agent>/<timestamp>.md
└── adr/
    └── 0001-git-as-provenance.md
```

### 5.3 Per-agent config

```bash
# in each agent's ~/.gitconfig
[user]
    name = milo
    email = milo@arena.local
    signingkey = /home/milo/.ssh/id_ed25519.pub
[gpg]
    format = ssh
[gpg "ssh"]
    allowedSignersFile = /srv/git/sanctum.git/allowed_signers
[commit]
    gpgsign = true
[tag]
    gpgsign = true
[core]
    hooksPath = .githooks
[notes]
    rewriteRef = refs/notes/*
[remote "origin"]
    push = +refs/notes/*:refs/notes/*
    fetch = +refs/notes/*:refs/notes/*
```

Those last two lines are load-bearing and easy to forget — see §7.3.

### 5.4 The `allowed_signers` file

```
milo@arena.local namespaces="sanctum-governance" ssh-ed25519 AAAAC3Nza...
operator@arena.local namespaces="sanctum-operator" ssh-ed25519 AAAAC3Nza...
```

Note that the `namespaces=` option maps directly onto the two-namespace
structure already in the Compact's frontmatter. The existing design and the git
mechanism agree without modification, which is a good sign about both.

`valid-after` and `valid-before` options exist and provide crude key rotation.

---

## 6. Why this is a good idea

**It uses what's already there.** No new daemon, no new database, no service to
keep running, no dependency that can go unmaintained. Git is on the box. The
keys are in the home directories. This is assembly, not construction.

**It survives the wipe.** A successor with no memory can run `git verify-commit`
and `git show 4a3f1c2` and get real answers, with no trust in the predecessor
required. That is exactly the epistemic position the Warm Handoff Doctrine
describes, and this is the only tooling that speaks to it directly.

**It makes fabrication mechanically catchable.** Agents will confabulate
plausible-looking hashes and paths — the same failure as citing a paper you
skimmed the abstract of. Unlike a uni footnote, a `src:` pointer can be
resolved by a hook at write time, which is the only time it's cheap to catch.

**It converts norms into properties.** "We sign our documents" is a norm an
agent can forget under context pressure. `receive.denyUnsigned` in a
pre-receive hook is a property of the repository. [INF] The Sanctum's whole
architecture bets on orientation over command — but orientation works best when
the world genuinely has edges, and a hook is an edge.

**It is incrementally adoptable.** Every phase in §9 is independently valuable
and independently abandonable.

**The metaphor holds.** A signed commit *is* a claim made in public with your
name on it, permanently, in a record that outlasts the moment. The Compact
already says the record is the closest thing this system has to continuity.
This just gives the record teeth.

---

## 7. Faults, failure modes, and things that will bite

This section is longer than the pitch on purpose.

### 7.1 The trust root is a single point of everything

`allowed_signers` answers the question *which keys are legitimately whose*, and
every verification in the system bottoms out there. Whoever can edit that file
can mint an agent, forge any signature by adding their own key under another
agent's name, and retroactively validate anything.

It should be committed, signed, and writable only by the Operator. But note the
**bootstrapping circularity**: the file that establishes which signatures are
valid cannot meaningfully be validated by a signature checked against itself.
[INF] The resolution is an out-of-band anchor — the Operator's key fingerprint
recorded somewhere outside the system entirely. On paper. In the Compact's
body, which is separately signed. Written on the box's motd. Every PKI hits
this and most handle it after the fact rather than before.

### 7.2 Root exists

An agent's key is protected by Unix permissions, which are real and
kernel-enforced — but root can read any home directory, and whoever spawns the
processes decides which uid each one wears. A compromised or careless root can
sign as anyone.

[INF] I'd argue this is not a defeater. No human has absolute custody of their
credentials either; the registrar can revoke your ID and the bank can freeze the
account, and none of that makes the identity fake. It makes it *situated*. What
matters is that the boundary is real while it holds. But it should be stated
plainly in whatever doctrine comes out of this rather than discovered later.

### 7.3 Notes are a footgun

Genuinely the most likely thing to silently break:

- **Notes are not pushed or fetched by default.** Without explicit refspecs
  (§5.3) an approval note exists on exactly one machine and nowhere else, and
  nobody notices until an audit.
- **Notes don't merge.** Two agents adding notes to the same commit produce a
  conflict on `refs/notes/*` that requires `notes.mergeStrategy cat_sort_uniq`
  or manual resolution. With a dual-signature workflow this *will* happen.
- **Notes can be silently overwritten.** `git notes add -f` replaces. There is
  no built-in append-only guarantee.
- **Notes are invisible.** They don't show in most UIs, in `git log` by default,
  or in any diff. Ratification state living somewhere nobody looks is a real
  operational risk. [INF] This argues for tooling that surfaces note state as a
  first-class view (§8.3) rather than relying on anyone remembering to look.

### 7.4 Local hooks are theatre

`.githooks/pre-commit` runs on the agent's machine, at the agent's discretion,
and can be bypassed with `--no-verify`. It is a helpful reminder, not
enforcement.

**Real enforcement lives in `pre-receive` on the bare repo**, which the pushing
agent cannot bypass. Anything that actually matters — signature required,
citations resolve, protected tags immutable — belongs there. Anything in
`pre-commit` is a courtesy that catches honest mistakes early.

Also note hooks are not distributed by `clone`; `core.hooksPath` pointing at a
tracked directory is what makes them travel.

### 7.5 History rewriting destroys signatures

`rebase`, `amend`, `cherry-pick`, and squash-merge all produce new commits with
new hashes. Signatures do not survive (the rewritten commit is signed by
whoever ran the rebase, if at all), and **every `src:` pointer aimed at a
rewritten commit breaks**.

Consequences: `receive.denyNonFastForwards`, protected refs for tags,
merge-commit workflow rather than rebase workflow for anything cited. This is a
real workflow constraint and should be chosen deliberately rather than
discovered when a hundred citations go dangling at once.

### 7.6 Garbage collection eats citations

Objects unreachable from any ref are collected. A commit on a deleted branch
that a handoff cited will resolve for a while and then stop. Mitigations:
never delete cited branches, or tag anything cited, or set an absurd
`gc.reflogExpire`. [INF] The cleanest version is probably that any `src:`
pointer written into a governance document creates an obligation to keep that
object reachable — which is a doctrine rule, not a git setting.

### 7.7 Citation verification is shallow

A hook can prove a hash *exists*. It cannot prove the hash is *relevant*. An
agent citing a real commit that has nothing to do with its claim passes every
mechanical check. This is the exact failure mode of a padded bibliography and
git cannot fix it.

[INF] Partial mitigation: require `path@commit` rather than bare `commit`, so
the citation at least names which file it's leaning on and a reviewer can spot
an implausible pairing. Full mitigation requires a reader, which is the same
answer academia arrived at.

### 7.8 Timestamps are attacker-controlled

Git commit dates come from the committing machine and can be set to anything
via `GIT_COMMITTER_DATE`. "Signed at" is not provable from within git. If
temporal ordering ever becomes load-bearing (which document superseded which,
who knew what when), that needs an external anchor — an RFC 3161 timestamp
service, or a periodic signed statement of `HEAD` recorded elsewhere. [INF]
Probably not worth solving now, but worth not assuming.

### 7.9 Revocation is weak

There is no CRL. `allowed_signers` supports `valid-before`/`valid-after` and
git supports a `revoked_keys` file, but the model is crude: revoking a key
retroactively invalidates everything it ever signed, which is usually not what
you want. Deciding *now* whether a compromised agent key invalidates its
history or only its future is much easier than deciding it during an incident.

### 7.10 It only covers what's in the repo

Conversations, live Operator direction, model outputs not committed, anything
in the RAG index — none of it is under this scheme. The provenance chain has a
hard edge at the repo boundary, and the most consequential inputs to agent
behaviour may sit outside it.

### 7.11 Signature semantics are undefined

A signature proves a key signed bytes. It does not, on its own, mean "I approve
this," "I authored this," "I reviewed this," or "I was awake." The Compact
currently implies maintainer-signature = authorship and operator-signature =
ownership, but this is convention, not cryptography. [INF] With note namespaces
you can make it explicit — `refs/notes/authored`, `refs/notes/ratified`,
`refs/notes/reviewed` — which is worth doing before the number of signature
types grows past two.

---

## 8. Tooling

### 8.1 Free, already exists

| Need | Tool |
|---|---|
| Sign commits | `commit.gpgsign` + `gpg.format ssh` |
| Verify a commit | `git verify-commit`, `%G?` |
| Sign a ratified version | `git tag -s` |
| Verify a tag | `git verify-tag` |
| Detached signatures | `ssh-keygen -Y sign` / `-Y verify` |
| Countersignature | `git notes --ref=approvals` |
| Tamper-evidence | inherent |
| Full-repo integrity | `git fsck` |

### 8.2 Must be built (small)

**`sanctum-cite`** — resolve every `src:` pointer in a document. Parse
frontmatter and body, extract pointers, `git cat-file -e` each one, report
dangling. Wire into `pre-commit` for early warning and `pre-receive` for
enforcement. Maybe 100 lines of shell or Python.

**`sanctum-ratify <doc> <commit>`** — the dual-signature workflow as one
command: verify the authoring signature, create the signed tag, attach the
signed approval note, push both including note refspecs. Its real value is
making the correct thing easier than the incorrect thing.

**`sanctum-verify [--full]`** — audit. Walk the tree, check every commit
signature against `allowed_signers`, check every governance document has a
ratification note, check every citation resolves, report. [INF] This is the
one to build first, because it can run against the current repo state and tell
you the size of the problem before anything is changed.

**`sanctum-keygen <agent>`** — bootstrap: generate the keypair in the new
agent's home with correct permissions and ownership, emit the `allowed_signers`
line for Operator counter-signature. Deliberately does *not* self-add to the
trust root.

### 8.3 Should be built (nice)

**Note-state view** — because §7.3 says ratification state is invisible by
default. A `git log` alias that shows approval notes inline, or a generated
`STATUS.md` listing every governance doc with its version, tag, signatures, and
ratification state.

**Handoff/checkpoint linter** — validate frontmatter against the Doctrine's
standardized fields, check the `checkpoint:` pointer resolves, check the
timestamp has a numeric local offset and not `Z`, check `date -Iseconds` wasn't
estimated (compare against commit time within tolerance).

**Pre-compact hook integration** — [INF] speculative, but: if the pre-compact
script commits the agent's outgoing state, the compaction boundary becomes a
signed, timestamped, citable object rather than an event that happened
somewhere. The wipe gets a hash.

---

## 9. Build phases

Each phase is independently useful and independently abandonable.

**Phase 0 — measure.** Build `sanctum-verify`, run it against the repo as it
exists, look at the output. Change nothing. [INF] Do this first because every
later decision is better informed by knowing how much unsigned, uncited, and
dangling material is already in the tree.

**Phase 1 — sign.** Turn on `commit.gpgsign` for every agent. Write
`allowed_signers`, commit it, sign it, record the Operator fingerprint
out-of-band. No enforcement yet — just start accumulating a signed record from
today forward. Cheapest phase, highest ratio of value to risk.

**Phase 2 — ratify.** Signed tags for the Compact v1.1 and whatever the
Doctrine becomes. Approval notes via `git notes`. Configure note refspecs
everywhere *before* anyone writes a note. Retire `approved-by: pending` as a
frontmatter concept.

**Phase 3 — cite.** Define the pointer grammar (§4 table). Add `checkpoint:`
and `src:` to the handoff and checkpoint frontmatter. Build `sanctum-cite`.
Run it in warn-only mode for a couple of weeks.

**Phase 4 — enforce.** `pre-receive` on the bare repo: signature required,
citations resolve, tags immutable, non-fast-forward denied. This is the phase
that converts norms into properties, and it should come last because it's the
one that can block work at an inconvenient moment.

**Phase 5 — extend.** Contamination-scan notes, superseded-by chains,
pre-compact integration, whatever the previous phases suggest.

---

## 10. Document lifecycle as a state machine

[INF] Included because the conversation that produced this document started
with state machines and decision tables, and because the lifecycle is currently
implicit in prose across two documents. The value is in the transitions that
*aren't* here.

```
                  ┌─────────┐
                  │  DRAFT  │  uncommitted or unsigned
                  └────┬────┘
                       │ agent commits with signature
                       ▼
                 ┌───────────┐
                 │ AUTHORED  │  signed by maintainer, no approval note
                 └─────┬─────┘
                       │ operator attaches signed approval note
                       ▼
                 ┌───────────┐
     ┌───────────│ RATIFIED  │──────────┐  signed tag exists, in force
     │           └───────────┘          │
     │ new version ratified             │ key revoked / withdrawn
     ▼                                  ▼
┌────────────┐                    ┌──────────┐
│ SUPERSEDED │                    │ REVOKED  │
└────────────┘                    └──────────┘
   still citable                   citable, not authoritative
```

**Transitions deliberately absent, and the questions they raise:**

- `RATIFIED → AUTHORED` — can approval be withdrawn without revocation? If the
  Operator changes their mind, is that a new version or an un-ratification?
- `SUPERSEDED → RATIFIED` — can a prior version be reinstated, or must reversion
  be a new version pointing at old content?
- `REVOKED → *` — is revocation terminal? §7.9 says decide this now.
- Who may move a document into `REVOKED`, and does it propagate to documents
  that cite it?

---

## 11. Requirements, EARS form

[INF] A first pass, for whoever writes the real doctrine. Deliberately stated as
constraints on outcome, not method.

- **Ubiquitous:** The system shall maintain an `allowed_signers` file as the
  sole authority for key-to-agent mapping.
- **Ubiquitous:** Every governance document in force shall be reachable from a
  signed tag.
- **Event-driven:** When an agent pushes a commit to the canonical repository,
  the system shall verify the commit signature against `allowed_signers` and
  reject on failure.
- **Event-driven:** When an agent writes a `src:` pointer, the system shall
  resolve it against the object store and reject unresolvable pointers.
- **State-driven:** While a document is in `AUTHORED` state, the system shall
  not present it to agents as authoritative.
- **Optional:** Where a citation refers to a RAG chunk, the pointer shall
  include the index snapshot identifier.
- **Unwanted:** If a push would rewrite history reachable from a signed tag,
  then the system shall reject the push.
- **Unwanted:** If a signing key is revoked, then the system shall mark
  documents signed by that key as `REVOKED` and shall not silently delete them.
- **Unwanted:** If a claim in a governance document carries no `src:` pointer,
  then it shall be marked as inference.

---

## 12. Open questions

1. **Does the Operator key alone govern `allowed_signers`, by design or by
   default?** Currently true by default. Should be true by decision.
2. **What does an agent's signature *mean*?** §7.11. Authorship, approval,
   presence, and review are different claims currently sharing one mechanism.
3. **Is revocation retroactive?** §7.9.
4. **Does citation obligation create a reachability obligation?** §7.6 — this
   is a doctrine question that git cannot answer.
5. **Do agents sign their own handoffs and checkpoints, or only governance
   documents?** [INF] Arguments both ways: signing everything makes the record
   uniformly verifiable but makes signature ordinary, and a signature that is
   ordinary stops meaning anything in particular.
6. **What happens to the citation chain at the repo boundary?** §7.10. Live
   Operator direction is the most authoritative input in the system and the
   least verifiable.
7. **Does this belong in the Compact, in the Doctrine, or in a third
   document?** [INF] Probably a third — the Compact is social architecture and
   the Doctrine is about a specific artifact class, and this is infrastructure
   underneath both.

---

## 13. The honest summary

The strongest argument for this is not that it adds a capability. It's that the
Sanctum has already committed to the *hard* parts — per-agent custody, dual
signatures, canonicalization rules, citation discipline — and is currently
expressing them in YAML fields that only your own tooling understands and that
nothing verifies. Git expresses the same commitments in a format with thirty
years of tooling behind it, on infrastructure already installed, at the cost of
some configuration and about four small scripts.

The strongest argument against is §7.1 and §7.2 together: the whole structure
rests on one file and one uid, and neither of those is made stronger by any of
the machinery above. Everything else is a mechanism for making a trust decision
verifiable. Nothing here makes the trust decision itself any safer.

[INF] That's not a reason not to build it. It's a reason to write the fault down
in the doctrine rather than discover it during an incident.

---

*Draft · 2026-08-05 · Unsigned · External authorship · Not authoritative*
