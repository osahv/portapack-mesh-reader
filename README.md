# Mesh Reader — a receive-only Meshtastic viewer for the HackRF PortaPack (Mayhem)

**Mesh Reader** listens to a [Meshtastic](https://meshtastic.org) network with a HackRF + PortaPack and shows what it hears:
channel messages, the list of nodes with signal and age, and every node ever heard. It **never transmits**: there is no send
button, no relaying, no beacons, no telemetry. It is a utility for watching a mesh, not a Meshtastic node.

It is built from the Mesh app of [Mayhem pull request #3306](https://github.com/portapack-mayhem/mayhem-firmware/pull/3306)
(a software LoRa modem running on the HackRF's M4) with the transmit side, the map, telemetry, settings pages, QR codes and
key exchange removed. That makes the app small enough (about 35 KB of flash instead of about 118 KB) that all the other
Mayhem apps keep working next to it.

> **Status:** tested on one setup: HackRF One R10 + PortaPack **H2**, **EU 433 MHz**, **LongFast**, against a Heltec V3 and a
> Wireless Paper on a private mesh with custom AES-256 channels. Messages, nodes, Cyrillic text and the "All heard" list work.
> The Mesh code is unmerged upstream work; this is an unofficial build. Not affiliated with Meshtastic, Mayhem or the PR author.

## What it does

- **Chat**: texts of the channel you pick (`Ch` button cycles your channels; up to ten, each with its own history on the card).
  Shows `name: text`, time, a colour stripe per node, a padlock for private channels. Damaged frames are shown marked.
- **Nodes**: the 14 freshest nodes with name, signal and age; sortable. **All heard** lists every node ever seen (kept on the
  card in `/LOGS/mesh_seen.bin`, up to 256).
- **Find** (tap or Select on a node): the node's SNR / RSSI, a bar graph of its last 48 packets, a CLOSE / FAR word and a
  WARMER / COLDER trend while you walk. With a position for both ends it also shows the exact distance and a north-up
  arrow. "Where I am" is **never a fixed coordinate**: it is a live GPS fix if the PortaPack has one, otherwise the latest
  position of the node you carry (open that node and press **Me**; saved as `mynode=` in the ini).
- **Channels with 128- and 256-bit keys** (stock PR #3306 takes only 128-bit).
- **Cyrillic and other alphabets** from glyph tables on the card (`APPS/mesh_font*.fnt`, `c_glyph=1`).
- Settings live in `/SETTINGS/meshtastic.ini` (no settings screens). Optional `log=1` writes every received frame to
  `/LOGS/mesh_rx.csv`.

## Install

1. Back up your SD card (`FIRMWARE`, `APPS`, `BASEBAND`, `SETTINGS`).
2. Unpack the release into the **root of the SD card** (merge with the existing folders). `FIRMWARE`, `APPS` and `BASEBAND`
   must all come from the same build: apps with another version hash are greyed out, and a missing baseband shows `NoImg`.
3. **Utilities → Flash Utility →** `portapack-mayhem_mesh-reader-4.bin`.
4. Edit `/SETTINGS/meshtastic.ini`: `region`, `preset`, `fslot` must match your network, and add your channels
   (`c1_name`, `c1_key` as 32 or 64 hex digits, `c1_en=1`, …). `tools/mesh_channels_to_ini.py` copies them from a node over USB:

   ```bash
   pip install meshtastic
   python3 tools/mesh_channels_to_ini.py --port /dev/cu.usbserial-0001 --ini /Volumes/<SD>/SETTINGS/meshtastic.ini
   ```
   The keys are stored **in plain text on the card**; keep the card private.
5. Start **MeshRead** in the main menu. Connect an antenna for your band.

To go back, flash an official `.bin` and restore your backup.

## Reception quality

A HackRF has no automatic gain control, so a strong nearby node can overload the receiver (`crc bad`, garbled text). Start
with LNA 16-24 and VGA 16-24 (the `L:` `V:` fields in the Chat status row, or `rxlna` / `rxvga` in the ini).

## Limits

- Nothing is ever sent, so a node does not see you and you cannot reply or request data.
- Six baseband images (APRS RX, AIS, BLE TX, RDS, AFSK TX, FSK TX) are loaded from the SD card to make room in flash; they need a free
  contiguous block of memory and may report `NoImg` if you opened many apps first. Restart and open them first.
- Flash is full: adding code requires removing something.

## Build it yourself

```bash
scripts/build.sh   # needs Docker; fetches PR #3306 at the tested commit 51135ec, applies patches/, builds
```

Output: `dist/mesh-reader-4/{FIRMWARE,APPS,BASEBAND}`. The full-featured variant with sending lives in
[portapack-mesh](https://github.com/osahv/portapack-mesh).

## Credits and license

- The Mesh app and the LoRa modem: **verhogladov** ([PR #3306](https://github.com/portapack-mayhem/mayhem-firmware/pull/3306)).
- Firmware: [Mayhem](https://github.com/portapack-mayhem/mayhem-firmware) (GPL).
- This repository: GPL-2.0-or-later ([LICENSE](LICENSE), [NOTICE](NOTICE)). Developed with AI assistance (Claude). Flash custom
  firmware at your own risk.
