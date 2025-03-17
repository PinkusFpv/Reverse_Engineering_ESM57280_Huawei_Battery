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

![Battery Cover Opening](![Image](https://github.com/user-attachments/assets/c4463ebe-7845-4437-ab05-45dd0d00cbdd))

Underneath the cover, you’ll find connections for:

- **COM2.0 Communication (CAN Bus)**
- **48 V DC Auxiliary Supply**
- **Manual Start Pinout**

![Connector Location](![Image](https://github.com/user-attachments/assets/d6136f5a-55da-4a5e-ae2f-1638dc0b6d44))

**Step 2: Accessing the BMS Module**

Further disassembly by removing the four screws gives direct access to the **Battery Management Module (BMS)**.

![BMS Module Access](![Image](https://github.com/user-attachments/assets/5648cc7e-b0e8-40d1-9567-684833d76071))

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

| Pin(s) | Function                           |
|--------|------------------------------------|
| 1,4    | CAN High (CAN-H, labeled “A”)      |
| 2,5    | CAN Low (CAN-L, labeled “B”)       |
| 3      | Auxiliary 48 V Positive Input (V+) |
| 8      | Auxiliary Negative/Ground (V−)     |
| 6,7    | Reserved (not used)                |

> **⚠️ Important:** Merely bridging pins (without proper CAN communication) **will NOT fully activate** the battery pack.

---

## 🔋 Official Charger Procedure (Huawei DCLT-7050)

The recommended Huawei charger (DCLT-7050) provides both the required CAN signals and auxiliary power:

1. **CAN & Aux Cable**: Connect charger to battery COM port (pins listed above).
2. **DC Power**: Connect charger cables to battery terminals.
3. **Power On**: Switch on the charger’s AC & DC breakers.
4. **CAN Handshake**: Charger initiates the proprietary CAN handshake.
5. **Activation**: BMS internal relays close, starting the charging process.

![Charger Connection Example](![Image](https://github.com/user-attachments/assets/d8dc05da-78a1-4396-a3a0-7a0f75984e43))

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
