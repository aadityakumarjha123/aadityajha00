// SMART VACUUM CLEANER ROBOT – ARDUINO UNO
// Compatible with: Arduino Uno, Nano, Mega
// Control via: Bluetooth (HC-05/HC-06), Serial Monitor
// Features: 4 motors, vacuum, servo, ultrasonic, auto mode
// NOTE: DHT sensor removed for simpler compilation

#include <Servo.h>
#include <SoftwareSerial.h>

// ========== MOTOR DRIVER PINS (L298N) ==========
#define IN1 2   // Left motor forward
#define IN2 3   // Left motor backward
#define IN3 4   // Right motor forward
#define IN4 5   // Right motor backward
#define ENA 9   // Left motor speed (PWM)
#define ENB 10  // Right motor speed (PWM)

// ========== VACUUM MOTOR ==========
#define VACUUM_PIN 8

// ========== SERVO ==========
Servo myServo;
#define SERVO_PIN 6

// ========== ULTRASONIC SENSOR (HC-SR04) ==========
#define TRIG_PIN 12
#define ECHO_PIN 13

// ========== MOISTURE/DUST SENSOR ==========
#define MOISTURE_PIN A0

// ========== BLUETOOTH MODULE (HC-05/HC-06) ==========
// Connect: BT TX -> Pin 11, BT RX -> Pin 10 (through voltage divider)
SoftwareSerial bluetooth(10, 11); // RX, TX

// ========== LED INDICATOR ==========
#define LED_PIN 13

// ========== GLOBAL VARIABLES ==========
int motorSpeed = 200;        // Speed: 0-255
int servoAngle = 90;         // Servo angle
bool autoMode = false;       // Auto cleaning mode
bool vacuumState = false;    // Vacuum on/off
unsigned long lastSensorRead = 0;
const unsigned long sensorInterval = 2000;

// ========== MOTOR CONTROL FUNCTIONS ==========
void setMotorSpeed(int speed) {
  motorSpeed = constrain(speed, 0, 255);
  analogWrite(ENA, motorSpeed);
  analogWrite(ENB, motorSpeed);
}

void forward() {
  digitalWrite(IN1, HIGH);
  digitalWrite(IN2, LOW);
  digitalWrite(IN3, HIGH);
  digitalWrite(IN4, LOW);
  setMotorSpeed(motorSpeed);
}

void backward() {
  digitalWrite(IN1, LOW);
  digitalWrite(IN2, HIGH);
  digitalWrite(IN3, LOW);
  digitalWrite(IN4, HIGH);
  setMotorSpeed(motorSpeed);
}

void turnLeft() {
  digitalWrite(IN1, LOW);
  digitalWrite(IN2, HIGH);
  digitalWrite(IN3, HIGH);
  digitalWrite(IN4, LOW);
  setMotorSpeed(motorSpeed);
}

void turnRight() {
  digitalWrite(IN1, HIGH);
  digitalWrite(IN2, LOW);
  digitalWrite(IN3, LOW);
  digitalWrite(IN4, HIGH);
  setMotorSpeed(motorSpeed);
}

void stopMotors() {
  digitalWrite(IN1, LOW);
  digitalWrite(IN2, LOW);
  digitalWrite(IN3, LOW);
  digitalWrite(IN4, LOW);
  analogWrite(ENA, 0);
  analogWrite(ENB, 0);
}

// ========== VACUUM CONTROL ==========
void vacuumOn() {
  digitalWrite(VACUUM_PIN, HIGH);
  vacuumState = true;
  Serial.println("Vacuum ON");
}

void vacuumOff() {
  digitalWrite(VACUUM_PIN, LOW);
  vacuumState = false;
  Serial.println("Vacuum OFF");
}

// ========== SERVO CONTROL ==========
void moveServo(int angle) {
  servoAngle = constrain(angle, 0, 180);
  myServo.write(servoAngle);
  Serial.print("Servo: ");
  Serial.println(servoAngle);
}

// ========== ULTRASONIC DISTANCE SENSOR ==========
long getDistance() {
  digitalWrite(TRIG_PIN, LOW);
  delayMicroseconds(2);
  digitalWrite(TRIG_PIN, HIGH);
  delayMicroseconds(10);
  digitalWrite(TRIG_PIN, LOW);
  
  long duration = pulseIn(ECHO_PIN, HIGH, 30000);
  long distance = duration * 0.034 / 2;  // Convert to cm
  
  return (distance > 0 && distance < 400) ? distance : 400;
}

// ========== AUTO CLEANING MODE ==========
void autoClean() {
  long dist = getDistance();
  
  if (dist < 20) {  // Obstacle detected within 20cm
    Serial.println("Obstacle detected!");
    stopMotors();
    delay(300);
    
    backward();
    delay(500);
    
    // Random turn direction
    if (random(0, 2) == 0) {
      Serial.println("Turning left...");
      turnLeft();
    } else {
      Serial.println("Turning right...");
      turnRight();
    }
    delay(random(500, 1000));
    
  } else {
    forward();
  }
}

// ========== SENSOR READINGS ==========
void readAndDisplaySensors() {
  int moisture = analogRead(MOISTURE_PIN);
  long distance = getDistance();
  
  Serial.println("========== SENSOR DATA ==========");
  
  Serial.print("Moisture/Dust: ");
  Serial.println(moisture);
  
  Serial.print("Distance: ");
  Serial.print(distance);
  Serial.println(" cm");
  
  Serial.print("Motor Speed: ");
  Serial.print(motorSpeed);
  Serial.println("/255");
  
  Serial.print("Vacuum: ");
  Serial.println(vacuumState ? "ON" : "OFF");
  
  Serial.print("Mode: ");
  Serial.println(autoMode ? "AUTO" : "MANUAL");
  
  Serial.print("Servo Angle: ");
  Serial.println(servoAngle);
  
  Serial.println("=================================\n");
}

// ========== COMMAND PROCESSING ==========
void processCommand(char cmd) {
  cmd = toupper(cmd);  // Convert to uppercase
  
  switch(cmd) {
    // Movement commands
    case 'F':  // Forward
      if (!autoMode) {
        forward();
        Serial.println("Moving Forward");
      }
      break;
      
    case 'B':  // Backward
      if (!autoMode) {
        backward();
        Serial.println("Moving Backward");
      }
      break;
      
    case 'L':  // Left
      if (!autoMode) {
        turnLeft();
        Serial.println("Turning Left");
      }
      break;
      
    case 'R':  // Right
      if (!autoMode) {
        turnRight();
        Serial.println("Turning Right");
      }
      break;
      
    case 'S':  // Stop
      stopMotors();
      autoMode = false;
      Serial.println("STOP");
      break;
    
    // Speed control
    case '1':  // Low speed
      motorSpeed = 150;
      Serial.println("Speed: LOW");
      break;
      
    case '2':  // Medium speed
      motorSpeed = 200;
      Serial.println("Speed: MEDIUM");
      break;
      
    case '3':  // High speed
      motorSpeed = 255;
      Serial.println("Speed: HIGH");
      break;
    
    // Vacuum control
    case 'V':  // Vacuum ON
      vacuumOn();
      break;
      
    case 'v':  // Vacuum OFF (lowercase)
      vacuumOff();
      break;
    
    // Servo control
    case 'U':  // Servo Up
      moveServo(servoAngle + 15);
      break;
      
    case 'D':  // Servo Down
      moveServo(servoAngle - 15);
      break;
      
    case 'C':  // Servo Center
      moveServo(90);
      break;
    
    // Auto mode
    case 'A':  // Auto mode ON
      autoMode = true;
      vacuumOn();
      Serial.println("AUTO MODE ACTIVATED");
      break;
      
    case 'a':  // Auto mode OFF (lowercase)
      autoMode = false;
      stopMotors();
      Serial.println("MANUAL MODE");
      break;
    
    // Sensor reading
    case 'I':  // Info/Sensors
      readAndDisplaySensors();
      break;
    
    // Help
    case 'H':  // Help
      printHelp();
      break;
    
    // Scan area
    case 'X':  // Scan surroundings
      scanArea();
      break;
      
    default:
      Serial.println("Unknown command. Send 'H' for help.");
      break;
  }
}

// ========== HELP MENU ==========
void printHelp() {
  Serial.println("\n========== COMMAND LIST ==========");
  Serial.println("MOVEMENT:");
  Serial.println("  F - Forward");
  Serial.println("  B - Backward");
  Serial.println("  L - Turn Left");
  Serial.println("  R - Turn Right");
  Serial.println("  S - Stop");
  Serial.println("\nSPEED:");
  Serial.println("  1 - Low Speed (150)");
  Serial.println("  2 - Medium Speed (200)");
  Serial.println("  3 - High Speed (255)");
  Serial.println("\nVACUUM:");
  Serial.println("  V - Vacuum ON");
  Serial.println("  v - Vacuum OFF");
  Serial.println("\nSERVO:");
  Serial.println("  U - Servo Up");
  Serial.println("  D - Servo Down");
  Serial.println("  C - Servo Center");
  Serial.println("\nMODE:");
  Serial.println("  A - Auto Clean ON");
  Serial.println("  a - Auto Clean OFF");
  Serial.println("\nINFO:");
  Serial.println("  I - Display Sensors");
  Serial.println("  X - Scan Area");
  Serial.println("  H - Help Menu");
  Serial.println("==================================\n");
}

// ========== SETUP ==========
void setup() {
  // Serial communication
  Serial.begin(9600);
  bluetooth.begin(9600);
  
  Serial.println("\n\n========================================");
  Serial.println("  SMART VACUUM ROBOT - ARDUINO UNO");
  Serial.println("         (No DHT - Simplified)");
  Serial.println("========================================\n");
  
  // Motor pins
  pinMode(IN1, OUTPUT);
  pinMode(IN2, OUTPUT);
  pinMode(IN3, OUTPUT);
  pinMode(IN4, OUTPUT);
  pinMode(ENA, OUTPUT);
  pinMode(ENB, OUTPUT);
  
  // Vacuum
  pinMode(VACUUM_PIN, OUTPUT);
  
  // Ultrasonic sensor
  pinMode(TRIG_PIN, OUTPUT);
  pinMode(ECHO_PIN, INPUT);
  
  // LED
  pinMode(LED_PIN, OUTPUT);
  
  // Servo
  myServo.attach(SERVO_PIN);
  myServo.write(90);
  
  // Initialize motors to stop
  stopMotors();
  vacuumOff();
  
  Serial.println("System Ready!");
  Serial.println("Send 'H' for command list\n");
  printHelp();
}

// ========== MAIN LOOP ==========
void loop() {
  // Check for Serial commands (USB)
  if (Serial.available()) {
    char cmd = Serial.read();
    processCommand(cmd);
  }
  
  // Check for Bluetooth commands
  if (bluetooth.available()) {
    char cmd = bluetooth.read();
    processCommand(cmd);
    
    // Echo to Serial monitor
    Serial.print("BT Command: ");
    Serial.println(cmd);
  }
  
  // Auto cleaning mode
  if (autoMode) {
    autoClean();
    digitalWrite(LED_PIN, HIGH);
  } else {
    digitalWrite(LED_PIN, LOW);
  }
  
  // Periodic sensor reading
  if (millis() - lastSensorRead >= sensorInterval && !autoMode) {
    lastSensorRead = millis();
    // Optional: Uncomment to auto-display sensors every 2 seconds
    // readAndDisplaySensors();
  }
  
  delay(50);  // Small delay for stability
}

// ========== AREA SCANNING ==========
void scanArea() {
  Serial.println("Scanning area...");
  
  for (int angle = 0; angle <= 180; angle += 30) {
    myServo.write(angle);
    delay(300);
    long dist = getDistance();
    Serial.print("Angle ");
    Serial.print(angle);
    Serial.print(": ");
    Serial.print(dist);
    Serial.println(" cm");
  }
  
  myServo.write(90);  // Return to center
  Serial.println("Scan complete!\n");
}

// ========== EMERGENCY STOP ==========
void emergencyStop() {
  stopMotors();
  vacuumOff();
  autoMode = false;
  Serial.println("!!! EMERGENCY STOP !!!");
}
