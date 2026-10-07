# Design D1: ASL Learning Assistant

Team: Braden Monnin, Connor Slutsky, Isaac Dowdy

**Goal:** give self-taught ASL learners instant English text and hand-shape feedback from a webcam, without storing their video data.

**Input:** live webcam video of the learner performing an ASL sign.
**Output:** English text translation of the sign, plus feedback on the user's formation of the sign.

TDL
 - [ ] do some research about possible existing models and how they work - inputs, outputs, details that may be important for augmentation
 - [ ] running down the sections

# Data model
the D1 diagram. For at least one core component: entities, the key attributes that matter for behavior (not every column), and relationships with cardinality on both ends. The sketch should fit on one page. Below it, a short table of structural decisions: why a thing is an entity rather than an attribute, why a relationship is one-to-many rather than many-to-many, and which fields are indexed and why. Say whether you chose a relational model or a simpler store, and why.

TODO

# Core algorithms
Name the one or two computations that, if slow or wrong, break the product. Document each with the five-part template from the lecture: what it is and the problem it solves, inputs and outputs with exact types, expected complexity at your realistic data size, why this one rather than the alternatives, and edge cases (empty input, duplicates, ties, failure conditions). Clear prose a teammate could implement is the standard; pseudocode is optional.

TODO

# Build-versus-reuse decisions
A table with one row per significant piece of the detailed components: what it is, build or reuse, the library or service if reused, its license, and a one-line reason. Check maturity, licensing, performance, and fit for every candidate library.

TODO

# API contract. 
For the detailed component's key endpoints or methods: name, inputs (each parameter's type, required or optional, valid range), outputs (the exact response shape with field types and units), and error responses (status or code, and when it happens). Give one example request and response in a code block. Add one sentence on versioning: what is versioned, how, and what counts as a breaking change.

TODO

# Technology choices with justification. 
One paragraph per major choice (database, backend language or framework, front end, messaging or hardware platform, hosting). Each paragraph must reference all five criteria from class: team skill fit, licensing, community support, performance, and cost and hosting. Name the alternative you passed over for at least two of the choices.

TODO

--- 

### suggestions

Iterate between the data model and the algorithm. If the algorithm needs a fast lookup, index that field. If it needs time ranges, store start and end explicitly.
Estimate complexity at your realistic data size and again at 100 times it. Then say whether the difference matters. Tuning a bottleneck you have not measured is wasted time.
Do not hand-roll authentication, date parsing, or sorting. Reuse it, record why, and spend the hours on the part specific to your project.
An API table without an error column is not a specification. "It returns null" does not tell a caller what to do.
Have a teammate read each technology paragraph and name which sentence covers which criterion. Add the ones that are missing.

---

### Grading

Criterion
	

What graders look for
	

Weight

Data model quality
	

Entities, key attributes, and relationships with cardinality are correct and clear; fits on one page; conventions explained; structural decisions justified and requirement IDs cited
	

30%

Algorithm documentation
	

Core algorithm named, complexity stated at realistic size, alternatives compared, and main edge cases listed and tied to requirements or interfaces
	

25%

API contract clarity
	

Key endpoints with inputs, outputs, and errors fully specified; example payload; versioning sentence and breaking-change rule present
	

25%

Technology justification
	

Every major choice has a paragraph referencing all five criteria, naming alternatives and the team's actual experience
	

20%