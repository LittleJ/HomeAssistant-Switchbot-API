# Home Assistant - Switchbot API

Connect Home Assistant to Switchbot API

## SECRETS.YAML

```
switchbot_meter1_status_url: "https://api.switch-bot.com/v1.0/devices/<DEVICE ID>/status"
switchbot_contactsensor1_status_url: "https://api.switch-bot.com/v1.0/devices/<DEVICE ID>/status"
switchbot_api: <TOKEN>
switchbot_lightstrip_deviceId: <DEVICE ID>
```


## CONFIGURATION.YAML

```
# CONNECT TO API
rest_command:
  switchbot_device_command:
    url: 'https://api.switch-bot.com/v1.0/devices/{{ deviceId }}/commands'
    method: post
    content_type: 'application/json'
    headers:
      Authorization: !secret switchbot_api
    payload: '{"command": "{{ command }}","parameter": "{{ parameter }}","commandType": "{{ commandType }}"}'

# SWITCHBOT LIGHTSTRIP
light:
  - platform: template
    lights:
      switchbot_lightstrip:
        friendly_name: Ruban chambre
        unique_id: Ruban chambre
        turn_on:
          service: rest_command.switchbot_device_command
          data:
            deviceId: !secret switchbot_lightstrip_deviceId
            command: "turnOn"
        turn_off:
          service: rest_command.switchbot_device_command
          data:
            deviceId: !secret switchbot_lightstrip_deviceId
            command: "turnOff"
        set_level:
          service: rest_command.switchbot_device_command
          data:
            deviceId: !secret switchbot_lightstrip_deviceId
            command: "setBrightness"
            parameter: "{{brightness}}"
        set_color:
          service: rest_command.switchbot_device_command
          data:
            deviceId: !secret switchbot_lightstrip_deviceId
            command: "setColor"
            parameter: "{{rgb_color}}"

# SWITCHBOT CONTACT SENSOR
binary_sensor:
  - platform: rest
    name: 'Contact Sensor 1 JSON'
    resource: !secret switchbot_contactsensor1_status_url
    method: GET
    # Refresh every 30 seconds
    scan_interval: 30
    headers:
      Authorization: !secret switchbot_api
      Content-Type: 'application/json'
    value_template: '{{ value_json.body }}'
  - platform: template
    sensors:
      switchbot_contactsensor1_openstate:
        friendly_name: "Contact sensor 1 Open State"
        value_template: '{{ states.sensor.meter1_json.attributes["openState"] }}'
        device_class: "opening"

# SWITCHBOT METER
sensor:
  - platform: rest
    name: 'Meter1 JSON'
    resource: !secret switchbot_meter1_status_url
    method: GET
    # Refresh every 5 minutes
    scan_interval: 300
    headers:
      Authorization: !secret switchbot_api
      Content-Type: 'application/json'
    value_template: '{{ value_json.body }}'
    json_attributes_path: "$.body"
    json_attributes:
      - deviceId 
      - deviceType
      - hubDeviceId
      - humidity
      - temperature
  - platform: template
    sensors:
      switchbot_meter1_temp:
        friendly_name: "Meter1 Temperature"
        value_template: '{{ states.sensor.meter1_json.attributes["temperature"] }}'
        unit_of_measurement: "°C"
        device_class: "temperature"
      switchbot_meter1_humidity:
        friendly_name: "Meter1 Humidity"
        value_template: '{{ states.sensor.meter1_json.attributes["humidity"] }}'
        unit_of_measurement: "%"
        device_class: "humidity"

```
