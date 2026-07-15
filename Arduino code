//  Pin Definitions
#define ENA 9
#define IN1 4
#define IN2 5

#define ENB 10
#define IN3 6
#define IN4 7

// Ultrasonic pins
#define TRIG_F A2
#define ECHO_F A3

#define TRIG_L A0
#define ECHO_L A1

#define TRIG_R A4
#define ECHO_R A5

// Parameters 
const int FRONT_THRESHOLD = 15;   // cm – obstacle distance in front
const int SIDE_THRESHOLD  = 15;   // cm – opening at sides

const int BASE_SPEED  = 100;
const int TURN_SPEED  = 120;

// TODO: put your tuned values here:
const int TURN_DELAY_90    = 440;   // ms – your calibrated 90° time
const int UTURN_DELAY_180  = 880;   // ms – your calibrated 180° time

// ===== Motor Helpers (using your corrected wiring) =====
void stopMotors() {
  analogWrite(ENA, 0);
  analogWrite(ENB, 0);
}

// LEFT forward = IN1 LOW, IN2 HIGH
// RIGHT forward = IN3 HIGH, IN4 LOW
void moveForward(int speed) {
  digitalWrite(IN1, LOW);
  digitalWrite(IN2, HIGH);   // left forward
  digitalWrite(IN3, HIGH);
  digitalWrite(IN4, LOW);    // right forward
  analogWrite(ENA, speed);
  analogWrite(ENB, speed);
}

void moveBackward(int speed) {
  digitalWrite(IN1, HIGH);
  digitalWrite(IN2, LOW);    // left backward
  digitalWrite(IN3, LOW);
  digitalWrite(IN4, HIGH);   // right backward
  analogWrite(ENA, speed);
  analogWrite(ENB, speed);
}

// Turn left in place: left backward, right forward
void turnLeft(int speed) {
  digitalWrite(IN1, HIGH);   // left backward
  digitalWrite(IN2, LOW);
  digitalWrite(IN3, HIGH);   // right forward
  digitalWrite(IN4, LOW);
  analogWrite(ENA, speed);
  analogWrite(ENB, speed);
}

// Turn right in place: left forward, right backward
void turnRight(int speed) {
  digitalWrite(IN1, LOW);    // left forward
  digitalWrite(IN2, HIGH);
  digitalWrite(IN3, LOW);
  digitalWrite(IN4, HIGH);   // right backward
  analogWrite(ENA, speed);
  analogWrite(ENB, speed);
}

// ===== Ultrasonic Helper =====
long readUltrasonic(int trigPin, int echoPin) {
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);

  long duration = pulseIn(echoPin, HIGH, 25000); // 25ms timeout
  if (duration == 0) {
    return 400; // no echo -> far away
  }

  long distance = duration * 0.034 / 2; // cm
  return distance;
}

// ===== Setup =====
void setup() {
  // Motor pins
  pinMode(ENA, OUTPUT);
  pinMode(IN1, OUTPUT);
  pinMode(IN2, OUTPUT);

  pinMode(ENB, OUTPUT);
  pinMode(IN3, OUTPUT);
  pinMode(IN4, OUTPUT);

  // Ultrasonic pins
  pinMode(TRIG_F, OUTPUT);
  pinMode(ECHO_F, INPUT);

  pinMode(TRIG_L, OUTPUT);
  pinMode(ECHO_L, INPUT);

  pinMode(TRIG_R, OUTPUT);
  pinMode(ECHO_R, INPUT);

  Serial.begin(9600);
  stopMotors();
  delay(1000);
}

// ===== Main Loop =====
void loop() {
  long distFront = readUltrasonic(TRIG_F, ECHO_F);
  long distLeft  = readUltrasonic(TRIG_L, ECHO_L);
  long distRight = readUltrasonic(TRIG_R, ECHO_R);

  // Debug prints
  Serial.print("F: "); Serial.print(distFront);
  Serial.print("  L: "); Serial.print(distLeft);
  Serial.print("  R: "); Serial.println(distRight);

  bool frontBlocked = (distFront <= FRONT_THRESHOLD);
  bool leftOpen     = (distLeft  > SIDE_THRESHOLD);
  bool rightOpen    = (distRight > SIDE_THRESHOLD);

  // 1) Move forward until front hits threshold
  if (!frontBlocked) {
    // Just keep going straight
    moveForward(BASE_SPEED);
  } else {
    // 2) Threshold reached -> STOP and DECIDE
    stopMotors();
    delay(150);  // small pause

    if (leftOpen || rightOpen) {
      // 2a. At least one side is open

      if (leftOpen) {
        // Prefer LEFT (LSRB style)
        Serial.println("Turning LEFT 90°");
        turnLeft(TURN_SPEED);
        delay(TURN_DELAY_90);
        stopMotors();
        delay(200);   // wait, then next loop will move forward
      }
      else if (rightOpen) {
        Serial.println("Turning RIGHT 90°");
        turnRight(TURN_SPEED);
        delay(TURN_DELAY_90);
        stopMotors();
        delay(200);
      }
    } else {
      // 2b. Dead end -> U-TURN
      Serial.println("DEAD END -> U-TURN");
      // (optional) back a little to avoid hitting wall
      moveBackward(BASE_SPEED);
      delay(250);
      stopMotors();
      delay(100);

      // 180° turn
      turnRight(TURN_SPEED);
      delay(UTURN_DELAY_180);
      stopMotors();
      delay(200);
    }
  }

  delay(50); // small loop delay
}
