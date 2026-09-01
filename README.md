# Bonsai-TargetPractice

Test project for target-acquisition. Tracks coloured object (effector) through camera capture, spawns targets on screen, registers hits when target &amp; effector overlap, and scores via mean time-to-hit over 10 rounds exported to CSV, alongisde target &amp; hand position for each round.

Built over a day with no prior exposure to Bonsai or Rx. Webcam & keyboard only. This is a learning tool, not a finished product. As such, this write-up will cover functionality as well as mistakes. 


# Function

Colour-tracked object (in test case, pink sticky note), acts as an effector. A target ring appears at a random position on the screen. When the effector's centroid crosses the ring for a sustained interval, a hit is registered, score increments, and new target is generated at new random position. After 10 hits, game is ended and mean time-to-hit is displayed. 

Everything is canvassed onto the camera feed, so participant can see themselves, effector, and target in mirrored view. 

Three CSV files are written during the game:
  1. Hits (timestamp + target position)
  2. HandPos (timestamp + sticky note position at spawn of new target)
  3. Score (mean time for target hit)

The graphical pipeline can be found here:
https://postimg.cc/t7XVrwmK
Or by loading the file directly into the Bonsai GUI.


# Why

I wanted to get a feel for bonsai using a tried-and-tested paradigm in motor control and neuroscience (centre-out reaching). It is a simple experiment, it enables simple follow-up calculations (like Fitt's Law for human movement), and it allows for a great deal of optional complexity in future iterations of the project. I come with several years of C# experience from Unity game development, which  did more harm than good in this case. Bonsai is:
  1. Right to left
  2. Not single threaded
  3. Not globally timed (no deltaTime, no global tick)
  4. Not designed for variables

These are by no means criticisms, and in learning these aspects of the tool I got a lot of things wrong:

From my experience, Unity tends to hold your hand through errors. Bonsai does not. When nothing is detected, LargestBinaryRegion will still emit a value, and all elements downstream will attempt to operate on it. It requires a greater attention to detail that I personally found far more enjoyable. Like bowling without the guardrails for the first time. It took a little getting used to, but in the fixes introduced me to Conditions & Filters which I imagine are a staple of any Bonsai project worth its salt.

A bigger problem I ran into was property mappings and triggers. I had expected to map inputs to externalised properties of an operator, and for that to trigger the operator to emit. When it didn't - and without the aforementioned error support - it took a little while to differentiate between property input and data input. I switched out inputs and transformers but nothing changed. I then connected my data source to both the externalised property inputs, AND the operator inputs, and found it worked perfectly. This initially felt like a hack to me, until I read the documentation and found that no, this was working exactly as intended.

Many smaller problems I encountered just served to broaden my understanding of Bonsai's offerings. 
  - DistinctUntilChanged fires on every change. Target hits included exits as well as entries. To solve this we can add a       condition to output only the true results, and a throttle to to add a minimum time of overlap for scoring.
  - TimeInterval gives a timespan, I did not find TotalSeconds or TotalMilliseconds initially, and was thus confused why        the time wrapped every 60 seconds.
  - HsvThreshold does not check what feed it is provided. Make sure to ConvertColor to Hsv before setting your threshold,       as you may be operating on a BGR feed. 


# Data Analysis

I ran 9 sessions, target radius at 35px, single participant, fixed indoor lighting. One session excluded due to tracking errors during recording. 
Targets advance on hit, thus the effector is at target N when target N+1 appears. Distance can therefore be calculated as separation between target N and target N+1. 90 hits, 81 trials (first trial of each run removed due to set up time. Call it a practice round.)

Mean time between onset and hit (MT) varied from 0.62 to 0.82 across sessions. Speed increased during progress of each session. Standard Deviation (SD) 0.184s. 

Index of Difficulty (bits) for Fitt's law, ID = log₂(2D/W), where D is distance to target and W is width. As per Fitt's hypothesis, movement time increases with target difficulty, and we can test that by changing the distance to the target (as shown), or changing the width of the target (an exciting follow-up!).

From distance alone, removing the between-session practice effect (trying -14 ms per session), the slope is +0.067s/bit, showing a moderate increase in time per 'bit' of difficulty. 95% CI [0.021, 0.112], p=0.005. Rig produces clean, time-stamped data. Hypothesis proved.


# What's next
Honestly Bonsai is incredibly fun to play around with. You can see successes clearly, and get to diagnose problems on your own. In my opinion the mark of a good tool is the unconscious desire to imagine different ways to use it, and I found myself doing so on repeat. Within this experiment - constrain reaching to a fixed plane of known distance, rebuild for hand tracking, replace random target placement with designed distances and widths. Beyond it, finger tracking, creating 2D shapes from 3D movements, kinematics calculations, to name a few.

# Repository Contents


```
workflow/       BonsaiTrackTest.bonsai
data/           HitList.csv, HandPos.csv, FinalScore.csv
figures/        screenshots, end-of-game screen
```

Bonsai 2.9.x, Windows, standard webcam.

