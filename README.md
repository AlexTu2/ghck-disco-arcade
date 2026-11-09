# GHCK Disco Arcade

A big arcade button for the **Good Hood Community Kitchen**. Press it and the room turns into a disco:
the IR-controlled party light switches on, a short random song plays through the ceiling speakers,
and the button glows. When the song ends, everything turns back off.

Built Oct 2025 – May 2026. Installed Feb 2026. Stable since the input filter fix on May 8, 2026.

## How it works

```
[Arcade button] ─GPIO22─► ESP32-P4-ETH (ESPHome "GHCK-ESP")
                               │  toggles input_boolean.automation_active
                               ▼
                    Home Assistant (DietPi on Orange Pi 3B)
                     ├─ "Arcade On"  → HS200 switch off, IR Power ─► ESP GPIO20 ─► IR LED ─► disco light
                     ├─ "Play Random Song & Auto-Stop" → MPD plays 1 of 3 clips (20–29 s) → AUX → speakers
                     └─ boolean state ─► ESP GPIO21 ─► button LED glow
```

- The ESP publishes the button as a **template switch** ("Automation State") that drives the HA helper.
  It also listens to that helper, so the button LED and switch stay in sync no matter who toggles it.
- A **3-second cooldown** (in firmware) ignores repeat presses while the IR and music catch up.
- The ESP exposes an **"MPD Volume"** slider. The automation reads it before playing.
- All 15 IR codes from the light's remote are exposed as HA buttons (Power, Stop Move, Mod 1–3, Flash, A1–A7, +/−).

## Repo layout

| Path | What |
|---|---|
| [esphome/ghck-esp.yaml](esphome/ghck-esp.yaml) | Full ESPHome firmware config. Needs `secrets.yaml`. |
| [esphome/secrets.yaml.example](esphome/secrets.yaml.example) | Template for the API encryption key |
| [homeassistant/automations.yaml](homeassistant/automations.yaml) | Arcade On + Play Random Song automations |
| [docs/wiring.md](docs/wiring.md) | Pinout, button filter circuit, IR LED, audio |
| [docs/history.md](docs/history.md) | Build log and every bug found along the way |
| [docs/bill-of-materials.md](docs/bill-of-materials.md) | Parts list |

## Flashing

```bash
cd esphome
cp secrets.yaml.example secrets.yaml   # paste the real key (kept outside this repo)
esphome run ghck-esp.yaml --device <ESP IP>
```

Use ESPHome ≥ 2025.11. To force a full rebuild, delete `.esphome/build/esp32-p4-eth` (there is no `--clean` flag).

## Songs

Stored in HA's local media folder. Each is trimmed to about 20–30 s with a fade-out:

- *Drink Makgeolli (막걸리 한잔)*, 20 s
- *Shabang Shabang (샤방샤방)*, 27 s
- *So Hot! (앗! 뜨거)*, 29 s

The source audio and Audacity projects are on the NAS, not in this repo.

## Open items

- [ ] Export "Arcade Off" automation YAML from the kitchen's HA and add it here
- [ ] Add the `ir_active` guard (block presses while IR is sending). It was drafted but never confirmed in the final firmware.
- [ ] Move the breadboard build onto a soldered perfboard
