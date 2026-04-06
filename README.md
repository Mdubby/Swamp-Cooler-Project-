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

<svg width="680" viewBox="0 0 680 580" xmlns="http://www.w3.org/2000/svg" font-family="Arial, sans-serif">
  <defs>
    <marker id="arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
      <path d="M2 1L8 5L2 9" fill="none" stroke="context-stroke" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
    </marker>
  </defs>

  <!-- Background -->
  <rect width="680" height="580" fill="#ffffff"/>

  <!-- ── STATE NODES ─────────────────────────────────────────────── -->

  <!-- DISABLED -->
  <rect x="60" y="60" width="140" height="56" rx="8" fill="#D3D1C7" stroke="#5F5E5A" stroke-width="0.5"/>
  <text x="130" y="82" text-anchor="middle" dominant-baseline="central" font-size="14" font-weight="500" fill="#2C2C2A">Disabled</text>
  <text x="130" y="102" text-anchor="middle" dominant-baseline="central" font-size="12" fill="#444441">Fan off · Yellow LED</text>

  <!-- IDLE -->
  <rect x="270" y="60" width="140" height="56" rx="8" fill="#C0DD97" stroke="#3B6D11" stroke-width="0.5"/>
  <text x="340" y="82" text-anchor="middle" dominant-baseline="central" font-size="14" font-weight="500" fill="#173404">Idle</text>
  <text x="340" y="102" text-anchor="middle" dominant-baseline="central" font-size="12" fill="#27500A">Fan off · Green LED</text>

  <!-- RUNNING -->
  <rect x="480" y="60" width="140" height="56" rx="8" fill="#B5D4F4" stroke="#185FA5" stroke-width="0.5"/>
  <text x="550" y="82" text-anchor="middle" dominant-baseline="central" font-size="14" font-weight="500" fill="#042C53">Running</text>
  <text x="550" y="102" text-anchor="middle" dominant-baseline="central" font-size="12" fill="#0C447C">Fan on · Blue LED</text>

  <!-- ERROR -->
  <rect x="270" y="380" width="140" height="56" rx="8" fill="#F7C1C1" stroke="#A32D2D" stroke-width="0.5"/>
  <text x="340" y="402" text-anchor="middle" dominant-baseline="central" font-size="14" font-weight="500" fill="#501313">Error</text>
  <text x="340" y="422" text-anchor="middle" dominant-baseline="central" font-size="12" fill="#791F1F">Fan off · Red LED</text>

  <!-- ── TRANSITIONS ──────────────────────────────────────────────── -->

  <!-- Disabled → Idle -->
  <path d="M200 88 L268 88" fill="none" stroke="#888780" stroke-width="1" marker-end="url(#arrow)"/>
  <text x="234" y="80" text-anchor="middle" font-size="12" fill="#5F5E5A">Start</text>

  <!-- Idle → Disabled (Stop) -->
  <path d="M300 60 Q230 20 190 60" fill="none" stroke="#888780" stroke-width="1" marker-end="url(#arrow)"/>
  <text x="238" y="24" text-anchor="middle" font-size="12" fill="#5F5E5A">Stop</text>

  <!-- Idle → Running -->
  <path d="M410 78 L478 78" fill="none" stroke="#888780" stroke-width="1" marker-end="url(#arrow)"/>
  <text x="444" y="70" text-anchor="middle" font-size="12" fill="#5F5E5A">Temp &gt; 21°C</text>

  <!-- Running → Idle -->
  <path d="M480 100 L412 100" fill="none" stroke="#888780" stroke-width="1" marker-end="url(#arrow)"/>
  <text x="444" y="116" text-anchor="middle" font-size="12" fill="#5F5E5A">Temp ≤ 21°C</text>

  <!-- Idle → Error -->
  <path d="M340 116 L340 378" fill="none" stroke="#888780" stroke-width="1" marker-end="url(#arrow)"/>
  <text x="318" y="250" text-anchor="middle" font-size="12" fill="#5F5E5A">Water</text>
  <text x="318" y="265" text-anchor="middle" font-size="12" fill="#5F5E5A">level low</text>

  <!-- Running → Error -->
  <path d="M550 116 L550 320 L412 400" fill="none" stroke="#888780" stroke-width="1" marker-end="url(#arrow)"/>
  <text x="586" y="220" text-anchor="middle" font-size="12" fill="#5F5E5A">Water</text>
  <text x="586" y="235" text-anchor="middle" font-size="12" fill="#5F5E5A">level low</text>

  <!-- Error → Idle (water restored) -->
  <path d="M370 380 L370 118" fill="none" stroke="#888780" stroke-width="1" marker-end="url(#arrow)"/>
  <text x="394" y="250" text-anchor="middle" font-size="12" fill="#5F5E5A">Water</text>
  <text x="394" y="265" text-anchor="middle" font-size="12" fill="#5F5E5A">restored</text>

  <!-- Error → Disabled (Stop) -->
  <path d="M270 408 L130 408 L130 118" fill="none" stroke="#888780" stroke-width="1" marker-end="url(#arrow)"/>
  <text x="78" y="270" text-anchor="middle" font-size="12" fill="#5F5E5A">Stop</text>

  <!-- Running → Disabled (Stop, arc across top) -->
  <path d="M540 60 Q340 -10 170 60" fill="none" stroke="#888780" stroke-width="1" marker-end="url(#arrow)"/>
  <text x="340" y="18" text-anchor="middle" font-size="12" fill="#5F5E5A">Stop</text>

  <!-- ── ANNOTATION NOTES ─────────────────────────────────────────── -->

  <!-- LCD note -->
  <rect x="230" y="160" width="220" height="44" rx="6" fill="#9FE1CB" stroke="#0F6E56" stroke-width="0.5"/>
  <text x="340" y="178" text-anchor="middle" dominant-baseline="central" font-size="12" fill="#04342C">LCD displays temp &amp; humidity</text>
  <text x="340" y="196" text-anchor="middle" dominant-baseline="central" font-size="12" fill="#04342C">in Idle and Running states</text>
  <path d="M340 160 L340 118" fill="none" stroke="#888780" stroke-width="0.5" stroke-dasharray="4 3"/>

  <!-- Vent note -->
  <rect x="60" y="300" width="178" height="44" rx="6" fill="#FAC775" stroke="#854F0B" stroke-width="0.5"/>
  <text x="149" y="318" text-anchor="middle" dominant-baseline="central" font-size="12" fill="#412402">Vent adjustable in all</text>
  <text x="149" y="336" text-anchor="middle" dominant-baseline="central" font-size="12" fill="#412402">states except Error</text>

  <!-- Error message note -->
  <rect x="420" y="460" width="210" height="44" rx="6" fill="#F7C1C1" stroke="#A32D2D" stroke-width="0.5"/>
  <text x="525" y="478" text-anchor="middle" dominant-baseline="central" font-size="12" fill="#501313">LCD: "Water level is too low"</text>
  <text x="525" y="496" text-anchor="middle" dominant-baseline="central" font-size="12" fill="#501313">Red LED active</text>
  <path d="M410 408 L448 462" fill="none" stroke="#888780" stroke-width="0.5" stroke-dasharray="4 3"/>

  <!-- Title -->
  <text x="340" y="555" text-anchor="middle" font-size="13" fill="#888780">Swamp Cooler FSM — CPE 301, UNR Spring 2022 · Matthew Wallace</text>
</svg>![swamp_cooler_state_diagram](https://github.com/user-attachments/assets/1573243f-73d2-4abe-adbb-3e93dc938276)
```

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
