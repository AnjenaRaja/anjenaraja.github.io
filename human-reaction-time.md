---
layout: page
title: Human Reaction Time
subtitle: An Experiment
---

# Which sensory input (Light/Sound/Touch) does a person react to fastest?
### Dependent Measure: Response time in seconds
### Independent Variable: Type of sensory input (Light/Sound/Touch)

## Hypothesis 
### I think a person will react fastest to sound because sound grabs my attention quickly.

## Theory-How Ears/Eyes/Skin Works
### Ear
1. Pinna directs sound to the ear canal and hits the eardrum
2. Eardrum starts vibrating and the vibrations are amplified by the ossicles and then pass it into the cochlea
3. Cochlear hair cells convert the wave vibrations into electrical signals
4. Signals are then transmitted through the auditory nerves to the brain

![Figure-1](/assets/img/hrt/fig-1.png){: .mx-auto.d-block :}

### Eyes
1. Light enters the eye through the cornea and it passes through the pupil
2. The amount of light passing through is regulated by the iris and it hits the lens
3. The lens focuses light rays onto the retina that has light sensitive nerve layer at the back of the eye
4. The optic nerve carries signals of light, dark, and colors to the brain
   
![Figure-2](/assets/img/hrt/fig-2.png){: .mx-auto.d-block :}
   
### Skin
1. Mechanoreceptors respond to stimuli such as tension, pressure or vibration
2. On receiving a stimuli, the mechanoreceptor pacinian corpuscle reacts by transmitting an impulse at a certain frequency 
3. This impulse is carried by the nerve endings to the brain

![Figure-3](/assets/img/hrt/fig-3.png){: .mx-auto.d-block :}

### Brain 

![Figure-4](/assets/img/hrt/fig-4.png){: .mx-auto.d-block :}

## Theory-Object Falling Freely

### Acceleration Due To Gravity

* If the effects of air resistance are ignored, any object dropped near Earth’s surface will freely fall with constant acceleration g towards the Earth’s center
* The magnitude of g is approximately 9.81 m/s2 and is a constant
* For an object initially at rest, when falling freely in a gravitational field,  if we can measure the distance d of its fall, then we could calculate the time t elapsed using the formula
  
![Figure-5](/assets/img/hrt/fig-5.png){: .mx-auto.d-block :}

* I had to calculate the time t based on the measured distance d for the reasons:
* I could not use the measured distance d directly as the time t taken is not linearly related to d and so I could not use average values of d for multiple observations and expect it to be average values for time t
* I was able to calculate t directly while programming the Arduino microcontroller for the Audio/Visual/Haptic cues and the release mechanism. This helped me measure the time t without involving others (to catch a falling ruler)

## Procedure

### A) Prepare materials and equipment
* Arduino Mini microcontroller, LED, Speaker, Vibration motor, Transistors, Diodes, Resistors, Switches, Electromagnet, Display, Breadboard, Battery, Wires, Plastic ruler, Clips, etc.,
* Program the Arduino to deactivate the Electromagnet at the same time the LED is lit, sound is played in the Speaker and Vibration motor vibrates.
### B) Initial setup
* Explain to the subject about the experiment and what is expected out of them and get the consent form filled up and signed. Parental consent is obtained in case the subject is underage. 
* Setup the equipment on a table and get the subject seated comfortably.
* Ask the subject to choose their dominant hand to catch the falling ruler.
* Perform few trials by asking the subject to catch the ruler to see whether foot ruler or yard stick needs to be used for the experiment.
### C) Experiment Details
1. Select the stimuli type ‘Light’
2. Test subject to sit at the table with the dominant hand over the edge
3. Attach the ruler to the electromagnet and hold the ruler such that the 0 cm mark is touching the index finger of the test subject. Test subject is reminded not to see/face the ruler but focus on the LED. 
4. After a random delay, the release mechanism will light up the LED and drop the ruler at the same time and the test subject catches the ruler between thumb and index finger.
5. Note down the cm mark and repeat the experiment 4 more times.
6. Select the stimuli type ‘Sound’ and repeat the steps #2 to #5. The test subject is asked to close their eyes and focus on the audio cue from the Speaker.
7. Select the stimuli type ‘Touch’ and repeat the steps #2 to #5. The test subject is asked to close their eyes and hold the vibration motor in their non dominant hand and focus on the vibration cue.
8. Tabulate the results and compute the time taken in milliseconds and record the average of the 5 measurements.
9. The stimuli type with the lowest reaction time is the fastest.

## Experimental Setup
![Figure-6](/assets/img/hrt/fig-6.png){: .mx-auto.d-block :}
![Figure-7](/assets/img/hrt/fig-7.png){: .mx-auto.d-block :}

