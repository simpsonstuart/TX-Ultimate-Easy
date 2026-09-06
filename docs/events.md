# Using TX Ultimate Easy Events

## Overview of the Event System

TX Ultimate Easy publishes each physical button as a native Home Assistant
event entity. Use these entities for new button automations. Native event
entities use the regular ESPHome entity connection, do not require permission
for the device to perform Home Assistant actions, and are not represented by a
short on/off state.

The firmware also continues to emit the legacy `esphome.tx_ultimate_easy`
event-bus event for existing automations and for touch gestures that do not yet
have their own event entity.

### Native Button Event Entities (Recommended)

Each configured gang exposes one entity:

- `Button 1 event`
- `Button 2 event`
- `Button 3 event`
- `Button 4 event`

The supported event types are `click`, `double_click`, `long_press`, and
`multiple_click`. In Home Assistant's automation editor, select **Event
received**, choose the button event entity, and select `click` as the event
type.

For example, this automation toggles a Zigbee light while leaving the TX
Ultimate relay alone:

```yaml
triggers:
  - trigger: event.received
    target:
      entity_id: event.living_room_switch_button_1_event
    options:
      event_type:
        - click
actions:
  - action: light.toggle
    target:
      entity_id: light.living_room
```

Replace both entity IDs with the IDs from your Home Assistant instance. Set
**Button 1 action** to **None** on the TX Ultimate device when the physical
relay must remain on.

### Legacy Event-Bus Structure

All events emitted by the device are categorized under the `esphome.tx_ultimate_easy` event type in Home Assistant.
The `data` field within the event payload provides details about the specific interaction.
If you keep using this legacy route, open the ESPHome integration's
configuration for the device and enable **Allow the device to perform Home
Assistant actions**.
Below is a detailed description of the keys used:

#### Example Event Payload

```yaml
  event:
    event_type: esphome.tx_ultimate_easy
    data:
      device_id: "abc123"
      device_name: "living_room_panel"
      firmware: "2024.12.1"
      domain: touch
      type: button
      action: click
      button_id: 2
      count: 1
      position: 3
```

### Key Definitions

#### `device_id`

- **Description**: A unique identifier for the device emitting the event.
- **Example**: `device_id: "abc123"`

#### `device_name`

- **Description**: A Home Assistant-compatible version of the hostname, with blanks and invalid characters replaced by `_`.
- **Example**: `device_name: "living_room_panel"`

#### `firmware`

- **Description**: The firmware version of the device, formatted as `YYYY.M.S`.
- **Example**: `firmware: "2024.12.1"`

#### `domain`

- **Description**: The high-level category of the event.
- **Examples**:
  - `boot`
  - `touch`

#### `type`

- **Description**: Specifies the subcategory within the `domain`.
- **Examples**:
  - For `domain: boot`: `start`, `done`
  - For `domain: touch`: `button`, `swipe`, `multi_touch`

#### `action`

- **Description**: Indicates the specific action or interaction.
- **Examples**:
  - For `type: button`: `click`, `double_click`, `long_press`, `multiple_click`
  - For `type: swipe`: `left`, `right`, `up`, `down`

#### `button_id`

- **Description**: Identifies the button involved in the event (specific to `type: button`).
- **Example**: `button_id: 2`

#### `count`

- **Description**: Indicates the number of clicks or interactions (used with `action: multiple-click`).
- **Example**: `count: 3`

#### `swipe-direction`

- **Description**: Specifies the direction of a swipe (used with `type: swipe`).
- **Examples**: `left`, `right`, `up`, `down`

#### `position`

- **Description**: An integer from 1 to 10 indicating the touch position (available on all touch events).
- **Example**: `position: 3`

### Supported Event Types

#### Boot Events

```yaml
  event:
    device_id: "abc123"
    device_name: "living_room_panel"
    firmware: "2024.12.1"
    domain: boot
    type: start
    action: null
```

```yaml
  event:
    device_id: "abc123"
    device_name: "living_room_panel"
    firmware: "2024.12.1"
    domain: boot
    type: done
    action: null
```

#### Touch Events

##### Button Event

```yaml
  event:
    device_id: "abc123"
    device_name: "living_room_panel"
    firmware: "2024.12.1"
    domain: touch
    type: button
    action: click
    button_id: 1
    count: 1
    position: 3
```

```yaml
  event:
    device_id: "abc123"
    device_name: "living_room_panel"
    firmware: "2024.12.1"
    domain: touch
    type: button
    action: multiple-click
    button_id: 1
    count: 5
    position: 3
```

##### Swipe Event

```yaml
  event:
    device_id: "abc123"
    device_name: "living_room_panel"
    firmware: "2024.12.1"
    domain: touch
    type: swipe
    action: left
    swipe-direction: left
    position: 3
```

```yaml
  event:
    device_id: "abc123"
    device_name: "living_room_panel"
    firmware: "2024.12.1"
    domain: touch
    type: swipe
    action: up
    swipe-direction: up
    position: 3
```

##### Multi-Touch Event

```yaml
  event:
    device_id: "abc123"
    device_name: "living_room_panel"
    firmware: "2024.12.1"
    domain: touch
    type: multi_touch
    action: release
    position: 3
```

### Legacy Integration in Automations

#### Listening to Legacy Events

To utilize these events in Home Assistant, follow these steps:

1. Open the **Developer Tools** in Home Assistant.
2. Navigate to the **Events** tab.
3. Enter `esphome.tx_ultimate_easy` in the "Event to subscribe to" field.
4. Click "Start listening."
5. Trigger interactions on your TX Ultimate Easy device to see the events in real-time.

#### Legacy Automation YAML

##### Automation for a Single Click

```yaml
trigger:
  - platform: event
    event_type: esphome.tx_ultimate_easy
    event_data:
      domain: touch
      type: button
      action: click
      button_id: 1
      position: 3
action:
  - service: light.toggle
    target:
      entity_id: light.living_room
```

##### Automation for a Swipe Gesture

```yaml
trigger:
  - platform: event
    event_type: esphome.tx_ultimate_easy
    event_data:
      domain: touch
      type: swipe
      action: left
      swipe-direction: left
      position: 3
action:
  - service: script.activate_scene
    target:
      entity_id: scene.relax_mode
```

This event engine provides a robust and extensible framework for handling device interactions,
allowing users to implement highly customized automations with ease.
