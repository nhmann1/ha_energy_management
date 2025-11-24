# Home Assistant Automatisierungen für Huawei LUNA2000 Batterie

Dieses Repository enthält zwei Home Assistant Automatisierungen zur intelligenten Steuerung der Huawei LUNA2000 Batterie. Die Automatisierungen berechnen täglich den optimalen Ladeplan basierend auf Solarprognose und Strompreisen und führen diesen automatisch aus, um die Energiekosten zu minimieren.

## Übersicht

Die Automatisierungen arbeiten zusammen, um die Batterie optimal zu laden:
1. **Ladeplan berechnen**: Berechnet täglich um 18:00 Uhr den günstigsten Ladeplan
2. **Netzladung ausführen**: Führt den berechneten Ladeplan automatisch aus

## Automatisierungen

### 1. Akku Ladeplan berechnen (Huawei LUNA2000)

**Datei**: `automation.akku_ladeplan_berechnen_huawei_luna2000.yaml`

**Funktion**: 
Berechnet täglich um 18:00 Uhr den günstigsten Ladeplan für den Huawei LUNA2000-Akku.

**Berechnungsgrundlagen**:
- Solarprognose für den nächsten Tag (`sensor.energy_production_tomorrow_2`)
- Aktueller Batterie-SOC (`sensor.batterien_batterieladung`)
- Strompreise (`sensor.ostrom_energy_spotpreis`)
- Täglicher Energiebedarf (11 kWh)
- Batteriekapazität (15 kWh)

**Funktionsweise**:
1. Berechnet den fehlenden Energiebedarf für den nächsten Tag
2. Sortiert die Strompreise nach günstigsten Stunden
3. Erstellt einen Ladeplan mit optimaler Start- und Endzeit
4. Speichert den Plan in `input_text.akku_ladeplan_json`

**Besonderheiten**:
- **Zwischenladen**: Wenn der SOC unter 20% fällt und die günstigste Zeit erst in mehr als 4 Stunden ist, wird sofort bis 30% geladen
- **Minimaler SOC**: Ziel-SOC wird auf mindestens 40% gesetzt
- **Benachrichtigungen**: Sendet Push-Benachrichtigungen an alle konfigurierten Geräte

**Konfigurierbare Variablen**:
- `daily_consumption`: 11 kWh (täglicher Energiebedarf)
- `battery_capacity`: 15 kWh (Batteriekapazität)
- `min_soc`: 15% (minimaler SOC)
- `min_soc_sicher`: 20% (Schwellwert für Zwischenladen)
- `reserve_soc`: 30% (Ziel-SOC beim Zwischenladen)
- `sicherheits_offset`: 4 Stunden (Mindestabstand für Zwischenladen)
- `charge_power_kw`: 5 kW (Ladeleistung)

### 2. Akku Netzladung gemäß Plan (Huawei LUNA2000)

**Datei**: `automation.akku_netzladung_ausfuhren_huawei_luna2000.yaml`

**Funktion**: 
Führt den von der ersten Automatisierung berechneten Ladeplan automatisch aus.

**Trigger**: 
Wird ausgelöst, wenn sich `input_text.akku_ladeplan_json` ändert.

**Funktionsweise**:

**Zwischenladen (sofort)**:
- Wenn `start_time` = "JETZT", startet das Laden sofort
- Lädt bis zum definierten Ziel-SOC (typischerweise 30%)
- Nutzt `huawei_solar.forcible_charge_soc` Service

**Reguläres Laden (zeitgesteuert)**:
- Wartet bis zur berechneten Startzeit
- Schaltet den Netzlade-Switch ein
- Startet das Laden mit `huawei_solar.forcible_charge_soc`
- Stoppt automatisch wenn:
  - Ziel-SOC erreicht wurde (SOC ≥ Ziel-SOC - 1%)
  - Oder Endzeit erreicht wurde
- Schaltet den Netzlade-Switch wieder aus

**Konfigurierbare Variablen**:
- `charge_power`: 5000 W (Ladeleistung)
- `inverter_device_id`: Device-ID des Huawei Inverters
- `stop_threshold`: Ziel-SOC - 1% (Stopp-Schwellwert)

## Voraussetzungen

### Home Assistant Entities

Die Automatisierungen benötigen folgende Entities:

**Sensoren**:
- `sensor.energy_production_tomorrow_2` - Solarprognose für morgen
- `sensor.batterien_batterieladung` - Aktueller Batterie-SOC
- `sensor.ostrom_energy_spotpreis` - Strompreise (mit `prices` Attribut)
- `sensor.huawei_inverter_battery_soc` - Batterie-SOC für Stopp-Erkennung

**Inputs**:
- `input_text.akku_ladeplan_json` - Speichert den berechneten Ladeplan als JSON

**Switches**:
- `switch.batterien_laden_aus_dem_netz` - Schalter für Netzladen

**Services**:
- `huawei_solar.forcible_charge_soc` - Startet erzwungenes Laden bis Ziel-SOC
- `huawei_solar.stop_forcible_charge` - Stoppt erzwungenes Laden

**Notifications** (optional):
- `notify.mobile_app_iphone_von_peter`
- `notify.mobile_app_nils_iphone`
- `notify.mobile_app_nils_iphone13`
- `notify.mobile_app_sm_g990b`

### Integrationen

- **Huawei Solar Integration**: Für die Steuerung der Batterie
- **OStrom Integration** (oder ähnlich): Für Strompreisdaten

## Installation

1. Kopiere die beiden YAML-Dateien in dein Home Assistant `config/automations/` Verzeichnis
2. Stelle sicher, dass alle benötigten Entities und Services vorhanden sind
3. Passe die Variablen in beiden Automatisierungen an deine Gegebenheiten an:
   - Batteriekapazität
   - Täglicher Energiebedarf
   - Device-ID des Inverters
   - Benachrichtigungs-Services
4. Aktiviere die Automatisierungen in Home Assistant

## Ladeplan JSON Format

Der Ladeplan wird im folgenden JSON-Format gespeichert:

```json
{
  "start": "18:00",
  "end": "22:00",
  "energy": 5.2,
  "avg_price": 0.125,
  "soc_target": 65
}
```

**Felder**:
- `start`: Startzeit (HH:MM) oder "JETZT" für Zwischenladen
- `end`: Endzeit (HH:MM) oder "ca. 1h" für Zwischenladen
- `energy`: Benötigte Energie in kWh
- `avg_price`: Durchschnittlicher Strompreis in €/kWh
- `soc_target`: Ziel-SOC in Prozent

## Anpassungen

### Batteriekapazität ändern
Ändere `battery_capacity` in der ersten Automatisierung (Standard: 15 kWh).

### Täglichen Energiebedarf anpassen
Ändere `daily_consumption` in der ersten Automatisierung (Standard: 11 kWh).

### Ladeleistung anpassen
Ändere `charge_power_kw` in der ersten Automatisierung und `charge_power` in der zweiten (Standard: 5 kW / 5000 W).

### Inverter Device-ID
Finde die Device-ID deines Huawei Inverters und passe `inverter_device_id` in der zweiten Automatisierung an.

### Benachrichtigungen
Passe die `notify.*` Services in beiden Automatisierungen an deine konfigurierten Geräte an.

## Logging

Beide Automatisierungen schreiben detaillierte Logs in das Home Assistant System-Log. Die Logs enthalten:
- Berechnete Werte (Preise, SOC, Energiebedarf)
- Ladeplan-Details
- Start/Stop-Ereignisse

## Fehlerbehebung

**Ladeplan wird nicht berechnet**:
- Prüfe ob `sensor.ostrom_energy_spotpreis` Preisdaten enthält
- Prüfe ob `sensor.energy_production_tomorrow_2` Werte liefert
- Prüfe die Home Assistant Logs für Fehlermeldungen

**Netzladung startet nicht**:
- Prüfe ob `input_text.akku_ladeplan_json` einen gültigen Plan enthält
- Prüfe ob der `huawei_solar` Service verfügbar ist
- Prüfe ob die Device-ID korrekt ist

**Zwischenladen funktioniert nicht**:
- Prüfe ob der aktuelle SOC unter `min_soc_sicher` (20%) liegt
- Prüfe ob die günstigste Zeit mehr als `sicherheits_offset` (4h) entfernt ist