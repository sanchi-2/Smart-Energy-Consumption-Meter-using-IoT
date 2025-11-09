/***************************************************
   Smart IoT Energy Meter (ESP32 + Blynk IoT + LCD + EmonLib)
   Author: Atul
   Compatible with Blynk IoT dashboard (Voltage, Current, Power, Unit)
****************************************************/

#define BLYNK_TEMPLATE_ID "TMPL3HsN0_lC-"
#define BLYNK_TEMPLATE_NAME "smart IOT energy meter"
#define BLYNK_AUTH_TOKEN "ouhoSKORym29HCdZgrPK8kLwEk8yRDpx"

#include <WiFi.h>
#include <BlynkSimpleEsp32.h>
#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <EmonLib.h>

// ✅ WiFi credentials
char ssid[] = "OPPO Reno5 Pro 5G";
char pass[] = "12121212";

// ✅ LCD Display 16x2 (check I2C address: 0x27 or 0x3F)
LiquidCrystal_I2C lcd(0x27, 16, 2);

// ✅ EmonLib Energy Monitor setup
EnergyMonitor emon;

// ✅ Calibration values (tune for your sensors)
#define vCalibration 135.0
#define currCalibration 0.5


// ✅ Variables
float voltage = 0;
float current = 0;
float realPower = 0;
float apparentPower = 0;
float powerFactor = 0;
float energy_kWh = 0;

// ✅ Tariff (for optional bill calculation)
#define UNIT_RATE 8.00

// ✅ Timers
unsigned long lastEnergyMillis = 0;
unsigned long lastLCDMillis = 0;
const unsigned long lcdInterval = 2000;
int screen = 0;

BlynkTimer timer;

// ✅ Send readings to Blynk Dashboard
void sendToBlynk()
{
  Blynk.virtualWrite(V0, voltage);     // Voltage gauge
  Blynk.virtualWrite(V1, current);     // Current gauge
  Blynk.virtualWrite(V2, realPower);   // Power gauge
  Blynk.virtualWrite(V3, energy_kWh);  // Unit (kWh) gauge
}

// ✅ Energy alert (optional)
void checkUnitLimit()
{
  if (energy_kWh >= 1.0)  // 1 kWh alert
  {
    String msg = "Energy: " + String(energy_kWh, 3) + " kWh > 1 Unit!";
    Blynk.logEvent("high_energy", msg);
  }
}

void setup()
{
  Serial.begin(9600);
  delay(100);

  // ✅ Connect to Wi-Fi and Blynk Cloud
  Blynk.begin(BLYNK_AUTH_TOKEN, ssid, pass);

  // ✅ LCD setup
  lcd.init();
  lcd.backlight();
  lcd.clear();
  lcd.setCursor(2, 0);
  lcd.print("IoT Energy");
  lcd.setCursor(4, 1);
  lcd.print("Meter");
  delay(2000);
  lcd.clear();

  // ✅ Configure EmonLib pins (ADC pins)
  emon.voltage(35, vCalibration, 1.7); // Voltage on pin 35
  emon.current(34, currCalibration);   // Current on pin 34

  lastEnergyMillis = millis();
  lastLCDMillis = millis();

  // ✅ Timers
  timer.setInterval(2000L, sendToBlynk);
  timer.setInterval(5000L, checkUnitLimit);
}

void loop()
{
  Blynk.run();
  timer.run();

  // ✅ Measure voltage/current
  emon.calcVI(25, 2000);

  voltage = emon.Vrms;
  current = emon.Irms;
  realPower = emon.realPower;
  apparentPower = emon.apparentPower;
  powerFactor = emon.powerFactor;

  // ✅ Debug output to Serial Monitor
  Serial.print("V: "); Serial.print(voltage, 2);
  Serial.print("  I: "); Serial.print(current, 3);
  Serial.print("  P: "); Serial.print(realPower, 2);
  Serial.print("  PF: "); Serial.println(powerFactor, 2);

  unsigned long now = millis();

  // ✅ Calculate Energy in kWh
  if (realPower > 1)
  {
    energy_kWh += realPower * (now - lastEnergyMillis) / 3600000000.0;
  }
  lastEnergyMillis = now;

  // ✅ Update LCD every few seconds
  if (now - lastLCDMillis >= lcdInterval)
  {
    lastLCDMillis = now;
    lcd.clear();

    switch (screen)
    {
      case 0:
        lcd.setCursor(0, 0);
        lcd.print("Voltage: ");
        lcd.print(voltage, 1);
        lcd.print("V");

        lcd.setCursor(0, 1);
        lcd.print("Current: ");
        lcd.print(current, 2);
        lcd.print("A");
        break;

      case 1:
        lcd.setCursor(0, 0);
        lcd.print("Power: ");
        lcd.print(realPower, 1);
        lcd.print("W");

        lcd.setCursor(0, 1);
        lcd.print("Energy: ");
        lcd.print(energy_kWh, 3);
        lcd.print("kWh");
        break;
    }
    screen = (screen + 1) % 2;
  }
}
