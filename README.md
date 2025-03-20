# Huawei Battery Pack CAN Bus Reverse Engineering

This repository provides an introduction and a practical starting point for reverse-engineering the Huawei battery pack CAN bus, enabling direct access to telemetry data (voltage, temperature, etc.) from the Battery Management System (BMS).

---

## 🛠️ Battery Overview

This battery pack comprises **18 lithium-ion cells** connected in series. It is controlled via two integrated systems:

- **Battery Management System (BMS)**
- **Energy Management System (EMS)**

Our mission is to extract telemetry data directly from the BMS through the CAN bus.

---

## 🔎 Hardware Inspection

**Step 1: Opening the Battery Pack**

Open the battery covers by pressing the marked points:
![Image](https://github.com/user-attachments/assets/75184c34-0d44-4c2b-960b-b86b1a61a63a)

Underneath the cover, you’ll find connections for:
![Image](https://github.com/user-attachments/assets/28ae33de-f1d8-4df5-8389-2581ef219147)
- **COM2.0 Communication (CAN Bus)**
- **48 V DC Auxiliary Supply**
- **Manual Start Pinout**

![Image](https://github.com/user-attachments/assets/d4e13ba2-122d-4ee1-85d8-5504717b3102)


**Step 2: Accessing the BMS Module**

Further disassembly by removing the four screws gives direct access to the **Battery Management Module (BMS)**.

![Image](https://github.com/user-attachments/assets/a4488ba5-4995-45be-88c6-323ed48639e0)

In the battery’s passive state, the output is disconnected internally by the EMS, but you can access terminals directly at the marked bolts if necessary.

---

## 🚦 Activating the Battery Pack

To activate the battery fully, two key conditions must be satisfied:

1. **48 V Auxiliary Power** to boot up the BMS electronics.
2. **Valid CAN Communication**, a proprietary Huawei handshake is required.

### Manual Start Pin Test

Connecting manual-start pins alone results in:

- An initial **orange LED**, followed by a **blinking green LED**.
- Battery deactivates after ~1 minute, indicating the need for valid CAN communication.

---

## 🔌 COM Port (RJ45) Pinout

The battery’s COM port (8-pin RJ45 style) integrates both communication and auxiliary power:

![Image](https://github.com/user-attachments/assets/336c878b-d96f-4de5-958d-55d845350452)


| Pin(s) | Function                           |
|--------|------------------------------------|
| 6      | CAN Low (CAN-L)                    |
| 12     | CAN High (CAN-H)                   |
| 1,2,3  | Auxiliary 48 V Positive Input (V+) |
| 7,8,9  | Auxiliary Negative/Ground (V−)     |

> 1,2 and 7,8 are internal connected  
> **⚠️ Important:** Merely bridging pins (without proper CAN communication) **will NOT fully activate** the battery pack.

---

## 🔋 Official Charger Procedure (Huawei DCLT-7050)

The recommended Huawei charger (DCLT-7050) provides both the required CAN signals and auxiliary power:

1. **CAN & Aux Cable**: Connect charger to battery COM port (pins listed above).
2. **DC Power**: Connect charger cables to battery terminals.
3. **Power On**: Switch on the charger’s AC & DC breakers.
4. **CAN Handshake**: Charger initiates the proprietary CAN handshake.
5. **Activation**: BMS internal relays close, starting the charging process.

![Image](https://github.com/user-attachments/assets/604a7c3e-56d1-4b8a-8515-9f9ca18867de)

![Image](https://github.com/user-attachments/assets/fe398753-7c54-40d8-a491-0114af279e25)

*Without CAN communication, the battery remains idle.*

---

## 🌐 Community Insights

- Huawei batteries rely on proprietary CAN protocols derived from Modbus/RS485.
- Direct pin bridging or standard CAN controllers usually fail to activate the battery.
- Community attempts show success requires reverse-engineering official CAN frames.

---

## 🕵️ Next Steps for Reverse Engineering

- Use a CAN sniffer to capture traffic between the official DCLT-7050 charger and battery.
- Analyze and decode the proprietary CAN messages.
- Replicate CAN communication independently to activate battery and read telemetry data.

---
