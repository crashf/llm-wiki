# Anker SOLIX F3000 MQTT Protocol

The Anker SOLIX F3000 (A1782) exposes device telemetry and control via MQTT over AWS IoT Core. The Python `anker-solix-api` library handles authentication, certificate-based TLS, and field decoding.

## Key Facts

- **Device:** Anker SOLIX F3000 (A1782)
- **Protocol:** MQTT over AWS IoT Core, certificate-based TLS
- **Library:** `anker-solix-api` (forked, patched)
- **Polling:** Every 60s via systemd timer
- **Database:** PostgreSQL, table `f3000_metrics`
- **Dashboard:** Grafana at http://10.255.61.20:3000/d/f3000-all-fields/

## MQTT Field Mapping

Fields are encoded in binary MQTT payloads with 2-byte hex keys mapped in `mqttmap.py`. The library uses a `NAME` → key mapping with per-field decoders for booleans, integers, strings, bitmasks, and power readings.

**Critical bug discovered 2026-04-26:** Field names in `mqttmap.py` had `?` suffixes as developer uncertainty markers (e.g. `ac_input_power?`). `apibase.py`'s field filter lists used exact names without `?`, causing those fields to be silently discarded during decode.

**Fix:** Strip `?` suffixes from `mqttmap.py` field names. Normalize field names in `apibase.py` filter checks.

## Confirmed Live Fields (39 total)

| Field | Type | Notes |
|-------|------|-------|
| `temperature` | decimal | Device temperature |
| `battery_soc` | decimal | Battery state of charge |
| `ac_input_power` | integer | AC grid input power (W) |
| `ac_output_power` | integer | AC outlet output power (W) |
| `output_power_total` | integer | Total output power (W) |
| `ac_input_limit` / `max` | integer | AC charge limit (W) |
| `ac_output_power_switch` | boolean | AC outlets on/off |
| `ac_output_mode` | integer | 0=Normal, 1=Smart |
| `ac_fast_charge_switch` | boolean | Fast charge on/off |
| `ac_output_timeout_seconds` | integer | AC auto-shutoff timer |
| `dc_output_power` | integer | DC output power (W) |
| `dc_output_power_switch` | boolean | DC outlets on/off |
| `dc_12v_output_mode` | integer | 0=Normal, 1=Smart |
| `dc_output_timeout_seconds` | integer | DC auto-shutoff timer |
| `usbc_1-4_power` | integer | USB-C port power (W) |
| `usbc_1-4_status` | integer | Port status code |
| `usba_1-2_power` | integer | USB-A port power (W) |
| `usba_1-2_status` | integer | Port status code |
| `display_mode` | integer | 0=Off, 1=Dim, 2=Medium, 3=Bright, 4=Max |
| `display_timeout_seconds` | integer | Display auto-off timer |
| `display_switch` | boolean | Display on/off |
| `port_memory_switch` | boolean | Port memory on/off |
| `device_timeout_min` | integer | Device auto-shutoff (min) |
| `expansion_packs` | integer | Number of BP3000 connected |
| `firmware_version` | string | e.g. `v1.2.2.5` |
| `device_sn` / `pn` | string | Serial number and part number |
| `msg_timestamp` | timestamp | Last MQTT update time |

## Dashboard

Grafana dashboard "Anker SOLIX F3000 - All Fields" has 36 panels:
- **Row 1:** Temp, SOC, AC Input Power, AC Output Power, Output Total, AC Out Switch
- **Row 2:** AC Charge Limit, Max SOC, DC Output, DC 12V Mode, AC Fast Charge, DC Out Switch
- **Row 3-4:** USB port power and status (8 panels)
- **Row 5:** Device settings (Display Mode, Timeout, Switch, Port Memory, Device Timeout, Expansion Packs)
- **Row 6-7:** Timeseries history (AC in/out, SOC, Output Total, USB power, Temperature)

## Files

- `projects/anker-f3000/anker-solix-api/api/mqttmap.py` — field definitions
- `projects/anker-f3000/anker-solix-api/api/apibase.py` — base API with MQTT decode
- `projects/anker-f3000/anker-solix-api/f3000_to_postgres.py` — poller script
- `projects/anker-f3000/grafana/f3000-all-fields-dashboard.json` — exported dashboard
