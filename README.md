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

## more notes
I have a yellow button Chamberlain garage door opener which means I have the Security+ 2.0. That came with the multi-function door wall control that does the door, light, locks and learn function over just the two wires. But that doesn’t give me a dry contact. So I spent $30 and bought the modified 883LM that does the light, door and gives me a dry contact 2 wire connection. I could have spent $10 and done some soldering but I didn’t feel like it. Then I ran a low voltage 2 wire from the door opener to the other side of my garage where the walkthrough from outside is and put the wall mount door opener there. From there I ran 2 wire up to a spot where I could mount a KR0548-1CH eWeLink/Sonoff switch that I bought on Amazon for $10. I’m powering the board with a 5V usb that I’m powering from an always outlet on a light socket. Then the dry contact from the modified 883LM is going to the Normally Open (NO) and Common (COM) on the relay module.

In the eWeLink app I configured the device to act as a switch, enabled LAN control, and turned on the inching settings set to 0.5sec. This gives me a chance to toggle the switch just log enough to make the contact to act as if the button was pushed.

Then without changing any firmware I was able to add the eWeLink Smart Home device to my Home Assistant server using the add-on in HACS as well as the SonoffLAN custom component to allow it to work even without the cloud.

As an added bonus I was able to get the eWeLink added to Alexa and I used the Home Assistant Home Bridge to add it to Apple Home. I have a camera in my garage that is next to the button I added to my dashboard and my phone still has the MyQ app to doublecheck the status of the door. In Alexa I created an automation that says push garage door button rather than garage open or closed since the inching just makes the contact and then goes back to open.
