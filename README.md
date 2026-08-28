# TRIUMPH X — AI-Powered Disaster Detection, Early Warning & Search-and-Rescue Drone

**Team:** TRIUMPH X  
**SIH 2026 Problem Statement ID:** SIH26219
**Theme:** Robotics and Drones  
**Category:** Hardware

## Overview

TRIUMPH X is a student/SIH prototype for an autonomous drone-based disaster-response platform. The concept extends the original search-and-rescue problem with a preventive **landslide-indicator monitoring and early-warning module**.

The prototype is designed around this workflow:

```text
Monitor → Capture → AI Analysis → Risk Assessment → GPS Location
       → Affected-Zone Estimate → Alert → Dashboard → Response
```

The platform has two complementary use cases:

1. **Before a disaster:** monitor vulnerable terrain and flag visual indicators that may require attention or an early-warning workflow.
2. **During/after a disaster:** support search and rescue by detecting people and recording location-tagged events.

> **Prototype limitation:** This repository is not a safety-certified landslide prediction or emergency-warning system. The current landslide detector uses lightweight computer-vision heuristics. Its risk score and affected-zone radii are demonstration values and must not be used for real evacuation or rescue decisions.

## Problem Context

Disaster-response teams may need rapid situational awareness over large, remote, flooded, mountainous or otherwise hard-to-reach areas. The project aims to use an autonomous drone, computer vision, GPS and a dashboard to convert aerial observations into actionable information for responders.

The landslide extension adds a preventive layer: when the image-analysis module detects configured visual indicators, the system can raise a high/critical demonstration alert and show the suspected location on the dashboard.

## Core Capabilities

### Landslide-indicator analysis

Implemented in `ai/landslide_detection.py`.

The current prototype analyzes a still image with OpenCV and calculates:

- a **ground-crack proxy** from image edge density
- a **debris-texture proxy** from grayscale texture variation

The function returns a demonstration risk score, risk level, detected indicators and confidence values.

These are visual proxies only. They are not measurements of soil stability, ground displacement or true landslide probability.

### Risk assessment

Implemented in `ai/risk_assessment.py`.

The prototype exposes a simple fusion function for:

- visual score
- rainfall index
- movement index

Rainfall and movement are currently integration inputs rather than live sensor streams. The weights in the code are prototype values and are not geotechnically validated.

### Person detection

Implemented in `ai/person_detection.py`.

The module provides an optional **Ultralytics YOLO** integration point. A compatible model file must be supplied separately at:

```text
models/person_model.pt
```

The repository does not include model weights.

### GPS and affected-zone estimation

Implemented in `gps/location_tracker.py`.

The module provides:

- `GPSPoint` for latitude/longitude values
- Haversine distance calculation
- a demonstration affected-zone radius selected from the risk level

Current prototype radii are:

| Risk level | Demo radius |
|---|---:|
| LOW | 100 m |
| MEDIUM | 200 m |
| HIGH | 350 m |
| CRITICAL | 500 m |

These are **not validated evacuation boundaries**.

### Alerts

Implemented in `alerts/alert_service.py` and `alerts/twilio_adapter.py`.

The default configuration is:

```text
ALERT_MODE=mock
```

In mock mode, the warning is printed to the terminal. An optional Twilio adapter is included as an integration point for SMS alerts using the team's own authorized credentials and recipient numbers.

No live public-warning infrastructure is included in this repository.

### Web dashboard

Implemented in:

```text
dashboard/app.py
dashboard/templates/index.html
dashboard/static/app.js
```

The Flask dashboard displays prototype state including GPS position, risk level, risk score, detected indicators, affected-zone radius, alert status and recent events.

Available routes:

```text
GET  /
GET  /api/state
POST /api/analyze
POST /api/demo
```

### Drone navigation placeholders

Implemented in:

```text
drone/flight_control.py
drone/waypoint_navigation.py
```

The current functions are **simulation/placeholders**. They do not connect to a real flight controller. The navigation module can generate simple lawnmower-style waypoint coordinates, while flight-control functions return simulation messages.

The SIH presentation identifies **ArduPilot/PX4** as the intended integration direction for future hardware integration.

## Architecture

```text
                  DRONE / TEST IMAGE
                          |
                          v
                   AI ANALYSIS LAYER
                 /                    \
                /                      \
    Landslide indicators           Person detection
      (prototype CV)               (optional YOLO)
                \                      /
                 \                    /
                  +--------+---------+
                           |
                           v
                     Risk assessment
                           |
                           v
                    GPS / location
                           |
                           v
                  Affected-zone estimate
                      /            \
                     /              \
                    v                v
              Alert service      Web dashboard
             mock / optional SMS     Flask
```

## Repository Structure

```text
TRIUMPH-X-Disaster-Drone/
├── ai/
│   ├── __init__.py
│   ├── landslide_detection.py
│   ├── person_detection.py
│   └── risk_assessment.py
├── alerts/
│   ├── __init__.py
│   ├── alert_service.py
│   └── twilio_adapter.py
├── dashboard/
│   ├── __init__.py
│   ├── app.py
│   ├── static/
│   │   └── app.js
│   └── templates/
│       └── index.html
├── drone/
│   ├── __init__.py
│   ├── flight_control.py
│   └── waypoint_navigation.py
├── gps/
│   ├── __init__.py
│   └── location_tracker.py
├── models/
│   └── README.md
├── data/
│   └── sample_images/
│       └── README.md
├── scripts/
│   └── demo.py
├── .env.example
├── .gitignore
├── requirements.txt
└── README.md
```

## Technology Stack

- **Python** — application and prototype logic
- **OpenCV** — image processing and visual heuristics
- **NumPy** — numerical image operations
- **Flask** — web dashboard and API
- **python-dotenv** — environment-variable loading
- **Ultralytics YOLO** — optional person-detection integration
- **GPS coordinate calculations** — location and distance handling
- **ArduPilot/PX4** — planned flight-controller integration direction
- **Twilio** — optional SMS integration point

## Installation

### 1. Clone the repository

```bash
git clone https://github.com/YOUR-USERNAME/TRIUMPH-X-AI-Disaster-Drone.git
cd TRIUMPH-X-AI-Disaster-Drone
```

Replace `YOUR-USERNAME` with your GitHub username.

### 2. Create a virtual environment

#### Windows

```bash
python -m venv .venv
.venv\Scripts\activate
```

#### Linux/macOS

```bash
python3 -m venv .venv
source .venv/bin/activate
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

The `requirements.txt` file contains Flask, python-dotenv, OpenCV, NumPy and an optional Ultralytics dependency.

### 4. Configure environment variables

Copy `.env.example` to `.env`.

Windows:

```bash
copy .env.example .env
```

Linux/macOS:

```bash
cp .env.example .env
```

The default alert mode is mock mode, so the basic demo does not require Twilio credentials.

## Run the Dashboard

From the repository root:

```bash
python dashboard/app.py
```

Open:

```text
http://127.0.0.1:5000
```

The application uses port `5000` and binds to `0.0.0.0` when started through `dashboard/app.py`.

## Run a Demonstration Event

The dashboard includes a demonstration endpoint:

```text
POST /api/demo
```

It accepts one of:

```text
LOW
MEDIUM
HIGH
CRITICAL
```

Example JSON body:

```json
{
  "level": "HIGH"
}
```

For `HIGH` and `CRITICAL`, the default mock alert is triggered.

## Analyze an Image

The dashboard exposes:

```text
POST /api/analyze
```

Upload an image using the multipart field name:

```text
image
```

The endpoint runs the current OpenCV-based landslide-indicator heuristic and updates the dashboard state.

For a local test image, place an image in:

```text
data/sample_images/
```

and run:

```bash
python scripts/demo.py
```

The command-line demo uses the first file found in that directory. The repository intentionally does not include a specific real-world landslide image dataset.

## Optional Person Detection

The person detector imports Ultralytics only when called. After supplying a compatible model at:

```text
models/person_model.pt
```

you can run:

```python
from ai.person_detection import detect_people

result = detect_people(
    "path/to/frame.jpg",
    model_path="models/person_model.pt"
)

print(result)
```

Model weights should not be committed unless their license and repository size requirements permit it.

## Optional SMS Alerts

The Twilio adapter is optional. The default `.env` setting is:

```text
ALERT_MODE=mock
```

To use the Twilio integration, install the Twilio package separately and configure:

```text
ALERT_MODE=twilio
TWILIO_ACCOUNT_SID=
TWILIO_AUTH_TOKEN=
TWILIO_FROM_NUMBER=
ALERT_TO_NUMBERS=
```

Keep credentials in `.env`. Do not commit secrets to GitHub.

## SIH Exhibition Prototype

A simple controlled demonstration can use:

- a small drone/RC aircraft or camera setup
- RGB/USB camera
- laptop or Raspberry Pi for processing
- optional GPS module
- Flask dashboard
- buzzer/LED for local demonstration
- a model hill made from cardboard/soil

A safe exhibition flow is:

```text
Model terrain / test image
          ↓
Camera capture
          ↓
Prototype image analysis
          ↓
Risk level
          ↓
GPS coordinate
          ↓
Demo affected-zone radius
          ↓
Mock warning
          ↓
Dashboard display
```

For the rescue portion, a controlled image containing people can be passed to the optional YOLO module after a compatible model is supplied.

## What Is Prototype vs. Future Work?

### Implemented in this repository

- OpenCV-based landslide-indicator heuristic
- Demonstration risk-level calculation
- Prototype affected-zone radius mapping
- GPS coordinate utilities
- Mock alert generation
- Optional Twilio SMS adapter
- Optional YOLO person-detection integration
- Flask dashboard and API routes
- Simulated drone takeoff/return-to-home functions
- Simple lawnmower waypoint generation

### Planned / future development

- Trained and field-validated landslide model
- Terrain segmentation and multi-frame change detection
- Rainfall, soil-moisture and ground-movement sensors
- Terrain elevation and slope information
- Validated geotechnical risk model
- Real-time drone telemetry
- Direct ArduPilot/PX4 hardware integration
- Thermal-image fusion for victim detection
- Production-grade communications and emergency-management integration
- Authority-approved alert and evacuation procedures

## Limitations and Safety

The current project is a **student prototype**. Specifically:

- Image edge density and texture are only proxies for possible visual indicators.
- The current risk thresholds are prototype rules.
- The affected-zone radius is a demonstration value, not an evacuation boundary.
- Rainfall and movement inputs are not connected to live sensors in this starter repository.
- Drone flight functions are simulation placeholders.
- The alert system is not a public emergency-warning service.
- No claim is made that the system can guarantee landslide prediction, victim detection, or safe autonomous flight.

Any real deployment would require domain validation, appropriate sensing, communications redundancy, flight safety procedures, regulatory compliance and integration with responsible disaster-management authorities.

## License

No open-source license is currently specified in this repository.

Unless the team adds a license, reuse and redistribution of original code should not be assumed to be permitted. Add a license only after deciding how the project should be shared.

## Team

**TRIUMPH X**  
Smart India Hackathon 2026

---

**Project title:** AI-Powered Autonomous Disaster Detection, Early Warning & Search-and-Rescue Drone
