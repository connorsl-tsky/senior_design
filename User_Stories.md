# User Stories and Use Cases

Team Members: Braden Monnin, Connor Slutsky, Isaac Dowdy

## User Stories

US-01 (primary): As a self-taught ASL learner, I want to see an English text translation from the sign I just performed, so that I can confirm the system understood my intended meaning. 

US-02 (primary): As a beginner ASL learner, I want to receive visual feedback on my hand shape accuracy while practicing, so that I can correct my mistakes before they become ingrained habits.

US-03 (secondary): As a Deaf conversation partner, I want the system to catch improper finger positions and hand shapes, so that I can easily understand the learner when we communicate in real life.

US-04 (hidden): As a State Privacy Regulator, I want the application to immediately discard webcam frames from RAM after processing rather than saving the user's body and hand geometry, so that the software legally complies with strict state biometric privacy laws.

## Use Case

UC-01  
Expands US-01(Primary Stakeholder: Self-taught ASL learner)  
Name: Translate ASL Sign to English Text  
Primary Actor: Self-taught ASL learner  
Secondary Actors: None  
Preconditions: The application is loaded, webcam permissions have been granted, and the user is clearly visible in the camera frame with adequate lighting.  
Main Success Flow:

1. Actor clicks to initiate the session.  
2. System activates the webcam and displays the live video feed.  
3. Actor performs ASL sign in front of camera.  
4. System captures the frames, extracts the hand landmark coordinates, and maps the spatial sequence through the machine learning model.  
5. Actor pauses or lowers their hands, signaling the completion of the gesture.  
6. System matches the gesture to a known sign with a high confidence score and displays the corresponding English text translation on the screen.

Alternate Flow (Low Confidence Match): At step 6, the system detects hand movement but cannot match it to a known sign in its database with high enough confidence. The system displays a "Sign not recognized, please try again" prompt rather than guessing an incorrect English word.

Exception Flow (Subject Out of Bounds): At step 4, the user's hands move completely outside the webcam's field of view mid-sign. The system halts the processing loop and displays a visual warning reading "Hands out of frame. Please move back."

Postcondition: The system has successfully generated text feedback for the performed gesture and resets its tracking state, ready for the user to perform the next sign.

## Acceptance Criteria

AC-01.1:   
Given the application is in an active practice session with the webcam running, and the user's hands are detected by the tracking overlay.

When the user performs a sign present in the training database, such as “Hello”, and then drops their hands to a resting position for at least 1 second.

Then the system must display the exact English text translation ("Hello") on the screen within 3 seconds of the hands returning to the resting position.

AC-01.2:   
Given the application is in an active session and tracking the user's hands.

When the recognized hand landmark coordinates move outside the webcam frame for more than 1 second.

Then the system must pause the text translation output and display a warning that hands are out of frame until hand landmarks are reacquired.