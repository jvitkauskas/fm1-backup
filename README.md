# M-VAVE FM-1: full flash backup

`fm1-dump.bin` is the whole flash of one M-VAVE FM-1, as it came: stock
bootloader and the factory-installed firmware 015, nothing changed before.

## The dump

| | |
| --- | --- |
| File | `fm1-dump.bin`, 1,048,576 bytes (the whole flash, 0x00000-0xFFFFF) |
| SHA-256 | `3d5315f924147ded9f43e1da0d160e434ab0aa03fdc3ad55e888b64b87591d72` (also in `SHA256SUMS`) |
| Taken | 2026-10-17 |
| Device | M-VAVE FM-1, JieLi AC791N (WL82 series), chip key 0x980F |
| Flash | 1 MiB SPI NOR, JEDEC ID 0x856014 |

Checked:

- read twice; both reads identical;
- each read matches the CRC32 the chip computed over the whole flash;
- the flash header and both file tables decode; `app.bin` and `cfg_tool.bin`
  pass their CRC16s.

## The firmware is not the published 015

The installed `app.bin` is 580,892 bytes, CRC16 `BD93`. M-VAVE's published
packages all differ from it:

| Package | `app.bin` |
| --- | --- |
| `FM-1_015.fwsc` | 581,564 bytes, `5D48` |
| `FM-1_014.fwsc` | 584,956 bytes, `C796` |
| `FM-1_009.fwsc` | 583,068 bytes, `8F57` |

It reports itself as version 015, but it is a different build, probably the
factory one. This dump is the only copy of it.

## What is in it

| Range | Contents | Unique to this unit? |
| --- | --- | --- |
| 0x00000-0x03FFF | flash header, stock bootloader (`uboot.boot`), boot config (`isd_config`) | no: identical to the 015 package |
| 0x04000-0x92FFF | app area: the factory 015 `app.bin` and `cfg_tool.bin` | **this build is not in any published package** |
| 0x93000-0xE8FFF | VM: settings store (281 bytes used) | **yes** |
| 0xE9000-0xE9FFF | BTIF: Bluetooth data (10 bytes used) | **yes** |
| 0xEA000-0xFBFFF | USR: stock resources (40,458 bytes used) | as installed with this firmware |
| 0xFC000-0xFEFFF | not listed in the region table | empty |
| 0xFF000-0xFFFFF | key_mac | empty on this unit (all 0xFF) |

The FM-1 has no factory calibration area like the SMK-37's.

## How it was taken

The unit was put in the chip's USB download mode (the stock firmware has no
key for it: a USB_KEY dongle does it, see
[smk37-pro-backup](https://github.com/jvitkauskas/smk37-pro-backup) for a
Raspberry Pi Pico one). Then, on Linux, with
[M-Flasher](https://github.com/jvitkauskas/m-flasher) 0.4.0:

```sh
m-flasher backup DIR
```

twice. M-Flasher runs its flash loader in the chip's RAM and only reads; the
loader checks each read against a CRC32 it computes on the chip.

## Restoring

In download mode, with M-Flasher: **Restore a backup** and choose
`fm1-dump.bin` (command line: `m-flasher restore fm1-dump.bin`). It shows
which sectors differ and writes only those, then verifies them. Writing the
whole dump back returns the unit to exactly this state, including the
factory firmware, which M-VAVE's packages cannot.
