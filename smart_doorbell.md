Make you're old wired doorbell interact with HomeAssistant

Inspiration:
 - https://www.youtube.com/watch?v=p596PnCCIX0
 - https://community.home-assistant.io/t/esphome-voltage-detection-integrate-doorbell/226653/12

Alternative solution:
 - https://frenck.dev/diy-smart-doorbell-for-just-2-dollar/

Components:
 - 5A Sensor Range of Single-Phase Module Ac Current Sensor Module For Arduino https://www.aliexpress.com/item/4000434009123.html
 - NodeMcu
 - HomeAssistant + ESPhome
 - USB power

Hardware setup:
 - Put the Current Sensor between the wire of your old doorbell
 - Connect pin A0 and GND to the Current Sensor
 - Power the board with USB

Script:
```
binary_sensor:
  - platform: template
    name: "Doorbell Chime"
    id: doorbell_chime

sensor:
  - platform: adc
    name: "Doorbell Chime Voltage"
    id: doorbell_chime_voltage
    pin: A0
    update_interval: 0.5s
    internal: true
    on_value_range:
      - above: 0.1
        then:
          if:
            condition:
              binary_sensor.is_off: doorbell_chime
            then:
              - binary_sensor.template.publish:
                  id: doorbell_chime
                  state: on
              - delay: 5s
              - binary_sensor.template.publish:
                  id: doorbell_chime
                  state: off
```
