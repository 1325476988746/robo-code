include <SoftwareSerial.h>

// Bluetooth Module Pins (RX, TX)
SoftwareSerial BTSerial(2, 3); 

// Left Wheel Driver Pins (BTS7960)
const int L_EN_REN_1 = 4;
const int RPWM_1 = 5; 
const int LPWM_1 = 6; 

// Right Wheel Driver Pins (BTS7960)
const int L_EN_REN_2 = 7;
const int RPWM_2 = 9; 
const int LPWM_2 = 10; 

// Claw Driver Channel A Pins (L298N - Lift Motor)
const int IN1 = 8;
const int IN2 = 11;

// Claw Driver Channel B Pins (L298N - Grip Motor)
const int IN3 = 12;
const int IN4 = 13;

void setup() {
  // Initialize communication
  BTSerial.begin(9600);
  Serial.begin(9600);
  
  // Configure Driver Pins
  pinMode(L_EN_REN_1, OUTPUT);
  pinMode(RPWM_1, OUTPUT);
  pinMode(LPWM_1, OUTPUT);
  
  pinMode(L_EN_REN_2, OUTPUT);
  pinMode(RPWM_2, OUTPUT);
  pinMode(LPWM_2, OUTPUT);
  
  pinMode(IN1, OUTPUT);
  pinMode(IN2, OUTPUT);
  pinMode(IN3, OUTPUT);
  pinMode(IN4, OUTPUT);
  
  // Enable BTS7960 Drivers
  digitalWrite(L_EN_REN_1, HIGH);
  digitalWrite(L_EN_REN_2, HIGH);
  
  // Initialize all motors to stop status
  stopRobot();
  stopClaw();
}

void loop() {
  if (BTSerial.available()) {
    char command = BTSerial.read();
    executeCommand(command);
  }
}

void executeCommand(char cmd) {
  switch (cmd) {
    // === Drivetrain Controls ===
    case 'F': // Move Forward (Both sides full speed)
      moveForward(255, 255); 
      break;
    case 'B': // Move Backward (Both sides full speed)
      moveBackward(255, 255);
      break;
    case 'L': // Turn Left (Left wheels slower, Right wheels faster)
      moveForward(100, 255); 
      break;
    case 'R': // Turn Right (Left wheels faster, Right wheels slower)
      moveForward(255, 100);
      break;
      
    // === Claw Channel A: Lift Motor ===
    case 'U': // Claw Up / Lift
      digitalWrite(IN1, HIGH);
      digitalWrite(IN2, LOW);
      break;
    case 'D': // Claw Down / Lower
      digitalWrite(IN1, LOW);
      digitalWrite(IN2, HIGH);
      break;
      
    // === Claw Channel B: Grip Motor ===
    case 'O': // Open Grip
      digitalWrite(IN3, HIGH);
      digitalWrite(IN4, LOW);
      break;
    case 'C': // Close Grip
      digitalWrite(IN3, LOW);
      digitalWrite(IN4, HIGH);
      break;
      
    // === Cut-Off Safety Stops ===
    case 'W': // Stop Claw Movement (Both channels grounded out)
      stopClaw();
      break;
    case 'S': // Stop Robot Motion
    default:
      stopRobot();
      break;
  }
}

void moveForward(int leftSpeed, int rightSpeed) {
  analogWrite(RPWM_1, leftSpeed);
  analogWrite(LPWM_1, 0);
  analogWrite(RPWM_2, rightSpeed);
  analogWrite(LPWM_2, 0);
}

void moveBackward(int leftSpeed, int rightSpeed) {
  analogWrite(RPWM_1, 0);
  analogWrite(LPWM_1, leftSpeed);
  analogWrite(RPWM_2, 0);
  analogWrite(LPWM_2, rightSpeed);
}

void stopRobot() {
  analogWrite(RPWM_1, 0);
  analogWrite(LPWM_1, 0);
  analogWrite(RPWM_2, 0);
  analogWrite(LPWM_2, 0);
}

void stopClaw() {
  digitalWrite(IN1, LOW);
  digitalWrite(IN2, LOW);
  digitalWrite(IN3, LOW);
  digitalWrite(IN4, LOW);
}
Use code with caution.
