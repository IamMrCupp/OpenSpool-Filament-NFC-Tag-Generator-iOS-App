# OpenSpool Filament NFC Tag Generator

An iOS application for writing NFC tags compatible with the [OpenSpool](https://github.com/spuder/OpenSpool) open-source filament management system. This app is specifically tailored to support the full field set of the **Snapmaker U1** running the [paxx12 extended firmware](https://github.com/paxx12/SnapmakerU1-Extended-Firmware).

## Overview

OpenSpool uses NFC tags embedded in filament spools to automatically communicate filament metadata to your 3D printer. This app lets you write those tags directly from your iPhone, giving you full control over every supported data field — including the extended fields recognized by the Snapmaker U1 with paxx12 firmware.

## Features

- Write OpenSpool-compatible NFC tags from your iPhone
- Full support for all standard and extended fields used by the paxx12 firmware
- Save and reuse filament profiles
- Compatible with NTAG215 and NTAG216 NFC tags

## OpenSpool Tag Format

Tags store an `application/json` NDEF record. The following fields are supported:

| Field | Required | Description |
|---|---|---|
| `protocol` | Yes | Always `"openspool"` |
| `version` | Yes | Always `"1.0"` |
| `type` | Yes | Material type (e.g. `"PLA"`, `"PETG"`, `"ABS"`, `"TPU"`) |
| `color_hex` | Yes | Color as a hex string (e.g. `"FFAABB"`) |
| `brand` | Optional | Filament manufacturer name |
| `min_temp` | Optional | Minimum nozzle temperature (string, °C) |
| `max_temp` | Optional | Maximum nozzle temperature (string, °C) |
| `bed_min_temp` | Optional | Minimum bed temperature (string, °C) |
| `bed_max_temp` | Optional | Maximum bed temperature (string, °C) |
| `subtype` | Optional | Material variant designation |
| `alpha` | Optional | Color transparency value |
| `additional_color_hexes` | Optional | Up to 4 additional color hex values |
| `weight` | Optional | Spool weight in grams |
| `diameter` | Optional | Filament diameter (e.g. `"1.75"`) |

### Example Tag Payload

```json
{
  "protocol": "openspool",
  "version": "1.0",
  "type": "PETG",
  "color_hex": "1616161",
  "brand": "Generic",
  "min_temp": "230",
  "max_temp": "270",
  "bed_min_temp": "40",
  "bed_max_temp": "80",
  "diameter": "1.75"
}
```

## Requirements

- iPhone with NFC capability (iPhone 7 or later)
- iOS 14.0 or later
- NFC-writable NTAG215 or NTAG216 tags (13.56 MHz)
- Snapmaker U1 running the [paxx12 extended firmware](https://github.com/paxx12/SnapmakerU1-Extended-Firmware)

## Usage

1. Open the app and select or create a filament profile.
2. Fill in the filament metadata fields.
3. Hold an NFC tag to the top of your iPhone.
4. Tap **Write Tag** to program the tag.
5. Apply the tag to your filament spool.

Your Snapmaker U1 will automatically read the tag when a spool is loaded and apply the stored settings.

## Related Projects

- [OpenSpool](https://github.com/spuder/OpenSpool) — The open NFC filament standard this app is built on
- [paxx12/SnapmakerU1-Extended-Firmware](https://github.com/paxx12/SnapmakerU1-Extended-Firmware) — The extended firmware for the Snapmaker U1
- [filament-detect](https://github.com/suchmememanyskill/filament-detect) — External filament detection system used by the paxx12 firmware
- [RFID Support Docs](https://snapmakeru1-extended-firmware.pages.dev/rfid_support) — Official paxx12 firmware documentation for RFID/NFC tag support

## Support

This app runs on caffeine and spite. If it saved you some time (or sanity), consider fueling the next feature:

- [Ko-fi](https://ko-fi.com/IamMrCupp) — Buy me a coffee
- [Buy Me a Coffee](https://buymeacoffee.com/IamMrCupp) — Buy me another coffee

Disclaimer: No caffeine was wasted in the making of this app. Much was consumed.

## License

Copyright 2026 Aaron Cupp (IamMrCupp)

Licensed under the [Apache License, Version 2.0](LICENSE). You may not use this project except in compliance with the License. A copy is included in this repository, or you may obtain one at http://www.apache.org/licenses/LICENSE-2.0.
