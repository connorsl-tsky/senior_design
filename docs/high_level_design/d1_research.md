# Research


i think first we want to establish what our expectations are
 - i'm thinking we start with just recognition of fingerspelling + a small set of signs
 - and we can gradually add more complexity such as adding an NLP model to turn the words into sentences or something like that

and i think next we need to find if there are any pretrained models or not
 - the kaggle ones likely aren't 
 - the tflite ones are

so we can make a decision with a possibility within the next 15 minutes

we can start with a pretrained model, so we just have to input data that is relevant to that model
would it be possible to implement another model that would expand the output of the first one?
how would we know whether to send the input to one or the other?
especially if the configuration is different
it's possible that we can compare the confidence across all the used models and choose the highest one - but that can be prone to failure

it's possible to train a model, but with a large dataset that might be challenging since we don't have the best computers
 - probably would take quite a while too

so we want to avoid training a model since that would add a lot of time to the development

what are our options for pretrained models?

i think Braden's model, the same as the top model from the competition is a good choice. 
does it also include fingerspelling?

we can probably use that pretrained model originally, with the option to expand it?
yeah, it seems possible and feasible to expand it, i like the claude method of UI to separate various models, perhaps



what do i need
 - what models are we using?
   - need inputs and outputs
 - analyze various datasets and what are we using




https://deepmind.google/blog/putting-sign-language-ai-into-users-hands/   
 - Google DeepMind - not a very useful article, but this kinda stuff is being done / worked on on a large scale

<br>

https://huggingface.co/ColdSlim/ASL-TFLite-Edge
- python tensorflowlite model - 59 ASL classes (is that signs?), something existing that we can try
- also pretrained, doesn't appear to be trainable

<br>

https://github.com/ashgadala/american-sign-language-detection
 - for ASL letters
 - pretrained but trainable
 - trainable ?

<br>

options for level of difficulty we should shoot for
 - just fingerspelling
   - probably too easy
 - fingerspelling words + small set of signs - just recognize
   - this would be a good first start
   - just be able to recognize a string of fingerspelling into words
   - and be able to recognize a small set of signs (ones without facial expressions)
 - fingerspelling words + small set of signs - sentences
   - this might be a good place to shoot for if we can do the above one
   - the '- sentences' means that it translates the input into letters and pieces them into words
   - model would output series of letters and words, we need something to turn the letters into words and the words into a coherent sentence - may need an NLP model in Results Processing
 - fingerspelling + many signs
   - less feasible
 - full translations
   - probably not feasible

<br>

kaggle ASL alphabet - dataset
 - https://www.kaggle.com/datasets/grassknoted/asl-alphabet
 - free to download (pretty sure most things on kaggle are), ~1.1GB

<br>

kaggle ASL Fingerspelling recognition competition (2023)
 - https://www.kaggle.com/competitions/asl-fingerspelling/code
 - has data, the best models achieved a score (not sure what stat) of ~0.8
 - https://www.kaggle.com/code/gusthema/asl-fingerspelling-recognition-w-tensorflow
 - notebook to walk through loading data and training on tensorflow transformer

<br>

kaggle google - isolated sign language reognition competition - 250 signs
 - https://www.kaggle.com/competitions/asl-signs
 - see above, this is for signs as a whole, not just fingerspelling
 - top place ~0.89
 - https://www.kaggle.com/competitions/asl-signs/writeups/hoyeol-sohn-1st-place-solution-1dcnn-combined-with
 - he published his code!!!! or model at least. something we could use
 - most of the top teams published their code - at the very least it would be models we don't have to train
 - (though that doesn't say anything about how long it would take to make a prediction)
 - https://www.kaggle.com/code/hoyso48/1st-place-solution-inference/output - tflite model?
 - the output is always a tflite model?

<br>

WLASL - dataset

<br>

ASL Citizen - dataset

<br>

MS-ASL - dataset

<br>

how2sign - dataset

<br>

signCLIP - research model (likely out of scope)

<br>

uni-sign - reserach model (likely out of scope)

<br>

ONNX

<br>

TFLite
 - https://www.geeksforgeeks.org/deep-learning/introduction-to-tensorflow-lite/
 - deploy trained machines on edge devices (phone, iot, embedded)
 - this gfg includes example implementation
 - i believe some of the other models in this list were deployed in tflite, so this might be a good resource

<br>

winning model for the kaggle - google isolated signs competition referenced above
 - https://huggingface.co/sign/kaggle-asl-signs-1st-place
 - braden says isolated words get more difficult to recognize with large vocabulary sets since they look more and more similar
   - something to keep in mind


<br>

what is a .parquet file
 - https://en.wikipedia.org/wiki/Apache_Parquet
 - column-oriented data storage format from apache hadoop

## claude

sonnet 5.5 medium effort

### prompt 1
Hello. For a university computer science capstone project, I (and some project-mates) are working on an ASL to text translator tool that would take webcam video footage via a web browser of a user signing and be able to translate that to text. For the model itself, we ideally want to use an existing model, but I'm having trouble finding some, if they exist. I wanted to ask if you knew any. Here's what I found so far:

* https://github.com/ashgadala/american-sign-language-detection

https://huggingface.co/ColdSlim/ASL-TFLite-Edge  
(didn't finish the prompt lol)

### response 1
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



# prompt 2

We found a couple pretrained TFLite models that meet our expectations. I'm thinking of starting off with recognizing fingerspelling and a small subset of signs with a possiblity of expanding the set of signs over time. How feasible is the ability to take a pretrained model and add another pretrained model to expand that functionality? Or is it one of those things where once we pick a model we need to settle on it?

# response 2

You don't have to settle on one model, but you can't merge them either. The distinction matters for your architecture.

**What doesn't work:** merging or stitching the weights of two pretrained models into one bigger model. Their internal representations aren't compatible, and with TFLite files (inference-only, quantized) it's essentially off the table.

**What does work: running several models side by side.** Since your pipeline is landmarks-in, label-out, you can share one MediaPipe extraction stage and feed its output to multiple classifiers:

```
webcam → MediaPipe landmarks → adapter A → fingerspelling model
                             → adapter B → isolated-sign model
                             → adapter C → (future model)
```

This is very feasible and is probably the best design for your plan. Fingerspelling and isolated signs are different tasks anyway, so separate specialized models usually beat one model trying to do both.

**Where the difficulty actually lies:**

1. **Input mismatches.** Each model expects a specific landmark subset, ordering, normalization (centering, scaling), and sequence length. One model might want 543 holistic landmarks while another wants only hands. Each needs its own small preprocessing adapter. Also check which MediaPipe version produced the training landmarks. The Kaggle models used the legacy Holistic solution, and the newer `tasks-vision` landmarkers can differ slightly in output, which can quietly hurt accuracy.
2. **Deciding which model's answer to use.** If both models run constantly, you need a rule for when to trust which. Options, from simplest to most complex:
   - A UI toggle between "fingerspell" and "signs" mode (easiest, and a fine capstone choice).
   - A heuristic gate, such as hand position or motion patterns, since fingerspelling tends to be small, held hand shapes near the shoulder.
   - A small classifier trained to route between modes (more work).
3. **Confidence isn't comparable across models.** A 0.8 softmax from one model doesn't mean the same as 0.8 from another, so avoid simply picking the highest score across models.
4. **Browser performance.** Running two or three models per frame on top of MediaPipe adds latency. Running only the active model (via the mode approach above) or running classifiers every few frames helps.

**Expanding the vocabulary of a single model** is harder than adding a second model. A TFLite file generally can't be fine-tuned. To add signs to an existing model, you'd need its original training code and weights (Keras or PyTorch), then retrain or fine-tune with a bigger output layer on a dataset like ASL Citizen or WLASL. Some Kaggle winners released this, so check each model's repo. If it's only a `.tflite` file, treat it as frozen.

**Practical recommendation:**
- Build a small `Model` interface in your code (`preprocess(landmarks)`, `predict()`, `labels`) so each model is a plug-in. Adding one later then means writing one adapter, not rewriting the app.
- Start with the mode toggle, and add automatic switching only if time allows.
- Keep a record of each model's license, training data, and expected input spec as you adopt it, since that saves a lot of debugging later.
- If the isolated-sign vocabulary ends up too limited, the long-term path is retraining on your own or public data, so find out now whether the source repos include training code.

If you tell me which two models you've found, I can help you compare their input specs and figure out how much adapter work they'd need.