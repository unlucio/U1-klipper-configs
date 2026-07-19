# SpoolLink Bridge

Automatic RFID → Spoolman integration for the Snapmaker U1 with PAXX CFW.

When a filament tag is scanned by the U1's built-in RFID reader, this bridge
automatically activates the matching spool in Spoolman — no manual selection needed.

**What it does:**
- Tag scanned by OpenRFID → webhook fires → Moonraker component looks up spool → `SET_ACTIVE_SPOOL`
- Works with **any tag type** OpenRFID can read: Bambu/Snapmaker MiFare Classic (UID only), OpenSpool NTAG215, ELEGOO, Anycubic
- Spoolman tracks filament consumption automatically from that point on

**What it does NOT do:**
- It does not change what the U1's touchscreen/Orca displays (that comes from the tag data itself, handled by OpenRFID's existing `success_exporter`)
- It is not a replacement for PAXX SpoolLink (see [Migration](#migration-to-paxx-spoollink) below)

---

## Prerequisites

- Snapmaker U1 with **PAXX CFW v1.4.1+**
- [Spoolman](https://github.com/Donkie/Spoolman) running on your network
- The [Spoolman multi-tool config](../spoolman/) already installed (for `SET_ACTIVE_SPOOL`, `SAVE_CURRENT_SPOOLS`, T0–T3 macros)
- [SpoolKid](https://github.com/marko-p/SpoolKid) (iOS, TestFlight) — recommended for registering UIDs to spools

---

## How it works

```
RFID Tag
   │
   ▼
OpenRFID (reads tag, fires webhooks)
   │
   ├─► success_exporter → /printer/filament_detect/set   ← existing, updates U1 GUI
   │
   └─► spoollink_bridge → /server/spoollink_bridge        ← NEW: this component
            │
            ▼
       Spoolman lookup (by UID in extra.card_uids or lot_nr)
            │
            ▼
       SET_GCODE_VARIABLE MACRO=T{ch} VARIABLE=spool_id VALUE={id}
       SAVE_CURRENT_SPOOLS
       SET_ACTIVE_SPOOL ID={id}
```

---

## Installation

### 1. Copy the Moonraker component

```bash
cp spoollink_bridge.py ~/moonraker/moonraker/components/spoollink_bridge.py
```

> **Note:** On PAXX CFW, Moonraker is at `/home/lava/moonraker/`.

### 2. Add the Moonraker config

Copy `moonraker_spoollink_bridge.cfg` to your Moonraker include directory:

```bash
cp moonraker_spoollink_bridge.cfg ~/printer_data/config/extended/moonraker/05_spoollink_bridge.cfg
```

Edit the file and set your Spoolman URL if it's not running on the printer itself:

```ini
[spoollink_bridge]
spoolman_url: http://192.168.1.100:7912
```

### 3. Add the OpenRFID webhook

Append the contents of `openrfid_webhook_addition.cfg` to your OpenRFID user config:

```bash
cat openrfid_webhook_addition.cfg >> /oem/printer_data/config/extended/openrfid_user.cfg
```

> **Important:** `/oem/` is the PAXX overlay filesystem.
> Run `touch /oem/.debug` once to enable persistence across reboots.
> All changes in `/oem/overlay/upper/` are lost on firmware updates — re-apply after updating.

### 4. Reboot the printer

A **full power cycle** (not just a Moonraker/Klipper restart) is required so OpenRFID picks up the new webhook config. OpenRFID runs as root and cannot be restarted as the `lava` user.

### 5. Verify

After reboot, check the Moonraker log:

```
grep -i spoollink ~/printer_data/logs/moonraker.log
```

Expected output:
```
Component (spoollink_bridge) loaded
SpoolLink Bridge: endpoint /server/spoollink_bridge registered
SpoolLink Bridge: extra.card_uids field already exists   ← or "created"
```

---

## Registering UIDs to spools

The bridge needs to know which tag UID belongs to which spool. There are two options:

### Option A: SpoolKid (recommended)

[SpoolKid](https://github.com/marko-p/SpoolKid) (iOS, free TestFlight beta) writes NFC tags and registers UIDs automatically.

1. Open SpoolKid → select spool → tap the NFC scan icon in the edit form
2. Hold the tag to your iPhone → UID is written to `lot_nr` in Spoolman
3. The bridge reads `lot_nr` format: `card_uid:UID` or `card_uid:UID1,card_uid:UID2`

> **Tip:** SpoolKid works with Spoolman over HTTP to private IP addresses (`http://192.168.x.x:7912`).
> HTTP to hostnames fails on iOS due to ATS — always use the IP address.

### Option B: Manual via Spoolman UI or API

In the Spoolman web UI, edit a spool and set the **Card UIDs** field (`extra.card_uids`) to the tag's UID.

The bridge creates this custom field automatically on first start.

To find a tag's UID without SpoolKid: hold the tag to the U1 reader, then check the Moonraker log:
```
grep 'card UID' ~/printer_data/logs/moonraker.log | tail -5
```

Or read it from Klipper's `filament_detect` object:
```bash
wget -qO- 'http://localhost:7125/printer/objects/query?filament_detect' | \
  python3 -c "import sys,json; d=json.load(sys.stdin); \
  [print(f'ch{i}:', bytes(ch['CARD_UID']).hex().upper() if isinstance(ch['CARD_UID'], list) else ch['CARD_UID']) \
  for i,ch in enumerate(d['result']['status']['filament_detect']['info'])]"
```

---

## UID formats by tag type

| Tag type | UID length | Example | Notes |
|----------|-----------|---------|-------|
| Bambu / Snapmaker MiFare Classic | 4 bytes | `E0BB543F` | UID only, content unreadable |
| OpenSpool NTAG215 / NTAG216 | 7 bytes | `04DBAD28BF2A81` | Full content readable |
| ELEGOO | 7 bytes | — | Content readable |

The bridge works with all of them — only the UID matters for Spoolman lookup.

---

## Migration to PAXX SpoolLink

PAXX SpoolLink (PR #491, coming in firmware v1.5.x) is the native version of this bridge, built directly into the firmware. Once it lands:

1. Remove the `[webhook_exporter spoollink_bridge]` section from `openrfid_user.cfg`
2. Remove `[spoollink_bridge]` from your Moonraker config
3. Remove `spoollink_bridge.py` from the Moonraker components directory
4. Migrate UIDs: SpoolLink uses `extra.card_uids`, SpoolKid uses `lot_nr` — check if a migration step is needed

UIDs already in `extra.card_uids` (registered via the Spoolman UI) are already in the format SpoolLink expects — no migration needed for those.

---

## Troubleshooting

**Bridge loaded but tag not recognized (`No spool found for UID ...`)**
→ UID not registered in Spoolman. Use SpoolKid or the manual method above.

**No log entries after tag scan**
→ OpenRFID did not fire the webhook. Check that:
- `/oem/.debug` exists (persistence enabled)
- The webhook section is in `/oem/printer_data/config/extended/openrfid_user.cfg`
- A full power cycle was done after adding the webhook

**`Component (spoollink_bridge)` missing from Moonraker log**
→ Config file not found. Check that `moonraker_spoollink_bridge.cfg` is in a directory included by `moonraker.conf`.

**Spoolman unreachable error in log**
→ Check `spoolman_url` in the config. On PAXX CFW, Spoolman runs on the host network, not on the printer. Use the host's IP address.
