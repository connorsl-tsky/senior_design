# Design D1: ASL Learning Assistant

Team: Braden Monnin, Connor Slutsky, Isaac Dowdy

**Goal:** give self-taught ASL learners instant English text and hand-shape feedback from a webcam, without storing their video data.

**Input:** live webcam video of the learner performing an ASL sign.
**Output:** English text translation of the sign, plus feedback on the user's formation of the sign.


**Component:** ML Model





# Data model
the D1 diagram. For at least one core component: entities, the key attributes that matter for behavior (not every column), and relationships with cardinality on both ends. The sketch should fit on one page. Below it, a short table of structural decisions: why a thing is an entity rather than an attribute, why a relationship is one-to-many rather than many-to-many, and which fields are indexed and why. Say whether you chose a relational model or a simpler store, and why.

No persistent storage will be required for most of our components. For our ML Model, we look to rely on a pretrained model, meaning we won't need persistent storage. However, if we have time to expand functionality, we may look to include additional trained models. In that situation, our data will be stored locally. The data that we found are all in .parquet files and sorted in directories as the competitions are for simpler accessing. 

need a why.
 - what does the data look like
 - how is it organized

# Core algorithms
Name the one or two computations that, if slow or wrong, break the product. Document each with the five-part template from the lecture: what it is and the problem it solves, inputs and outputs with exact types, expected complexity at your realistic data size, why this one rather than the alternatives, and edge cases (empty input, duplicates, ties, failure conditions). Clear prose a teammate could implement is the standard; pseudocode is optional.

The ML Model needs to predict in a relatively fast time for the app to be usable. The user would not be happy if they had to spend over 30 seconds waiting for a response from the model to recognize a simple sign.   

 - what it is and what problem it solves
   - The prediction algorithm will vary and be unique for the model we're implementing. The algorithm itself is not something that we implement. It's the algorithm that determines what the sign is based on an input of landmark values
 - inputs and outputs
   - the inputs and outputs will be dependent on the models. The inputs will be landmarks of the hands (perhaps a vector of floats), and the outputs will be a prediction of what sign it is (likely an int that corresponds to the class)
 - expected complexity at realistic data size 
   - The complexity of the prediction algorithm will depend on the model used. 
 - why this one
   - Each model comes with one and only one prediction algorithm. 
 - edge cases
   - improper input can be caught before being sent to the model to save time via input validation, so time isn't wasted for the model to catch the error and for us to interpret the panic. I doubt we have to worry aobut duplicate inputs, and since multiple users of the application won't interfere with each other, that also won't be relevant. Even if we structured it so the model was deployed as a backend, each request will be queued so they don't interfere with each other. Failure conditions will be handled depending on the failure - an internal failure of the model can result in a 500 response or as an exception which would then prompt a UI component. A failure in inputs can be caught before reaching the model and displayed as an appropriate UI element (e.g. a popup "There was an issue in tracking your signs. Please try again")

# Build-versus-reuse decisions
A table with one row per significant piece of the detailed components: what it is, build or reuse, the library or service if reused, its license, and a one-line reason. Check maturity, licensing, performance, and fit for every candidate library.

 - 

# API contract. 
For the detailed component's key endpoints or methods: name, inputs (each parameter's type, required or optional, valid range), outputs (the exact response shape with field types and units), and error responses (status or code, and when it happens). Give one example request and response in a code block. Add one sentence on versioning: what is versioned, how, and what counts as a breaking change.

 - key endpoints/methods
 - name
 - inputs
 - outputs
 - error responses

 - exmaple request
 - response

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
