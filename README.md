# Huawei Battery Pack CAN Bus Reverse Engineering

This repository provides an introduction and a practical starting point for reverse-engineering the Huawei battery pack CAN bus, enabling direct access to telemetry data (voltage, temperature, etc.) from the Battery Management System (BMS).
In the default state the battery is switched of and the BMS is not activated. The power input and output ports are disabled. 

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
- **Manual Start**

![Image](https://github.com/user-attachments/assets/d4e13ba2-122d-4ee1-85d8-5504717b3102)


**Step 2: Accessing the BMS Module**

Further disassembly by removing the four screws gives direct access to the **Battery Management Module (BMS)**.

![Image](https://github.com/user-attachments/assets/a4488ba5-4995-45be-88c6-323ed48639e0)

In the battery’s passive state, the output is disconnected internally by the EMS, but you can access terminals directly at the marked bolts if necessary.

---

## 🔓 Activating methods 

To activate the battery fully, two key conditions must be satisfied:

1. **48 V Auxiliary Power** to boot up the BMS electronics.
2. **Valid CAN Communication**, a proprietary Huawei handshake is required.

### Manual Start Pin Test

Manual start is activated by bridging all 4 pins and it results in:

- An initial **red LED** (Faulty), followed by a **blinking green LED** (Standby mode).
- Battery deactivates after ~1 minute, indicating the need for valid CAN communication.
- The manual start is not needed to start the battery but it can give us an understanding abbaut the CAN signals from the batterypack
![Image](https://github.com/user-attachments/assets/1389e682-49af-4a0a-ace1-b5a070874bc1)

---

## 🔌 COM Port Pinout

The battery’s COM port (8-pin) integrates both communication and auxiliary power, above that is the 4-pin +48V supply port, this is needed to independently power the BMS and EMS:

![Image](https://github.com/user-attachments/assets/336c878b-d96f-4de5-958d-55d845350452)


| Pin(s) | Function                           |
|--------|------------------------------------|
| 6      | CAN Low (CAN-L)                    |
| 12     | CAN High (CAN-H)                   |
| 1,2,3  | Auxiliary Negative/Ground (V−)     |
| 7,8,9  |Auxiliary 48 V Positive Input (V+)  |

 Auxiliary 48 V Positive Input (V+
> 1,2 and 7,8 are internal connected  
> **⚠️ Important:** Merely bridging pins (without proper CAN communication) **will NOT fully activate** the battery pack.

---

## 📝 Official Charger Procedure (Huawei DCLT-7050)

The recommended Huawei charger (DCLT-7050) provides both the required CAN signals and auxiliary power:

1. **CAN & Aux Cable**: Connect charger to battery COM port (pins listed above).
2. **DC Power**: Connect charger cables to battery terminals.
3. **Power On**: Switch on the charger’s AC & DC breakers.
4. **CAN Handshake**: Charger initiates the proprietary CAN handshake.
5. **Activation**: BMS internal relays close, starting the charging process.

![Image](https://github.com/user-attachments/assets/604a7c3e-56d1-4b8a-8515-9f9ca18867de)

![Image](https://github.com/user-attachments/assets/fe398753-7c54-40d8-a491-0114af279e25)

*Without CAN communication, the battery remains in standby mode.*

---

## 🔋 Captured CAN Bus Data Analysis from manual start

The CAN bus data was collected using the **Flipper-addon-CANBUS** from ElectronicCats and the **Flipper Zero** device, analyzed using **Savvy CAN**.

### Battery Status Transition

The first captured data shows the battery status transitioning from **Faulty** (terminal voltage: 0V) to **Standby**  (terminal voltage: 17V), and after **105 seconds**, the battery switches off. CAN bus Data was only received in the standby mode.

![Image](https://github.com/user-attachments/assets/9c25a197-8805-40e8-b808-7c7051c35b5e)

### CAN Signal Analysis

- **ID 0x1FFF3A74**: Signal turns on briefly, likely controlling a relay or terminal switch.

![Image](https://github.com/user-attachments/assets/07481ca1-1dc0-4d6e-a06d-134849889299)

- **ID 0x1FFF8407**: Similar pattern to above, but includes additional signals towards the end, likely sending an explicit "off" command to the relay.

![Image](https://github.com/user-attachments/assets/e6efcbf9-18b4-4bc8-928f-5f1d313de9a6)

- **ID 0x1FFFF841**: Likely represents a periodic "heartbeat" signal indicating battery activity.

![Image](https://github.com/user-attachments/assets/7132c4ac-be05-44ea-90cb-56bcde1410e6)

---

## 🚦 Activating the Battery Pack

To activate the battery fully, the following condition must be satisfied:

- **48 V DC Voltage Supply** to boot up the BMS electronics. Connect this supply to:
  - **Positive (V+)** to Pin 7, 8, 9
  - **Negative/Ground (V−)** to Pin 1, 2, 3

The power supply should provide **at least 300W**, as the internal fans require **4.5A at 48V**.

> **Note:** The manual start pin connection is no longer necessary.

---
## 🌐 Community Insights

- Huawei batteries rely on proprietary CAN protocols derived from Modbus/RS485.
- Direct pin bridging or standard CAN controllers usually fail to activate the battery.
- Community attempts show success requires reverse-engineering official CAN frames.

---

## 🕵️ Next Steps for Reverse Engineering

- Capture traffic using a CAN sniffer between the official charger and battery.
- Decode the proprietary CAN messages.
- Independently replicate CAN communication to activate battery and read telemetry data.

---
