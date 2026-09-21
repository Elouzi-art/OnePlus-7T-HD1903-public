# OnePlus 7T HD1903 — Root Magisk 30.7 + NetHunter

**Reference documentation for rooting a OnePlus 7T HD1903 running OxygenOS 11, then installing Kali NetHunter.**

|                     |                                                                            |
| ------------------- | -------------------------------------------------------------------------- |
| **Device**          | OnePlus 7T HD1903 (`hotdogb`)                                              |
| **ROM**             | OxygenOS 11 / Android 11                                                   |
| **Reference build** | `OnePlus7TOxygen_14.E.35_GLO_0350_2206171459`                              |
| **Magisk**          | 30.7 (30700)                                                               |
| **Method**          | Magisk patch → `fastboot boot` (test) → `dd` (backup) → Direct Install    |
| **Status**          | Tested and working                                                        |

---

## Why this repo exists

The **HD1903** (India / global) variant of the OnePlus 7T is poorly covered by existing documentation:

- A matching stock `boot.img` for this exact variant and build is **very hard to find** online. Most forum links are dead, hosted on expired services, or belong to other variants (HD1901, HD1905, HD1907) or other OxygenOS builds.
- Flashing a `boot.img` from another variant or build very quickly leads to **Qualcomm CrashDump**.
- Many available guides present the steps in a logically impossible order (extracting the boot via `dd` **before** having root — see the note below).

This repo provides an **end-to-end verified procedure**, with the correct step order and the pitfalls encountered along the way.

---

## The step that blocks everyone

Many tutorials ask you to extract the stock boot with:

```bash
dd if=/dev/block/bootdevice/by-name/boot_b of=/sdcard/boot_b-stock.img
```

This command requires `su` — **which you don't have yet at this stage**. It's a chicken-and-egg problem.

The correct order, detailed in the guide:

```text
1. Get a stock boot.img WITHOUT root (extracted from the OOS firmware)
2. Patch it with Magisk
3. fastboot boot            → TEMPORARY ROOT
4. dd                       → back up the device's real stock boot
5. Magisk → Direct Install  → PERSISTENT ROOT
```

The `dd` step is still necessary, but as a **reference backup**, not as a starting point. It must be done **before** Direct Install, otherwise you end up backing up an already-patched boot.

---

## Repo contents

```text
.
└── OnePlus 7T HD1903 — Stock Boot Image → Magisk Patch → Safe Temporary Boot → Persistent Root.md
```

Full step-by-step guide, including:

- Exact build identification before doing anything
- Extracting and verifying the stock `boot.img` (SHA-256)
- Magisk patching, safe testing via `fastboot boot`
- Backing up the real boot before persistent installation
- Troubleshooting (CrashDump, boot failure, `su` not responding)
- A known WLAN compatibility issue (`qca_cld3_wlan.ko`)

**[→ Read the full guide](./OnePlus%207T%20HD1903%20%E2%80%94%20Stock%20Boot%20Image%20%E2%86%92%20Magisk%20Patch%20%E2%86%92%20Safe%20Temporary%20Boot%20%E2%86%92%20Persistent%20Root.md)**

> This repo does **not** redistribute any binaries (Magisk, NetHunter, firmware). Always use the official sources linked below.

---

## Related contribution

A condensed version of this guide's key safety points (safe Magisk boot patching + the known WLAN
module compatibility issue) has been submitted upstream to the official Kali NetHunter kernels repo,
for the `oneplus7-oos` (OxygenOS 11) kernel:

**[→ Merge request #464 — kali-nethunter-kernels](https://gitlab.com/kalilinux/nethunter/build-scripts/kali-nethunter-kernels/-/merge_requests/464)**

This repo (GitHub) holds the full detailed walkthrough; the upstream MR (GitLab) holds a condensed
version scoped to what the official NetHunter docs don't already cover.

---

## Prerequisites and official downloads

- A PC with `adb` and `fastboot` installed
- A working USB cable
- Unlocked bootloader, USB debugging enabled
- **The OxygenOS firmware matching your exact installed build**
- [Magisk (official releases, topjohnwu)](https://github.com/topjohnwu/Magisk/releases)
- [Kali NetHunter — official installation docs](https://www.kali.org/docs/nethunter/)
- [NetHunter Store — official OffSec repo](https://gitlab.com/kalilinux/nethunter/apps/nethunterstore)

Check this first, before doing anything else:

```bash
adb shell getprop ro.product.model
adb shell getprop ro.build.version.release
adb shell getprop ro.build.display.id
adb shell getprop ro.boot.slot_suffix
```

If your `ro.build.display.id` differs from the reference build above, **do not use a `boot.img` from another build** — extract your own from your own firmware, as explained in the guide.

---

## Safety rule

Never do this directly:

```bash
fastboot flash boot magisk_patched-xxxxx.img
```

Always test first:

```bash
fastboot boot magisk_patched-xxxxx.img
```

`boot` is temporary: if the image is bad, a simple reboot returns you to the previous state. `flash` is permanent — this is what caused Qualcomm CrashDump during the first attempts documented in the guide.

---

## Contributing

Contributions are welcome, especially:

- Confirmations that the procedure works on other OxygenOS 11 builds
- SHA-256 checksums of stock `boot.img` files for other variants (HD1901, HD1905, HD1907)
- Corrections and clarifications to the procedure

Please include the following in your issues:

```text
Model         : (ro.product.model)
Build         : (ro.build.display.id)
Version       : (ro.build.version.release)
Active slot   : (ro.boot.slot_suffix)
Magisk version:
Blocked at    :
```

---

## Disclaimer

Rooting a device voids the warranty, may break certain apps (banking, contactless payments, DRM), and carries a risk of temporary bricking.

All operations described here are performed **at your own risk**. The author of this repo is not responsible for damaged devices or lost data.

**Back up your data before starting.**

---

## License

Documentation published under [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/).

Magisk is a project by John Wu, distributed under the GPL-3.0 license.
Kali NetHunter is an OffSec project.
Firmware remains the property of OnePlus / OPPO; this repo does not redistribute any firmware files or third-party binaries.
