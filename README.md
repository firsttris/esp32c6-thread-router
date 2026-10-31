<div align="center">

# 🔗 ESP32-C6 Thread Router

**Extend Your Thread Network Coverage with an Affordable ESP32-C6!**

<img src="./thread-router-image.jpg" alt="Seeed XIAO ESP32C6 as a Thread router in a 3D-printed case with an external SMA antenna" width="600">

[![ESPHome](https://img.shields.io/badge/ESPHome-2026.8%2B-blue?logo=esphome)](https://esphome.io/)
[![Thread](https://img.shields.io/badge/Thread-1.3-green)](https://www.threadgroup.org/)
[![Home Assistant](https://img.shields.io/badge/Home%20Assistant-Integration-41BDF5?logo=homeassistant)](https://www.home-assistant.io/)
[![Build](https://github.com/firsttris/esp32c6-thread-router/actions/workflows/build.yml/badge.svg)](https://github.com/firsttris/esp32c6-thread-router/actions/workflows/build.yml)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue)](LICENSE)

This ESPHome configuration turns a ~5 € ESP32-C6 board into a **Thread 1.3 router** (FTD, Full Thread Device) for **Home Assistant**. Thread is the network under **Matter**, so it extends the mesh for your Matter-over-Thread devices too. No WiFi needed, with diagnostics and OTA updates over Thread.

---

</div>

## ✨ Features

- 📶 **Thread router (FTD)**: extends your Thread mesh, no WiFi needed
- 🔐 **Encrypted API and OTA**: Home Assistant connection and firmware updates use one encryption key
- 📡 **Antenna switch** (Seeed XIAO ESP32C6): choose the built-in or external antenna from Home Assistant
- 📊 **Diagnostics in Home Assistant**: Thread role, RLOC16, channel, IP address, parent RSSI, TX retries, CCA errors, partition changes, uptime
- 💡 **Status LED**: blinks when something is wrong
- 🔁 **Restart button** in Home Assistant

## 📋 Prerequisites

| Requirement | Description |
|-------------|-------------|
| **🔧 Hardware** | ESP32-C6 board (tested with [Seeed Studio XIAO ESP32C6](https://www.seeedstudio.com/Seeed-Studio-XIAO-ESP32C6-p-5884.html)) |
| **🌐 Network** | Thread Border Router in network (e.g., Home Assistant with Thread integration) |
| **💻 Software** | ESPHome **2026.8 or newer** (required for the Thread diagnostic sensors). CI builds with the release pinned in [`requirements.txt`](requirements.txt) (`pip install -r requirements.txt`), and `docker-compose.yml` runs the same one; Renovate updates both |
| **🔌 Cable** | USB cable for initial flashing |
| **📦 Optional** | 3D-printed case - [XIAO ESP32-C6 Case](https://www.printables.com/model/1543275-xiao-esp32-c6-zigbee-router-case-split-lid-sma-ext) |

> **💡 Other boards:** The config also works on other ESP32-C6 boards. Remove the block marked `Seeed XIAO ESP32C6 specific` in `esp32c6-thread-router.yaml`, since it drives GPIO3, GPIO14 and GPIO15. ESPHome also supports Thread on ESP32-C5, ESP32-H2 and (since 2026.7) nRF52 boards, but this config is only tested on the ESP32-C6.

## 🔑 Required Information

### 🌐 1. Thread Network Dataset (TLV)

> **💡 Important:** The TLV contains ALL network parameters - you don't need to generate or enter anything manually!

> **🧭 Which network is the right one?** The network of your OpenThread Border Router (OTBR) is not automatically the one your devices are in. Devices paired via Google Home, Apple Home or Alexa usually live in the network of a Google TV Streamer, Nest Hub, HomePod or Echo. Use the *preferred network* under **Settings → Devices & Services → Thread**.
>
> **Recommendation: bring everything into one network.** Either let the OTBR join the other network via the *Open Thread Border Router* integration in Home Assistant (first import the credentials from your phone via the Companion App, then click *Add to preferred network* for the OTBR on the Thread page), or re-pair your devices through Home Assistant.

#### 📥 Retrieve TLV from existing Thread network:

**✅ Easiest method - via Home Assistant:**

1. Go to **Settings → Devices & Services → Thread**
2. Click the **ⓘ** icon next to your preferred network
3. Copy the hex string shown under **Active dataset TLVs**

> **💡 This always gives you Home Assistant's preferred network**, even if your border router is not an OTBR (e.g. a Google or Apple device).

**🖥️ Alternative - via SSH/Terminal:**

**Step 1:** SSH to your Server

**Step 2:** Retrieve TLV data (use `podman` instead of `docker` if you run Podman):
```bash
docker exec otbr ot-ctl dataset active -x
```

**Step 3:** Copy the hex string (e.g., `0e080000000000010000...`)

Put it into `secrets.yaml` (see [Installation, Step 1](#-step-1-configure-secrets)). The TLV contains:

<table>
<tr><td>✅ Channel</td><td>✅ Network Name</td><td>✅ PAN ID & Extended PAN ID</td></tr>
<tr><td>✅ Network Key</td><td>✅ PSKc</td><td>✅ Mesh-Local Prefix</td></tr>
</table>

> ⚠️ **Keep the TLV secret!** The network key is stored in the TLV **in plain text**. Anyone with your TLV can join your Thread network. Never commit `secrets.yaml` or post the TLV anywhere. The firmware image also contains the TLV, so don't share compiled `.bin` files.

### 🔐 2. API Encryption Key

The Home Assistant API and OTA updates are encrypted with a shared key. Generate one:

```bash
openssl rand -base64 32
```

Or copy a freshly generated key from the [ESPHome API docs](https://esphome.io/components/api/).

## 🚀 Installation

### 📝 Step 1: Configure Secrets

Create `secrets.yaml` next to `esp32c6-thread-router.yaml`:
```yaml
thread_tlv: "YOUR_TLV_HEX_HERE"
api_encryption_key: "YOUR_BASE64_KEY_HERE"
```

That's all you need - no WiFi credentials required!

### 📡 Step 2: Choose the Antenna (Seeed XIAO ESP32C6 only)

The XIAO ESP32C6 has a built-in ceramic antenna and a U.FL connector for an external antenna (e.g. the SMA antenna of the recommended case). The antenna is selected by an RF switch (GPIO3 = switch power, GPIO14 = antenna select).

If you use an **external antenna**, change this substitution before flashing:
```yaml
substitutions:
  antenna_default: RESTORE_DEFAULT_ON   # external antenna
```

You can also toggle the **External Antenna** switch in Home Assistant later. The setting is kept across reboots.

> ⚠️ Without an external antenna connected, keep the built-in antenna selected. Otherwise the range will be very poor.

### ⚡ Step 3: Compile and Flash Firmware

Connect your ESP32-C6 via USB and flash the firmware using local ESPHome:

```bash
esphome run esp32c6-thread-router.yaml --device=/dev/ttyACM0
```

> **💡 Note:** Replace `/dev/ttyACM0` with your device path (see troubleshooting below)

### 🔄 Updating Later (OTA over Thread)

After the first USB flash, updates go over the air via Thread, no USB cable needed:

```bash
esphome run esp32c6-thread-router.yaml --device esp32c6-thread-router.local
```

Works as long as your border routers route the Thread network into your LAN (IPv6). Over Thread an update takes a few minutes. After a network change you need USB once, see [Thread Network Was Recreated](#-thread-network-was-recreated--tlv-changed--networks-merged).

<details>
<summary><b>📚 Important Notes & Troubleshooting</b></summary>

### 📍 Device Paths by OS

| OS | Typical Paths |
|----|--------------|
| 🐧 Linux | `/dev/ttyUSB0`, `/dev/ttyACM0`, `/dev/ttyUSB1` |
| 🍎 macOS | `/dev/cu.usbserial-*`, `/dev/cu.usbmodem*` |
| 🪟 Windows | `COM3`, `COM4`, etc. |

**Check available ports:**
- Linux/macOS: `ls /dev/tty*`
- Windows: Device Manager

### 🔐 USB Permission Issues

**Standard Linux - Add user to dialout group:**
```bash
sudo usermod -a -G dialout $USER
# Then log out and back in
```

**Fedora Atomic/Bazzite with rootless Docker/Podman:**

The dialout group doesn't work reliably on immutable systems. You need to fix permissions before each flash:

```bash
# Check permissions
ls -la /dev/ttyACM0
# Output: crw-rw----. 1 root dialout 166, 0 ...

# Fix temporarily (resets on USB reconnect)
sudo chmod 666 /dev/ttyACM0

# If using Docker/Podman, restart the container
docker compose restart
```

> ⚠️ **Note:** You need to run `sudo chmod 666` each time you reconnect the USB device.

**Inside a Distrobox/Toolbox container:** the device shows up as `nobody:nogroup` and `sudo` inside the box can't change it. Run the `chmod` on the host, e.g. `distrobox-host-exec sudo chmod 666 /dev/ttyACM0`.

**SELinux (Fedora) with rootless Podman:** even with `chmod 666`, the container may get `Permission denied` on the serial port. Disable SELinux labeling for the container (`security_opt: label=disable`, already set in `docker-compose.yml`), or with `podman run`:

```bash
podman run --rm --security-opt label=disable -v "$PWD":/config \
  --device /dev/ttyACM0 ghcr.io/esphome/esphome:2026.10.1 \
  run esp32c6-thread-router.yaml --device /dev/ttyACM0
```

</details>

---

<details>
<summary><b>🐳 Alternative: Using Docker/Podman</b></summary>

The included `docker-compose.yml` mounts this repository as `/config` and passes the USB device into the container.

> ⚠️ **Important:** The USB device must be plugged in **before** the container starts. When using Docker/Podman (rootless), fix USB permissions first (see above).

```bash
# 1. Start container (default device: /dev/ttyACM0)
docker compose up -d
# or with another device:
ESPHOME_DEVICE=/dev/ttyUSB0 docker compose up -d

# 2. Flash the firmware
docker compose exec esphome esphome run /config/esp32c6-thread-router.yaml --device=/dev/ttyACM0
```

**Update over the air (Thread)** - no USB needed, runs a one-off container without the USB device:
```bash
docker run --rm -it --network host -v "$PWD":/config ghcr.io/esphome/esphome:2026.10.1 run esp32c6-thread-router.yaml --device esp32c6-thread-router.local
```

</details>

---

<details>
<summary><b>🌐 Alternative: Web Dashboard (GUI)</b></summary>

**Docker Dashboard:**
```bash
docker compose up -d
```
The ESPHome container starts the dashboard automatically. Open **http://localhost:6052** in your browser.

> ⚠️ The dashboard has no password and listens on all interfaces (host networking). Only run it on a trusted network, or stop the container after flashing.

**Local Dashboard** (requires local ESPHome):
```bash
esphome dashboard .
```
Then open **http://localhost:6052** in your browser and use the web interface.

**Browser Flashing via [web.esphome.io](https://web.esphome.io/):**

web.esphome.io can only **flash** a finished firmware file. It cannot compile your YAML. Because the TLV is compiled into the firmware, you have to build it yourself first:

1. Compile locally: `esphome compile esp32c6-thread-router.yaml`
2. The firmware is written to `.esphome/build/esp32c6-thread-router/build/firmware.factory.bin`
3. Open **https://web.esphome.io/** in Chrome or Edge, click "Connect", choose your ESP32-C6 and select "Install"
4. Upload the `firmware.factory.bin` file

</details>

## ✅ Verification & Testing

### 🔍 Check if Thread Router is working:

#### 📋 Method 1: View Logs

```bash
esphome logs esp32c6-thread-router.yaml
```

Choose USB or "Over The Air" when prompted. You should see:
- IPv6 address like `fdxx:xxxx:xxxx:1:...` (routable address in the Thread network's OMR prefix)
- `[openthread:xxx] Device Type: FTD`
- No continuous error messages

---

#### 🌐 Method 2: Check Thread Network
```bash
# List all Thread routers in the network
docker exec otbr ot-ctl router table
# Your ESP32-C6 should appear with Extended MAC and good Link Quality

# Show network state
docker exec otbr ot-ctl state
```

> **💡 Note:** A new FTD first joins as a child (*REED*, Router Eligible End Device). Thread only promotes it to *router* when the mesh needs another router, which can take a few minutes, or not happen at all in a small network. It only shows up in the router table after that. You can see the current role in the **Thread Role** sensor in Home Assistant.

---

#### 🏠 Method 3: Home Assistant Integration
- Go to **Settings → Devices & Services**
- The ESP32-C6 should appear as a discovered device
- Add it to Home Assistant (use `esp32c6-thread-router.local` or IPv6 address)
- Home Assistant needs IPv6 and host networking (Docker) to reach Thread devices
- Enter the `api_encryption_key` from your `secrets.yaml` when asked
- Check device status - should show "Online"
- Re-flashed with a new key? Home Assistant asks for the new key. If the entry is stale, delete it and add the device again.

## 📊 Diagnostics in Home Assistant

The device exposes these diagnostic entities:

| Entity | Meaning |
|--------|---------|
| **Thread Role** | `router`, `child`, `leader`, `detached` or `disabled` |
| **Thread RLOC16** | Short address in the mesh. Routers end in `00` (e.g. `0x3400`) |
| **Thread Channel** / **Thread IP Address** | Current channel and IPv6 address |
| **Thread Parent RSSI** | Signal strength to the parent. Only meaningful while the device is a child, not as router |
| **Thread TX Retries** / **Thread TX CCA Errors** | Rising fast = poor link or busy channel (e.g. WiFi on the same frequencies) |
| **Thread Partition Changes** | Rising = the mesh keeps splitting, check placement/range |
| **Uptime** | Detects unexpected reboots |
| **External Antenna** (XIAO only) | Switch between built-in and external antenna |
| **Restart** | Restart the device |

More sensors (link quality, RX/TX totals, attach attempts, ...) are available in the [openthread_info](https://esphome.io/components/text_sensor/openthread_info/) component and can be added to the `sensor:` section.

## 📈 Watch Your Thread Mesh Over Time

The `ot-ctl` output and the diagnostic entities above show the mesh right now.
[Funkloch](https://github.com/firsttris/funkloch) (German for *dead spot*) asks your OpenThread
Border Router for its diagnostics and keeps the history, so you see what the new router changed:

- **Signal history**: the RSSI of every Thread device over days and weeks, not one snapshot.
- **Mesh map**: which routers are linked and how each device reaches the border router.
- **Placement check**: move the ESP32-C6, wait a day, compare. No more guessing.
- **Early warning**: a traffic light and trend per device, and a notification (ntfy, webhook,
  MQTT) when one drops out.

One Docker container next to your OpenThread Border Router (it reads the OTBR REST API, so
Apple, Google or Eve border routers won't do), read-only, no cloud:
[Thread setup](https://firsttris.github.io/funkloch/thread.html).

## ⚙️ Advanced Options

### 📶 Transmit Power

Since ESPHome 2026.3 the transmit power can be set. The ESP32-C6 supports -15 to 20 dBm:

```yaml
openthread:
  output_power: 10dBm
```

> ⚠️ Respect the regulatory limits for 2.4 GHz in your country. More power only helps if the other devices can also reach the router, since Thread links are bidirectional.

### 🔄 Thread Network Was Recreated / TLV Changed / Networks Merged

The router stores the Thread dataset in flash after the first start and then **ignores** the `tlv` from the config. If the dataset of your network changes, the router stays in the old network. This happens when you:

- recreate your Thread network
- let Home Assistant's OTBR join another network, e.g. the Google or Apple network (**most common cause**, see [Which network is the right one?](#-1-thread-network-dataset-tlv))
- switch the preferred network in Home Assistant

> ⚠️ **OTA over Thread no longer works after a network change.** The router stays in the old network, which no longer has a border router, so it has no reachable IPv6 address. You have to flash the new TLV **via USB**, just like the first installation.

**Symptoms:** the device is *unavailable* in Home Assistant, `ping` to its address fails with `Network is unreachable`, and the address prefix stored in Home Assistant no longer shows up in `ip -6 route`.

To check which network the router is in, compare the network name / Extended PAN ID of your `thread_tlv` with the one in **Settings → Devices & Services → Thread**.

To apply a new TLV:

1. Copy the current TLV (see [Retrieve TLV](#-retrieve-tlv-from-existing-thread-network)) into `thread_tlv` in `secrets.yaml`
2. Uncomment `force_dataset: true` in the `openthread:` section and flash via USB
3. Once the router has joined, comment it out again and flash once more (now via OTA)

> **💡 Why not keep `force_dataset: true` permanently?** The Thread network can change its dataset at runtime, e.g. when Home Assistant moves it to another channel. With `force_dataset: true` the router would fall back to the old dataset from the config after every reboot.

## ❓ FAQ

### Does it work with Matter devices?

Yes. Matter-over-Thread devices (Eve, Nanoleaf, IKEA, Aqara, ...) talk over the Thread mesh, and this router relays their traffic like any other Thread router. The router itself is not a Matter device: it shows up in Home Assistant through the ESPHome integration.

### Does it work with Apple, Google or Amazon border routers?

Yes, as long as the router gets the dataset (TLV) of the network your devices are in, see [Which network is the right one?](#-1-thread-network-dataset-tlv). For the Home Assistant connection and OTA updates the border routers have to route the Thread network into your LAN (IPv6).

### Can the ESP32-C6 be a Thread border router?

Not with this config. It is a router inside the mesh and needs a border router (e.g. OpenThread Border Router in Home Assistant, a HomePod, Nest Hub or Echo) to reach your LAN.

### Why is it a child and not a router?

A new FTD first joins as a child (REED). Thread promotes it to router only when the mesh needs one, see [Check Thread Network](#-method-2-check-thread-network). The **Thread Role** sensor shows the current role.

### ESP32-C6 or ESP32-H2?

Both work with ESPHome's OpenThread component, and so do the ESP32-C5 and nRF52 boards. This config is only tested on the ESP32-C6, see [Other boards](#-prerequisites).

### Can the same board run Thread and Zigbee at the same time?

No. The ESP32-C6 has one 802.15.4 radio, and a firmware uses it for either Thread or Zigbee. For Zigbee, flash the [ESP32-C6 Zigbee Router](https://github.com/firsttris/esp32c6-zigbee-router) instead.

## 🔀 Multiple Thread Routers

To flash multiple ESP32-C6 devices and use them as separate routers in the same Thread network, create a small file per additional device (e.g. `esp32c6-thread-router-2.yaml`) that reuses the main config:

```yaml
substitutions:
  name: esp32c6-thread-router-2            # Must be unique!
  friendly_name: ESP32-C6 Thread Router 2
  # antenna_default: RESTORE_DEFAULT_ON    # Optional: external antenna

packages:
  base: !include esp32c6-thread-router.yaml
```

Then flash it:
```bash
esphome run esp32c6-thread-router-2.yaml --device=/dev/ttyACM0
```

All devices share the same `secrets.yaml` (same TLV = same Thread network). Changes to the main config apply to all devices on the next flash. Each device will join the same Thread network and act as an independent router, extending your mesh coverage.

> **🔎 Which board is connected?** Check the MAC before flashing and compare it with *Device info* in Home Assistant: `esptool --port /dev/ttyACM0 read-mac`. On Linux it also works without port permissions: `cat /sys/class/tty/ttyACM0/device/../serial`

## 📂 File Structure

```
.
├── esp32c6-thread-router.yaml    # Main ESPHome configuration
├── esp32c6-thread-router-2.yaml  # Optional: second router you create (not in git)
├── secrets.yaml                  # Shared secrets you create (not in git)
├── docker-compose.yml            # Optional: ESPHome in Docker/Podman
├── requirements.txt              # The pinned ESPHome release (CI, pip)
├── .github/workflows/            # CI: validates and compiles the config
├── .gitignore                    # Excludes secrets and build artifacts
├── thread-router-image.jpg       # Image used in this README
├── LICENSE                       # MIT license
└── README.md                     # This file
```

## 📚 Further Resources

| Resource | Description |
|----------|-------------|
| 📖 [ESPHome OpenThread Documentation](https://esphome.io/components/openthread/) | Official ESPHome Thread component docs |
| 📊 [ESPHome OpenThread Info](https://esphome.io/components/text_sensor/openthread_info/) | Thread diagnostic sensors |
| 📡 [Seeed XIAO ESP32C6 Wiki](https://wiki.seeedstudio.com/xiao_esp32c6_getting_started/) | Pinout and RF switch of the XIAO ESP32C6 |
| 🔗 [OpenThread Primer](https://openthread.io/guides/thread-primer/) | Learn the basics of Thread networking |
| 🏠 [Home Assistant Thread Integration](https://www.home-assistant.io/integrations/thread/) | How Thread works in Home Assistant |
| 📈 [Funkloch](https://github.com/firsttris/funkloch) | Signal history, mesh map and alerts for Thread and Zigbee2MQTT devices |

---

<div align="center">

⭐ Like the ESP32-C6 Thread Router? A [star on GitHub](https://github.com/firsttris/esp32c6-thread-router) helps others find it.<br>
🐛 [Report a bug](https://github.com/firsttris/esp32c6-thread-router/issues/new) · 💡 [Request a feature](https://github.com/firsttris/esp32c6-thread-router/issues/new)

<sub>License: <a href="LICENSE">MIT</a> · © Tristan Teufel and contributors</sub>

</div>
