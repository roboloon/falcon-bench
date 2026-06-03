# Falcon Bench — Usage Guide

A step-by-step guide to operating the **Pergam Falcon Plus 2** with the Falcon Bench
application. For download and installation see the [README](README.md).

---

## 1. Before you start

- The sensor communicates over **Ethernet/UDP**. Connect it to the PC (directly or via
  a USB/USB‑C Ethernet adapter) and **power it on ~2 minutes** before connecting.
- The sensor uses DHCP, falling back to **link-local Auto-IP** (`169.254.x.x`). For a
  direct PC↔sensor link, leave the PC NIC on automatic (it gets a `169.254.x.x`
  address too) or set it manually to `169.254.0.1`.
- Find the PC's address on that adapter with `ipconfig` — note the `169.254.x.x` value.
  That is the **Interface IP** you'll enter in the app.

> ⚠️ **Laser safety.** Pressing **Start** (or **Test**) switches the measurement laser
> **on**. Only do this when the beam path is safe/contained. **Stop** switches it off.

## 2. Connect and bring the sensor up

1. **Interface IP** (top-left field): enter the PC's address on the sensor's network,
   e.g. `169.254.7.165`.
2. Click **Connect / Discover**. On success the bar shows **Device: 169.254.x.x** and
   **State: Idle**.
3. Click **Start (laser on → Work)**. The sensor advances through:
   `Idle → Temp stabilizing → Line stabilizing → Auto test wait → Auto test → Work`,
   with a **progress %** shown for each stabilization phase.
   - A **cold unit can take 1–2 minutes** and may **auto-retry** if temperature
     stabilization times out — this is expected.
4. At **Work**, live readings stream (the result packet arrives every 10 ms).
5. Click **Stop (laser off)** to return the sensor to **Idle**.

## 3. The tabs

### Live
The primary view:
- **Leak cont** — the headline methane reading, in **ppm·m** (column density), shown
  large.
- **Leak indicator** — green (*no leak*) / red (*LEAK*), from result-flag bit 4.
- **Ampl** (analytical amplitude; ≥ 3.0 is a healthy signal), **Distance**,
  **Temperature**.
- A **rolling plot** of Leak cont over the last ~300 samples.

### Expert
- A **notifications-received** summary: counts of result / vector / event / progress /
  error packets, plus the last event and last error.
- A live table of **every decoded field** of the result packet (amplitudes, all
  concentrations, correlations, full GPS block, the flag bits, etc.).

### Waveform
- A live plot of one of the **18 visualization vectors** (the device's internal
  processed signals). The selector defaults to **`log anl`** — the analytical-channel
  absorption line shape. (Vectors are Ethernet-only.)

### Config
Query and edit the device's **settings blocks** (§5.3.2):
1. Pick a block (Calculation, Operating mode, Test mode, Communication,
   Distance/calibration, Laser, GPS).
2. **Query** — fetches the block and shows its fields in an editable table plus a raw
   hex dump. (Query works in any state.)
3. Edit values, then **Set** — sends your changes. Only the fields you edit change;
   arrays/reserved bytes are preserved. *Operating-mode, laser and distance blocks can
   only be set in Idle or Test mode.*
4. **Save to ROM** persists the current values; **Restore from ROM** reloads them;
   **Factory reset** restores defaults (⚠️ for the distance block this wipes
   calibration — confirmation required).

### Errors
A running log of **device error notifications** (time, code, and a description — e.g.
temperature-stabilization timeout). Use **Clear** to reset the list.

### About
Version and contact details.

## 4. Settings & control (⚙ button)

The **⚙ Settings** button opens a dialog with:
- **Laser ON (Test mode) / Laser OFF (Idle).**
- **Control commands:** Start, Stop, Test, Write base, Fix base.
- **Measurement parameters** (with *Apply* / *Refresh*):
  - **Leak threshold** — Level 1–5.
  - **Temperature mode** — Cold / Moderate / Hot. *(Idle/Test only.)*
  - **Calculation mode** — Fast (10 ms) / Medium (20 ms) / Slow (40 ms). *(Idle/Test only.)*

## 5. Logging

Enable any of the four toggles in the top bar **before** pressing **Start**. Each run
creates `logs/<timestamp>/` (next to the app) containing that session's files:

| Toggle | File(s) | Description |
|--------|---------|-------------|
| **CSV** | `csv.csv` | the complete result packet — 201 columns: every field, the 7 flag bits, and all 123 leak-line points. One row per result. |
| **Pergam** | `pergam_fast.dat`, `pergam_slow.dat`, `pergam_track.dat`, `pergam_leak.txt` | the vendor log format. |
| **Vectors** | `vector_NN_<name>.dat` | all 18 visualization vectors (heavy, ~0.9 MB/s). |
| **Errors** | `error.csv` | device error notifications (`timestamp_ms,code,description`). |

Click **Open logs folder** to browse them. All writing happens on a background thread,
so logging never slows the live display.

### File formats (for analysis)
- **`csv.csv`** and **`error.csv`** are plain CSV with a header row.
- **`pergam_*.dat`** and **`vector_*.dat`** are little-endian binary. Each record is a
  **64-bit LabVIEW double timestamp** (seconds since 1904-01-01) followed by
  **32-bit floats**: 8 fields for `fast`, 12 for `slow`, 9 for `track`, and the point
  array (123 or 200) for each vector. `pergam_leak.txt` is tab-delimited text.

## 6. Reading the data

- **Leak cont** is in **ppm·m** (concentration × beam path length).
- A real reading needs a **target** the beam reflects off (giving a distance) and
  methane in the path. With the sensor on a bench aimed at nothing, **Leak cont and
  distance read 0** — that's normal.

## 7. Troubleshooting

| Symptom | Fix |
|---------|-----|
| **Bind failed / device not found** | The Interface IP isn't your PC's address on the sensor's adapter, or the cable/adapter dropped. Re-check `ipconfig`, reseat the cable, confirm the adapter is present. |
| **Stuck in Temp/Line stabilizing, or "error 0/1"** | Cold-start stabilization can time out and retry; allow a couple of minutes. See the **Errors** tab. |
| **Set rejected: "Idle/Test mode only"** | Operating-mode/laser/distance blocks can only be written in Idle or Test mode — Stop to Idle, then Set. |
| **No log files** | Enable a log toggle **before** Start; then use **Open logs folder**. |
| **SmartScreen blocks the installer** | The app isn't code-signed — click **More info → Run anyway**. |

---

Questions or support: **jajnabalkya.guhathakurta@roboloon.com**
