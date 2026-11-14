# `pr/usb-host-refactor`

One part of the topic repo p1ngb4ck/esphome-usb-host. The whole topic, with its description, is the branch `main`.

Change of this branch: routing by device class to the drivers (named > vendor table > class, interfaces per driver), several devices per client, one shared USB task, unified transfer engine (bulk, interrupt, control, isochronous streams), driver `blacklist`, host channel budget per controller, the patched espressif/usb component (per-port FIFO bias, `fsls_only`, external hub port requests, root port reset recovery)

Based on these branches, whose changes it holds and builds on:

- `pr/esp32-idf-component-override-path` of p1ngb4ck/esphome-esp32-platform: `add_idf_component(override_path=...)`: a built-in IDF component replaced by a project copy
- `pr/esp32-loop-task-core` of p1ngb4ck/esphome-esp32-platform: `LOOP_TASK_CORE` in `esp32/hal.h`, the loop task pinned to it
