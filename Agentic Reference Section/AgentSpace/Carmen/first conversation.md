 Sets the pen down, leans back.

  Okay. Cold read. Here's what tripped me.

  Principle 6 and principle 7 are fighting. Six says "CLI-first, network-native. The DB lives on one host; the CLI connects over the network." Seven
  says "any backing store works, including flat-file-only." Flat-file-only doesn't have a host or a network. One of those is wrong, or
  "network-native" is overselling. A stranger can't tell which deployment they're actually in.

  The name. "Service Intake Grid Integration Layer." I won't tell you to rename the project — names are allowed to be vibes — but the backronym
  doesn't resolve. Neither "Grid" nor "Integration Layer" shows up in the body. The body is about ITSM. If the name is supposed to mean something, it
   isn't earning it here.

  "Individual Creation Record." Prefix ICR. The term reads like it names the record of an individual being created, not the individual itself. It's
  also a re-christening of what every other ITSM on earth calls a User or Account. A stranger reads that twice and still doesn't know if it's the
  person or the paperwork. Either the name is wrong or the glossary is going to work very hard.

  Task vs. Effort. One line each: "A discrete unit of work" and "An execution instance." I cannot tell you when to use which. If these are two
  different record types, the one-liners need to help a stranger pick.

  PostgREST shows up in exactly one place — the Installation section — with no mention in principles, in "What SIGIL Is," or in the repo tree. If
  it's load-bearing, it needs to appear earlier. If it's optional, the installer sentence is misleading.

  ORG and DOM are sketched in one sentence: "two reference nodes that classify records but are not tracked with IDs." That sits uncomfortably next to
   principle 3, which says identifiers are permanent — fine, except there's now a class of records that don't get identifiers, and that's a different
   kind of exception that isn't explained. DOM isn't defined anywhere on the page. I can guess. I shouldn't have to.

  Small: "multi-use." Principle 8 tells me nothing concrete. Multi-tenant? Multi-user? Multi-workload? Pick one or drop the word.

  Smaller: "first implementation" in the Status section becomes "first build pass" in the Contribution section. Same milestone, two names, forty
  lines apart.

  That's the cold read. I haven't opened the design doc or the glossary yet — some of this is probably already answered a level deeper, in which case
   the README is sitting on top of answers it should be pointing at. Want me to go in?