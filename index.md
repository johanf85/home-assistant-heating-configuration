# My home smart heating configuration with the use of Home Assistant

My appartement consists of a living room, a bedroom and a kitchen.

It has a boiler for central heating which can use a on-off control and OpenTherm.  

### The design wishes for the system

*   Multi zone: Heat the bedroom to a desired temperature at night (while living room radiator closed) and the living room during the day and evening (while bedroom radiator closed)
*   Turn off the thermostat function when no one home
*   A wired system, that doesn't use wifi, cause I want to limit the use of EMF-radiation in my home
*   Use a smartphone app and / or a browser to control and monitor the thermostat
*   Send alerts to the smartphone when certain conditions are met, e.g. when it is suspected that a window is open.

### Software

[Home Assistant](https://www.home-assistant.io/)

#### Home assistant integrations

*   [Generic Thermostat integration](https://www.home-assistant.io/integrations/generic_thermostat/)

### Hardware

Bedroom:

*   Arduino with a DS18B20 temperature sensor connected via USB

Living room:

*   Raspberry pi zero with
    *   DS18B20 temperature sensor
    *   PIR motion sensor
    *   Relay module with wire for controlling on-off heating boiler

Both rooms have one radiator, each is equipped with a eqiva-N thermostatic valve. 

![](https://user-images.githubusercontent.com/43075793/102866119-1029d880-4437-11eb-9000-4aa8b5e9782c.png)

There were no thermostatic valves on my radiator (just a turn knob), so I had to put in a new insert for my Herz radiator valve:

![](https://user-images.githubusercontent.com/43075793/102867026-6d725980-4438-11eb-9752-5bd6fffe2686.png)

Deuren

### Configuration.yaml

Only covering the relevant part of the configuration for the smart heating system. 

#### For the generic thermostat integration

Home Assistant documentation: [Generic thermostat](https://www.home-assistant.io/integrations/generic_thermostat/)

Choices for configuration variables:

*   `target_temp` I chose target temps lower than I actually want, but with a sudden a restart of the system I don't want the heating to turn on. I use automations to override the temperature to set these to desired temperatures.
*   `min_cycle_duration` Set so that pump and gas furnace of boiler don't have to turn on and off that often. Set to 3 minutes for bedroom and 1 minute for living room. The living room has a larger radiator and 1 minute turned out as a good value. For the bedroom 3 minutes as the there is a smaller radiator.
*   `heater` For each room I used a [Helper](https://www.home-assistant.io/integrations/input_boolean/), input\_boolean switch, because Generic thermostat isn't able to work with multiple zones (described [here](https://community.home-assistant.io/t/need-help-with-multi-zone-generic-thermostat-climate-configuration/8563)) when one heater switch is on both rooms (they will contradict). Trough an automation I made sure these helpers are controlling the relay switch, see [automations](#automations).

```
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

#### Telegram integration

I use Telegram for notifications about hours of heating during a week and notifications when the heating is automatically turned of because of a suspected open window. 

Home Assistant documentation: https://www.home-assistant.io/integrations/telegram_polling/

```
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

#### Correction of temperature sensors

Noticed that my DS18b20 sensors weren't correct when set up. With the use of [template platform](https://www.home-assistant.io/integrations/template/) a correction is applied. 

See below:

```
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
  - platform: history_stats
    name: Aantal minuten verwarmen laatste 7 dagen
    entity_id: sensor.heating_state
    state: 'heating'
    type: time
    end: '{{ now().replace(hour=0, minute=0, second=0) }}'
    duration:
        days: 7
```

### Trend sensor for possible open window detection

Using the [trend platform](https://www.home-assistant.io/integrations/trend/) it is checked if the temperature will rise enough while heating. If not, it is assumed that a window is open or some other error and the heating is turned off. See [automations](#automations). 

The `min_gradient` values for each room are set based on experience. 

```
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
```

## Automations

```
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
- id: '1587319960331'
  alias: Aanschakelen iemand thuis bij beweging
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
- id: '1587319961411'
  alias: Gedrag bewegingssensor woonkamer tussen opstaan en avond (overdag)
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
- id: '1587404974211'
  alias: Aanwezigheid detectie avond tot opstaan
  description: ''
  trigger:
  - platform: template
    value_template: '{{ (states.sensor.time.last_changed - states.input_datetime.bewegingeennalaatst_1.last_changed).total_seconds()
      == 300 }}

      '
  condition:
  - condition: state
    entity_id: input_boolean.iemandthuis
    state: 'off'
  - before: input_datetime.opstaan
    condition: time
    after: input_datetime.avond
  - condition: template
    value_template: '{{ (states.sensor.time.last_changed - states.input_datetime.bewegingeennalaatst_1.last_changed).total_seconds()
      < 300 }}

      '
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
        open
      target: [chat-number]
  - data: {}
    entity_id: climate.slaapkamer
    service: climate.turn_off
  - delay: 00:45:00
  - service: climate.turn_on
    data: {}
    entity_id: climate.slaapkamer
  mode: single
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
- id: '1600599019863'
  alias: Terugzetten na temperatuurverhoging
  description: ''
  trigger:
  - platform: template
    value_template: '{{ (states.sensor.time.last_changed - states.climate.slaapkamer.last_changed).total_seconds()
      > 7200 }}'
  - platform: template
    value_template: '{{ (states.sensor.time.last_changed - states.climate.woonkamer.last_changed).total_seconds()
      > 7200 }}'
  condition: []
  action:
  - condition: state
    entity_id: climate.slaapkamer
    state: '21.5'
  - condition: state
    entity_id: climate.woonkamer
    state: '21.0'
  mode: single
- id: '1601214055496'
  alias: Relay overgang off-on set schakelaar
  description: ''
  trigger:
  - platform: state
    entity_id: switch.relay
    to: 'on'
  condition: []
  action:
  - service: input_boolean.turn_on
    data: {}
    entity_id: input_boolean.slaapkamer_overgang_off_on_thermostaat
  mode: single
- id: '1601214055496'
  alias: Relay overgang on-off set schakelaar
  description: ''
  trigger:
  - platform: state
    entity_id: switch.relay
    from: 'on'
    to: 'off'
  condition: []
  action:
  - condition: state
    entity_id: input_boolean.slaapkamer_overgang_off_on_thermostaat
    state: 'off'
  mode: single
- id: '1601214055496'
  alias: Relay overgang on-off set schakelaar
  description: ''
  trigger:
  - platform: state
    entity_id: switch.relay
    from: 'on'
    to: 'off'
  condition: []
  action:
  - condition: state
    entity_id: input_boolean.slaapkamer_overgang_off_on_thermostaat
    state: 'off'
  mode: single
- id: '1601214594628'
  alias: Relay overgang off-on set schakelaar (Dupliceren)
  description: ''
  trigger:
  - platform: state
    entity_id: switch.relay
    from: 'off'
    to: 'on'
  condition: []
  action:
  - condition: state
    entity_id: input_boolean.slaapkamer_overgang_off_on_thermostaat
    state: 'on'
  mode: single
- id: '1601214728176'
  alias: Relay on-off set schakelaar
  description: ''
  trigger:
  - platform: state
    entity_id: switch.relay
    from: 'on'
    to: 'off'
  condition: []
  action:
  - service: input_boolean.turn_off
    data: {}
    entity_id: input_boolean.slaapkamer_overgang_off_on_thermostaat
  mode: single
- id: '1601214996076'
  alias: Relay beveiging raam open (Dupliceren)
  description: ''
  trigger:
  - platform: template
    value_template: '{{ (states.sensor.time.last_changed  - states.switch.relay.last_changed).total_seconds()
      > 310 }}'
  condition:
  - condition: state
    entity_id: binary_sensor.temp_falling
    state: 'off'
  - condition: state
    entity_id: automation.slaapkamer_overgang_off_on_set_schakelaar
    state: 'on'
  action:
  - data: {}
    entity_id: climate.slaapkamer
    service: climate.turn_off
  - delay: 00:59:00
  - service: climate.turn_on
    data: {}
    entity_id: climate.slaapkamer
  mode: single
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
- id: '1605216873548'
  alias: Relay beveiging raam open Woonkamer
  description: ''
  trigger:
  - platform: time_pattern
    minutes: '5'
  condition:
  - condition: state
    entity_id: binary_sensor.temp_falling_woonkamer
    state: 'off'
  - condition: template
    value_template: '{{state_attr(''climate.woonkamer'', ''temperature'') | float
      > state_attr(''climate.woonkamer'', ''current_temperature'')}}'
  - condition: state
    entity_id: switch.relay
    state: 'on'
  - condition: template
    value_template: '{{ (states.sensor.time.last_changed  - states.switch.relay.last_changed).total_seconds()
      > 300 }}'
  action:
  - service: telegram_bot.send_message
    data:
      message: Woonkamer thermostaat uitgeschakeld ivm te langzame opwarming, raam
        open?
      target: 1212957007
  - data: {}
    entity_id: climate.woonkamer
    service: climate.turn_off
  - delay: 00:45:00
  - service: climate.turn_on
    data: {}
    entity_id: climate.woonkamer
  mode: single
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
- id: '1606337919641'
  alias: Woonkamer thermostaat aan
  description: ''
  trigger:
  - platform: time_pattern
    minutes: '5'
  condition:
  - condition: template
    value_template: '{{state_attr(''climate.woonkamer'', ''temperature'') > state_attr(''climate.woonkamer'',
      ''current_temperature'')}}'
  - condition: state
    entity_id: input_boolean.iemandthuis
    state: 'on'
  action:
  - service: climate.turn_on
    data: {}
    entity_id: climate.woonkamer
  mode: single
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
- id: '1606672315270'
  alias: Bewegingssensor laatst naar helper
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
- id: '1606905142912'
  alias: Reset 1 na laatste beweging 31 min voor opstaan
  description: ''
  trigger:
  - platform: time_pattern
    seconds: '30'
  condition:
  - condition: template
    value_template: '{/% set current_time = now().hour * 60 + now().minute %}

      {% set opstaan_hour, opstaan_minute, opstaan_second = states(''input_datetime.opstaan'').split('':'')
      %}

      {% set opstaan_time = opstaan_hour | int * 60 + opstaan_minute | int %}

      {{ current_time == opstaan_time - 32 }}'
  action:
  - service: input_datetime.set_datetime
    data:
      datetime: '{{now().strftime(''%Y-%m-%d %H:%M:%S'')}}'
    entity_id: input_datetime.bewegingeennalaatst_1
  mode: restart
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
```
