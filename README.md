# Home-Automation
// ============================================================
// Home Automation System Using Arduino + LDR + Bluetooth
// Haldia Institute of Technology
// ============================================================
 
#include <SoftwareSerial.h>
 
// --- Bluetooth Serial (HC-05) ---
SoftwareSerial BTSerial(10, 11);  // Pin 10 = RX, Pin 11 = TX
 
// --- Relay Pins (Active-LOW: LOW = ON, HIGH = OFF) ---
const int RELAY1 = 2;   // Light 1 (also controlled by LDR)
const int RELAY2 = 3;   // Light 2 / Fan
const int RELAY3 = 4;   // Appliance 3
const int RELAY4 = 5;   // Appliance 4
 
// --- LDR Sensor ---
const int LDR_PIN = A0;
const int LDR_THRESHOLD = 500;  // 0-1023; below this = dark
const int HYSTERESIS    = 30;   // prevents relay chattering
 
// --- State tracking ---
bool manualOverride = false;
bool lightState     = false;
 
// ============================================================
void setup() {
  Serial.begin(9600);       // Debug via USB
  BTSerial.begin(9600);     // HC-05 Bluetooth
 
  // Relay pins as output
  pinMode(RELAY1, OUTPUT);
  pinMode(RELAY2, OUTPUT);
  pinMode(RELAY3, OUTPUT);
  pinMode(RELAY4, OUTPUT);
 
  // Start with all relays OFF (active-low: HIGH = OFF)
  digitalWrite(RELAY1, HIGH);
  digitalWrite(RELAY2, HIGH);
  digitalWrite(RELAY3, HIGH);
  digitalWrite(RELAY4, HIGH);
 
  Serial.println("Home Automation System Ready!");
}
 
// ============================================================
void loop() {
 
  // --- LDR-based Automatic Light Control ---
  if (!manualOverride) {
    int ldrValue = analogRead(LDR_PIN);
    Serial.print("LDR Value: ");
    Serial.println(ldrValue);
 
    if (ldrValue < (LDR_THRESHOLD - HYSTERESIS) && !lightState) {
      // Room is dark: turn on Light 1
      digitalWrite(RELAY1, LOW);
      lightState = true;
      Serial.println("Auto: Light 1 ON (dark detected)");
    }
    else if (ldrValue > (LDR_THRESHOLD + HYSTERESIS) && lightState) {
      // Room is bright: turn off Light 1
      digitalWrite(RELAY1, HIGH);
      lightState = false;
      Serial.println("Auto: Light 1 OFF (light detected)");
    }
  }
 
  // --- Bluetooth Manual Control ---
  if (BTSerial.available()) {
    char cmd = BTSerial.read();
    Serial.print("BT Command: ");
    Serial.println(cmd);
 
    switch (cmd) {
      // Manual ON commands
      case '1': digitalWrite(RELAY1, LOW);  manualOverride=true;  lightState=true;  break;
      case '2': digitalWrite(RELAY2, LOW);  break;
      case '3': digitalWrite(RELAY3, LOW);  break;
      case '4': digitalWrite(RELAY4, LOW);  break;
      // Manual OFF commands
      case 'a': digitalWrite(RELAY1, HIGH); manualOverride=false; lightState=false; break;
      case 'b': digitalWrite(RELAY2, HIGH); break;
      case 'c': digitalWrite(RELAY3, HIGH); break;
      case 'd': digitalWrite(RELAY4, HIGH); break;
      // Reset to automatic mode
      case 'r': manualOverride = false; Serial.println('Auto mode ON'); break;
    }
  }
 
  delay(100);  // 100ms loop delay
}

