# The dehumidifier: Wesiop OL09-D031M

The automation runs a **Wesiop OL09-D031M** (full model OL09-D031MMN2B1/NFE), a 24-pint compressor dehumidifier with built-in Wi-Fi. Home Assistant controls it over the local network through **Tuya Local**. Any dehumidifier that Tuya Local supports with the same entities works; see [entity-map.md](entity-map.md).

## Specifications

| Item | Value |
|---|---|
| Type | Compressor (refrigerant) dehumidifier, auto defrost |
| Moisture removal | Up to 24 pints/day (rated) |
| Rated coverage | Up to 1,500 sq ft |
| Water tank | 0.44 gal, with a full-tank shutoff |
| Continuous drain | Gravity drain port; drain hose supplied |
| Power | 115 V AC, 295 W rated (about 2.6 A) |
| **Typical draw** | **About 100–120 W** on High at the EcoFlow AC output (median 105 W, peaks about 150 W) |
| Airflow | 135 CFM |
| Fan speeds | 2 (Low, High) |
| Noise | About 38 dB(A) (manufacturer figure) |
| Humidity setting | 30–80 % in 5 % steps |
| Operating ambient | **5–35 °C (41–95 °F)**, per the manual |
| Dimensions | 8.66 in D × 10 in W × 17.51 in H |
| Other features | Auto-restart memory after power loss, 1–24 h timer, child lock, ionizer, washable filter with a filter-clean reminder, sleep mode |
| Connectivity | 2.4 GHz Wi-Fi |

**Two details the automation relies on:**

- **Auto-restart memory.** After a power cut, the unit comes back in the state it was in. EcoFlow-mode Manual On depends on this, because it only switches the outlet.
- **The 5–35 °C operating range.** This is where `min_temp_f (41)` and `max_temp_f (95)` in the defaults block come from.

## What Home Assistant sees

Tuya Local controls the unit directly over the local network, using one of its built-in dehumidifier profiles ("Sygonix Smarter" matches this unit). It provides:

| Entity | Use |
|---|---|
| `humidifier.trailer_dehumidifier` | On/off, target humidity (30–80 % in 5 % steps) and modes (continuous, auto, sleep). Its `current_humidity` attribute is the room humidity |
| `fan.trailer_dehumidifier` | Fan speed: 50 % = Low, 100 % = High |
| `sensor.trailer_dehumidifier_temperature` | Room temperature from the unit's own sensor |
| `binary_sensor.trailer_dehumidifier_tank` | Tank full, or the drain is backing up |
| `binary_sensor.trailer_dehumidifier_problem` | Any fault. Its `fault_code` attribute holds the fault bits |
| `binary_sensor.trailer_dehumidifier_filter`, `sensor.trailer_dehumidifier_filter_life`, `button.trailer_dehumidifier_filter_reset` | Filter reminder |
| `select.trailer_dehumidifier_timer`, `lock.trailer_dehumidifier_child_lock`, `switch.trailer_dehumidifier_ionizer`, `select.trailer_dehumidifier_temperature_unit`, `button.trailer_dehumidifier_runtime_reset` | Unit settings |

`packages/optional/tuya_local_helpers.yaml` adds the three entities the automation needs that Tuya Local doesn't provide:

| Helper | Built from |
|---|---|
| `sensor.trailer_dehumidifier_humidity` | The humidifier's `current_humidity` |
| `binary_sensor.trailer_dehumidifier_low_temperature` | `fault_code` bit 8 |
| `binary_sensor.trailer_dehumidifier_high_temperature` | `fault_code` bit 16 |

The fault bits follow the Tuya Local profile. If your unit uses a different profile, check them.

All of these are unavailable (or unknown) whenever the unit has no power, for example while the EcoFlow AC port is off.

## Installation in the trailer

### Placement

- The unit sits on an **elevated table** in the trailer. Two reasons:
  - The drain works by gravity, so the unit's drain port has to be higher than the drain it empties into.
  - The intake and exhaust are kept clear of the floor and of anything stored there.
- Leave a few inches of space around the intake and exhaust grilles so air can circulate.
- Keep the unit level, so water runs toward the drain port and the tank float reads correctly.

### Drain hookup

- The **drain hose supplied by Wesiop** connects to the unit's continuous-drain port.
- The hose runs into the **bathtub drain**. From there the water goes to the trailer's gray-water tank, so the unit never needs emptying.
- The hose has to run **downhill the whole way**, with no loops, sags, kinks or rises. Any of these traps water and stops the flow.
- Secure the hose end in the tub drain so it can't pop out, but leave it open to the air. A drain sealed airtight around the hose can stop it flowing.
- **A "tank full" alert means a drain problem:** a kinked, blocked or dislodged hose, not a full tank. Clear the hose and empty the tank, and the unit resumes.
- **Cold weather:** water standing in the hose or the tub trap can freeze. The automation won't run the unit below 41 °F, but drain the hose before the trailer sits in freezing weather.

### Power hookup

The **Grid Mode** switch on the dashboard tells the automation where the unit is plugged in. Set it to match the physical plug.

| Grid Mode | Plugged into | Behaviour |
|---|---|---|
| **Off: EcoFlow mode** | The EcoFlow Delta 3 Plus AC outlet. The trailer's solar-charged battery keeps the EcoFlow charged | The automation switches the EcoFlow AC port on and off. The unit samples the air every few hours, and runs only when needed and only inside the low-rate window (weekday nights, all weekend). At 20 % trailer battery, Mode is set to Off and your phone is told to set the trailer On Grid |
| **On: Grid mode** | House current (shore power) | The unit is always powered. It runs on its own humidistat during the low-rate window (weekday nights, all weekend) |

At about 100–120 W, an hour of running uses roughly 0.1–0.12 kWh from the EcoFlow, plus inverter losses.

### Network and Home Assistant

1. **Put the unit on 2.4 GHz Wi-Fi** and give it a **fixed IP address** with a DHCP reservation in your router. Tuya Local connects to it by address.
2. **Install Tuya Local** from HACS. Then go to **Settings → Devices & services → Add integration → Tuya Local** and enter the unit's IP address, device ID and local key. Accept the dehumidifier profile it suggests.
3. **Name the device "Trailer Dehumidifier"** so the entity IDs match this repository. Then install `packages/optional/tuya_local_helpers.yaml`. If you use other names, follow [entity-map.md](entity-map.md).

Each time the unit powers up, it has to boot and rejoin Wi-Fi before Tuya Local can reach it. That's why the EcoFlow check waits until a reading arrives, for up to `online_timeout (00:10:00)`, instead of using a fixed delay. While the unit is unpowered, Tuya Local logs "Unable to Connect" errors; that's expected.
