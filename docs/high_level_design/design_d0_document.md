# Design D0: ASL Learning Assistant

Team: Braden Monnin, Connor Slutsky, Isaac Dowdy

**Goal:** give self-taught ASL learners instant English text and hand-shape feedback from a webcam, without storing their video data.

**Input:** live webcam video of the learner performing an ASL sign.
**Output:** English text translation of the sign, plus feedback on the user's formation of the sign.


---
## Block Diagram

https://app.diagrams.net/#Lasl_block_diagram.drawio#%7B%22pageId%22%3A%22asl-block%22%7D

![ASL Learning Assistant Block Diagram](images/asl_block_diagram.drawio.png)

---

## Component responsibility table

| Component | Responsibility | Interfaces in | Interfaces out | Primary owner |
|---|---|---|---|---|
| Frame capture | Holds each webcam frame in RAM only until the Hand tracker has finished with it. | I2 | I3 |  |
| Hand tracker | Finds hand positioning from video frames, finishing with the rest position. | I3 | I4 |  |
| ML Model | Classifies completed signs to its highest confidence match. | I4 | I5, I6 |  |
| ASL to Text Translation | Maps a classified sign label to its English text. | I6 | I7 |  |
| Determine Sign Accuracy | Scores how closely the performed hand shape matches the reference shape for the recognized sign. | I5 | I8 |  |
| Confidence Check | Decides whether a result is trustworthy enough to show, or the sign should be reported as not recognized. | I7, I8 | I9 |  |
| Send Feedback | Presents the final text, hand-shape feedback, or warning to the learner on screen. | I9 | I10 |  |

External (not built by us): **ASL learner** (I1 out, I10 in) and **Webcam** (I1 in, I2 out).

---

## Interface specification table

| ID | Inputs | Outputs | Output Data format | Output Protocol | Error handling |
|---|---|---|---|---|---|
| I1 | Hand signs (physical movement) from ASL Learner (User) | Hand signs (physical movement) to Webcam | N/A - handled by hardware | N/A - handled by hardware | Error in webcam - outputted to client via Feedback, unrecognized hand signs - low confidence score from Confidence Checker outputed via Feedback |
| I2 | Visual information (video) from Webcam | Visual information (video) to Frame Capture | N/A - handled by operating system | N/A handled by operating system | Corrupted video - caught by Frame Capture and outputted eventually via Feedback | 
| I3 | Frame object (JPEGs) from Frame Capture | Frame object (JPEGs) to Hand Tracker | JPEG, encoded in JSON | HTTPS | Error in dividing video to frames - caught by Frame Capture and outputted via Feedback | 
| I4 | Landmarks (tensor) of hands from Hand Tracker | Landmarks (tensor) of hands to ML Model | Tensor / JSON | Direct transfer / HTTP (REST) | Landmarks not detected - caught by hand tracker and outputted eventually via Feedback |    
| I5 | Label (string) and landmark sequence (tensor) from ML model | label and landmark sequence (vector) to Sign Accuracy Determiner | Vector | Direct transfer | Gaps in landmark sequence caught by Sign Accuracy Determiner |
| I6 | Sign label (string) / confidence (double) from ML Model | Sign label / confidence (vector) to Translator | Vector / JSON | Direct transfer / HTTP (REST) | Unrecognized sign caught by ML Model | 
| I7 | Text (string) and confidence (double) from Translator | text and confidence (vector) to Confidence Checker | Vector | Direct transfer | Translation error (e.g. unable to translate) caught by Translator |
| I8 | Scores (tensor) from Sign Accuracy Determiner | Scores (tensor) to Confidence Checker (no change) | Tensor | Direct transfer | Direct transfer without change won't cause errors? | 
| I9 | Text (string) and shape scores (tensor) from Confidence Checker | Text and shape scores (object) to Feedback (client) | JSON | HTTPS | Error in inputs in Confidence Checker caught and transferred to Feedback |
| I10 | Text (string) and shape scores (tensor) from Feedback | text (string) and shape scores (string) outputted to ASL Learner / User | Strings | Browser display | Error in processing text and scores caught by Feedback and displayed to User |

Notes / Assumptions: 
 - I3 - hand tracker might be backend
 - I3 - hand tracker frames should be JPEG since we don't need the pixel-perfect accuracy of PNG, but alternatively we could also use something like TFRecord/HDF5/Tar to bundle images together. I'm not sure if that means you can group images together (e.g. if they're the same sign).
 - I3 - we can start development locally with http, but https is probably preferred for a full deployment for the security
 - I4 - hand tracker and ML model might both be backend components, and if they're in the same source code file you could pass the landmarks directly to the model. Otherwise, if we want to split them into separate services we can use HTTP / REST API. 

### Example 

---

## Data-flow diagram

https://app.diagrams.net/#Lasl_data_flow_diagram.drawio#%7B%22pageId%22%3A%22asl-dfd%22%7D

![ASL Learning Assistant Data-Flow Diagram](images/asl_data_flow_diagram.drawio.png)

---

## Architecture pattern and justification


---

## Decision log

