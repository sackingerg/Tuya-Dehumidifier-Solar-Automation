# Variables and settings

This page describes every value the package keeps or uses: what it is, its default, and what reads it. Values are written as their name followed by the default in brackets, for example `settle_time (00:03:00)`.

## 1. Defaults block

`sensor.trailer_dehum_defaults` is the first block of `packages/trailer_dehumidifier.yaml`. Its state is the package version, and its attributes hold every starting value and tunable constant. Templates read them with `state_attr('sensor.trailer_dehum_defaults', '<name>')`. Fields that can't use templates read the YAML anchors shown in the last column.

After editing this block, go to **Developer tools → YAML** and reload **Template entities**, **Scripts** and **Automations**.

### Starting values for the dashboard settings

**Reset Settings to Defaults** (`script.trailer_dehum_reset_defaults`) copies these four values into the dashboard sliders.

| Name | Default | Description | Used by |
|---|---|---|---|
| `start_humidity` | 45 % | EcoFlow-mode Auto keeps a run going only when humidity is above this. Never below `stop_humidity` | Reset script |
| `stop_humidity` | 40 % | A run ends when humidity drops below this. It is also the humidistat target sent to the unit | Reset script, apply and check scripts, dashboard |
| `check_hours` | 3 h | Time between EcoFlow-mode samples | Reset script, scheduled check, dashboard |
| `trailer_low` | 20 % | Trailer battery level that, with Grid mode off, sets Mode to Off and sends *Time to set the trailer "On Grid".* Also applies when Grid mode is turned off with the battery already below it | Reset script |

### Operating range

| Name | Default | Description | Anchor |
|---|---|---|---|
| `min_temp_f` | 41 °F | Lowest ambient temperature the unit is run at (manual: 5 °C) | `*manual_min_f` |
| `max_temp_f` | 95 °F | Highest ambient temperature the unit is run at (manual: 35 °C) | `*manual_max_f` |
| `range_margin_f` | 3 °F | How far back inside the range the temperature must be before the temperature lockout clears (both modes) | — |
| `out_of_range_for` | 00:02:00 | How long the dehumidifier's temperature must stay outside the range before the temperature lockout sets | `*out_of_range_for` |
| `back_in_range_for` | 00:10:00 | In Grid mode, how long the temperature must stay `range_margin_f` inside the range before the lockout clears and the unit turns back on. In EcoFlow mode the next check's reading clears it | `*back_in_range_for` |

### Low-rate window

| Name | Default | Description |
|---|---|---|
| `window_evening_hour` | 21 | Hour the weekday low-rate window opens (24-hour clock) |
| `window_morning_hour` | 7 | Hour the low-rate window closes. Saturday and Sunday are inside the window all day |

Auto runs the dehumidifier only inside this window, in both modes. In EcoFlow mode, checks run at any time, but outside the window they only record the reading. When the window closes, a run in progress ends; when it opens, a check starts right away. Off and Manual On ignore it in EcoFlow mode.

### EcoFlow sampling and battery

| Name | Default | Description | Anchor |
|---|---|---|---|
| `online_timeout` | 00:10:00 | Longest wait, after the AC port turns on, for the dehumidifier to report a humidity reading. If it doesn't, the AC port turns off and the check is logged as "did not come online" | `*online_timeout` |
| `settle_time` | 00:03:00 | Time the unit runs before its reading is used, so the fan draws trailer air across the sensor | `*settle_time` |
| `low_battery_for` | 00:05:00 | How long the trailer battery must stay below `trailer_low` before Mode is set to Off and the message is sent | `*low_battery_for` |

### Tracking and log

| Name | Default | Description |
|---|---|---|
| `run_min_minutes` | 10 | A run shorter than this gets no drying rate and no last-run summary |
| `log_length` | 40 | Activity-log entries kept |

### Fixed values (not in the block)

| Value | Where | Why fixed |
|---|---|---|
| Every 15 minutes | Scheduled-check trigger (`time_pattern`) | A time pattern can't read a template |
| Fan percentage 100 = High | Fan speed sensor | Set by the device |

## 2. Dashboard settings and controls

| Entity | Range | Description |
|---|---|---|
| `input_select.trailer_dehum_mode` | Off / Auto / Manual On | **Off:** never runs. **Auto:** the rules decide. **Manual On:** on now, with no automatic stopping (alerts still sent). In EcoFlow mode, Manual On only switches the AC port on |
| `input_boolean.trailer_dehum_grid_mode` | on / off | Where the unit is plugged in. **Off:** EcoFlow AC port. **On:** house current |
| `input_number.trailer_dehum_start_humidity` | 40–80 %, step 1 | Start setting; starts from `start_humidity` |
| `input_number.trailer_dehum_stop_humidity` | 30–75 %, step 1 | Stop setting and humidistat target; starts from `stop_humidity`. Start and Stop are tied: moving one past the other drags it along |
| `input_number.trailer_dehum_check_hours` | 1–12 h, step 1 | Sampling interval; starts from `check_hours` |
| `input_number.trailer_dehum_trailer_low` | 10–60 %, step 1 | Battery cutoff; starts from `trailer_low` |

## 3. Internal state (set by the automation)

| Entity | Description |
|---|---|
| `input_boolean.trailer_dehum_running` | On while an EcoFlow-mode Auto run is in progress |
| `input_boolean.trailer_dehum_temp_out_of_range` | The temperature lockout. On while the dehumidifier's temperature is out of range (see `out_of_range_for`, `range_margin_f`, `back_in_range_for`). Blocks runs in both modes; shown in red on the Lockouts card |
| `input_text.trailer_dehum_mode_before_low_battery` | The Mode in use when a low trailer battery set it to Off (Auto or Manual On). Turning Grid mode on puts the Mode back and clears this; choosing Auto or Manual On by hand also clears it. Empty otherwise |
| `input_datetime.trailer_dehum_last_check` | Start time of the latest EcoFlow-mode check. The next check is due `check_hours` later |
| `input_text.trailer_dehum_status` | The latest automation decision in one line, prefixed with the time (for example `10/19 21:30 — Grid: on (Auto), humidistat 50%`) |

## 4. Template sensors and their attributes

### `binary_sensor.trailer_dehum_offline`
On when a check logs "did not come online". Off at the dehumidifier's next real humidity reading. Shown in amber on the Lockouts card.

### `binary_sensor.trailer_dehum_grid_window`
The low-rate window. On Monday to Friday from `window_evening_hour` to `window_morning_hour`, and all day Saturday and Sunday. Auto in both modes runs the dehumidifier only while it's on.

### `sensor.dehumidifier_fan_speed`
**Off**, **Low** or **High**, from `fan.trailer_dehumidifier`'s state and percentage. Unavailable while the unit has no power.

### `sensor.trailer_last_known_humidity`
The latest real humidity reading. It keeps its value while the unit is unpowered.

| Attribute | Description |
|---|---|
| `measured_at` | When the reading was taken (ISO time) |
| `temperature` | Room temperature at that moment (°F) |

### `sensor.trailer_dehumidifier_drying_rate`
**State:** the current run's drying rate in %/h once the run is `run_min_minutes` old. Between runs, it shows the last run's rate. Negative means humidity rose.

| Attribute | Description |
|---|---|
| `running` | true while the unit is on and reporting humidity |
| `start_ts` | Run start, as epoch seconds |
| `start_humidity` | Humidity when the run started (%) |
| `minutes` | Run length so far |
| `drop` | Humidity points removed so far |
| `last_humidity` | Latest humidity reading (%) |
| `last_run` | Summary of the last finished run of `run_min_minutes` or more: `started` (m/d HH:MM), `minutes`, `start_humidity`, `end_humidity`, `drop`, `rate` (%/h) |

The calculation is in `custom_templates/trailer_dehum_run.jinja`, macro `trailer_dehum_run(prev, now_ts, min_minutes)`:

| Parameter | Description |
|---|---|
| `prev` | The sensor's previous attributes (`this.attributes`) |
| `now_ts` | Time of this update, as epoch seconds |
| `min_minutes` | `run_min_minutes`. Shorter runs get no rate and no summary |

It returns the attributes above as JSON.

### `sensor.trailer_dehumidifier_activity_log`
**State:** time of the latest entry. **Attribute `entries`:** the newest `log_length` events, newest first. It records:

- mode and power-source changes
- every status line
- EcoFlow AC port run times
- finished drying runs (each logged once)
- tank-full, too-cold and too-hot stops
- low-battery switch-offs (as status lines)

## 5. Scripts

| Script | Description |
|---|---|
| `script.trailer_dehum_apply` | The single policy. Sets the AC port and the dehumidifier from the current mode, power source and low-rate window. Field `reason`: `mode`, `grid`, `window` or `temperature` (what triggered it) |
| `script.trailer_dehum_check` | One EcoFlow-mode sample: AC port on, wait up to `online_timeout`, unit on at the Stop setting, wait `settle_time`, then keep running (only inside the low-rate window) or turn off |
| `script.trailer_dehum_reset_defaults` | Copies `start_humidity`, `stop_humidity`, `check_hours` and `trailer_low` into the dashboard settings |

## 6. Outside entities

Entities from other integrations (dehumidifier, EcoFlow, trailer battery, phone) are listed in [entity-map.md](entity-map.md).
