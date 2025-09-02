# KEYOTA Locks – Home Assistant Integration

Diese benutzerdefinierte Integration verbindet [KEYOTA](https://www.keyota.de) digitale Schließsysteme mit Home Assistant.

## Features
- Steuerung von KEYOTA-Schlössern als `lock`-Entitäten.
- **Visueller 5-Sekunden-Puls**: Beim Öffnen über HA oder bei eingehendem Webhook wird das Schloss für 5 s als „offen“ angezeigt und danach automatisch wieder „geschlossen“. (Ideal für HomeKit.)
- Webhook-Unterstützung: Ereignisse aus der KEYOTA-Cloud werden direkt verarbeitet.
- Dynamisches Hinzufügen von Schlössern, sobald sie in der API erscheinen.

## Voraussetzungen
- Home Assistant **2025.8.x** oder neuer empfohlen.
- KEYOTA **Realm ID** und **API Key**.
- Externer Zugriff auf deine HA-Instanz (für Webhooks), z. B. über HTTPS/Reverse Proxy.

## Installation
### HACS (empfohlen)
1. HACS → **Integrations** → **Custom repositories** → dein Git-Repo hinzufügen → Kategorie: *Integration*.
2. **KEYOTA Locks** installieren.
3. Home Assistant **neu starten**.

### Manuell
1. Kopiere dieses Verzeichnis nach: config/custom_components/keyota/
2. Home Assistant **neu starten**.

## Einrichtung
1. Home Assistant → **Einstellungen → Geräte & Dienste → Integration hinzufügen**.
2. **KEYOTA Locks** wählen.
3. Eingeben:
- **Realm ID**
- **API Key**
- **Base URL** (Standard: `https://api.keyota.de:5060/v1/OpenAPI`)
- **Polling-Intervall** (nur für Lock-Liste; Webhook ist push-basiert)
4. Nach erfolgreicher Einrichtung zeigt HA eine **persistente Benachrichtigung** mit deiner **Webhook-ID** an.

Beispiel-Webhook-URL: https:///api/webhook/<webhook_id>

## Webhook in KEYOTA konfigurieren
- Hinterlege die oben genannte URL im KEYOTA-Portal (GET oder POST möglich; GET empfohlen).
- KEYOTA sendet bei Ereignissen folgende Daten (Query-Parameter bei GET / JSON bei POST): EventType={EventType}&Timestamp={Timestamp}&RealmId={RealmId}&BridgeId={BridgeId}&BridgeSerialId={BridgeSerialId}&LockId={LockId}&LockSerialId={LockSerialId}

### Relevante EventType-Werte
- **Öffnend** (führt zum 5-Sekunden-Puls „offen“):  
  `Unlock`, `Open`, `Opened`, `LockDoorOpen`
- **Schließend** (setzt sofort auf „geschlossen“):  
  `Lock`, `Closed`, `LockDoorClosed`

## Verhalten & HomeKit
- **Manuelles Öffnen in HA** → echter Aktor-Aufruf; Entität zeigt 5 s „offen“, dann wieder „geschlossen“.
- **Webhook-Ereignis** (z. B. Transponder, App) → Entität zeigt 5 s „offen“, dann wieder „geschlossen“ (ohne zusätzlichen API-Aufruf).
- In HomeKit steht so immer ein stabiler Zustand zur Verfügung (keine dauerhafte Ungewissheit).

## Test & Debug
### Events live prüfen
- **Entwicklerwerkzeuge → Ereignisse**  
  bei *Zu lauschendem Ereignis* `keyota_event` eintragen und **Starten**.
- Beispiel-Test (GET):
  ```bash
  curl -G "https://<deine-ha-domain>/api/webhook/<WEBHOOK_ID>" \
    --data-urlencode "EventType=LockDoorOpen" \
    --data-urlencode "Timestamp=$(date -Iseconds)" \
    --data-urlencode "LockId=<DEIN-LOCK-ID>"
