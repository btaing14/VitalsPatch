# VitalsPatch

**A wearable multi-patient health monitoring system**
*EE 475 Embedded Systems Capstone, University of Washington · 6-person team*

> 📁 **Team repository:** [ArnavMohan18/VitalsPatch](https://github.com/ArnavMohan18/VitalsPatch)
> This page summarizes the project and my part in it.

## Overview

VitalsPatch is a wearable that lets one caregiver monitor several patients at once. Each patch continuously tracks **heart rate, SpO2, body temperature and motion**, and flags **falls and out-of-range vitals** in real time on a shared dashboard.

```
Sensors (MAX30102, MPU-6050, temp)  →  ESP32-C3  →  BLE  →  STM32F4  →  UART  →  Raspberry Pi 4
     one patch per patient          acquisition           alert logic           dashboard & logs
```

## My role: alert logic and system testing

- **Alert logic:** the rules that turn incoming vitals into prioritized alert codes:

  | Code | Event |
  |---|---|
  | 0 | Normal |
  | 1 / 2 | Heart rate above / below threshold |
  | 3 / 4 | Temperature spike / drop |
  | 5 | Low SpO2 |
  | 6 | Fall detected |

- **System testing:** a 9-case alert test suite covering the normal baseline, each alert on its own, and combined multi-alert scenarios. All cases pass.

## Validation

- **Sensors:** heart rate and SpO2 cross-checked against an Apple Watch; fall detection checked against standing, walking and simulated falls; temperature checked against an external thermometer; motion-artifact rejection tested with deliberate wrist movement.
- **Dashboard:** two patients tracked at the same time, with separate data, prioritized alerts and per-patient logs.

## Status

- ✅ **Phase 1:** each module (sensors, STM32 alert logic, dashboard) working on its own
- ✅ **Phase 2:** two sensor patches transmitting with patient IDs to a two-patient dashboard
- 🔄 Finishing the STM32 → Raspberry Pi serial link for full end-to-end operation

## Tech

STM32F4 · ESP32-C3 · Raspberry Pi 4 · MAX30102 · MPU-6050 · BLE · UART

## Team

Eeshani Shilamkar (UI & project management) · Arnav Mohan (sensor integration & wireless) · Maya Desai (STM32 data processing) · Nate Snyder (calibration & wearable assembly) · Anushka Misra (hardware integration & acquisition firmware) · **Bobby Taing (alert logic & system testing)**
