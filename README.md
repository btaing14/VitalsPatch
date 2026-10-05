# VitalsPatch

**A wearable multi-patient health monitoring system**
*EE 475 Embedded Systems Capstone, University of Washington, Spring 2026 · 6-person team*

> 📁 Team repository: [ArnavMohan18/VitalsPatch](https://github.com/ArnavMohan18/VitalsPatch) · my original work is on the [`bobby` branch](https://github.com/ArnavMohan18/VitalsPatch/tree/bobby)

VitalsPatch lets one caregiver monitor several patients at once. Each sensor node tracks **heart rate, SpO2, temperature and motion**, detects **falls and out-of-range vitals**, and sends readings wirelessly to a central dashboard that shows prioritized alerts for each patient.

## Demo

https://github.com/user-attachments/assets/9d52a5d2-b57e-4a6b-9b01-fe2860853154

*The dashboard tracks two patients live. Alerts escalate from normal (green) to temperature spike (orange) to fall detected (red).*

## System architecture

```
 Patient node (×2)                            Hub                          Monitoring station
┌──────────────────────────────┐   BLE    ┌────────────────────────┐ UART ┌─────────────────────────┐
│ ESP32-C3                     │ ───────► │ HM-19 #1 → USART2      │ ───► │ Raspberry Pi 4          │
│  MAX30102 (HR, SpO2, temp)   │          │ HM-19 #2 → USART3      │      │ two-patient dashboard,  │
│  MPU-6500 (fall detection)   │          │ STM32F4                │      │ alerts, trend graphs    │
│  alert logic + packetizing   │          └────────────────────────┘      └─────────────────────────┘
└──────────────────────────────┘
```

Each ESP32-C3 acts as a **BLE central**. It finds its assigned HM-19 module by MAC address and writes packets to the module's `FFE0`/`FFE1` UART-transparent service. Each HM-19 passes the data to its own STM32 UART, which gives every patient a separate channel.

## My role: wireless communication and system integration

I built and tested the BLE link between the patient nodes and the STM32 hub, including the dual-channel setup that two-patient monitoring depends on. That code is in [`ble-link-prototype/`](ble-link-prototype):

| File | What it does |
|---|---|
| `esp32/ESP32WRITE`, `ESP32AWRITE`, `ESP32BWRITE` | NimBLE central: scans for a specific HM-19 by MAC, connects, finds the `FFE0` service and `FFE1` characteristic, and writes data, with error handling at each step. The A and B versions target two different HM-19s, one per patient. |
| `esp32/ESP32Receive` | Subscribes to `FFE1` notifications to receive data sent back from the STM32 |
| `stm32/main.c` | Configures an HM-19 over UART with AT commands and checks for its `OK` reply |
| `stm32/main2.c` | Runs **two HM-19 modules at once** on USART2 and USART3 (renamed `HM19A` / `HM19B`) and drives a separate LED for each channel |

**Result:** both BLE channels ran independently with no cross-talk (verified with per-channel LED tests). This was the Phase 2 two-patient milestone. The same connection method (central → HM-19 by MAC → `FFE0`/`FFE1`) is what the final sensor-node firmware uses.

## Team results (from the final report)

| Test | Result |
|---|---|
| Heart rate vs. Apple Watch / manual count | within ±4 BPM |
| SpO2 vs. reference oximeter | within 1% |
| Temperature (after calibration) vs. thermometer | within 0.8 °F |
| Fall detection | 10/10 simulated falls detected, 0/10 false alarms from normal motion |
| Multi-patient | 2 patients monitored at once with correct data separation |

## Design evolution

1. **Phase 1:** a single sensor node sends packets over BLE to the Raspberry Pi.
2. **Phase 2:** two nodes connect to an STM32F4 through two HM-19 modules (my BLE work), and the STM32 runs alert logic and fall detection.
3. **Phase 3:** alert logic moves onto each ESP32-C3 for scalability, after an STM32 hardware failure. The STM32 becomes a communication bridge.

## Repository contents

```
final-system/         Team's final submitted code
  ESP32-C3/           Sensor node: acquisition, alert logic, BLE transmission
  STM32F4/            STM32CubeIDE project for the hub
  RaspberryPi/        two_patient_dashboard.py
ble-link-prototype/   My BLE communication code (see above)
docs/                 Final report and presentation slides
```

## Notes on this snapshot

- `final-system/` is the team's submission as-is. Its STM32 `main.c` contains the generated peripheral setup (USART2/3, etc.) but not the forwarding loop, and the dashboard's serial input is commented out, so it doesn't reproduce the full end-to-end demo by itself.
- The wearable patch packaging was not finished. Testing used breadboarded sensor nodes.

## Tech

ESP32-C3 · STM32F4 (HAL, STM32CubeIDE) · HM-19 BLE · NimBLE · UART · I²C · Raspberry Pi 4 · Python · C/C++

## Team

Eeshani Shilamkar · Arnav Mohan · Maya Desai · **Bobby Taing** · Anushka Misra · Nate Snyder
