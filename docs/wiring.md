# Wiring

Board: **Waveshare ESP32-P4-ETH** (ESPHome board `esp32-p4-evboard`, wired Ethernet, IP101 PHY).

| Function | GPIO | Notes |
|---|---|---|
| IR LED (transmitter) | GPIO20 | `inverted: true`, 38 kHz carrier, 33% duty |
| Arcade button LED | GPIO21 | LEDC PWM → exposed to HA as light "Button Glow" |
| Arcade button input | GPIO22 | `INPUT_PULLUP`, `inverted: true` (pressed = LOW) |
| Ethernet | GPIO31/52/51/50 | MDC / MDIO / power / clock in. Don't reuse. |

Earlier builds used GPIO4, then GPIO18 for the button. GPIO5 gave a "strapping pin" warning, so avoid it.

## Button input (final, working since May 8, 2026)

```
3V3 ──[4.7kΩ]──┬──── GPIO22
               │
            [100nF]          ← ceramic cap, either way round (not polarized)
               │
GND ───────────┴──── button COM

button NO ──── GPIO22 junction
```

- Arcade microswitch terminals: **COM (1)**, **NC (2)**, **NO (3)**. Use COM + NO.
- Put the resistor and capacitor **at the ESP end** of the button cable.
- Why: the internal pull-up is weak (~45 kΩ). The 38 kHz IR signal on GPIO20 coupled into the
  GPIO22 line and caused phantom presses. The 4.7 kΩ resistor holds the line high more firmly, and the
  100 nF capacitor shunts the high-frequency noise to ground.

## IR LED

```
GPIO20 ──[100Ω]── IR LED anode (+)   IR LED cathode (−) ── GND
```

## Button LED

GPIO21 drives the arcade button's built-in LED. If the LED needs more current or a higher voltage than the
pin can supply, switch it through an NPN transistor (e.g. 2N2222): GPIO21 → 1 kΩ → base, emitter → GND,
LED− → collector, LED+ → supply.

## Power and audio

- ESP32-P4 powered from 5V USB (Orange Pi 3B USB port).
- Audio: Orange Pi 3B (DietPi, MPD) → AUX → amp → 2× Herdio 4" ceiling speakers.
- A ground loop isolator on the AUX line fixed static/hum.

## Breadboard tips

When the LED and IR stopped working after a rewire, a faulty breadboard was suspected. The fix was to
run each ground and 3V3 connection straight to the ESP header instead of through the shared power rails.
