# Design D1: ASL Learning Assistant

Team: Braden Monnin, Connor Slutsky, Isaac Dowdy

**Goal:** give self-taught ASL learners instant English text and hand-shape feedback from a webcam, without storing their video data.

**Input:** live webcam video of the learner performing an ASL sign.
**Output:** English text translation of the sign, plus feedback on the user's formation of the sign.


**Component:** ML Model



Things I made up 
 - python backend
 - training our model
 - range of inputs for params of request
 - react

**Note**: We are debating between using a pretrained model or training a model ourselves. A pretrained model would save a lot of training time, but would require us to modify the data and results to fit the needs of the model, whereas a self-trained model would require time and resources, but we could define it's specifications to the needs of our application. These questions are answered under the assumption we are using a model that we are designing and training ourselves. 


# Data model, the D1 diagram. 
For at least one core component: entities, the key attributes that matter for behavior (not every column), and relationships with cardinality on both ends. The sketch should fit on one page. Below it, a short table of structural decisions: why a thing is an entity rather than an attribute, why a relationship is one-to-many rather than many-to-many, and which fields are indexed and why. Say whether you chose a relational model or a simpler store, and why.

The application itself will not require persistent storage, as we are not collecting data from the user. The video footage will be processed into frames which will be processed into landmark tensors which will be sent to the machine learning model whose results will be processed before being displayed to the user. No data storage will be required for the core functionality of the application. The following is a description of the local storage that we can use for training the model:

However, for training the model, we will need data storage. The application requires a pretrained model for faster processing and better versioning, so training the model will happen asynchronously from the running application and locally. The data for the signs will have to be videos. We can take inspiration for the data storage format, entities, and relationships from [Google's Kaggle Competition for training a model to classify ASL signs](https://www.kaggle.com/competitions/asl-signs/data). The landmarks are stored in `landmarks/<participantId>/<sequenceId>.parquet` file formats. `.parquet` files are a data storage format in the Apache Hadoop ecosystem, which can be read by the pandas python library. Each `.parquet` file contains row_id, frame, type, landmark_index, and position of the landmarks. There's a separate `train.csv` file that contains the participantId, sequenceId, file path (to the .parquet file), and the sign (or label). We may use this dataset for training as it's freely available, and seems to be plenty large, but we may need to use a GPU (e.g. form Google Colab) for training. 



# Core algorithms
Name the one or two computations that, if slow or wrong, break the product. Document each with the five-part template from the lecture: what it is and the problem it solves, inputs and outputs with exact types, expected complexity at your realistic data size, why this one rather than the alternatives, and edge cases (empty input, duplicates, ties, failure conditions). Clear prose a teammate could implement is the standard; pseudocode is optional.

Prediction is the core algorithm that is essential to the ML Model Component. The components needs to predict in a relatively fast time for the app to be usable. The user would not be happy if they had to spend over 30 seconds waiting for a response from the model to recognize a simple sign. However, prediction is an algorithm that's already implemented by AI/ML frameworks such as TensorFlow or PyTorch. The following attempts to answer the above questions given these conditions: 

<br>

 - **What it is and what problem it solves**
   - The prediction algorithm will be already implemented by the frameworks we choose to use for designing/training the model (e.g. TensorFlow's `.predict()` function). The problem it solves is performing classification or regression with the trained model on (usually) unseen data. 
 - **Inputs and outputs**
   - Inputs will be identical to the format of the training data - which will primarily consist of a tensor of x,y coordinates pertaining to landmarks of the hands (and maybe face). Those coordinate I believe will be integers - corresponding to relative pixel displacement
   - Since this is multi-class classification, outputs will be an integer corresponding to a class (e.g. '0' for 'A', '1' for 'B'...'100' for 'Dog', etc.).
 - **Expected complexity at realistic data size**
   - The complexity of the prediction algorithm will be unknown, and likely depends on the complexity of the model.
 - **Why this one**
   - Each model comes with one and only one prediction algorithm. 
 - **Edge cases**
   - Improper input (e.g. containing NaNs or null values) can be caught before being sent to the model via input validation, so time isn't wasted for the model to catch the error and for us to interpret the panic. This will be outputted to the user as a UI popup saying "An error occurred in reading the signs. Please try again."
   - If an input is duplicated and sent to the ML Model twice, It'll read the duplication as if it were another input and output a classification accordingly. It shouldn't be an error. 
   - Multiple users of the application won't interfere with each other. Each request to the model can be queued so requests that appear at the same time won't interfere with one another. 
   - An internal failure of the model can result in a HTTP Status Code 500 Response or as an exception which would then prompt a UI component, e.g. "An error occuredin determining the signs. Please try again."

# Build-versus-reuse decisions
A table with one row per significant piece of the detailed components: what it is, build or reuse, the library or service if reused, its license, and a one-line reason. Check maturity, licensing, performance, and fit for every candidate library.

| Subcomponent Name | What it is | Build or Reuse | Library or service (if reused) | License (if reused) | Reasoning |
|---|---|---|---|---|---|
| Model | The model to be trained and conduct the precisions | Build | TensorFlow | N/A | Building will let us learn how to design, build, and train a deep learning model for image processing, and let us design the shape of the inputs and outputs |
| Model Input Processor | Processes the HTTP request from the UI client and transforms the data into tensor input for the model | Build | Python request / JSON processing library (built-in Python `json` and `requests` libraries) | N/A | Request/JSON payload processing is simple enough that we won't need to use a tool other than built-in libraries |
| Model Output Builder | Takes the output of the model, identifies the class that it's associated with, and builds the HTTP request and JSON payload to be sent to the Results Processing componnet | Build | Python built-in `json` and `requests` libraries | N/A | Same as above, the implementation doesn't require non-built-in libraries |


# API contract. 
For the detailed component's key endpoints or methods: name, inputs (each parameter's type, required or optional, valid range), outputs (the exact response shape with field types and units), and error responses (status or code, and when it happens). Give one example request and response in a code block. Add one sentence on versioning: what is versioned, how, and what counts as a breaking change.

 - ENDPOINTS
   - `/predict`
 - NAME
   - Predict - form a prediction given the payload data
 - INPUTS - JSON
| PARAM | TYPE | REQUIRED? | RANGE |
|---|---|---|---|
| Face landmarks | Tensor of ints, corresponding to relative position of landmarks of the face | Yes | 0-255, for each entry | 
| Left hand landmarks | Tensor of ints, corresponding to relative position of landmarks of left hand | Yes | 0-255, for each entry |
| Right Hand landmarks | Tensor of ints, corresponding to relative position of landmarks of right hand | Yes | 0-255, for each entry |
Note: the shape of the tensors are unknown, and depend on the data (which we currently don't have)
 - OUTPUTS - JSON
| PARAM | TYPE | RANGE |
|---|---|---|
| Class | String | From list of classes derived from the data |
| Confidence | Float | 0-1 |
| Face landmarks | Tensor of ints | 0-255, for each entry | 
| Left hand landmarks | Tensor of ints | 0-255, for each entry |
| Right Hand landmarks | Tensor of ints | 0-255, for each entry |
 - ERROR RESPONSES
   - 500 - internal server error - internal error within the model prediction
 - EXAMPLE REQUEST
   - Request to `<model_domain>/predict`
   - Payload: 
     - ```json
     {
       "face": [15, 26, 49, 10, 254, 21],
       "left": [15, 23, 10, 43, 12, 43],
       "right": [17, 23, 9, 43, 12, 143]
     }
     ```
 - response
   - Response status code 200
   - ```json
     {
       "class": "Dog",
       "confidence": 0.78,
       "face": [15, 26, 49, 10, 254, 21],
       "left": [15, 23, 10, 43, 12, 43],
       "right": [17, 23, 9, 43, 12, 143]
     }
     ```
   - Response status code 500
   - ```json
     {
       "message": "Internal Server Error",
       "description": "Inputs are wrong shape <insert error from model>"
     }
     ```

 

# Technology choices with justification. 
One paragraph per major choice (database, backend language or framework, front end, messaging or hardware platform, hosting). Each paragraph must reference all five criteria from class: team skill fit, licensing, community support, performance, and cost and hosting. Name the alternative you passed over for at least two of the choices.

- team skill fit
- licensing
- community support
- performance
- cost and hosting
- alternative for two of the choices

- python backend language
- frontend framework - vanilla and react 
- machine learning library tensorflow
- hosting - local and cloud

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
