---
layout: page
title: Laser Communication in Space
subtitle: Internship Project at RTX (Raytheon)
---

# Problem Statement
The amount of data people are using is growing exponentially with new tools for sifting through vast swaths of data emerging. Following this trend, scientists are requiring ever-increasing amounts of information from space for their research. However, space communication technologies are struggling to keep up. For example, the 2020 Mars Rover, Perseverance, can receive 10 to 30 bits of data per second with its specialized low-gain antenna. The rover can receive communications from a base station on Earth for roughly 8 minutes each Martian day. This amounts to 0.6 kB to 1.8 kB of data received per Martian day. For comparison, the image below is roughly 60 kB. This means the rover, which cost over $2 billion to build, will, in the worst case, take 100 days to receive a simple image. With scientists and engineers requiring an increasing amount of data to be transferred, the current means of communication will prove ineffective.

![Figure-1](/assets/img/lcs/fig-1.png){: .mx-auto.d-block :}

NASA is researching a new means of space communication: lasers. Pulsating lasers can be used to increase the speed of data transmission in space 10 to 100 times.
# Existing Technology
Since the first radio communication to space in 1962, radio waves have been the de facto standard of communication. While smaller wavelengths are absorbed by large objects, radio waves scatter, contributing to their prevalence. They are able to pass through the atmosphere and many items. However, radio waves are the root of the current inadequacies of space communications. These can have wavelengths of 10 meters and information can be transmitted only in proportion to each cycle.

If a smaller wavelength were to be used, more data could fit in a single length. This denser packing of data allows for greater bandwidth, speeding up communication. Infrared lasers with a wavelength of 830 to 1550 nm provide a viable alternative to traditional radio communications. Since laser technology is in its nascent stages, there are many improvements that can help make this technology more effective and easier to implement.
# Suggestions
## Two-way Communication
Current laser models only implement laser communication going from the rover back to Earth. This is to send out detailed data and images in a small window of communication. Instructions for the rover are still transmitted using slow radio waves. Even though less information needs to go from a base station to the rover, two-way communication can still be extremely beneficial.

Two-way communications can allow the base station to make real-time adjustments to the rover, instructing it to take specific actions. For example, based on soil sample data, particular locations for future samples and analysis can be specified. This gives the base station finer control of rover actions. Since instructions can be transferred more quickly with laser beams, more instructions can be sent to the rover, allowing it to know more pieces of data to collect and send back. This two-way system will help the base station adapt better to rover data, helping the base station make better future queries and commands.

This system can also be used to provide the infrastructure for an acknowledgment (ACK) protocol. Once a rover receives a data transmission from the base station, it will send a reply stating the transmission was successful. The same principle is applied to data transmissions going from the rover to the base station. This way, if the transmitter does not receive a message acknowledging the data transfer, it knows it was unsuccessful and will try again. If the rover receives data that has been distorted or damaged during transmission, it can request a retransmission. Rapid two-way communication is a necessary part of this back-and-forth ACK system. The ACK protocol allows for more reliable and resilient communications.
## Repeaters
The Perseverance rover can only communicate with a base station for only an 8-minute window, creating major limitations in data transmission. This is because a direct line of sight is required between the rover and base stations. When these are out of alignment such as if the rover is on the opposite side of Mars, communication is currently impossible. 

Repeaters are devices that receive data and can amplify and redirect the signal. There are telephone and radio repeaters, for example, which help relay a message over a long distance. A similar infrastructure can be implemented using a satellite network. Using these, transfers can happen from many more angles, increasing the window of communication. Repeaters could receive a message from a rover or another satellite and redirect the signal, giving it a greater reach. This is depicted in the image below. Because there is no line of sight between the satellite on the right and the desired base station, the repeater satellite on the left retransmits the data to the recipient.

![Figure-2](/assets/img/lcs/fig-2.png){: .mx-auto.d-block :}

Repeaters can also be used to facilitate the transition to laser communications. These can use radio waves for communication with rovers and base stations, but laser communications amongst themselves. This allows for laser communications to be used over long distances while still being compatible with radio wave communication infrastructure, preserving most of the improvements in transmission speed.
## Error Correction
With data transmission, there is always the chance of interference or noise distorting the original message. Lasers are especially sensitive to this since they have a smaller wavelength. One such source of interference is atmospheric disruption. To have more robust communications, error correction mechanisms can be implemented. Parity bits add up the values in a group of bits and if the sum is even it will store a 0 and it’s odd, a 1. Once a message is received if the sum of the data bits does not correspond with the parity bit, an error must have occurred. However, the error cannot be corrected. Hamming codes can be used to add on a few bits of data which can be used to detect errors and locate them. This works by having a parity bit for certain groups of bits. By comparing which sums are accurate, the location of the error can be identified. If the damage is too severe, then retransmission can be requested. This curbs the number of retransmission requests, making more effective use of communication.
## Encryption and Cybersecurity
Any time data is sent, there is a chance of attack, so cybersecurity risks are a forefront concern. Radio waves spread out, increasing the attack surface. So, lasers are much safer since they utilize point-to-point communications. However, precautions, such as encrypting communications, should still be taken. The Advanced Encryption Standard (AES) is a highly secure encryption algorithm that is even used to protect sensitive government information. AES is also considered safe against brute-force attacks and is designed to be easily implemented and integrated into existing systems. To prevent attackers from accessing the keys for
communication, multi-factor authentication should be used. 

Items launched into outer space cannot afford to have any errors or vulnerabilities since there is little a base station can do to resolve these. An error can lead to the loss of costly equipment and time. To ensure such incidents do not happen, secure coding practices should be used. This includes extensive testing, but there are also other considerations. A NASA article stated that uncontrolled code complexity can have many adverse effects, stating it causes, “higher cost, less rigorous testing, poorer operability, reduced performance, inhibited usage, lower reliability, and occasionally lost opportunities or lost missions.” Improper memory handling is increasingly a source of vulnerabilities, especially in programming languages such as C. The Homeland Security Systems Engineering and Development Institute put out a list of 2023’s top 25 software weaknesses. Some of the highest-ranked vulnerabilities include out-of-bounds write and read, use after free, and NULL pointer dereference. These are all basic memory-related errors, yet these errors manage to slip through. Precautions should be taken to avoid such issues such as avoiding heap usage and dereferencing pointers one at a time. Checks should be made to ensure code does not have unexpected behavior in unlikely scenarios or that
errors are handled appropriately. These are crucial since in space, if an error occurs, there is no direct way of handling it.
# Working Model
## How it works
Data is stored as bits made of 1s and 0s. These will correspond to a laser being turned on or off respectively. For laser transmission to occur, data must be translated from bits to on or off signals. This is done by a Raspberry Pi Pico microcontroller which varies the amount of current going to the transmitter. These signals are sent out using an infrared LED (light emitting diode). The signal is then picked up by a phototransistor. This device collects incoming photons and
converts them to varying amounts of current. The amount of current the phototransistor sends out is then sensed by the recipient’s microcontroller.

![Figure-3](/assets/img/lcs/fig-3.png){: .mx-auto.d-block :}

There are two boards with one representing the base station, JPL, and the other representing the rover. These were coded using the Arduino IDE due to the vast number of built-in libraries, helping quickly extend functionality. The Raspberry Pi Picos have a clock speed of roughly 125 MHz with some versions having higher defaults, compared to an Arduino’s 16 MHz clock speed. This means the Pi Pico can handle roughly 10 times the number of communications compared to an Arduino. Arduino IDE can be configured to handle many different boards including the Pi Pico, allowing the two to work together. 

###### [Link to video of the demo](https://drive.google.com/file/d/1SWhcR6_NXC8evElUV2Yza1MXnxlzi6qL/view?usp=sharing){:target="_blank"}
###### [Link to presentation slides](https://docs.google.com/presentation/d/1zszqOG7fdWVzj55BVmTPCa5D9BhMQLyoGpIfbuguXEk/edit?usp=sharing){:target="_blank"}

![Figure-4](/assets/img/lcs/fig-4.png){: .mx-auto.d-block :}

Commands are typed into the Arduino IDE serial monitor for JPL. 

![Figure-5](/assets/img/lcs/fig-5.png){: .mx-auto.d-block :}

The monitor corresponding to the rover outputs a “received” message once it gets a signal. It then provides a “sent” message after it has finished transmitting data via the infrared LED. These are used for debugging purposes to ensure transmission works smoothly. 

![Figure-6](/assets/img/lcs/fig-6.png){: .mx-auto.d-block :}

The JPL monitor has received the requested data and has displayed it in its serial monitor. A timestamp is included in the rover and JPL messages to confirm the order of events. 

## Temperature

| JPL Input Command | CORE_TEMP |
| :------ |:--- |
| Rover Debug Messages | CORE_TEMP - received.<br>CORE temperature - sent. |
| Rover Action | - |
| JPL Displayed Message Example | CORE temperature: 301.7 K |

As shown above, the rover can send over its temperature using the CORE_TEMP command, returning a value in Kelvin. This is measured using the rover’s onboard temperature sensor. This illustrates the rover can return collected sensor data. 

## Beacon

| JPL Input Command | BEACON_ON |
| :------ |:--- |
| Rover Debug Messages | BEACON_ON - received.<br>BEACON turned ON - sent. |
| Rover Action | Flashing LED is turned on. |
| JPL Displayed Message Example | BEACON turned ON. |
 
| JPL Input Command | BEACON_OFF |
| :------ |:--- |
| Rover Debug Messages | BEACON_OFF - received.<br>BEACON turned OFF - sent. |
| Rover Action | Flashing LED is turned off. |
| JPL Displayed Message Example | BEACON turned OFF. |

A flashing LED, representing the rover’s beacon can be turned on and off using the BEACON_ON and BEACON_OFF commands. The rover returns a message confirming it has taken the action. This allows the base station to remotely control the rover’s peripherals. 

## Image transmission

| JPL Input Command | SEND_IMAGE |
| :------ |:--- |
| Rover Debug Messages | SEND_IMAGE - received.<br>Base64 encoded image - sent. |
| Rover Action | - |
| JPL Displayed Message Example | Base64 encoded image: iVBO…CYII= |

### Sample base 64 encoded image
```
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
```
### Sample base 64 decoded image

![rover.png](/assets/img/lcs/rover.png)

Image transmission is one of the most important uses of high-speed laser communication. An image needs to be encoded into a text format so that it can be sent via infrared LEDs. This can be accomplished using the Base64 encoding. The desired image is encoded using Base64 and the characters are stored in the rover’s code. When JPL sends its request for the image, the rover simply returns this already encoded image. The data JPL receives can then be restored to its original format using a Base64 decoder. A link is included below. 

###### [Link to image conversion site](https://codebeautify.org/base64-to-image-converter){:target="_blank"}

## Camera movement

| JPL Input Command | CAMERA_ANGLE_90 or 0 or 180 |
| :------ |:--- |
| Rover Debug Messages | CAMERA_ANGLE_90 - received.<br>Camera rotation - sent. |
| Rover Action | Servo motor is rotated. |
| JPL Displayed Message Example | Camera rotated. |

A servo motor is rotated, representing a rover’s camera being adjusted to take a photo. This shows that JPL can control the mechanical aspects of the rover. The command CAMERA_ANGLE_0, CAMERA_ANGLE_90, or CAMERA_ANGLE_180 is specified and the servo motor moves to the corresponding position. So, if the same command is specified twice, the motor will not move. 

## JPL Breadboard Model

![Figure-7](/assets/img/lcs/fig-7.png){: .mx-auto.d-block :}

## Rover Breadboard Model

![Figure-8](/assets/img/lcs/fig-8.png){: .mx-auto.d-block :}

## JPL Circuit Diagram

![Figure-9](/assets/img/lcs/fig-9.png){: .mx-auto.d-block :}

## Rover Circuit Diagram

![Figure-10](/assets/img/lcs/fig-10.png){: .mx-auto.d-block :}

## JPL code

``` cpp
#include <SoftwareSerial.h>

// define the GPIO for serial communication with JPL
#define RX 18              // this is the phototransistor
#define TX 28              // this is the IR LED
#define ROVER_BAUD 9600    // communication speed
#define CONSOLE_BAUD 9600  // communication speed

// setup serial communcation with JPL
SoftwareSerial JPL = SoftwareSerial(RX, TX);

void setup() {
  // open serial communication with Rover
  JPL.begin(ROVER_BAUD);
  // open serial communication with console
  Serial.begin(CONSOLE_BAUD);

  // wait until the communication channels are ready
  while (!Serial.available()) {
    delay(100);
  }
  while (!JPL.available()) {
    delay(100);
  }
  Serial.println("Ready for your command ...");
}

#define TRANSMIT 0
#define RECEIVE 1

byte communicationMode = TRANSMIT;

void loop() {
  if (communicationMode == TRANSMIT) {
    //read operator's command from console
    String command = Serial.readStringUntil('\n');
    // send operator's command to Rover
    JPL.println(command);
    // now switch to receive mode to see how Rover responds
    communicationMode = RECEIVE;
  }

  if (communicationMode == RECEIVE) {
    int idata = JPL.read();
    if (idata != 0) {
      // listen what Rover has to say
      String result = JPL.readStringUntil('\n');
      // print the message from the Rover to the console
      if (result.length() > 3) {
        Serial.println(result);
      }
      // Rover has finished communicating
      // now switch to transmit mode  for operator's commands
      communicationMode = TRANSMIT;
    }
  }
}
```

## Rover code

``` cpp
#include <SoftwareSerial.h>
#include <Servo.h>

char image[] = "Base64 encoded image: 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";

// define the GPIO for serial communication with JPL
#define RX 18              // this is the phototransistor
#define TX 28              // this is the IR LED
#define JPL_BAUD 9600      // communication speed
#define CONSOLE_BAUD 9600  // communication speed

// define the GPIO for beacon
#define BEACON 14

// enable debug messages to console during development
#define DEBUG
#ifdef DEBUG
#define PRINT(x) Serial.println(x)
#else
#define PRINT(x)
#endif

// setup serial communcation with JPL
SoftwareSerial Rover = SoftwareSerial(RX, TX);

// servo to control the direction of the camera
#define CAMERA 10
Servo cameraMotor;

void setup() {
  // open serial communication with JPL
  Rover.begin(JPL_BAUD);

// conditionally open serial communication with console
#ifdef DEBUG
  Serial.begin(CONSOLE_BAUD);
#endif
  // initialize beacon
  pinMode(BEACON, OUTPUT);
  digitalWrite(BEACON, LOW);

  // initialize camera servo
  cameraMotor.attach(CAMERA);
  cameraMotor.write(0);

  // check to see whether serial communcation started
  // if so Rover can start listening to the commands from JPL
  while (!Rover.available()) {
    delay(100);
  }

  PRINT("Rover initialization complete.");
}

void loop() {
  // start reading the signal from JPL
  // if the signal from JPL has a non NULL value start reading the command
  String command = Rover.readStringUntil('\n');
  // process the command from JPL
  processCommand(command);
}

void processCommand(String command) {
  if (command.length() > 3) {
    // print the command received as is from JPL on the console
    PRINT(command + " - received.");

    // check for known commands
    if (command.startsWith("BEACON_ON")) {
      // process beacon request
      handleBeaconRequest(1);
    } else if (command.startsWith("BEACON_OFF")) {
      // process beacon request
      handleBeaconRequest(0);
    } else if (command.startsWith("CORE_TEMP")) {
      // process CORE request
      handleCoreRequest();
    } else if (command.startsWith("SEND_IMAGE")) {
      // process image request
      handleImageRequest();
    } else if (command.startsWith("CAMERA_ANGLE_0")) {
      // process image request
      handleCameraRequest(0);
    } else if (command.startsWith("CAMERA_ANGLE_90")) {
      // process image request
      handleCameraRequest(90);
    } else if (command.startsWith("CAMERA_ANGLE_180")) {
      // process image request
      handleCameraRequest(180);
    } else {
      // this is not a command that Rover could understand
      // let JPL know that Rover did not understand the command
      Rover.println(command + " - Unknown command.");
      PRINT(command + " - Unknown command - sent.");
    }
  }
}

void handleBeaconRequest(byte action) {
  // enable or disable beacon
  if (action == 1) {
    // turn on beacon
    digitalWrite(BEACON, HIGH);
    // let JPL know that the beacon has been turned on
    Rover.println("BEACON turned ON.");
    PRINT("BEACON turned ON - sent.");
  } else {
    // turn off beacon
    digitalWrite(BEACON, LOW);
    // let JPL know that the beacon has been turned off
    Rover.println("BEACON turned OFF.");
    PRINT("BEACON turned OFF - sent.");
  }
}

void handleCoreRequest() {
  // let JPL know about the Core temperature status
  Rover.printf("CORE temperature: %2.1f K \n", (analogReadTemp() + 273.15));
  PRINT("CORE temperature - sent.");
}

void handleImageRequest() {
  // send JPL a base 64 encoded text of an image
  Rover.println(image);
  PRINT("Base64 encoded image - sent.");
}

void handleCameraRequest(int angle) {
  // rotate camera
  cameraMotor.write(angle);
  // send JPL the confirmation on rotation
  Rover.println("Camera rotated.");
  PRINT("Camera rotation - sent.");
}
```

## Future improvements
My model features the capabilities of laser transmission technology and can perform several actions. It can still be extended to implement more of my suggestions and maximize the capabilities of this technology. My model uses infrared LEDs instead of lasers, primarily for safety concerns during development. However, adding these lasers will allow for true point-to-point communications. This will improve the security of transmissions and preserve light intensity over long distances, increasing signal range. Hamming codes and an ACK protocol can enable more resilient communications, improving transmission fidelity. These improvements will better the communication capabilities of my model. 

# Conclusion
With the amount of human involvement in space increasing, a reliable and rapid means of communication is necessary. Pulsating infrared lasers are a promising and viable solution to this since they can transfer data at 10 to 100 times the current speed. However, this emerging technology has room for improvement. A two-way communication system with an ACK protocol, repeaters, error correction, encryption, and secure coding practices are innovations that will allow lasers to go from theory to practice. My working model illustrates the viability of pulsating optical transmissions. It uses exceedingly cost-effective and easily available materials, emphasizing the facility with which this technology can be implemented and scaled. 

This pulsating infrared technology is well suited for a myriad of other use cases. It can be used when existing radio communication infrastructure is damaged. Laser communications will maintain their intensity for longer distances, allowing for longer-range communications if repeaters are damaged. Signal jammers are used to block radio communications, but are not capable of blocking infrared signals. This lets lasers be used on Earth when existing communication infrastructure is not reliable. 

# References

+ Advanced Encryption Standard (AES). 1 Nov. 2001, doi.org/10.6028/nist.fips.197.
+ Baird, Daniel. “Laser Communications: Empowering More Data Than Ever Before.” NASA, May 2021, nasa.gov/feature/goddard/2021/laser-communications-empowering-more-data-than-ever-before.
+ Buinhas, Luisa, et al. “Key Challenges in Establishing Laser Space Communication Standards and Recommendations of the SGC Space...” ResearchGate, Oct. 2018, researchgate.net/publication/328478337_Key_Challenges_in_Establishing_Laser_Space_Communication_Standards_and_Recommendations_of_the_SGC_Space_Technologies_Working_Group.
+ “Communications System Achieves Fastest Laser Link From Space Yet.” MIT News. Massachusetts Institute of Technology, 30 Nov. 2022, news.mit.edu/2022/communications-system-achieves-fastest-laser-link-space-yet-1130.
+ CWE -  2023 CWE Top 25 Most Dangerous Software Weaknesses. cwe.mitre.org/top25/archive/2023/2023_top25_list.html.
+ Diolante. “Wireless Laser Communication With Arduino: A Prototype.” Instructables, May 2022, instructables.com/Sending-Messages-Using-Lasers-With-Arduino.
+ HobbyTransform. “Arduino Encoded and Modulated Laser and Infrared Serial Communication.” Instructables, Sept. 2017, instructables.com/Arduino-Encoded-and-Modulated-Laser-and-Infrared-S.
+ Low Level Learning. “How NASA Writes Space-proof Code.” YouTube, 3 June 2023, youtube.com/watch?v=GWYhtksrmhE.
+ Mars.Nasa.Gov. Communications - NASA. mars.nasa.gov/mars2020/spacecraft/rover/communications.
+ Martini, Rainer, et al. “Optical Free-space Communications at Middle-infrared Wavelengths.” Proceedings of SPIE, SPIE, 2004, doi.org/10.1117/12.516517.
+ Monaghan, Heather. “Delay/Disruption Tolerant Networking Overview.” NASA, Dec. 2021, nasa.gov/directorates/heo/scan/engineering/technology/disruption_tolerant_networking.
+ NASA. “Communications With Earth.” NASA Mars Exploration, mars.nasa.gov/msl/mission/communications.
+ NASA. “Optical Communications Challenges.” NASA, Aug. 2021, nasa.gov/directorates/heo/scan/opticalcommunications/challenges.
+ Pahor, David. “Reliable Laser Communications Between Space and Ground. Cosylab.” Cosylab, 13 Dec. 2022, cosylab.com/2022/03/25/reliable-laser-communications-between-space-and-ground

# Further materials

###### [Link to video of the demo](https://drive.google.com/file/d/1SWhcR6_NXC8evElUV2Yza1MXnxlzi6qL/view?usp=sharing){:target="_blank"}
###### [Link to presentation slides](https://docs.google.com/presentation/d/1zszqOG7fdWVzj55BVmTPCa5D9BhMQLyoGpIfbuguXEk/edit?usp=sharing){:target="_blank"}
###### [Link to video of the presentation](https://drive.google.com/file/d/1wFwA7b_56GKfGtUBW1VQU9VrLL4A28Vd/view){:target="_blank"}
