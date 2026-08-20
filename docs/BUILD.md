# Build Instructions

A step-by-step assembly order for going from a pile of AliExpress parts to a
lamp that doesn't let the magic smoke out. Read it once before you touch a
soldering iron — most of the mistakes here are only fixable *before* you glue
the diffuser channel shut.

## 1. Before you solder anything

- Check off every part against [`Arc_lamp_parts.xlsx`](../Arc_lamp_parts.xlsx)
  — Sheet1 for the core BOM, "Additional Hardware" section for the
  connectors/resistors/wire you'll inevitably realize you're missing at 11pm.
- Read [`docs/wiring.md`](wiring.md) in full at least once. You don't need to
  memorize it, you just need to know it exists so you stop and check it
  instead of guessing which GPIO was which.
- Bench-test the power path **before it's anywhere near the ESP32-S3 or the
  strip.** Feed the CH224K from a USB-C PD source, confirm it's actually
  outputting 12V on a multimeter, then set the LM2596 trimpot to exactly
  5.00V into a dummy load (or just measure it open-circuit — close enough to
  not fry anything). Skipping this step is how boards die.

## 2. Power stage first

1. Wire USB-C connector → CH224K IN.
2. Bridge the CH224K's 12V solder-jumper pad. Confirm 12V on OUT+ with a
   multimeter before connecting anything downstream.
3. Wire CH224K OUT → LM2596 IN and, separately, to the WS2815 strip's 12V
   input.
4. Adjust the LM2596 trimpot until OUT+ reads 5.00V. Do this with nothing
   else connected to the 5V rail yet — cheap trimpots drift a little under
   load, so recheck once the ESP32-S3 is attached.
5. Add the fuse/polyfuse between USB-C VBUS and the CH224K input, and the
   bulk cap on the 12V rail near the strip header, per
   [`docs/wiring.md`](wiring.md).

If you only do one thing carefully in this whole build, make it this section.
Everything downstream is 3.3–5V logic that's much harder to kill; this is the
part with enough current to actually cause damage.

## 3. Perfboard layout

Use [`Arc_lamp_parts.xlsx`](../Arc_lamp_parts.xlsx) → **Perfboard Build**
sheet as your checklist while you go. Placed / Soldered / Tested columns are
dropdowns for a reason — resist the urge to skip straight to "Soldered"
before you've dry-fit the layout.

Rough placement order that keeps you from soldering yourself into a corner:

1. Power section (CH224K, LM2596, fuse, bulk cap) on one side of the board,
   physically separated from the logic section — keeps your fat 12V traces
   away from the sensitive I2C/UART lines.
2. ESP32-S3 dev board socketed or header-mounted in the middle — don't
   solder it in directly, you may want to pull it back out.
3. LD2410B and MPR121/BH1750 (shared I2C bus) on the logic side, near their
   respective GPIO headers.
4. Ground bus running the full length of the board, tied to every module.
5. Decoupling caps last, right at each IC's VIN/VDD pin — not off to the
   side where they're decorative instead of useful.

## 4. Bring-up, one module at a time

Don't wire everything and hope. Bring it up incrementally:

1. Power only — ESP32-S3 connected, nothing else. Flash a blank ESPHome
   config, confirm it boots and connects to Wi-Fi (or falls back to its own
   AP, which is also a pass).
2. Add the WS2815 strip. Flash, confirm `light.arc_lamp` turns on and
   responds to brightness/color commands.
3. Add the LD2410B. Confirm the `Presence` binary sensor flips when you
   walk in and out of range. Give it a minute to calibrate on first boot —
   it's not broken, it's just learning the room.
4. Add the MPR121 + BH1750 on the shared I2C bus. Confirm `i2c: scan: true`
   in the logs finds both addresses (`0x5A` and `0x23`) before debugging
   anything else — a missing device on the scan means a wiring problem, not
   a firmware problem.
5. Only once all four are independently confirmed working, flash the full
   [`arc_lamp.yaml`](../arc_lamp.yaml) and test the automations together
   (touch presets, presence auto-on with ambient-scaled brightness).

## 5. Final assembly

- Mount the strip in the silicone diffuser channel before final wiring —
  much easier with slack in the wires than after everything's fixed in
  place.
- Route the LD2410B so it isn't facing a mirror or other reflective surface
  at close range — see the wiring notes for why.
- Leave yourself a way to reflash. A dev board buried in a sealed enclosure
  with no accessible USB port is a lesson you only need to learn once.

## 6. You're done when

- The lamp turns on when you sit down and off when you leave.
- Touch pads reliably hit their five brightness steps.
- It doesn't dazzle you at 2pm or leave you fumbling in the dark at
  midnight — the ambient sensor should be doing its job invisibly.
- Nothing on the power stage is warm enough to be concerning after twenty
  minutes at full brightness. If the LM2596 is, add a small heatsink.

From here it's just iteration — tune the LD2410B's sensitivity via ESPHome
logs, adjust the brightness curve in `arc_lamp.yaml` to taste, and enjoy not
having a lamp that phones home to a cloud server just to turn on.
