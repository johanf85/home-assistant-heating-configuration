# 1. My home smart heating configuration with the use of Home Assistant

Home Assistant is Open Source software that runs on various devices, for instance Raspberry Pi, and acts as a central server for smart home devices and/or self build modules to make automatizations in the home. It has an active community and a large library of integrations with products on the market. Home Assistant is a non-cloud system, which means there is not necessarily a dependance on external cloud services (and an internet connection).

![https://www.home-assistant.io/](https://user-images.githubusercontent.com/43075793/110300847-b4172f00-7ff7-11eb-94d2-3394992bd090.png)

With the use of Home Assistant I created a smart thermostat, which I was able to customize to my personal situation. Against low costs and all wired to limit the EMF radiation in the home. In this document I will describe my configuration. I am Dutch, so the names I chose for various entities are in Dutch.

Screenshots of my dashboard:

![](https://user-images.githubusercontent.com/43075793/110309572-ce560a80-8001-11eb-85fc-a04b34ca5de8.png)

Here is a tab with variables:

![](https://user-images.githubusercontent.com/43075793/110309259-699ab000-8001-11eb-95a1-4a28d13302c3.png)

And on my phone:

![](https://user-images.githubusercontent.com/43075793/110310007-51776080-8002-11eb-9b64-755373b3a415.png)

*   TOC  
    {:toc}

My appartement consists of a living room, a bedroom and a kitchen.

<table><tbody><tr><td><figure class="image"><img src="https://user-images.githubusercontent.com/43075793/117958666-8a41f780-b31b-11eb-812e-aa2912945f4b.png"></figure></td></tr><tr><td>A schematic view of the floorplan with the relevant rooms</td></tr></tbody></table>

<table><tbody><tr><td><figure class="image"><img src="https://user-images.githubusercontent.com/43075793/110297333-a790d780-7ff3-11eb-839d-b3b001fa90a7.png"></figure></td></tr><tr><td>A 3D view of the floorplan, the door between living room and bedroom can be closed off<br>(made with <a href="https://roomstyler.com/3dplanner">https://roomstyler.com/3dplanner</a>)</td></tr></tbody></table>

The appartement has a boiler (Intergas kompakt hre 24/18) for central heating which can use a on-off control and OpenTherm.  

<table><tbody><tr><td><figure class="image"><img src="https://user-images.githubusercontent.com/43075793/117959173-08060300-b31c-11eb-9171-167f414ecc1a.png"></figure></td></tr><tr><td>Intergas hre 24/18</td></tr></tbody></table>

## 1.1 1 The design wishes for the system

*   Multi zone: Heat the bedroom to a desired temperature at night (while living room radiator closed) and the living room during the day and evening (while bedroom radiator closed)
*   Turn off the thermostat function when no one home
*   A wired system, that doesn't use wifi, to limit the use of EMF-radiation in the home. See [this TED talk](https://www.youtube.com/watch?v=F0NEaPTu9oI) about possible dangers of EMF radiation in the living environment.
*   A smartphone app and / or a browser to control and monitor the thermostat
*   Send alerts to the smartphone when certain conditions are met, e.g. when it is suspected that a window is open.
*   Reverting to normal set temperature after a certain amount of time after a manual change
*   Easily setting variables like bedtime, waking time and revert-to-initial-time with input fields in the front-end

## 1.2 Software

[Home Assistant](https://www.home-assistant.io/) (The main platform on which everything is running)

Arduino IDE (for uploading Arduino sketches to the arduino mcu)

**Home assistant integrations for the thermostat**

Several integrations are used. The most important is the Generic Thermostat integration, designed to act as a on-off thermostat with a switch:

*   [Generic Thermostat integration](https://www.home-assistant.io/integrations/generic_thermostat/)

## 1.3 Hardware

**Bedroom**

*   Arduino nano with a DS18B20 temperature sensor connected via USB to the Home Assistant server

![](https://user-images.githubusercontent.com/43075793/121677790-abcb0600-cab6-11eb-9732-bff4973d8bb1.png)

![](https://user-images.githubusercontent.com/43075793/117957906-cb85d780-b31a-11eb-8d61-c71f36264ce6.png)

Store: [Aliexpress](https://nl.aliexpress.com/item/1005002674336082.html)

Arduino sketch used:

```c++
&#35;include <OneWire.h>
&#35;include <DallasTemperature.h>

// Data wire is plugged into port 2 on the Arduino
&#35;define ONE_WIRE_BUS 2

// Setup a oneWire instance to communicate with any OneWire devices (not just Maxim/Dallas temperature ICs)
OneWire oneWire(ONE_WIRE_BUS);

// Pass our oneWire reference to Dallas Temperature.
DallasTemperature sensors(&oneWire);

/*
   The setup function. We only start the sensors here
*/
void setup(void)
{
  // start serial port
  Serial.begin(9600);

  // Start up the library
  sensors.begin();
  sensors.setResolution(12);
}

/*
   Main function, get and show the temperature
*/
void loop(void)
{
  int numberOfDevices = sensors.getDeviceCount();
  if (numberOfDevices > 0) {
    sensors.requestTemperatures();
      }
      float t = sensors.getTempCByIndex(0);
      Serial.println(t);

  delay(30000);
}
```

**Living room**

**Server with sensors:**

![](https://user-images.githubusercontent.com/43075793/121671494-e6c93b80-caae-11eb-8e96-3eeadb19c349.png)

Case: [Aliexpress](https://s.click.aliexpress.com/e/_ABlPWR)  
PIR case (only used as case and fitted in 5V PIR): [Aliexpress](https://s.click.aliexpress.com/e/_A9CbmJ)

Contains:

*   Raspberry pi Zero W (acts as server) with:

![](https://user-images.githubusercontent.com/43075793/117957217-210db480-b31a-11eb-834b-1250b6fbd008.png)

*   DS18B20 temperature sensor

![](https://user-images.githubusercontent.com/43075793/117957906-cb85d780-b31a-11eb-8d61-c71f36264ce6.png)

Store: [Aliexpress](https://nl.aliexpress.com/item/1005002674336082.html)

*   PIR motion sensor module

![](https://user-images.githubusercontent.com/43075793/117958088-f8d28580-b31a-11eb-999f-f8b17d1bf0c5.png)

Store: [Aliexpress](https://s.click.aliexpress.com/e/_9HU2TH)

*   Relay module with wire to boiler for controlling on-off heating

![](https://user-images.githubusercontent.com/43075793/117958501-5c5cb300-b31b-11eb-8065-645693a0284e.png)

Store: [Aliexpress](https://s.click.aliexpress.com/e/_A5z0ab)

*   USB hub with Ethernet and connected USB storage device for the OS.
*   MicroSD with bootcode.bin file (needed, as pi0 doesn't boot from USB by default)

The pi Zero W is the Raspberry with the least specifications. However it does run well for a year now, as I only use it for nothing else than the thermostat function. \`

Dowload Home-Assistant  OS for pi0 at: [https://github.com/home-assistant/operating-system/releases](https://github.com/home-assistant/operating-system/releases) for pi Zero W choose: hassos\_rpi0-w-x.xx.img.xz. 

Both rooms have one radiator, each is equipped with a eqiva-N thermostatic radiator valve (TRV). These are only used to be open and close the radiators at the beginning and end of the day. They are programmed to setpoint 12° C when the desired heating for the room is off and to 25° C when the desired heating is on. The limitation of using these TRVs is that they aren't accessible from within Home Assistant, as I chose a non smart TRV.

<table><tbody><tr><td><figure class="image"><img src="https://user-images.githubusercontent.com/43075793/102992726-0a59f300-451c-11eb-83e4-a0b0d94b93bb.jpg"></figure></td></tr><tr><td><i>The eqiva-N thermostatic radiator valve (TRV)</i></td></tr></tbody></table>

Radiator valve before:

<table><tbody><tr><td><figure class="image"><img src="https://user-images.githubusercontent.com/43075793/102994879-41320800-4520-11eb-9485-fe36122d2637.jpg"></figure></td></tr><tr><td><i>The Herz valve knob - old situation</i></td></tr></tbody></table>

There were no thermostatic valves on my radiator (just a turn knob), to get it working with the Eqiva TRV I had to put in a new insert, part #1639091 for Herz radiator valves:  

<table><tbody><tr><td><figure class="image"><img src="https://user-images.githubusercontent.com/43075793/102867026-6d725980-4438-11eb-9752-5bd6fffe2686.png" alt=""></figure></td></tr><tr><td><i>The Herz 1639091 insert to make the valve a thermostatic valve</i></td></tr></tbody></table>

An extra adapter was necessary to fit the eqiva-N TRV as it doesn't fit the Herz system with the adapters provided on buying. 

<table><tbody><tr><td><figure class="image"><img src="https://user-images.githubusercontent.com/43075793/102991803-40967300-451a-11eb-9a69-aa1a5d3b32cc.jpg"></figure></td></tr><tr><td><i>IMI adapter for Herz part no. 15071304</i></td></tr></tbody></table>

A doorspring was put on both rooms door, so that they will be kept closed as much as possible to avoid heat loss.  

<table><tbody><tr><td><figure class="image"><img src="https://user-images.githubusercontent.com/43075793/102992183-0b3e5500-451b-11eb-8786-723f359d2996.jpeg"></figure></td></tr><tr><td><i>Doorspring for door closing</i></td></tr></tbody></table>

## 1.4 Configuration in Home Assistant

### 1.4.1 Helpers 

Created the following helpers via configuration > helpers within Home Assistant

|   | Type |   |
| --- | --- | --- |
| **Someone home?** | input\_boolean (toggle) |   |
|   |   | input\_datetime.bedtijd |
|   |   | input\_text.woonk\_nacht |
|   |   | input\_number.current\_insteltemp\_woonkamer |

### 1.4.2 Variables

To easily change settings to which I would like them, I make use of input variables. I made a tab in Lovelace to edit this variables.

![](https://user-images.githubusercontent.com/43075793/121800703-a941eb00-cc33-11eb-9dcc-9041997f328b.png)

Most of them speak for themselves. The someone home switch is turned on depending on presence detection. When the set temperature of the thermostat is changed manually, it will revert back to the initial set temperature according to program after the set Duration manual change value. The countdown shows the amount of time left until revert to initial set temperature. 

### 1.4.3 Configuration.yaml

Only covering the relevant part of the configuration for the smart heating system. 

#### 1.4.3.1 For the generic thermostat integration

Home Assistant documentation: [Generic thermostat](https://www.home-assistant.io/integrations/generic_thermostat/)

Choices for configuration variables:

*   `target_temp` I chose target temps within configuration.yaml lower than I actually want, cause with a sudden a reboot of the system I don't want the heating to turn on. I use automations to override the temperature to set these to desired temperatures. After a restart the `target_temp` is reverted back to the last setting before the reboot. See Automations further on.
*   `min_cycle_duration` Set so that pump and gas furnace of boiler don't have to turn on and off that often. Set to 3 minutes for bedroom and 1 minute for living room. The living room has a larger radiator and 1 minute turned out as a good value. For the bedroom 3 minutes as the there is a smaller radiator.
*   `heater` I use two `generic_thermostat` instances with separate heater switches, because Generic thermostat isn't able to work with multiple zones (described [here](https://community.home-assistant.io/t/need-help-with-multi-zone-generic-thermostat-climate-configuration/8563)) when one heater switch is on both rooms (they will contradict). For each room I used a [Helper](https://www.home-assistant.io/integrations/input_boolean/), `input_boolean` switch. Trough an automation I made sure these helpers are controlling the relay switch, which controls the on-off signal to the boiler, see [automations](#automations).

```yaml
climate:
- platform: generic_thermostat
  name: Livingroom
  heater: input_boolean.schakelaar_woonkamer
  target_sensor: sensor.ds18b20_woonkamer_correctie
  min_temp: 12
  max_temp: 22
  ac_mode: false
  min_cycle_duration:
    minutes: 1
  target_temp: 18.0
  cold_tolerance: 0
  hot_tolerance: 0
  initial_hvac_mode: "heat"
  away_temp: 16
  precision: 0.1
- platform: generic_thermostat
  name: Bedroom
  heater: input_boolean.schakelaar_slaapkamer
  target_sensor: sensor.serial_sensor
  min_temp: 10
  max_temp: 22
  ac_mode: false
  min_cycle_duration:
    minutes: 3
  target_temp: 12.0
  cold_tolerance: 0
  hot_tolerance: 0
  initial_hvac_mode: "heat"
  away_temp: 16
  precision: 0.1
```

#### 1.4.3.2 Telegram integration

I use [Telegram](https://telegram.org/) for notifications. Currently I am using two notifications:

*   Hours of heating during a week on Sunday at 18:00

![](https://user-images.githubusercontent.com/43075793/110303144-53d5bc80-7ffa-11eb-98dc-0a7ae96f4686.png)

"Last week there were 21 hours of heating"

*   When the heating is automatically turned off because of a suspected open window (See Verwijzing).

![](https://user-images.githubusercontent.com/43075793/110302916-13763e80-7ffa-11eb-8579-ccd264169956.png)

Translation: "Bedroom thermostat turned off bc of too slow heating up, window open?"

Home Assistant documentation: [Telegram polling](https://www.home-assistant.io/integrations/telegram_polling/)

```yaml
telegram_bot:
  - platform: polling
    api_key: secret
    allowed_chat_ids:
      -  secret


notify:
  - platform: telegram
    name: Telegramnotifier
    chat_id: secret
```

#### 1.4.3.3 Correction of temperature sensors

My DS18B20 sensors ([onewire](https://www.home-assistant.io/integrations/onewire/)) need a correction to match the right temperature value. With the use of [template platform](https://www.home-assistant.io/integrations/template/) a correction is applied to the onewire sensors. 

See below:

```yaml
{% raw %}
sensor:
  - platform: serial
    serial_port: /dev/ttyUSB0
  - platform: onewire
  - platform: time_date
    display_options:
      - 'time'
      - 'date'
      - 'date_time'
  - platform: template
    sensors:
      ds18b20_woonkamer_correctie:
        value_template: "{{ states('sensor.28_00000913d350_temperature')|float - 1.2}}"
        friendly_name: 'Woonkamer temp'
        unit_of_measurement: degrees
      ds18b20_slaapkamer_correctie:
        value_template: "{{ states('sensor.28_011937d1c3d1_temperature')|float - 0.6}}"
        friendly_name: 'Slaapkamer temp'
        unit_of_measurement: degrees
  - platform: history_stats
    name: Aantal minuten verwarmen laatste 7 dagen
    entity_id: sensor.heating_state
    state: 'heating'
    type: time
    end: '{{ now().replace(hour=0, minute=0, second=0) }}'
    duration:
        days: 7
{% endraw %}
```

#### 1.4.3.4 Trend sensor for possible open window detection

Using the [trend platform](https://www.home-assistant.io/integrations/trend/) it is checked if the temperature will rise enough while heating. If not, it can be assumed that a window is open or some other error is happening and the heating is turned off. See [automations](#automations). 

The `min_gradient` value, which is the temperature rising per second, for each room is set based on trial and error. 

```yaml
{% raw %}
binary_sensor:
  - platform: rpi_gpio
    invert_logic: false
    ports:
     16: Kleine Motion sensor
  - platform: rpi_gpio
    invert_logic: false
    ports:
     26: Motion sensor
  - platform: trend
    sensors:
      temp_falling_slaapkamer:
        entity_id: sensor.serial_sensor
        sample_duration: 150
        max_samples: 3
        min_gradient: 0.0003
        invert: false
        friendly_name: DeltaT slaapk voldoende
      temp_falling_woonkamer:
        entity_id: sensor.ds18b20_woonkamer_correctie
        sample_duration: 150
        max_samples: 3
        min_gradient: 0.0005
        invert: false
        friendly_name: DeltaT woonk voldoende
 {% endraw %}
```

```yaml
sensor:
  - platform: serial
    serial_port: /dev/ttyUSB0
  - platform: onewire
  - platform: time_date
    display_options:
      - 'time'
      - 'date'
      - 'date_time'
  - platform: template
    sensors:
      deltat_slaapkamer:
        value_template: "{{state_attr('binary_sensor.temp_falling', 'gradient')|float * 1000}}"
        friendly_name: 'Slaapkamer temp gradient'
        unit_of_measurement: 'graden'
      deltat_slaapkamer_grens:
        value_template: "{{state_attr('binary_sensor.temp_falling', 'min_gradient')|float * 1000}}"  
        friendly_name: 'Slaapkamer min gradient'
        unit_of_measurement: 'graden'
      deltat_woonkamer:
        value_template: "{{state_attr('binary_sensor.temp_falling_woonkamer', 'gradient')|float * 1000}}" 
        friendly_name: 'Woonkamer temp gradient'
        unit_of_measurement: 'graden'
      deltat_woonkamer_grens:
        value_template: "{{state_attr('binary_sensor.temp_falling_woonkamer', 'min_gradient')|float * 1000}}"
        friendly_name: 'Woonkamer min gradient'
        unit_of_measurement: 'graden'    
      heating_state:
        value_template: "{{state_attr('climate.woonkamer', 'hvac_action')}}"
        friendly_name: 'thermostaat state'
```

### 1.4.4 Automations

#### 1.4.4.1 Setting temperature time program

Two automations per room, one for setting the desired set-temperature at bedtime and one at wake-up time. Also a helper `input_number.current_insteltemp_slaapkamer` is set with the current-set temperature. This is needed for restoring the set temperatures after restart of the system and after a manual change.  

##### 1.4.4.1.1 Bedroom set temperature after bedtime

```yaml
{% raw %}
- id: '1587807892892'
  alias: Slaapkamer naar instelwaarde 's nachts
  description: ''
  trigger:
  - at: input_datetime.bedtijd
    platform: time
  condition: []
  action:
  - data_template:
      entity_id: climate.slaapkamer
      temperature: '{{ states.input_text.slaapkamer_insteltemp_nacht.state }}'
    service: climate.set_temperature
  - service: input_number.set_value
    data:
      value: '{{ states.input_text.slaapkamer_insteltemp_nacht.state }}'
    entity_id: input_number.current_insteltemp_slaapkamer
  mode: single
{% endraw %}
```

#####  Bedroom to set temperature after bedtime

```yaml
{% raw %}
- id: '1589611935632'
  alias: Woonkamer instelwaarde na opstaan
  description: ''
  trigger:
  - at: input_datetime.opstaan
    platform: time
  condition: []
  action:
  - data_template:
      entity_id: climate.woonkamer
      temperature: '{{ states.input_text.woonk_overdag.state }}'
    service: climate.set_temperature
  - service: input_number.set_value
    data:
      value: '{{ states.input_text.woonk_overdag.state }}'
    entity_id: input_number.current_insteltemp_woonkamer
  mode: single
{% endraw %}
```

#####  Bedroom to set temperature during day time

```yaml
{% raw %}
- id: '1587807715263'
  alias: Slaapkamer naar instelwaarde overdag
  description: ''
  trigger:
  - at: input_datetime.opstaan
    platform: time
  condition: []
  action:
  - data_template:
      entity_id: climate.slaapkamer
      temperature: '{{ states.input_text.slaapkamer_insteltemp_overdag.state }}'
    service: climate.set_temperature
  - service: input_number.set_value
    data:
      value: '{{ states.input_text.slaapkamer_insteltemp_overdag.state }}'
    entity_id: input_number.current_insteltemp_slaapkamer
  mode: single
{% endraw %}
```

#####  Living room set temperature after bedtime

```
{% raw %}
- id: '1587310221936'
  alias: Woonkamer instelwaarde na bedtijd
  description: ''
  trigger:
  - at: input_datetime.bedtijd
    platform: time
  condition: []
  action:
  - data_template:
      entity_id: climate.woonkamer
      temperature: '{{ states.input_text.woonk_nacht.state }}'
    service: climate.set_temperature
  - service: input_number.set_value
    data:
      value: '{{ states.input_text.woonk_nacht.state }}'
    entity_id: input_number.current_insteltemp_woonkamer
  mode: single
{% endraw %}
```

###  Presence detection

![](https://user-images.githubusercontent.com/43075793/117958088-f8d28580-b31a-11eb-999f-f8b17d1bf0c5.png)

Presence detection is done with both the mobile phone GPS location and a PIR movement sensor in the living room. If the mobile phone was the only source for presence detection this would have been used, but since there is a PIR as well, the away modus of Generic thermostat integration isn't used in this configuration, instead of this a helper switch input (Someone home?).  

I used a PIR sensor next to the smartphone, because there are some disadvantages in using smartphones only for presence detection. There are scenarios like when the battery is down or the phone is on flight mode, in which the system will think someones is home.  Also it is less suitable when having guests if the smartphone is used as only presence detection source. 

The configuration is set so, that when the phone `device_tracker.pra_lx1` changes location from away to home or home to away the helper switch `input_boolean.iemandthuis`  is toggled. So only on **state change**, not on current state. Next to that, the rule is followed that if two movements aren't being detected within the last half hour, the 'someone home status' `input_boolean.iemandthuis` is set to off. 

In the evening the presence detection by the PIR motion detector should be different. It is assumed that if two times a motion is detected within half an hour during `evening time` and `bedtime`, that someone will be home during the entire night. 

As it was noticed that the PIR used in very few occasions can have a false motion detection, the number of two motion detections was chosen. 

This configuration set up is based on a one person household, so only one smartphone with the home assistant app running. 

| Triggers | Between hours | set 'someone home status' to |
| --- | --- | --- |
| Two times a motion detected within 30 minutes period 'someone home status' is off | waking up and evening | on |
| An half an hour with one or less motion detected while 'someone home status' is on | waking up and evening | off |
| Two times a motion detected within 30 minutes period while 'someone home status' is off | evening and bedtime | on |
| Smartphone location state change to away | always | off |
| Smartphone location state change to home | always | on |

The following automations were set to achieve this. 

#### 1.4.4.2 Automations for presence detection

##### 1.4.4.2.1 Automation for minimum of two movements detected to trigger other automations

Set two input\_datetime fields on a motion detection with the PIR. One input field is set to date/time of the last movement detected and on the next motion this value is passed to one-before-last-input boolean.

This is used, bc it is desired that only when a minimum of two movements are detected in the last 30 minutes the status of some one should be turned to 'on'.

```yaml
- id: '1606672315270'
  alias: Bewegingssensor last 
  description: ''
  trigger:
  - platform: state
    entity_id: binary_sensor.motion_sensor
    to: 'on'
  condition: []
  action:
  - service: input_datetime.set_datetime
    data:
      datetime: '{{states(''input_datetime.beweginglaatst_0'')}}'
    entity_id: input_datetime.bewegingeennalaatst_1
  - service: input_datetime.set_datetime
    data:
      datetime: '{{ now().strftime(''%Y-%m-%d %H:%M:%S'') }}'
    entity_id: input_datetime.beweginglaatst_0
  mode: single
```

##### 1.4.4.2.2 Reset the one-before-last-input boolean 31 minutes before waking time

Needed for the someone home status to turn on immediately when entering the living room in the morning, otherwise first two motions need to be detected, which can take a while. 

```yaml
- id: '1606905142912'
  alias: Reset 1 na laatste beweging 31 min voor opstaan
  description: ''
  trigger:
  - platform: time_pattern
    seconds: '30'
  condition:
  - condition: template
    value_template: '{/% set current_time = now().hour * 60 + now().minute %}

      {/% set opstaan_hour, opstaan_minute, opstaan_second = states(''input_datetime.opstaan'').split('':'')
      %}

      {/% set opstaan_time = opstaan_hour | int * 60 + opstaan_minute | int %}

      {{ current_time == opstaan_time - 32 }}'
  action:
  - service: input_datetime.set_datetime
    data:
      datetime: '{{now().strftime(''%Y-%m-%d %H:%M:%S'')}}'
    entity_id: input_datetime.bewegingeennalaatst_1
  mode: restart
```

##### 1.4.4.2.3 Automation to turn on someone home status

```yaml
{% raw %}
- id: '1587319960331'
  alias: Turn on someone home status
  description: ''
  trigger:
  - entity_id: device_tracker.pra_lx1
    platform: state
    to: home
  - entity_id: binary_sensor.motion_sensor
    platform: state
    to: 'on'
  condition:
  - condition: state
    entity_id: input_boolean.iemandthuis
    state: 'off'
  action:
  - data: {}
    entity_id: input_boolean.iemandthuis
    service: input_boolean.turn_on
  mode: single
{% endraw %}
```

#####  Automation during evening and getting up 

```
{% raw %}
- id: '1587404974211'
  alias: Aanwezigheid detectie avond tot opstaan
  description: ''
  trigger:
  - platform: template
    value_template: '{{ (states.sensor.time.last_changed - states.input_datetime.bewegingeennalaatst_1.last_changed).total_seconds()
      == 300 }}'
  condition:
  - condition: state
    entity_id: input_boolean.iemandthuis
    state: 'off'
  - before: input_datetime.opstaan
    condition: time
    after: input_datetime.avond
  - condition: template
    value_template: '{{ (states.sensor.time.last_changed - states.input_datetime.bewegingeennalaatst_1.last_changed).total_seconds()
      < 300 }}'
  action:
  - data: {}
    entity_id: input_boolean.iemandthuis
    service: input_boolean.turn_on
  mode: single
- id: '1587373774458'
  alias: Thermostaat aan bij iemand thuis
  description: ''
  trigger:
  - entity_id: input_boolean.iemandthuis
    platform: state
    to: 'on'
  condition: []
  action:
  - data: {}
    entity_id: climate.woonkamer
    service: climate.turn_on
  - data: {}
    entity_id: climate.slaapkamer
    service: climate.turn_on
{% endraw %}
```

#####  Automation between getting up time and evening time

```yaml
{% raw %}
- id: '1587319961411'
  alias: Behaviour of motion sensor living room between wakeup time and evening time
  description: ''
  trigger:
  - platform: template
    value_template: '{{ (states.sensor.time.last_changed - states.input_datetime.bewegingeennalaatst_1.last_changed).total_seconds()
      > 1800 }}

      '
  - platform: state
    entity_id: person.johan
    to: not_home
  condition:
  - condition: state
    entity_id: input_boolean.iemandthuis
    state: 'on'
  - before: input_datetime.bedtijd
    condition: time
    after: input_datetime.opstaan
  action:
  - data: {}
    entity_id: input_boolean.iemandthuis
    service: input_boolean.turn_off
  mode: single
{% endraw %}
```

##### 1.4.4.2.4 Behavior based on smart phone location with Home Assistant app

```

- id: '1606221709908'
  alias: Smartphone thuiskomen / weggaan
  description: ''
  trigger:
  - platform: state
    entity_id: person.johan
    to: not_home
  condition: []
  action:
  - service: input_boolean.turn_off
    data: {}
    entity_id: input_boolean.iemandthuis
  - wait_for_trigger:
    - platform: state
      entity_id: person.johan
      to: home
  - service: input_boolean.turn_on
    data: {}
    entity_id: input_boolean.iemandthuis
  mode: restart
```

#####  Turning off thermostat when Someone home status is off

```yaml
- id: '1587373899805'
  alias: Thermostaat uit bij niemand thuis
  description: ''
  trigger:
  - entity_id: input_boolean.iemandthuis
    platform: state
    to: 'off'
  condition: []
  action:
  - data: {}
    entity_id: climate.woonkamer
    service: climate.turn_off
  - data: {}
    entity_id: climate.slaapkamer
    service: climate.turn_off 
```

### 1.4.5 Heat for 5 minutes straight

##### 1.4.5.1.1 Automation:

```yaml

- id: '1588717917351'
  alias: 5 minuten verwarmen als schakelaar aan
  description: ''
  trigger:
  - entity_id: input_boolean.30_min_verwarmen_schakelaar
    platform: state
    to: 'on'
  condition: []
  action:
  - data: {}
    entity_id: switch.relay
    service: switch.turn_on
  - timeout: 00:05
    wait_template: ''
  - data: {}
    entity_id: switch.relay
    service: switch.turn_off
  - service: input_boolean.turn_off
    data: {}
    entity_id: input_boolean.30_min_verwarmen_schakelaar
  mode: single
```

### 1.4.6 Window open detection (based on speed of temperature rise)

Uses the [Trend sensor](https://www.home-assistant.io/integrations/trend/) to make the `binary_sensor.temp_falling`

After 300 seconds of heating without reaching the treshold of de trend sensor, the relay switch is turned off and sends a Telegram notification. 

```yaml

- id: '1588799650928'
  alias: Relay beveiging raam open
  description: ''
  trigger:
  - platform: time_pattern
    minutes: '5'
  condition:
  - condition: state
    entity_id: binary_sensor.temp_falling
    state: 'off'
  - condition: template
    value_template: '{{state_attr(''climate.slaapkamer'', ''temperature'') | float
      > state_attr(''climate.slaapkamer'', ''current_temperature'')}}'
  - condition: state
    entity_id: switch.relay
    state: 'on'
  - condition: template
    value_template: '{{ (states.sensor.time.last_changed  - states.switch.relay.last_changed).total_seconds()
      > 310 }}'
  action:
  - service: telegram_bot.send_message
    data:
      message: Slaapkamer thermostaat uitgeschakeld ivm te langzame opwarming, raam
        open?
      target: [chat-number]
  - data: {}
    entity_id: climate.slaapkamer
    service: climate.turn_off
  - delay: 00:45:00
  - service: climate.turn_on
    data: {}
    entity_id: climate.slaapkamer
  mode: single
```

```yaml

- id: '1601215001454'
  alias: Relay beveiging raam open (woonkamer)
  description: ''
  trigger:
  - platform: template
    value_template: '{{ (states.sensor.time.last_changed  - states.switch.relay.last_changed).total_seconds()
      > 400 }}'
  condition:
  - condition: state
    entity_id: automation.slaapkamer_overgang_off_on_set_schakelaar
    state: 'on'
  action:
  - data: {}
    entity_id: climate.woonkamer
    service: climate.turn_off
  - delay: 00:59:00
  - service: climate.turn_on
    data: {}
    entity_id: climate.woonkamer
  mode: single
```

##### 1.4.6.1.1 Revert back to programmed set temperature after manual change  

According to `input_datetime.duur_manuele_verhoging` value a timer is started after which the set temperature will revert back to set temperature according to program. 

Uses the [Timer integration](https://www.home-assistant.io/integrations/timer/)

```yaml
- id: '1604920070266'
  alias: Countdown bij manual wijziging
  description: ''
  trigger:
  - platform: state
    entity_id: climate.slaapkamer
    attribute: temperature
  - platform: state
    entity_id: climate.woonkamer
    attribute: temperature
  condition: []
  action:
  - service: timer.start
    data:
      duration: '{{ states.input_datetime.duur_manuele_verhoging.state }}'
    entity_id: timer.countdown
  - delay: '{{ states.input_datetime.duur_manuele_verhoging.state }}'
  - service: climate.set_temperature
    data:
      temperature: '{{ states.input_number.current_insteltemp_slaapkamer.state  }}'
    entity_id: climate.slaapkamer
  - service: climate.set_temperature
    data:
      temperature: '{{ states.input_number.current_insteltemp_woonkamer.state  }}'
    entity_id: climate.woonkamer
  mode: restart
```

##### 1.4.6.1.2 Telegram notification hours of heating past week on Sunday

```yaml
- id: '1604938488226'
  alias: Notificatie aantal uren verwarmen in week
  description: ''
  trigger:
  - platform: time
    at: '18:00'
  condition:
  - condition: time
    weekday:
    - sun
  action:
  - service: telegram_bot.send_message
    data:
      message: De afgelopen week is er {{ states.sensor.aantal_minuten_verwarmen_laatste_7_dagen.state
        | int}} uren verwarmd.
  mode: single
```

##### 1.4.6.1.3 Turn of heating when there is no signal of DS18B20 temperature sensor

It occasionally happens that there is no signal of the DS18B20 temperature sensor or that by mistake the USB cable gets unplugged. The displayed temperature then can get below set temperature and will trigger heating while not really desired. To avoid this an automation is set to turn off. 

```yaml
- id: '1606221459609'
  alias: Beveiliging uitvallen temp sensor
  description: ''
  trigger:
  - platform: numeric_state
    entity_id: climate.slaapkamer
    attribute: current_temperature
    below: '10'
  - platform: numeric_state
    entity_id: sensor.ds18b20_woonkamer_correctie
    below: '10'
  condition: []
  action:
  - service: climate.turn_off
    data: {}
    entity_id: climate.slaapkamer
  - service: climate.turn_off
    data: {}
    entity_id: climate.woonkamer
  mode: single
```

##### 1.4.6.1.4 Controlling two generic thermostat entities

The [generic thermostat integration](https://www.home-assistant.io/integrations/generic_thermostat/) is equipped to work only with one temperature sensor. You can run two instances of the generic thermostat integration. However when the heater option is set to the same switch, then when one thermostat is turned on, the other will automatically turn on too (bc they use the same switch,  generic thermostat is programmed like that). This sometimes can lead to situations in which the thermostat of a room will turn on while cold air is flowing in because of a open window. 

Therefore a couple of input\_booleans are created and are set as heater switch. Via an automation the relay switch will be turned on if one of these input\_booleans are turned on. 

Also the someone status `input_boolean.iemandthuis` is taken into account 

###### 1.4.6.1.4.1 Living room thermostat turn on

```yaml
- id: '1606337912735'
  alias: Woonkamer thermostaat aan
  description: ''
  trigger:
  - platform: template
    value_template: '{{state_attr(''climate.woonkamer'', ''temperature'') > state_attr(''climate.woonkamer'',
      ''current_temperature'')}}'
  - platform: template
    value_template: '{{ (states.sensor.time.last_changed - states.input_boolean.iemandthuis.last_changed).total_seconds()
      > 5 }}

      '
  - platform: time_pattern
    minutes: '3'
  condition:
  - condition: template
    value_template: '{{state_attr(''climate.woonkamer'', ''temperature'')  > state_attr(''climate.woonkamer'',
      ''current_temperature'')}}'
  - condition: state
    entity_id: input_boolean.iemandthuis
    state: 'on'
  - condition: state
    entity_id: climate.woonkamer
    state: 'off'
  action:
  - service: climate.turn_on
    data: {}
    entity_id: climate.woonkamer
  mode: single
  max: 10
```

###### 1.4.6.1.4.2 Bedroom thermostat turn on / off 

```yaml
- id: '1606338268883'
  alias: 'Slaapkamer thermostaat aan '
  description: ''
  trigger:
  - platform: template
    value_template: '{{state_attr(''climate.slaapkamer'', ''temperature'') > state_attr(''climate.slaapkamer'',
      ''current_temperature'')}}'
  - platform: time_pattern
    minutes: '3'
  - platform: template
    value_template: '

      {{ (states.sensor.time.last_changed - states.input_boolean.iemandthuis.last_changed).total_seconds()
      > 5 }}

      '
  condition:
  - condition: template
    value_template: '{{state_attr(''climate.slaapkamer'', ''temperature'') > state_attr(''climate.slaapkamer'',
      ''current_temperature'')}}'
  - condition: state
    entity_id: input_boolean.iemandthuis
    state: 'on'
  action:
  - service: climate.turn_on
    data: {}
    entity_id: climate.slaapkamer
  - wait_for_trigger:
    - platform: template
      value_template: '{{state_attr(''climate.slaapkamer'', ''temperature'') < state_attr(''climate.slaapkamer'',
        ''current_temperature'')}}'
  - service: climate.turn_off
    data: {}
    entity_id: climate.slaapkamer
  mode: restart
```

#####  Living room turn thermostat off

```yaml
{% raw %}

- id: '1606839006446'
  alias: Woonkamer thermostaat uit
  description: ''
  trigger:
  - platform: template
    value_template: '{{state_attr(''climate.woonkamer'', ''temperature'') < state_attr(''climate.woonkamer'',
      ''current_temperature'')}}'
  - platform: time_pattern
    minutes: '2'
  condition:
  - condition: state
    entity_id: climate.woonkamer
    state: 'on'
  - condition: template
    value_template: '{{state_attr(''climate.woonkamer'', ''temperature'') < state_attr(''climate.woonkamer'',
      ''current_temperature'')}}'
  action:
  - service: climate.turn_off
    data: {}
    entity_id: climate.woonkamer
  mode: restart
  max: 10

- id: '1608290218329'
  alias: Bij opstaan aanwezigheid uit
  description: ''
  trigger:
  - platform: time
    at: input_datetime.opstaan
  condition: []
  action:
  - service: input_boolean.turn_off
    data: {}
    entity_id: input_boolean.iemandthuis
  mode: single
- id: '1608297641472'
  alias: terug naar instelwaarde bij restart
  description: ''
  trigger:
  - platform: time_pattern
    minutes: '15'
  condition: []
  action:
  - service: climate.set_temperature
    data:
      temperature: '{{states(''input_number.current_insteltemp_slaapkamer'')}}'
    entity_id: climate.slaapkamer
  - service: climate.set_temperature
    data:
      temperature: '{{states(''input_number.current_insteltemp_woonkamer'')}}'
    entity_id: climate.woonkamer
  mode: single
- id: '1608298248187'
  alias: Helper schakelaars voor klimaat
  description: ''
  trigger:
  - platform: state
    entity_id: input_boolean.schakelaar_slaapkamer
    to: 'on'
  - platform: state
    entity_id: input_boolean.schakelaar_woonkamer
    to: 'on'
  condition: []
  action:
  - service: switch.turn_on
    data: {}
    entity_id: switch.relay
  mode: single
- id: '1608298257006'
  alias: Helper schakelaars UIT klimaat
  description: ''
  trigger:
  - platform: time_pattern
    seconds: '5'
  - platform: state
    entity_id: input_boolean.schakelaar_slaapkamer
    to: 'off'
  - platform: state
    entity_id: input_boolean.schakelaar_woonkamer
    to: 'off'
  condition:
  - condition: state
    entity_id: input_boolean.schakelaar_slaapkamer
    state: 'off'
  - condition: state
    entity_id: input_boolean.schakelaar_woonkamer
    state: 'off'
  action:
  - service: switch.turn_off
    data: {}
    entity_id: switch.relay
  mode: single
- id: '1608318082068'
  alias: Aanwezigheid aan
  description: ''
  trigger:
  - platform: state
    entity_id: input_boolean.iemandthuis
    to: 'on'
  condition: []
  action:
  - service: climate.turn_on
    data: {}
    entity_id: climate.slaapkamer
  - service: climate.turn_on
    data: {}
    entity_id: climate.woonkamer
  mode: single
- id: '1608318174333'
  alias: Aanwezigheid uit
  description: ''
  trigger:
  - platform: state
    entity_id: input_boolean.iemandthuis
    to: 'off'
  condition: []
  action:
  - service: climate.turn_off
    data: {}
    entity_id: climate.slaapkamer
  - service: climate.turn_off
    data: {}
    entity_id: climate.woonkamer
  mode: single
- id: '1608318195860'
  alias: Aanwezigheid uit
  description: ''
  trigger:
  - platform: state
    entity_id: input_boolean.iemandthuis
    to: 'off'
  condition: []
  action:
  - service: climate.turn_off
    data: {}
    entity_id: climate.slaapkamer
  - service: climate.turn_off
    data: {}
    entity_id: climate.woonkamer
  mode: single
{% endraw %}
```

## 1.5 Links to other similar

I used these webpages for inspiration:

[Siytek - the ultimate home-assistant DIY guide for multiple zone heating](https://siytek.com/the-ultimate-home-assistant-diy-thermostat-guide-for-single-or-multiple-zone-heating/)

[Hacking a Eqiva EQ-3 thermostatic radiator valve.](https://www.youtube.com/watch?v=LlPHrdXHBTU)

An interesting idea: [using thermal sensors as room presence detection](https://www.youtube.com/watch?v=-beIaL-RmvQ)
