# Tuya Dehumidifier: Home Assistant trailer automation

A Home Assistant package that runs a Wesiop OL09-D031M Wi-Fi dehumidifier in a travel trailer, controlled entirely over the local network with Tuya Local. It can run from a solar-charged battery bank through an EcoFlow power station, or from house current. It decides when to run, protects the battery, keeps the unit inside its rated temperature range, tracks how fast it dries the air, and sends phone alerts when something needs attention.

## Features

- **Three modes:** Off, Auto and Manual On, plus a **Grid Mode** switch for the power source:
  - **EcoFlow mode:** the dehumidifier is plugged into the EcoFlow AC port and runs on the battery.
  - **Grid mode:** it's plugged into house current.
- **Battery-friendly sampling** in EcoFlow mode. The dehumidifier's own sensor is the only humidity reading, so the AC port is switched on every few hours. The automation waits for the unit to come online, lets it settle, then keeps it running or turns it off.
- **Low-rate window:** Auto runs the dehumidifier only on weekday nights (21:00–7:00) and all weekend, in both Grid and EcoFlow mode. Outside it, EcoFlow-mode checks still read the humidity but don't start a run; when the window closes, everything turns off.
- **Low trailer battery:** at 20 %, Mode is set to Off and your phone gets *Time to set the trailer "On Grid".* Turning Grid mode on puts Mode back. Nothing is locked.
- **Temperature lockout:** the unit is kept within the manual's 5–35 °C (41–95 °F) in both modes, with hysteresis so it doesn't flicker on and off. The dashboard shows a red *Trailer Temp out of range* warning while it's set.
- **Lockouts card:** the dashboard lists every active lockout, or *No Lockouts Active*.
- **Start humidity is never below Stop humidity:** the two sliders are tied together.
- **Drying progress** (%/h, time to target), **last known humidity** with a timestamp, and a 40-entry **activity log** that survives restarts.
- **Phone alerts:**
  - tank full (a blocked drain hose)
  - too cold or too hot
  - trailer battery low: *Time to set the trailer "On Grid".*
- **All defaults in one place:** every default and tunable constant lives in the first block of the package.

How it all fits together, including a flow diagram: [docs/how-it-works.md](docs/how-it-works.md).
The dehumidifier itself (specifications, power draw, placement, drain and power hookup): [docs/dehumidifier.md](docs/dehumidifier.md).
Every variable, setting and attribute: [docs/variables.md](docs/variables.md).
Lockouts and what to do when it isn't running: [Appendix A — Troubleshooting](docs/appendix-a-troubleshooting.md).

## Repository layout

```
packages/
  trailer_dehumidifier.yaml          the automation (package)
  optional/
    tuya_local_helpers.yaml          humidity and too-cold/too-hot sensors for Tuya Local
custom_templates/
  trailer_dehum_run.jinja            drying-run tracking macro
dashboards/
  trailer_dehumidifier.yaml          Trailer / Solar dashboard view
docs/
  how-it-works.md                    behaviour, flow and settings
  dehumidifier.md                    Wesiop OL09-D031M specs and hookup
  variables.md                       every variable, setting and attribute
  appendix-a-troubleshooting.md      Appendix A: lockouts and troubleshooting
  entity-map.md                      every external entity to replace with yours
```

## Requirements

| Need | Used for |
|---|---|
| Home Assistant 2026.10 or later | Template `default_entity_id`, `triggers:`/`actions:` syntax |
| A Wesiop OL09-D031M (or a similar dehumidifier) in the **Tuya Local** integration (HACS), plus the helper file `packages/optional/tuya_local_helpers.yaml` | Humidity, temperature, humidistat, fan, tank and temperature-fault sensors |
| A switchable outlet for the dehumidifier. Here, an EcoFlow Delta 3 Plus via the **EcoFlow BLE** custom integration (HACS) | Powering the unit on and off in EcoFlow mode |
| A battery state-of-charge and temperature sensor for the trailer battery. Here, a JBD-style BMS via the **BLE Battery Management System** custom integration (HACS) | Low-battery switch-off |
| The **Home Assistant Companion app** on your phone | Alerts |
| **Bubble Card** (HACS → Frontend) | The tank-status card on the dashboard |
| Optional: a **Victron BLE** solar charger | The charge-current gauge on the dashboard |

## Installation

1. **Enable packages** in `configuration.yaml`, if you haven't already:
   ```yaml
   homeassistant:
     packages: !include_dir_named packages
   ```
2. **Copy the files** into your Home Assistant config folder:
   - `packages/trailer_dehumidifier.yaml` → `/config/packages/`
   - `custom_templates/trailer_dehum_run.jinja` → `/config/custom_templates/`
   - `packages/optional/tuya_local_helpers.yaml` → `/config/packages/` (see [docs/dehumidifier.md](docs/dehumidifier.md))
3. **Map your entities.** Replace the external entity IDs with your own, in both the package and the dashboard. The full list is in [docs/entity-map.md](docs/entity-map.md). At minimum change `notify.mobile_app_your_phone`.
4. **Check and restart.** Go to **Developer tools → YAML → Check configuration**, then restart Home Assistant. The first install adds new helpers, so a reload isn't enough.
5. **Set the starting values.** Run **Trailer Dehumidifier — Reset Settings to Defaults** (`script.trailer_dehum_reset_defaults`) once. It sets the sliders to the values in the defaults block.
6. **Add the dashboard.** Either:
   - add `dashboards/trailer_dehumidifier.yaml` as a YAML-mode dashboard, or
   - open an existing dashboard → **Edit** → **⋮** → **Raw configuration editor** and paste in the view under `views:`.
7. **Start it.** Set **Grid Mode** to match where the dehumidifier is plugged in, then set Mode to **Auto**.

## Defaults

Everything below is in the first block of `packages/trailer_dehumidifier.yaml` (`sensor.trailer_dehum_defaults`). The first four are starting values for the dashboard sliders. The rest are fixed constants.

| Name | Default | Meaning |
|---|---|---|
| `start_humidity` | 45 % | Auto (EcoFlow) starts a run above this |
| `stop_humidity` | 40 % | Runs end below this. Also the humidistat target |
| `check_hours` | 3 h | EcoFlow-mode sampling interval |
| `trailer_low` | 20 % | Trailer battery level that sets Mode to Off and sends the "On Grid" message |
| `min_temp_f` / `max_temp_f` | 41 / 95 °F | Operating range from the dehumidifier manual (5–35 °C) |
| `window_evening_hour` / `window_morning_hour` | 21 / 7 | Weekday low-rate window |
| `range_margin_f` | 3 °F | How far back inside the range before it turns on again |
| `out_of_range_for` / `back_in_range_for` | 00:02:00 / 00:10:00 | Hold times for the temperature lockout |
| `online_timeout` | 00:10:00 | How long to wait for the dehumidifier to report after power-up |
| `settle_time` | 00:03:00 | Fan time before a reading counts |
| `low_battery_for` | 00:05:00 | How long the trailer battery must stay below `trailer_low` before Mode is set to Off |
| `run_min_minutes` | 10 | Shorter runs get no rate and no summary |
| `log_length` | 40 | Activity-log entries kept |

After you edit the defaults block, go to **Developer tools → YAML** and reload **Template entities**, **Scripts** and **Automations**. Full descriptions of every variable: [docs/variables.md](docs/variables.md).

## Hardware notes

- **Use a dehumidifier that remembers its power state.** In EcoFlow mode with Manual On, only the outlet is switched. The unit has to come back on by itself.
- **A drain hose avoids a full tank.** The unit sits on an elevated table, and the supplied Wesiop hose runs downhill into the bathtub drain, which empties into the gray-water tank. A "tank full" alert means the hose is blocked, kinked or dislodged.
- **Typical draw** is about 100–120 W on High (rated 295 W). Details: [docs/dehumidifier.md](docs/dehumidifier.md).
- **Expect a short delay after power-up.** The unit has to boot and rejoin Wi-Fi before Tuya Local can reach it. That's why the automation waits for a reading instead of a fixed delay.

## License

MIT. See [LICENSE](LICENSE).
