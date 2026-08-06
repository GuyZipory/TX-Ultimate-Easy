# TX Ultimate Easy

[![Version][version-shield]](https://github.com/GuyZipory/TX-Ultimate-Easy/tags)
[![License][license-shield]](LICENSE)
[![ESPHome][esphome-shield]](https://esphome.io/)

<!-- markdownlint-disable MD013 MD033 -->
| &nbsp;<picture><source media="(prefers-color-scheme: dark)" srcset="Assets/Logo_dark.png"><source media="(prefers-color-scheme: light)" srcset="Assets/Logo_light.png"><img alt="TX Ultimate Easy Logo" src="Assets/Logo_light.png"></picture> | ESPHome firmware for the **Sonoff TX Ultimate** touch wall switch. Flash it once over serial, then configure the relays, LED ring, gestures and feedback from the Home Assistant UI — no YAML editing after setup. |
| --- | :-- |
<!-- markdownlint-enable MD013 MD033 -->

[version-shield]: https://img.shields.io/github/v/tag/GuyZipory/TX-Ultimate-Easy?label=version
[license-shield]: https://img.shields.io/github/license/GuyZipory/TX-Ultimate-Easy
[esphome-shield]: https://img.shields.io/badge/powered%20by-ESPHome-blue

> **This is a fork** of [edwardtfn/TX-Ultimate-Easy](https://github.com/edwardtfn/TX-Ultimate-Easy)
> by Edward Firmo, maintained by [GuyZipory](https://github.com/GuyZipory). It tracks upstream and
> adds a reworked touch path, cleaned-up LED and relay entities, and defaults tuned for a switch
> that responds on the first tap. See [What's different in this fork](#whats-different-in-this-fork).

## What's different in this fork

Upstream's firmware, with the touch path made reliable and one control per concept instead of
several that fight each other.

<!-- markdownlint-disable MD013 -->
| Area | What changed |
| --- | --- |
| **Defaults** | Ships as a fast single-tap switch: the relay moves the instant you lift your finger, and the gesture recognition that used to swallow taps is off. See [Factory defaults](#factory-defaults). |
| **Relays** | `relay_N_mode` picks `switch`, `light` or `disabled` per relay. `light` gives you a real `light.` entity from the firmware, so Home Assistant's *show as Light* helper is no longer needed. See [Relays](#relays). |
| **LED indicators** | One entity per job. `Relay N indicator` sets the colour **and** brightness (the brightness slider used to do nothing). `Relay N indicator area` picks which part of the arc lights up. `LED ring` owns the whole strip while it is on, instead of fighting the indicators. See [LED indicators](#led-indicators). |
| **Night Mode** | The colour picker is `Night Mode - LED`, filed under *Configuration*. See [Night Mode](#night-mode). |
| **Advanced tuning** | Two optional substitutions free up IRAM on newer ESP32 silicon. See [Advanced tuning](#advanced-tuning). |
<!-- markdownlint-enable MD013 -->

### Reliability fixes

- Boot handlers were being silently dropped by a package-merge quirk, so the device never
  finished booting. The whole sequence — NVS restore, initialisation, boot animation — now runs.
- Taps that failed to toggle the relay are fixed, as is a stuck gesture flag that could block
  toggling entirely until the next reboot.
- Relay state is flushed to NVS on every toggle, so it survives a power cut instead of losing up
  to a minute of changes.
- A single tap no longer produces doubled vibration and click feedback.
- The `firmware` field in Home Assistant events was always empty; it now carries the version.

Everything else tracks upstream.

## Contents

- [Key features](#key-features)
- [Hardware support](#hardware-support)
- [Requirements](#requirements)
- [Installation](#installation)
- [Configuring the device](#configuring-the-device)
- [Contributing](#contributing)
- [Acknowledgments](#acknowledgments)
- [License](#license)

## Key features

- **Touch panel** — clicks, long-presses, swipes and multi-touch, exposed to Home Assistant as
  events and sensors.
- **Relays** — 1 to 4 gangs, each independently a switch, a light, or disabled.
- **LED ring** — per-relay status indicators with configurable colour, brightness and arc, plus
  manual control with an effects library.
- **Night Mode** — locks the LEDs to a fixed colour and brightness and can silence vibration and
  sound. Survives reboots.
- **Haptic and audio feedback** — vibration motor and built-in speaker, both configurable.
- **Bluetooth proxy** — compatible with ESPHome's `bluetooth_proxy` component.
- **ESP-NOW** — optional peer-to-peer control so one switch can toggle relays on another without
  Home Assistant. See the [ESP-NOW docs](docs/espnow.md).
- **UI configuration** — everything above is adjustable from Home Assistant. Only the device
  format and gang count live in YAML, because they are compile-time.

## Hardware support

All Sonoff TX Ultimate variants:

- **EU** format (square, T5-xC-86)
- **US** format (rectangle, T5-xC-120)
- 1, 2, 3 and 4 gang

## Requirements

- A Sonoff TX Ultimate device
- Home Assistant
- ESPHome **2025.11.0 or later** — older versions will not compile
- A USB-to-UART adapter for the first flash (see [Flash the firmware](#flash-the-firmware))

The firmware builds on **ESP-IDF**, ESPHome's default framework for the ESP32. Arduino is not
tested and not supported here.

## Installation

### Set up the device

In the ESPHome dashboard, click **+ New Device**, name it, choose **ESP32**, then replace the
generated configuration with this:

```yaml
substitutions:
  name: tx-ultimate-easy           # Must be unique per device
  friendly_name: TX Ultimate Easy  # Must be unique per device
  device_format: EU                # 'EU' or 'US' — uppercase only
  gang_count: 1                    # Number of relays/buttons: 1, 2, 3 or 4

# Not included in the packages below — add whichever you want
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
# web_server:
# captive_portal:

packages:
  remote_package:
    url: https://github.com/GuyZipory/TX-Ultimate-Easy
    ref: main  # Or pin a tag for controlled updates, e.g. ref: v2026.6.1
    refresh: 5min
    files:
      - ESPHome/TX-Ultimate-Easy-ESPHome_core.yaml      # Essential
      - ESPHome/TX-Ultimate-Easy-ESPHome_standard.yaml  # Recommended
```

Then click **Save** → **Install**.

The substitutions:

| Substitution | Required | Values | Notes |
| --- | --- | --- | --- |
| `name` | Yes | slug | Must be unique per device |
| `friendly_name` | Yes | text | Must be unique per device |
| `device_format` | Yes | `EU` or `US` | Case-sensitive, uppercase only |
| `gang_count` | Yes | `1`–`4` | Must be an integer, not a quoted string |
| `expose_relays_leds_to_ha` | No | `true` / `false` | Defaults to `false` |

`device_format` and `gang_count` are validated at compile time and need a rebuild to change.
Everything else is adjustable from the Home Assistant UI.

- [Full list of released versions](https://github.com/GuyZipory/TX-Ultimate-Easy/tags)
- [Latest version of this YAML](TX-Ultimate-Easy-ESPHome.yaml)

### Flash the firmware

The first flash must be done over serial. [ESPHome Web](https://web.esphome.io) is the simplest
way to do it. After that, all updates go over the air from the ESPHome dashboard.

<!-- markdownlint-disable MD028 -->
> [!IMPORTANT]
> **SAFETY WARNINGS**
>
> - ALWAYS disconnect the device from mains power before opening
> - NEVER work on the device while connected to mains power
> - Ensure the device is completely powered off before making any connections
> - Keep your workspace clean, dry, and static-free
> - Use proper insulated tools when working with electronics
> - If unsure about any step, seek help from an experienced person

> [!CAUTION]
> ⚡ **CRITICAL: VOLTAGE WARNING**
> Using a UART adapter with voltage higher than 3.3V WILL DAMAGE YOUR DEVICE.
> Double-check your adapter's voltage before connecting - many common FTDI adapters
> default to 5V which will permanently damage the ESP32 chip.
<!-- markdownlint-enable MD028 -->

<!-- markdownlint-disable MD033 -->
<details>
<summary><b>Hardware needed and step-by-step flashing process</b></summary>

#### Required hardware

- USB-to-UART adapter:
  - 3.3V logic level ONLY (DO NOT use 5V adapters)
  - Must be capable of supplying at least 500mA
  - Common adapters: CP2102, CH340, FTDI (ensure 3.3V setting)
- Small Phillips screwdriver
- 5 wires for connections (including one for BOOT to GND)

#### Flashing process

1. Open your TX Ultimate device carefully.
2. Locate the programming header pins.
3. Connect your USB-to-UART adapter:

   | Adapter | Device |
   |---------|--------|
   | GND     | GND    |
   | 3.3V    | 3.3V   |
   | TX      | RX     |
   | RX      | TX     |

4. Put the device in flash mode:
   - Temporarily connect the BOOT pin to GND using a jumper wire
   - While holding BOOT to GND, power up the device
   - After the device powers up (wait a couple of seconds), remove the BOOT to GND connection
5. Visit [ESPHome Web](https://web.esphome.io).
6. Connect to your device and flash the firmware.
7. After a successful flash the device restarts and is ready for OTA updates.

#### Video walkthroughs

<!-- markdownlint-disable MD013 -->
These cover the physical disassembly, which is the fiddly part:

- 🇬🇧 [Let's Automate](https://youtu.be/fIOgWQXndhI?si=O3j7sjwn7PvH1vxn) - English video tutorial
- 🇪🇸 [Un loco y su tecnología](https://youtu.be/58v8oqSQgXQ?t=143) - Spanish video tutorial
- 🇩🇪 [SmartHome yourself](https://youtu.be/naDLhX89enQ?t=465) - German video tutorial
<!-- markdownlint-enable MD013 -->

They use different firmware, but the physical flashing process is the same.

</details>
<!-- markdownlint-enable MD033 -->

### Add it to Home Assistant

Once the device is flashed and on the same network, Home Assistant discovers it within a minute or
two. Accept the discovery notification and it appears under **Devices & Services**.

<!-- markdownlint-disable MD033 -->
<details>
<summary><b>If the device isn't discovered</b></summary>

- Check it is actually on the network — look for it in your router's client list, and consider
  giving it a [manual IP](https://esphome.io/components/wifi.html#manual-ips).
- Add it by hand: **Settings → Devices & Services → Add Integration → ESPHome**, then enter the
  device's IP address.
- Confirm your network allows mDNS, and that no VLAN or client isolation is in the way.

</details>
<!-- markdownlint-enable MD033 -->

### Optional packages

Add these to the `files:` list alongside the core and standard packages.

| Package | What it adds |
| --- | --- |
| `_addon_ble_proxy.yaml` | Bluetooth proxy wiring |
| `_espnow.yaml` | Peer-to-peer relay control — see the [ESP-NOW docs](docs/espnow.md) |
| `_hw_leds_individual.yaml` | Per-LED control of the ring, instead of the single `LED ring` entity |
| `_hw_speaker.yaml` | Basic speaker, as an alternative to `_media_player.yaml` — use one or the other, never both |

**Bluetooth proxy** can also be enabled with ESPHome's native component, by adding this to your
device configuration:

```yaml
bluetooth_proxy:
  # active: true
```

> [!WARNING]
> Bluetooth proxy is experimental here and not fully tested. It needs the ESP-IDF framework and a
> fair amount of free memory, so adding further custom components on top may not fit. It is not
> included by default, to keep memory usage sane. Use at your own risk.

For finer control over which packages get pulled in, start from the
[advanced configuration template](TX-Ultimate-Easy-ESPHome_advanced.yaml). It is useful for
isolating a misbehaving component or trimming memory usage.

<!-- markdownlint-disable MD033 -->
<details>
<summary><b>Advanced configuration example</b></summary>

```yaml
substitutions:
  name: tx-ultimate-easy
  friendly_name: TX Ultimate Easy
  device_format: EU
  gang_count: 1

packages:
  remote_package:
    url: https://github.com/GuyZipory/TX-Ultimate-Easy
    ref: main
    refresh: 5min
    files:
      # Core (essential) packages
      - ESPHome/TX-Ultimate-Easy-ESPHome_common.yaml      # Basic shared settings
      - ESPHome/TX-Ultimate-Easy-ESPHome_hw_buttons.yaml  # Button logic
      - ESPHome/TX-Ultimate-Easy-ESPHome_hw_leds.yaml     # LED configuration
      - ESPHome/TX-Ultimate-Easy-ESPHome_hw_touch.yaml    # Touch panel support

      # Optional but recommended
      - ESPHome/TX-Ultimate-Easy-ESPHome_hw_relays.yaml     # Relay control
      - ESPHome/TX-Ultimate-Easy-ESPHome_hw_vibration.yaml  # Haptic feedback

      # Audio — use none, or exactly one of these
      - ESPHome/TX-Ultimate-Easy-ESPHome_media_player.yaml  # Media player (recommended)
      # - ESPHome/TX-Ultimate-Easy-ESPHome_hw_speaker.yaml  # Basic speaker

      # Add-ons
      # - ESPHome/TX-Ultimate-Easy-ESPHome_espnow.yaml
```

</details>
<!-- markdownlint-enable MD033 -->

> [!NOTE]
> Excluding core packages may cause instability or reduced functionality.

## Configuring the device

Everything below is adjustable from the device page in Home Assistant
(**Settings → Devices & Services → ESPHome → your device**), without recompiling.

### Relays

Each relay's exposure is set with the `relay_N_mode` substitution:

| Mode | Result |
| --- | --- |
| `switch` (default) | A `switch.` entity named **Relay N** |
| `light` | A `light.` entity named **Relay N light** |
| `disabled` | The relay is not exposed at all |

Both modes are plain on/off — the relay is a dry contact, so it has no dimming of its own
regardless of which one you pick. Use `relay_N_mode: light` if you want a `light.` entity; there
is no need for Home Assistant's *show as Light* (`switch_as_x`) helper, which leaves a second,
hidden copy of the relay on the device page.

### LED indicators

The LED ring is driven by three kinds of entity, each owning exactly one job:

| Entity | Type | What it does |
| --- | --- | --- |
| **Relay N indicator** | Light (config) | How relay N's indicator looks when the relay is on — **colour**, **brightness**, and on/off as the master enable for that indicator |
| **Relay N indicator area** | Select (config) | Which part of relay N's arc lights up: `Bottom`/`Left`, `Side`, `Top`/`Right`, or `All` |
| **LED ring** | Light | Manual control of the whole ring, with the effects library |

**Relay N indicator** does not drive the LEDs directly — it is the style the firmware paints with
whenever relay N turns on. Changing its colour or brightness updates the panel live. Turn it off
to stop relay N lighting the ring at all.

**LED ring** takes over the whole strip while it is on: relay indicators stand down rather than
fight it for the same LEDs. Turn it off and the indicators repaint themselves immediately (or
Night Mode reapplies, if that is active).

### Night Mode

Night Mode freezes the LEDs at a fixed colour and brightness and, optionally, suppresses vibration
and click-sound feedback. The state survives reboots.

| Entity | Type | Default | Description |
| --- | --- | --- | --- |
| Night Mode | Switch | Off | Enable/disable night mode |
| Night Mode - LED | Light (config) | Off | The LED colour and brightness used in night mode |
| Night Mode - Suppress Vibration | Switch (config) | On | Block haptic feedback while night mode is active |
| Night Mode - Suppress Sound | Switch (config) | On | Block click sounds while night mode is active |

**Usage:**

1. Set **Night Mode - LED** to the colour and brightness you want.
2. Enable the **Night Mode** switch — the colour is snapshotted and the LEDs lock in.
3. Toggle the suppression switches under *Configuration* to allow or block vibration and sound.
4. To change the colour: disable Night Mode, adjust **Night Mode - LED**, then re-enable. That
   re-snapshots the colour and persists it across reboots.
5. Disable the switch to restore normal relay indicator behaviour.

Night Mode never changes the **LED ring** entity's own state. If you switch the LED ring on while
Night Mode is active it paints over the night colour, and turning it back off restores the night
colour rather than the relay indicators.

### Touch and buttons

| Entity | What it controls |
| --- | --- |
| **Button N action** | What a press does locally: nothing (leaving the button free for Home Assistant automations) or toggle the matching relay |
| **Button - Long-press delay** | How long a press must be held to count as a long-press instead of a click |
| **Button - Multi-click delay** | The double-click window — and therefore the delay before a relay toggles |
| **Touch - Event duration** | How long the touch event sensors stay ON in Home Assistant. Does not affect click detection or relay latency |
| **Touch - Multi-touch gestures** | Enables multi-touch recognition |
| **Touch - Swipe gestures** | Enables swipe recognition |
| **Touch - Vibration feedback** | `Disabled`, `On press`, `On release` or `Always` |
| **Vibration - Duration** | How long the motor runs per pulse |

### Factory defaults

The firmware ships tuned as a fast, reliable **single-tap** switch. Everything below is opt-in from
the Home Assistant UI — nothing is removed, it just starts off:

| Setting | Default | Why |
| --- | --- | --- |
| Button - Long-press delay | `2000` ms | Long-press triggers no relay action, so a press longer than this does nothing at all. A high value means a slow or lingering tap still toggles the light. |
| Button - Multi-click delay | `0` ms | The relay toggles the instant you lift your finger, with no lag. |
| Touch - Event duration | `100` ms | Pulse width of the touch event sensors in HA. Does not affect click detection or relay latency. |
| Touch - Multi-touch gestures | Off | The panel's MCU often misreads an ordinary tap as a gesture, and a gesture cancels the pending click — so the light would not turn on. |
| Touch - Swipe gestures | Off | Same reason as multi-touch. |
| Touch - Vibration feedback | Disabled | Haptics are a matter of taste; the motor is audible in a quiet room. |
| Vibration - Duration | `150` ms | Long enough for the motor to spin up and actually be felt, once feedback is enabled. |

**Double-click is off by default.** `Button - Multi-click delay: 0` is what makes the relay respond
instantly, but it also means the click counter never reaches two, so the double-click and
multiple-click events never fire. This is an unavoidable trade — the window needed to detect a
second tap (~250 ms) *is* the delay before the relay moves. Set *Button - Multi-click delay* to
around `250` to get double-click back, accepting that lag on every toggle.

**To use swipe or multi-touch gestures**, turn on *Touch - Swipe gestures* / *Touch - Multi-touch
gestures*, then enable the corresponding event entities (they are hidden by default) under the
device page → **+ N entities not shown**.

> [!NOTE]
> These settings live in NVS. On an update, a setting you have never changed by hand picks up the
> new shipped default, while one you *have* changed keeps your value.

### Advanced tuning

Two optional substitutions free up IRAM on newer ESP32 silicon. Both default to the safe, existing
behaviour — **only set them once you have confirmed your device supports them**, by reading its
boot log.

```yaml
substitutions:
  esp32_minimum_chip_revision: "3.1"   # default: "3.0"
  esp32_sram1_as_iram: "true"          # default: "false"
```

- **`esp32_minimum_chip_revision`** — frees ~10 kB of IRAM.
  Safe only if your boot log reports `Chip rev >= 3.0 detected` **and** the chip is revision 3.1
  or newer. The bootloader will refuse to start on older silicon.
- **`esp32_sram1_as_iram`** — frees ~40 kB of IRAM.
  Safe only if your boot log says `Bootloader supports SRAM1 as IRAM`.
  Requires a bootloader from ESP-IDF v5.1 or newer.

> [!WARNING]
> Enabling `esp32_sram1_as_iram` on a device with an older bootloader means the app will not boot.
> **OTA updates cannot replace the bootloader** — recovery requires a USB flash. Check the boot log
> first.

To find those lines, look near the top of the device's log output right after a restart:

```text
[W][app:168]: Chip rev >= 3.0 detected. Set minimum_chip_revision: "3.1" ...
[W][app:198]: Bootloader supports SRAM1 as IRAM (+40KB). Set sram1_as_iram: true ...
```

ESPHome only prints each line when that option is actually applicable to your hardware.

### Events and automation

The device fires Home Assistant events for clicks, long-presses, swipes and multi-touch, which is
the reliable way to trigger automations — sensors tell you the current state, events capture the
action. Automation itself is left entirely to Home Assistant; this firmware focuses on exposing
the device properly.

Payload schema and example automations are in the **[Events docs](docs/events.md)**.

## Contributing

Fork the repo, branch from `main`, and open a pull request targeting `main`. Changes need to pass
the lint gates (YAML, C++, Markdown) and the Arduino and ESP-IDF build matrix.

Bugs and feature requests belong in
[GitHub Issues](https://github.com/GuyZipory/TX-Ultimate-Easy/issues).

## Acknowledgments

This project builds on the work of several others:

<!-- markdownlint-disable MD013 -->
- [edwardtfn/TX-Ultimate-Easy](https://github.com/edwardtfn/TX-Ultimate-Easy) by Edward Firmo — the upstream project this is forked from
- [SmartHome yourself - SONOFF TX Ultimate for ESPHome](https://github.com/SmartHome-yourself/sonoff-tx-ultimate-for-esphome)
- [Un loco y su tecnología - Sonoff TX Ultimate with ESPHome](https://www.youtube.com/watch?v=58v8oqSQgXQ)
- [@PxPert](https://github.com/PxPert) - [Sonoff TX Ultimate and Voice Assistant](https://community.home-assistant.io/t/sonoff-tx-ultimate-and-voice-assistant/682214?u=edwardtfn)
<!-- markdownlint-enable MD013 -->

## License

MIT — see [LICENSE](LICENSE).

---

<!-- markdownlint-disable MD033 -->
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="Assets/Logo_dark.png">
  <source media="(prefers-color-scheme: light)" srcset="Assets/Logo_light.png">
  <img alt="TX Ultimate Easy Logo" src="Assets/Logo_light.png" width="100">
</picture>
<!-- markdownlint-enable MD033 -->
