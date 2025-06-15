# 🌱 Smart Agriculture System

An IoT-based smart agriculture solution using **STM32F446RE** and **RuggedBoard A5D2x** to monitor environmental conditions and control irrigation automatically via **ThingsBoard**.

---

## 🚀 Overview

This project helps automate agricultural monitoring and irrigation using:
- **Soil Moisture Sensor** for detecting soil wetness
- **DHT22 Sensor** for temperature and humidity readings
- **STM32F446RE** for reading sensor data and sending it via UART
- **RuggedBoard A5D2x** for MQTT communication with ThingsBoard
- **Relay Control** based on sensor thresholds set from the ThingsBoard dashboard

---

## 📡 Data Flow

1. **Sensor Reading (STM32):**
   - Reads soil moisture and temperature/humidity data.
   - Formats data into a **JSON string**.
   - Sends via **UART** to the RuggedBoard.

2. **Data Handling (RuggedBoard A5D2x):**
   - Receives JSON over UART.
   - Parses and publishes data to **ThingsBoard** via **MQTT** over Ethernet.
   - Subscribes to a ThingsBoard RPC topic to listen for relay control commands.

3. **Dashboard Interaction (ThingsBoard):**
   - Visualizes real-time soil moisture and environmental conditions.
   - Provides a switch to turn the **irrigation relay ON/OFF** manually or automatically based on sensor thresholds.

---

## 🔧 Hardware Components

| Component             | Description                          |
|-----------------------|--------------------------------------|
| STM32F446RE           | Microcontroller for sensor interface |
| Soil Moisture Sensor  | Analog soil wetness reader           |
| DHT22 Sensor          | Temp & Humidity digital sensor       |
| RuggedBoard A5D2x     | Linux board with Ethernet, UART      |
| Relay Module          | Controls irrigation pump/valve       |
| Ethernet Cable        | Network connectivity for MQTT        |

---

## 📦 Data Format (JSON)

**Example Data Sent from STM32 to RuggedBoard:**
```json
{
  "temperature": 28.4,
  "humidity": 62.5,
  "soil_moisture": 310
}
```
## 📡 MQTT Configuration
  - **Broker:** ThingsBoard Cloud or Local
  - **Topic (Telemetry):** v1/devices/me/telemetry
  - **Topic (RPC/Relay Control):** v1/devices/me/rpc/request/+
  - **Authentication:** Access token from ThingsBoard device

---
## 💻 Software/Tools Used
  - STM32CubeIDE for embedded C firmware
  - Python or C app on RuggedBoard to handle UART & MQTT
  - ThingsBoard for dashboard and control panel
---
## 🔌 Relay Control Logic
You can configure the ThingsBoard dashboard to:
  - Turn relay ON if soil moisture is below a certain threshold.
  - Or allow manual control via dashboard switch.
---
## 📊 Example Dashboard
The ThingsBoard dashboard displays:
  - Live temperature & humidity graphs
  - Soil moisture level gauge
  - Relay control switch
---


