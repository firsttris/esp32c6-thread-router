# ESP32-C6 Thread Router Configuration

ESPHome configuration for an ESP32-C6 Thread Router that integrates with Home Assistant. This setup allows you to extend your Thread network coverage using an affordable ESP32-C6 board.

## Required Information

### 1. Thread Network Dataset (TLV)

**The TLV contains ALL network parameters** - you don't need to generate or enter anything manually!

#### Retrieve TLV from existing Thread network:

**Recommended method - via SSH/Terminal:**

1. **SSH** to your Home Assistant (or open Terminal Add-on)
2. **Retrieve TLV data:**
   ```bash
   # For Home Assistant OS (podman):
   podman exec otbr ot-ctl dataset active -x
   
   # For Home Assistant Container (docker):
   docker exec addon_core_openthread_border_router ot-ctl dataset active -x
   ```
3. **Copy** the hex string (e.g., `0e080000000000010000...`)
4. **Add** to `secrets.yaml`:
   ```yaml
   thread_tlv: "YOUR_TLV_HEX_HERE"
   ```

**That's it!** The TLV contains:
- ✅ Channel
- ✅ Network Name
- ✅ PAN ID & Extended PAN ID
- ✅ Network Key (encrypted)
- ✅ PSKc (encrypted)
- ✅ Mesh-Local Prefix
- ✅ All other parameters

### 2. ESPHome Secrets

In `secrets.yaml` you need:

- **api_encryption_key**: Auto-generated on first `esphome run`
- **ota_password**: Secure password for OTA updates
- **thread_tlv**: Your Thread network commissioning data (see above)

### 3. WiFi - Yes or No?

**For a dedicated Thread Router: WiFi NOT recommended**
- ❌ Thread and WiFi both use 2.4 GHz → possible interference
- ❌ Higher power consumption
- ✅ Thread router communicates via Thread network (IPv6)
- ✅ OTA updates work over Thread/API

**Only enable WiFi if:**
- You need dual-mode (Thread + WiFi simultaneously)
- Helpful for initial debugging

**Current config has WiFi commented out = optimal for Thread router!**

## Installation

1. **Configure secrets**: Edit `secrets.yaml` with your credentials
2. **Compile and flash**:
   ```bash
   esphome run esp32c6-thread-router.yaml
   ```

## Prerequisites

- ESP32-C6 board (e.g., ESP32-C6-DevKitM-1)
- Thread Border Router in network (e.g., Home Assistant with Thread integration)
- ESPHome installed
- USB cable for initial flashing

## Device Type: FTD vs MTD

- **FTD (Full Thread Device)**: Can act as router and connect other devices (recommended for routers)
- **MTD (Minimal Thread Device)**: End device, cannot route, more power efficient

## File Structure

```
.
├── esp32c6-thread-router.yaml  # Main ESPHome configuration
├── secrets.yaml                # Sensitive credentials (not in git)
├── .gitignore                  # Excludes secrets and build artifacts
└── README.md                   # This file
```

## Further Resources

- [ESPHome OpenThread Documentation](https://esphome.io/components/openthread/)
- [OpenThread Primer](https://openthread.io/guides/thread-primer/)
- [Home Assistant Thread Integration](https://www.home-assistant.io/integrations/thread/)
