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

<span style="color:red">Wichtig: Der Driver für das **ILI9486** Display (3,5") wird momentan noch nicht von den unten verwendeten Libraries unterstützt!</span>

### 1. Libraries installieren

Über die Arduino IDE (Menü "Sketch" > "Bibliothek einbinden" > "Bibliotheken verwalten") folgende Bibliotheken installieren:

* Adafruit TFTLCD
* Adafruit Touchscreen
* (testen, ob es benötigt wird) Adafruit BusIO
* (testen, ob es benötigt wird) Adafruit GFX

### 2. Display-Auflösung in Library konfigurieren

* Im Bibliotheken-Ordner (C:\Users\Daniel\Dokumente\Arduino\libraries) folgende Änderungen für die **Adafruit TFTLCD** Bibliothek vornehmen:

  * Adafruit_TFTLCD.h
    Prüfen, ob folgende Zeile auskommentiert ist (muss auskommentiert sein):

    ``````
    //#define USE_ADAFRUIT_SHIELD_PINOUT 1
    ``````

  * Adafruit_TFTLCD.cpp
    Die Zeilen mit der richtigen Displayauflösung einkommentieren, die anderen Zeilen auskommentieren:

    ``````c++
    //#define TFTWIDTH   320
    //#define TFTHEIGHT  480
    
    #define TFTWIDTH 240
    #define TFTHEIGHT 320
    ``````

### 3. Am Arduino Uno anstecken

Folgendes Pin-Mapping ist für das Coding wichtig:

* A4 = LCD_RST
* A3 = LCD_CS
* A2 = LCD_RS
* A1 = LCD_WR
* A0 = LCD_RD

### 4. Beispiel-Code

Folgender Codeblock erstellt sowohl eine Instanz vom Typ **TFT** (Anzeige), als auch **Touchscreen** (Eingabe):

* Hinweis: Die Pins A2 und A3 werden hier doppelt belegt!

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
#define YP A2  // must be an analog pin, use "An" notation!
#define XM A3  // must be an analog pin, use "An" notation!
#define YM 8   // can be a digital pin
#define XP 9   // can be a digital pin

Adafruit_TFTLCD tft(LCD_CS, LCD_CD, LCD_WR, LCD_RD, LCD_RESET);
TouchScreen ts = TouchScreen(XP, YP, XM, YM, 300);

void setup() {
  Serial.begin(9600);
  Serial.print("Starting...");
  randomSeed(analogRead(0));
 
  // Initialize TFT Display
  tft.reset();
  tft.begin(0x9341);		// DAS HIER MUSS DEM DISPLAY-DRIVER ENTSPRECHEN (hier wird ILI9486 noch nicht unterstützt)
  tft.setRotation(1);
}
``````

**Beispiel: Zeichne Formen und Text das Display**

``````c++
#define BLACK   0x0000
#define RED     0xF800
#define WHITE   0xFFFF

void setup() {
  ... // Code von oben
  
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

**Beispiel: Reagiere auf Touch-Events**

``````c++
... 							// Initialisierung von oben

void setup() {
  ... 							// Setup-Code von oben
}

void loop() {
  TSPoint p = ts.getPoint();  	//Get touch point
    
  if (p.z > ts.pressureThreshhold) {
    Serial.print("X = "); Serial.print(p.x);
    Serial.print("\tY = "); Serial.print(p.y);
    Serial.print("\tPressure = "); Serial.println(p.z); 
    
    delay(100)					// Verhindere Prellen
  }
}
``````

**Skaliere den gedrückten Punkt auf dem Display auf eine (240 x 320 Matrix)**

``````c++
... 							// Initialisierung von oben

// Setze äußere Grenzwerte des Touchscreens (mithilfe des obigen Codes => ohne Skalierung)
#define TS_MINX 110
#define TS_MINY 110
#define TS_MAXX 960
#define TS_MAXY 920

void setup() {
  ... 							// Setup-Code von oben
}

void loop() {
  TSPoint p = ts.getPoint();  	//Get touch point
    
  if (p.z > ts.pressureThreshhold) {
    
    // Skaliere den Wert des berührten Punkts (bei 320 x 240 pixel TFT)
    p.x = map(p.x, TS_MINX, TS_MAXX, 0, 320);
    p.y = map(p.y, TS_MINY, TS_MAXY, 0, 240);
      
    Serial.print("X = "); Serial.print(p.x);
    Serial.print("\tY = "); Serial.print(p.y);
    Serial.print("\tPressure = "); Serial.println(p.z); 
    
    delay(100);					// Verhindere Prellen
  }
}
``````

**Prüfung auf unterstützten Display-Typen**

``````c++
void initDisplay()
{
  tft.reset();
  uint16_t identifier = tft.readID();
    
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

### 5. Komplett-Beispiel: Reagiere auf Touch-Events und Schreibe auf das Display

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
#define YP A2  // must be an analog pin, use "An" notation!
#define XM A3  // must be an analog pin, use "An" notation!
#define YM 8   // can be a digital pin
#define XP 9   // can be a digital pin

// Setze äußere Grenzwerte des Touchscreens (mithilfe des obigen Codes)
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
  tft.begin(0x9341);    // DAS HIER MUSS DEM DISPLAY-DRIVER ENTSPRECHEN (hier wird ILI9486 noch nicht unterstützt)
  tft.setRotation(1);

  drawClickMeScreen();
}

void loop() {
  TSPoint p = ts.getPoint();    //Get touch point

  if (p.z > ts.pressureThreshhold) {

    // Skaliere den Wert des berührten Punkts (bei 320 x 240 pixel TFT)
    p.x = map(p.x, TS_MINX, TS_MAXX, 0, 320);
    p.y = map(p.y, TS_MINY, TS_MAXY, 0, 240);

    Serial.print("X = "); Serial.print(p.x);
    Serial.print("\tY = "); Serial.print(p.y);
    Serial.print("\tPressure = "); Serial.println(p.z);

    //This is important, because the libraries are sharing pins => Switch to "Writing Mode" before Display drawing
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
      // Was the touch inside the Button? Show info text
      tft.setCursor(30, 50);
      tft.setTextColor(WHITE);
      tft.setTextSize(2);
      tft.print("Button Clicked");
    }

    delay(100);          // Verhindere Prellen
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

### Fehlersuche

* Zeichnen und Touch-Events funktionieren nicht zusammen:
  Hier muss geprüft werden, ob vor dem Zeichnen die beiden doppelt verwendeten Pins auf Modus "Output" gestellt wurden, ggf. ergänzen:

  ``````
  //This is important, because the libraries are sharing pins => Switch to "Writing Mode" before Display drawing
  pinMode(XM, OUTPUT);
  pinMode(YP, OUTPUT);
  ``````