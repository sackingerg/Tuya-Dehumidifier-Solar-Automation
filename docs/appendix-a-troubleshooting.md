# Appendix A — Troubleshooting

Values are written as their variable name followed by the default in brackets, for example `trailer_low (20 %)`. Every variable is described in [variables.md](variables.md).

## Where to look first

| Place | What it tells you |
|---|---|
| **Lockouts card** (dashboard, under the Mode selector) | Every active lockout, colour-coded. Shows a green **No Lockouts Active** when there are none |
| **Last** line in the status card (`input_text.trailer_dehum_status`) | The latest decision, with its time |
| **Dehumidifier Activity Log** (`sensor.trailer_dehumidifier_activity_log`) | The newest `log_length (40)` events: mode and power-source changes, every status line, AC port run times, drying runs, tank and temperature stops |

Switching between Grid and EcoFlow mode isn't a lockout. Set the **Grid Mode** switch to match where the dehumidifier is plugged in.

## Lockouts

| # | Lockout | What triggers it | Lockouts card | How it's removed |
|---|---|---|---|---|
| 1 | **Mode Off** | Mode set to Off by hand | Blue: *Mode is Off.* | Set Mode to Auto or Manual On |
| 2 | **Low trailer battery** | Trailer battery below `trailer_low (20 %)` (the **Trailer bat low — cutoff** slider) for `low_battery_for (00:05:00)` with Grid mode off, or already below it when Grid mode is turned off. Mode is set to Off and the phone gets *Time to set the trailer "On Grid".* | Red: battery %, cutoff, and the Mode it returns to | Turn Grid mode on: Mode goes back to what it was. Choosing a Mode by hand also works |
| 3 | **Temperature lockout** | The dehumidifier's temperature is outside `min_temp_f (41 °F)` – `max_temp_f (95 °F)` for `out_of_range_for (00:02:00)`, in either mode | Red: *Trailer Temp out of range, (xx°F).* | The temperature is at least `range_margin_f (3 °F)` inside the range (44–92 °F). In Grid mode it has to stay there for `back_in_range_for (00:10:00)`, then the unit turns back on. In EcoFlow mode the next check's reading clears it |
| 4 | **Tank full** | The unit's float trips. With the drain hose fitted, that means the hose is blocked, kinked or dislodged. The phone gets an alert | Red: clear the drain hose and empty the tank | Clear the hose and empty the tank. The unit resumes; in EcoFlow mode the next check restarts it |
| 5 | **Unit stopped itself: too cold or too hot** | The dehumidifier's own protection (`fault_code` bits 8 and 16). The phone gets an alert | Red, with the temperature | The unit resumes on its own once the trailer is back in range |
| 6 | **Did not come online** | No humidity reading within `online_timeout (00:10:00)` after a check turns the AC port on | Amber | Clears at the unit's next real reading. If it repeats, check the unit's power, Wi-Fi and fixed IP address, and Tuya Local |
| 7 | **Humidity waiting for the low-rate window** | Mode Auto, humidity above the Start setting, on weekdays from `window_morning_hour (7)` to `window_evening_hour (21)`. In Grid mode the unit is off; in EcoFlow mode checks still read the air but don't start a run. With humidity at or below Start this is normal, not a lockout: the card shows green *Humidity within set parameters, system will check at 21:00.* | Amber: humidity, Start setting and the opening time | The window opens at `window_evening_hour (21)` on weekdays and all day on Saturday and Sunday. In EcoFlow mode a check starts as it opens. In Grid mode, Manual On runs the unit until the window next closes |

## Not lockouts, but it isn't running

| Symptom | Cause | What to do |
|---|---|---|
| EcoFlow mode, Auto: nothing happens for a while | A new check starts no sooner than `check_hours (3 h)` after the last one. The status card shows the next check time | Wait, or change Mode to start a check straight away |
| A check runs, then the AC port turns off | Humidity isn't above the Start setting (`start_humidity (45 %)`). The status line says *checked, not needed* | Nothing; it runs when the trailer is humid enough. Lower the Start setting to run sooner |
| Manual On in EcoFlow mode, but the unit stays off | Manual On only powers the outlet. The unit starts in whatever state it was last left in | Switch the unit on once, from the **Humidity Level** tile or its own panel. It then comes back on by itself each time |
| Readings blank or *unknown* | With the AC port off, the unit has no power, so its sensors are unavailable | Normal. **Last known humidity** keeps the last real reading and its time |
| Run stopped before reaching Stop | The low-rate window closed (#7), a lockout set (#3, #4) or Mode changed. The activity log shows which | See the matching lockout above |

## Logs and errors

- **Tuya Local "Unable to Connect" errors** in the Home Assistant log are normal while the AC port is off and the unit has no power.
- **"did not come online"** in the activity log is lockout #6.
