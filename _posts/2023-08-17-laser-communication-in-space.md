---
layout: post
title: Laser Communication in Space
subtitle: Internship Project at RTX (Raytheon)
thumbnail-img: /assets/img/pi-pico.svg
share-img: /assets/img/pi-pico.svg
tags: [microcontroller, pi-pico]
author: Anjena Raja
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

Image transmission is one of the most important uses of high-speed laser communication. An image needs to be encoded into a text format so that it can be sent via infrared LEDs. This can be accomplished using the Base64 encoding. The desired image is encoded using Base64 and the characters are stored in the rover’s code. When JPL sends its request for the image, the rover simply returns this already encoded image. The data JPL receives can then be restored to its original format using a Base64 decoder. A link is included below. 

###### [Link to image conversion site](https://codebeautify.org/base64-to-image-converter)

## Camera movement

| JPL Input Command | CAMERA_ANGLE_90 or 0 or 180 |
| :------ |:--- |
| Rover Debug Messages | CAMERA_ANGLE_90 - received.<br>Camera rotation - sent. |
| Rover Action | Servo motor is rotated. |
| JPL Displayed Message Example | Camera rotated. |

A servo motor is rotated, representing a rover’s camera being adjusted to take a photo. This shows that JPL can control the mechanical aspects of the rover. The command CAMERA_ANGLE_0, CAMERA_ANGLE_90, or CAMERA_ANGLE_180 is specified and the servo motor moves to the corresponding position. So, if the same command is specified twice, the motor will not move. 
