# R-Control IP4 – MQTT Interface (Home Assistant / Generic)

This document describes the **MQTT interface** of the R-Control IP4 from an integration perspective. It targets customers/partners who want to connect the device via an MQTT broker (especially **Home Assistant MQTT Discovery**).

> Status: Implementation in `components/ha_mqtt` (firmware repo). The interface is broker-based and publish/subscribe oriented.

## 1. Overview

**Purpose**
- Publish relay states and switch relays via command topics.
- Optional: publish temperature value (NTC).
- Optional: publish button/input events as Home Assistant “Device Automation Triggers”.
- Home Assistant auto-discovery via MQTT (discovery prefix configurable).

**Characteristics**
- Transport: MQTT over TCP (URI-based, e.g. `mqtt://<broker>:1883`)
- Authentication: broker login optional (`username`/`password`)
- QoS: 0..2 (configurable)
- Retain: configurable for state topics
- Availability / LWT: set (online/offline)

## 2. Activation & Configuration

The MQTT feature is enabled via the device configuration.

### 2.1 Configuration Path
All MQTT settings are located under:
- `config.interfaces.ha_mqtt.*`

### 2.2 Configuration Keys
Required/default:
- `config.interfaces.ha_mqtt.enabled` (bool)
  - `true` = MQTT client is started
  - `false` = MQTT client is stopped
- `config.interfaces.ha_mqtt.uri` (string)
  - broker URI, e.g. `mqtt://192.168.0.2:1883`
- `config.interfaces.ha_mqtt.username` (string, optional)
- `config.interfaces.ha_mqtt.password` (string, optional)
- `config.interfaces.ha_mqtt.base_topic` (string)
  - default: `r-control-ip4`
- `config.interfaces.ha_mqtt.discovery_prefix` (string)
  - default: `homeassistant`
- `config.interfaces.ha_mqtt.qos` (int 0..2)
  - default: `1`
- `config.interfaces.ha_mqtt.retain` (bool)
  - default: `true`
- `config.interfaces.ha_mqtt.keepalive` (int, seconds)
  - range: `15..3600`
  - default: `60`

Optional temperature publishing:
- `config.interfaces.ha_mqtt.temperature.enabled` (bool)
  - default (firmware): `true`
- `config.interfaces.ha_mqtt.temperature.min_interval_ms` (int)
  - default (firmware): `10000`
  - `0` = no throttling

Note: The example `spiffs/config.json` typically contains only the base fields. The `temperature.*` values are optional and use firmware defaults if not present.

## 3. Device Identity (device_id)

The MQTT topics include a `device_id`, which is generated automatically by the firmware:
- Base is `config.network.lan.eth0.hostname`.
- The hostname is normalized to an MQTT/HA compatible ID format (lowercase, special characters → `_`).
- Additionally (if available) the last 3 bytes of the MAC address are appended.

Example (schematic):
- Hostname: `R-Control-IP4`
- device_id: `r_control_ip4_a1b2c3`

## 4. Topic Structure

### 4.1 Base Topics
With `base_topic=<BT>` and `device_id=<ID>`:

- Device availability (global):
  - `<BT>/<ID>/status`
  - payload: `online` / `offline` (retained)

- Relays (1..4):
  - State: `<BT>/<ID>/relay/<n>/state`
    - payload: `ON` / `OFF` (retained depending on `retain`)
  - Command: `<BT>/<ID>/relay/<n>/set`
    - payload: `ON` / `OFF` / `TOGGLE`

- Button events (1..4):
  - `<BT>/<ID>/button/<n>/event`
  - payloads: `pressed`, `released`, `click`, `double_click`, `long_press`, `long_press_release`
  - retain: `false`

- Temperature (optional):
  - State: `<BT>/<ID>/temperature/state`
    - payload: float as string with 2 decimals (e.g. `22.50`)
  - Availability (sensor-specific): `<BT>/<ID>/temperature/status`
    - payload: `online` (only when sensor status OK) / `offline`

### 4.2 Home Assistant Discovery Topics
With `discovery_prefix=<DP>`:

- Relays as HA switch entities:
  - `<DP>/switch/<ID>/relay_<n>/config`

- Temperature as HA sensor entity (if enabled):
  - `<DP>/sensor/<ID>/temperature/config`

- Buttons as HA device automation triggers:
  - `<DP>/device_automation/<ID>/button_<n>_<event>/config`

Discovery payloads are JSON (unformatted), retained (`true`).

## 5. Payloads & Semantics

### 5.1 Relay Command Payload
Accepted payloads on `<BT>/<ID>/relay/<n>/set`:
- `ON` → turn relay on
- `OFF` → turn relay off
- `TOGGLE` → toggle relay

The firmware accepts commands case-insensitively (internally normalized to uppercase).

### 5.2 Relay State Payload
On `<BT>/<ID>/relay/<n>/state` the firmware publishes:
- `ON` or `OFF`

Publishing occurs:
- initially on MQTT connect (snapshot)
- on relay events (state changed / impulse done / blink tick)

### 5.3 Availability (LWT)
- On connect the device publishes `online` to `<BT>/<ID>/status`.
- A last will is configured to publish `offline` if the connection drops unexpectedly.
- LWT is retained.

### 5.4 Temperature Publishing (Optional)
- Only if `config.interfaces.ha_mqtt.temperature.enabled=true`.
- Sensor availability is tracked separately:
  - `online` only for `NTC_SENSOR_STATUS_OK`
  - otherwise `offline`
- Temperature values are throttled:
  - minimum interval `temperature.min_interval_ms`
  - additionally, publishing is allowed despite throttling if the value changes by at least `0.2°C`.

## 6. Integration Notes (Best Practices)

### 6.1 Topic Design
- For custom integrations it is recommended to use only the state/command topics.
- For Home Assistant, MQTT discovery (discovery prefix) is the preferred approach.

### 6.2 Idempotency
- Idempotent: `ON`, `OFF`
- Not idempotent: `TOGGLE` (retries may cause unexpected state changes)

Recommendation: In robust systems, avoid `TOGGLE` and send deterministic `ON/OFF`.

### 6.3 QoS/Retain
- QoS `1` is a good default for LAN integrations.
- Retain `true` for relay state is useful so new subscribers immediately receive the last known state.
- Button events are intentionally not retained.

## 7. Security Notes

Security largely depends on the MQTT broker.

Recommendations:
- Enable broker authentication (user/pass) and restrict permissions per client (ACLs).
- Operate the broker only in a trusted network or use TLS/`mqtts://` (depending on broker/setup).
- Design topics / configure ACLs so that only authorized systems are allowed to publish to `.../relay/<n>/set`.

## 8. Examples

### 8.1 Example (mosquitto_sub)
```bash
# Watch all device topics
mosquitto_sub -h 192.168.0.2 -t 'r-control-ip4/#' -v

# Watch discovery
mosquitto_sub -h 192.168.0.2 -t 'homeassistant/#' -v
```

### 8.2 Example (mosquitto_pub)
```bash
# Turn relay 1 on
mosquitto_pub -h 192.168.0.2 -t 'r-control-ip4/<device_id>/relay/1/set' -m 'ON'

# Turn relay 1 off
mosquitto_pub -h 192.168.0.2 -t 'r-control-ip4/<device_id>/relay/1/set' -m 'OFF'
```

Tip: You can easily find `<device_id>` in the broker by first running `mosquitto_sub -t 'r-control-ip4/+/status' -v`.

## 9. Compatibility & Changes

The MQTT integration is Home Assistant compatible (discovery) and may be extended in future firmware versions.

For integrations with very specific requirements (e.g. complex state models), the JSON REST API or the WebSocket gateway can be used alternatively (see `docs/API.md`).
