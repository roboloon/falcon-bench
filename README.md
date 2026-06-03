# Falcon Bench

A Windows desktop application for the **Pergam Falcon Plus 2** laser methane analyzer.
It connects to the sensor over **Ethernet/UDP**, drives it from standby to operation,
and shows live readings — total methane concentration (**Leak cont**, in ppm·m), the
leak flag, distance, temperature, GPS, and the device's internal signal waveforms — with
optional logging to CSV and the vendor `.dat` format.

> **Owner:** roboloon UG (haftungsbeschränkt) · **Author:** Jajnabalkya Guhathakurta
> · **Contact:** jajnabalkya.guhathakurta@roboloon.com

---

## Download

Get the latest installer from the **[Releases](../../releases/latest)** page, or directly:

**→ [falcon_bench_Setup.exe](../../releases/latest/download/falcon_bench_Setup.exe)**

📖 For a full step-by-step operating guide, see **[USAGE.md](USAGE.md)**.

## System requirements

- **Windows 10 or 11, 64-bit.**
- A wired **Ethernet** connection to the sensor (a USB‑C/USB Ethernet adapter is fine).
- No runtime to install — the Qt libraries are bundled in the installer.

## Installation

1. Download **`falcon_bench_Setup.exe`**.
2. Run it. It installs **per-user** (no administrator rights required); you can opt to
   install for all users if you have admin rights.
3. Choose whether to create a **desktop shortcut**, then finish. Launch from the
   Start menu / desktop (**Falcon Bench**).
4. **Windows SmartScreen:** the app is not code-signed, so SmartScreen may show
   "Windows protected your PC". Click **More info → Run anyway**. (It's the unmodified
   installer you downloaded from this page.)
5. To remove it later: **Settings → Apps → Falcon Bench → Uninstall**.

## Connecting the hardware

1. Power the sensor and connect its **Ethernet** to the PC (directly or via a hub).
   Allow ~2 minutes after power-on before expecting communication.
2. The sensor uses DHCP, falling back to **link-local Auto-IP** (`169.254.x.x`). For a
   direct PC↔sensor link, leave the PC's NIC on automatic/APIPA so it also gets a
   `169.254.x.x` address, or set it manually to `169.254.0.1`.
3. Find your PC's interface IP on that adapter: **Settings → Network → (adapter) →
   details**, or run `ipconfig` and note the `169.254.x.x` address of the sensor's NIC.

## Using the application

### Connect & bring up
1. In the top bar, set **Interface IP** to your PC's address on the sensor's network
   (e.g. `169.254.7.165`) and click **Connect / Discover**. When found, the device IP
   appears (e.g. `169.254.1.0`) and the state shows **Idle**.
2. Click **Start (laser on → Work)** to bring the sensor up. It runs
   Idle → Temp-stabilizing → Line-stabilizing → Auto-test → **Work**. A **cold unit can
   take 1–2 minutes** and may auto-retry temperature stabilization — this is normal.
3. When it reaches **Work**, live readings stream. Click **Stop (laser off)** to return
   the sensor to Idle.

> ⚠️ **Laser safety:** *Start* switches the measurement laser **on**. Only do so when it
> is safe to operate the laser (clear/contained beam path). *Stop* switches it off.

### Tabs
- **Live** — large **Leak cont (ppm·m)** readout, a green/red **leak** indicator,
  Ampl / Distance / Temperature, and a rolling plot of Leak cont.
- **Expert** — every decoded field of the result packet, live, plus a count of all
  received notifications.
- **Waveform** — live plot of any of the 18 internal signal vectors (default
  *log anl*, the methane absorption line shape).
- **Config** — query and edit the device's settings blocks (calculation, operating
  mode, test mode, communication, distance/calibration, laser, GPS), with
  *Save to ROM / Restore / Factory reset*.
- **Errors** — a log of device error notifications (time, code, description).
- **About** — version and contact information.

### Settings & control (⚙ button)
The **⚙ Settings** button opens controls for: **Laser ON/OFF** (Test mode / Idle), the
control commands (Start, Stop, Test, Write base, Fix base), and the measurement
parameters — **leak threshold level**, **temperature mode** (Cold/Moderate/Hot), and
**calculation mode** (Fast 10 ms / Medium 20 ms / Slow 40 ms). Temperature and
calculation mode can only be set in Idle or Test mode.

### Logging
Tick any of the four log toggles in the top bar **before** pressing Start:

| Toggle | File(s) written | Contents |
|--------|-----------------|----------|
| **CSV** | `csv.csv` | the full result packet — every field + flag bits + 123 leak-line points |
| **Pergam** | `pergam_fast/slow/track.dat`, `pergam_leak.txt` | vendor-format binary logs |
| **Vectors** | `vector_NN_name.dat` | all 18 visualization vectors (heavy, ~0.9 MB/s) |
| **Errors** | `error.csv` | device error notifications |

Each run creates its own folder `logs/<timestamp>/` next to the app holding that
session's files. Use **Open logs folder** to browse them. (All file writing runs on a
background thread, so logging never slows the display.)

## Reading

- **Leak cont** is the headline value, in **ppm·m** (column density — concentration
  integrated over the beam path).
- A reading needs a **target** the beam reflects off (so a distance is measured) and
  methane in the path; with the sensor aimed at nothing on a bench, Leak cont and
  distance read **0**.

## Troubleshooting

- **"Bind failed" / device not found** — the Interface IP isn't your PC's address on
  the sensor's adapter, or the cable/adapter dropped. Re-check `ipconfig`, reseat the
  cable, and confirm the adapter is present.
- **Stays in Temp-stabilizing / "error 0"** — a cold start can time out and retry; give
  it a couple of minutes. Check the **Errors** tab for details.
- **No logs appear** — make sure a log toggle was enabled before **Start**; click
  **Open logs folder**.

## About this repository

This repository hosts the **installer downloads** only. The application source is
maintained privately. For questions or support, contact
**jajnabalkya.guhathakurta@roboloon.com**.
