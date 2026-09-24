# Interfaces of the R - Control IP 4

The device can be integrated into other systems in several ways. This page helps you choose —
each interface has its own document with a quick start and examples.

## Which interface do I need?

| I want to … | Use | Document |
|---|---|---|
| integrate the device into **Home Assistant** | MQTT | [MQTT](MQTT/md/MQTT_EN.md) |
| connect a **PLC** or **building management system** | Modbus TCP | [Modbus](Modbus/md/MODBUS_EN.md) |
| switch and configure from **my own software** | HTTP/JSON API | [REST](REST/md/HTTP-JSON-REST-API_EN.md) |
| **be notified** when something happens | Webhook | [Webhook](Webhook/md/WEBHOOK_EN.md) |
| follow state changes **in real time** | WebSocket | [WebSocket](Websocket/md/WEBSOCKET_EN.md) |
| keep an **existing installation** running | HTTP or UDP legacy | [HTTP legacy](HTTP-Legacy/md/HTTP-LEGACY-API_EN.md) · [UDP legacy](UDP-Legacy/md/UDP-LEGACY-API_EN.md) |

**When in doubt:** For your own application the **HTTP/JSON API** is the right starting point. It
can do everything the web interface can and needs no additional infrastructure.

## At a glance

| | Direction | You need | Can switch | Real time |
|---|---|---|---|---|
| **HTTP/JSON API** | you ask the device | nothing | yes | no (polling) |
| **MQTT** | both directions | a broker | yes | yes |
| **Modbus TCP** | you ask the device | Modbus master | yes | no (polling) |
| **Webhook** | device reports to you | a reachable URL | no | yes |
| **WebSocket** | both directions | WebSocket client | yes | yes |
| **HTTP/UDP legacy** | you ask the device | nothing | yes | no |

The legacy interfaces exist for compatibility only and offer no modern protection. For new
projects use one of the others.

## Enabling

Except for the HTTP/JSON API and WebSocket (always available), all interfaces have to be enabled
first:

**Web interface → Settings → Interfaces**

Each interface there also links to its manual.

## Languages and formats

Every document is available as Markdown (`md/`) and PDF (`pdf/`), in German (`_DE`) and English
(`_EN`).

## Internal documents

| Document | Content |
|---|---|
| [REST – full version](REST/internal/HTTP-JSON-REST-API-INTERN_EN.md) | **Not for customers.** All endpoints including production, provisioning, factory lock and NTC calibration |

The customer edition of the REST documentation deliberately omits everything that is not needed
in the field. When a shared endpoint changes, **update both documents**.
