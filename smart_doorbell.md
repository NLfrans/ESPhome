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
//WIFI
char ssid[] = "NameOfWifi";
char pass[] = "*****";

//MQTT
char mqtt_server[]    = "192.168.1.1";
int  mqtt_port        = 1883;
char mqtt_user[]      = "mqtt-user";
char mqtt_pass[]      = "*****";
char mqtt_client_id[] = "doorbell";

//Doorbell settings
const int doorbell_pin       = A0;  //current sensor connected to A0 and ground
int interval                 = 500; //only allow one DingDong per second
unsigned long previousMillis = 0;   //The last time the doorbell rang

#include <ArduinoMqttClient.h>
#include <ESP8266WiFi.h>


WiFiClient wifiClient;
MqttClient mqttClient(wifiClient);

void setup() {
  //Initialize serial and wait for port to open:
  Serial.begin(115200);
  pinMode(doorbell_pin, INPUT);
  pinMode(D1, INPUT);///
  while (!Serial) {
    ; // wait for serial port to connect. Needed for native USB port only
  }
  
  Serial.print("Attempting to connect to WPA SSID: ");
  Serial.println(ssid);

  WiFi.begin(ssid, pass);
  while (WiFi.status() != WL_CONNECTED) {
    Serial.print(".");
    delay(500);
  }

  Serial.println("You're connected to the network");
  Serial.println();
  Serial.print("Attempting to connect to the MQTT broker: ");
  Serial.println(mqtt_server);

  mqttClient.setUsernamePassword(mqtt_user, mqtt_pass);
  if (!mqttClient.connect(mqtt_server, mqtt_port)) {
    Serial.print("MQTT connection failed! Error code = ");
    Serial.println(mqttClient.connectError());
    while (1);
  }

  Serial.println("You're connected to the MQTT broker!");
  Serial.println();
}

void loop() {
  mqttClient.poll();

  delay(interval);

  int senseDoorbell = analogRead(doorbell_pin);  //read the doorbell sensor
  if (senseDoorbell > 50) {                      //mine read between 0 and 7 with no current and 200 with it.  50 seemed to be safe.
    mqttClient.beginMessage("doorbell/press");
    mqttClient.print("True");
    mqttClient.endMessage();
    Serial.println("doorbell/press: True");
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
