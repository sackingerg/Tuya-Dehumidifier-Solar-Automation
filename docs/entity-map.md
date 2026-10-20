# Entity map

The package and dashboard use the entity IDs below for devices that come from other integrations. Your IDs will almost certainly differ. Find and replace each one in **both** `packages/trailer_dehumidifier.yaml` and `dashboards/trailer_dehumidifier.yaml`.

To look up your own IDs, go to **Settings → Devices & services → Entities** and search for the device.

## Dehumidifier (Tuya Local integration)

The dehumidifier is connected through **Tuya Local** (HACS), which talks to it directly on your local network. With a device named "Trailer Dehumidifier", the Tuya Local IDs match without replacing.

| Entity ID used here | What it is | Comes from |
|---|---|---|
| `humidifier.trailer_dehumidifier` | On/off and target humidity (humidistat) | Tuya Local |
| `fan.trailer_dehumidifier` | Fan; its `percentage` is 50 for Low and 100 for High | Tuya Local |
| `sensor.trailer_dehumidifier_temperature` | Room temperature measured by the dehumidifier (°F) | Tuya Local |
| `binary_sensor.trailer_dehumidifier_tank` | Water tank full | Tuya Local |
| `sensor.trailer_dehumidifier_humidity` | Room humidity measured by the dehumidifier | Helper: `packages/optional/tuya_local_helpers.yaml` |
| `binary_sensor.trailer_dehumidifier_low_temperature` | Dehumidifier stopped itself: too cold | Helper: same file |
| `binary_sensor.trailer_dehumidifier_high_temperature` | Dehumidifier stopped itself: too hot | Helper: same file |

Tuya Local doesn't give this unit a separate humidity sensor or temperature-fault sensors. The helper file builds them from the humidifier's `current_humidity` and from the `fault_code` of `binary_sensor.trailer_dehumidifier_problem`.

## EcoFlow Delta 3 Plus (EcoFlow BLE custom integration)

| Entity ID used here | What it is |
|---|---|
| `switch.ecoflow_delta_3_plus_ac_ports` | AC output ports; the dehumidifier plugs in here |
| `sensor.ecoflow_delta_3_plus_battery_level` | EcoFlow battery % (shown on the dashboard only) |

Any power station or smart plug works, as long as it gives you a switch for the outlet the dehumidifier uses.

## Trailer battery (BLE Battery Management System custom integration)

| Entity ID used here | What it is |
|---|---|
| `sensor.trailer_bms_battery` | Trailer battery state of charge (%). Drives the low-battery switch-off and "On Grid" message |
| `sensor.trailer_bms_current`, `_power`, `_voltage`, `_temperature`, `_cycles`, `_runtime`, `_stored_energy`, `binary_sensor.trailer_bms_charging` | Dashboard only |

## Solar charger (Victron BLE integration), dashboard only

| Entity ID used here | What it is |
|---|---|
| `sensor.trailer_solar_charger_battery_current` | Charge current gauge |

## Notifications

| Entity ID used here | What it is |
|---|---|
| `notify.mobile_app_your_phone` | Where alerts go. Use the notify entity from your Home Assistant Companion app |

## Created by this package

You don't need to replace these. The package creates them:

- `input_select.trailer_dehum_mode`
- `input_boolean.trailer_dehum_*`
- `input_number.trailer_dehum_*`
- `input_datetime.trailer_dehum_last_check`
- `input_text.trailer_dehum_status`
- `sensor.trailer_dehum_defaults`
- `sensor.dehumidifier_fan_speed`
- `binary_sensor.trailer_dehum_grid_window`
- `sensor.trailer_last_known_humidity`
- `sensor.trailer_dehumidifier_activity_log`
- the `script.trailer_dehum_*` scripts and their automations

The three helper entities in `packages/optional/tuya_local_helpers.yaml` are listed in the Tuya Local table above.
