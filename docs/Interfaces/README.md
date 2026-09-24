# Schnittstellen des R - Control IP 4

Das Gerät lässt sich auf mehreren Wegen in andere Systeme einbinden. Diese Seite hilft bei der
Auswahl — jede Schnittstelle hat ein eigenes Dokument mit Schnellstart und Beispielen.

## Welche Schnittstelle brauche ich?

| Ich möchte … | Nehmen Sie | Dokument |
|---|---|---|
| das Gerät in **Home Assistant** einbinden | MQTT | [MQTT](MQTT/md/MQTT_DE.md) |
| eine **SPS** oder **Gebäudeleittechnik** anbinden | Modbus TCP | [Modbus](Modbus/md/MODBUS_DE.md) |
| aus **eigener Software** schalten und konfigurieren | HTTP/JSON-API | [REST](REST/md/HTTP-JSON-REST-API_DE.md) |
| **benachrichtigt werden**, wenn etwas passiert | Webhook | [Webhook](Webhook/md/WEBHOOK_DE.md) |
| Zustandsänderungen **in Echtzeit** verfolgen | WebSocket | [WebSocket](Websocket/md/WEBSOCKET_DE.md) |
| eine **bestehende Installation** weiternutzen | HTTP- bzw. UDP-Legacy | [HTTP-Legacy](HTTP-Legacy/md/HTTP-LEGACY-API_DE.md) · [UDP-Legacy](UDP-Legacy/md/UDP-LEGACY-API_DE.md) |

**Im Zweifel:** Für eine eigene Anwendung ist die **HTTP/JSON-API** der richtige Einstieg. Sie
kann alles, was die Weboberfläche kann, und braucht keine zusätzliche Infrastruktur.

## Auf einen Blick

| | Richtung | Sie brauchen | Kann schalten | Echtzeit |
|---|---|---|---|---|
| **HTTP/JSON-API** | Sie fragen das Gerät | nichts | ja | nein (Abfrage) |
| **MQTT** | beide Richtungen | einen Broker | ja | ja |
| **Modbus TCP** | Sie fragen das Gerät | Modbus-Master | ja | nein (Abfrage) |
| **Webhook** | Gerät meldet an Sie | erreichbare URL | nein | ja |
| **WebSocket** | beide Richtungen | WebSocket-Client | ja | ja |
| **HTTP/UDP-Legacy** | Sie fragen das Gerät | nichts | ja | nein |

Die Legacy-Schnittstellen existieren nur aus Kompatibilitätsgründen und bieten keine moderne
Absicherung. Für neue Projekte nehmen Sie eine der anderen.

## Aktivierung

Außer der HTTP/JSON-API und WebSocket (immer verfügbar) müssen alle Schnittstellen zuerst
eingeschaltet werden:

**Weboberfläche → Einstellungen → Schnittstellen**

Dort steht je Schnittstelle auch ein Link auf die zugehörige Anleitung.

## Sprachen und Formate

Jedes Dokument liegt als Markdown (`md/`) und als PDF (`pdf/`) vor, jeweils auf Deutsch (`_DE`)
und Englisch (`_EN`).

## Interne Dokumente

| Dokument | Inhalt |
|---|---|
| [REST – Vollversion](REST/internal/HTTP-JSON-REST-API-INTERN_DE.md) | **Nicht für Kunden.** Alle Endpunkte inkl. Fertigung, Provisionierung, Factory-Lock und NTC-Kalibrierung |

Die Kundenfassung der REST-Dokumentation lässt bewusst alles weg, was im Feld nicht gebraucht
wird. Wird ein gemeinsamer Endpunkt geändert, **beide Dokumente nachziehen**.
