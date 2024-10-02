---
layout: page
title: Marquee Display Board
subtitle: Individual Creativity Project
---

![Figure-1](/assets/img/db/fig-1.png){: .mx-auto.d-block :}
![Figure-2](/assets/img/db/fig-2.png){: .mx-auto.d-block :}
![Figure-3](/assets/img/db/fig-3.png){: .mx-auto.d-block :}
![Figure-4](/assets/img/db/fig-4.png){: .mx-auto.d-block :}
![Figure-5](/assets/img/db/fig-5.png){: .mx-auto.d-block :}
![Figure-6](/assets/img/db/fig-6.png){: .mx-auto.d-block :}
![Figure-7](/assets/img/db/fig-7.png){: .mx-auto.d-block :}
![Figure-8](/assets/img/db/fig-8.png){: .mx-auto.d-block :}
![Figure-9](/assets/img/db/fig-9.png){: .mx-auto.d-block :}
![Figure-10](/assets/img/db/fig-10.png){: .mx-auto.d-block :}

## Display Code
``` cpp
// Connections for ESP8266 hardware SPI are:
// Vcc       3v3     LED matrices seem to work at 3.3V
// GND       GND     GND
// CLK        D5     CLK or HCLK
// DIN        D7     HSPID or HMOSI
// CS or LD   D8     HSPICS or HCS

#include <ESP8266WiFi.h>
#include <MD_MAX72xx.h>
#include <SPI.h>

#define DEBUG 0

#if DEBUG
  #define PRINT(s, v) { Serial.print(F(s)); Serial.print(v); }
  #define PRINTS(s)   { Serial.print(F(s)); }
#else
  #define PRINT(s, v)
  #define PRINTS(s)
#endif

// Define the number of devices we have in the chain and the hardware interface
#define HARDWARE_TYPE MD_MAX72XX::ICSTATION_HW
#define MAX_DEVICES 8

#define CLK_PIN   D5 // or SCK
#define DATA_PIN  D7 // or MOSI
#define CS_PIN    D8 // or SS

// SPI hardware interface 
// Serial Peripheral Interface
MD_MAX72XX mx = MD_MAX72XX(HARDWARE_TYPE, CS_PIN, MAX_DEVICES);

// Arbitrary pins
//MD_MAX72XX mx = MD_MAX72XX(HARDWARE_TYPE, DATA_PIN, CLK_PIN, CS_PIN, MAX_DEVICES);

// Router connection details
// Access Point's name or SSID (Service Set IDentifier)
const char *ssid = "Display-Controller";
// Access Point's password 
const char *password = "AnjenaRaja";

// Web Server object and parameters
// Port 80 is HTTP
// Hyper Text Transfer Protocol
WiFiServer server(80);

// Global message buffers shared by Wifi and Scrolling functions
const uint8_t MESG_SIZE = 255;
const uint8_t CHAR_SPACING = 1;
uint8_t SCROLL_DELAY = 50;

char curMessage[MESG_SIZE];
char newMessage[MESG_SIZE];
bool newMessageAvailable = false;

char WebResponse[] = "HTTP/1.1 200 OK\nContent-Type: text/html\n\n";

char WebPage[] =
"<!DOCTYPE html>" \
"<html>" \
"<head>" \
"<title>Display Control - Anjena Raja</title>" \

"<script>" \
"strLine = \"\";" \

"function SendText()" \
"{" \
"  nocache = \"/&nocache=\" + Math.random() * 1000000;" \
"  var request = new XMLHttpRequest();" \
"  strLine = \"&MSG=\" + document.getElementById(\"txt_form\").Message.value;" \
"  request.open(\"GET\", strLine + nocache, false);" \
"  request.send(null);" \
"}" \
"</script>" \
"</head>" \

"<body>" \
"<p><b>Message</b></p>" \

"<form id=\"txt_form\" name=\"frmText\">" \
"<label>Msg:<input type=\"text\" name=\"Message\" maxlength=\"255\"></label><br><br>" \
"</form>" \
"<br>" \
"<input type=\"submit\" value=\"Send\" onclick=\"SendText()\">" \
"</body>" \
"</html>";

char *err2Str(wl_status_t code)
{
  switch (code)
  {
    case WL_IDLE_STATUS:    return("IDLE");           break; // WiFi is in process of changing between statuses
    case WL_NO_SSID_AVAIL:  return("NO_SSID_AVAIL");  break; // case configured SSID cannot be reached
    case WL_CONNECTED:      return("CONNECTED");      break; // successful connection is established
    case WL_CONNECT_FAILED: return("CONNECT_FAILED"); break; // password is incorrect
    case WL_DISCONNECTED:   return("CONNECT_FAILED"); break; // module is not configured in station mode
    default: return("??");
  }
}

uint8_t htoi(char c)
{
  c = toupper(c);
  if ((c >= '0') && (c <= '9')) return(c - '0');
  if ((c >= 'A') && (c <= 'F')) return(c - 'A' + 0xa);
  return(0);
}

boolean getText(char *szMesg, char *psz, uint8_t len)
{
  boolean isValid = false;  // text received flag
  char *pStart, *pEnd;      // pointer to start and end of text

  // get pointer to the beginning of the text
  pStart = strstr(szMesg, "/&MSG=");

  if (pStart != NULL)
  {
    pStart += 6;  // skip to start of data
    pEnd = strstr(pStart, "/&");

    if (pEnd != NULL)
    {
      while (pStart != pEnd)
      {
        if ((*pStart == '%') && isdigit(*(pStart+1)))
        {
          // replace %xx hex code with the ASCII character
          char c = 0;
          pStart++;
          c += (htoi(*pStart++) << 4);
          c += htoi(*pStart++);
          *psz++ = c;
        }
        else
        {
          *psz++ = *pStart++;
        }
      }

      *psz = '\0'; // terminate the string
      isValid = true;
    }
  }

  String speed = "";
  pStart = strstr(szMesg, "/&SPD=");

  PRINT("\npStart=", pStart);
  if (pStart != NULL)
  {
    pStart += 6;  // skip to start of data
    pEnd = strstr(pStart, "/&");

    PRINT("\npEnd=", pEnd);

    if (pEnd != NULL)
    {
      while (pStart != pEnd)
      {
        if ((*pStart == '%') && isdigit(*(pStart+1)))
        {
          // replace %xx hex code with the ASCII character
          char c = 0;
          pStart++;
          c += (htoi(*pStart++) << 4);
          c += htoi(*pStart++);
          PRINT("\nC=", c);
        }
        else
        {
          speed += *pStart;
          PRINT("\npStart=", *pStart);
          *pStart++;
        }
      }
      PRINT("\npSpeed=", speed);
      SCROLL_DELAY = speed.toInt();
    }
  }
  
  return(isValid);
}

void handleWiFi(void)
{
  static enum { S_IDLE, S_WAIT_CONN, S_READ, S_EXTRACT, S_RESPONSE, S_DISCONN } state = S_IDLE;
  static char szBuf[1024];
  static uint16_t idxBuf = 0;
  static WiFiClient client;
  static uint32_t timeStart;

  switch (state)
  {
    case S_IDLE:   // initialize
      PRINTS("\nS_IDLE");
      idxBuf = 0;
      state = S_WAIT_CONN;
      break;
  
    case S_WAIT_CONN:   // waiting for connection
      {
        client = server.available();
        if (!client) break;
        if (!client.connected()) break;
        
        #if DEBUG
          char szTxt[20];
          sprintf(szTxt, "%03d:%03d:%03d:%03d", client.remoteIP()[0], client.remoteIP()[1], client.remoteIP()[2], client.remoteIP()[3]);
          PRINT("\nNew client @ ", szTxt);
        #endif
        
        timeStart = millis();
        state = S_READ;
      }
      break;
  
    case S_READ: // get the first line of data
      PRINTS("\nS_READ");
      while (client.available())
      {
        char c = client.read();
        if ((c == '\r') || (c == '\n'))
        {
          szBuf[idxBuf] = '\0';
          client.flush();
          PRINT("\nRecv: ", szBuf);
          state = S_EXTRACT;
        }
        else
        {
          szBuf[idxBuf++] = (char)c;
        }
      }
      if (millis() - timeStart > 1000)
      {
        PRINTS("\nWait timeout");
        state = S_DISCONN;
      }
      break;
  
    case S_EXTRACT: // extract data
      PRINTS("\nS_EXTRACT");
      // Extract the string from the message if there is one
      newMessageAvailable = getText(szBuf, newMessage, MESG_SIZE);
      PRINT("\nNew Msg: ", newMessage);
      state = S_RESPONSE;
      break;
  
    case S_RESPONSE: // send the response to the client
      PRINTS("\nS_RESPONSE");
      // Return the response to the client (web page)
      client.print(WebResponse);
      client.print(WebPage);
      state = S_DISCONN;
      break;
  
    case S_DISCONN: // disconnect client
      PRINTS("\nS_DISCONN");
      client.flush();
      client.stop();
      state = S_IDLE;
      break;
  
    default:  state = S_IDLE;
  }
}

void scrollDataSink(uint8_t dev, MD_MAX72XX::transformType_t t, uint8_t col)
// Callback function for data that is being scrolled off the display
{
  // we need this only to feed the data to the next line
  // but we have a single line display and so we do not have to do anything
}

uint8_t scrollDataSource(uint8_t dev, MD_MAX72XX::transformType_t t)
// Callback function for data that is required for scrolling into the display
{
  static enum { S_IDLE, S_NEXT_CHAR, S_SHOW_CHAR, S_SHOW_SPACE } state = S_IDLE;
  static char *p;
  static uint16_t curLen, showLen;
  static uint8_t  cBuf[8];
  uint8_t colData = 0;

  // finite state machine to control what we do on the callback
  switch (state)
  {
    case S_IDLE: // reset the message pointer and check for new message to load
      PRINTS("\nS_IDLE");
      p = curMessage;      // reset the pointer to start of message
      if (newMessageAvailable)  // there is a new message waiting
      {
        strcpy(curMessage, newMessage); // copy it in
        newMessageAvailable = false;
      }
      state = S_NEXT_CHAR;
      break;
  
    case S_NEXT_CHAR: // Load the next character from the font table
      PRINTS("\nS_NEXT_CHAR");
      if (*p == '\0')
        state = S_IDLE;
      else
      {
        showLen = mx.getChar(*p++, sizeof(cBuf) / sizeof(cBuf[0]), cBuf);
        curLen = 0;
        state = S_SHOW_CHAR;
      }
      break;
  
    case S_SHOW_CHAR: // display the next part of the character
      PRINTS("\nS_SHOW_CHAR");
      colData = cBuf[curLen++];
      if (curLen < showLen)
        break;
  
      // set up the inter character spacing
      showLen = (*p != '\0' ? CHAR_SPACING : (MAX_DEVICES*COL_SIZE)/2);
      curLen = 0;
      state = S_SHOW_SPACE;
      // fall through
  
    case S_SHOW_SPACE:  // display inter-character spacing (blank column)
      PRINT("\nS_ICSPACE: ", curLen);
      PRINT("/", showLen);
      curLen++;
      if (curLen == showLen)
        state = S_NEXT_CHAR;
      break;
  
    default:
      state = S_IDLE;
  }

  return(colData);
}

void scrollText(void)
{
  static uint32_t	prevTime = 0;

  // Is it time to scroll the text?
  if (millis() - prevTime >= SCROLL_DELAY)
  {
    mx.transform(MD_MAX72XX::TSL);  // scroll along - the callback will load all the data
    prevTime = millis();      // starting point for next time
  }
}

void setup()
{
  #if DEBUG
    Serial.begin(115200);
    PRINTS("\n[WiFi Message Display]\nType a message for the scrolling display from your internet browser");
  #endif

  // Display initialization
  mx.begin();
  mx.control(MD_MAX72XX::INTENSITY, 0);
  mx.control(MD_MAX72XX::UPDATE, MD_MAX72XX::ON);
  mx.clear();

  mx.setShiftDataInCallback(scrollDataSource);
  mx.setShiftDataOutCallback(scrollDataSink);

  curMessage[0] = newMessage[0] = '\0';

  // this is the IP (Internet Protocol) address of my web server
  IPAddress ip(192,168,1,24);

  // this is the address of my gateway
  // this is the end point and all communications stop at this address
  IPAddress gateway(192,168,1,1); 
  IPAddress subnet(255,255,255,0); 

  // configure the access point's ip and gateway
  WiFi.softAPConfig(ip, gateway, subnet);

  // we need to secure the access point so that unauthorized users cannot connect to it
  WiFi.softAP(ssid, password);

  // Start the server
  server.begin();

  sprintf(curMessage, "%s", "I am Ready!");
}

void loop()
{
  handleWiFi();
  scrollText();
}
```

## Display Router
``` cpp
// Anjena Raja - Individual Creativity Project 

#include <ESP8266WiFi.h>
#include <MD_MAX72xx.h>
#include <SPI.h>

#define DEBUG 0

#if DEBUG
#define PRINT(s, v) { Serial.print(F(s)); Serial.print(v); }
#define PRINTS(s)   { Serial.print(F(s)); }
#else
#define PRINT(s, v)
#define PRINTS(s)
#endif

// Define the number of devices we have in the chain and the hardware interface
#define HARDWARE_TYPE MD_MAX72XX::ICSTATION_HW
#define MAX_DEVICES 8

#define CLK_PIN   D5 // or SCK
#define DATA_PIN  D7 // or MOSI
#define CS_PIN    D8 // or SS

// SPI hardware interface
MD_MAX72XX mx = MD_MAX72XX(HARDWARE_TYPE, CS_PIN, MAX_DEVICES);
// Arbitrary pins
//MD_MAX72XX mx = MD_MAX72XX(HARDWARE_TYPE, DATA_PIN, CLK_PIN, CS_PIN, MAX_DEVICES);


// Animation

// --------------------
// Constant parameters
//
#define ANIMATION_DELAY 75  // milliseconds
#define MAX_FRAMES      4   // number of animation frames

// ========== General Variables ===========
//
const uint8_t pacman[MAX_FRAMES][18] =  // ghost pursued by a pacman
{
  { 0xfe, 0x73, 0xfb, 0x7f, 0xf3, 0x7b, 0xfe, 0x00, 0x00, 0x00, 0x3c, 0x7e, 0x7e, 0xff, 0xe7, 0xc3, 0x81, 0x00 },
  { 0xfe, 0x7b, 0xf3, 0x7f, 0xfb, 0x73, 0xfe, 0x00, 0x00, 0x00, 0x3c, 0x7e, 0xff, 0xff, 0xe7, 0xe7, 0x42, 0x00 },
  { 0xfe, 0x73, 0xfb, 0x7f, 0xf3, 0x7b, 0xfe, 0x00, 0x00, 0x00, 0x3c, 0x7e, 0xff, 0xff, 0xff, 0xe7, 0x66, 0x24 },
  { 0xfe, 0x7b, 0xf3, 0x7f, 0xf3, 0x7b, 0xfe, 0x00, 0x00, 0x00, 0x3c, 0x7e, 0xff, 0xff, 0xff, 0xff, 0x7e, 0x3c },
};
const uint8_t DATA_WIDTH = (sizeof(pacman[0])/sizeof(pacman[0][0]));

uint32_t prevTimeAnim = 0;  // remember the millis() value in animations
int16_t idx;                // display index (column)
uint8_t frame;              // current animation frame
uint8_t deltaFrame;         // the animation frame offset for the next frame

void resetMatrix(void)
{
  mx.control(MD_MAX72XX::INTENSITY, 0);
  mx.control(MD_MAX72XX::UPDATE, MD_MAX72XX::ON);
  mx.clear();
}

void animationLoop(void)
{
  static boolean bInit = true;  // initialise the animation

  // Is it time to animate?
  if (millis()-prevTimeAnim < ANIMATION_DELAY)
    return;
  prevTimeAnim = millis();      // starting point for next time

  mx.control(MD_MAX72XX::UPDATE, MD_MAX72XX::OFF);

  // Initialize
  if (bInit)
  {
    mx.clear();
    idx = -DATA_WIDTH;
    frame = 0;
    deltaFrame = 1;
    bInit = false;

    // Lay out the dots
    for (uint8_t i=0; i<MAX_DEVICES; i++)
    {
      mx.setPoint(3, (i*COL_SIZE) + 3, true);
      mx.setPoint(4, (i*COL_SIZE) + 3, true);
      mx.setPoint(3, (i*COL_SIZE) + 4, true);
      mx.setPoint(4, (i*COL_SIZE) + 4, true);
    }
  }

  // now run the animation
  PRINT("\nINV I:", idx);
  PRINT(" frame ", frame);

  // clear old graphic
  for (uint8_t i=0; i<DATA_WIDTH; i++)
    mx.setColumn(idx-DATA_WIDTH+i, 0);
  // move reference column and draw new graphic
  idx++;
  for (uint8_t i=0; i<DATA_WIDTH; i++)
    mx.setColumn(idx-DATA_WIDTH+i, pacman[frame][i]);

  // advance the animation frame
  frame += deltaFrame;
  if (frame == 0 || frame == MAX_FRAMES-1)
    deltaFrame = -deltaFrame;

  // check if we are completed and set initialise for next time around
  bInit = (idx == mx.getColumnCount()+DATA_WIDTH);

  mx.control(MD_MAX72XX::UPDATE, MD_MAX72XX::ON);

  return;
}

// I am creating my own WiFi Access Point
// Access Point's name or SSID (Service Set IDentifier)
const char *ssid = "Mottu_Kottu";
// Access Point's password 
const char *password = "AnjenaRaja";

// WiFi Server object and parameters
WiFiServer server(80);

// Global message buffers shared by Wifi and Scrolling functions
const uint8_t MESG_SIZE = 255;
const uint8_t CHAR_SPACING = 1;
uint8_t SCROLL_DELAY = 50;

char curMessage[MESG_SIZE];
char newMessage[MESG_SIZE];
bool newMessageAvailable = false;

char WebResponse[] = "HTTP/1.1 200 OK\nContent-Type: text/html\n\n";

char WebPage[] =
"<!DOCTYPE html>" \
"<html>" \
"<head>" \
"<title>Display Control</title>" \

"<script>" \
"strLine = \"\";" \

"function SendText()" \
"{" \
"  nocache = \"/&nocache=\" + Math.random() * 1000000;" \
"  var request = new XMLHttpRequest();" \
"  strLine = \"&MSG=\" + document.getElementById(\"txt_form\").Message.value;" \
"  request.open(\"GET\", strLine + nocache, false);" \
"  request.send(null);" \
"}" \
"</script>" \
"</head>" \

"<body>" \
"<p><b>Type the Message</b></p>" \

"<form id=\"txt_form\" name=\"frmText\">" \
"<label>Msg:<input type=\"text\" name=\"Message\" maxlength=\"255\"></label><br><br>" \
"</form>" \
"<br>" \
"<input type=\"submit\" value=\"Send\" onclick=\"SendText()\">" \
"</body>" \
"</html>";

char *err2Str(wl_status_t code)
{
  switch (code)
  {
    case WL_IDLE_STATUS:    return("IDLE");           break; // WiFi is in process of changing between statuses
    case WL_NO_SSID_AVAIL:  return("NO_SSID_AVAIL");  break; // case configured SSID cannot be reached
    case WL_CONNECTED:      return("CONNECTED");      break; // successful connection is established
    case WL_CONNECT_FAILED: return("CONNECT_FAILED"); break; // password is incorrect
    case WL_DISCONNECTED:   return("CONNECT_FAILED"); break; // module is not configured in station mode
    default: return("??");
  }
}

uint8_t htoi(char c)
{
  c = toupper(c);
  if ((c >= '0') && (c <= '9')) return(c - '0');
  if ((c >= 'A') && (c <= 'F')) return(c - 'A' + 0xa);
  return(0);
}

boolean getText(char *szMesg, char *psz, uint8_t len)
{
  boolean isValid = false;  // text received flag
  char *pStart, *pEnd;      // pointer to start and end of text

  // get pointer to the beginning of the text
  pStart = strstr(szMesg, "/&MSG=");

  if (pStart != NULL)
  {
    pStart += 6;  // skip to start of data
    pEnd = strstr(pStart, "/&");

    if (pEnd != NULL)
    {
      while (pStart != pEnd)
      {
        if ((*pStart == '%'))// && isdigit(*(pStart+1)))
        {
          // replace %xx hex code with the ASCII character
          char c = 0;
          pStart++;
          c += (htoi(*pStart++) << 4);
          c += htoi(*pStart++);
          *psz++ = c;
        }
        else
        {
          *psz++ = *pStart++;
        }
      }

      *psz = '\0'; // terminate the string
      isValid = true;
    }
  }

  String speed = "";
  pStart = strstr(szMesg, "/&SPD=");

  PRINT("\npStart=", pStart);
  if (pStart != NULL)
  {
    pStart += 6;  // skip to start of data
    pEnd = strstr(pStart, "/&");

    PRINT("\npEnd=", pEnd);

    if (pEnd != NULL)
    {
      while (pStart != pEnd)
      {
        if ((*pStart == '%') && isdigit(*(pStart+1)))
        {
          // replace %xx hex code with the ASCII character
          char c = 0;
          pStart++;
          c += (htoi(*pStart++) << 4);
          c += htoi(*pStart++);
          PRINT("\nC=", c);
        }
        else
        {
          speed += *pStart;
          PRINT("\npStart=", *pStart);
          *pStart++;
        }
      }
      PRINT("\npSpeed=", speed);
      SCROLL_DELAY = speed.toInt();
    }
  }
  
  return(isValid);
}

void handleWiFi(void)
{
  static enum { S_IDLE, S_WAIT_CONN, S_READ, S_EXTRACT, S_RESPONSE, S_DISCONN } state = S_IDLE;
  static char szBuf[1024];
  static uint16_t idxBuf = 0;
  static WiFiClient client;
  static uint32_t timeStart;

  switch (state)
  {
    case S_IDLE:   // initialize
      PRINTS("\nS_IDLE");
      idxBuf = 0;
      state = S_WAIT_CONN;
      break;
  
    case S_WAIT_CONN:   // waiting for connection
      {
        client = server.available();
        if (!client) break;
        if (!client.connected()) break;
        
        #if DEBUG
          char szTxt[20];
          sprintf(szTxt, "%03d:%03d:%03d:%03d", client.remoteIP()[0], client.remoteIP()[1], client.remoteIP()[2], client.remoteIP()[3]);
          PRINT("\nNew client @ ", szTxt);
        #endif
        
        timeStart = millis();
        state = S_READ;
      }
      break;
  
    case S_READ: // get the first line of data
      PRINTS("\nS_READ");
      while (client.available())
      {
        char c = client.read();
        if ((c == '\r') || (c == '\n'))
        {
          szBuf[idxBuf] = '\0';
          client.flush();
          PRINT("\nRecv: ", szBuf);
          state = S_EXTRACT;
        }
        else
        {
          szBuf[idxBuf++] = (char)c;
        }
      }
      if (millis() - timeStart > 1000)
      {
        PRINTS("\nWait timeout");
        state = S_DISCONN;
      }
      break;
  
    case S_EXTRACT: // extract data
      PRINTS("\nS_EXTRACT");
      // Extract the string from the message if there is one
      newMessageAvailable = getText(szBuf, newMessage, MESG_SIZE);
      PRINT("\nNew Msg: ", newMessage);
      state = S_RESPONSE;
      break;
  
    case S_RESPONSE: // send the response to the client
      PRINTS("\nS_RESPONSE");
      // Return the response to the client (web page)
      client.print(WebResponse);
      client.print(WebPage);
      state = S_DISCONN;
      break;
  
    case S_DISCONN: // disconnect client
      PRINTS("\nS_DISCONN");
      client.flush();
      client.stop();
      state = S_IDLE;
      break;
  
    default:  state = S_IDLE;
  }
}

void scrollDataSink(uint8_t dev, MD_MAX72XX::transformType_t t, uint8_t col)
// Callback function for data that is being scrolled off the display
{
}

uint8_t scrollDataSource(uint8_t dev, MD_MAX72XX::transformType_t t)
// Callback function for data that is required for scrolling into the display
{
  static enum { S_IDLE, S_NEXT_CHAR, S_SHOW_CHAR, S_SHOW_SPACE } state = S_IDLE;
  static char *p;
  static uint16_t curLen, showLen;
  static uint8_t  cBuf[8];
  uint8_t colData = 0;

  // finite state machine to control what we do on the callback
  switch (state)
  {
    case S_IDLE: // reset the message pointer and check for new message to load
      PRINTS("\nS_IDLE");
      p = curMessage;      // reset the pointer to start of message
      if (newMessageAvailable)  // there is a new message waiting
      {
        strcpy(curMessage, newMessage); // copy it in
        newMessageAvailable = false;
      }
      state = S_NEXT_CHAR;
      break;
  
    case S_NEXT_CHAR: // Load the next character from the font table
      PRINTS("\nS_NEXT_CHAR");
      if (*p == '\0')
        state = S_IDLE;
      else
      {
        showLen = mx.getChar(*p++, sizeof(cBuf) / sizeof(cBuf[0]), cBuf);
        curLen = 0;
        state = S_SHOW_CHAR;
      }
      break;
  
    case S_SHOW_CHAR: // display the next part of the character
      PRINTS("\nS_SHOW_CHAR");
      colData = cBuf[curLen++];
      if (curLen < showLen)
        break;
  
      // set up the inter character spacing
      showLen = (*p != '\0' ? CHAR_SPACING : (MAX_DEVICES*COL_SIZE)/2);
      curLen = 0;
      state = S_SHOW_SPACE;
      // fall through
  
    case S_SHOW_SPACE:  // display inter-character spacing (blank column)
      PRINT("\nS_ICSPACE: ", curLen);
      PRINT("/", showLen);
      curLen++;
      if (curLen == showLen)
        state = S_NEXT_CHAR;
      break;
  
    default:
      state = S_IDLE;
  }

  return(colData);
}

void scrollText(void)
{
  static uint32_t	prevTime = 0;

  // Is it time to scroll the text?
  if (millis() - prevTime >= SCROLL_DELAY)
  {
    mx.transform(MD_MAX72XX::TSL);  // scroll along - the callback will load all the data
    prevTime = millis();      // starting point for next time
  }
}

void setup()
{
  #if DEBUG
    Serial.begin(115200);
    PRINTS("\n[Anjena Display]\nType a message for the scrolling display from your internet browser");
  #endif

  // Connect to and initialize WiFi network
  PRINT("\nConnecting to ", ssid);

  // this is the IP (Internet Protocol) address of my web server
  IPAddress ip(192,168,1,24);
  IPAddress gateway(192,168,1,1); 
  IPAddress subnet(255,255,255,0); 
  
  WiFi.config(ip, subnet, gateway);
  WiFi.begin(ssid, password);
  
  // Display initialization
  mx.begin();
  resetMatrix();

  // play animation
  prevTimeAnim = millis();

  int counter = 2000;
  while (counter > 0) 
  {
    animationLoop();
    delay(ANIMATION_DELAY/10);
    counter--;
  }

  mx.setShiftDataInCallback(scrollDataSource);
  mx.setShiftDataOutCallback(scrollDataSink);

  curMessage[0] = newMessage[0] = '\0';

  while (WiFi.status() != WL_CONNECTED)
  {
    PRINT("\n", err2Str(WiFi.status()));
    delay(500);
  }
  PRINTS("\nWiFi connected");

  // Start the server
  server.begin();
  PRINTS("\nServer started");
  // Set up first message as the IP address
  //sprintf(curMessage, "%03d:%03d:%03d:%03d", WiFi.localIP()[0], WiFi.localIP()[1], WiFi.localIP()[2], WiFi.localIP()[3]);

  sprintf(curMessage, "%s", "I am ready!");
}

void loop()
{
  handleWiFi();
  scrollText();
}
```

#### [Link to video of the demo](https://www.youtube.com/watch?v=nYh8BSovpZA){:target="_blank"}
