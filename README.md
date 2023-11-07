#include <SoftwareSerial.h>
#include <ArduinoJson.h>
#include <ESP8266WiFi.h>
#include <ThingSpeak.h>

SoftwareSerial nodemcu(D6, D5);  // Using D6 as Rx and D5 as Tx

unsigned long previousMillis = 0;
unsigned long currentMillis;
const unsigned long period = 10000;  // Timer to run Arduino code every 10 seconds
const unsigned long timeoutDuration = 1000;  // Timeout duration in milliseconds
const size_t expectedJsonLength = 40;  // Expected length of JSON string

String receivedData;  // String to store received data

// WiFi credentials
const char* ssid = "SOUGAT";
const char* password = "87654321";

// ThingSpeak parameters
unsigned long channelID = 2336326;
const char* writeAPIKey = "RSHANR06U9G8M42R";

void setup() {
    Serial.begin(9600);
    nodemcu.begin(9600);
    while (!Serial);

    // Connect to Wi-Fi
    WiFi.begin(ssid, password);
    ThingSpeak.begin(client);
}

void loop() {
    currentMillis = millis();

    if (currentMillis - previousMillis >= period) {
        receivedData = "";  // Clear the received data string

        unsigned long start = millis();
        while (nodemcu.available() < expectedJsonLength && millis() - start < timeoutDuration) {
            // Wait for the complete JSON string
        }

        while (nodemcu.available()) {
            char c = nodemcu.read();
            receivedData += c;  // Concatenate characters to form a complete JSON string
        }

        if (receivedData.length() >= expectedJsonLength) {
            StaticJsonDocument<200> doc;
            DeserializationError error = deserializeJson(doc, receivedData);

            if (error) {
                Serial.print(F("deserializeJson() failed: "));
                Serial.println(error.c_str());
            } else {
                float averageSpeed = doc["average_speed"];
                float averageDistance = doc["average_distance"];

                Serial.println("JSON Object Received");
                Serial.print("Received Average Speed:  ");
                Serial.println(averageSpeed);
                Serial.print("Received Average Distance:  ");
                Serial.println(averageDistance);
                Serial.println("-----------------------------------------");

                // Update ThingSpeak fields
                ThingSpeak.writeField(channelID, 1, averageSpeed, writeAPIKey);
                ThingSpeak.writeField(channelID, 2, averageDistance, writeAPIKey);
            }
        } else {
            Serial.println("Incomplete or no data received within the timeout.");
        }

        previousMillis = currentMillis;
    }
}
