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

# 🔋 Captured BMS Data Analysis with FYSETC UCAN Board

The data from the Battery Management System (BMS) was captured using a **FYSETC UCAN Board** based on an STM32F072 USB-to-CAN adapter, enabling direct sending and receiving through **Savvy CAN**.

## 48V Power Supply and Setup

To activate the BMS and capture its CAN data, a stable **48V DC power supply** was provided with the following connections:

- **Positive (V+)** connected to Pins 7, 8, and 9 via a breakout board.
- **Negative/Ground (V−)** connected to Pins 1, 2, and 3 via the breakout board.

A DC/DC Buck converter from **12V DC to 48V DC** was used. Alternatively, a laboratory power supply can be used. Typically, a power supply capable of delivering at least **300W** (minimum **4.5A at 48V**) is recommended to power internal cooling fans. However, if the fan is disconnected, the 300W supply is unnecessary.

> **Important:** Keep the internal fan operational during battery charging or discharging to prevent overheating.

### Connection Example

![Image](https://github.com/user-attachments/assets/84f01ebd-95b1-4ed9-8bb1-cf3c7bcd0645)

---

## Detailed CAN Signal Analysis

### Timestamps for the CAN IDs:

- **0x13106226**: Cell Data
- **0x1FFF6200**: Battery Pack Data
- **0x13106224**: Warning Messages

Initially, the BMS sends 2 frames with ID 0x1FFF6200, showing Battery Pack Data. Subsequently, it continuously transmits frames with ID 0x1FFF6200, which do not stop even if the battery enters faulty mode. At the onset of faulty mode, the BMS sends 5 frames of ID 0x13106224, symbolizing the warning messages.

![Image](https://github.com/user-attachments/assets/bca8c600-7239-4d47-af03-9c4346b0c272)


### Example Data from CAN Logs

| Timestamp | ID       | Data Bytes (D1-D8)                  | Meaning                              |
|-----------|----------|-------------------------------------|--------------------------------------|
|13858923   |1FFF6200  |10 FF 31 FB 4A 07 FA 00              |Battery Pack operational data/status |
|15052093   |1FFF6200  |10 FF 31 FB 4A 07 FA 01              |Operational status update            |
|16315285   |13106226  |A2 00 00 03 31 00 E5 B4              |Cell voltage: 3V, Temp: 24.5°C       |
|20120471   |13106226  |C3 00 FA 07 4A FB 00 01              |Cell operational detailed data       |
|24777279   |13106226  |F0 00 00 01 00 00 00 00              |Cell status update                   |

These examples illustrate typical patterns captured during BMS operations, demonstrating voltage, temperature, and pack status information.

---

## 🔍 Bitwise Analysis of CAN ID `13106226`, `13106224` and `1FFF6200`

### 📌 CAN ID `13106226`

Let's take a closer look at the first three captured signals for the CAN ID `13106226`:

| ID       | D1 | D2 | D3 | D4 | D5 | D6 | D7 | D8 |
|----------|----|----|----|----|----|----|----|----|
|13106226  | C3 | 00 | FA | 07 | 4A | FB | 00 | 01 |
|13106226  | F0 | 00 | 00 | 01 | 00 | 00 | 00 | 00 |
|13106226  | C3 | 00 | FA | 07 | 4A | FB | 00 | 02 |

Converting these hexadecimal values to decimal provides further clarity:

| ID       | D1 | D2 | D3  | D4 | D5 | D6  | D7 | D8 |
|----------|----|----|-----|----|----|-----|----|----|
|13106226  |195 |  0 | 250 |  7 | 74 | 251 |  0 |  1 |
|13106226  |240 |  0 |   0 |  1 |  0 |   0 |  0 |  0 |
|13106226  |195 |  0 | 250 |  7 | 74 | 251 |  0 |  2 |

Analyzing these converted values reveals:

- **Byte D8** acts as an incremental counter, clearly increasing (`1 → 2`) with each subsequent data packet, likely indicating packet sequencing or cell addressing.
- **Bytes D1 and D3** might represent operational flags or status codes (195 and 250 appearing consistently may indicate normal operation conditions or status messages).
- **Byte D4** and **D5** potentially contain battery or cell-specific metrics such as voltage or temperature data (e.g., `07` → 7 could represent a stable cell voltage reading around 3.5V or similar, `4A` → 74 possibly temperature-related).

### Example Interpretation:
- **D1 (195, 240)**: Likely status or control flags.
- **D3 (250, 0)**: May indicate fault/status conditions.
- **D4 and D5 (7 and 74)**: Voltage/temperature indicators (e.g., 74 might translate to a temperature like 37°C if scaled by 0.5°C increments).
- **D8 (1, 2)**: Incremental identifier for cell indexing or message sequence.

This structured approach can assist in accurately decoding the BMS communication protocol. Further systematic recording and comparison of more messages are recommended for a comprehensive analysis.

---

### 📌 CAN ID `13106224`

Analyzing captured signals for CAN ID `13106224`:

| ID       | D1 | D2 | D3 | D4 | D5 | D6 | D7 | D8 |
|----------|----|----|----|----|----|----|----|----|
|13106224  | C0 | 00 | 00 | 00 | 00 | 00 | 00 | 00 |
|13106224  | C1 | 00 | 00 | 00 | 00 | 00 | 00 | 00 |
|13106224  | C2 | 00 | 00 | 00 | 00 | 00 | 00 | 00 |

Hexadecimal to decimal conversion:

| ID       | D1 | D2 | D3 | D4 | D5 | D6 | D7 | D8 |
|----------|----|----|----|----|----|----|----|----|
|13106224  |192 |  0 |  0 |  0 |  0 |  0 |  0 |  0 |
|13106224  |193 |  0 |  0 |  0 |  0 |  0 |  0 |  0 |
|13106224  |194 |  0 |  0 |  0 |  0 |  0 |  0 |  0 |

Observations:

- **Byte D1** shows a sequential increment (`192 → 193 → 194`), acting as a status indicator or message sequence.
- **D2 to D8** consistently hold `0`, potentially indicating no current warnings or faults detected during these specific data captures.

### Interpretation:
- **D1 (192–194)**: Likely represents warning states or sequential event codes.
- **D2–D8**: Possibly unused or reserved for future states or detailed warning data.

---

### 📌 CAN ID `1FFF6200`

Captured signals for CAN ID `1FFF6200`:

| ID       | D1 | D2 | D3 | D4 | D5 | D6 | D7 | D8 |
|----------|----|----|----|----|----|----|----|----|
|1FFF6200  | 10 | FF | 31 | FB | 4A | 07 | FA | 00 |
|1FFF6200  | 10 | FF | 31 | FB | 4A | 07 | FA | 01 |
|1FFF6200  | 11 | 31 | 31 | 00 | 00 | 00 | 00 | 00 |

Hexadecimal to decimal conversion:

| ID       | D1 | D2  | D3 | D4  | D5 | D6 | D7 | D8 |
|----------|----|-----|----|-----|----|----|----|----|
|1FFF6200  | 16 | 255 | 49 | 251 | 74 |  7 |250 |  0 |
|1FFF6200  | 16 | 255 | 49 | 251 | 74 |  7 |250 |  1 |
|1FFF6200  | 17 |  49 | 49 |   0 |  0 |  0 |  0 |  0 |

Observations:

- **Byte D1** (`16 → 17`) changes occasionally, possibly denoting different battery states or modes.
- **Byte D8** acts as an incremental counter or an indexing marker (e.g., `0 → 1`), useful for identifying sequential transmissions or packet indexing.
- **Bytes D2-D7** hold values indicative of battery pack operational parameters like temperature, voltage, or system flags:
  - `255` (FF) might represent maximum scale or a special operational condition.
  - `49` (31) consistently appears, possibly referencing a common voltage or temperature value.
  - `251`, `74`, and `250` could be related to other battery metrics or operational thresholds.

### Interpretation:
- **D1 (16–17)**: Battery operational mode or state identifier.
- **D2–D7**: Likely battery operational data such as temperature thresholds, voltage measurements, or status flags.
- **D8 (0–1)**: Sequential packet indexing or cell addressing counter.

---

This analysis helps clarify the probable purpose of each byte in the respective CAN IDs. Further extended logging and analysis would confirm these interpretations.

## 🚦 Battery Pack Activation Procedure

Activation requires:

- Supplying stable **48V DC** power via breakout board connections to initialize the BMS.
- No additional manual start pins required.
- Send the unkown CAN Signal

Following these procedures ensures efficient and safe data capture and operational testing.

---

## 🕵️ Next Steps for Reverse Engineering

- Capture traffic using a CAN sniffer between the official charger and battery, or the working BESS.
- Decode the proprietary CAN messages.
- Independently replicate CAN communication to activate battery and read telemetry data.

---
