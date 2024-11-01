/*
 *
 * This Arduino UNO R4 code is made available for public use without any restriction
 *
 */

#include <WiFiS3.h>
#include <MQTTClient.h>

const int ledPin = 3;

const int PIN_RED = 9;
const int PIN_GREEN = 10;
const int PIN_BLUE = 11;

const char WIFI_SSID[] = "ITFORGE_UFO";          // CHANGE TO YOUR WIFI SSID
const char WIFI_PASSWORD[] = "";  // CHANGE TO YOUR WIFI PASSWORD

const char MQTT_BROKER_ADRRESS[] = "phycom.it.kmitl.ac.th";
const int MQTT_PORT = 1883;
const char MQTT_CLIENT_ID[] = "Server_1730395795786";  // CHANGE IT AS YOU DESIRE
const char MQTT_USERNAME[] = "";                        // CHANGE IT IF REQUIRED, empty if not required
const char MQTT_PASSWORD[] = "";                        // CHANGE IT IF REQUIRED, empty if not required

// The MQTT topics that Arduino should publish/subscribe
const char PUBLISH_TOPIC[] = "66070011/temp";    // CHANGE IT AS YOU DESIRE
const char SUBSCRIBE_TOPIC[] = "660670011/temp";  // CHANGE IT AS YOU DESIRE

const int PUBLISH_INTERVAL = 5000;  // 5 seconds

WiFiClient network;
MQTTClient mqtt = MQTTClient(256);

unsigned long lastPublishTime = 0;

void setup() {
  Serial.begin(9600);
  pinMode(ledPin, OUTPUT);

  // ULTRASONIC
  pinMode(4, OUTPUT);
  pinMode(2, INPUT);

  // LED
  pinMode(PIN_RED, OUTPUT);
  pinMode(PIN_GREEN, OUTPUT);
  pinMode(PIN_BLUE, OUTPUT);

  digitalWrite(PIN_RED, HIGH);
  digitalWrite(PIN_GREEN, HIGH);
  digitalWrite(PIN_BLUE, HIGH);

  int status = WL_IDLE_STATUS;
  while (status != WL_CONNECTED) {
    Serial.print("Arduino UNO R4 - Attempting to connect to SSID: ");
    Serial.println(WIFI_SSID);
    // Connect to WPA/WPA2 network. Change this line if using open or WEP network:
    status = WiFi.begin(WIFI_SSID);

    // wait 10 seconds for connection:
    delay(10000);
  }
  // print your board's IP address:
  Serial.print("IP Address: ");
  Serial.println(WiFi.localIP());

  connectToMQTT();
}

void loop() {
  mqtt.loop();

  if (millis() - lastPublishTime > PUBLISH_INTERVAL) {
    sendToMQTT();
    lastPublishTime = millis();
  }
}

void connectToMQTT() {
  // Connect to the MQTT broker
  mqtt.begin(MQTT_BROKER_ADRRESS, MQTT_PORT, network);

  // Create a handler for incoming messages
  mqtt.onMessage(messageHandler);

  Serial.print("Arduino UNO R4 - Connecting to MQTT broker");

  while (mqtt.connect(MQTT_CLIENT_ID, MQTT_USERNAME, MQTT_PASSWORD)) {
    Serial.print(".");
    delay(100);
  }
  Serial.println();

  if (!mqtt.connected()) {
    Serial.println("Arduino UNO R4 - MQTT broker Timeout!");
    return;
  }

  // Subscribe to a topic, the incoming messages are processed by messageHandler() function
  if (mqtt.subscribe(SUBSCRIBE_TOPIC))
    Serial.print("Arduino UNO R4 - Subscribed to the topic: ");
  else
    Serial.print("Arduino UNO R4 - Failed to subscribe to the topic: ");

  Serial.println(SUBSCRIBE_TOPIC);
  Serial.println("Arduino UNO R4 - MQTT broker Connected!");
}

void sendToMQTT() {

  // TEMPERATURE MCP9700
  int sensorValue = analogRead(A0);
  float voltage = sensorValue * (5.0 / 1023.0);
  float temperatureC = (voltage * 100) - 32;
  Serial.println((temperatureC) * (0.556));
  String val_str = String(temperatureC);

  //Pointometer
  // int potValue = analogRead(A1);  // Read potentiometer value (0-1023)
  // Serial.println(potValue);  // Print value to Serial Monitor for debugging
  // String val_str = String(potValue);

  // ULTRASONIC
  // digitalWrite(4, HIGH);
  // delayMicroseconds(10);
  // digitalWrite(4, LOW);
  // int pulseWidth = pulseIn(2, HIGH);
  // Serial.print("Pulse Width: ");
  // Serial.println(pulseWidth);
  // long distance = pulseWidth / 29 / 2;
  // Serial.print("Distance: ");
  // Serial.println(distance);

  // String val_str = String(distance);
  // Convert the string to a char array for MQTT publishing
  char messageBuffer[10];
  // val_str.toCharArray(messageBuffer, 10);

  // Publish the message to the MQTT topic
  mqtt.publish(PUBLISH_TOPIC, messageBuffer);

  // Print debug information to the Serial Monitor in one line
  Serial.println("Arduino UNO R4 - sent to MQTT: topic: " + String(PUBLISH_TOPIC) + " | payload: " + String(messageBuffer));
}


void messageHandler(String &topic, String &payload) {
    Serial.println("Arduino UNO R4 - received from MQTT: topic: " + topic + " | payload: " + payload);

    int val = payload.toInt();

    setLED(val);
}

void setLED(int val)
{
    digitalWrite(PIN_RED, HIGH);
    digitalWrite(PIN_GREEN, HIGH);
    digitalWrite(PIN_BLUE, HIGH);

    if (val >= 10 && val <= 25) {
        Serial.println("Setting LED to Green");
        digitalWrite(PIN_GREEN, LOW);
    }
    else if (val >= 26 && val <= 35) {
        Serial.println("Setting LED to Blue");
        digitalWrite(PIN_BLUE, LOW);
    }
    else if (val >= 36 && val <= 50) {
        Serial.println("Setting LED to Red");
        digitalWrite(PIN_RED, LOW);
    } else
    {
      digitalWrite(PIN_RED, HIGH);
      digitalWrite(PIN_GREEN, HIGH);
      digitalWrite(PIN_BLUE, HIGH);
    }
}
