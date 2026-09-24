# Smart-Blinds-Automation-System
An ESP32-based automated smart blinds controller simulated in Cirkit Designer using an LDR light sensor, A4988 driver, NEMA 17 stepper motor, and AccelStepper library.
>  **Note:** This project was designed, wired, and simulated online using **Cirkit Designer**.

---

## What is this project?

Basically, it’s an automated window blind setup built with an **ESP32**. It uses an **LDR light sensor** to measure ambient daylight and an **A4988 driver** to control a **NEMA 17 stepper motor**. 

When sunlight hits a specific brightness threshold, the ESP32 smoothly spins the motor to adjust the blinds—no human effort needed!

---

## Why build it?

Manually opening and closing window blinds throughout the day gets repetitive and is easy to forget. Unmanaged sunlight creates annoying glare on screens and heats up rooms, driving up air conditioning costs. This little system automates daylight management to keep rooms comfortable.

---

## Hardware & Pinout 

| Component | ESP32 Pin | Function |
| :--- | :--- | :--- |
| **LDR Sensor (Analog)** | GPIO 34 (ADC) | Measures ambient light intensity |
| **LDR Sensor (Digital)** | GPIO 35 | Threshold digital trigger |
| **A4988 STEP Pin** | GPIO 13 | Sends step pulses to motor |
| **A4988 DIR Pin** | GPIO 12 | Sets motor rotation direction |
| **NEMA 17 Stepper** | Connected to A4988 | Actuator for window blinds |

---

## How It Works

1. **Reading Sunlight:** The LDR sensor samples ambient light intensity and sends the analog reading to GPIO 34 (12-bit ADC), converting light levels to a value between 0 (darkness) and 4095 (bright light).
2. **Checking Threshold:** The ESP32 compares this reading against a calibrated cutoff value (`LIGHT = 2000`).
3. **Motor Control:**
   * **Sunlight Detected (> 2000):** The system triggers the A4988 driver to step the NEMA 17 motor forward to position `5000` to adjust the blinds.
   * **Low Light / Darkness (< 2000):** The motor automatically returns back to position `0` to close the blinds.
4. **Smooth Movement:** The code uses the non-blocking `AccelStepper` library (speed = 500 steps/sec, acceleration = 250 steps/sec²) so motor movements are smooth and continuous.



https://github.com/user-attachments/assets/6e3459bd-9762-4844-9bff-532428761903



---

## Arduino / C++ Code 
```cpp
#include <AccelStepper.h>

int LDRD = 35;
int LDRA = 5;
int SDir = 12;
int SStep = 13;

 AccelStepper Motor(1, SDir, SStep);

 int LIGHT = 2000;

void setup() {
    Serial.begin(115200);
    Serial.println("Starting Smart Blinds System");

    pinMode(LDRD, INPUT);
    pinMode(LDRA, INPUT);
    pinMode(SStep, OUTPUT);
    pinMode(SDir, OUTPUT);

    Motor.setMaxSpeed(500);
    Motor.setAcceleration(250);
    Motor.setCurrentPosition(0);
    digitalWrite(SDir, HIGH);
    analogReadResolution(12);

}

void loop() {
  Serial.print("Light Status: ");
  int Dresult = digitalRead(LDRD);
  Serial.print(Dresult);

  Serial.print(" | Sunlight Value: ");
  float Aresult = analogRead(LDRA);
  Serial.print(Aresult);

  if (Aresult > LIGHT) {
    Serial.println("Sunlight detected -> Moving Blinds...");
     digitalWrite(SStep, HIGH);
    delayMicroseconds(100); 
    digitalWrite(SStep, LOW);
    delayMicroseconds(100);
    Motor.moveTo(5000);
  }
  else {
    Serial.println("No sunlight detected -> Motor Stopped.");
    Motor.moveTo(0);
 }


Motor.runToPosition();

delay(100);

}




