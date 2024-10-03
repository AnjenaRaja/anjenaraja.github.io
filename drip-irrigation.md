---
layout: page
title: Watering Plants - Drip Irrigation
subtitle: An Experiment
---

# Does drip irrigation or periodic watering provide the best amount of water to plants?
# Hypothesis 
## Drip irrigation provides the best amount of water for unattended watering of potted plants.

# Theory
1. Plant growth factors
   * **Photosynthesis** - The process of converting energy from light into sugar, in the presence of chlorophyll using carbon dioxide and water
   * **Respiration** - The process of burning sugars using oxygen to get energy for growth, and life
   * **Transpiration** - The process of transporting nutrients using water from the soil through roots
2. Plants need Air (Carbon dioxide and Oxygen), Water, Sunlight, and Nutrients to survive and grow
3. Potted plans need to be watered regularly to survive

# Procedure
1. Prepare materials and equipment
   * Select two potted plants of the same kind and size
   * 2 empty 1-gallon milk cans, plastic tubes, small valve, toy pump
   * Equipment needed – moisture sensor, timer, and kitchen scale
   * Place the 2 potted plants side by side such that they would get the same amount of sunlight
2. Drip Irrigation Setup
   * Fill the milk cans with water, make a small hole in the bottle cap for the tube, and insert the plastic tube through the hole
   * Place the milk can in an elevated position relative to the pot and adjust the valve such that 20 drops of water flow in 10 minutes
   * Insert the moisture sensor into the soil
3. Periodic Watering Setup
   * Fill the milk cans with water, make a small hole in the bottle cap for the tube, and insert the plastic tube through the hole
   * Make sure that the pump reaches the bottom of the can
   * Place the milk can at the same level as the pot
   * Adjust the timer such that 150 ml of water is delivered by the pump every day (Turn on the pump for about 10 seconds per day)
4. Allow the 2 watering systems to run for 3 days and adjust the systems if needed until they work reliably
5. Measure the maximum, minimum, and average moisture levels of the 2 plants daily. Take multiple readings to arrive at the average. Record your observation in a table. In the test setup, the microcontroller took hundreds of readings every minute.
6. Measure the amount of water dispensed using the kitchen scale. Find the difference in the weight of milk cans between the present and the previous day to calculate the ml of water delivered to the plants. Record your observation in a table. 
7. Continue the experiment for 21 days.

# Experimental Setup

![Figure-1](/assets/img/dr/fig-1.png){: .mx-auto.d-block :}

![Figure-2](/assets/img/dr/fig-2.png){: .mx-auto.d-block :}

## Arduino Code
``` cpp
// This program requires a Nokia 5110 LCD module.
//
// It is assumed that the LCD module is connected to
// the following pins using a levelshifter to get the
// correct voltage to the module.
//      SCK  - Pin 8
//      MOSI - Pin 9
//      DC   - Pin 10
//      RST  - Pin 11
//      CS   - Pin 12
//
//#include <Time.h>
#include <LCD5110_Graph.h>

LCD5110 myGLCD(8,9,10,11,12);

extern uint8_t SmallFont[];

//set the current time here
int h = 19;
int m = 00;
int s = 00;
unsigned long start, now;

double m1_Average = 0;
int m1_Maximum = 0;
int m1_Minimum = 100;

double m2_Average = 0;
int m2_Maximum = 0;
int m2_Minimum = 100;

int sampleSize = 1;

int motor = 7; // for the pump
int led = 13; // one second blip to show that the thingy is on

boolean toggle = false;

void setup()
{
  myGLCD.InitLCD(70);
  myGLCD.setFont(SmallFont);
  myGLCD.invert(false);

  // initialize the digital pin as an output.
  pinMode(led, OUTPUT);     
  pinMode(motor, OUTPUT);

  pinMode(A0, INPUT);
  pinMode(A1, INPUT);
  
  digitalWrite(motor, LOW);
}

void incrementTime()
{
  // increment seconds
  s += 1;
  
  if(s == 60)
  { 
    s = 0; 
    m += 1; 
  }
  
  if(m == 60)
  {
    m = 0; 
    h += 1; 
  }
  
  if(h == 24)
  { 
    // we are interested in 12 hours clock
    h = 0; 
    resetValues();
  } 
  
  // water plants at 8 am 
  // this is for production
  if(h == 8 && m == 0 && s == 0)
  {
    waterPlant();
  }
  
  // this is for testing the system
  // every 15 seconds as we cannot sit and wait for hours
  // just comment this out when in production
  if(s == 0 || s == 30)
  {
    waterPlant();
  }
}

void resetValues()
{
  m1_Average = 0;
  m1_Maximum = 0;
  m1_Minimum = 100;
  
  m2_Average = 0;
  m2_Maximum = 0;
  m2_Minimum = 100;

  sampleSize = 1;
}

void waterPlant()
{
  printWatering();
  // start motor
  digitalWrite(motor, HIGH);
  
  // delay for 10 seconds
  for(int i = 0; i< 10; i++)
  {
    delay(1000);
    incrementTime();
    printTime();
  }

  // stop motor
  digitalWrite(motor, LOW);
  myGLCD.clrScr();
}

void loop()
{
  now = millis();
  if(now - start >= 1000)
  {
    // one second up as per hardware clock
    start = millis();
    incrementTime();

    // blink on board LED as a sign that the fish feeder is infact working
    toggle = !toggle;
    digitalWrite(led, toggle);   // we get the blink effect as the toggle switches for each second

  }

  printTime();
  printMoisture();
}

void printMoisture()
{
  myGLCD.invertText(true);
  myGLCD.print("P-1", 36, 8);
  myGLCD.print("P-2", 66, 8);
  myGLCD.invertText(false);

  myGLCD.print("Min:", LEFT, 16);
  myGLCD.print("Max:", LEFT, 24);
  myGLCD.print("Avg:", LEFT, 32);
  myGLCD.print("Now:", LEFT, 40);
  
  int value = analogRead(A0);
  int percent = map(value, 1023, 350, 0, 100);
  if (percent > 100)
  {
    percent = 100;
  }  

  myGLCD.printNumI(percent, 30, 40, 3);
  myGLCD.print("%", 48, 40);

  sampleSize++;

  if (percent > m1_Maximum)
  {
    m1_Maximum = percent;
  }

  if (percent < m1_Minimum)
  {
    m1_Minimum = percent;
  }

  m1_Average = (m1_Average * (sampleSize-1) +  percent) / sampleSize;

  myGLCD.printNumI(m1_Minimum, 30, 16, 3);
  myGLCD.print("%", 48, 16);

  myGLCD.printNumI(m1_Maximum, 30, 24, 3);
  myGLCD.print("%", 48, 24);

  myGLCD.printNumI(m1_Average, 30, 32, 3);
  myGLCD.print("%", 48, 32);

  value = analogRead(A1);
  percent = map(value, 1023, 400, 0, 100);
  if (percent > 100)
  {
    percent = 100;
  }  
  myGLCD.printNumI(percent, 60, 40, 3);
  myGLCD.print("%", 78, 40);

  if (percent > m2_Maximum)
  {
    m2_Maximum = percent;
  }

  if (percent < m2_Minimum)
  {
    m2_Minimum = percent;
  }

  m2_Average = (m2_Average * (sampleSize-1) +  percent) / sampleSize;

  myGLCD.printNumI(m2_Minimum, 60, 16, 3);
  myGLCD.print("%", 78, 16);

  myGLCD.printNumI(m2_Maximum, 60, 24, 3);
  myGLCD.print("%", 78, 24);

  myGLCD.printNumI(m2_Average, 60, 32, 3);
  myGLCD.print("%", 78, 32);
  
  myGLCD.update();  
}

void printTime()
{
  myGLCD.print("Time: ", 0, 0);
  myGLCD.printNumI(h, 36, 0, 2, '0');
  myGLCD.print(":", 48, 0);
  myGLCD.printNumI(m, 54, 0, 2, '0');
  myGLCD.print(".", 66, 0);
  myGLCD.printNumI(s, 72, 0, 2, '0');
  myGLCD.update();  
}

void printWatering()
{
    myGLCD.clrScr();
    myGLCD.print("Watering!", CENTER, 36);
    myGLCD.update();  
}
```

# Results

![Figure-3](/assets/img/dr/fig-3.png){: .mx-auto.d-block :}

# Comparison

![Figure-4](/assets/img/dr/fig-4.png){: .mx-auto.d-block :}

# Observations
![Figure-5](/assets/img/dr/fig-5.png){: .mx-auto.d-block :}
![Figure-6](/assets/img/dr/fig-6.png){: .mx-auto.d-block :}

# Conclusions
## Drip irrigation provides the best amount of water for unattended watering of potted plants.
* Drip irrigation: Simple setup, cost-effective, saves water, but it may not be suitable for all types of plants.
* Periodic watering system: Hard to set up, expensive, and wastes water, but it can be adjusted for all types of plants.
* A periodic drip irrigation system could achieve the merits of the periodic watering system and still save water.

# References
## Books
* What do roots, stems, leaves, and flowers do? – Ruth Owen
* Watering Systems for Lawn & Garden – R. Dodge Woodsen
* Ortho’s All About Sprinklers and Drip Systems  – Larry Hodgson
* Water-Conserving Gardens and Landscapes  – John M. O’Keefe
* Drip Irrigation Guide – Product information guide from DIG Corporation (digicorp.com) 
 
## Online Resources
* Plant Growth Factors: Photosynthesis, Respiration, and Transpiration - ext.colostate.edu/mg/gardennotes/141.html 
* Trickle Irrigation: Using and Conserving Water in the Home Garden - umaine.edu/publications/2160e
* How Does Water Affect Plant Growth? - gardeningknowhow.com/special/children/how-does-water-affect-plant-growth.htm 
* Arduino Soil Moisture Sensor along with a Nokia 5110 LCD display - educ8s.tv/arduino-soil-moisture-sensor 
* Siphon - en.wikipedia.org/wiki/Siphon
* How centrifugal home water pump works - youtube.com/watch?v=_BAnnTLpros 
* Arduino Lessons: DC Motors - learn.adafruit.com/adafruit-arduino-lesson-13-dc-motors?view=all 

