#include <ESP8266WiFi.h>
#include <ESP8266WebServer.h>

const char* ssid = "your_SSID";
const char* password = "your_PASSWORD";

ESP8266WebServer server(80);

int gpio2State = LOW;
int gpio4State = LOW;
int gpio33State = LOW;

// Timer for auto-reset after 1 minute
unsigned long lastChange2 = 0;
unsigned long lastChange4 = 0;
unsigned long lastChange33 = 0;

void setup() {
  Serial.begin(115200);

  // Connect to WiFi
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.println("Connecting to WiFi...");
  }
  Serial.println("Connected to WiFi");

  // Setup GPIOs
  pinMode(2, OUTPUT);
  pinMode(4, OUTPUT);
  pinMode(33, OUTPUT);

  // Start the web server
  server.on("/update", handleUpdate);
  server.on("/status", handleStatus);
  server.begin();
}

void loop() {
  server.handleClient();
  checkAutoReset();
}

// Handle update requests
void handleUpdate() {
  String output = server.arg("output");
  String state = server.arg("state");

  if (output == "2") {
    gpio2State = state.toInt();
    digitalWrite(2, gpio2State);
    lastChange2 = millis();  // Reset timer
  } else if (output == "4") {
    gpio4State = state.toInt();
    digitalWrite(4, gpio4State);
    lastChange4 = millis();  // Reset timer
  } else if (output == "33") {
    gpio33State = state.toInt();
    digitalWrite(33, gpio33State);
    lastChange33 = millis();  // Reset timer
  }

  server.send(200, "text/plain", "OK");
}

// Serve the current status of the switches
void handleStatus() {
  String response = "{\"gpio2\": " + String(gpio2State) +
                    ", \"gpio4\": " + String(gpio4State) +
                    ", \"gpio33\": " + String(gpio33State) + "}";
  server.send(200, "application/json", response);
}

// Automatically reset the GPIO after 1 minute
void checkAutoReset() {
  if (millis() - lastChange2 >= 60000 && gpio2State == HIGH) {
    gpio2State = LOW;
    digitalWrite(2, LOW);
  }
  if (millis() - lastChange4 >= 60000 && gpio4State == HIGH) {
    gpio4State = LOW;
    digitalWrite(4, LOW);
  }
  if (millis() - lastChange33 >= 60000 && gpio33State == HIGH) {
    gpio33State = LOW;
    digitalWrite(33, LOW);
  }
}
