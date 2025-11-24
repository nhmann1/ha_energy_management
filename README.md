
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