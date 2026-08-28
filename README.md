TRIUMPH X — AI Disaster Detection, Early Warning & Search-Rescue Drone

A college/SIH prototype for an autonomous drone platform that can:

Detect landslide warning indicators from camera imagery.

Estimate a simple risk level and map the suspected affected zone.

Raise timely alerts for registered residents/authorities through a pluggable alert module.

Detect people and hazards after a disaster and show GPS-tagged events on a web dashboard.

This repository is a prototype/starter implementation, not a safety-certified landslide prediction system. A real deployment needs validated geotechnical models, reliable sensors, communications redundancy, field calibration, emergency-management integration, and regulatory approval.

Architecture

             DRONE CAMERA / TEST IMAGE
                       |
                       v
                AI INFERENCE LAYER
              /                     \
     Person/Vehicle              Landslide Indicators
       Detection                 (prototype CV rules)
              \                     /
               +--------+-----------+
                        |
                  GPS + Risk Engine
                        |
                Affected-Zone Map
                        |
                +-------+--------+
                |                |
           Alert Service      Dashboard
          SMS / Mock Alert   Flask Web UI

Repository structure

TRIUMPH-X-Disaster-Drone/
├── ai/
│   ├── landslide_detection.py
│   ├── person_detection.py
│   └── risk_assessment.py
├── alerts/
│   ├── alert_service.py
│   └── twilio_adapter.py
├── dashboard/
│   ├── app.py
│   ├── templates/index.html
│   └── static/app.js
├── drone/
│   ├── flight_control.py
│   └── waypoint_navigation.py
├── gps/
│   └── location_tracker.py
├── models/
│   └── README.md
├── data/sample_images/
├── scripts/
│   └── demo.py
├── requirements.txt
├── .env.example
└── README.md

1. Setup

Python 3.10+ is recommended.

python -m venv .venv
# Windows
.venv\\Scripts\\activate
# Linux/macOS
source .venv/bin/activate

pip install -r requirements.txt

Copy environment settings:

copy .env.example .env

or on Linux/macOS:

cp .env.example .env

2. Run the dashboard

python dashboard/app.py

Open http://127.0.0.1:5000.

The dashboard includes:

current GPS position

landslide risk level

detected indicators

affected-zone radius

alert status

rescue/landslide event history

3. Run the command-line demo

python scripts/demo.py

This simulates a drone observation, evaluates risk and produces an alert event without requiring a physical drone or a trained model.

4. AI model integration

The current landslide module intentionally uses a prototype image-analysis heuristic so the repository can run immediately. For a real AI model, replace or extend it with a trained segmentation/object-detection model.

For person detection, the repository supports Ultralytics YOLO when a model file is available.

Example:

from ai.person_detection import detect_people
result = detect_people("frame.jpg", model_path="models/person_model.pt")
print(result)

Recommended future model labels

landslide_crack
soil_displacement
rockfall_debris
water_seepage
blocked_road
person
vehicle

5. Alert workflow

AI observation
     ↓
Risk assessment
     ↓
GPS coordinate
     ↓
Affected radius
     ↓
Alert message
     ↓
Registered residents + authorities

The default alert adapter is a console/mock adapter, which is safe for demonstration. A Twilio adapter is included as an optional integration point and must be configured with your own credentials.

6. Suggested college prototype

You can demonstrate the complete concept with:

small drone or RC aircraft/camera

Raspberry Pi or laptop for processing

GPS module (or simulated coordinates)

USB/RGB camera

optional thermal camera

Flask dashboard

buzzer/LED for local warning

mock SMS alert for the exhibition

A simple physical demo can use a model hill made from cardboard/soil. Add artificial cracks and loose debris, capture the scene, and trigger the risk engine.

Safety / scope note

Do not present the prototype risk score as a guaranteed prediction of a real landslide. The current implementation is intended for demonstration of the software workflow. Production warning decisions should incorporate validated geotechnical measurements such as rainfall, soil moisture, slope deformation and ground movement, together with local disaster-management procedures.
