# esphome-usb-host

## Purpose
A reworked `usb_host` for ESPHome: plug in a device and it runs. The host hands every device to the drivers by its
interface classes, so a keyboard, a USB stick, a camera or a sound card works without its vendor and product ID. On
the ESP32-P4 both USB controllers run at once; devices that start in the wrong mode are switched automatically.

## How it works
One shared USB task and one transfer engine (bulk, interrupt, control, isochronous) serve every driver. Each device
goes to the drivers in this order: a driver whose filters name it (`vid` / `pid` / names), a driver whose built-in
table names it, then the drivers of its interface classes - per interface, so one device can feed several drivers.
A driver's `blacklist` keeps a device away. The drivers themselves are in esphome-usb-drivers; take `usb_host` from
this repository only, not the older copy in esphome-usb-drivers.

## Quick start
Chips: ESP32-S2, ESP32-S3, ESP32-P4, ESP32-H4, ESP32-S31

Needs: [esphome-esp32-platform](https://github.com/p1ngb4ck/esphome-esp32-platform)

```yaml
external_components:
  - source: github://p1ngb4ck/esphome-usb-host@main
    components: [usb_host, tinyusb]
  - source: github://p1ngb4ck/esphome-esp32-platform@main
    components: [esp32]

usb_host:
  enable_hubs: true
```

Then add the drivers you need from [esphome-usb-drivers](https://github.com/p1ngb4ck/esphome-usb-drivers). Both
controllers of an ESP32-P4: `dual_host: true`; a USB device (`tinyusb: port:`) on one P4 controller beside
`usb_host` on the other.

## Main options
| Option | What it does | Default |
|---|---|---|
| [`enable_hubs`](CONFIGURATION.md#usb_host) | devices behind USB hubs | `false` |
| [`max_transfer_requests`](CONFIGURATION.md#usb_host) | transfers in flight per client, 1-32 | `16` |
| [`controllers` / `dual_host`](CONFIGURATION.md#usb_host) | P4: `high_speed`, `full_speed`, `both` | `high_speed` |
| [`fs_pins`](CONFIGURATION.md#usb_host) | ESP32-P4: GPIO pair of the full-speed controller | `26_27` |
| [`mode_switch`](CONFIGURATION.md#mode_switch) | `hid2hci`, usb_modeswitch `bulk_messages`, `control` | - |
| [`vid` / `pid` / `blacklist`](CONFIGURATION.md#options-every-usb-driver-has) | options of every driver | any device |
| [`tinyusb: port`](CONFIGURATION.md#tinyusb-beside-usb_host-esp32-p4) | P4: the controller it uses | `full_speed` |

## Catalog
- Components (folder on main, configuration):
  - `usb_host`: [folder](https://github.com/p1ngb4ck/esphome-usb-host/tree/main/esphome/components/usb_host),
    [configuration](CONFIGURATION.md#usb_host), [mode switch](CONFIGURATION.md#mode_switch),
    [driver options](CONFIGURATION.md#options-every-usb-driver-has)
  - `tinyusb`: [folder](https://github.com/p1ngb4ck/esphome-usb-host/tree/main/esphome/components/tinyusb),
    [configuration](CONFIGURATION.md#tinyusb-beside-usb_host-esp32-p4)
- Branches:
  - [`main`](https://github.com/p1ngb4ck/esphome-usb-host/tree/main): the whole topic
  - [`pr/usb-host-refactor`](https://github.com/p1ngb4ck/esphome-usb-host/tree/pr/usb-host-refactor)
  - [`pr/usb-host-dual-host`](https://github.com/p1ngb4ck/esphome-usb-host/tree/pr/usb-host-dual-host)
  - [`pr/usb-host-mode-switch`](https://github.com/p1ngb4ck/esphome-usb-host/tree/pr/usb-host-mode-switch)
  - [`pr/tinyusb-port`](https://github.com/p1ngb4ck/esphome-usb-host/tree/pr/tinyusb-port)
- Needed repos: [esphome-esp32-platform](https://github.com/p1ngb4ck/esphome-esp32-platform) (esp32)
- Drivers: [esphome-usb-drivers](https://github.com/p1ngb4ck/esphome-usb-drivers)
- All topics together: `github://p1ngb4ck/esphome@dev`

---

# Detailed version

A reworked `usb_host` for ESPHome: **plug in a device and it runs.** The host hands every device to the drivers by
what it is (its interface classes), so a keyboard, a USB stick, a camera or a sound card works without knowing its
vendor and product ID in advance. On the ESP32-P4 both USB controllers can run at once, and devices that start in
the wrong mode (Bluetooth dongles that enumerate as a driver CD, HID-proxy dongles) are switched automatically.

The device drivers (USB storage, optical drives, HID, UVC cameras, Bluetooth dongles, audio, serial adapters,
Ethernet) are in [esphome-usb-drivers](https://github.com/p1ngb4ck/esphome-usb-drivers). esphome-usb-drivers' `main`
carries an older copy of `usb_host`: take `usb_host` from this repository only. `esp32` comes from its owner
[esphome-esp32-platform](https://github.com/p1ngb4ck/esphome-esp32-platform); use only one `esp32` source in a
config.

## Examples

```yaml
external_components:
  - source: github://p1ngb4ck/esphome-usb-host@main
    components: [usb_host, tinyusb]
  - source: github://p1ngb4ck/esphome-esp32-platform@main
    components: [esp32]

usb_host:
  enable_hubs: true
```

Then add the drivers you need from esphome-usb-drivers - each picks up its devices by class. Both controllers of an
ESP32-P4 (high-speed and full-speed port):

```yaml
usb_host:
  dual_host: true
  enable_hubs: true
```

`tinyusb` comes from this repository as well: upstream's `tinyusb` refuses to be combined with `usb_host`; this one
runs on one ESP32-P4 controller beside `usb_host` on the other (`tinyusb: port:`).

## How routing works

`usb_host` hands each device to the drivers in this order: a driver whose configured filters name the device
(`vid` / `pid` / manufacturer / product), then a driver whose built-in table names it (devices that cannot be routed
by class), then the drivers of its interface classes. A driver gets the interfaces it matches, not the whole device,
so the other interfaces of the same device go to the drivers that match them (a camera's video beside its
microphone). `vid` / `pid` filters stay useful to give one particular device to one particular driver, and a
driver's `blacklist` keeps a device away from it. The serial number is read with every device's descriptors, so
devices of the same type can be told apart.

## More

**[CONFIGURATION.md](CONFIGURATION.md)**: every option - controllers, hubs, transfer limits, blacklists, mode
switching (hid2hci, usb_modeswitch messages, control requests), `tinyusb` beside `usb_host` on the ESP32-P4.

## Branches

One branch per change; `main` contains every branch. Base: ESPHome `dev` at `a5a75738ad`.

| Branch | Change | Chips | Needs |
|---|---|---|---|
| `pr/usb-host-refactor` | routing by device class to the drivers (1) | S2, S3, S31, P4, H4 | (2) |
| `pr/usb-host-dual-host` | `controllers`, `dual_host`, `fs_pins`, `full_speed_only` | P4 (3) | `pr/usb-host-refactor` |
| `pr/usb-host-mode-switch` | `mode_switch`: wrong-mode devices switched (4) | as `usb_host` | `pr/usb-host-refactor` |
| `pr/tinyusb-port` | `tinyusb` `port` on the ESP32-P4 (5) | P4 | `pr/usb-host-dual-host` |

1. Named > vendor table > class, interfaces per driver; several devices per client, one shared USB task, unified
   transfer engine (bulk, interrupt, control, isochronous streams), driver `blacklist`, host channel budget per
   controller, the patched espressif/usb component (per-port FIFO bias, `fsls_only`, external hub port requests,
   root port reset recovery).
2. `pr/esp32-loop-task-core`, `pr/esp32-idf-component-override-path`
   ([esphome-esp32-platform](https://github.com/p1ngb4ck/esphome-esp32-platform)).
3. `full_speed_only`: on P4 and S31.
4. Switched before they are routed - `hid2hci`, usb_modeswitch `bulk_messages`, any `control` request; switchable at
   runtime, kept in the preferences.
5. `high_speed` / `full_speed`: a USB device on one controller beside `usb_host` on the other.

All topics together: `github://p1ngb4ck/esphome@dev`. `external_components` replaces components only; the ESPHome
core of the build host stays as installed.

## License

- ESPHome's own files, also where this repository changes them: the [ESPHome License](LICENSE) (MIT for the Python
  code and everything else, GPLv3 for the C++ runtime files).
- New files written for this repository (new components, their code, tests and documentation): MIT,
  [LICENSE-MIT](LICENSE-MIT).

## Thanks

The mode switch methods follow BlueZ's `hid2hci` and Josua Dietze's usb_modeswitch - thank you for documenting
what all those dongles need. Everyone this project builds on:
[ATTRIBUTIONS.md](https://github.com/p1ngb4ck/esphome/blob/dev/ATTRIBUTIONS.md).

## Branch structure

Each `pr/` branch holds one part of this topic on top of upstream and says in its `BRANCH.md` what it is based on.
`main` holds all of them.

| Branch | Based on |
|---|---|
| `pr/tinyusb-port` | `pr/usb-host-dual-host` |
| `pr/usb-host-dual-host` | `pr/usb-host-refactor` |
| `pr/usb-host-mode-switch` | `pr/usb-host-refactor` |
| `pr/usb-host-refactor` | `pr/esp32-idf-component-override-path`, `pr/esp32-loop-task-core` (both esp32-platform) |
