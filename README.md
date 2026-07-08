# zmk-config — CannonKeys Link + PandaKB dongle

ZMK config for a **CannonKeys Link** split keyboard driven through a
**PandaKB nRF52840 dongle** (nice!nano v2 compatible, 1.3" SH1106 OLED on
I2C: SDA P0.17 / SCL P0.20). The dongle is the BLE central and holds the
keymap; both Link halves run as peripherals.

## Build targets (GitHub Actions)

| Artifact | Board | What it is |
|---|---|---|
| `link_dongle` | `nice_nano_v2` + `link_dongle dongle_display` | Central + keymap + OLED + ZMK Studio |
| `link_left_peripheral` | `link_left` | Left half, demoted to peripheral via `-DCONFIG_ZMK_SPLIT_ROLE_CENTRAL=n` |
| `link_right_peripheral` | `link_right` | Right half (peripheral by default) |
| `settings_reset_dongle` / `settings_reset_link_left` / `settings_reset_link_right` | respective boards | Bond-clearing images |

## Flash order (first setup)

Old split bonds cause the halves to keep looking for each other instead of
the dongle, so reset everything first:

1. Flash **settings_reset** to the dongle, the left half, and the right half
   (double-tap reset to enter the bootloader, copy the matching `.uf2`).
   Give each a few seconds to boot after flashing.
2. Flash **`link_dongle.uf2`** to the dongle.
3. Flash **`link_left_peripheral.uf2`** to the left half.
4. Flash **`link_right_peripheral.uf2`** to the right half.
5. Plug the dongle into USB, then power on both halves. They pair to the
   dongle automatically; the OLED shows a battery gauge per half once
   connected.

Re-flashing keymap updates later only needs step 2 (only the dongle holds
the keymap).

## ZMK Studio

The dongle build includes the `studio-rpc-usb-uart` snippet and the Link
physical layout, so [ZMK Studio](https://zmk.studio) works over the dongle's
USB connection. Locking is disabled (`CONFIG_ZMK_STUDIO_LOCKING=n`); the
stock keymap also keeps `&studio_unlock` on the lower layer.

## OLED widgets

The status screen comes from
[englmaxi/zmk-dongle-display](https://github.com/englmaxi/zmk-dongle-display)
(v0.3): per-half battery gauges, output, layer name, modifiers, and Bongo
Cat. Toggles for `config/link_dongle.conf`:

- `CONFIG_ZMK_DONGLE_DISPLAY_BONGO_CAT=n` — disable Bongo Cat
- `CONFIG_ZMK_DONGLE_DISPLAY_WPM=y` — add a WPM widget
- `CONFIG_ZMK_DONGLE_DISPLAY_MAC_MODIFIERS=y` — macOS modifier symbols

Note: the two battery gauges are ordered by peripheral slot (registration
order), not labeled left/right.

## Sources / provenance

- Board ids `link_left` / `link_right` are verbatim from
  `zmk-cannonkeys-keyboards/boards/arm/link/link.zmk.yml`. The module is
  pinned in `config/west.yml` to commit `0b6f0219`, whose main branch
  includes the fixed physical-layout key order.
- The `link_dongle` shield vendors the Link matrix transform
  (`default_transform`), physical layout (`physical_layout0`), and sensor
  slots from that commit, plus the SH1106 OLED node from PandaKB's own
  `zmk-sofle-j-dongle` dongle overlay (same dongle hardware, including its
  `width = <129>` quirk, kept verbatim from the known-working config).

## Known caveats (unverified until CI runs)

- CannonKeys' own CI builds the Link against a personal ZMK fork
  (`awkannan/zmk@develop`). The board only uses mainline features
  (grep-verified: no custom drivers/compatibles), but this repo builds it
  against mainline `zmk@v0.3` — confirm with the first GitHub Actions run.
- If your Link halves have nice!view displays, add `shield: nice_view` to
  the `link_left` / `link_right` entries in `build.yaml` (that is how
  CannonKeys' CI builds them).
