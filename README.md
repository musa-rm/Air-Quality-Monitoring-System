# Air-Quality-Monitoring-System
 An Arduino-powered air quality and temperature monitoring system
#include <SPI.h>
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>
#include <Fonts/FreeSans9pt7b.h>
#include <Fonts/FreeMonoOblique9pt7b.h>
#include <DHT.h>

#define SCREEN_WIDTH 128 
#define SCREEN_HEIGHT 64
#define OLED_RESET -1
#define SCREEN_ADDRESS 0x3D ///< See datasheet for Address; 0x3D for 128x64, 0x3C for 128x32
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, OLED_RESET);

#define sensor    A0 
#define DHTPIN 2          // Digital pin 2
#define DHTTYPE DHT11     // DHT 11

#define BUTTON_PIN 8      // Define push button pin
int buttonState = 0;      // Variable for reading the push button status
int lastButtonState = 0;  // Variable to store the last button state
unsigned long lastDebounceTime = 0; // the last time the output pin was toggled
unsigned long debounceDelay = 50;    // the debounce time; increase if the output flickers

bool systemOn = false;   // Boolean to track the system state (on/off)

int gasLevel = 0;         // int variable for gas level
String quality = "";
DHT dht(DHTPIN, DHTTYPE);

#define GREEN_LED_PIN 11
#define RED_LED_PIN 12

void sendSensor() {
  float h = dht.readHumidity();
  float t = dht.readTemperature();

  if (isnan(h) || isnan(t)) {
    Serial.println("Failed to read from DHT sensor!");
    return;
  }
  display.setTextColor(WHITE);
  display.setTextSize(1);
  display.setFont();
  display.setCursor(0, 43);
  display.println("Temp  :");
  display.setCursor(80, 43);
  display.println(t);
  display.setCursor(114, 43);
  display.println("C");
  display.setCursor(0, 56);
  display.println("RH    :");
  display.setCursor(80, 56);
  display.println(h);
  display.setCursor(114, 56);
  display.println("%");
}

void air_sensor() {
  gasLevel = analogRead(sensor);

  if (gasLevel < 100) {
    quality = "Great!";
    digitalWrite(GREEN_LED_PIN, HIGH);
    digitalWrite(RED_LED_PIN, LOW);
  } else if (gasLevel > 100 && gasLevel < 200) {
    quality = "Good!";
    digitalWrite(GREEN_LED_PIN, HIGH);
    digitalWrite(RED_LED_PIN, LOW);
  } else if (gasLevel > 200 && gasLevel < 300) {
    quality = "Moderate!";
    digitalWrite(GREEN_LED_PIN, LOW);
    digitalWrite(RED_LED_PIN, LOW);
  } else if (gasLevel > 300 && gasLevel < 400) {
    quality = "Bad!";
    digitalWrite(GREEN_LED_PIN, LOW);
    digitalWrite(RED_LED_PIN, HIGH);
  } else {
    quality = "TOXIC!";
    digitalWrite(GREEN_LED_PIN, LOW);
    digitalWrite(RED_LED_PIN, HIGH);
  }

  display.setTextColor(WHITE);
  display.setTextSize(1);  
  display.setCursor(1, 5);
  display.setFont();
  display.println("Air Quality:");
  display.setTextSize(1);
  display.setCursor(20, 23);
  display.setFont(&FreeMonoOblique9pt7b);
  display.println(quality); 
}

void setup() {
  Serial.begin(9600);
  pinMode(sensor, INPUT);
  pinMode(GREEN_LED_PIN, OUTPUT);
  pinMode(RED_LED_PIN, OUTPUT);
  pinMode(BUTTON_PIN, INPUT_PULLUP);  // Set button pin as input with internal pull-up resistor
  dht.begin();
  if (!display.begin(SSD1306_SWITCHCAPVCC, 0x3c)) { // Address 0x3D for 128x64
    Serial.println(F("SSD1306 allocation failed"));
  }
  display.clearDisplay();
  display.setTextColor(WHITE);

  display.setTextSize(2);
  display.setCursor(50, 0);
  display.println("Air");
  display.setTextSize(1);
  display.setCursor(23, 20);
  display.println("Quality Monitor");
  display.display();
  delay(1200);
  display.clearDisplay();

  display.setTextSize(1);
  display.setCursor(20, 30);
  display.println("Loading.....");
  display.display();
  delay(2000);
  display.clearDisplay();    
}

void loop() {
  int reading = digitalRead(BUTTON_PIN);

  if (reading != lastButtonState) {
    lastDebounceTime = millis();
  }

  if ((millis() - lastDebounceTime) > debounceDelay) {
    if (reading != buttonState) {
      buttonState = reading;
      if (buttonState == LOW) {
        systemOn = !systemOn;
      }
    }
  }

  lastButtonState = reading;

  if (systemOn) {
    display.clearDisplay();
    air_sensor();
    sendSensor();
    display.display();
  } else {
    display.clearDisplay();
    display.display();
    digitalWrite(GREEN_LED_PIN, LOW);
    digitalWrite(RED_LED_PIN, LOW);
  }
}
