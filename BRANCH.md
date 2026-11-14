# `pr/usb-host-mode-switch`

One part of the topic repo p1ngb4ck/esphome-usb-host. The whole topic, with its description, is the branch `main`.

Change of this branch: `mode_switch`: devices that enumerate in the wrong mode switched before they are routed - `hid2hci`, usb_modeswitch `bulk_messages`, any `control` request; switchable at runtime, kept in the preferences

Based on these branches, whose changes it holds and builds on:

- `pr/usb-host-refactor`: routing by device class to the drivers (named > vendor table > class, interfaces per driver), several devices per client, one shared USB task, unified transfer engine (bulk, interrupt, control, isochronous streams), driver `blacklist`, host channel budget per controller, the patched espressif/usb component (per-port FIFO bias, `fsls_only`, external hub port requests, root port reset recovery)
