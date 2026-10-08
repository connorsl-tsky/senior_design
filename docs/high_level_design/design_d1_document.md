# Design D1: ASL Learning Assistant

Team: Braden Monnin, Connor Slutsky, Isaac Dowdy

**Goal:** give self-taught ASL learners instant English text and hand-shape feedback from a webcam, without storing their video data.

**Input:** live webcam video of the learner performing an ASL sign.
**Output:** English text translation of the sign, plus feedback on the user's formation of the sign.


**Component:** ML Model

**Note**: We are debating between using a pretrained model or training a model ourselves. A pretrained model would save a lot of training time, but would require us to modify the data and results to fit the needs of the model, whereas a self-trained model would require time and resources, but we could define it's specifications to the needs of our application. These questions are answered under the assumption we are using a model that we are designing and training ourselves.


# Data model, the D1 diagram. 

**NOTE: We do not need persistent data storage for our application, and the reasoning for that is specified below. As such a description of entities, attributes, and relationships are not relevant for our project. Thus, we do not include a diagram. Instead, we discuss the details of local file storage for the training of our model.**

The application itself will not require persistent storage, as we are not collecting data from the user. The video footage will be processed into frames which will be processed into landmark tensors which will be sent to the machine learning model whose results will be processed before being displayed to the user. All data will be stored locally and in motion. No persistent data storage will be required for the core functionality of the application. The following is a description of the local storage that we can use for training the model:

For training the model, we will need local data storage. The application requires a pretrained model for faster processing and better versioning, so training the model will happen asynchronously from the running application, and it will happen locally. The data for the signs will have to be videos. We can take inspiration for the data storage format, entities, and relationships from [Google's Kaggle Competition for training a model to classify ASL signs](https://www.kaggle.com/competitions/asl-signs/data). The landmarks are stored in `landmarks/<participantId>/<sequenceId>.parquet` file paths. `.parquet` files are a data storage format in the Apache Hadoop ecosystem, which can be read by the pandas python library. Each `.parquet` file contains row_id, frame, type, landmark_index, and position of the landmarks. There's a separate `train.csv` file that contains the participantId, sequenceId, file path (to the .parquet file), and the sign (or label). We may use this dataset for training as it's freely available, and seems to be plenty large, but we may need to use a GPU (e.g. form Google Colab) for training. 



# Core algorithms

Prediction is the core algorithm that is essential to the ML Model Component. The component needs to predict in a relatively fast time for the app to be usable. The user would not be happy if they had to spend over 30 seconds waiting for a response from the model to recognize a simple sign. However, **prediction is an algorithm that's already implemented by AI/ML frameworks such as TensorFlow or PyTorch**. The following attempts to answer the above questions given these conditions: 

<br>

 - **What it is and what problem it solves**
   - The prediction algorithm will be already implemented by the frameworks we choose to use for designing/training the model (e.g. TensorFlow's `.predict()` function). The problem it solves is performing classification (in our case) with the trained model on unseen data. 
 - **Inputs and outputs**
   - Inputs will be identical to the format of the training data - which will primarily consist of a tensor of x,y coordinates pertaining to landmarks of the hands (and maybe face). Those coordinate I believe will be integers - corresponding to relative pixel displacement
   - Since this is multi-class classification, outputs will be an integer corresponding to a class (e.g. '0' for 'A', '1' for 'B'...'100' for 'Dog', etc.).
 - **Expected complexity at realistic data size**
   - The complexity of the prediction algorithm will be unknown, and likely depends on the complexity of the model. a CNN model will have a more complex prediction operation than a Linear Regression model.
 - **Why this one**
   - Each model comes with one and only one prediction algorithm. There are no alternatives
 - **Edge cases**
   - Improper input (e.g. containing NaNs or null values) can be caught before being sent to the model via input validation, so time isn't wasted for the model to catch the error and for us to interpret the panic. This will be outputted to the user as a UI popup saying "An error occurred in reading the signs. Please try again."
   - If an input is duplicated and sent to the ML Model twice, it'll read the duplication as if it were another input and output a classification accordingly. It shouldn't be an error. 
   - Multiple users of the application won't interfere with each other. Each request to the model can be queued so requests that appear at the same time won't interfere with one another. 
   - An internal failure of the model can result in a HTTP Status Code 500 Response or as an exception which would then prompt a UI component, e.g. "An error occuredin determining the signs. Please try again."

# Build-versus-reuse decisions

| Subcomponent Name | What it is | Build or Reuse | Library or service (if reused) | License (if reused) | Reasoning |
|---|---|---|---|---|---|
| Model | The model to be trained and conduct the precisions | Build | TensorFlow | N/A | Building will let us learn how to design, build, and train a deep learning model for image processing, and let us design the shape of the inputs and outputs |
| Model Input Processor | Processes the HTTP request from the UI client and transforms the data into tensor input for the model | Build | Python request / JSON processing library (built-in Python `json` and `requests` libraries) | N/A | Request/JSON payload processing is simple enough that we won't need to use a tool other than built-in libraries |
| Model Output Builder | Takes the output of the model, identifies the class that it's associated with, and builds the HTTP request and JSON payload to be sent to the Results Processing componnet | Build | Python built-in `json` and `requests` libraries | N/A | Same as above, the implementation doesn't require non-built-in libraries |


# API contract. 

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
```json
{
  "face": [15, 26, 49, 10, 254, 21],
  "left": [15, 23, 10, 43, 12, 43],
  "right": [17, 23, 9, 43, 12, 143]
}
```
 - response
   - Response status code 200
```json
{
  "class": "Dog",
  "confidence": 0.78,
  "face": [15, 26, 49, 10, 254, 21],
  "left": [15, 23, 10, 43, 12, 43],
  "right": [17, 23, 9, 43, 12, 143]
}
```
   - Response status code 500
```json
{
  "message": "Internal Server Error",
  "description": "Inputs are wrong shape <insert error from model>"
}
```
   - Versioning
     - We will use semantic versioning (e.g. X.Y.Z) with patches incrementing Z, minor changes incrementing Y, and breaking changes incrementing X. Breaking changes involving endpoints will necessitate a /v2 subpath for endpoints (e.g. `/v2/predict`)

 

# Technology choices with justification. 

We will use python as a backend language. All of our team has experience using Python, and Python is a cornerstone language for AI/ML for it's associated libraries and frameworks. Python, I believe, has it's own license that is open-source and GPL compliant (https://docs.python.org/3/license.html). Python has significant community support, but is known for having performance issues compared with lower-level languages such as C++. I think this is more than made up for it's significant community support and powerful AI/ML libraries. Python has no cost associated with it.

For the frontend, we will be using React JavaScript. Although the frontend won't have major functionality, React nevertheless simplifies UI/UX development and opens the door for potential future expansion of the UI. One of our team members has significant experience with React, while another has experience with JavaScript, so team skill fit isn't optimal. However, since we're deploying our app in the browser, we have to use JavaScript or a JavaScript framework, so it's our best option. React has Creative Commons licensing (https://github.com/reactjs/react.dev/blob/main/LICENSE-DOCS.md) and is free and open source. React is known for being slower client-side for it's Client-Side Rendering, but our app will be simple enough that it shouldn't add significant latency. Alternatively, we considered vanilla JavaScript, as our UI won't be complex and utilizing a framework such as React may add unnecessary complexity, but we decided otherwise for the one major reason. That reason was the possibilty of future expansion of the UI complexity, where vanilla JavaScript is difficult to use with a complex UI compared with a framework, so implementing a framework early opens that opportunity. 

For the AI/ML framework, we settled on Python's TensorFlow. This decision is also sub-optimal in terms of team skill fit, as only one of our members as experience with TensorFlow, but none of us have significant experience with AI/ML frameworks as a whole. TensorFlow has Apache licensing (https://github.com/tensorflow/tensorflow/blob/master/LICENSE) and is free and open source. It also has significant use in the AI/ML community with plenty of documentation. In terms of performance, we considered PyTorch, but TensorFlow and PyTorch have similar performance, and Tensorflow is said to be slightly more efficient memory-wise for large models. That and none of us having experience with PyTorch led us to our decision. 


For hosting, for the vast majority of our development our hosting will be 100% local. This is to prevent unnecessary cost accumulation from deployment via a cloud management service. However, for the final deployment, we settled on Azure as our choice of cloud deployment, mostly as a result of two of our members having experience with Azure. Azure is proprietary software, but has a free-tier version that we intend to utilize. There may be small cost accumulation however, if demand exceeds a specific threshold. For performance, Azure's main strength is enterprise-level deployment, which is not our intended level of deployment (AWS is better for granular deployments), but due to the team skill fit, we thought Azure would be sufficient enough. 
