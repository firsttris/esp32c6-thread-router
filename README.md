<div align="center">

# 🔗 ESP32-C6 Thread Router

**Extend Your Thread Network Coverage with an Affordable ESP32-C6!**

<img src="./thread-router-image.jpg" alt="ESP32-C6 Thread Router" width="600">

[![ESPHome](https://img.shields.io/badge/ESPHome-Compatible-blue?logo=esphome)](https://esphome.io/)
[![Thread](https://img.shields.io/badge/Thread-1.3-green)](https://www.threadgroup.org/)
[![Home Assistant](https://img.shields.io/badge/Home%20Assistant-Integration-41BDF5?logo=homeassistant)](https://www.home-assistant.io/)

This ESPHome configuration turns an ESP32-C6 board into a Thread FTD (Full Thread Device) router that seamlessly integrates with Home Assistant to expand your Thread mesh network and improve connectivity for your Thread devices.

---

</div>

## 📋 Prerequisites

| Requirement | Description |
|-------------|-------------|
| **🔧 Hardware** | ESP32-C6 board (tested with [Seeed Studio XIAO ESP32C6](https://www.seeedstudio.com/Seeed-Studio-XIAO-ESP32C6-p-5884.html)) |
| **🌐 Network** | Thread Border Router in network (e.g., Home Assistant with Thread integration) |
| **💻 Software** | ESPHome installed |
| **🔌 Cable** | USB cable for initial flashing |
| **📦 Optional** | 3D-printed case - [XIAO ESP32-C6 Case](https://www.printables.com/model/1543275-xiao-esp32-c6-zigbee-router-case-split-lid-sma-ext) |

## 🔑 Required Information

### 🌐 1. Thread Network Dataset (TLV)

> **💡 Important:** The TLV contains ALL network parameters - you don't need to generate or enter anything manually!

#### 📥 Retrieve TLV from existing Thread network:

**✅ Recommended method - via SSH/Terminal:**

**Step 1:** SSH to your Server

**Step 2:** Retrieve TLV data:
```bash
docker exec otbr ot-ctl dataset active -x
```

**Step 3:** Copy the hex string (e.g., `0e080000000000010000...`)

**Step 4:** Add to `secrets.yaml`:
```yaml
thread_tlv: "YOUR_TLV_HEX_HERE"
```

**🎉 That's it!** The TLV contains:

<table>
<tr><td>✅ Channel</td><td>✅ Network Name</td><td>✅ PAN ID & Extended PAN ID</td></tr>
<tr><td>✅ Network Key (encrypted)</td><td>✅ PSKc (encrypted)</td><td>✅ Mesh-Local Prefix</td></tr>
</table>

### 🔐 2. ESPHome Secrets

In `secrets.yaml` you need:

| Key | Description | Required |
|-----|-------------|----------|
| **thread_tlv** | Your Thread network commissioning data | ✅ Required |

## 🚀 Installation

### 📝 Step 1: Configure Secrets

Edit `secrets.yaml` with your Thread TLV:
```yaml
thread_tlv: "YOUR_TLV_HEX_HERE"
```

That's all you need - no WiFi credentials required!

### ⚡ Step 2: Compile and Flash Firmware

Connect your ESP32-C6 via USB and flash the firmware using local ESPHome:

```bash
esphome run esp32c6-thread-router.yaml --device=/dev/ttyACM0
```

> **💡 Note:** Replace `/dev/ttyACM0` with your device path (see troubleshooting below)

<details>
<summary><b>📚 Important Notes & Troubleshooting</b></summary>

### 📍 Device Paths by OS

| OS | Typical Paths |
|----|--------------|
| 🐧 Linux | `/dev/ttyUSB0`, `/dev/ttyACM0`, `/dev/ttyUSB1` |
| 🍎 macOS | `/dev/cu.usbserial-*`, `/dev/cu.wchusbserial*` |
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
```

> ⚠️ **Note:** You need to run `sudo chmod 666` each time you reconnect the USB device.

</details>

---

<details>
<summary><b>🐳 Alternative: Using Docker/Podman</b></summary>

> ⚠️ **Important:** When using Docker/Podman (rootless), you need to fix USB permissions before flashing.

```bash
# 1. Start container
docker-compose up -d

# 2. Flash the firmware
docker-compose exec esphome esphome run /config/Thread/esp32c6-thread-router.yaml --device=/dev/ttyACM0
```

> **📌 Note:** The docker-compose.yml mounts the parent `config/` directory to `/config` in the container. That's why you need `/config/Thread/` in the path above. This is necessary so ESPHome can find the build files in `.esphome/`.

</details>

---

<details>
<summary><b>🌐 Alternative: Web Dashboard (GUI)</b></summary>

Choose between local or hosted dashboard:

**Docker Dashboard:**
```bash
# Start container if not already running
docker-compose up -d

# Start the dashboard
docker-compose exec esphome esphome dashboard /config
```
Then open **http://localhost:6052** in your browser and navigate to the Thread folder.

**Local Dashboard** (requires local ESPHome):
```bash
esphome dashboard .
```
Then open **http://localhost:6052** in your browser and use the web interface.

**Hosted Dashboard** (no installation needed):
> **🎉 No installation needed!** Flash directly from your browser.

1. Visit **https://web.esphome.io/**
2. Click "Connect" and select your ESP32-C6 device
3. Upload your `esp32c6-thread-router.yaml` configuration file
4. Click "Install" to compile and flash

**Perfect for:** Users who prefer GUI over command line, or quick flashing without local ESPHome installation.

</details>

## ✅ Verification & Testing

### 🔍 Check if Thread Router is working:

#### 📋 Method 1: View Logs

> **🎉 Good news:** You can view logs over-the-air even with WiFi disabled!

**Logging Options:**

**🔌 Option A: Serial Connection (USB)**
- Always available - Connect via USB cable

**📡 Option B: Over Thread Network (OTA)**
- Works without WiFi! The device uses its Thread IPv6 address to connect.

```bash
esphome logs esp32c6-thread-router.yaml
```

Choose "Over The Air" option when prompted. You should see:
- IPv6 address like `fd3d:8f96:a13d:1:...` (Thread mesh-local address)
- `[openthread:xxx] Device Type: FTD`
- No continuous error messages

---

#### 🌐 Method 2: Check Thread Network
```bash
# List all Thread devices in network
podman exec otbr ot-ctl router table
# Your ESP32-C6 should appear with Extended MAC and good Link Quality

# Show network state
podman exec otbr ot-ctl state
```

---

#### 🏠 Method 3: Home Assistant Integration
- Go to **Settings → Devices & Services**
- The ESP32-C6 should appear as a discovered device
- Add it to Home Assistant (use `esp32c6-thread-router.local` or IPv6 address)
- Check device status - should show "Online"

##  Multiple Thread Routers

To flash multiple ESP32-C6 devices and use them as separate routers in the same Thread network:

### 1. Create Device-Specific Configuration Files

For each additional device, create a new YAML file (e.g., `esp32c6-thread-router-2.yaml`):

```yaml
esphome:
  name: esp32c6-thread-router-2          # Must be unique!
  friendly_name: ESP32-C6 Thread Router 2  # Optional but helpful

openthread:
  device_type: FTD
  tlv: !secret thread_tlv  # Same TLV = same Thread network
```
### 2. Flash Additional Devices

Each device will join the same Thread network and act as an independent router, extending your mesh coverage.

## 📂 File Structure

```
.
├── esp32c6-thread-router.yaml    # Main ESPHome configuration
├── esp32c6-thread-router-2.yaml  # Optional: Second router
├── esp32c6-thread-router-3.yaml  # Optional: Third router
├── secrets.yaml                  # Shared credentials (not in git)
├── .gitignore                    # Excludes secrets and build artifacts
└── README.md                     # This file
```

## 📚 Further Resources

| Resource | Description |
|----------|-------------|
| 📖 [ESPHome OpenThread Documentation](https://esphome.io/components/openthread/) | Official ESPHome Thread component docs |
| 🔗 [OpenThread Primer](https://openthread.io/guides/thread-primer/) | Learn the basics of Thread networking |
| 🏠 [Home Assistant Thread Integration](https://www.home-assistant.io/integrations/thread/) | How Thread works in Home Assistant |

---

<div align="center">

**Made by the open source community**

⭐ Star us on [GitHub](https://github.com/firsttris/esp32c6-thread-router) • 🐛 [Report a Bug](https://github.com/firsttris/esp32c6-thread-router/issues) • 💡 [Request a Feature](https://github.com/firsttris/esp32c6-thread-router/issues)

</div>

