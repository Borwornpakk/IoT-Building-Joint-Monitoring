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
// Define Engineering Constraints (Based on DOWSIL™ 795 Datasheet)
const initialWidth = 10.0;    // Standard design joint width (mm)
const criticalLimit = 40.0;   // 400% Elongation limit before failure

// Real-time Safety Evaluation
let currentMovement = msg.payload.joint_mm;

if (currentMovement >= criticalLimit) {
    msg.payload = "CRITICAL: Potential Sealant Failure Detected!";
    msg.status = "danger"; // Triggers Red Alert on Dashboard
} else if (currentMovement >= (criticalLimit * 0.8)) {
    msg.payload = "WARNING: Approaching Elongation Limit.";
    msg.status = "warning"; // Triggers Amber Alert
} else {
    msg.payload = "SAFE: Operational movement within limits.";
    msg.status = "success"; // Normal Status
}

return msg;ing Javascript.js…]()
```
The logic uses a 10mm baseline joint width to monitor the 400% elongation threshold (40mm movement), which is the critical point for DOWSIL™ 795 before reaching its ultimate elongation capacity.

By setting a warning threshold at 80% of the critical limit, the system enables facility managers to perform inspections before any physical damage occurs.


## Proposed Results & Dashboard Visualization



The system processes incoming telemetry and visualizes it on a centralized digital twin dashboard.

Real-Time Analytics: The dashboard features historical line charts correlating temperature variations with joint movement trends over time.

Automated Risk Mitigation: An interactive alert log changes status dynamically. When simulated joint movement breaches critical structural design limits, the system triggers a visual critical alarm on the dashboard, demonstrating how data-driven insights can effectively streamline asset management, validate project warranties, and prevent catastrophic facade failures.

### ## How to Run
1. **Prerequisites:** Ensure you have [Node-RED](https://nodered.org/docs/getting-started/local) installed.
2. **Download Flow:** Download the [flows.json](./flows.json) file from this repository.
3. **Import to Node-RED:** - Open Node-RED in your browser.
   - Go to **Menu** (top right) > **Import**.
   - Select the `flows.json` file you just downloaded.
4. **Deploy:** Click the **Deploy** button to start the simulation.

