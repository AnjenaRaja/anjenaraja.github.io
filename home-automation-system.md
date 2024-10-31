---
layout: page
title: Home Automation System
subtitle: Genius Hour Project
---

## Overview
This project involves a Home Automation System using the ESP8266 Wi-Fi module. It allows the control of lights, the garage door, and other home appliances over a web interface that can be accessed via a Wi-Fi Access Point created by the system itself. The project also uses a DHT11 sensor to monitor temperature and humidity, sending the data to a server for display.

## Key Features:
* Wi-Fi Access Point Creation: The system does not connect to a home router for safety reasons but instead creates its own Access Point to isolate the home network from potential security risks.
* Web-based Control Interface: Users can control lights and the garage door through a simple web interface. The interface includes buttons to turn individual lights on or off and control the garage door.
* Automatic Porch Light Control: Based on the light intensity outside, the porch light is turned on automatically at night or turned off during the day.
DHT11 Sensor for Temperature and Humidity Monitoring: The system uses a DHT11 sensor to monitor temperature and humidity, and displays the current readings on the web page.
* Simple User Interface: The project incorporates the use of HTML and CSS to display an intuitive user interface, complete with icons representing different rooms, devices, and states of the home.

## Hardware Components
* ESP8266 Wi-Fi Module: Central to the project, used for networking and web server capabilities.
* Servo Motor: Attached to the garage door for opening and closing it.
* DHT11 Sensor: Used to measure the humidity and temperature inside the house.
* Lights (LEDs): Used to simulate lights in various rooms.
* Relay (optional): Can be used to control high-power devices such as actual home lights and garage doors.

## Software Components
### Libraries Used:
* ESP8266WiFi.h: To enable Wi-Fi functionality and create an Access Point.
* ESP8266WebServer.h: To manage HTTP requests and serve web pages.
* Servo.h: To control the servo motor for the garage door.
* DHT.h: To interact with the DHT11 temperature and humidity sensor.
* ESP8266HTTPClient.h: To send HTTP requests with the sensor data.

## Detailed Explanation of Code
### Wi-Fi Access Point Setup
The ESP8266 creates its own Access Point with the following credentials:

```cpp
const char *ssid = "Home-Automation-225";
const char *password = "Anjena225";
```
* SSID is the name of the network.
* Password ensures only authorized devices can connect to it.

A custom IP configuration is used for the server, ensuring the system can be accessed via a known IP address:

```cpp
IPAddress ip(1, 2, 3, 4);
IPAddress gateway(1, 2, 3, 1);
IPAddress subnet(255, 255, 255, 0);
```
### Web Server and Control Interface
The system uses two web servers:

1. WiFiServer webServer(80): Handles the main control page.
2. ESP8266WebServer webServerSensors(8080): Handles temperature and humidity readings sent from the DHT11 sensor.
HTML Interface:

The web page includes icons and buttons for controlling different devices in the house. For example, the living room light can be toggled on/off using these commands:

```cpp
if (header.indexOf("GET /living/light/on") >= 0) {
    livingLightState = "on";
    digitalWrite(livingLight, HIGH);
}
else if (header.indexOf("GET /living/light/off") >= 0) {
    livingLightState = "off";
    digitalWrite(livingLight, LOW);
}
```

Each room's light and the garage door have similar commands, and a table is used to display the current state along with control buttons.

## Servo Motor Control (Garage Door)
The garage door is controlled using a servo motor. The position of the door is adjusted to 0 degrees for closed and 90 degrees for open:

```cpp
garageDoorMotor.write(0); // close
garageDoorMotor.write(90); // open
```

## Light Dependent Resistor (LDR)
The LDR measures light intensity, controlling the porch light:

```cpp
ldrValue = analogRead(A0); // read the LDR value
if (ldrValue < 50) {
    // If it's dark, turn on the porch light
    digitalWrite(porchLight, HIGH);
}
```

## Temperature and Humidity Monitoring
The system uses a DHT11 sensor to monitor the temperature and humidity. It reads values from the sensor and sends them to the web server via an HTTP request:

```cpp
float hTemp = dht.readHumidity();
float tTemp = dht.readTemperature(true);
```

This data is then displayed on the web page alongside control buttons.

## Automation Logic for Air Conditioning (AC)
The project controls an air conditioner (simulated with a built-in LED), turning it on if the temperature exceeds 80°F:

```cpp
if(tempf > 80) {
    digitalWrite(ac, LOW); // AC ON
}
```

## Overview of HTML UI
The HTML UI serves as the front-end for controlling and monitoring the home automation system. It is displayed in a web browser when a user connects to the ESP8266 access point. The page refreshes every 5 seconds to show the latest status of the devices.

### Key Components of the HTML UI
#### HTML Structure:
* The UI starts with the `<!DOCTYPE html>` declaration, which defines it as an HTML5 document.
* The `<html>`, `<head>`, and `<body>` tags organize the structure of the page.

#### Viewport Meta Tag:

```html
<meta name="viewport" content="width=device-width, initial-scale=1">
```
* This tag ensures the page is responsive, adjusting to different screen sizes, which is essential for usability on mobile devices.
    
#### Auto-refresh:

```html
<meta http-equiv="refresh" content="5; url=http://1.2.3.4">
```

* The page automatically refreshes every 5 seconds, allowing real-time updates without needing a manual refresh.

#### CSS Styling:

```css
<style>
html { font-family: Helvetica; display: inline-block; margin: 0px auto; text-align: center; }
.button { background-color: #195B6A; border: none; color: white; padding: 4px 10px; width: 80px; text-decoration: none; font-size: 20px; margin: 2px; cursor: pointer; }
.button2 {background-color: #77878A;}
.bigbutton { background-color: #195B6A; border: none; color: white; padding: 4px 10px; width: 300px; text-decoration: none; font-size: 20px; margin: 2px; cursor: pointer; }
.bigbutton2 {background-color: #77878A;}
</style>
``` 

* The CSS styles define the appearance of the page, including font styles, button sizes, and colors, enhancing the user experience.

#### Page Title and Header:

```cpp
client.println("<body>");
client.println(house);
client.println("<font size=\"6\">Home Automation</font>");
client.println("<br/>");
client.println("<font size=\"4\"><strong>by Anjena Raja, Grade 5</strong></font>");
client.println("<br/>");
client.println("<font size=\"4\"><strong>Room 225, Mrs. Carr</strong></font>");
```

* This section introduces the application with a title and some identification of the creator, making the UI more personalized and informative.

#### Device Status and Controls:
* The UI displays the status of various devices (lights and garage door) in a table format. Each device's current state and control buttons (ON/OFF or OPEN/CLOSE) are included:

```cpp
client.println("<table align=\"center\" cellpadding=\"3\">");
```

* The table structure organizes the information clearly, making it easy for users to view and interact with the controls.
* Dynamic Status Updates: Each device's state is dynamically updated based on the conditions defined in the code. For example:

```cpp
if (livingLightState=="off") 
{
    client.println("<td>" + lightOff + "</td>");
    client.println("<td><a href=\"/living/light/on\"><button class=\"button\">ON</button></a></td>");
} 
else 
{
    client.println("<td>"+ lightOn + "</td>");
    client.println("<td><a href=\"/living/light/off\"><button class=\"button button2\">OFF</button></a></td>");
}
```

* This conditional structure allows for different buttons to be displayed based on the current state of each light.

#### Light Control Buttons:
* There are also buttons to control all lights at once:

```cpp
client.println("<td colspan=\"5\"><a href=\"/all/light/on\"><button class=\"bigbutton\">ALL LIGHTS ON</button></a></td>");
client.println("<td colspan=\"5\"><a href=\"/all/light/off\"><button class=\"bigbutton bigbutton2\">ALL LIGHTS OFF</button></a></td>");
```

* These buttons provide a convenient way to control multiple devices with a single click.

#### Environmental Sensors:
* The UI also displays readings from the temperature and humidity sensors:

```cpp
client.print("<p>Temp: " + temperature.substring(0,4) + "&#8457;&nbsp;&nbsp;&nbsp;&nbsp;Humidity: " + humidity.substring(0,2) + "%&nbsp;&nbsp;&nbsp;&nbsp;A/C: ");
```

* This information is crucial for monitoring the environment and managing the air conditioning system.

## Home Automation System Code

``` cpp
/*
 * Home Automation System
 * by
 * Anjena Raja, Grade 5, Room 225, Mrs. Carr
 * This project is for my Genius Hour submission 
 *
 */

#include <ESP8266WiFi.h>
#include <ESP8266WebServer.h>
#include <Servo.h>

// I am creating my own WiFi Access Point and so I do not not need to connect to my home router for safety
// - any security problem in my home automation project will not compromise our internet security

// Access Point's name or SSID (Service Set IDentifier)
const char *ssid = "Home-Automation-225";
// Access Point's password 
const char *password = "Anjena225";

// This is my web server and it has to listen for connections from browsers on port 80 for HTTP (Hyper Text Transfer Protocol)
WiFiServer webServer(80);
ESP8266WebServer webServerSensors(8080);

// html header
String header;

// variables to store the current state of lights and doors
String livingLightState = "off";
String anjenaLightState = "off";
String parentsLightState = "off";
String garageDoorState = "close";

// variables for GPIO pins
const int livingLight = 5;
const int anjenaLight = 4;
const int parentsLight = 14;
const int garageDoor = 3;
const int porchLight = 1;
const int ac = BUILTIN_LED;

// icons 
String house = "<font size=\"7\">&#127969;</font>";
String world = "<font size=\"5\">&#127758;</font>";

String sun = "<font size=\"5\">&#127773;</font>";
String moon = "<font size=\"5\">&#127770;</font>";
String cloud = "<font size=\"5\">&#127767;</font>";

String light = "<font size=\"5\">&#128161;</font>";
String lightOn = "<font size=\"4\">&#128309;</font>";
String lightOff = "<font size=\"4\">&#8855;</font>"; //&#128308;

String door = "<font size=\"5\">&#128682;</font>";
String doorOpen = "<font size=\"5\">&#128275;</font>";
String doorLocked = "<font size=\"5\">&#128274;</font>";

String living = "<font size=\"6\">&#128106;</font>";
String parents = "<font size=\"6\">&#128107;</font>";
String anjena = "<font size=\"6\">&#128008;</font>";
String garage = "<font size=\"6\">&#128663;</font>";

String thermometer = "<font size=\"5\">&#127777;</font>";
String hygrometer = "<font size=\"5\">&#127778;</font>";
String acOn = "<font size=\"5\">&#128039;</font>";
String acOff = "<font size=\"5\">&#128037;</font>";

// this is used to measure the light intensity
// the value is used for displaying icons and to turn on the porch light
int ldrValue;

// this is used to open and close the garage door
Servo garageDoorMotor;

void setup() 
{
  // initialize pins of micro controller
  pinMode(livingLight, OUTPUT);
  pinMode(anjenaLight, OUTPUT);
  pinMode(parentsLight, OUTPUT);
  pinMode(porchLight, OUTPUT);
  pinMode(ac, OUTPUT);
      
  // turn off lights
  digitalWrite(livingLight, LOW);
  digitalWrite(anjenaLight, LOW);
  digitalWrite(parentsLight, LOW);
  digitalWrite(porchLight, LOW);

  // keep door closed
  garageDoorMotor.attach(garageDoor);
  delay(500);
  garageDoorMotor.write(0);
  delay(500);
  garageDoorMotor.detach();

  //keep AC off
  digitalWrite(ac, HIGH);
    
  // this is the IP (Internet Protocol) address of my web server
  IPAddress ip(1,2,3,4);

  // this is the address of my gateway
  // this is the end point and all communications stop at this address
  IPAddress gateway(1,2,3,1); 
  IPAddress subnet(255,255,255,0); 

  // configure the access point's ip and gateway
  WiFi.softAPConfig(ip, gateway, subnet);

  // we need to secure the access point so that unauthorized users cannot connect to it
	WiFi.softAP(ssid, password);
  
  // start the web server
	webServer.begin();
  
  webServerSensors.on("/feed", handle_feed);
  webServerSensors.begin();
}

// Handling the /feed page from my server
String temperature ="-";
String humidity ="-";
void handle_feed() 
{
  temperature = webServerSensors.arg("temp");
  humidity = webServerSensors.arg("hum");

  webServerSensors.send(200, "text/plain", "Home automation system: Message received.");
}

void loop() 
{
 // Listen for incoming clients
 webServerSensors.handleClient();
 
 WiFiClient client = webServer.available();

 if (client) 
 {                             
  // If a new client connects
    String currentLine = "";                // make a String to hold incoming data from the client
    while (client.connected()) 
    {            
      // loop while the client's connected
      if (client.available()) 
      {             
        // if there's bytes to read from the client,
        char c = client.read();             // read a byte, then
        header += c;
        if (c == '\n') 
        {
          // if the byte is a newline character
          // if the current line is blank, you got two newline characters in a row.
          // that's the end of the client HTTP request, so send a response:
          if (currentLine.length() == 0) 
          {
            // HTTP headers always start with a response code (e.g. HTTP/1.1 200 OK)
            // and a content-type so the client knows what's coming, then a blank line:
            client.println("HTTP/1.1 200 OK");
            client.println("Content-type:text/html");
            client.println("Connection: close");
            client.println();
            
            // turns the GPIOs on and off
            if (header.indexOf("GET /living/light/on") >= 0) 
            {
              livingLightState = "on";
              digitalWrite(livingLight, HIGH);
            } 
            else if (header.indexOf("GET /living/light/off") >= 0) 
            {
              livingLightState = "off";
              digitalWrite(livingLight, LOW);
            } 
            else if (header.indexOf("GET /anjena/light/on") >= 0) 
            {
              anjenaLightState = "on";
              digitalWrite(anjenaLight, HIGH);
            } 
            else if (header.indexOf("GET /anjena/light/off") >= 0) 
            {
              anjenaLightState = "off";
              digitalWrite(anjenaLight, LOW);
            }
            else if (header.indexOf("GET /parents/light/on") >= 0) 
            {
              parentsLightState = "on";
              digitalWrite(parentsLight, HIGH);
            } 
            else if (header.indexOf("GET /parents/light/off") >= 0) 
            {
              parentsLightState = "off";
              digitalWrite(parentsLight, LOW);
            }
            if (header.indexOf("GET /all/light/on") >= 0) 
            {
              livingLightState = "on";
              anjenaLightState = "on";
              parentsLightState = "on";
              digitalWrite(livingLight, HIGH);
              digitalWrite(anjenaLight, HIGH);
              digitalWrite(parentsLight, HIGH);
            } 
            else if (header.indexOf("GET /all/light/off") >= 0) 
            {
              livingLightState = "off";
              anjenaLightState = "off";              
              parentsLightState = "off";
              digitalWrite(livingLight, LOW);
              digitalWrite(anjenaLight, LOW);              
              digitalWrite(parentsLight, LOW);
            }
            else if (header.indexOf("GET /garage/door/close") >= 0) 
            {
              garageDoorState = "close";
              
              garageDoorMotor.attach(garageDoor);
              delay(500);
              garageDoorMotor.write(0);
              delay(500);
              garageDoorMotor.detach();
            } 
            else if (header.indexOf("GET /garage/door/open") >= 0) 
            {
              garageDoorState = "open";

              garageDoorMotor.attach(garageDoor);
              delay(500);
              garageDoorMotor.write(90);
              delay(500);
              garageDoorMotor.detach();
            }

            // Display the HTML web page
            client.println("<!DOCTYPE html><html>");
            client.println("<head><meta name=\"viewport\" content=\"width=device-width, initial-scale=1\">");
            client.println("<meta http-equiv=\"refresh\" content=\"5; url=http://1.2.3.4\">");
            //client.println("<link rel=\"icon\" href=\"data:,\">");

            // CSS to style the on/off buttons 
            client.println("<style>html { font-family: Helvetica; display: inline-block; margin: 0px auto; text-align: center;}");
            client.println(".button { background-color: #195B6A; border: none; color: white; padding: 4px 10px; width: 80px; ");
            client.println("text-decoration: none; font-size: 20px; margin: 2px; cursor: pointer;}");
            client.println(".button2 {background-color: #77878A;}");
            client.println(".bigbutton { background-color: #195B6A; border: none; color: white; padding: 4px 10px; width: 300px; ");
            client.println("text-decoration: none; font-size: 20px; margin: 2px; cursor: pointer;}");
            client.println(".bigbutton2 {background-color: #77878A;}</style></head>");
            
            // Web Page Heading
            client.println("<body>");
            client.println(house);
            client.println("<font size=\"6\">Home Automation</font>");
            client.println("<br/>");
                        
            client.println("<font size=\"4\"><strong>by Anjena Raja, Grade 5</strong></font>");
            client.println("<br/>");
            client.println("<font size=\"4\"><strong>Room 225, Mrs. Carr</strong></font>");

            // Display the outside lighting condition
            ldrValue = analogRead(A0); // read analog input pin 0
            if (ldrValue > 150)
            {
              // sunny
              client.println("<p>Sun Light: " + sun + "&nbsp;&nbsp;&nbsp;&nbsp;Porch Light: " + lightOff + "</p>");
              digitalWrite(porchLight, LOW);
            }
            else if (ldrValue > 50)
            {
              // cloudy
              client.println("<p>Sun Light: " + cloud + "&nbsp;&nbsp;&nbsp;&nbsp;Porch Light: " + lightOff + "</p>");
              digitalWrite(porchLight, LOW);
            }
            else 
            {
              // night
              client.println("<p>Sun Light: " + moon + "&nbsp;&nbsp;&nbsp;&nbsp;Porch Light: " + lightOn + "</p>");
              digitalWrite(porchLight, HIGH);
            }
            
            client.println("<table align=\"center\" cellpadding=\"3\">");
                        
            // Display current state, and ON/OFF buttons for livingLight  
            client.println("<tr>");
            client.println("<td>" + light + "</td>");
            client.println("<td>Living</td>");
            client.println("<td>" + living + "</td>");
            // If the livingLightState is off, it displays the ON button       
            if (livingLightState=="off") 
            {
              client.println("<td>" + lightOff + "</td>");
              client.println("<td><a href=\"/living/light/on\"><button class=\"button\">ON</button></a></td>");
            } 
            else 
            {
              client.println("<td>"+ lightOn + "</td>");
              client.println("<td><a href=\"/living/light/off\"><button class=\"button button2\">OFF</button></a></td>");
            } 
            client.println("</tr>");
               
            // Display current state, and ON/OFF buttons for anjenaLight  
            client.println("<tr>");
            client.println("<td>" + light + "</td>");
            client.println("<td>Anjena</td>");
            client.println("<td>" + anjena + "</td>");
            // If the anjenaLightState is off, it displays the ON button       
            if (anjenaLightState=="off") 
            {
              client.println("<td>" + lightOff + "</td>");
              client.println("<td><a href=\"/anjena/light/on\"><button class=\"button\">ON</button></a></td>");
            } 
            else 
            {
              client.println("<td>"+ lightOn + "</td>");
              client.println("<td><a href=\"/anjena/light/off\"><button class=\"button button2\">OFF</button></a></td>");
            }
            client.println("</tr>");

            // Display current state, and ON/OFF buttons for parentsLight  
            client.println("<tr>");
            client.println("<td>" + light + "</td>");
            client.println("<td>Parents</td>");
            client.println("<td>" + parents + "</td>");
            // If the parentsLightState is off, it displays the ON button       
            if (parentsLightState=="off") 
            {
              client.println("<td>" + lightOff + "</td>");
              client.println("<td><a href=\"/parents/light/on\"><button class=\"button\">ON</button></a></td>");
            } 
            else 
            {
              client.println("<td>"+ lightOn + "</td>");
              client.println("<td><a href=\"/parents/light/off\"><button class=\"button button2\">OFF</button></a></td>");
            }
            client.println("</tr>");

            // Display current state, and CLOSE/OPEN buttons for garageDoor  
            client.println("<tr>");
            client.println("<td>" + door + "</td>");
            client.println("<td>Garage</td>");
            client.println("<td>" + garage + "</td>");
            // If the garageDoorState is off, it displays the CLOSE button       
            if (garageDoorState=="open") 
            {
              client.println("<td>" + doorOpen + "</td>");
              client.println("<td><a href=\"/garage/door/close\"><button class=\"button button2\">CLOSE</button></a></td>");
            } 
            else 
            {
              client.println("<td>"+ doorLocked + "</td>");
              client.println("<td><a href=\"/garage/door/open\"><button class=\"button\">OPEN</button></a></td>");
            }
            client.println("</tr>");

            // one easy button to turn on/off all lights inside the house
            client.println("<tr>");
            client.println("<td colspan=\"5\"><a href=\"/all/light/on\"><button class=\"bigbutton\">ALL LIGHTS ON</button></a></td>");
            client.println("</tr>");

            client.println("<tr>");
            client.println("<td colspan=\"5\"><a href=\"/all/light/off\"><button class=\"bigbutton bigbutton2\">ALL LIGHTS OFF</button></a></td>");
            client.println("</tr>");

            client.println("</table>");
            
            //Display the inside temperature
            client.print("<p>Temp: " + temperature.substring(0,4) + "&#8457;&nbsp;&nbsp;&nbsp;&nbsp;Humidity: " + humidity.substring(0,2) + "%&nbsp;&nbsp;&nbsp;&nbsp;A/C: ");

            float tempf = temperature.toFloat();
            if (isnan(tempf))
            {
              client.print(acOff);
            }
            else
            {
              if(tempf > 80)
              {
                digitalWrite(ac, LOW);
                client.print(acOn);
              }
              else
              {
                digitalWrite(ac, HIGH);
                client.print(acOff);
              }
            }
            client.println("</p>");
 
            client.println("</body></html>");
                        
            // The HTTP response ends with another blank line
            client.println();
            // Break out of the while loop
            break;
          } 
          else 
          { // if you got a newline, then clear currentLine
            currentLine = "";
          }
        } 
        else if (c != '\r') 
        {  // if you got anything else but a carriage return character,
          currentLine += c;      // add it to the end of the currentLine
        }
      }
    }
    // Clear the header variable
    header = "";
    // Close the connection
    client.stop();
  }
}
```

## Remote Temperature Sensor Code

``` cpp
// client
#include <DHT.h>
#include <ESP8266WiFi.h>
#include <ESP8266HTTPClient.h>

#define DHTTYPE DHT11

// DHT Sensor
const int DHTPin = 5;
// Initialize DHT sensor.
DHT dht(DHTPin, DHTTYPE);

// Access Point's name or SSID (Service Set IDentifier)
const char *ssid = "Home-Automation-225";
// Access Point's password 
const char *password = "Anjena225";

// Local ESP web-server address
String serverHost = "http://1.2.3.4:8080/feed";
String data;

float h;
float t;
// Static network configuration
IPAddress ip(1, 2, 3, 201);
IPAddress gateway(1, 2, 3, 1);
IPAddress subnet(255, 255, 255, 0);
WiFiClient client;

void setup() 
{
  dht.begin();  

  WiFi.mode(WIFI_STA);
  WiFi.config(ip, gateway, subnet); 
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) 
  {
    delay(500);
  }
  Serial.begin(115200);
}

void loop() 
{
  readDHTSensor();
  buildDataStream();
  sendHttpRequest();

  // wait for 5 seconds 
  delay(5000);
}

void readDHTSensor() 
{
  float hTemp = dht.readHumidity();
  float tTemp = dht.readTemperature(true);
  if(!isnan(tTemp))
  {
    t = tTemp;
  }
  if (!isnan(hTemp))   
  {
    h = hTemp;
  }
}

void buildDataStream() 
{
  data = "temp=";
  data += String(t);
  data += "&hum=";
  data += String(h);
}

void sendHttpRequest() 
{
  HTTPClient http;
  http.begin(serverHost);
  http.addHeader("Content-Type", "application/x-www-form-urlencoded");
  http.POST(data);
  http.end();
  Serial.println(data);
}
```

## Video - Working Demo

###### [Link to video of the demo](https://www.youtube.com/watch?v=fXEqKRG6nCM){:target="_blank"}

#### [![Link to video of the demo](https://img.youtube.com/vi/fXEqKRG6nCM/0.jpg)](https://www.youtube.com/watch?v=fXEqKRG6nCM){:target="_blank"}
