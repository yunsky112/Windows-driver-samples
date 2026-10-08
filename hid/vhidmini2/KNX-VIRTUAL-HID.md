# KNX virtual HID variant

The UMDF2 sample now exposes a KNX HID application collection using the
thelsing/knx Usage Pages `0xFFA0` and `0xFFA1`. Report ID `0x01` has 63 data
bytes plus its one-byte ID in both directions. KNX USB fragmentation and BAS
responses remain in the companion Qt application; this driver transports
complete 64-byte HID reports.

The device attributes use VID `0x28C2`, PID `0x001C`, and version `0x0200`.
These are the active USB device descriptor values in
`KAIStack_V2.21/appl/USBIF/src/USB_User/usb_desc.c` (`C2 28 1C 00`). The source
comment mentioning `0483/0018` conflicts with those bytes. Use of this VID/PID
outside development requires permission from its owner.

The virtual HID also has a separate private application collection, Usage Page
`0xFFA2`, Usage `0x01`, Feature Report ID `0xF0`. Only the Qt bridge should open
that collection:

| Byte | SetFeature (`Qt` to driver) | GetFeature (driver to `Qt`) |
| --- | --- | --- |
| 0 | `F0` | `F0` |
| 1 | `01` = inject input report | `01` = output available, `00` = none |
| 2–65 | 64-byte KNX HID report | 64-byte KNX HID report when available |

`WriteReport` and `SetOutputReport` queue reports from ETS. `GetFeature`
dequeues them for Qt. `SetFeature` queues reports from Qt. The driver completes
pending `ReadReport` requests with those input reports. Both queues hold up to
32 reports and return an error instead of silently dropping full-queue writes.

The INF installs a root-enumerated virtual HID (`root\KnxVirtualHid`). It is
not a physical USB device and does not create a USB bus `USB\VID_28C2&PID_001C`
node. ETS visibility must be verified on the target Windows release. The
Microsoft UMDF2 sample INF requires Windows build 22000 or later. The GitHub
workflow builds the x64 package; installation still requires an appropriate
driver signature and a supported Windows system.
