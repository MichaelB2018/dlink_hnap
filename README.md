[![Validate with hassfest](https://github.com/MichaelB2018/dlink_hnap/actions/workflows/hassfest.yaml/badge.svg)](https://github.com/MichaelB2018/dlink_hnap/actions/workflows/hassfest.yaml)
[![HACS Validation](https://github.com/MichaelB2018/dlink_hnap/actions/workflows/validate.yaml/badge.svg)](https://github.com/MichaelB2018/dlink_hnap/actions/workflows/validate.yaml)
[![hacs_badge](https://img.shields.io/badge/HACS-Custom-orange.svg)](https://hacs.xyz/)
[![Code style: black](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/ambv/black)

# D-Link HNAP

A [Home Assistant](https://www.home-assistant.io/) custom integration for D-Link sensors using the HNAP protocol.

This repository is a community-maintained fork of the original integration. Upstream improvements have not been merged, so releases are published here for users who want the maintained version.

**Key features:**

- **100% local** — communicates directly with devices on your network, no cloud or internet required
- **UI configurable** — full config flow with step-by-step setup from the Home Assistant UI
- **Auto-discovery** — automatically finds D-Link devices on your network via SSDP
- **Automatic capability detection** — probes the device and creates only the entities it supports
- **Configurable options** — adjust polling interval and motion timeout from the UI without restarting
- **Device info** — registers devices with manufacturer, model, firmware version, and hardware version
- **Diagnostics** — built-in diagnostics download with automatic redaction of sensitive data
- **YAML migration** — legacy YAML configurations are automatically imported as config entries

## Supported Devices

| Device | Model | Capabilities |
| ------ | ----- | ------------ |
| Motion Sensor | DCH-S150 | Motion detection |
| Water Leakage Sensor | DCH-S160, DCH-S161 | Water detection, Temperature |

Other D-Link HNAP-compatible devices may also work. The integration automatically detects which capabilities a device supports by querying its available SOAP actions.

## Entities

The integration auto-creates entities based on device capabilities:

| Entity | Type | Device Class | Description |
| ------ | ---- | ------------ | ----------- |
| Water Detected | Binary Sensor | `moisture` | `on` when water is detected. Only created for devices with water detection capability. |
| Motion Detected | Binary Sensor | `motion` | `on` when motion is detected, turns `off` after the configured motion timeout. Only created for devices with motion detection capability. |
| Temperature | Sensor | `temperature` | Current temperature reading in °C. Only created for devices that report temperature. |
| Firmware Version | Sensor | *(diagnostic)* | Reports the device firmware version. Always created for all devices. |

All entities are associated with a **device** in the Home Assistant device registry, showing manufacturer (D-Link), model, firmware version, and hardware version.

## Installation

**Requirements:** Home Assistant **2024.11.0** or newer.

Pick the method that matches your Home Assistant version:

| Your Home Assistant version | Recommended method |
| --------------------------- | ------------------ |
| **2026.11 or newer** | [Option 1: Home Assistant Marketplace](#option-1-home-assistant-marketplace-202611-and-newer) (built in, nothing extra to install) |
| **2024.11 – 2026.10** with HACS installed | [Option 2: HACS](#option-2-hacs-before-202611) |
| Any supported version, without HACS or Marketplace | [Option 3: Manual installation](#option-3-manual-installation) |

> **Switching from the original `postlund/dlink_hnap`?** Both use the same `dlink_hnap` domain, so only one can be installed at a time. Uninstall the original first (in HACS/Marketplace, or by deleting `config/custom_components/dlink_hnap`), then install this version using one of the options below and restart Home Assistant. Your existing YAML configuration is [migrated automatically](#legacy-yaml-configuration-deprecated).

### Option 1: Home Assistant Marketplace (2026.11 and newer)

Starting with Home Assistant 2026.11, HACS is built into Home Assistant as the **Marketplace**. The quickest way is to click this button:

[![Open your Home Assistant instance and open this repository in the Marketplace.](https://my.home-assistant.io/badges/marketplace_repository.svg)](https://my.home-assistant.io/redirect/marketplace_repository/?owner=MichaelB2018&repository=dlink_hnap&category=integration)

Or install it step by step:

1. In Home Assistant, go to **Settings** → **Marketplace**.
2. The first time you open the Marketplace, read the notice and select **I understand the risks and agree**.
3. Search for **D-Link HNAP**.
   - **Not listed?** Select **Add from link** in the upper-right corner, enter `MichaelB2018/dlink_hnap` (or `https://github.com/MichaelB2018/dlink_hnap`), and select **Add**. If the Marketplace asks for the type, choose **Integration** and select **Add** again.
   - Adding a repository from a link requires a one-time GitHub connection. The Marketplace uses GitHub device sign-in, so you approve it on GitHub without entering your password in Home Assistant.
4. Open the **D-Link HNAP** page and select **Install**.
5. You land straight in the setup flow — **no restart needed** for a new installation. Continue with [Configuration](#configuration).

**Updates:** new releases appear under **Settings** → **Updates**, next to Home Assistant's own updates. After updating, a repair notification tells you when a restart is needed and can restart Home Assistant for you.

> **Already installed via HACS?** There is nothing to do. When you upgrade to 2026.11, Home Assistant migrates HACS to the Marketplace automatically, and D-Link HNAP stays installed and keeps receiving updates.

### Option 2: HACS (before 2026.11)

If you run Home Assistant older than 2026.11 and have [HACS](https://hacs.xyz/docs/use/) installed, click this button to open the repository directly in HACS:

[![Open your Home Assistant instance and open this repository in HACS.](https://my.home-assistant.io/badges/hacs_repository.svg)](https://my.home-assistant.io/redirect/hacs_repository/?owner=MichaelB2018&repository=dlink_hnap&category=integration)

Or install it step by step:

1. Open **HACS** from the Home Assistant sidebar.
2. Select the three-dot menu (⋮) in the upper-right corner → **Custom repositories**.
3. Enter `https://github.com/MichaelB2018/dlink_hnap` as the **Repository**, choose **Integration** as the **Type**, and select **Add**. Close the dialog.
4. Search for **D-Link HNAP**, open it, and select **Download** (bottom right). Confirm the latest version and select **Download** again.
5. Restart Home Assistant: **Settings** → **System** → three-dot menu (⋮) → **Restart Home Assistant**.
6. Continue with [Configuration](#configuration).

**Updates:** HACS shows an update entity for D-Link HNAP under **Settings** → **Updates** when a new release is available. Install it and restart Home Assistant.

### Option 3: Manual installation

Use this if you don't use the Marketplace or HACS. You'll need access to your Home Assistant configuration folder, for example via the Samba share, SSH, or the Studio Code Server / File editor app.

1. Download the latest release from the [releases page](https://github.com/MichaelB2018/dlink_hnap/releases/latest) — under **Assets**, choose **Source code (zip)**.
2. Extract the archive.
3. Copy the `custom_components/dlink_hnap` folder into the `custom_components` folder of your Home Assistant configuration directory (the folder that contains `configuration.yaml`). Create `custom_components` if it doesn't exist yet. The result should look like this:

   ```text
   config/
   ├── configuration.yaml
   └── custom_components/
       └── dlink_hnap/
           ├── __init__.py
           ├── manifest.json
           ├── brand/
           └── ...
   ```

4. Restart Home Assistant: **Settings** → **System** → three-dot menu (⋮) → **Restart Home Assistant**.
5. Continue with [Configuration](#configuration).

**Updates:** manual installations don't get update notifications. To update, watch the [releases page](https://github.com/MichaelB2018/dlink_hnap/releases), replace the `custom_components/dlink_hnap` folder with the new version, and restart Home Assistant.

## Configuration

### UI Setup (recommended)

If you installed through the Marketplace, the setup flow opens automatically after you select **Install**. Otherwise, click this button:

[![Open your Home Assistant instance and start setting up D-Link HNAP.](https://my.home-assistant.io/badges/config_flow_start.svg)](https://my.home-assistant.io/redirect/config_flow_start/?domain=dlink_hnap)

Or set it up manually:

1. In Home Assistant, go to **Settings** → **Devices & Services**
2. Click **+ Add Integration** (bottom right)
3. Search for **D-Link HNAP**
4. Fill in the connection details:

| Field | Description |
| ----- | ----------- |
| **Host** | IP address or hostname of the D-Link device (e.g. `192.168.1.50`) |
| **Username** | Authentication username — almost always `Admin` (pre-filled) |
| **Password** | The PIN code printed on the device label |

5. Click **Submit** — the integration will connect to the device, verify credentials, and detect capabilities
6. Entities are created automatically based on what the device supports

> **Tip:** The PIN code is typically printed on a label on the back or bottom of the device.

### SSDP Auto-Discovery

If a D-Link device is on your local network, Home Assistant will automatically discover it via SSDP. When discovered:

1. A notification appears: *"D-Link device discovered"*
2. Click the notification or go to **Settings** → **Devices & Services**
3. You'll see the discovered device — click **Configure**
4. Enter the **Username** and **PIN code** (password)
5. Click **Submit** to complete setup

### Options

After a device is configured, you can adjust settings without restarting:

1. Go to **Settings** → **Devices & Services**
2. Find the **D-Link HNAP** integration and click **Configure**
3. Adjust the options:

| Option | Default | Range | Description |
| ------ | ------- | ----- | ----------- |
| **Update interval** | 30 seconds | 5–300 seconds | How frequently the device is polled for new data. Lower values give faster updates but increase network traffic. |
| **Motion timeout** | 30 seconds | 5–600 seconds | How long after the last detected motion before the motion sensor entity turns `off`. Only relevant for motion-capable devices. |

4. Click **Submit** — changes take effect immediately (the integration reloads automatically)

### Re-authentication

If the device stops accepting the stored PIN code (for example after a factory reset), Home Assistant shows **Authentication expired** for the D-Link device under **Settings** → **Devices & Services**. Select **Reconfigure**, enter the current PIN code from the device label, and select **Submit**. The integration verifies the PIN, saves it, and reloads, without needing to delete and re-add the device.

## Legacy YAML Configuration *(deprecated)*

> ⚠️ **YAML configuration is deprecated and will be removed in a future version.**
> Existing YAML configurations are **automatically migrated** to UI config entries on the next restart.
> After migration, you can safely remove the `dlink_hnap` entries from your `configuration.yaml`.

The legacy YAML format is shown below for reference only:

```yaml
binary_sensor:
  - platform: dlink_hnap
    name: Kitchen Motion
    host: 10.0.0.10
    type: motion
    username: Admin
    password: 123456
    timeout: 35
  - platform: dlink_hnap
    name: Kitchen Leakage
    host: 10.0.0.11
    type: water
    username: Admin
    password: 123456
```

<details>
<summary>Legacy YAML configuration options</summary>

| Key | Required | Type | Default | Description |
| --- | -------- | ---- | ------- | ----------- |
| `name` | No | string | `D-Link Motion Sensor` | Name for the sensor |
| `type` | **Yes** | string | | Sensor type: `motion` or `water` |
| `host` | **Yes** | string | | IP address of the device |
| `username` | No | string | `Admin` | Username for authentication |
| `password` | **Yes** | string | | PIN code printed on the device |
| `timeout` | No | int | `35` | Seconds before motion sensor turns `off` after last trigger (motion only) |

</details>

## Diagnostics

This integration supports Home Assistant's built-in diagnostics feature:

1. Go to **Settings** → **Devices & Services** → **D-Link HNAP**
2. Click on the device
3. Click the three-dot menu (⋮) → **Download diagnostics**

The diagnostics file includes:
- Configuration entry data (with password redacted)
- Configured options (scan interval, motion timeout)
- Detected device capabilities
- Current device data (with serial number redacted)

## Troubleshooting

| Problem | Solution |
| ------- | -------- |
| **"Failed to connect"** during setup | Verify the IP address is correct and the device is powered on and reachable. Try pinging the device. |
| **"Invalid username or password"** | The password is the numeric PIN code on the device label, not a user-chosen password. Username is almost always `Admin`. |
| **"Authentication expired"** | The device no longer accepts the stored PIN. Follow the [re-authentication](#re-authentication) steps to enter the current PIN. |
| **Entities not appearing** | The integration only creates entities for capabilities the device reports. Check diagnostics to see detected capabilities. |
| **Motion sensor stuck on** | Increase the motion timeout in Options. The sensor turns `off` only after no motion is detected for the configured timeout period. |
| **Stale data / slow updates** | Decrease the scan interval in Options (minimum 5 seconds). Note that very low values increase network traffic. |
| **Device becomes unavailable** | The device may have changed IP address. Delete and re-add the integration with the new IP, or assign a static IP/DHCP reservation. |

## How It Works

This integration communicates with D-Link devices using the **HNAP** (Home Network Administration Protocol) over SOAP/HTTP. The protocol flow is:

1. **Authentication** — A challenge-response handshake using HMAC-MD5, establishing a session cookie
2. **Capability detection** — Queries the device's supported SOAP actions to determine which sensors are available
3. **Polling** — Periodically fetches sensor data (water state, motion events, temperature) based on detected capabilities

All communication happens locally on your network. No data is sent to the cloud.

## Resolved Issues

Version 2.0.0 addresses the following open issues:

| Issue | Description | How it's resolved |
| ----- | ----------- | ----------------- |
| [Upstream #18](https://github.com/postlund/dlink_hnap/issues/18) | 2025.1.b5 failed to load sensor | Removed deprecated `DEVICE_CLASS_MOTION` / `DEVICE_CLASS_MOISTURE` imports; fully rewritten with modern `BinarySensorDeviceClass` enums |
| [Upstream #21](https://github.com/postlund/dlink_hnap/issues/21) | DLink HNAP issues with HA 2025.2.x | Complete rewrite resolves all compatibility issues with modern Home Assistant |
| [Upstream #12](https://github.com/postlund/dlink_hnap/issues/12) | Excessive errors when device is in power-saving mode | `DataUpdateCoordinator` rate-limits error logging; proper `CannotConnect` → `UpdateFailed` handling avoids log spam; configurable scan interval reduces poll frequency |
| [Upstream #17](https://github.com/postlund/dlink_hnap/issues/17) | No device health / offline indication | `DataUpdateCoordinator` automatically marks entities as **unavailable** when the device can't be reached — standard HA device health pattern |
| [Upstream #20](https://github.com/postlund/dlink_hnap/issues/20) | DCH-S162 not working | Automatic capability detection probes each device's SOAP actions, so any HNAP-compatible device should work. *(Note: The DCH-S162 may use a different protocol — cannot verify without hardware)* |

Not yet addressed: [Upstream #11](https://github.com/postlund/dlink_hnap/issues/11) (Battery attribute) — The SOAP action name for battery state is unknown. Contributions from users with battery-powered devices (DCH-S161) are welcome.

## Credits

- **[Pierre Ståhl](https://github.com/postlund)** — Original author of the integration and HNAP protocol implementation
- **[Kyle Cackett](https://github.com/kyle-cackett)** — Bug fixes and compatibility updates
- **[Roger Selwyn](https://github.com/RogerSelwyn)** — Contributions
- **[MichaelB2018](https://github.com/MichaelB2018)** — Maintained fork releases, v2.0.0 architecture update, config flow, SSDP discovery, coordinator, options flow, temperature sensor, diagnostics, and modern HA architecture

## License

This project is licensed under the [MIT License](LICENSE).
