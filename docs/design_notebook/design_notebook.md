
# 10/7/26 Meeting Minutes

### Attendance

Braden, Connor, Isaac

### Purpose

Talk about Assignment 5 (high level design), and start planning for Assignment 6 (due tomorrow) and Assignment 7 (due next Wednesday 10/14/126) (both detailed system design)


### Decisions

 - Assignment 5
   - We combined the components in A5 from 9 to 5 (Video Capture, Input Processing, ML Model, Results Processing, and Feedback/UI)
     - This will reduce future workload, and simplify the components that were pretty similar
   - ML model may need enough development to be considered internal
     - Braden said he found some models, but they may not have everything that we need, so we may need to augment them
     - don't want too many of our components to be external. currently 2/5
 - Design
   - try to use existing ML model if possible - haven't found one that 100% what we want - e.g. max 250 ASL signs
   - MediaPipe - library/framework for hand tracker, some models that utilize it
     - html5 media devices api as input
   - web assembly - we can use this for higher processing for web browser stuff
   - we are settling on web browser for finalized deployment
     - don't have to worry about confusing OS system calls, don't have to sacrifice portability, seems to be relatively straightforward in terms of video capture, fits well into a client/server approach, gives us options in terms of either all javascript, or javascript and some backend (e.g. python)
   - PWA - multithreaded web app in browser - offline local application without .exe
   - github pages link to download everything and run on device - serverless
   - probably don't need to deploy it in cloud ever unless we love the result

### Tasks

 - Assignments 6 
   - for A6, ML Model component is best
     - it will probably have a data component and algorithms that are asked by the assignment
   - I can do ML model by tomorrow
   - Isaac has some time in hte evening if i don't finish
   - Braden can help a little with research
   - get into submittable but not perfect state by tomorrow
 - Assignment 7
   - Braden can do send feedback and Isaac can do results processing
     - these assignments are not strict
   - I will continue to polish/expand ML Model
   - for external components - last, and whoever finishes their components first will do those
   - aim to finish A7 stuff by next wednesday
     - meet around same time
 - Aurisano
   - we talk to Aurisano a little a week after next week (10/21/26, ~220pm), and meet her and potentially schedule a meeting
     - office hours are right after class monday/wednesday
     - class is UI
 - Other tasks
   - i can fix the interfaces table for v2 - but low priority
   - i can create a github org and migrate repo to their - sometime soon
   


# 9/9/26 Meeting Minutes

### Attendance

All three team members  

### Purpose

Narrow down project topics and begin talking about advisor ideas  

### Results

We narrowed down to ASL to text translator framed as a tool to help people learn ASL (see topic statement)  

Runner up ideas include the Scheduler, or interactive calendar. Framed as an improvement to when2meet.com, it could move commitments around dynamically if plans change, would need integration with existing calendars (outlook, google, apple), maybe include AI optimizations by learning other people's calendars when something would be best to schedule, can create plans considering others class schedules, and could potentially read from Discord/Slack to dynamically update calendars based on messages. These were all possible ideas, and not features, but IIRC none of us were super stoked about it as it didn't really stand out compared to existing applications  

Another runner up idea was campus navigation, which would help people find their classes and find the best route between buildings, but the bearcat app / my bearcat network already has a tool to traverse buildings, and it would be very difficult to traverse between classes as it would either be scraping building plans or tediously walking around buildings, plus it would be difficult to deal with construction or changing routes or rooms moving and that kind of stuff, so it was bumped down

the budgeting app and skill gap app (take in resume or curriculum, scan local job listings, and determine what skills you should work on or how to improve your resume to better improve your job listings) was also notable, but we would have to compete with chatgpt in terms of functionality so it wouldn't be much of an improvement to something. 

over the next week we'll separately research and investigate advisors, then share via text what our ideas were then start reaching out, one at a time


(taken by connor on 9/16, not really sure what meeting minutes look like but yeah feel free to edit it)
