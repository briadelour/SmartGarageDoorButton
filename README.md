# DIY Smart Garage Door Controller

A cost-effective alternative to MyQ using Home Assistant, eWeLink smart relay, and Sonoff door sensor. Total cost: ~$23-55.

<img width="501" height="274" alt="image" src="https://github.com/user-attachments/assets/8d918e7c-5508-45c8-9b0f-8a071f630616" />

Based on [this Home Assistant community guide](https://community.home-assistant.io/t/diy-23-55-alternative-to-myq/642620).

## Overview

This project creates a smart garage door controller that:
- Works with Home Assistant, Alexa, and Apple HomeKit
- Operates locally without cloud dependency (using SonoffLAN)
- Provides door status monitoring via a door sensor
- Costs significantly less than commercial solutions like MyQ
- If you want something more polished and willing to spend more, I highly recommend checking out the [ratgdo32](https://ratcloud.llc/).

## Parts List

| Item | Price | Link |
|------|-------|------|
| eWeLink WiFi Smart Relay 1CH-10A | ~$10 | [Amazon](https://www.amazon.com/dp/B0C7QQHMCX) |
| USB wire and 5V power brick | $0 | sitting around the house |
| low voltage (22awg) wire for garage door sensors | $0 | found extra in garage |
| Sonoff DW2 WiFi Door Sensor | ~$8 | [Amazon](https://www.amazon.com/dp/B08BC98RKR) |
| Battery to AC Power Adapter (optional) | ~$5 | [Amazon](https://www.amazon.com/dp/B0873Y5MQ5) |
| Modified Chamberlain 883LM (for Security+ 2.0 openers) | ~$30 | Provides dry contact terminals |

**Note 1:** If you have a Security+ 2.0 opener (yellow button Chamberlain), you'll need the modified 883LM wall control to get dry contact terminals. Alternatively, you can modify your existing wall control with some soldering to save ~$20.

**Note 2:** I originally had an outdoor outlet cover as a case for the smart relay but that was too large and I didn't like the look.  So I found that you can modify a Harry's Razor Blade Refill plastic container and it works well as a compact housing.

<div align=center>
![IMG_0066](https://github.com/user-attachments/assets/33b9a775-b10e-40fa-ba3d-3876914b96e3)
</div>

## Wiring Setup

### For Chamberlain Security+ 2.0 Openers

1. **Install the 883LM Wall Control**
   - Mount the modified 883LM wall control near your garage door opener
   - Connect the control to your garage door opener per manufacturer instructions
   - This provides light control, door control, and most importantly, dry contact terminals

2. **Wire the eWeLink Relay**
   - Run low-voltage 2-wire cable from the 883LM dry contact terminals to your relay location
   - Connect the dry contact wires to the **Normally Open (NO)** and **Common (COM)** terminals on the relay
   - Power the relay using a 5V USB adapter plugged into a nearby outlet

3. **Install the Door Sensor**
   - Mount the Sonoff DW2 sensor on the garage door and frame
   - Follow the sensor's pairing instructions to add it to your eWeLink app

### General Setup (Any Garage Door Opener)

- Connect your relay's dry contact terminals to your garage door opener's wall button terminals
- Power the relay with a 5V USB power supply
- Position the relay near a power outlet and within WiFi range

## eWeLink App Configuration

1. **Add the Relay**
   - Open the eWeLink app and add your smart relay device
   - Configure it as a **Switch**
   
2. **Enable Local Control**
   - Go to device settings
   - Enable **LAN Control** (allows local operation without internet)
   
3. **Configure Inching Mode**
   - Enable **Inching** mode in device settings
   - Set the duration to **0.5 seconds**
   - This simulates a momentary button press instead of a continuous on/off switch

4. **Add the Door Sensor**
   - Pair the Sonoff DW2 door sensor
   - The sensor will report "on" when the door is open and "off" when closed

## Home Assistant Configuration

### Prerequisites

Install one of the following via HACS:
- **eWeLink Smart Home** official integration
- **SonoffLAN** custom component (enables local control without cloud), https://github.com/AlexxIT/SonoffLAN
(I prefer the SonoffLAN so it's all local)

### Add Template Cover

Add this configuration to your `templates.yaml` or `configuration.yaml`:

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

**Important:** Replace the entity IDs with your actual device entity IDs:
- `binary_sensor.sonoff_1001d8a21c` - Your door sensor
- `switch.sonoff_1001e92800_1` - Your relay switch

### Dashboard Card

Add this card to your dashboard for a clean interface:

```yaml
type: entities
entities:
  - entity: switch.sonoff_1001e92800_1
    name: Virtual Garage Door Button
    icon: mdi:button-pointer
  - entity: binary_sensor.sonoff_1001d8a21c
    name: Garage Door Sensor
  - entity: cover.garage_door
  - entity: sensor.sonoff_1001d8a21c_battery
title: Garage Door
state_color: true
```

## Voice Assistant Integration

### Alexa

1. Add the eWeLink skill to Alexa
2. Discover devices
3. Create a routine with a custom phrase like "push garage door button" instead of "open" or "close" since the relay just simulates a button press

### Apple HomeKit

1. Install the **Home Assistant HomeKit Bridge** integration
2. Add the garage door cover entity to the bridge
3. The garage door will appear in Apple Home

## Features

- ✅ Local control (works even if internet is down)
- ✅ Real-time door status monitoring
- ✅ Battery status monitoring for door sensor
- ✅ Integration with Alexa, Apple HomeKit, and Google Assistant
- ✅ Works alongside existing garage door remotes and wall buttons
- ✅ Visual confirmation in Home Assistant dashboard
- ✅ Can be combined with garage camera for visual monitoring

## Troubleshooting

**Relay doesn't trigger the door:**
- Verify inching mode is enabled and set to 0.5 seconds
- Check wire connections to NO and COM terminals
- Test the relay manually in the eWeLink app

**Door sensor not updating:**
- Check battery level
- Verify the sensor is within WiFi range
- Adjust sensor mounting for better magnet alignment

**No local control:**
- Ensure LAN mode is enabled in eWeLink app
- Verify SonoffLAN component is installed in Home Assistant
- Check that devices are on the same network

## Safety Notes

- This project interfaces with garage door electrical systems - ensure you understand your garage door opener's wiring before proceeding
- Test thoroughly before relying on it as your primary access method
- Keep backup access methods (physical remotes, wall button) functional
- The relay only simulates a button press - your garage door's safety features remain active

## License

Feel free to use and modify this project. Attribution appreciated but not required.

## Contributing

Found an improvement or have suggestions? Feel free to open an issue or pull request!
