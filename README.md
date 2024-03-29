[![Stand With Ukraine](https://raw.githubusercontent.com/vshymanskyy/StandWithUkraine/main/banner-direct-team.svg)](https://stand-with-ukraine.pp.ua)

# GGreg20_V3 under Home Assistant with ESPHome setup example
IoT-devices GGreg20_V3 Ionizing Radiation Detector module under Home Assistant server with ESPHome plugin yaml-script setup example.

Hackaday Project Page: https://hackaday.io/project/183103-ggreg20v3-ionizing-radiation-detector

ESPHome-Devices Project Page: https://www.esphome-devices.com/devices/IoT-devices-GGreg20-V3/

<a href="https://www.tindie.com/stores/iotdev/?ref=offsite_badges&utm_source=sellers_iotdevices&utm_medium=badges&utm_campaign=badge_large"><img src="https://d2ss6ovg47m0r5.cloudfront.net/badges/tindie-larges.png" alt="I sell on Tindie" width="200" height="104"></a>

# ESPHome and Home Assistant Compatibility
This hardware device is designed to be compatible with as many common software platforms and hardware systems as possible. GGreg20_V3 is compatible with any of the following systems: Arduino, ESP8266, ESP32, STM32, Raspberry Pi, ESPHome, Home Assistant, Tasmota, MicroPython, NodeMCU, Node-RED and many others. All you need to connect the GGreg20_V3 is a system with a pulse counter on the GPIO and a timer to measure time.

<img src="https://github.com/iotdevicesdev/ggreg20-v3-homeassistant-esphome-example/blob/main/made-for-esphome-black-on-white.png" width="250">

# Documentation
- Product description https://iot-devices.com.ua/en/product/ggreg20_v3-ionizing-radiation-detector-with-geiger-tube-sbm-20/
- Datasheet https://iot-devices.com.ua/wp-content/uploads/2021/11/ggreg20_v3-datasheet-eng.pdf

# Install
## The Server
### Step 1. Install (or start) the Home Assistant server
If you already have a server installed, just start it. If you need to deploy the server, we recommend that you review the instructions we developed for deploying Home Assistant in a virtual machine running Windows 10. https://alterstrategy.com/2021/05/03/home-assistant-server-instructions-for-deploying-to-a-windows-virtual-machine/

## ESPHome plugin for Home Assistant
### Step 2. Connect the ESPHome extension for the Home Assistant server via the Supervisor -> Add-on Store menu
The procedure for installing an official Add-on, such as ESPHome, in Home Assistant is quite simple. We recommend that you review Step 8. Installing the ESPHome Plugin (Option) the instructions mentioned earlier. https://alterstrategy.com/2021/05/03/home-assistant-server-instructions-for-deploying-to-a-windows-virtual-machine/

## YAML-config of the new ESP device with GGreg20
### Step 3. Download an example yaml-file for the GGreg20_V3 from this repository
The YAML file is a common text script file in Home Assistant (in this case dedicated to ESPHome) which is used as config when building the firmware.
We have developed such a file for ESP8266 with GGreg20_V3 and posted it here for free download and use by anyone who needs to connect GGreg20 to Home Assistant via ESPHome.

Let's look on the main parts of the ggreg20_esp8266_esphome.yaml file prepared by us:

To calculate the value of the ionizing radiation power of microsieverts per hour, we used Pulse Counter Sensor - an API-component of the ESPHome plug-in:
https://esphome.io/components/sensor/pulse_counter.html

This part of the yaml code is responsible for that:
```yaml
sensor:
- platform: pulse_counter
  pin: D3
  unit_of_measurement: 'uSv/Hour'
  name: 'Ionizing Radiation Power'
  count_mode: 
    rising_edge: DISABLE
    falling_edge: INCREMENT
  update_interval: 60s
  accuracy_decimals: 3
  id: my_doze_meter
  filters:
    - sliding_window_moving_average: # 5-minutes moving average (MA5) here
        window_size: 5
        send_every: 5      
    - multiply: 0.0057 # SBM20 tube conversion factor of pulses into uSv/Hour 
```

To calculate the total radiation dose received in microsieverts, the Integration Sensor, also a component of the ESPHome API, is used:
https://esphome.io/components/sensor/integration.htm

This part of the yaml code is responsible for that:
```yaml
sensor:
- platform: integration
  name: "Total Ionizing Radiation Dose"
  unit_of_measurement: "uSv"
  sensor: my_dose_meter # link entity id to the pulse_counter values above
  icon: "mdi:radioactive"
  accuracy_decimals: 5
  time_unit: min # integrate values every next minute
  filters:
    # obtained dose. Converting from uSv/hour into uSv/minute: [uSv/h / 60] OR [uSv/h * 0.0166666667]. 
    # if my_dose_meter in CPM, then [0.0054 / 60 minutes] = 0.00009; so CPM * 0.00009 = dose every next minute, uSv.
    - multiply: 0.0166666667
```
Also, in order to be able to test the pulse counter without the GGreg20 sensor, a binary sensor of the "Flash" button on the ESP8266 controller has been added to the configuration - on the same GPIO0 (D3) as the Pulse Counter - but another ESPHome API component, GPIO Binary Sensor, is used:
https://esphome.io/components/binary_sensor/gpio.html

This part of the yaml code is responsible for that:
```yaml
binary_sensor:
  - platform: gpio
    name: "D3 Input Button"
    pin:
      number: 0
      inverted: True
      mode: INPUT_PULLUP
```

The ESPHome plugin has sufficient documentation for these components with examples, so we will not go into detailed explanations.

### Step 4. Create (based on the example) in ESP Home the appropriate yaml configuration file
After downloading the file, we suggest you open it with any text editor and become familiar with its contents.
Next, in the interface of the ESP Home plugin on the Home Assistant web page with administrator access rights, you need to create your own yaml file by clicking "+" and answering a few initial questions of the wizard.
After completing the "wizard", a standard file with the basic parameters appears in the ESP Home interface. Now you need to add to this file the parts that you will find in our ggreg20_esp8266_esphome.yaml example file.

## Hardware connection GGreg20_V3 and controller
In steps 5 and 6, we give an example of a connection for ESP8266. For the ESP32 controller, everything is the same. The only difference is the number, purpose and numbering of ESP32's GPIOs as a more powerful hardware platform. But the general logic is identical.
![GGreg20_V3 wired connection](https://github.com/iotdevicesdev/ggreg20-v3-homeassistant-esphome-example/blob/main/ESPHome_GGreg20_V3_and_NodeMCU_ESP8266_wired_connection.jpg)

### Step 5. Select the GPIO pin on the controller that will register the pulses from GGreg20
If this is your first time dealing with ESP8266 platform and you do not know which GPIO is best to use for a pulse counter, we recommend using the GPIO Planning and Application Standard for ESP8266-12 / NodeMCU / Lua projects developed by alterstrategy.lab. https://alterstrategy.com/recommended-pin-use-standard/

For example, it could be GPIO0 (D3). This pin is convenient because it has a built-in Flash button in most devices and boards based on the ESP8266 module - in case you need to check how the controller counts pulses without a sensor, it is possible to simulate pulses with a button.

### Step 6. Connect the GGreg20_V3 radiation detector to the ESP8266 controller via the Out connector to the selected GPIO of the controller
As you can see, the connection is quite simple - you only need to supply power from the NodeMCU for the GGreg20 module, and connect the output (Out) of the sensor to the input (D3) of the controller and supply 5V to the micro USB connector of the NodeMCU board.

## Flashing the ESP device
### Step 7. Build and write firmware for the controller
Before building the firmware, you need to validate the yaml file you created. This will protect you from file errors that may have accidentally made.

Note. If the controller is new - you need to compile and download to the PC a binary firmware bin-file in the ESP Home interface - click DOWNLOAD BINARY after successful compilation.

Next you need to write the firmware to the controller (ESP8266 / ESP32). This can be done by means of the ESPHome-Flasher utility. It can be freely found and downloaded via the Internet. https://github.com/esphome/esphome-flasher/releases

After flashing the firmware and restarting the new device, it is recommended to restart the Home Assistant server too.

Important! After starting the server you need to go to the menu Configuration -> Integration. Find there a new device that we flashed and connect it to the server configuration, if it is not connected automatically.

## Entities and values of the device on the server
### Step 8. Check the ESPHome log of the ESP8266 (optional)
### Step 9. Check for new entities on the server side
There are two ways to verify that the corresponding GGreg20 entities are registered in the Home Assistant server:
- go to the Developer Tools menu on the sidebar of the Home Assistant interface and search for the relevant data;
- or go to the menu Configuration -> Integration and search there.

## Visualization
### Step 10. Add GGreg20 radiation sensor widgets to the Dashboard
Here is an example of a demo tab from an active server for two devices GGreg20_V1 and GGreg20_V3 located in different coordinate axes. Each device uses the same yaml file as we created above.
![GGreg20_V3 Dashboard widgets example](https://github.com/iotdevicesdev/ggreg20-v3-homeassistant-esphome-example/blob/main/HA_Rad-Dashboard_Both-GGreg2-GGreg3_dark_2021-04-29_132657.jpg)
# Buy GGreg_V3 Ionizing Radiation Detector Module
On Tindie: https://www.tindie.com/products/iotdev/ggreg20_v3-ionizing-radiation-detector/

IoT-devices Online Shop: https://iot-devices.com.ua/en/product/ggreg20_v3-ionizing-radiation-detector-with-geiger-tube-sbm-20/

# Watch demo video

<a href="https://www.youtube.com/watch?feature=player_embedded&v=lGIwdO35k1w" target="_blank">
 <img src="https://img.youtube.com/vi/lGIwdO35k1w/mqdefault.jpg" alt="Watch the video" border="10" />
</a><br>

IoT-devices YouTube Channel: 
https://www.youtube.com/channel/UCHpPOVVlbbdtYtvLUDt1NZw/videos
