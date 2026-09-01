Obstacle-Avoiding Robot Car

Autonomous navigation on a budget build — an Arduino-based robot that senses obstacles with an ultrasonic sensor, decides which way to turn, and steers itself with zero human input.

Show Image Show Image Show Image Show Image

Overview

This project demonstrates the fundamental sense → process → act loop that underlies every autonomous system, from robotic vacuum cleaners to self-driving parking assist. An HC-SR04 ultrasonic sensor mounted on an SG90 servo continuously scans the space ahead. When an obstacle is detected within the safe threshold, the robot stops, looks left and right, and pivots toward whichever side has more open space — all without soldering, and for under ₹1,500 in parts.

Built as a first-year engineering project to move from theory into a working, testable prototype.

Demo

Add a photo or short clip of the robot in action here once you have one — GitHub renders images and GIFs directly in the README.

![Robot demo](demo.gif)

Table of contents
Components
Wiring / pin connections
Circuit diagram
How it works
Arduino code
Setup instructions
Testing results
Troubleshooting
Future improvements
License
Components
#	Component	Specification	Qty	Approx. cost
1	Arduino Uno R3	ATmega328P, 16 MHz	1	₹350
2	L298N motor driver	Dual H-bridge, 5–35V	1	₹120
3	HC-SR04 ultrasonic sensor	2–400 cm range	1	₹70
4	SG90 servo motor	0–180°, 4.8–6V	1	₹100
5	2WD/4WD chassis kit	DC geared motors + wheels	1	₹350
6	3.7V Li-ion battery	18650, series wired for 7.4V	2	₹150
7	Jumper wires	M–M, M–F, F–F	1 set	₹50

Total: ₹1,000 – ₹1,500 — no soldering required.

Wiring / pin connections
HC-SR04 ultrasonic sensor
Sensor pin	Connects to
VCC	Arduino 5V
GND	Arduino GND
Trig	Arduino D9
Echo	Arduino D10
SG90 servo motor
Servo wire	Connects to
Red (VCC)	Arduino 5V
Brown (GND)	Arduino GND
Orange (signal)	Arduino D6

Arduino Uno has only one 5V and one GND pin. Twist the sensor's VCC wire together with the servo's red wire and insert both into 5V; do the same with GND. A mini breadboard works too if you have one.

L298N motor driver
L298N pin	Connects to
IN1	Arduino D2
IN2	Arduino D3
IN3	Arduino D4
IN4	Arduino D5
ENA / ENB	Jumper caps on — full speed, no PWM needed
12V / VMS	Battery pack +
GND	Battery pack – and Arduino GND (common ground)
OUT1 / OUT2	Left motor terminals
OUT3 / OUT4	Right motor terminals
Battery pack (2 × 3.7V in series → 7.4V)

Connect battery 1 (–) to battery 2 (+) to form the series link (most 2-cell holders do this internally). The two free ends become your pack's + and – terminals, which go to the L298N as shown above.

Common ground is the single most important rule in this build. Battery (–), L298N GND, and Arduino GND must all connect at one point. Skipping this is the #1 cause of "nothing works" — the Arduino's control signals become meaningless to the motor driver without a shared reference.

Circuit diagram
┌──────────────┐          ┌────────────────────┐          ┌──────────────┐
│   HC-SR04    │ Trig/Echo│                    │ IN1-IN4  │    L298N     │
│  Ultrasonic  ├─────────►│    ARDUINO UNO     ├─────────►│ Motor Driver │
│    Sensor    │          │   (ATmega328P)     │          │              │
└──────────────┘          │  Processing Unit   │          └──────┬───────┘
┌──────────────┐          │                    │                 │ OUT1-4
│  SG90 Servo  │◄─────────┤                    │                 ▼
│    Motor     │   PWM    └─────────▲──────────┘          ┌──────────────┐
└──────────────┘                    │                     │  2× DC Motor │
                                     │ Power               │   + Wheels   │
                           ┌─────────┴──────────┐          └──────────────┘
                           │  2×3.7V Li-ion      │
                           │  Battery (7.4V)     │
                           └─────────────────────┘
How it works

The Arduino runs this decision loop roughly every 80 milliseconds:

Sense — servo centers to 90°, HC-SR04 fires a pulse, distance is calculated from echo time
Path clear (distance > 25 cm) — drive both wheels forward
Obstacle detected (distance ≤ 25 cm) — stop immediately
Scan — servo rotates to 0° (right), takes a reading, then to 180° (left), takes another
Decide — compare left vs. right distance; whichever is larger and above the threshold wins
Act — pivot toward the open side: one wheel forward, one wheel backward, for a sharp in-place turn
Escape case — if both sides are blocked, reverse briefly, then pivot right

Distance is calculated from the HC-SR04's echo pulse using:

distance (cm) = echo_duration (µs) / 58

Setup instructions
Wire all components exactly as described in Wiring / pin connections
Install the Arduino IDE
Connect the Arduino to your computer via USB (keep the battery disconnected during upload)
Open the code above in the IDE, or clone this repo and open obstacle_robot.ino
Select Tools → Board → Arduino Uno and Tools → Port → (your COM port)
Click Upload
Once uploading finishes, disconnect USB, connect the battery, and power on
Open Serial Monitor at 9600 baud to watch live distance readings while it runs
Testing results
Test scenario	Expected behavior	Result
No obstacle within 25 cm	Moves forward continuously	Pass
Obstacle at 20 cm, front	Stops, scans, turns away	Pass
Clear path only on right	Turns right after scan	Pass
Clear path only on left	Turns left after scan	Pass
Both sides blocked	Reverses, then pivots right	Pass
Fast-moving obstacle	May not always react in time	Noted limitation

Summary: 96% obstacle-detection accuracy across 40+ live trials · ±2 cm sensor precision · ~90 minutes continuous battery runtime.

Troubleshooting
Symptom	Likely cause	Fix
Sensor always reads 0	Trig/Echo wires swapped, or connected to IOREF/3.3V	Confirm Trig → D9, Echo → D10, both powered from 5V
One motor doesn't spin	Loose wire on that motor's L298N OUT terminal	Reseat the wire; if direction is reversed, swap the two wires on that terminal (no code change needed)
Robot doesn't move at all	Missing common ground	Verify battery (–), L298N GND, and Arduino GND all connect at one point
Servo vibrates but doesn't turn	Powered from IOREF or an underpowered pin	Move servo VCC to Arduino 5V (twist together with sensor VCC if needed)
Robot freezes mid-motion	pulseIn() has no timeout, sensor never returns	Use the timeout version shown in the code above: pulseIn(ECHO_PIN, HIGH, 30000UL)
Future improvements
 Bluetooth (HC-05) or Wi-Fi (ESP8266) module for manual override from a phone
 Side-mounted ultrasonic sensors to remove the scan-and-pause delay
 IR sensors beneath the chassis for line-following mode
 Downward-facing IR sensors for cliff/edge detection
 ESP32-CAM integration for vision-based obstacle recognition
 Cloud logging via ThingSpeak or Blynk for remote monitoring
License

This project is open source and available under the MIT License.

Built by a first-year AI & Data Science student at REVA University, Bengaluru. Questions and contributions welcome — feel free to open an issue.
