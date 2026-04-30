# Automatizare centrală termică – Home Assistant
## 4 termostate Zigbee + contact uscat

---

## Ce conține automatizarea

| Componentă | Descriere |
|---|---|
| `binary_sensor.cere_caldura_vreun_termostat` | Template sensor: ON dacă cel puțin 1 termostat solicită căldură |
| Automatizare 1 – Pornire | Contact uscat ON când se cere căldură |
| Automatizare 2 – Oprire | Contact uscat OFF când toate camerele sunt satisfăcute |
| Automatizare 3 – Timeout | Oprire forțată după 3 ore (protecție) |
| Automatizare 4 – Alertă offline | Notificare dacă un termostat pierde conexiunea Zigbee |

---

## Pași de instalare

### 1. Adaptează entity_id-urile

Înlocuiește în fișierul YAML:

| Placeholder din YAML | Entity-ul tău real |
|---|---|
| `climate.termostat_camera_1` | Entity-ul primului termostat din HA |
| `climate.termostat_camera_2` | Entity-ul celui de-al doilea |
| `climate.termostat_camera_3` | Entity-ul celui de-al treilea |
| `climate.termostat_camera_4` | Entity-ul celui de-al patrulea |
| `switch.relay_contact_uscat_centrala` | Switch-ul relay-ului tău (Zigbee/WiFi) |

Ca să găsești entity_id-urile: **HA → Developer Tools → States** și caută după nume.

### 2. Copiază în Home Assistant

**Varianta A – fișier separat (recomandat):**
```
config/
  packages/
    centrala.yaml   ← pune conținutul YAML aici
```

În `configuration.yaml` adaugă (dacă nu e deja):
```yaml
homeassistant:
  packages: !include_dir_named packages
```

**Varianta B – direct în configuration.yaml:**
Copiază secțiunile `template:` și `automation:` direct în fișier.

### 3. Verifică și reîncarcă

```
HA → Developer Tools → Check Configuration → Restart
```

---

## Conexiune fizică – Contact uscat

```
Centrală termică
  Terminal TA (thermostat extern)  ──┐
                                     ├── Relay contact uscat (NO = Normally Open)
  Terminal TA (thermostat extern)  ──┘

Relay alimentat din HA (Zigbee sau WiFi switch cu output uscat)
```

> **Important:** Centralele termice moderne au terminalele TA (room thermostat) care
> acceptă contact uscat 24V sau potențial-free. **Nu conecta tensiune de rețea (230V)**
> la terminalele TA! Verifică manualul centralei tale.

---

## Termostate Zigbee compatibile

Automatizarea funcționează cu orice termostat care expune atributul `hvac_action`:
- Sonoff TRVZB
- TuYa TS0601 TRV
- Moes BHT-002-GCLZB  
- Eurotronic Spirit Zigbee
- Danfoss Ally

Dacă termostatul tău nu expune `hvac_action`, modifică template sensor-ul
să compare `current_temperature` cu `temperature` (setpoint):

```yaml
state: >
  {% set t1 = state_attr('climate.termostat_camera_1','current_temperature') | float(0)
              < state_attr('climate.termostat_camera_1','temperature') | float(0) - 0.3 %}
  ...
  {{ t1 or t2 or t3 or t4 }}
```

---

## Ajustări recomandate

- **`delay_on: 30s`** – evită porniri la fluctuații scurte de temperatură
- **`delay_off: 60s`** – lasă centrala să mai ruleze 1 minut după ce se satisface cererea
- **Timeout 3 ore** – ajustează la `hours: 4` sau `hours: 5` dacă încălzești spații mari
