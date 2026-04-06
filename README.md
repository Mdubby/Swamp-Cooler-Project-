# Embedded Environmental Control System — ATmega2560
### CPE 301: Embedded Systems Design | University of Nevada, Reno | Spring 2022

A fully functional embedded swamp cooler controller built on the ATmega2560 microcontroller, implementing a 4-state finite state machine (FSM) with real-time sensor feedback, mixed-voltage subsystem management, and multi-actuator control.

---

## System Overview

The firmware manages a swamp cooler across four discrete operating states — **Disabled**, **Idle**, **Running**, and **Error** — with autonomous state transitions driven by real-time temperature, humidity, and water level sensor readings. The system was designed with hardware isolation between the 3.3V water level sensor rail and the 5V main system rail to prevent signal interference with the LCD display and temperature sensor.

---

## Hardware Components

| Component | Function |
|-----------|----------|
| ATmega2560 (Arduino Mega) | Main microcontroller |
| DHT11 | Temperature & humidity sensing |
| Water level sensor | Analog water level detection (3.3V isolated rail) |
| DS1307 RTC | Real-time clock for state transition logging |
| DC fan motor + H-bridge driver | Fan actuation in Running state |
| Stepper motor + ULN2003 driver | Vent position control (all states except Error) |
| 16x2 LCD display | Live temperature/humidity HMI readout |
| RGB LEDs | State indicator (Yellow=Disabled, Green=Idle, Blue=Running, Red=Error) |
| Push buttons | Start (Disabled→Idle), stepper CW/CCW/Stop control |

---

## State Machine Architecture

```
DISABLED ──[Start button]──► IDLE
   ▲                          │
   │                    [Temp > 21°C]
   │                          │
   │                          ▼
   │                       RUNNING
   │                          │
   └──[Reset button]── ERROR ◄─┘
                         ▲
               [Water level < threshold]
               (from Idle or Running)
```

### State Behaviors

**DISABLED** — System startup state. Fan off. Yellow LED on. Transitions to Idle on button press.

**IDLE** — Fan off. Green LED on. Displays live temperature and humidity on LCD. Stepper vent control active. Monitors temperature and water level continuously. Transitions to Running if temp > 21°C, to Error if water level drops below threshold.

**RUNNING** — Fan motor active. Blue LED on. Displays live temp/humidity on LCD. Stepper vent control active. Transitions back to Idle if temp drops below threshold, to Error if water level drops below threshold.

**ERROR** — Fan motor stopped. Red LED on. LCD displays "Water Level Low!". Stepper vent control active. Transitions back to Idle only when water level is restored above threshold.

---

## Key Design Decisions

- **Mixed-voltage isolation**: The water level sensor operates on a separate 3.3V rail while the rest of the system draws 5V. This prevents sensor current draw from introducing noise into the LCD and DHT11 readings.
- **H-bridge motor driver**: The DC fan draws ~2.4W average — well beyond ATmega2560 GPIO current limits — requiring a separate H-bridge driver circuit for safe switching.
- **ULN2003 stepper driver**: Half-step pole array sequencing via Darlington array for smooth vent position control with minimal microcontroller pin current draw.
- **RTC logging**: DS1307 real-time clock enables timestamped serial logging of all state transitions for debugging and audit purposes.

---

## Libraries Required

- `DHT` — Adafruit DHT sensor library
- `LiquidCrystal` — Arduino built-in LCD library
- `RTClib` — Adafruit RTClib for DS1307
- `ezButton` — Arduino button debounce library
- `Wire` — Arduino built-in I2C library

---

## Pin Reference

| Pin | Assignment |
|-----|------------|
| 31 | DHT11 data |
| A0 | Water level sensor (analog) |
| 2,3,4,5,6,7 | LCD (RS, EN, D4-D7) |
| 39,40,41 | DC motor (ENABLE, DIRA, DIRB) |
| 51,52,53 | LEDs (Blue, Green, Red) |
| 22 | Start button |
| 8,9,10,11 | Stepper motor (ULN2003 IN1-IN4) |
| 36,34,13 | Stepper control buttons (CW, Stop, CCW) |

---

## Project Documentation

Full project overview including system schematic (EasyEDA), Gantt chart, and hardware photos available in the `/docs` folder.

- **Schematic**: Drawn by Matthew Wallace in EasyEDA, dated 2022-05-01
- **Operating temperature threshold**: 21°C
- **Water level thresholds**: Error entry < 120 (analog), Error exit > 200 (analog)

---

## Author

**Matthew Wallace** — Biomedical Engineering, University of Nevada, Reno (B.S. 2023)  
Pursuing M.S. Electrical Engineering, UNR, Fall 2026  
linkedin.com/in/mattwallace-bme

**Monte Howell** — Biomedical Engineering, University of Nevada, Reno (B.S. 2023)
