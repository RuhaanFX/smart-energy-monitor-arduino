# Smart Energy Monitoring & Power Factor Correction System
### Arduino UNO Based Prototype

---

## Project Overview

This project implements a **Smart Energy Monitoring and Power Factor Correction System** using an Arduino UNO. It monitors electrical parameters in real time and automatically triggers a correction response when power factor drops below an acceptable threshold — simulating how industrial capacitor bank switching systems work.

> Built as an educational prototype to demonstrate power quality monitoring concepts used in factories and power grids.

---

## What It Does

- Reads AC **voltage** and **current** continuously
- Calculates **Real Power (W)**, **Apparent Power (VA)** and **Power Factor**
- Displays live readings on a **16x2 LCD**
- Triggers a **relay** automatically when Power Factor drops below 0.90
- Relay activates an **LED** representing capacitor bank correction

---

## Components Used

| Component | Purpose |
|---|---|
| Arduino UNO | Main microcontroller |
| ZMPT101B | AC Voltage sensing |
| ACS712 (5A) | AC Current sensing |
| 16x2 LCD Display | Live parameter display |
| 5V Relay Module (1 channel) | Automatic switching |
| LED + 220Ω Resistor | Correction indicator |
| Breadboard + Jumper Wires | Connections |

**Total Cost: Under Rs. 800**

---

## Circuit Connections

### LCD → Arduino (Direct 4-bit mode)
```
LCD Pin 1  (VSS) → GND
LCD Pin 2  (VDD) → 5V
LCD Pin 3  (V0)  → GND
LCD Pin 4  (RS)  → D12
LCD Pin 5  (RW)  → GND
LCD Pin 6  (EN)  → D11
LCD Pin 7-10     → NOT CONNECTED
LCD Pin 11 (D4)  → D5
LCD Pin 12 (D5)  → D4
LCD Pin 13 (D6)  → D3
LCD Pin 14 (D7)  → D2
LCD Pin 15 (A)   → 5V
LCD Pin 16 (K)   → GND
```

### Sensors → Arduino
```
ZMPT101B OUT → A0
ACS712   OUT → A1
```

### Relay + LED → Arduino
```
Relay VCC → 5V
Relay GND → GND
Relay IN  → D7
Relay COM → 5V
Relay NO  → LED long leg (+)
LED short leg (-) → 220Ω resistor → GND
```

---

## How It Works

```
Every 2 seconds:

1. Read voltage (ZMPT101B → A0)
2. Read current (ACS712 → A1)
3. Calculate:
      Real Power (W)     = V × I × PF
      Apparent Power (VA) = V × I
      Power Factor        = Real Power / Apparent Power
4. Display on LCD:
      Row 1 → Voltage and Current
      Row 2 → Watts and Power Factor
5. Check PF:
      PF ≥ 0.90 → Relay OFF → LED OFF (Normal)
      PF < 0.90 → Relay ON  → LED ON  (Correction triggered)
```

---

## LCD Output

```
V:220.0V I:2.50A
W:522.5  PF:0.95   ← LED OFF (good)

V:218.5V I:2.30A
W:426.1  PF:0.85   ← LED ON  (correction triggered!)
```

---

## Arduino Code

```cpp
#include <LiquidCrystal.h>

// LCD pins: RS, EN, D4, D5, D6, D7
LiquidCrystal lcd(12, 11, 5, 4, 3, 2);

#define RELAY_PIN 7

// Simulated readings (replace with EmonLib for real AC measurements)
float voltage[]     = {220.0, 218.5, 221.0, 219.0, 220.5};
float current[]     = {2.50,  2.30,  2.60,  2.40,  2.50};
float powerFactor[] = {0.95,  0.85,  0.72,  0.91,  0.68};

int totalReadings = 5;
int idx = 0;

void setup() {
  Serial.begin(9600);

  pinMode(RELAY_PIN, OUTPUT);
  digitalWrite(RELAY_PIN, HIGH);

  lcd.begin(16, 2);
  lcd.print("Energy Monitor");
  lcd.setCursor(0, 1);
  lcd.print("Starting up...");
  delay(2000);
  lcd.clear();
}

void loop() {
  float V  = voltage[idx];
  float I  = current[idx];
  float PF = powerFactor[idx];
  float W  = V * I * PF;

  Serial.print("V:"); Serial.print(V);
  Serial.print(" I:"); Serial.print(I);
  Serial.print(" W:"); Serial.print(W);
  Serial.print(" PF:"); Serial.println(PF);

  lcd.setCursor(0, 0);
  lcd.print("V:");
  lcd.print(V, 1);
  lcd.print("V I:");
  lcd.print(I, 2);
  lcd.print("A  ");

  lcd.setCursor(0, 1);
  lcd.print("W:");
  lcd.print(W, 1);
  lcd.print(" PF:");
  lcd.print(PF, 2);
  lcd.print("  ");

  if (PF < 0.90) {
    digitalWrite(RELAY_PIN, LOW);   // Relay ON → LED lights up
  } else {
    digitalWrite(RELAY_PIN, HIGH);  // Relay OFF → LED off
  }

  idx = (idx + 1) % totalReadings;
  delay(2000);
}
```

---

## Libraries Required

| Library | How to Install |
|---|---|
| `LiquidCrystal` | Built into Arduino IDE — no install needed |
| `EmonLib` | Arduino IDE → Library Manager → search EmonLib |

---

## Power Factor Explained

| PF Value | Meaning |
|---|---|
| 1.0 | Perfect — all power used efficiently |
| 0.90 - 0.99 | Good — acceptable range |
| 0.85 - 0.89 | Warning — penalties may apply |
| Below 0.85 | Poor — fines applied in India under Electricity Act |

> Under the Indian Electricity Act, consumers are penalised when PF drops below 0.85

---

## Real World vs Our Prototype

| Feature | Real System | Our Prototype |
|---|---|---|
| Current Sensor | SCT-013 (clamp type) | ACS712 5A |
| Correction | Capacitor banks | LED indicator |
| Relay | 4 channel staged | 1 channel |
| Display | I2C LCD | Direct wired LCD |
| Measurements | Live AC | Simulated values |
| Phases | 3 phase | Single phase |

---

## Drawbacks & Limitations

- Simulated readings — not connected to real AC supply
- Single phase only — real factories use 3 phase
- ACS712 affected by temperature and magnetic interference
- No data logging or historical tracking
- No IoT connectivity or remote monitoring
- Capacitor correction not physically implemented

---

## Future Improvements

- Add ESP8266/ESP32 for WiFi and IoT dashboard
- Implement real AC measurements with proper isolation
- Extend to 3 phase monitoring
- Add SD card for data logging
- Integrate actual capacitor bank for real correction
- Build mobile app for remote alerts

---

## Disclaimer

> This is an educational prototype built for demonstration purposes. The AC sensing components (ZMPT101B, ACS712) are not connected to live mains supply in this version. Do not connect to live AC without proper safety measures, isolation, and enclosure.

---

*Smart Energy Monitoring System | Arduino UNO | 2025-26*
