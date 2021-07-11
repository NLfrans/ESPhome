Make you're old wired doorbell interact with HomeAssistant

Inspiration:
 - https://www.youtube.com/watch?v=p596PnCCIX0
 - https://community.home-assistant.io/t/esphome-voltage-detection-integrate-doorbell/226653/12

Alternative solution:
 - https://frenck.dev/diy-smart-doorbell-for-just-2-dollar/

Components:
 - 5A Sensor Range of Single-Phase Module Ac Current Sensor Module For Arduino https://www.aliexpress.com/item/4000434009123.html
 - NodeMcu
 - HomeAssistant + Arduino
 - USB power

Hardware setup:
 - Put the Current Sensor between the wire of your old doorbell
 - Connect pin A0 and GND to the Current Sensor
 - Power the board with USB

Script (Arduino):
```
//WIFI settings
char ssid[] = "NameOfWifi";
char pass[] = "*****";

//MQTT settings
char mqtt_server[] = "192.168.1.1";
char mqtt_user[] = "mqtt-user";
char mqtt_pass[] = "*****";
char mqtt_client_id[] = "doorbell";

//Doorbell settings
const int doorbell_pin = A0;        //current sensor connected to A0 and ground
int senseDoorbell = 0;              //variable to hold doorbell sensor reading
int debounce = 1000;                //only allow one DingDong per second
unsigned long currentMillis = 0;    //how many milliseconds since the Arduino booted
unsigned long prevRing = 0;         //The last time the doorbell rang

// Base ESP8266
#include <ESP8266WiFi.h>
WiFiClient WIFI_CLIENT;

// MQTT
#include <PubSubClient.h>
PubSubClient MQTT_CLIENT;

// This function runs once on startup
void setup() {
  // Initialize the serial port
  Serial.begin(115200);
  pinMode(doorbell_pin, INPUT);

  WiFi.begin(ssid, pass);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
  }
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
}

void reconnect() {
  // Set our MQTT broker address and port
  MQTT_CLIENT.setServer(mqtt_server, 1883);
  MQTT_CLIENT.setClient(WIFI_CLIENT);

  // Loop until we're reconnected
  while (!MQTT_CLIENT.connected()) {
    // Attempt to connect
    Serial.println("Attempt to connect to MQTT broker");
    MQTT_CLIENT.connect(mqtt_client_id, mqtt_user, mqtt_pass);

    // Wait some time to space out connection requests
    delay(3000);
  }

  Serial.println("MQTT connected");
}

void loop() {

  currentMillis = millis();
  if (currentMillis - prevRing >= debounce) {
    senseDoorbell = analogRead(doorbell_pin);  //read the doorbell sensor
    
    if (senseDoorbell > 50) {              //mine read between 0 and 7 with no current and 200 with it.  50 seemed to be safe.
      Serial.println("DingDong");
      Serial.println(senseDoorbell);
      
      if (!MQTT_CLIENT.connected()) {
        reconnect();
      }
      MQTT_CLIENT.publish("doorbell/press", "True");
      Serial.println("doorbell/press: True");

      delay(3000);
      if (!MQTT_CLIENT.connected()) {
        reconnect();
      }
      MQTT_CLIENT.publish("doorbell/press", "False");
      Serial.println("doorbell/press: False");
      prevRing = currentMillis;            //engage debounce mode!
    }
  }

}
```

Automations Home Assistant
```
1) Make a new automation in Home Assistant
2) Make a trigger
   MQTT
   Topic: doorbell/press
   Payload: True
3) Make a Actions
```
