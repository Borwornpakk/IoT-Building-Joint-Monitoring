# IoT-Based Structural Health Monitoring: Real-Time Joint Movement Analysis for Building Facades

> Developed a Node-RED and MQTT-based IIoT monitoring workflow simulating real-time thermal and joint movement data for building facade assessment. The system applies DOWSIL™ 795 TDS specifications directly — using **±50% joint movement capability (ASTM C719 Class 50)** as the design limit for a 10 mm joint (= ±5 mm), with warning at 80% of limit (4 mm) and alarm beyond design movement (>5 mm), while referencing **400% ultimate elongation at break (ASTM C1135)** as the absolute material failure boundary. This enables predictive maintenance alerting grounded in manufacturer's certified test data.

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [System Architecture](#2-system-architecture)
3. [Hardware & Software Requirements](#3-hardware--software-requirements)
4. [Implementation Details](#4-implementation-details)
5. [Usage & Demonstration](#5-usage--demonstration)
6. [Discussion & Conclusion](#6-discussion--conclusion)
7. [Future Work & Enhancements](#7-future-work--enhancements)
8. [References](#8-references)

---

## 1. Introduction

### 1.1 Problem Statement (Pain Point)

Building facades — particularly curtainwall and structural glazing systems — undergo continuous cyclic movement driven by thermal expansion, wind load, and seismic forces. The silicone sealant at each panel joint absorbs these movements throughout the service life of the building. Without a systematic monitoring approach, sealant degradation and joint over-extension remain largely invisible until physical failure occurs, resulting in water infiltration, glass detachment, or structural compromise.

Current industry practice relies on periodic visual inspection, typically conducted every one to three years. This reactive approach fails to capture the cumulative movement history, provides no early warning of approaching design limits, and offers no data-driven basis for maintenance scheduling. In high-rise or large-scale facade systems, the cost and safety risk associated with undetected sealant failure are significant.

### 1.2 How This Project Addresses the Problem

This project proposes a **conceptual IoT-based structural health monitoring workflow** using Node-RED and MQTT to demonstrate how continuous, real-time data collection could replace or supplement conventional inspection cycles. By integrating sensor simulation directly with the published mechanical limits of DOWSIL™ 795 Structural Glazing Sealant — specifically the ±50% joint movement capability (ASTM C719 Class 50) — the system provides:

- **Continuous visibility** into joint displacement relative to the engineered design limit
- **Threshold-based alerting** at 80% of the design limit (early warning) and at the design limit itself (alarm)
- **Correlation analysis** between ambient temperature and joint displacement, supporting root-cause assessment
- **A foundation for predictive maintenance**, enabling maintenance teams to act on data rather than calendar schedules

---

## 2. System Architecture

### 2.1 Workflow Overview

<img width="1313" height="379" alt="Screenshot 2026-06-05 162019" src="https://github.com/user-attachments/assets/2861fa75-35eb-42ee-a67d-d11c7acb9738" />

### 2.2 Data Flow Description

| Stage | Node Type | Function |
|---|---|---|
| **Simulation** | Inject | Triggers data generation every 2 seconds |
| **Data Generation** | Function | Produces randomised temperature and displacement values with diurnal thermal cycle |
| **IIoT Transport** | MQTT Out / MQTT In | Publishes and subscribes to topic `facade/joint/data` via MQTT protocol |
| **Alarm Evaluation** | Function | Compares displacement against DOWSIL 795 TDS thresholds; routes alerts |
| **Notification** | ui_notification | Delivers pop-up alert to dashboard on threshold breach |
| **Chart Preparation** | Function | Formats payload for dual-axis real-time chart via WebSocket |
| **Visualisation** | ui_template | Renders metrics, live chart, sensor table, and simulation controls |

### 2.3 Alert Threshold Logic (Based on DOWSIL™ 795 TDS)

| Condition | Displacement | Elongation | Action |
|---|---|---|---|
| **Normal** | < 4.0 mm | < 40% | Green — no action |
| **Warning** | ≥ 4.0 mm | ≥ 40% | Amber — schedule inspection |
| **Alarm** | ≥ 5.0 mm | ≥ 50% | Red — exceeds ASTM C719 design limit |
| **Critical** | ≥ 40.0 mm | ≥ 400% | Dark red — exceeds ultimate elongation at break |

> Design limit derivation: Joint width = 10 mm; ±50% (ASTM C719 Class 50) = ±5 mm maximum allowable movement.

---

## 3. Hardware & Software Requirements

### 3.1 Hardware

In this project, physical sensors are **simulated entirely within Node-RED** using randomised data generation with a diurnal temperature cycle. The architecture is designed, however, to be compatible with the following hardware components in a physical deployment:

| Component | Description | Simulated In This Project |
|---|---|---|
| Displacement Sensor | Linear potentiometer or LVDT for joint gap measurement | ✅ Yes |
| Temperature Sensor | NTC thermistor or PT100 for ambient facade temperature | ✅ Yes |
| IoT Gateway / Edge Device | Raspberry Pi or industrial gateway (e.g. Advantech WISE) | ✅ Replaced by Node-RED on PC |
| Network Infrastructure | Wi-Fi, Ethernet, or LoRaWAN for sensor data transmission | ✅ Replaced by localhost MQTT |

### 3.2 Software & Platform (Tech Stack)

| Component | Tool / Version | Purpose |
|---|---|---|
| **Flow Engine** | Node-RED v3.x | Visual workflow programming for IIoT logic |
| **Message Broker** | Eclipse Mosquitto (MQTT v3.1.1) | IIoT-standard pub/sub messaging — `mqtt://localhost:1883` |
| **Dashboard** | node-red-dashboard v2.x | Real-time UI rendering (ui_template, ui_notification) |
| **Chart Library** | Chart.js 2.x (bundled) | Dual-axis line chart with dynamic reference lines |
| **Transport Layer** | WebSocket (`/ws/chart`) | Real-time streaming from Node-RED to browser dashboard |
| **Runtime** | Node.js v18+ | Underlying runtime for Node-RED |
| **Operating System** | Windows 10/11 or Linux (Ubuntu 22.04) | Host environment |

### 3.3 MQTT Topic Structure

```
facade/
  joint/
    data      ← sensor readings (temperature, displacement, elongation)
    alarm     ← triggered alarm payloads (level, message, sensor ID)
```

---

## 4. Implementation Details

### 4.1 Sensor Data Simulation

The `Generate Random Sensor Data` Function Node simulates a physically realistic diurnal temperature cycle using a sinusoidal function, then derives joint displacement as a correlated response to thermal change — consistent with the thermal expansion behaviour of aluminium curtainwall framing:

```javascript
// Diurnal temperature cycle
const hourAngle = (now.getHours() + now.getMinutes()/60) / 24 * 2 * Math.PI;
const baseTempC = 28 + 12 * Math.sin(hourAngle - Math.PI/2);  // 16–40°C range

// Displacement correlated to temperature (thermal movement)
const tempFactor = (tempC - 15) / 35;   // normalise 15–50°C → 0–1
const dispMm = rnd(
    Math.max(0.1, DESIGN_MOVE_MM * tempFactor * 0.5),
    Math.min(DESIGN_MOVE_MM * 1.5, DESIGN_MOVE_MM * tempFactor * 1.2 + 1)
);
```

Four virtual sensors are simulated simultaneously, each assigned to a different facade zone (North, South, East, West) and floor level.

### 4.2 Alarm Logic — Direct Application of DOWSIL™ 795 TDS

All threshold values are derived directly from the manufacturer's Technical Data Sheet (Form No. 63-1217-01-0124):

```javascript
// ASTM C719 Class 50: ±50% joint movement capability
// Joint width = 10 mm → Design movement limit = ±5 mm
const ALARM_MM    = 5.0;   // Exceeds ASTM C719 design limit → ALARM
const WARN_MM     = 4.0;   // 80% of design limit            → WARNING
const ULTIMATE_MM = 40.0;  // 400% ultimate elongation (ASTM C1135, 21-day cure) → CRITICAL
```

The alarm function evaluates three tiers independently and routes qualified events to the `ui_notification` node for real-time popup delivery.

### 4.3 Dual-Axis Real-Time Chart

The dashboard chart uses Chart.js 2.x with two independent Y-axes, delivered as a custom `ui_template` node receiving data via a dedicated WebSocket endpoint (`/ws/chart`):

- **Left Y-axis:** Ambient temperature (°C), range 10–55°C
- **Right Y-axis:** Joint displacement (mm), range 0–8 mm, labelled as percentage of design limit
- **Reference line 1:** Alarm threshold at 5 mm (ASTM C719, dashed red)
- **Reference line 2:** Warning threshold at 4 mm (dashed amber)

Using WebSocket rather than AngularJS `$scope` binding resolves compatibility issues between Chart.js and the AngularJS runtime embedded in node-red-dashboard v2.

### 4.4 Notification System

When displacement exceeds the warning or alarm threshold, the `Format Notification msg` Function Node constructs a structured alert message including sensor ID, zone, floor, measured displacement, elongation percentage, and applicable TDS reference. The message is then passed to `ui_notification` for immediate popup display on the dashboard.

---

## 5. Usage & Demonstration

### 5.1 Dashboard Layout

The dashboard is accessible at `http://localhost:1880/ui` after deploying the flow. It comprises four integrated sections rendered within a single `ui_template` node:

<img width="1267" height="717" alt="Screenshot 2026-06-05 152807" src="https://github.com/user-attachments/assets/9d5d1923-9871-4e31-850e-865a476226d6" />


### 5.2 Chart Interpretation

The dual-axis line chart provides the primary analytical view of the monitoring system:

**What the chart shows:**
The red line (left axis) represents ambient temperature fluctuation across the facade. The blue line (right axis) represents the measured joint displacement. The dashed red line marks the ASTM C719 alarm threshold (5 mm / 100% of design limit); the dashed amber line marks the warning threshold (4 mm / 80%).

**What to observe:**
As temperature rises, joint displacement increases in a correlated manner — consistent with thermal expansion of the aluminium framing system. This positive correlation between temperature and displacement validates the physical mechanism being simulated. When the blue line approaches or crosses the dashed reference lines, the system enters warning or alarm state.

**What value this provides:**
The chart enables maintenance personnel to identify periods of elevated joint stress (typically mid-afternoon peak temperatures), assess whether displacement is trending toward the design limit, and determine whether the rate of increase is accelerating — all without requiring physical site access.

### 5.3 Alert System Behaviour

When displacement exceeds threshold values, the system responds across three layers simultaneously:

1. **Metric card colour change** — the Displacement and Design Limit badges change from green (Within Limit) → amber (WARNING) → red (ALARM) → dark red (CRITICAL)
2. **Status bar update** — the live status text changes to display the sensor ID, measured displacement, percentage of design limit used, and the applicable TDS standard reference
3. **Pop-up notification** — the `ui_notification` node delivers a temporary alert containing full sensor identification, measured values, and maintenance recommendation

**Example alarm message:**
```
🚨 ALARM — Sensor SNS-F03-E (Zone East, Floor 3F)
Displacement: 5.23 mm (104.6% of design limit)
Exceeds ASTM C719 Class 50 design limit (±50% = ±5 mm)
Immediate inspection and structural assessment required.
```

---

## 6. Discussion & Conclusion

### 6.1 Key Findings

This project demonstrates that a Node-RED and MQTT-based monitoring workflow can be effectively structured around published material specifications to provide technically grounded, real-time structural health indicators. The following findings are noteworthy:

**Thermal correlation is directly observable.** The simulated data reproduces the expected positive correlation between ambient temperature and joint displacement, consistent with the thermal expansion coefficient of aluminium (approximately 23 × 10⁻⁶ /°C). This confirms the physical validity of the simulation model and suggests that real sensor data from an equivalent system would exhibit the same pattern.

**Multi-tier alerting enables proportionate response.** By distinguishing between a warning state (80% of design limit) and an alarm state (100% of design limit), the system provides sufficient lead time for preventive maintenance scheduling — rather than only alerting at the point of exceedance. The additional critical tier (400% elongation, ASTM C1135) defines the absolute material boundary, providing a reference for failure risk assessment.

**TDS-grounded thresholds improve engineering credibility.** Anchoring alarm logic directly to ASTM C719 Class 50 and ASTM C1135 data from the DOWSIL™ 795 TDS ensures that monitoring decisions are traceable to certified manufacturer test data, rather than arbitrary engineering assumptions.

### 6.2 How the System Supports Safety and Maintenance Planning

| Benefit | Mechanism |
|---|---|
| **Early warning** | Warning alert at 80% of design limit provides time to schedule inspection before structural risk arises |
| **Targeted inspection** | Sensor-level identification (ID, zone, floor) directs inspectors to specific locations, reducing survey time and cost |
| **Historical context** | Continuous data capture enables trend analysis across temperature cycles and seasons |
| **Maintenance deferral justification** | If displacement consistently remains below 50% of the design limit, inspection intervals may be safely extended with supporting data |

### 6.3 Limitations

| Limitation | Description |
|---|---|
| **Simulated data only** | No physical sensors are connected; all readings are algorithmically generated and do not represent actual structural measurements |
| **No data persistence** | Dashboard data is lost on browser refresh or Node-RED restart; no database or logging layer is implemented |
| **Single joint width assumption** | The system is configured for a 10 mm joint; real-world facades may have varying joint widths requiring per-sensor configuration |
| **No rate-of-change detection** | Alarms trigger on absolute thresholds only; rapid displacement events (e.g., seismic) may not be captured appropriately |
| **MQTT without TLS** | The current configuration uses unencrypted MQTT over localhost; production deployment would require TLS and authentication |

---

## 7. Future Work & Enhancements

| Priority | Enhancement | Description |
|---|---|---|
| 🔴 High | **Data Persistence (InfluxDB / SQLite)** | Log all sensor readings with timestamps to enable long-term trend analysis and reporting |
| 🔴 High | **Physical Sensor Integration** | Replace simulated data with real LVDT displacement sensors and PT100 temperature probes connected via MQTT-capable edge devices |
| 🟡 Medium | **Rate-of-Change (Velocity) Alerting** | Alert when displacement changes faster than a defined threshold per unit time — critical for detecting seismic or impact events |
| 🟡 Medium | **Thermal Expansion Calculation** | Implement ΔL = α × L₀ × ΔT to decompose measured displacement into thermal and structural components |
| 🟡 Medium | **Remaining Service Life Estimation** | Estimate remaining sealant service life based on cumulative elongation cycles and UV weathering exposure |
| 🟢 Low | **AI/ML Predictive Maintenance** | Train a time-series model (e.g., LSTM or Prophet) on historical displacement data to forecast threshold exceedance events |
| 🟢 Low | **BMS Integration** | Connect to Building Management System via BACnet or OPC-UA to correlate joint movement data with HVAC, occupancy, and structural load data |
| 🟢 Low | **Multi-joint Width Configuration** | Allow per-sensor joint width parameters to support facades with varied joint geometries |
| 🟢 Low | **Automated Maintenance Report Generation** | Scheduled PDF report export summarising displacement statistics, alarm events, and maintenance recommendations |

---

## 8. References

### Primary Product Reference

> The Dow Chemical Company. (2024). *DOWSIL™ 795 Structural Glazing Sealant — Technical Data Sheet* (Form No. 63-1217-01-0124 S2D). Dow Inc. Retrieved from dow.com

**Key values applied in this project (from TDS):**

| Property | Test Standard | Value | Application in System |
|---|---|---|---|
| Joint movement capability | ASTM C719 Class 50 | **±50%** | Primary design limit → Alarm threshold |
| Ultimate elongation at break | ASTM C1135 (21-day cure) | **400%** | Absolute material failure boundary |
| Elongation (7-day cure) | ASTM D412 | 670% | Reference only |
| Tensile strength at 100% elongation | ASTM C1135 | 0.6 MPa | Structural load reference |
| Ultimate tensile strength at break | ASTM C1135 | 1.2 MPa | Failure load reference |
| Minimum structural bite depth | TDS Joint Design | 6 mm | Joint geometry constraint |
| Minimum glueline thickness | TDS Joint Design | 6 mm | Joint geometry constraint |
| Bite to glueline ratio | TDS Joint Design | ≤ 3:1 | Joint design guideline |

### Standards Referenced

| Standard | Title | Role in Project |
|---|---|---|
| ASTM C719 | *Test Method for Adhesion and Cohesion of Elastomeric Joint Sealants* | Defines ±50% Class 50 movement capability — primary alarm basis |
| ASTM C1135 | *Test Method for Determining Tensile Adhesion Properties of Structural Sealants* | Defines 400% ultimate elongation — critical failure boundary |
| ASTM C1184 | *Specification for Structural Silicone Sealants* | Product qualification standard |
| ASTM C920 | *Specification for Elastomeric Joint Sealants* | General sealant specification (Class 50) |
| ASTM D412 | *Test Methods for Vulcanized Rubber and TPE — Tension* | 670% elongation (7-day cure) — reference property |

### Tools & Frameworks

- Node-RED: https://nodered.org
- Eclipse Mosquitto MQTT Broker: https://mosquitto.org
- node-red-dashboard: https://flows.nodered.org/node/node-red-dashboard
- Chart.js: https://www.chartjs.org

---

*This project was developed as a conceptual IIoT monitoring prototype. All sensor data is simulated. Threshold values are derived from the DOWSIL™ 795 Technical Data Sheet and applicable ASTM standards. This system is not a substitute for structural engineering assessment or manufacturer-approved joint design review.*
