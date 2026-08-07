# BlueStamp Bluetooth Controlled Gesture Robot
This project is a Bluetooth controlled robot that drives based on hand gestures. One of the two parts of the robot is the glove/hand part that has a Ardiuno Nano 33 BLE Sense and an accelerometer to read the tilt of my hand and then sends data wirelessly using a HC-05 Bluetooth module. The 2nd part of the project, the robot uses an Ardiuno Uno, a motor driver and another HC-05 Bluetooth module. 


| **Engineer** | **School** | **Area of Interest** | **Grade** |
|:--:|:--:|:--:|:--:|
| Adarsh M | Flint Hill  | Mechanical Engineering | Incoming Junior

**Replace the BlueStamp logo below with an image of yourself and your completed project. Follow the guide [here](https://tomcam.github.io/least-github-pages/adding-images-github-pages-site.html) if you need help.**

![Headstone Image](logo.svg)
  
# Final Milestone

<iframe width="560" height="315" src="https://www.youtube.com/embed/OcDhmMhwXHM" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
With my final milestone completed the project its officially complete! Since milestone 2 I've added a new robotic hand that uses a servo motor to open and close when I press a button that's on the hand module. Overall my biggest challenge at BSE was time, because of how finicky the Bluetooth was getting it to work took a majority of my time on the project, but in the end I was still able to complete and be proud of my project. One of the biggest things I learned was how Bluetooth works and it is super interesting and complicated. In the future I want to apply the skills I learned at BSE to a fully personal project, not connected to school or a summer program, just something for myself.

For your final milestone, explain the outcome of your project. Key details to include are:
- What you've accomplished since your previous milestone
- What your biggest challenges and triumphs were at BSE
- A summary of key topics you learned about
- What you hope to learn in the future after everything you've learned at BSE



# Second Milestone

<iframe width="560" height="315" src="https://www.youtube.com/embed/745LfcR1r0Y" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
For my second milestone I fully constructed the hand portion of the project and got the HC-05 Bluetooth modules working on and talking on both of the parts. A challenge I faced for this part was getting the Bluetooth modules synced to each other, I solved this by reconfiguring them multiple times and during this process I learned more about how they work. Another challenge I faced was the code, the original code was meant for a different motor driver so I had to modify it so it would work for my setup. For my next steps I want to add a robotic hand to the robot so that it can grab things and interact with its surroundings.


# First Milestone

First Milestone
<iframe width="560" height="315" src="https://www.youtube.com/embed/m1lT-EZTn98" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
For my first milestone, I got the robot chassis moving using basic Arduino code, testing forward, backward, left, and right movements to confirm that the motor wiring and control logic were working correctly. My biggest challenge was untangling the motor wiring — sorting out which wires controlled speed versus direction for the left and right sides — since a mix-up here made my code behave unpredictably at first. Once I traced and corrected the wiring, the robot responded smoothly to each command. Next, I plan to build the handheld glove controller, which will use an accelerometer and a second Bluetooth adapter to send gesture-based commands wirelessly to the robot — for example, tilting the glove forward to drive the robot forward and tilting it right to turn right.

# Schematics 

(
# Code

UNO code
```c++
/*
  ===================================================================
  COMBINED SKETCH — Model Y 2.0 Motor Test + Bluetooth Claw Control
  ===================================================================
  Motors (Channel B test, BK1-4 sockets):
    Left side  (BK1/BK2): ENA=10, IN1=9, IN2=8
    Right side (BK3/BK4): ENB=5,  IN3=6, IN4=7

  Bluetooth (HC-05 via SoftwareSerial) + Claw Servo:
    BT_RX_PIN = 2   (Uno pin <- HC-05 TXD, safe as-is)
    BT_TX_PIN = 3   (Uno pin -> HC-05 RXD, NEEDS a voltage divider)
    SERVO_PIN = 13  (any free PWM-capable pin)

  Sends 'T' over Bluetooth to toggle the claw open/closed.

  NOTE ON TIMING:
  The motor test uses delay() calls, which block execution. During
  those delays, incoming Bluetooth bytes queue up in the SoftwareSerial
  buffer but aren't processed until the delay ends. checkBluetooth()
  is called between each motor phase so the claw responds as promptly
  as this blocking structure allows. If you need instant claw response
  even mid-motor-move, the loop would need to be rewritten to use
  millis() timing instead of delay() — let me know if you want that
  version instead.
  ===================================================================
*/

#include <Servo.h>
#include <SoftwareSerial.h>

// ---------------- Motor pins ----------------
#define enLeft   10
#define inLeft_1 9
#define inLeft_2 8

#define enRight   5
#define inRight_1 6
#define inRight_2 7

int Speed = 150;
int testTime = 2000;
int pauseTime = 1500;

// ---------------- Bluetooth / Servo pins ----------------
const int BT_RX_PIN = 2;    // Uno pin <- HC-05 TXD
const int BT_TX_PIN = 3;    // Uno pin -> HC-05 RXD (voltage divider needed)
const int SERVO_PIN  = 13;  // claw servo

SoftwareSerial btSerial(BT_RX_PIN, BT_TX_PIN); // RX, TX
Servo clawServo;

const int OPEN_ANGLE = 90;    // adjust to whatever fully opens your claw
const int CLOSED_ANGLE = 0;   // adjust to whatever fully closes your claw
bool clawOpen = true;

void setup() {
  Serial.begin(9600);         // USB serial, for debugging only

  // Motors
  pinMode(enLeft, OUTPUT); pinMode(inLeft_1, OUTPUT); pinMode(inLeft_2, OUTPUT);
  pinMode(enRight, OUTPUT); pinMode(inRight_1, OUTPUT); pinMode(inRight_2, OUTPUT);
  stopAll();

  // Bluetooth + Servo
  btSerial.begin(38400);      // must match HC-05 baud rate
  clawServo.attach(SERVO_PIN);
  clawServo.write(OPEN_ANGLE);

  delay(2000);
}

void loop() {
  Serial.println("LEFT side FORWARD");
  driveSide(enLeft, inLeft_1, inLeft_2, 1);
  delay(testTime);
  stopAll();
  checkBluetooth();
  delay(pauseTime);

  Serial.println("LEFT side BACKWARD");
  driveSide(enLeft, inLeft_1, inLeft_2, -1);
  delay(testTime);
  stopAll();
  checkBluetooth();
  delay(pauseTime);

  Serial.println("RIGHT side FORWARD");
  driveSide(enRight, inRight_1, inRight_2, 1);
  delay(testTime);
  stopAll();
  checkBluetooth();
  delay(pauseTime);

  Serial.println("RIGHT side BACKWARD");
  driveSide(enRight, inRight_1, inRight_2, -1);
  delay(testTime);
  stopAll();
  checkBluetooth();
  delay(pauseTime);

  Serial.println("Loop complete. Pausing before repeat...");
  checkBluetooth();
  delay(4000);
}

// ---------------- Motor helpers ----------------
void driveSide(int enPin, int in1Pin, int in2Pin, int dir) {
  if (dir == 1) {
    digitalWrite(in1Pin, LOW);
    digitalWrite(in2Pin, HIGH);
  } else if (dir == -1) {
    digitalWrite(in1Pin, HIGH);
    digitalWrite(in2Pin, LOW);
  } else {
    digitalWrite(in1Pin, LOW);
    digitalWrite(in2Pin, LOW);
  }
  analogWrite(enPin, (dir == 0) ? 0 : Speed);
}

void stopAll() {
  driveSide(enLeft, inLeft_1, inLeft_2, 0);
  driveSide(enRight, inRight_1, inRight_2, 0);
}

// ---------------- Bluetooth / claw helper ----------------
void checkBluetooth() {
  while (btSerial.available()) {
    char c = btSerial.read();
    if (c == 'T') {
      clawOpen = !clawOpen;
      clawServo.write(clawOpen ? OPEN_ANGLE : CLOSED_ANGLE);
      Serial.println(clawOpen ? "Claw: OPEN" : "Claw: CLOSED");
    }
  }
}

```
Nano code

```c++
/*
  ===================================================================
  NANO 33 BLE SENSE — COMBINED TRANSMITTER
  Button toggle ('T') + IMU gesture commands (f/b/l/r/s)
  Sent over HC-05 wired to Serial1 (pins D0/D1).
  ===================================================================
  Wiring:
    Button:  one leg -> D2, other leg -> GND  (uses internal pull-up)
    HC-05:   VCC -> 5V or 3.3V per YOUR module's spec
             GND -> GND
             TXD -> Nano D0 (RX)
             RXD -> Nano D1 (TX)   <-- check logic-level requirements

  HC-05 baud rate: 38400 (confirmed). This matches the Uno claw
  receiver sketch's btSerial.begin(38400) — no changes needed there.
  ===================================================================
*/

#include <Arduino_BMI270_BMM150.h>

// ---------------- Bluetooth ----------------
#define BT_Serial Serial1
const long BT_BAUD = 38400;

// ---------------- Button ----------------
const int BUTTON_PIN = 2;   // change if your free pin differs

bool lastReading = HIGH;          // HIGH = not pressed (INPUT_PULLUP)
int stableState = HIGH;
unsigned long lastChangeTime = 0;
const unsigned long DEBOUNCE_MS = 50;

// ---------------- IMU / gestures ----------------
float x, y, z;
int flag = 0;

void setup() {
  Serial.begin(115200);        // USB Serial Monitor, debugging only
  while (!Serial);

  pinMode(BUTTON_PIN, INPUT_PULLUP);

  BT_Serial.begin(BT_BAUD);    // HC-05 data mode

  Serial.println("Nano 33 BLE Sense Controller (button + IMU)");

  if (!IMU.begin()) {
    Serial.println("Failed to initialize IMU!");
    while (1);
  }
  Serial.println("IMU Ready");
}

void loop() {
  checkButton();
  Read_accelerometer();
  checkGestures();
  delay(100);
}

// ---------------- Button handling ----------------
void checkButton() {
  int reading = digitalRead(BUTTON_PIN);

  if (reading != lastReading) {
    lastChangeTime = millis();
  }

  if ((millis() - lastChangeTime) > DEBOUNCE_MS) {
    if (reading != stableState) {
      stableState = reading;
      if (stableState == LOW) {     // button just pressed (active LOW)
        BT_Serial.print('T');
        Serial.println("Sent toggle command: T");
      }
    }
  }

  lastReading = reading;
}

// ---------------- IMU handling ----------------
void Read_accelerometer() {
  if (IMU.accelerationAvailable()) {
    IMU.readAcceleration(x, y, z);
    Serial.print("X: ");
    Serial.print(x);
    Serial.print("\tY: ");
    Serial.print(y);
    Serial.print("\tZ: ");
    Serial.println(z);
  }
}

void checkGestures() {
  // Forward
  if (x < -0.8 && flag == 0) {
    flag = 1;
    BT_Serial.write('f');
    Serial.println("Forward");
  }
  // Backward
  if (x > 0.8 && flag == 0) {
    flag = 1;
    BT_Serial.write('b');
    Serial.println("Backward");
  }
  // Left
  if (y < -0.8 && flag == 0) {
    flag = 1;
    BT_Serial.write('l');
    Serial.println("Left");
  }
  // Right
  if (y > 0.8 && flag == 0) {
    flag = 1;
    BT_Serial.write('r');
    Serial.println("Right");
  }
  // Stop
  if ((x > -0.3 && x < 0.3) &&
      (y > -0.3 && y < 0.3) &&
      flag == 1) {
    flag = 0;
    BT_Serial.write('s');
    Serial.println("Stop");
  }
}

```
# Bill of Materials

| **Part** | **Note** | **Price** | **Link** |
|:--:|:--:|:--:|:--:|
| Car Chassis Kit | Motorized base with wheels and motors that the rest of the electronics are mounted to | $39.99 | <a href="https://www.amazon.com/dp/B0DJ7BT1V5"> Link </a> |
| Screwdriver Kit | Used to assemble the chassis and fasten components in place | $5.94 | <a href="https://www.amazon.com/Small-Screwdriver-Set-Mini-Magnetic/dp/B08RYXKJW9/"> Link </a> |
| Arduino Uno Clone | Microcontroller onboard the car that drives the motors based on received commands | $14.98 | <a href="https://www.amazon.com/ELEGOO-Board-ATmega328P-ATMEGA16U2-Compliant/dp/B01EWOE0UU/"> Link </a> |
| Electronics Kit | Assorted resistors, wires, and small components used for prototyping the circuit | $14 | <a href="https://www.amazon.com/Smraza-Electronics-Potentiometer-tie-Points-Breadboard/dp/B0B62RL725/"> Link </a> |
| Breadboard Kit | Solderless breadboards used to wire and test the circuit before final assembly | $8.79 | <a href="https://www.amazon.com/Breadboards-Solderless-Breadboard-Distribution-Connecting/dp/B07DL13RZH/"> Link </a> |
| Arduino Nano 33 BLE Sense | Microcontroller with a built-in accelerometer, used as the handheld motion controller | $39.7 | <a href="https://www.amazon.com/Arduino-Nano-Sense-headers-ABX00070/dp/B0BQHZ88WD/"> Link </a> |
| Micro USB Cable | Used to program and power the Arduino Uno | $5 | <a href="https://www.amazon.com/Charging-Transfer-Android-Trustable-MYFON/dp/B098DW7485/"> Link </a> |
| Accelerometer | Measures tilt/motion, used to generate steering or movement commands | $9 | <a href="https://www.amazon.com/dp/B0D2TJVMNY"> Link </a> |
| HC05 | Bluetooth module that wirelessly links the controller and the car | $9 | <a href="https://www.amazon.com/DSD-TECH-HC-05-Pass-through-Communication/dp/B01G9KSAF6/"> Link </a> |
| Breadboard Power Supply | Regulates and supplies power to the breadboard circuit | $8 | <a href="https://www.amazon.com/ALAMSCN-Solderless-Breadboard-Battery-Arduino/dp/B08JYPMCZY/"> Link </a> |
| 9V Batteries | Powers the chassis motors and/or breadboard circuit | $8.69 | <a href="https://www.amazon.com/Amazon-Basics-Performance-All-Purpose-Batteries/dp/B00MH4QM1S/"> Link </a> |
| Velcro Tape | Secures components (battery, breadboard, etc.) to the chassis | $8 | <a href="https://www.amazon.com/Art3d-Sticky-Double-Sided-Command-Adhesive/dp/B0B58FGF8H/"> Link </a> |
| DMM | Digital multimeter used to test voltage, continuity, and debug wiring | $9.99 | <a href="https://www.amazon.com/dp/B0CXM242J1"> Link </a> |







