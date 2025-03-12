#include <AFMotor.h>
#include <SoftwareSerial.h>

// Define motor pins
AF_DCMotor front_left(1, MOTOR12_64KHZ);
AF_DCMotor rear_left(2, MOTOR12_64KHZ);
AF_DCMotor front_right(3, MOTOR12_64KHZ);
AF_DCMotor rear_right(4, MOTOR12_64KHZ);

// Define ultrasonic sensor pins
#define echoPin 9
#define trigPin 10 

// Define GSM Module (SIM800L)
SoftwareSerial gsmSerial(7, 8); // RX, TX for GSM module

// Define GPS Module (NEO-6M)
SoftwareSerial gpsSerial(4, 5); // RX, TX for GPS

int count = 1;
int direction = 1;
int distance = 0;

void setup() {
    Serial.begin(9600);
    gsmSerial.begin(9600);
    gpsSerial.begin(9600);

    pinMode(trigPin, OUTPUT);
    pinMode(echoPin, INPUT);

    Serial.println("System Started...");
    delay(3000);
}

void loop() {
    // Measure distance using ultrasonic sensor
    digitalWrite(trigPin, LOW);
    delayMicroseconds(2);
    digitalWrite(trigPin, HIGH);
    delayMicroseconds(10);
    digitalWrite(trigPin, LOW);

    int duration = pulseIn(echoPin, HIGH);
    distance = (duration * 0.034) / 2;

    Serial.print("Distance: ");
    Serial.print(distance);
    Serial.println(" cm");

    if (distance < 10) {  // If an obstacle is detected
        sendAlertSMS();
        captureImage();
        changeDirection();
    } else {
        moveForward();
    }

    delay(500);
}

void moveForward() {
    front_left.setSpeed(255);
    rear_left.setSpeed(255);
    front_right.setSpeed(255);
    rear_right.setSpeed(255);

    front_left.run(FORWARD);
    rear_left.run(FORWARD);
    front_right.run(FORWARD);
    rear_right.run(FORWARD);
}

void changeDirection() {
    front_left.setSpeed(255);
    rear_left.setSpeed(255);
    front_right.setSpeed(255);
    rear_right.setSpeed(255);

    front_left.run(BACKWARD);
    rear_left.run(BACKWARD);
    front_right.run(BACKWARD);
    rear_right.run(BACKWARD);

    delay(1000); // Move back for a while
}

void sendAlertSMS() {
    gsmSerial.println("AT+CMGF=1");  // Set SMS mode
    delay(100);
    gsmSerial.println("AT+CMGS=\"+91XXXXXXXXXX\"");  // Replace with your mobile number
    delay(100);
    gsmSerial.println("Obstacle detected! Robot is changing direction.");
    delay(100);
    gsmSerial.write(26);  // End of SMS
    delay(1000);
    Serial.println("SMS Sent!");
}

void captureImage() {
    Serial.println("Capturing Image... (Camera module required)");
    // This function will work with ESP32-CAM or OV7670
}

