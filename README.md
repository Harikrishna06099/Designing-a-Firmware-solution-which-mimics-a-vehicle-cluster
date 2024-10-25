# Designing-a-Firmware-solution-which-mimics-a-vehicle-cluster

#include <LiquidCrystal.h>
#include <SD.h>
#include <EEPROM.h>

// Define constants and variables
const int POTENTIOMETER_PIN = A0;
const int SWITCH_PIN = 2;
const int PUSH_BUTTON_PIN = 3;
const int LCD_RS_PIN = 4;
const int LCD_EN_PIN = 5;
const int LCD_DATA_PIN = 6;
const int SD_CS_PIN = 7;

LiquidCrystal lcd(LCD_RS_PIN, LCD_EN_PIN, LCD_DATA_PIN);

int speed = 0;
int batteryLevel = 0;
int gear = 0;
int driveMode = 0;
int currentTime = 0;
int currentDate = 0;

void setup() {
  Serial.begin(9600);
  // Initialize LCD display
  lcd.begin(16, 2);
  // Initialize SD card
  if (!SD.begin(SD_CS_PIN)) {
    Serial.println("SD card initialization failed");
    while(1);
  }
  // Initialize EEPROM
  EEPROM.begin();
  pinMode(PUSH_BUTTON_PIN, INPUT_PULLUP);
  attachInterrupt(digitalPinToInterrupt(PUSH_BUTTON_PIN), ISR_pushButton, FALLING);
}

void loop() {
  // Read sensor inputs
  speed = analogRead(POTENTIOMETER_PIN);
  batteryLevel = map(analogRead(POTENTIOMETER_PIN), 0, 1023, 0, 100);
  gear = digitalRead(SWITCH_PIN);
  driveMode = digitalRead(SWITCH_PIN);

  // Update LCD display
  lcd.setCursor(0, 0);
  lcd.print("Speed: ");
  lcd.print(map(speed, 0, 1023, 0, 100));
  lcd.print(" km/h");
  lcd.setCursor(0, 1);
  lcd.print("Battery: ");
  lcd.print(batteryLevel);
  lcd.print("%");
  lcd.setCursor(8, 1);
  lcd.print("Gear: ");
  lcd.print(gear);
  lcd.print(" ");

  // Log data to SD card
  File dataFile = SD.open("log.txt", FILE_WRITE);
  if (dataFile) {
    dataFile.print(millis());
    dataFile.print(",");
    dataFile.print(map(speed, 0, 1023, 0, 100));
    dataFile.print(",");
    dataFile.print(batteryLevel);
    dataFile.print(",");
    dataFile.println(gear);
    dataFile.close();
  } else {
    Serial.println("Error opening log file");
  }

  delay(5000);
}

void ISR_pushButton() {
  // Handle push button press
  // Set current time and date
  currentTime = EEPROM.read(0);
  currentDate = EEPROM.read(1);
}

void ISR_timer() {
  // Update time and date
  currentTime++;
  EEPROM.write(0, currentTime);
}
