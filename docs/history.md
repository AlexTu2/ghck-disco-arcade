# Build history

Rebuilt from the original Claude chats. The chat links are in the private notes, not in this repo.

## Oct 2025: start
The kitchen had looked for about 2 years for someone to build this. Parts ordered: Orange Pi 3B, IR receiver kit,
SSD, PSU, ceiling speakers.

## Feb 18, 2026: first ESPHome build
- Goal: physical button → HA automation, with the button LED controllable from HA.
- First board: ESP32 dev board flashed with ESPHome Web (`esphome-web-f0b114`). The button was on GPIO4.
- Bugs: duplicate `on_click` keys made the YAML invalid. GPIO5 is a strapping pin, so it was avoided.
- Learned the arcade microswitch pinout: COM / NC / NO.

## Feb 22–27, 2026: install and noise
- Static/hum on the speakers. A ground loop isolator on the AUX line fixed it.
- Switched to a **Waveshare ESP32-P4-ETH** with wired Ethernet.

## Mar 30, 2026: phantom presses start
The button sometimes triggered by itself. First theory was microswitch bounce or spring weight.

## Apr 9, 2026: debugging and volume
- Button moved to GPIO22. Added press counter, uptime sensor and `ESP_LOGW` press/release logs to catch ghost presses.
- MPD volume: `media_player.volume_set` failed ("does not support"), and `mpc volume` reported `n/a`.
  Volume is now exposed as an ESPHome number slider that calls `shell_command.set_mpd_volume` in HA (the MPD mixer in `mpd.conf` was looked at too).

## Apr 25, 2026: it broke again
- Logs showed hundreds of presses (`PRESS #521`) with nobody touching it.
- Tried `delayed_on`/`delayed_off` filters. With 100 ms on both, the button stopped responding at all.
  Settled on `delayed_on: 50ms` plus the 3 s cooldown.
- HA's ESPHome integration wasn't loading. Repaired the HA venv on DietPi
  (`python -m ensurepip`, then reinstalled `numpy hassil`). Note: the DietPi shell is fish, so `source activate` fails. Call the venv's python directly.
- Placing `on_connect` under `api: encryption:` is invalid YAML.

## May 8, 2026: root cause and fix ✅
- **Cause:** the 38 kHz IR signal on GPIO20 coupled into the button line on GPIO22. The weak internal pull-up let
  that noise read as a press.
- **Fix:** an external 4.7 kΩ pull-up to 3V3 plus a 100 nF capacitor to GND on GPIO22, at the ESP end (see [wiring.md](wiring.md)).
- After a rewire the LED and IR went dead. Fix: wire grounds and 3V3 straight to the header, not through the breadboard rails.
- Flashing an older YAML failed: the PlatformIO RISC-V toolchain's `as.exe` was missing on the Windows build machine.
- **Working.** Last issue: fast presses desynced the IR, LED and music. Cause: the HA automations used `mode: single`
  with a 5 s delay, so new triggers were dropped. Changed to `mode: restart` with a 500 ms delay.
