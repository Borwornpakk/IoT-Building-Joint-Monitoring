# IoT-Building-Joint-Monitoring
IoT-Based Joint Movement Monitoring System for Building Facades: An applied engineering case study utilizing Node-RED and MQTT to monitor real-time thermal expansion and predict sealant failure limits.

## Project Title
IoT-Based Structural Health Monitoring: Real-Time Joint Movement Analysis for Building Facades

https://github.com/user-attachments/assets/8cd05003-4eb0-45cb-8ea7-597d939ae8e3

## Abstract / Introduction

In modern high-rise architectures, building facades are constantly subjected to extreme temperature fluctuations, causing significant thermal expansion and contraction at structural joints. Elastomeric weather-seals and structural glazing sealants must continuously accommodate these cyclic movements. If the movement exceeds the sealant's elongation limits or design bite capacity, it leads to adhesive/cohesive failure, resulting in water infiltration, structural deterioration, and costly maintenance.

This project proposes a real-time, digital monitoring solution. By continuously tracking joint displacement alongside substrate surface temperatures, this system enables predictive building maintenance, allowing technical specialists to detect high-risk structural movements before physical sealant failure occurs.

## Key Features

◾ **Real-Time Facade Telemetry:** Continuous monitoring of joint movement (mm) and substrate temperature (°C).

◾ **Predictive Safety Logic:** Automated threshold detection based on DOWSIL™ 795 ultimate elongation capacity (400% limit).

◾ **Multi-Channel Notifications:** Instant visual alerts via Dashboard pop-ups and status indicators for critical structural risks.

◾ **IIoT Ready:** Fully compatible with MQTT brokers for seamless integration into smart building ecosystems.

## Methodology & System Architecture
This case study demonstrates a lightweight, industrial IoT (IIoT) architecture designed for structural health monitoring. The data flow and integration are engineered as follows:

◾ **Data Simulation (Edge Layer):** A simulated environment generates synchronized real-time data reflecting high-rise facade behavior, specifically joint displacement (mm) and substrate surface temperature (°C) influenced by diurnal thermal cycles.

◾ **Data Transmission (Network Layer):** The simulated sensor data is packaged into JSON format and transmitted over a secured network using the MQTT protocol, simulating low-latency industrial data ingestion.

◾ **Data Ingestion & Processing (Application Layer):** Node-RED acts as the central orchestration engine. It subscribes to the MQTT broker, parses the incoming telemetry, and applies a threshold-based logic to evaluate structural safety. If the displacement exceeds the pre-defined maximum elongation capacity of a standard structural silicone sealant, an automated alert system is triggered.

◾ **Core Engineering Logic:** Predictive Failure Analysis

To bridge the gap between material science and digital monitoring, I implemented a safety evaluation logic based on the DOWSIL™ 795 technical specifications. The system calculates the real-time elongation percentage and triggers alerts based on the material's ultimate capacity.

```javascript
const data = msg.payload;
if (!data || data.displacement === undefined) return null;

// DOWSIL 795 — groove 10mm
// 400% elongation = 40mm → ALARM (exceed design value)
// 240% elongation = 24mm → WARNING (60% ของ limit)
const ALARM_MM = data.alarmMm || 40;
const WARN_MM = data.warnMm || 24;
const disp = data.displacement;
const elong = data.elongation;

let level = 'OK';
let message = '';

if (disp >= ALARM_MM) {
    level = 'ALARM';
    message = '🚨 CRITICAL — Sensor ' + data.sensorId +
        ' (Zone ' + data.zone + ', Floor ' + data.floor + 'F)' +
        '\nDisplacement: ' + disp.toFixed(2) + 'mm' +
        ' (Elongation ' + elong + '%)' +
        '\nExceeds the DOWSIL 795 limit of 400%. (' + ALARM_MM + 'mm)' +
        '\nIt needs to be inspected and repaired immediately.!';
} else if (disp >= WARN_MM) {
    level = 'WARNING';
    message = '⚠️ WARNING — Sensor ' + data.sensorId +
        ' (Zone ' + data.zone + ', Floor ' + data.floor + 'F)' +
        '\nDisplacement: ' + disp.toFixed(2) + 'mm' +
        ' (Elongation ' + elong + '%)' +
        '\nEnter the surveillance zone (>' + WARN_MM + 'mm)';
}

if (level !== 'OK') {
    msg.payload = {
        level: level,
        message: message,
        sensor: data.sensorId,
        zone: data.zone,
        floor: data.floor,
        timestamp: data.timestamp,
        values: {
            displacement: disp,
            elongation: elong,
            temperature: data.temperature,
            alarmMm: ALARM_MM,
            warnMm: WARN_MM
        }
    };
    msg.topic = 'facade/joint/alarm';
    return msg;
}
return null;
```
The logic uses a 10mm baseline joint width to monitor the 400% elongation threshold (40mm movement), which is the critical point for DOWSIL™ 795 before reaching its ultimate elongation capacity.

By setting a warning threshold at 60% of the critical limit, the system enables facility managers to perform inspections before any physical damage occurs.


## Results & Dashboard Demonstration

<img width="1259" height="593" alt="Screenshot 2026-06-04 215636" src="https://github.com/user-attachments/assets/537f9f41-0091-4764-b365-d65183c09577" />

The centralized digital twin dashboard provides an intuitive, data-driven overview of structural health by correlating environmental factors with physical joint reactions.

**Dual-Axis Real-Time Graph:**

📉Left Y-Axis (Red): Tracks ambient and structural temperature variations over time.

📈Right Y-Axis (Blue): Measures the corresponding joint displacement (mm) and elongation percentage (%).

**Smart Alert Thresholds:**

➜ Orange Dashed Line (Warning Limit): Triggers a Warning status in the data table when the joint displacement exceeds 60% of its designed capacity.

➜ Red Dashed Line (Alarm Limit): Represents a critical structural design limit. Breaching this threshold changes the system state to a critical Alarm status.
    
**Sensor Network Status & Asset Management**

The Sensor Network Status table aggregates live telemetry from deployed sensor nodes across the facility, mapping physical locations (Zone and Floor) directly to key metrics: Temperature (°C), Displacement (mm), and % Elongation.

## Value Proposition & Predictive Maintenance:

By continuously capturing and analyzing these interrelated data points, the system significantly enhances building safety. It allows facility managers to shift from reactive repairs to data-driven Predictive Maintenance—precisely identifying high-risk zones prone to silicone sealant degradation and structural fatigue before catastrophic failures occur.

### ## How to Run
1. **Prerequisites:** Ensure you have [Node-RED](https://nodered.org/docs/getting-started/local) installed.
2. **Download Flow:** Download the [monitoring-flows.json](./monitoring-flows.json) file from this repository.
3. **Import to Node-RED:** - Open Node-RED in your browser.
   - Go to **Menu** (top right) > **Import**.
   - Select the `monitoring-flows.json` file you just downloaded.
4. **Deploy:** Click the **Deploy** button to start the simulation.

