---
layout: post
title: Arduino&#58;&nbsp;Working with the 2.8" Touchscreen Module
author: Daniel Müller
date: '2019-10-16'
category: guides
tags: arduino
summary: Playing around with Arduino Uno and 2.8" Touchscreen Module
thumbnail: post1_touchscreen/logo.jpg
---

# Arduino: 2,8" Touchscreen (ILI9341)

<span style="color:red">Important: The libraries I used in this blog post won't support the driver of the **ILI9486** display (3.5") yet!</span>

### 1. Installing the Libraries

Install the following libraries using the Arduino IDE (Menu "Scetch" > "Include Library" > "Manage Libraries"):

* Adafruit TFTLCD
* Adafruit Touchscreen
* (optional?) Adafruit BusIO
* (optional?) Adafruit GFX

### 2. Configure the Correct Display Resoultion in Library files

* In your library directory (defaults to C:\Users\Username\Documents\Arduino\libraries) make the following changes in the **Adafruit TFTLCD** library:

  * Adafruit_TFTLCD.h
    Check, if following line is commented out (it must be commented out): 

    ``````
    //#define USE_ADAFRUIT_SHIELD_PINOUT 1
    ``````

  * Adafruit_TFTLCD.cpp
    Comment in the lines of code that contain the correct resolution for your touchscreen (and comment out the others):

    ``````c++
    //#define TFTWIDTH   320
    //#define TFTHEIGHT  480
    
    #define TFTWIDTH 240
    #define TFTHEIGHT 320
    ``````

### 3. Plug in the Touchscreen on Arduino Uno

The following pin mapping is important for coding later:

* A4 = LCD_RST
* A3 = LCD_CS
* A2 = LCD_RS
* A1 = LCD_WR
* A0 = LCD_RD

### 4. Example Code

Following code block creates an instance of type **TFT** (displaying) and **Touchscreen** (input):

* Hint: The pins A2 and A3 are used twice here!

``````c++
#include <Adafruit_TFTLCD.h>     
#include <TouchScreen.h>
//#include <Adafruit_GFX.h>

// Pins for LCD
#define LCD_RESET A4 
#define LCD_CS    A3 
#define LCD_CD    A2 
#define LCD_WR    A1 
#define LCD_RD    A0 

// Pins for Touchscreen
#define YP A2  // must be an analog pin
#define XM A3  // must be an analog pin
#define YM 8   // can be a digital pin
#define XP 9   // can be a digital pin

// Create the instances
Adafruit_TFTLCD tft(LCD_CS, LCD_CD, LCD_WR, LCD_RD, LCD_RESET);
TouchScreen ts = TouchScreen(XP, YP, XM, YM, 300);

void setup() {
  Serial.begin(9600);
  Serial.print("Starting...");
  randomSeed(analogRead(0));
 
  // Initialize TFT Display
  tft.reset();
  tft.begin(0x9341);		// This must correspond to your display driver (e.g. ILI9486 is not supported here, yet)
  tft.setRotation(1);
}
``````

**Example: Draw some forms and show a text on the display**

``````c++
#define BLACK   0x0000
#define RED     0xF800
#define WHITE   0xFFFF

void setup() {
  ... // Code from above
  
  // Background
  tft.fillScreen(BLACK);
    
  // White Rectangle
  tft.drawRect(0,0,319,240,WHITE);
  
  // Red Text
  tft.setCursor(30,100);
  tft.setTextColor(RED);
  tft.setTextSize(4);
  tft.print("Hello World");
    
  //Create Red Button
  tft.fillRect(60,180, 200, 40, RED);
  tft.drawRect(60,180,200,40,WHITE);
  tft.setCursor(90,188);
  tft.setTextColor(WHITE);
  tft.setTextSize(3);
  tft.print("Click Me");
}
``````

**Example: React on touch events**

``````c++
... 							// Initialization from above

void setup() {
  ... 							// Setup-Code from above
}

void loop() {
  TSPoint p = ts.getPoint();  	//Get touch point
    
  if (p.z > ts.pressureThreshhold) {
    Serial.print("X = "); Serial.print(p.x);
    Serial.print("\tY = "); Serial.print(p.y);
    Serial.print("\tPressure = "); Serial.println(p.z); 
    
    delay(100)					// Avoid bouncing
  }
}
``````

**Scale the pressed point on touchscreen to a 240 x 320 matrix**

``````c++
... 							// Initialization from above

// Set outer limits of the touchscreen (use the code above => without scaling)
#define TS_MINX 110
#define TS_MINY 110
#define TS_MAXX 960
#define TS_MAXY 920

void setup() {
  ... 							// Setup-Code from above
}

void loop() {
  TSPoint p = ts.getPoint();  	// Get touch point
    
  if (p.z > ts.pressureThreshhold) {
    
    // Scaling the value of the touched point (here for a 320 x 240 pixel TFT)
    p.x = map(p.x, TS_MINX, TS_MAXX, 0, 320);
    p.y = map(p.y, TS_MINY, TS_MAXY, 0, 240);
      
    Serial.print("X = "); Serial.print(p.x);
    Serial.print("\tY = "); Serial.print(p.y);
    Serial.print("\tPressure = "); Serial.println(p.z); 
    
    delay(100);					// Avoid bouncing
  }
}
``````

**Check for supported display types**

``````c++
void initDisplay()
{
  tft.reset();
  uint16_t identifier = tft.readID(); 					// Read the ID of the TFT instance (this returns the driver code)
    
  if(identifier == 0x9325) {
    Serial.println(F("Found ILI9325 LCD driver"));
  } else if(identifier == 0x9328) {
    Serial.println(F("Found ILI9328 LCD driver"));
  } else if(identifier == 0x7575) {
    Serial.println(F("Found HX8347G LCD driver"));
  } else if(identifier == 0x9341) {
    Serial.println(F("Found ILI9341 LCD driver"));
  } else if(identifier == 0x8357) {
    Serial.println(F("Found HX8357D LCD driver"));
  } else {
    Serial.print(F("Unknown LCD driver chip: "));
    Serial.println(identifier);
    return;
  }
  
  tft.begin(identifier);
  tft.setRotation(1);
}
``````

### 5. Complete example: React on touch events and write the position on the display

``````c++
#include <Adafruit_TFTLCD.h>
#include <TouchScreen.h>
//#include <Adafruit_GFX.h>

#define BLACK   0x0000
#define RED     0xF800
#define WHITE   0xFFFF

// Pins for LCD
#define LCD_RESET A4
#define LCD_CS    A3
#define LCD_CD    A2
#define LCD_WR    A1
#define LCD_RD    A0

// Pins for Touchscreen
#define YP A2  // must be an analog pin
#define XM A3  // must be an analog pin
#define YM 8   // can be a digital pin
#define XP 9   // can be a digital pin

// Set outer limits of the touchscreen (use the code above => without scaling)
#define TS_MINX 110
#define TS_MINY 110
#define TS_MAXX 960
#define TS_MAXY 920

Adafruit_TFTLCD tft(LCD_CS, LCD_CD, LCD_WR, LCD_RD, LCD_RESET);
TouchScreen ts = TouchScreen(XP, YP, XM, YM, 300);

void setup() {
  Serial.begin(9600);
  Serial.print("Starting...");
  randomSeed(analogRead(0));

  // Initialize TFT Display
  tft.reset();
  tft.begin(0x9341);    // This must correspond to your display driver (e.g. ILI9486 is not supported here, yet)
  tft.setRotation(1);

  drawClickMeScreen();
}

void loop() {
  TSPoint p = ts.getPoint();    //Get touch point

  if (p.z > ts.pressureThreshhold) {

    // Scaling the value of the touched point (here for a 320 x 240 pixel TFT)
    p.x = map(p.x, TS_MINX, TS_MAXX, 0, 320);
    p.y = map(p.y, TS_MINY, TS_MAXY, 0, 240);

    Serial.print("X = "); Serial.print(p.x);
    Serial.print("\tY = "); Serial.print(p.y);
    Serial.print("\tPressure = "); Serial.println(p.z);

    // This is important, because the libraries are sharing pins => Switch to "Writing Mode" before Display drawing
    pinMode(XM, OUTPUT);
    pinMode(YP, OUTPUT);

    // Reset to the "initial" Display content (remove the last coordinate text)
    drawClickMeScreen();

    // Show position as Text
    tft.setCursor(30, 30);
    tft.setTextColor(WHITE);
    tft.setTextSize(2);
    tft.print(String(p.x) + " / " + String(p.y));

    if(p.x >= 60 && p.x <= 260 && p.y >= 180 && p.y <= 220) {
      // Was the touch inside the button? => if yes, show info text
      tft.setCursor(30, 50);
      tft.setTextColor(WHITE);
      tft.setTextSize(2);
      tft.print("Button Clicked");
    }

    delay(100);          // Avoid bouncing
  }
}

void drawClickMeScreen() {
  // Background
  tft.fillScreen(BLACK);

  // White Rectangle
  tft.drawRect(0, 0, 319, 240, WHITE);

  // Red Text
  tft.setCursor(30, 100);
  tft.setTextColor(RED);
  tft.setTextSize(4);
  tft.print("Hello World");

  //Create Red Button
  tft.fillRect(60, 180, 200, 40, RED);
  tft.drawRect(60, 180, 200, 40, WHITE);
  tft.setCursor(90, 188);
  tft.setTextColor(WHITE);
  tft.setTextSize(3);
  tft.print("Click Me");
}
``````

### Problem solving

* **Drawing and Touch events won't work together:**
  Here you must check, if you set the double-used pins (A2 and A3 in the examples avove) to "output" mode before using the methods for drawing:

  ``````c++
  //This is important, because the libraries are sharing pins => Switch to "Writing Mode" before Display drawing
  pinMode(XM, OUTPUT);
  pinMode(YP, OUTPUT);
  
  // Now you can draw something
  tft.drawRect(0, 0, 319, 240, WHITE);
  ``````