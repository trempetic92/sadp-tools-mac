# SADP Tools for macOS

A native macOS utility for discovering, activating, configuring, testing, and viewing Hikvision devices on the local network.

The application is built with **Flutter** and a native macOS bridge to the **Hikvision SADP SDK**. It adds direct ISAPI management, native RTSP live view, snapshots, password-reset workflows, non-destructive security checks, and a visual smart-event editor.

> **Platform:** macOS  
> **Device communication:** Local LAN  
> **Primary protocols:** SADP, ISAPI, RTSP, HTTP Digest/Basic authentication

---

## Features

### SADP device discovery

- Automatically discovers compatible Hikvision devices on the local network.
- Receives live device updates from the native SADP SDK.
- Refreshes or stops discovery without restarting the application.
- Displays detailed device information, including:
  - Activation state
  - IP address
  - Serial number
  - Device model
  - MAC address
  - Firmware/software version
  - Subnet mask and gateway
  - SDK, HTTP, HTTPS, and SDK-over-TLS ports
  - DHCP state
  - Manufacturer
  - SADP state
  - Last-seen time
- Search across device details.
- Sort devices by table column.
- Show, hide, and resize table columns.

### Device activation

- Activates previously inactive devices.
- Validates the new administrator password before submitting it.
- Refreshes the device list after activation.

### Network configuration

- Changes device network parameters directly through SADP.
- Supports:
  - Static IP configuration
  - DHCP
  - IP address
  - Subnet mask
  - Default gateway
  - HTTP port
  - SDK port
- Displays SADP retry and lock information when a configuration request fails.

### Secure device credentials

- Remember credentials per device.
- Reuse stored credentials for live view, snapshots, and network changes.
- Forget stored credentials from the device context menu.
- Credentials are stored locally using `flutter_secure_storage`, which uses the macOS Keychain.

### Native RTSP live view

- Plays camera and recorder streams through a native macOS RTSP platform view.
- Reads available streaming channels through ISAPI.
- Lets the user select the required camera/channel and stream.
- Supports stream restart and stop controls.
- Keeps credentials out of the visible RTSP status text.
- Supports camera video verification/encryption codes where required by the device.

### Snapshot capture

- Reads available streaming channels from the device.
- Captures JPEG/PNG snapshots through ISAPI.
- Shows the captured image in the application.
- Saves snapshots to the user's Downloads folder.

### Smart-event editor

The live-view screen includes a visual smart-event editor. Event types are discovered from the selected camera instead of being shown unconditionally.

Depending on the camera model and firmware, the editor can expose:

- Intrusion detection
- Line crossing
- Region entrance
- Region exit
- Loitering
- People gathering
- Rapid movement
- Parking detection
- Unattended baggage
- Object removal

Available controls are capability-driven and may include:

- Global event enable/disable
- Per-rule enable/disable
- Multiple event rules
- Polygon-region drawing
- Line-crossing drawing
- Draggable region points and line endpoints
- Crossing direction
- Human filtering
- Vehicle filtering
- Other camera-provided target classes
- Sensitivity
- Time threshold
- Object occupation percentage
- Minimum accepted target-size box
- Maximum accepted target-size box

Minimum and maximum target sizes are configured visually by drawing and resizing boxes over the camera image. Their normalized dimensions are shown as read-only information; manual pixel entry is intentionally not used.

The editor maps the displayed video area to the camera's normalized coordinate system and only displays event types and options that the camera exposes.

### Password reset using XML files

- Exports the encrypted SADP password-reset request XML.
- Lets the user choose the response XML returned by Hikvision or an authorized support provider.
- Submits the matching response XML together with a new password.
- Reports native SADP errors and device lock/retry information.

The response XML must match the exact request XML exported for that device.

### Non-destructive security checks

The application performs limited, non-destructive checks on discovered private or link-local devices:

- **CVE-2017-7921** — tests known snapshot endpoints and only marks the device vulnerable when unauthenticated image data is actually returned.
- **CVE-2021-36260** — performs a conservative firmware/build-date risk review without executing command-injection payloads.

The checks:

- Run only against private, loopback, or link-local IPv4 addresses.
- Do not extract credentials.
- Do not execute commands on the device.
- Do not save images returned by the security probe.
- Run with a limited parallel worker count.

These checks are an initial risk indicator, not a replacement for a professional security assessment or the vendor's model-specific firmware advisory.

### ISAPI authentication and compatibility

- Supports HTTP Digest authentication.
- Supports HTTP Basic authentication when offered by the device.
- Discovers streaming channels and smart-event capabilities dynamically.
- Reuses the configuration endpoint accepted by the device for later updates.
- Handles model and firmware differences by hiding unsupported controls.

### Local WebSDK proxy

The project also includes a localhost HTTP/WebSocket proxy used for Hikvision WebSDK compatibility and diagnostics. It can forward WebSDK HTTP and WebSocket traffic to the selected device and answer supported HTTP authentication challenges with the credentials supplied by the user.

---

## Typical workflow

1. Start the application and wait for SADP discovery.
2. Select a device from the table.
3. Activate it if it is inactive.
4. Configure DHCP or static network parameters.
5. Remember the administrator credentials when needed.
6. Open **Live view** or capture a **Snapshot** from the device context menu.
7. Open the smart-event editor from Live View to configure supported analytics.
8. Use **Forgot password** to export/import SADP reset XML files when required.

---

## Project structure

```text
lib/
├── main.dart
├── isapi/
│   ├── hikvision_isapi_service.dart
│   └── hikvision_smart_event_service.dart
├── pentest/
│   └── hikvision_pentest_service.dart
├── rtsp/
│   └── sadp_rtsp_live_view.dart
├── storage/
│   └── device_credentials_store.dart
└── websdk/
    └── sadp_websdk_proxy_server.dart

macos/
└── Runner/
    └── Native SADP SDK and RTSP platform-view integration
```

---

## Requirements

- A Mac with a supported macOS version for the selected Flutter release.
- Flutter SDK with macOS desktop support enabled.
- Xcode and the macOS command-line tools.
- Hikvision SADP SDK headers and native libraries integrated into the macOS runner.
- Required RTSP/native player dependencies configured in the macOS project.
- The Mac and target devices must be reachable on the same local network for SADP discovery.

Check that the macOS target has the network permissions and entitlements required by your deployment configuration. SADP discovery may require local-network or multicast access.

---


For distribution outside the development Mac, configure Apple code signing, hardened runtime, notarization, and any required native-library embedding before publishing the application.

---

## Known limitations

- This project targets macOS; the native SADP and RTSP integrations are not automatically portable to Windows, Linux, iOS, Android, or Web.
- Smart-event XML structures and endpoints differ between Hikvision model families and firmware versions.
- Only event types and controls successfully exposed by the selected camera are displayed.
- Some cameras expose the main smart-event rule but not a writable target-size calibration endpoint.
- RTSP playback depends on the codec and capabilities supported by the native macOS player implementation.
- Encrypted video may require the device's video verification code in addition to administrator credentials.
- Password-reset response files must be obtained through an authorized Hikvision/support process.

---


## Disclaimer

This is an independent utility and is not an official Hikvision product. Hikvision, SADP, ISAPI, and related names and trademarks belong to their respective owners.

The software is provided without any guarantee that every device model or firmware version will be compatible. Test configuration changes on a non-critical device before deploying them in production.
