# Sonoff S31 Tasmota Flashing Guide — UM Technothon 2026

> **Hardware:** 2× Sonoff S31 (not Lite) + CP2102 USB-to-Serial adapter

---

## 1. Hardware Setup

### Pin Connections

| CP2102 | Sonoff S31 PCB |
|--------|---------------|
| 3.3V | 3.3V |
| GND | GND |
| TX | RX |
| RX | TX |
| GND | **GPIO0** (jumper to enter flash mode) |

### Steps

1. Open the S31 casing — pry open the clips around the edges using a flat tool.
2. Locate the 4-pin header on the PCB (labeled VCC, TX, RX, GND).
3. Solder or use pogo pins to connect CP2102 to the header. DO NOT apply 5V — use 3.3V only.
4. Connect GPIO0 to GND (use a jumper wire). This puts the ESP8266 into flash mode.
5. Plug CP2102 into your USB port. Do not power the S31 via AC yet.

> **If the CP2102 3.3V output cannot supply enough current (~200mA):**
> Power the S31 separately via its USB input (if available) or AC mains, while keeping GND and signal lines connected. GPIO0 must still be pulled to GND.

---

## 2. Flash Tasmota

### Download Firmware
- **Recommended:** `tasmota-lite.bin` (minimal — no unnecessary drivers)
- **Download:** https://github.com/arendst/Tasmota/releases/latest

### Using Tasmotizer (Windows — easiest)

1. Download Tasmotizer from: https://github.com/tasmota/tasmotizer/releases
2. Launch Tasmotizer, select your COM port (check Device Manager for CP2102).
3. Click the folder icon, select `tasmota-lite.bin`.
4. Click **"Tasmotize!"** — wait for progress bar (30-60 seconds).
5. When done, remove the GPIO0→GND jumper.
6. Press the physical button on the S31 or power cycle the CP2102.
7. The S31 will now broadcast a Wi-Fi access point named `tasmota-XXXX`.

### Using esptool.py (alternative)

```bash
# Erase flash first
esptool.py --port COM3 erase_flash

# Flash Tasmota
esptool.py --port COM3 --baud 115200 write_flash -fs 1MB -fm dout 0x0 tasmota-lite.bin

# Remove GPIO0→GND jumper and power cycle after completion
```

> Repeat for the second S31. Each plug gets the same firmware — they differentiate by MQTT topic later.

---

## 3. Initial Configuration

1. Connect your computer/phone to the Wi-Fi network `tasmota-XXXX`.
2. Open browser to `192.168.4.1`.
3. Click **"Configure Wi-Fi"** — enter your 2.4GHz network credentials.
4. S31 reboots and connects to your Wi-Fi. Note its IP address (check router DHCP list or serial monitor).

---

## 4. Apply S31 Template

The template configures all GPIO pins and enables energy monitoring.

1. Open browser to the plug's IP address.
2. Go to **Configuration → Configure Other**.
3. Paste the template below into the **Template** field:

```json
{"NAME":"Sonoff S31","GPIO":[17,145,0,146,0,0,0,0,21,56,0,0,0],"FLAG":0,"BASE":41}
```

4. Click **Save**. The plug will reboot.
5. Go to **Configuration → Configure Module** — verify it shows "Sonoff S31" (User Module).
6. Click **Save** again.

### Pin Assignment Reference

| GPIO | Component | Function |
|------|-----------|----------|
| GPIO0 | Button | Physical toggle |
| GPIO1 | CSE7766 Tx | Energy monitoring data |
| GPIO3 | CSE7766 Rx | Energy monitoring data |
| GPIO12 | Relay | Power switching |
| GPIO13 | LED (inverted) | Status indicator |

> The energy monitoring IC is CSE7766 (works identically to HLW8032 from the architecture doc — both communicate via UART and produce the same SENSOR telemetry).

---

## 5. MQTT Configuration

1. Go to **Configuration → Configure MQTT**.
2. Fill in:

| Field | Value |
|-------|-------|
| **Host** | MQTT broker IP (e.g., `192.168.1.100`) |
| **Port** | `1883` |
| **Client** | `DVES_XXXX` (default, or set custom) |
| **Topic** | `smartplug1` (for plug #1), `smartplug2` (for plug #2) |
| **Full Topic** | `%topic%/%prefix%/` |

3. Click **Save**. The plug reboots.
4. Verify MQTT is working — messages appear on topic `tele/smartplug1/SENSOR`.

---

## 6. Tasmota Settings via Console

Open **Console** (Tools → Console) and run these commands:

```
# Set power-on state to OFF (safe default)
PowerOnState 0

# Enable energy monitoring
SetOption21 1

# Telemetry interval — report every 60 seconds
TelePeriod 60

# MQTT topic for second plug
Topic smartplug2
```

---

## 7. Verify MQTT Messages Match Architecture

Once configured, the plug will publish:

### Sensor Report (every 60s)
```
tele/smartplug1/SENSOR
{
  "ENERGY": {
    "TotalStartTime": "2025-01-01T00:00:00",
    "Total": 0.000,
    "Yesterday": 0.000,
    "Today": 0.000,
    "Power": 0,
    "ApparentPower": 0,
    "ReactivePower": 0,
    "Factor": 0.00,
    "Voltage": 230,
    "Current": 0.000,
    "PowerFactor": 0.00,
    "Period": 60,
    "ImportActive": 0.000,
    "ExportActive": 0.000,
    "ImportReactive": 0.000,
    "ExportReactive": 0.000,
    "TotalTariff": 0.000
  }
}
```

### Relay State
```
stat/smartplug1/POWER → {"POWER":"ON"} or {"POWER":"OFF"}
```

### Commands (sent from app)
```
cmnd/smartplug1/POWER → ON, OFF, or TOGGLE
cmnd/smartplug1/POWER ON
```

These topics match exactly with the architecture document's MQTT communication layer — **no code changes needed**.

---

## 8. Power Monitoring Calibration

Energy readings from CSE7766 may drift. To calibrate:

1. Plug a known load (e.g., 100W incandescent bulb) into the S31.
2. In Console, run:

```
# Set voltage calibration (default 230.0)
VoltageSet 230.0

# Set current calibration (adjust until reading matches load)
CurrentSet 0.435   # for 100W at 230V

# Set power calibration
PowerSet 100
```

3. Compare Tasmota's reported values with a reference meter. Adjust until <2% error.

> Calibration constants persist across reboots. Repeat for each plug.

---

## 9. Ordering Summary

| Item | Qty | Est. Cost |
|------|-----|-----------|
| Sonoff S31 (not Lite) | 2 | ~RM 70 |
| CP2102 USB-to-serial | 1 | ~RM 12 |
| Jumper wires (F-F) | 4 | ~RM 3 |
| US-to-UK plug adapter (if S31 is US type) | 2 | ~RM 10 |
| **Total** | | **~RM 95** |

---

## Appendix: Troubleshooting

| Problem | Fix |
|---------|-----|
| No serial connection | Check CP2102 drivers (silabs.com) + verify COM port in Device Manager |
| Flashing fails mid-way | Power the S31 separately (AC input or separate 3.3V supply) — CP2102 may not supply enough current |
| No Wi-Fi AP after flash | Hold button 5s for factory reset; or reflash with GPIO0→GND connected |
| Energy readings all zero | Verify template applied correctly — GPIO1 and GPIO3 must be set to CSE7766 Tx/Rx |
| MQTT not connecting | Verify broker IP, port 1883 open, same subnet |
