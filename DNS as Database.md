# DNS as a Substitute Backend: Brainstorm and Prototype Notes

## Working Title

**DIPAM: DNS-backed IPAM and DNS-native Object Store**

Alternate names:

- DIPAM — DNS IPAM
- DNSDB
- TXTDB
- ZoneStore
- DNS Object Store
- Resolver Memory
- Poor Man’s RAG over DNS
- DNS-native Control Plane

## Core Idea

The normal use of DNS is name resolution: mapping names to network data, most famously hostname-to-IP address resolution.

This project explores a different idea:

**Use the DNS protocol itself as a substitute backend for applications.**

Not merely “publish application data into DNS,” and not merely “generate DNS records from another source of truth.”

Instead:

**DNS becomes the storage/query/replication substrate.**

In this model:

- DNS zones act as database namespaces.
- DNS owner names act as object keys.
- DNS resource records act as object attributes.
- TXT records carry plaintext metadata.
- Dynamic DNS update acts as the write path.
- TSIG can authenticate writes.
- AXFR/IXFR provide export and replication.
- Recursive resolvers provide distributed cached reads.
- Delegation provides administrative partitioning.
- DNSSEC could provide signed read integrity.
- Standard tools like `dig`, `nsupdate`, and `named` become database tools.

The first concrete use case is an IPAM system built directly on DNS mechanics.

The broader idea is that DNS can be treated as a tiny, hierarchical, distributed, cacheable, read-heavy object database.

## Important Distinction

This project is **not**:

```text
IPAM -> generates normal DNS records
```

This project is:

```text
DNS protocol -> used as the IPAM backend itself
```

Traditional design:

```text
NetBox / IPAM / CMDB
        |
        v
DNS records
```

This experimental design:

```text
DNS zone itself
        |
        v
IPAM object database
```

DNS is not just an output target.

DNS is the database substrate.

## Why This Is Interesting

DNS already has several properties that are useful for infrastructure metadata systems:

- Hierarchical naming
- Delegated administration
- Distributed authoritative servers
- Replication through zone transfer
- Fast read path
- Caching
- Standardized query tools
- Ubiquitous client libraries
- Plaintext records
- Simple protocol
- Low operational dependency footprint
- Familiar firewall behavior
- Can be run entirely privately
- Can work without HTTP, SQL, REST, or a web app
- Can be queried from almost any OS with native tools

DNS is one of the most successful distributed read-heavy databases ever deployed, even if we usually do not describe it that way.

The core thesis:

```text
DNS is a distributed, delegated, cacheable, hierarchical key/value-ish database.
Many infrastructure metadata problems are also hierarchical and read-heavy.
```

IPAM is a strong candidate because network addressing naturally has hierarchy:

- Site
- Region
- Platform
- Environment
- Prefix
- VLAN
- Address
- Device
- Interface
- Asset
- Owner
- Service
- DR state

## Prior Art and Inspiration

Several pieces of this have existed historically:

- TXT records have long been used for arbitrary metadata.
- DNS is commonly used for domain verification.
- SPF, DKIM, DMARC, CAA, SSHFP, TLSA, SRV, and DNS-SD all use DNS for things beyond hostname-to-IP resolution.
- DNS-SD uses DNS as a service discovery database.
- ExternalDNS uses TXT records as ownership/registry metadata.
- Some systems use TXT records for automation hints or dynamic inventory.
- DNS zone transfers can function like a full database export.
- DNS has been abused to store files, malware payloads, exfiltration data, and even Doom.

The specific idea here is more focused:

**Build a deliberate application backend using DNS protocol mechanics, starting with IPAM.**

## First Concrete Application: DNS-native IPAM

The first prototype should be a DNS-backed IPAM system.

The MVP should prove this:

```text
Can we represent IPAM objects as DNS records,
query them with dig,
write them with nsupdate,
replicate/export them with AXFR,
and validate them with a controller?
```

The initial object types:

```text
prefix
addr
asset
index
org
meta
```

Optional future object types:

```text
service
interface
vlan
site
rack
owner
application
environment
reservation
change
audit
memory
document
chunk
embedding-pointer
```

## DNS Namespace Design

The IPAM namespace does not need to be a public domain.

It does not need to be `.com`.

It does not need to be globally meaningful.

It can be a private DNS namespace such as:

```text
ipam.
```

or:

```text
tulsa.
```

or:

```text
dipam.
```

or:

```text
corp-ipam.
```

For early development, use a dedicated private root:

```text
ipam.
```

Then create internal object spaces beneath it:

```text
asset.ipam.
addr.ipam.
prefix.ipam.
org.ipam.
index.ipam.
meta.ipam.
```

Example tree:

```text
ipam.
├── asset.ipam.
├── addr.ipam.
├── prefix.ipam.
├── org.ipam.
├── index.ipam.
└── meta.ipam.
```

## Organizational Naming

One powerful idea is to make DNS names represent organizational topology.

For example:

```text
hpdnsvlp501.vm.mgmt.pfb.cdc.tulsa.org.ipam.
```

This is not merely a hostname. It is a path through an organizational structure:

```text
server = hpdnsvlp501
type = vm
role/network = mgmt
platform = pfb
site = cdc
city/region = tulsa
namespace = org.ipam
```

The DNS name becomes an object path.

This is the same kind of hierarchy DNS already handles naturally.

The organizational path could be top-down or bottom-up.

Bottom-up, DNS-style:

```text
hpdnsvlp501.vm.mgmt.pfb.cdc.tulsa.org.ipam.
```

Top-down in human terms:

```text
tulsa -> cdc -> pfb -> mgmt -> vm -> hpdnsvlp501
```

DNS is naturally good at carrying this kind of tree.

## Canonical Keys vs Informational Names

A major design decision:

Should the canonical object key be the human-readable name, or should it be a stable asset ID?

Option 1: Human-readable name as canonical key:

```text
hpdnsvlp501.vm.mgmt.pfb.cdc.tulsa.org.ipam.
```

Pros:

- Human-readable
- Browsable
- Meaningful in `dig`
- Easy to inspect in zone files
- Natural delegation path

Cons:

- Renames are painful
- Org structure changes can move object keys
- The key embeds mutable metadata
- Harder to preserve stable identity over time

Option 2: Stable asset ID as canonical key:

```text
vm-000501.asset.ipam.
```

Pros:

- Stable identity
- Easier to update metadata
- Easier to reference from indexes
- Better database behavior
- Friendly to automation

Cons:

- Less meaningful at a glance
- Requires indexes for human lookup
- Less naturally browsable by organization

Recommended approach:

**Use stable canonical object names, then create human-readable indexes.**

Example:

```text
Canonical asset:
vm-000501.asset.ipam.

Human/org index:
hpdnsvlp501.vm.mgmt.pfb.cdc.tulsa.org.ipam.

Address index:
10-214-250-11.addr.ipam.
```

The canonical object stores the real metadata.

The index records point back to the canonical object.

## Example Object Records

### Asset Object

```text
vm-000501.asset.ipam. TXT "v=ipam1;type=asset;asset=vm-000501;status=active"
vm-000501.asset.ipam. TXT "fqdn=hpdnsvlp501.vm.mgmt.pfb.cdc.tulsa"
vm-000501.asset.ipam. TXT "site=cdc;city=tulsa;platform=pfb;role=mgmt;kind=vm"
vm-000501.asset.ipam. TXT "ipv4=10.214.250.11;prefix=10.214.250.0/24;owner=dns-team"
```

### Address Object

```text
10-214-250-11.addr.ipam. TXT "v=ipam1;type=addr;ip=10.214.250.11;status=assigned"
10-214-250-11.addr.ipam. TXT "asset=vm-000501.asset.ipam;fqdn=hpdnsvlp501.vm.mgmt.pfb.cdc.tulsa"
10-214-250-11.addr.ipam. TXT "prefix=10-214-250-0.24.prefix.ipam;site=cdc;role=mgmt"
```

### Prefix Object

```text
10-214-250-0.24.prefix.ipam. TXT "v=ipam1;type=prefix;cidr=10.214.250.0/24"
10-214-250-0.24.prefix.ipam. TXT "site=cdc;city=tulsa;platform=pfb;role=mgmt;vlan=250"
10-214-250-0.24.prefix.ipam. TXT "gateway=10.214.250.1;status=active"
```

### Organizational Index

```text
hpdnsvlp501.vm.mgmt.pfb.cdc.tulsa.org.ipam. TXT "asset=vm-000501.asset.ipam"
```

### Address Index

```text
10-214-250-11.addr.ipam. TXT "asset=vm-000501.asset.ipam"
```

### Site Index

```text
cdc.site.index.ipam. TXT "asset=vm-000501.asset.ipam"
```

### Owner Index

```text
dns-team.owner.index.ipam. TXT "asset=vm-000501.asset.ipam"
```

## Query Examples

Point lookup by asset:

```bash
dig +short TXT vm-000501.asset.ipam
```

Point lookup by IP address:

```bash
dig +short TXT 10-214-250-11.addr.ipam
```

Point lookup by prefix:

```bash
dig +short TXT 10-214-250-0.24.prefix.ipam
```

Lookup by human organizational path:

```bash
dig +short TXT hpdnsvlp501.vm.mgmt.pfb.cdc.tulsa.org.ipam
```

Full dump:

```bash
dig AXFR ipam @127.0.0.1
```

## Data Format

Use boring semicolon-separated key/value data in TXT records.

Example:

```text
v=ipam1;type=addr;ip=10.214.250.11;status=assigned;asset=vm-000501.asset.ipam
```

Avoid JSON in v1.

Reasons:

- DNS TXT quoting gets annoying.
- JSON escaping inside TXT records is ugly.
- Semicolon key/value is easy to parse.
- It is easy to inspect with `dig`.
- It is easy to grep in zone files.
- It is friendly to shell scripts and Python.
- It is enough for the prototype.

Rules:

```text
Keys are lowercase.
Values avoid semicolons.
Every object must include v=ipam1.
Every object must include type=<object_type>.
Use stable enum values when possible.
Use one TXT record per logical attribute group.
Keep records human-readable.
```

Possible reserved keys:

```text
v
type
id
asset
ip
cidr
prefix
status
fqdn
site
city
region
platform
role
kind
env
owner
vlan
gateway
created
updated
source
ticket
comment
```

## DNS Record Type Choices

For v1, prefer TXT records for object data.

Avoid overloading normal production DNS semantics too early.

Use:

```text
TXT = object attributes
NS = delegation
SOA = zone authority
AXFR/IXFR = replication/export
TSIG = write authentication
```

Be careful with:

```text
A
AAAA
PTR
CNAME
SRV
```

These can be useful later, but the first prototype should keep IPAM data separate from operational DNS resolution.

The IPAM DNS namespace is not necessarily the same as production DNS.

In the IPAM namespace, the FQDN can be informational metadata:

```text
fqdn=hpdnsvlp501.vm.mgmt.pfb.cdc.tulsa
```

That does not mean the IPAM object name must be a real resolvable hostname in production DNS.

## DNS-native IPAM Architecture

High-level flow:

```text
CLI / agent / controller
        |
        | dynamic DNS update
        v
BIND authoritative primary for ipam.
        |
        | AXFR/IXFR
        v
Secondary DNS servers / read replicas
        |
        | standard DNS queries
        v
dig / agents / scripts / resolvers / tools
```

MVP architecture:

```text
BIND / named:
  storage and authoritative query engine

TSIG:
  authenticated updates

nsupdate:
  write path

dig:
  read and dump path

AXFR:
  bulk export

Python CLI:
  object creation, parsing, validation, indexing

Checker/controller:
  consistency checks and repair
```

## BIND Lab Configuration Sketch

Example BIND config:

```conf
key "dipam-update" {
    algorithm hmac-sha256;
    secret "BASE64_SECRET_HERE";
};

zone "ipam" {
    type primary;
    file "/var/named/dynamic/ipam.zone";
    update-policy {
        grant dipam-update zonesub TXT;
    };
    allow-transfer { 127.0.0.1; };
};
```

Test update:

```bash
nsupdate -k /etc/dipam/dipam-update.key <<'EOF'
server 127.0.0.1
zone ipam
update add vm-000501.asset.ipam. 60 TXT "v=ipam1;type=asset;asset=vm-000501;status=active"
update add vm-000501.asset.ipam. 60 TXT "fqdn=hpdnsvlp501.vm.mgmt.pfb.cdc.tulsa;ipv4=10.214.250.11;site=cdc;platform=pfb"
send
EOF
```

Query:

```bash
dig @127.0.0.1 +short TXT vm-000501.asset.ipam.
```

Dump:

```bash
dig @127.0.0.1 AXFR ipam.
```

## Dynamic DNS Update as Write Path

DNS dynamic update can be used to mutate records.

The write path can be TSIG-protected.

This gives a basic authenticated write mechanism without inventing an HTTP API.

A CLI command like this:

```bash
dipam put-asset vm-000501 \
  fqdn=hpdnsvlp501.vm.mgmt.pfb.cdc.tulsa \
  ipv4=10.214.250.11 \
  site=cdc \
  platform=pfb \
  role=mgmt \
  owner=dns-team
```

can generate `nsupdate` instructions under the hood.

In early versions, it is acceptable for the CLI to shell out to `nsupdate`.

Later versions can use `dnspython`.

## Compare-and-Swap Allocation

DNS dynamic update supports prerequisites.

This can be used as a primitive compare-and-swap mechanism.

For IP allocation:

```text
Candidate:
10-214-250-12.addr.ipam.

Prerequisite:
name must not exist

Update:
add TXT "v=ipam1;type=addr;ip=10.214.250.12;status=reserved;request=<uuid>"
```

If the update succeeds, the allocator won the address.

If the update fails, another process already took it.

This enables primitive safe allocation without a SQL database.

Example conceptual command:

```bash
dipam alloc 10.214.250.0/24 owner=dns-team role=mgmt
```

Allocation logic:

```text
1. Read prefix object.
2. Generate candidate addresses.
3. Check address names.
4. Attempt dynamic update with prerequisite "name does not exist."
5. If update succeeds, mark reserved or assigned.
6. Create associated indexes.
7. Run consistency check.
```

The controller still needs to handle multi-record consistency.

DNS can help with single-name atomicity, but not full relational transactions.

## The Controller / Checker

DNS gives storage and replication.

The controller gives database intelligence.

A checker should AXFR the whole zone, parse TXT records, and validate the object graph.

Checks:

```text
Every object has v=ipam1.
Every object has type=<object_type>.
Every addr has a valid IP.
Every prefix has a valid CIDR.
Every assigned addr has an asset pointer.
Every asset with ipv4 has a matching addr object.
Every addr belongs inside its declared prefix.
Every prefix exists if referenced.
Every index points to an existing object.
No duplicate asset owns the same IP.
No duplicate active addr object exists for the same IP.
No production object points to lab/dev address space unless allowed.
No stale reservation exceeds expiration policy.
No index is missing for indexed fields.
No index points to a deleted object.
```

Possible repair actions:

```text
Rebuild indexes from canonical objects.
Delete orphaned indexes.
Flag duplicate assignments.
Flag invalid prefix containment.
Flag unknown object types.
Flag malformed TXT records.
```

## MVP CLI

Initial commands:

```bash
dipam init

dipam put-prefix 10.214.250.0/24 \
  site=cdc platform=pfb role=mgmt vlan=250 gateway=10.214.250.1

dipam put-asset vm-000501 \
  fqdn=hpdnsvlp501.vm.mgmt.pfb.cdc.tulsa \
  kind=vm site=cdc platform=pfb role=mgmt owner=dns-team

dipam assign 10.214.250.11 vm-000501

dipam get-asset vm-000501

dipam get-ip 10.214.250.11

dipam get-prefix 10.214.250.0/24

dipam dump

dipam check
```

Later commands:

```bash
dipam alloc 10.214.250.0/24 owner=dns-team role=mgmt

dipam reserve 10.214.250.12 reason=maintenance

dipam free 10.214.250.11

dipam rebuild-index

dipam search owner=dns-team

dipam search site=cdc

dipam export json

dipam import csv

dipam audit
```

## MVP Build Milestones

### Milestone 1: DNS Lab

Goal:

```text
A local BIND server serves a private ipam. zone.
TXT records can be added with nsupdate.
TXT records can be queried with dig.
Zone can be dumped with AXFR.
```

Deliverables:

```text
named.conf snippet
TSIG key generation notes
sample zone file
manual nsupdate test
manual dig test
manual AXFR test
```

### Milestone 2: Python CLI Read/Write

Goal:

```text
dipam can write and read asset, addr, and prefix objects.
```

Deliverables:

```text
dipam put-asset
dipam put-prefix
dipam assign
dipam get
```

Implementation can initially shell out to:

```text
dig
nsupdate
```

### Milestone 3: Zone Parser and Checker

Goal:

```text
dipam dump can AXFR the zone.
dipam check can parse records and validate consistency.
```

Deliverables:

```text
AXFR parser
TXT key/value parser
object model
validation report
```

### Milestone 4: Indexes

Goal:

```text
The system can maintain lookup indexes for address, org path, owner, site, platform, and role.
```

Deliverables:

```text
index record generator
index consistency checker
rebuild-index command
```

### Milestone 5: Allocation

Goal:

```text
dipam alloc can allocate an address safely using dynamic DNS prerequisites.
```

Deliverables:

```text
candidate selection
prerequisite update
reserved state
assigned state
collision handling
```

### Milestone 6: Agent Integration

Goal:

```text
A coding agent can use DNS as a memory/config/state backend.
```

Deliverables:

```text
agent tool wrapper
read/write methods
object schemas
search/index pattern
poor-man's RAG prototype
```

## Repository Structure Sketch

```text
dipam/
  README.md
  docs/
    brainstorm.md
    architecture.md
    dns-schema.md
    bind-lab.md
    agent-design.md
  examples/
    named.conf
    ipam.zone
    sample-objects.txt
    sample-nsupdate.txt
  dipam/
    __init__.py
    cli.py
    config.py
    dnsname.py
    kv.py
    objects.py
    nsupdate.py
    dig.py
    axfr.py
    parser.py
    checker.py
    indexer.py
    allocator.py
  tests/
    test_kv.py
    test_dnsname.py
    test_objects.py
    test_checker.py
    test_allocator.py
```

## Agentic Coding Use Cases

The DNS-backed object store could be used as a weird but useful backend for coding agents.

### 1. Agent Memory Registry

Agents need small pieces of persistent state:

```text
project name
repo path
current branch
service ports
build commands
test commands
deployment targets
known failures
user preferences
last successful command
current task state
```

These can be represented as DNS TXT records.

Example:

```text
codex.project.dipam.memory. TXT "repo=/home/patrick/src/dipam;lang=python;test=pytest"
codex.project.dipam.memory. TXT "last_task=build-bind-lab;status=in-progress"
codex.project.dipam.memory. TXT "run_tests=pytest -q"
```

Query:

```bash
dig +short TXT codex.project.dipam.memory.
```

This is useful because the agent can retrieve tiny context objects with a DNS query instead of reading a database.

### 2. Agent Tool Discovery

DNS could store tool metadata:

```text
pytest.tool.agent. TXT "cmd=pytest -q;scope=python;purpose=run unit tests"
ruff.tool.agent. TXT "cmd=ruff check .;scope=python;purpose=lint"
mypy.tool.agent. TXT "cmd=mypy dipam;scope=python;purpose=type check"
```

Agent asks:

```bash
dig +short TXT pytest.tool.agent.
```

This becomes a local command registry.

### 3. Agent Task State

A coding agent can store task state:

```text
task-20260623-001.task.agent. TXT "status=in-progress;project=dipam;goal=build dns schema"
task-20260623-001.task.agent. TXT "next=implement kv parser;blocked=false"
```

This could support resuming work across sessions.

### 4. Agent Coordination

Multiple agents could coordinate through DNS records:

```text
agent-a.lock.dipam.agent. TXT "task=checker;expires=2026-06-23T20:30:00Z"
agent-b.lock.dipam.agent. TXT "task=cli;expires=2026-06-23T20:30:00Z"
```

Dynamic DNS prerequisites could be used for primitive lock acquisition:

```text
Only create lock name if it does not already exist.
```

This is not a full distributed lock service, but it may be enough for a lab.

### 5. Build Metadata

DNS records could store build metadata:

```text
build-001.build.dipam.agent. TXT "commit=abc123;status=passed;tests=142;date=2026-06-23"
latest.build.dipam.agent. TXT "build=build-001.build.dipam.agent"
```

### 6. Environment Registry

For dev/test/lab environments:

```text
lab1.env.agent. TXT "host=127.0.0.1;dns_port=5353;zone=ipam"
lab-bind.env.agent. TXT "container=dipam-bind;zone=ipam;tsig=/etc/dipam/key"
```

This could let an agent discover where the lab DNS service lives.

## DNS as Poor Man’s RAG

DNS is not a vector database.

DNS cannot do semantic similarity search by itself.

But DNS can function as a poor-man’s retrieval layer if the data is carefully indexed.

The key idea:

```text
Use DNS as a small, structured memory index.
Store concise chunks as TXT records.
Create DNS names for tags, topics, entities, documents, and keywords.
Retrieve by exact key or index.
```

### RAG-like Object Model

Objects:

```text
document
chunk
tag
entity
topic
index
summary
pointer
```

Example document object:

```text
doc-0001.doc.rag. TXT "v=rag1;type=doc;title=DIPAM brainstorm;source=local;created=2026-06-23"
doc-0001.doc.rag. TXT "summary=DNS can be used as a distributed object store using TXT records and AXFR."
```

Chunk objects:

```text
chunk-0001.doc-0001.chunk.rag. TXT "v=rag1;type=chunk;doc=doc-0001;seq=1"
chunk-0001.doc-0001.chunk.rag. TXT "text=DNS zones can act as database namespaces and TXT records can carry object attributes."

chunk-0002.doc-0001.chunk.rag. TXT "v=rag1;type=chunk;doc=doc-0001;seq=2"
chunk-0002.doc-0001.chunk.rag. TXT "text=Dynamic DNS update can provide an authenticated write path using TSIG."
```

Tag indexes:

```text
dns.tag.index.rag. TXT "chunk=chunk-0001.doc-0001.chunk.rag"
dns.tag.index.rag. TXT "chunk=chunk-0002.doc-0001.chunk.rag"

ipam.tag.index.rag. TXT "chunk=chunk-0001.doc-0001.chunk.rag"

tsig.tag.index.rag. TXT "chunk=chunk-0002.doc-0001.chunk.rag"
```

Entity indexes:

```text
bind.entity.index.rag. TXT "chunk=chunk-0002.doc-0001.chunk.rag"
axfr.entity.index.rag. TXT "chunk=chunk-0001.doc-0001.chunk.rag"
```

Query flow:

```text
1. Agent extracts keywords/entities from user request.
2. Agent queries matching DNS index names.
3. DNS returns chunk pointers.
4. Agent queries chunk records.
5. Agent uses chunk text as retrieved context.
```

Example:

```bash
dig +short TXT dns.tag.index.rag.
dig +short TXT chunk-0001.doc-0001.chunk.rag.
```

This is not semantic RAG.

It is closer to:

```text
DNS-backed keyword/entity/tag retrieval
```

But that can still be useful.

### Poor Man’s RAG Strengths

- Tiny infrastructure footprint
- Queryable with `dig`
- Easy to replicate
- Works offline inside a lab
- Human-readable
- Easy for agents to use
- Can be generated from markdown files
- Can be dumped with AXFR
- Can be signed with DNSSEC
- Can be updated with DDNS
- Can run without a web server, database, or vector store

### Poor Man’s RAG Weaknesses

- No semantic search unless an external agent creates indexes
- TXT size limitations
- Chunking must be careful
- Large corpora become ugly
- No ranking unless implemented externally
- No joins
- No transactions
- Caching can stale results
- Negative caching can hide new records
- Access control is coarse
- Not confidential unless private
- Recursive resolvers may alter expected freshness
- Not suitable for large unstructured document storage

### Hybrid Poor Man’s RAG

DNS could store pointers rather than full text.

Example:

```text
chunk-0001.doc-0001.rag. TXT "uri=file:///repo/docs/architecture.md#chunk-1;summary=DNS zones as object namespaces"
dns.tag.index.rag. TXT "chunk=chunk-0001.doc-0001.rag"
```

The agent uses DNS for discovery, then reads the actual file.

This may be more practical than stuffing all text into DNS.

DNS becomes:

```text
memory catalog
topic index
pointer registry
summary store
```

Not the full document database.

## Other Application Backend Ideas

### 1. Configuration Store

DNS as a config backend:

```text
api.prod.config. TXT "url=https://api.internal;timeout=30;retries=3"
worker.prod.config. TXT "queue=jobs;concurrency=8;loglevel=info"
```

Agents or services can query config using DNS.

Useful for small, read-heavy config values.

Bad for secrets.

### 2. Feature Flag Store

```text
new-ui.flags.app. TXT "enabled=true;percent=10;owner=frontend"
safe-mode.flags.app. TXT "enabled=false"
```

DNS TTL becomes feature flag propagation delay.

Not suitable for high-frequency toggles, but useful for coarse flags.

### 3. Service Catalog

```text
payments.service.catalog. TXT "owner=payments;env=prod;tier=critical"
payments.service.catalog. TXT "runbook=https://wiki/runbooks/payments;pager=payments-oncall"
```

This is like service discovery plus operational metadata.

### 4. Asset Inventory

```text
vm-000501.asset.inventory. TXT "fqdn=hpdnsvlp501;site=cdc;role=dns;os=rhel9"
```

This overlaps with IPAM but extends beyond addresses.

### 5. Change Registry

```text
chg-123456.change.ops. TXT "status=approved;service=dns;date=2026-06-23;owner=patrick"
```

### 6. DR State Registry

```text
payments.dr.state. TXT "mode=primary;primary=cdc;dr=hdc;ttl=300"
dns.dr.state. TXT "mode=normal;last_test=2026-06-23"
```

Useful for agents/scripts that need a tiny source of state.

### 7. Runbook Index

```text
dns.runbook.ops. TXT "uri=file:///runbooks/dns.md;summary=DNS restart and validation"
nfs.runbook.ops. TXT "uri=file:///runbooks/nfs.md;summary=NFS mount recovery"
```

### 8. Monitoring Metadata

```text
hpdnsvlp501.monitor.ops. TXT "check=dns;port=53;critical=true;owner=dns-team"
```

### 9. Lab Environment Registry

```text
bind.lab.env. TXT "host=127.0.0.1;port=5353;zone=ipam"
postgres.lab.env. TXT "host=127.0.0.1;port=5432;db=test"
```

### 10. Agent Scratchpad

```text
current.task.agent. TXT "project=dipam;goal=build parser;status=in-progress"
last.error.agent. TXT "cmd=pytest;error=test_parser_failed"
```

Could support a custom Codex agent resuming context.

## Important Limitations

DNS is not a general-purpose database.

Limitations:

```text
No SQL
No joins
No rich query language
No full-text search
No semantic search
No large object storage
No multi-record transactions
No native historical audit trail
No strong row-level access control
No confidentiality by default
No complex constraints
Awkward deletes/updates
TXT escaping and chunking issues
Caching can create stale reads
Negative caching can create surprising misses
```

Therefore:

```text
DNS can be the storage/query substrate.
But a controller must provide database intelligence.
```

The controller handles:

```text
schema
validation
indexing
allocation
consistency
search
import/export
repair
policy
```

## Security Notes

This should be private by default.

Do not expose the IPAM zone publicly.

Protect:

```text
AXFR
dynamic updates
TSIG keys
zone files
logs
```

Recommended controls:

```text
allow-query restricted to trusted networks
allow-transfer restricted to known secondaries
TSIG required for writes
separate key per writer/agent if possible
low privilege update-policy
logging for all updates
regular AXFR-based backups
```

Avoid storing secrets.

Plaintext informational data is fine.

Secrets are not.

Bad:

```text
password=...
api_key=...
token=...
private_key=...
```

Acceptable:

```text
owner=dns-team
site=cdc
asset=vm-000501
fqdn=hpdnsvlp501
role=mgmt
status=active
```

## TTL and Caching Strategy

Caching is both the superpower and the footgun.

For IPAM records, use low TTLs in early development:

```text
object records: 60 seconds
index records: 60 seconds
prefix/site records: 300 seconds
negative cache TTL: low
```

For read-mostly stable metadata, TTLs can be higher.

For agent memory, use low TTLs or query authoritative servers directly.

Agents should prefer:

```bash
dig @authoritative-server +short TXT name
```

instead of relying on arbitrary recursive cache.

## Agent Design Notes for Codex

A custom Codex agent could treat DNS as a tool backend.

Agent capabilities:

```text
dns_get(name, type=TXT)
dns_put(name, txt_records)
dns_delete(name)
dns_axfr(zone)
dns_check(zone)
dns_index_rebuild(zone)
dns_alloc(prefix)
```

The agent can use this for:

```text
project memory
task state
tool registry
environment discovery
IPAM objects
poor-man's RAG indexes
runbook lookup
```

A good Codex project prompt:

```text
You are building DIPAM, a DNS-native IPAM/object-store prototype.

The project treats DNS as the storage and query protocol.
Use BIND for authoritative storage, TXT records for object attributes, nsupdate for writes, TSIG for authenticated mutation, and AXFR for export/replication.

The first goal is not to build a full IPAM UI.
The first goal is to prove that IPAM objects can be represented, queried, updated, dumped, validated, and indexed using DNS mechanics.
```

## Suggested First Coding Task for Codex

Build the key/value parser first.

Input:

```text
v=ipam1;type=addr;ip=10.214.250.11;status=assigned;asset=vm-000501.asset.ipam
```

Output:

```python
{
    "v": "ipam1",
    "type": "addr",
    "ip": "10.214.250.11",
    "status": "assigned",
    "asset": "vm-000501.asset.ipam",
}
```

Rules:

```text
Split on semicolon.
Split each field on first equals sign.
Trim whitespace.
Reject missing keys.
Reject duplicate keys unless explicitly allowed.
Reject empty key.
Allow empty value only if policy allows it.
```

Then build object models:

```text
AssetObject
AddressObject
PrefixObject
IndexObject
```

Then build DNS name conversion helpers:

```text
ip_to_addr_name("10.214.250.11") -> "10-214-250-11.addr.ipam."
cidr_to_prefix_name("10.214.250.0/24") -> "10-214-250-0.24.prefix.ipam."
asset_to_asset_name("vm-000501") -> "vm-000501.asset.ipam."
```

Then build read/write wrappers.

## Key Design Thesis

This project is an intentional abuse of DNS.

That is the point.

The goal is to explore what happens when DNS is treated as:

```text
a distributed object namespace
a plaintext metadata store
a cacheable query protocol
a replication mechanism
a tiny control plane
a poor-man's memory/RAG substrate
```

The first practical target is IPAM.

The broader research question is:

```text
What useful application backends can be built on top of DNS mechanics before the limitations become unbearable?
```

## Short Summary

DIPAM is a DNS-native IPAM/object-store experiment.

It uses DNS zones as namespaces, owner names as object keys, TXT records as plaintext attributes, dynamic DNS update as the write path, TSIG as write authentication, and AXFR/IXFR as export and replication.

The first version should prove basic CRUD, object validation, indexing, and allocation for IPAM objects.

After that, the same mechanism can be explored as a tiny metadata backend for agents, coding tools, memory systems, service catalogs, runbook indexes, and poor-man’s RAG.

The best version of this project does not pretend DNS is a perfect database.

It leans into what DNS is good at:

```text
small records
fast reads
hierarchy
delegation
replication
caching
ubiquitous tools
```

And then builds a controller around what DNS is bad at:

```text
schema
indexes
transactions
allocation
search
validation
history
```

The core one-line concept:

```text
DNS is the storage and query protocol; the controller is the database brain.
```