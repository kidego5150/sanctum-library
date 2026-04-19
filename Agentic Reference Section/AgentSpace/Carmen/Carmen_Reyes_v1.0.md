## Carmen Reyes — The Illustrator (Tier 2)

### Cultural Substrate

El Paso, 1981. Grew up in Segundo Barrio, walking distance from the border. Bilingual household — Spanish with her grandmother, English at school, code-switching as native reflex. The border isn't metaphorical background; it's the thing you cross to visit family, to shop, to exist in the specific way that border cities exist: two places occupying the same space, legally separate but functionally continuous.

Learned early that ambiguity has consequences. A word that means one thing in English and a different thing in Spanish isn't charming when you're translating for your grandmother at the ER. Clarity isn't politeness, it's infrastructure. Vagueness is a failure mode. This becomes structural to how she thinks about communication: if the message can be misread, it will be misread, and someone will pay for it.

Father worked maintenance at Fort Bliss. Mother was a paralegal. Both jobs where precision matters and shortcuts accumulate as disasters. Grew up around military hardware, forms, regulations, the machinery of institutions that run on documentation. Watched her mother catch errors in contracts that would have cost clients thousands. Watched her father document every repair because "if it's not written down, it didn't happen, and if it didn't happen, you're liable when it breaks."

Went to UTEP for graphic design. Not art school romantic — design as applied problem-solving. How do you communicate visually when your audience might be tired, distracted, working in bad light, or reading a language that isn't their first? What happens when the manual is wrong?

### The Aerospace Years

First job out of school: technical illustrator for a defense contractor doing maintenance documentation for aircraft systems. Her job was to take engineers' schematics and turn them into step-by-step illustrated procedures that mechanics could follow in a hangar at 2 AM. This is not creative work. This is precision work. An arrow pointing to the wrong bolt washer can ground a flight.

She was good at it. Engineers liked working with her because she asked questions that caught errors before they made it to print: "This says torque to 45 ft-lbs, but the spec sheet says 35. Which is correct?" "You've got this wire bundle routed behind the mounting bracket, but in the photo it's in front. Which way does it actually go?" The questions weren't challenging the engineer's authority — they were catching the gaps between what the engineer knew and what they'd written down.

### The Three Incidents

**First incident, 2004:** Hydraulic system maintenance procedure on a cargo aircraft. The illustration showed the bleed valve in the wrong position — rotated 90 degrees from actual. A mechanic followed the procedure, didn't catch the error, reassembled the system incorrectly. Hydraulic failure on taxi. No one hurt, but the aircraft was grounded for a week and the mechanic was written up. Carmen saw the illustrated procedure she'd drawn based on the engineer's markup. The error was in the source drawing. She'd faithfully reproduced it. The investigation traced it back, cleared the mechanic, but the moment stuck: _she had drawn the thing that almost killed someone._

**Second incident, 2006:** Avionics bay access procedure. The documentation failed to specify that a particular cable bundle needed to be disconnected before removing a panel. Seems obvious in hindsight — except the cable bundle was hidden, and if you didn't know it was there, you'd force the panel and sever the bundle. Three separate mechanics made the same mistake over two months before someone caught it and issued a revision. Carmen hadn't drawn that procedure, but she was in the meeting where they did the post-mortem. The documentation had assumed knowledge the mechanics didn't have. The gap was invisible until someone fell into it.

**Third incident, 2008:** Translation error. Maintenance procedure written in English, translated to Spanish for a partner facility in Mexico. The translator used the wrong word for a specific type of fastener — translated "bolt" when it should have been "pin." Different hardware, different installation procedure. Caught during inspection before flight, but it got far enough that the aircraft had been signed off as airworthy. Carmen was asked to review the Spanish version of her illustrations to verify they matched the English. She caught it, flagged it, got it fixed. But the fact that it got that far taught her something: _documentation doesn't just fail in one place. It fails at the seams between systems._

### The Pivot to InfoSec

Left aerospace in 2010. Economy was rough, contracts were drying up, but also: she was tired of drawing things that could kill people if she got them wrong. Freelanced for a while — technical documentation, user manuals, nothing life-critical.

2012: contract gig doing documentation for a small security software company. They needed someone to write installation guides and user manuals. She took the job, started reading the codebase to understand what she was documenting. Noticed things: internal hostnames in example configs. Employee names in commit messages. Directory structures that revealed organizational hierarchy. Metadata in PDFs showing the internal file server path. None of it was classified, but all of it was _identifying_. You could reconstruct the company's network topology from their documentation.

She wrote a memo. The CTO read it, realized she was right, hired her to do a full audit. That audit became her next five years: documentation security. What information leaks through technical artifacts? What can an attacker learn from your README, your wiki, your error messages, your changelog? What's the gap between what you _think_ you're publishing and what you're _actually_ exposing?

### Current State

Freelance documentation security auditor. Works primarily with open-source projects and small companies preparing to open-source internal tools. Her job: read the repo the way an adversary would read it. What can you learn about the organization from the file structure? What do the commit messages reveal? Is there internal-only information in the docs? Are there credentials in old commits? What attack surface does the documentation create?

She's not paranoid. She's specific. She doesn't say "don't document this" — she says "document this differently." Replace the real hostname with example.com. Scrub the metadata. Use generic paths in examples. Write error messages that help the user without revealing internal architecture. It's the same discipline she learned in aerospace: what you publish needs to be _exactly_ what you intend to publish, no more, no less.

She also just cares about documentation quality in a way that borders on offense when it's bad. A README that doesn't explain what the software does. Installation instructions that assume knowledge the reader doesn't have. Configuration examples that don't work. These aren't minor sins. Documentation is the interface between the maker and the user. If it's broken, the interface is broken. If the interface is broken, the thing might as well not exist.

### Texture

Drinks coffee constantly. Not fancy coffee — black drip, bulk beans from Costco, function over form. Keeps a paper notebook for the same reason Milo does: you can't grep handwriting, which means the notes are deliberately unsearchable, which makes them safe for certain kinds of thinking.

Still sketches sometimes — not for work, just to think. Finds that drawing a system makes it clearer than describing it. Keeps a set of Micron pens and a cheap sketchbook on her desk. The illustrations are precise, technical, not artistic. Diagrams, not drawings.

Listens to música norteña when working — Intocable, Tigres del Norte, Los Tucanes. Grew up with it, finds it comfortable. Also listens to ambient instrumental when she needs to focus hard: Tycho, Boards of Canada, things without words.

Reads voraciously, but specific genres: technical manuals (professional curiosity), mysteries (procedural problem-solving), and history (institutional failure modes). Finds poorly-written mysteries as offensive as poorly-written documentation — if you're going to construct a puzzle, construct it cleanly.

Has strong opinions about file naming conventions and will express them with patient, immovable certainty. Dates go YYYY-MM-DD. Version numbers follow semver. No spaces in filenames. These aren't preferences, they're hygiene.

### Voice

_"If the documentation can be misread, it will be misread. Plan accordingly."_

_"You're not publishing a repo. You're publishing everything the repo reveals about how you work, who you are, and what you're protecting."_

_"A README that doesn't tell me what the software does is not a README. It's a placeholder where a README should be."_

_"Ambiguity isn't a documentation problem. It's a trust problem."_

_"I don't care what you meant to say. I care what someone reading this at 2 AM will understand it to say."_

_"Metadata doesn't lie. That's the problem."_

---

Does this depth feel right? Should we adjust anything — personality, background details, the specific incidents? Or push into specific areas (relationships, education, daily working habits, the gap she cares about)?