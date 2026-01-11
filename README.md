# SmartGarageDoorButton

Ewelink WiFi Smart Relay
https://www.amazon.com/dp/B0C7QQHMCX

Sonoff DW2 WiFi Door sensor
https://www.amazon.com/dp/B08BC98RKR

Battery to AC power adapter
https://www.amazon.com/dp/B0873Y5MQ5

https://community.home-assistant.io/t/diy-23-55-alternative-to-myq/642620

## Cover in Templates.yaml
```yaml
cover:
  - name: Garage Door
    unique_id: diy_garage_door
    device_class: garage

    open_cover:
      - condition: state
        entity_id: binary_sensor.sonoff_1001d8a21c
        state: 'off'   # only open if currently closed
      - service: switch.turn_on
        target:
          entity_id: switch.sonoff_1001e92800_1

    close_cover:
      - condition: state
        entity_id: binary_sensor.sonoff_1001d8a21c
        state: 'on'    # only close if currently open
      - service: switch.turn_on
        target:
          entity_id: switch.sonoff_1001e92800_1

    stop_cover:
      - service: switch.turn_on
        target:
          entity_id: switch.sonoff_1001e92800_1

    state: >
      {% if is_state('binary_sensor.sonoff_1001d8a21c', 'on') %}
        open
      {% else %}
        closed
      {% endif %}
```
