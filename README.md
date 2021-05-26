# ESPhome

How the get this board working in ESPhome

CP2014 (WeMos TTGO ESP8266)
https://www.aliexpress.com/item/33014477509.html?spm=a2g0s.9042311.0.0.53ed4c4dC6gUln

Follow this https://www.youtube.com/watch?v=7R30c-H8Rro
- install ESPhome add-on in HomeAssistant
- connect you'r board using USB to the you server (RPI or whatever you are using)
- 
   0) Click on the green (+) button
   1) Enter a name
   2) Pick "NodeMCU"
   3) Enter wifi details
   4) Click "Submit"
- Click on "EDIT"
- Put this code under the rest of the code
```
font:
  - file: "fonts/ARIALN.ttf"
    id: my_font
    size: 16

i2c:
  sda: D2
  scl: D1
display:
  - platform: ssd1306_i2c
    model: "SSD1306 128x32"
    reset_pin: D0
    address: 0x3C
    lambda: |-
      it.printf(0, 0, id(my_font), "%s Wh", id(METER).state.c_str());
      it.printf(0, 16, id(my_font), "%s C / %s %%", id(TEMP).state.c_str(), id(HUM).state.c_str());

      
text_sensor:
  - platform: homeassistant
    entity_id: sensor.meter_huidig_verbruik
    id: METER
  - platform: homeassistant
    entity_id: sensor.woonkamer_temperature
    id: TEMP
  - platform: homeassistant
    entity_id: sensor.woonkamer_humidity
    id: HUM
```
~i had some issues with de Dx number's if it's not working try, when you put a static text on the display press the "RST" button!!!~
```
SDA -- D4
SCL -- D5
RST -- D2 
```

- Click on "Save"
- Clik on "Upload" your config (this proces will replace the firmware on you board)
- you'r done
