# `/SETTINGS/meshtastic.ini` reference

Mesh Reader has no settings screens: everything lives in this plain text file on the SD card (`key=value`, one per line). The
app creates it with defaults on first start and saves it when you change the channel, the gains or the "carried node". Edit it
on a computer (Utilities → SD Over USB) while the app is closed.

## Radio

| Key | Meaning | Default |
|---|---|---|
| `region` | Index: 0 US, 1 EU 868, 2 **EU 433**, 3 Australia, 4 New Zealand, 5 Korea, 6 Taiwan, 7 Russia, 8 India, 9 Japan, 10 Malaysia, 11 Singapore, 12 Philippines, 13 China, 14 unset (the default: set yours) | 14 |
| `preset` | Index: 0 **LONG_FAST**, 1 LONG_SLOW, 2 VERY_LONG_SLOW, 3 MEDIUM_SLOW, 4 MEDIUM_FAST, 5 SHORT_SLOW, 6 SHORT_FAST, 7 SHORT_TURBO, 8 LONG_MODERATE | 0 |
| `fslot` | Frequency slot of the channel (the number your nodes show). 0 = derive it from the channel name hash, like stock nodes do | 0 |
| `freq_hz` | Fixed frequency in Hz; 0 = derive from region/preset/slot | 0 |
| `cr` | Coding rate override (5..8 = 4/5..4/8); other = the preset's own. Reception follows each packet's own header either way | 0 |
| `rxlna`, `rxvga`, `rxamp` | Receiver gains (LNA 0-40 dB, VGA 0-62 dB, RF amp 0/1). A HackRF has no AGC: start with 16-24 / 16-24 and watch the share of damaged frames | 32 / 32 / 0 |

## Channels

Up to ten channels besides the primary one. The primary channel uses the well-known default key and the name in `pchname`
(empty = the preset's default name).

| Key | Meaning |
|---|---|
| `pchname` | Primary channel name |
| `c1_name` … `c10_name` | Channel name, exactly as on your node (it is part of the channel hash) |
| `c1_key` … `c10_key` | Key as **32 hex digits (AES-128)** or **64 hex digits (AES-256)**; any other text is hashed as a passphrase |
| `c1_en` … `c10_en` | 1 = listen to this channel |
| `act_ch` | Channel shown in Chat (0 = primary, 1..10 = custom); the **Ch** button cycles it |

`tools/mesh_channels_to_ini.py` copies names and keys from a node over USB or WiFi. **Keys are stored in plain text**: keep the
card private.

## Display and logs

| Key | Meaning | Default |
|---|---|---|
| `c_glyph` | 1 = draw non-Latin text (Cyrillic etc.) from `APPS/mesh_font.fnt` and `mesh_font5.fnt` (switch it on for Cyrillic) | 0 |
| `c_fsize` | Chat font: 8 = compact 5x8, 16 = standard 8x16 | 16 |
| `c_time` | 0 none, 1 HH:MM, 2 HH:MM:SS | 1 |
| `c_name` | Sender label: 0 long name, 1 short name, 2 node id, 3 none | 0 |
| `c_db` | 1 = show signal (RSSI/SNR of the last packet) in the status row | 1 |
| `c_save` | Messages kept per channel on the card (history files `/LOGS/mesh_c<N>.txt`); 0 = do not save | 0 |
| `hline` | Lines of scrollback kept in memory | 20 |
| `log` | 1 = append every received frame to `/LOGS/mesh_rx.csv` | 0 |
| `crcpol` | 1 = show damaged text marked `[?]`; 2 = drop frames whose payload CRC failed | 1 |
| `ign_mqtt` | 1 = ignore frames that came through MQTT | 0 |
| `beep_rx` | 1 = chime on a new message | 0 |
| `key_repeat` | 1 = keep the key auto-repeat of the firmware; 0 = one press, one step | 0 |

## Find screen

| Key | Meaning |
|---|---|
| `mynode` | Hex id of the node **you carry** (set with the **Me** button in the Find screen). Its own position reports are "where I am" for distance and bearing. A live GPS fix, if the PortaPack has one, takes precedence. Coordinates are never stored |

## Files the app writes

| File | What |
|---|---|
| `/LOGS/mesh_c<N>.txt` | Chat history of channel N |
| `/LOGS/mesh_dms.txt` | Names (and keys) of known nodes, restored at start |
| `/LOGS/mesh_seen.bin` | Every node heard, 64-byte records, read with `tools/read_seen.py` |
| `/LOGS/mesh_rx.csv` | Frame log (only with `log=1`) |
