# Design D1: ASL Learning Assistant

Team: Braden Monnin, Connor Slutsky, Isaac Dowdy

**Goal:** give self-taught ASL learners instant English text and hand-shape feedback from a webcam, without storing their video data.

**Input:** live webcam video of the learner performing an ASL sign.
**Output:** English text translation of the sign, plus feedback on the user's formation of the sign.

TDL
 - [ ] do some research about possible existing models and how they work - inputs, outputs, details that may be important for augmentation
 - [ ] running down the sections

# Research

https://deepmind.google/blog/putting-sign-language-ai-into-users-hands/   
 - Google DeepMind - not a very useful article, but this kinda stuff is being done / worked on on a large scale

https://huggingface.co/ColdSlim/ASL-TFLite-Edge
- python tensorflowlite model - 59 ASL classes (is that signs?), something existing that we can try

https://github.com/ashgadala/american-sign-language-detection
 - for ASL letters
 - trainable ?

## ask claude

### prompt
Hello. For a university computer science capstone project, I (and some project-mates) are working on an ASL to text translator tool that would take webcam video footage via a web browser of a user signing and be able to translate that to text. For the model itself, we ideally want to use an existing model, but I'm having trouble finding some, if they exist. I wanted to ask if you knew any. Here's what I found so far:

* https://github.com/ashgadala/american-sign-language-detection

https://huggingface.co/ColdSlim/ASL-TFLite-Edge
(didn't finish the prompt lol)

### response
Existing models do exist, but how useful they are depends on what you mean by "ASL to text." There are three tiers of difficulty, and pretrained options thin out quickly as you go up.

**1. Fingerspelling and alphabet (easiest)**
- Many small projects classify static letters (A-Z), usually from the Kaggle "ASL Alphabet" dataset. These are often CNNs on cropped hand images, and they're fragile across lighting, skin tones, and backgrounds. J and Z involve motion, so static classifiers can't handle them.
- The Kaggle *ASL Fingerspelling Recognition* competition (2023) has open-source winning solutions that recognize fingerspelled phrases from landmarks, and many were exported to TFLite. These are much stronger than the alphabet classifiers.

**2. Isolated signs (a realistic target)**
- Kaggle's *Google - Isolated Sign Language Recognition* competition covered about 250 signs. The top solutions are public, with write-ups and notebooks, and they take MediaPipe landmarks as input, which suits a browser pipeline well. The TFLite model on your Hugging Face link may well come from this lineage, but I haven't verified that, so check its model card.
- Datasets for training or fine-tuning: **WLASL**, **ASL Citizen** (crowdsourced, more diverse signers), and **MS-ASL**.

**3. Continuous signing to English sentences (hardest)**
- This is still an open research problem. Look at **How2Sign** (the main continuous ASL dataset) and research models like SignCLIP and Uni-Sign. They're research-grade, heavy, and not realistically browser-ready or accurate enough for a polished demo.

**Browser architecture**

The most practical approach is a two-stage pipeline:
1. **MediaPipe** (Hand/Pose/Holistic Landmarker via the `@mediapipe/tasks-vision` JS package) extracts landmarks in real time in the browser.
2. A **small classifier** (TFLite or ONNX, run through TensorFlow.js or ONNX Runtime Web) takes a window of landmark frames and outputs a sign or letter.

Working on landmarks instead of raw pixels makes the model smaller, faster, and less sensitive to lighting and background. You can also fine-tune on your own recorded data.

**Suggestions for scoping**
- Set expectations early: a vocabulary of a few dozen to a couple hundred signs, plus fingerspelling, is a solid capstone result. Full ASL translation isn't achievable with current open models.
- Check the licenses of any model or dataset you adopt, since some research datasets restrict use.
- ASL grammar relies on facial expressions, body movement, and spatial referencing, not just hand shapes, so word-by-word output will never be a true translation. Mentioning this in your write-up will strengthen it.

I haven't looked at the two links you found, so I can't vouch for their quality. If you'd like, I can review them, including license, training data, and whether they use landmarks or raw images, or help sketch the browser pipeline. Which sign vocabulary are you aiming for?




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
