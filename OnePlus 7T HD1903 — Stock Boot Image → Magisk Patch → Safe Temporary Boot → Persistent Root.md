# OnePlus 7T HD1903 — Stock Boot Image → Magisk Patch → Safe Temporary Boot → Persistent Root

> **Device:** OnePlus 7T HD1903 (`hotdogb`)  
> **OS:** OxygenOS 11 / Android 11  
> **Reference build:** `OnePlus7TOxygen_14.E.35_GLO_0350_2206171459`  
> **Build ID:** `HD1903_14_220617`  
> **Incremental:** `2206171329`  
> **Reference Magisk:** 30.7  
> **Method:** exact stock boot → Magisk patch → `fastboot boot` → temporary root → stock boot backup → Magisk Direct Install

---

## 1. Purpose

This document provides a reproducible procedure for **OnePlus 7T HD1903** users who need a compatible stock `boot.img` for Magisk but cannot find the correct image online.

The important part of this guide is not a prebuilt image.

It is the method:

```text
Identify the exact firmware
        ↓
Obtain the matching OxygenOS package
        ↓
Extract the stock boot image
        ↓
Verify the image
        ↓
Patch it with Magisk
        ↓
Test it with fastboot boot
        ↓
Verify temporary root
        ↓
Back up the real stock boot
        ↓
Only then install Magisk persistently
```

This procedure was tested on a OnePlus 7T HD1903 running OxygenOS 11.

It should **not** be assumed that the same `boot.img` is valid for every HD1903 build or every OnePlus 7T variant.

---

# 2. Why the exact `boot.img` matters

The OnePlus 7T family has multiple variants and multiple OxygenOS builds.

A file named simply:

```text
boot.img
```

does not provide enough information to establish compatibility.

Do not assume that a boot image from:

- another OnePlus 7T variant,
- another OxygenOS build,
- another region,
- or another Android release

is interchangeable.

For this reason, the procedure starts by identifying the exact firmware installed on the phone.

The reference device reported:

```text
Model:              HD1903
Product:            OnePlus7T
Product name:       OnePlus7T_EEA
Codename:           hotdogb
Android:            11
Build ID:           HD1903_14_220617
OTA version:        OnePlus7TOxygen_14.E.35_GLO_0350_2206171459
Incremental:        2206171329
```

These values were obtained directly from the device.

---

# 3. Prerequisites

On the PC:

- `adb`
- `fastboot`
- a working USB cable
- an unlocked bootloader
- USB debugging enabled
- Magisk
- the OxygenOS firmware package corresponding to the installed build
- a payload extraction tool if the firmware uses `payload.bin`

Check ADB:

```bash
adb version
```

Check Fastboot:

```bash
fastboot --version
```

> **Important:** Make a backup of personal data before modifying the boot process.

---

# 4. Identify the exact device and firmware

Connect the phone while Android is running.

Check the device:

```bash
adb devices
```

Then collect the important build information:

```bash
adb shell getprop ro.product.model
adb shell getprop ro.product.device
adb shell getprop ro.product.name
adb shell getprop ro.build.version.release
adb shell getprop ro.build.display.id
adb shell getprop ro.build.version.incremental
adb shell getprop ro.build.version.ota
adb shell getprop ro.build.fingerprint
adb shell getprop ro.boot.slot_suffix
```

Reference output:

```text
HD1903
OnePlus7T
OnePlus7T_EEA
11
HD1903_14_220617
2206171329
OnePlus7TOxygen_14.E.35_GLO_0350_2206171459
_b
```

The real device log additionally confirms the firmware fingerprint:

```text
OnePlus/OnePlus7T_EEA/OnePlus7T:11/RKQ1.201022.002/2206171329:user/release-keys
```

and the kernel build identifier:

```text
4.14-G2206171459
```

### STOP condition

If your device reports a different build, **do not blindly use the reference boot image from this documentation**.

Instead, obtain the firmware corresponding to your own build and extract its `boot.img`.

---

# 5. Check the active slot

This device uses an A/B partition layout.

Check the active slot:

```bash
adb shell getprop ro.boot.slot_suffix
```

Possible results include:

```text
_a
```

or:

```text
_b
```

The reference device was running:

```text
_b
```

Do not assume that your device is also on `_b`.

Always check.

Before any boot-related operation in fastboot, check again:

```bash
fastboot getvar current-slot 2>&1
```

---

# 6. Obtain the stock `boot.img`

At this point the phone is **not yet root**.

Therefore this is not the moment to use:

```bash
dd
```

The initial stock boot image must come from the matching OxygenOS firmware package.

If the package contains `payload.bin`, extract the boot partition with a compatible payload extraction tool.

Example:

```bash
python3 payload_dumper.py --partitions boot payload.bin
```

The result should be:

```text
boot.img
```

Rename it:

```bash
mv boot.img boot_oos11-stock.img
```

Calculate its checksum:

```bash
sha256sum boot_oos11-stock.img
```

Keep the checksum.

---

# 7. Verify the source of the boot image

Before patching anything, record:

```text
Device:
Model:
Android version:
OxygenOS build:
OTA version:
Incremental:
Firmware source:
SHA-256:
```

Example:

```text
Device:       OnePlus 7T
Model:        HD1903
Android:      11
Build:        HD1903_14_220617
OTA:          OnePlus7TOxygen_14.E.35_GLO_0350_2206171459
Incremental:  2206171329
SHA-256:      <calculated locally>
```

The SHA-256 value must be generated from the actual file being used.

Do not copy a checksum from an unrelated source.

---

# 8. Install Magisk

Install Magisk on the phone.

For the reference procedure, Magisk 30.7 was used.

The APK can be copied to the device:

```bash
adb push Magisk-v30.7.apk /sdcard/Download/
```

Verify the file:

```bash
adb shell ls -lh /sdcard/Download/Magisk-v30.7.apk
```

Install it from Android and open Magisk.

At this stage:

```text
Magisk installed
        ≠
root already active
```

Check:

```bash
adb shell su -c id
```

It is normal for root not to be available yet.

---

# 9. Copy the stock boot to the phone

Copy the exact stock image obtained from the matching firmware:

```bash
adb push boot_oos11-stock.img /sdcard/Download/boot_oos11-stock.img
```

Verify:

```bash
adb shell ls -lh /sdcard/Download/boot_oos11-stock.img
```

Do not replace this image with a boot image downloaded from another build.

---

# 10. Patch the stock boot with Magisk

Open Magisk:

```text
Install
    ↓
Select and Patch a File
```

Select:

```text
/sdcard/Download/boot_oos11-stock.img
```

Magisk should generate a patched image similar to:

```text
/sdcard/Download/magisk_patched-xxxxx.img
```

The exact filename is generated by Magisk.

---

# 11. Retrieve and verify the patched image

List the generated images:

```bash
adb shell ls -lt /sdcard/Download/magisk_patched*.img
```

Pull the exact file:

```bash
adb pull /sdcard/Download/magisk_patched-xxxxx.img .
```

Check it:

```bash
ls -lh magisk_patched-xxxxx.img
sha256sum magisk_patched-xxxxx.img
```

Keep the patched image on the PC.

---

# 12. CRITICAL: Do not flash the patched boot yet

Do **not** immediately run:

```bash
fastboot flash boot magisk_patched-xxxxx.img
```

The reference procedure specifically encountered:

```text
Qualcomm CrashDump
```

after a direct flash attempt.

For this reason, the safer validation path used here is:

```text
fastboot boot
```

instead of:

```text
fastboot flash
```

The objective is to test whether the patched image can boot before performing a persistent installation.

---

# 13. Enter fastboot mode

From Android:

```bash
adb reboot bootloader
```

Check that fastboot detects the phone:

```bash
fastboot devices
```

Expected format:

```text
<device>    fastboot
```

Check the active slot:

```bash
fastboot getvar current-slot 2>&1
```

Do not continue if the slot information is unexpected.

---

# 14. Test the patched boot temporarily

Use:

```bash
fastboot boot magisk_patched-xxxxx.img
```

**Do not substitute `flash` here.**

The purpose of this operation is to load the patched image temporarily and test it.

Wait for Android to boot completely.

---

# 15. Verify temporary root

Check ADB:

```bash
adb devices
```

Then:

```bash
adb shell su -c id
```

Expected:

```text
uid=0(root)
```

Check Magisk:

```bash
adb shell su -c 'magisk -c'
```

Reference result:

```text
30.7:MAGISK:R (30700)
```

At this point the expected state is:

```text
Patched boot
     ↓
Temporary boot successful
     ↓
Android starts
     ↓
Magisk available
     ↓
su → uid=0(root)
```

### STOP condition

If Android does not boot correctly, or `su -c id` does not provide root:

**Do not continue to persistent installation.**

Do not perform Direct Install until the temporary boot has been successfully validated.

---

# 16. Back up the real stock boot

This is the point where `dd` becomes possible.

The temporary Magisk boot provides root access.

First verify:

```bash
adb shell su -c id
```

Expected:

```text
uid=0(root)
```

Check the active slot again:

```bash
adb shell getprop ro.boot.slot_suffix
```

For the reference device:

```text
_b
```

For slot `_b`, verify the partition:

```bash
adb shell su -c 'ls -l /dev/block/bootdevice/by-name/boot_b'
```

Then create the stock backup:

```bash
adb shell su -c 'dd if=/dev/block/bootdevice/by-name/boot_b of=/sdcard/boot_b-stock.img'
```

Check the resulting file:

```bash
adb shell ls -lh /sdcard/boot_b-stock.img
```

Pull it to the PC:

```bash
adb pull /sdcard/boot_b-stock.img .
```

Calculate the checksum:

```bash
sha256sum boot_b-stock.img
```

---

# 17. Why the `dd` step comes here

A common mistake is to try:

```bash
dd if=/dev/block/bootdevice/by-name/boot_b ...
```

before root is available.

That does not work in the initial unrooted state.

The correct sequence is:

```text
Exact stock boot from firmware
        ↓
Magisk patch
        ↓
fastboot boot
        ↓
Temporary root
        ↓
dd
        ↓
Stock boot backup
```

The `dd` backup must also happen **before Magisk Direct Install**.

Otherwise, the partition may already contain the patched/persistently modified boot and the backup would no longer represent the original stock boot.

---

# 18. Preserve the stock backup

Keep these files on the PC:

```text
boot_oos11-stock.img
boot_b-stock.img
magisk_patched-xxxxx.img
```

Their roles are different:

| File | Purpose |
|---|---|
| `boot_oos11-stock.img` | Stock boot extracted from the matching firmware |
| `boot_b-stock.img` | Stock boot backed up from the tested device |
| `magisk_patched-xxxxx.img` | Magisk-patched image used for temporary testing |

The original stock files should not be modified.

---

# 19. Persistent Magisk installation

Only after:

- the patched boot has successfully booted,
- Android starts normally,
- `su` works,
- and the stock boot backup has been completed,

open Magisk:

```text
Install
    ↓
Direct Install (Recommended)
```

Allow Magisk to complete the installation.

Then reboot:

```bash
adb reboot
```

Wait for Android to start normally.

---

# 20. Verify persistent root

Check:

```bash
adb shell su -c id
```

Expected:

```text
uid=0(root)
```

Check Magisk:

```bash
adb shell su -c 'magisk -c'
```

Check the active slot:

```bash
adb shell getprop ro.boot.slot_suffix
```

Check the firmware:

```bash
adb shell getprop ro.build.display.id
adb shell getprop ro.product.model
adb shell getprop ro.build.version.release
```

Reference values:

```text
OnePlus7TOxygen_14.E.35_GLO_0350_2206171459
OnePlus7T
11
```

---

# 21. Complete safety flow

The complete tested workflow is:

```text
┌───────────────────────────────────────────────┐
│ 1. Identify exact model / build / slot       │
└──────────────────────┬────────────────────────┘
                       ↓
┌───────────────────────────────────────────────┐
│ 2. Obtain matching OxygenOS firmware         │
└──────────────────────┬────────────────────────┘
                       ↓
┌───────────────────────────────────────────────┐
│ 3. Extract stock boot.img                    │
└──────────────────────┬────────────────────────┘
                       ↓
┌───────────────────────────────────────────────┐
│ 4. Verify build + calculate SHA-256          │
└──────────────────────┬────────────────────────┘
                       ↓
┌───────────────────────────────────────────────┐
│ 5. Patch stock boot with Magisk              │
└──────────────────────┬────────────────────────┘
                       ↓
┌───────────────────────────────────────────────┐
│ 6. Retrieve magisk_patched image             │
└──────────────────────┬────────────────────────┘
                       ↓
┌───────────────────────────────────────────────┐
│ 7. fastboot boot  ← TEMPORARY TEST            │
└──────────────────────┬────────────────────────┘
                       ↓
┌───────────────────────────────────────────────┐
│ 8. Android boots + su = uid=0(root)          │
└──────────────────────┬────────────────────────┘
                       ↓
┌───────────────────────────────────────────────┐
│ 9. dd → stock boot backup                    │
└──────────────────────┬────────────────────────┘
                       ↓
┌───────────────────────────────────────────────┐
│ 10. Magisk Direct Install                    │
└──────────────────────┬────────────────────────┘
                       ↓
┌───────────────────────────────────────────────┐
│ 11. Reboot + verify persistent root          │
└───────────────────────────────────────────────┘
```

---

# 22. The most important rule

Do not change this order casually:

```text
STOCK BOOT
    ↓
MAGISK PATCH
    ↓
FASTBOOT BOOT
    ↓
TEMPORARY ROOT
    ↓
DD STOCK BACKUP
    ↓
MAGISK DIRECT INSTALL
```

In particular, avoid jumping directly from:

```text
boot.img
    ↓
fastboot flash boot
```

without first verifying the exact firmware and testing the patched image.

The direct-flash path used during the reference testing resulted in Qualcomm CrashDump. The documented procedure therefore uses temporary boot as the validation stage.

This does not guarantee that every possible boot image will work; it provides an additional compatibility check before persistent modification.

---

# 23. Recovery principle

If a patched boot does not work, do not randomly try another `boot.img`.

First return to the known information:

```text
Model
Build
Android version
OTA version
Active slot
Boot image source
SHA-256
```

A recovery image should correspond to the appropriate firmware/build and slot.

Do not assume that a boot image from another OxygenOS version is a valid recovery image.

---

# 24. Privacy when publishing logs

When contributing information to a public repository, do not publish complete raw logs if they contain unnecessary personal or device-specific information.

Remove or redact:

```text
ADB device serial / identifier
Personal usernames
Home-directory paths
Private IP addresses
Google/account information
Unrelated system logs
Unique device identifiers
```

Keep only the technical information required to reproduce the procedure:

```text
Model
Device
Android version
OxygenOS build
OTA version
Incremental
Fingerprint
Active slot
Kernel version
SHA-256
Relevant error messages
```

The reference documentation intentionally does not require publishing the complete device log.

---

# 25. Reference device information

The real device log used for this documentation reported:

```text
Model:              HD1903
Product:            OnePlus7T
Product name:       OnePlus7T_EEA
Android:            11
Build:              HD1903_14_220617
Incremental:        2206171329
OTA:                OnePlus7TOxygen_14.E.35_GLO_0350_2206171459
Kernel ID:          4.14-G2206171459
```

The device codename was:

```text
hotdogb
```

The tested active slot was:

```text
_b
```

These values identify the reference environment only. Users should collect their own values before starting.

---

# 26. Validation results and limitations

The reference procedure successfully demonstrated:

- exact firmware identification;
- extraction of a stock boot image from the corresponding firmware;
- Magisk patching;
- temporary boot with `fastboot boot`;
- temporary root;
- extraction of the device's stock boot with `dd`;
- persistent Magisk installation.

However, the real device log also showed a separate WLAN module compatibility problem:

```text
wlan: disagrees about version of symbol module_layout
```

The device contained:

```text
qca_cld3_wlan.ko
```

Therefore, successful boot/root validation should **not** be interpreted as proof that every NetHunter kernel module or Wi-Fi feature is functional.

Boot compatibility and NetHunter feature compatibility are separate validation targets.

---

# 27. Troubleshooting checklist

## `adb devices` does not show the phone

Check:

- USB debugging;
- USB cable;
- ADB installation;
- authorization prompt on the phone.

---

## The build does not match the reference

Do not use the reference boot image.

Extract a boot image from the firmware corresponding to the installed build.

---

## `fastboot boot` does not start Android

Stop the procedure.

Do not immediately use:

```bash
fastboot flash boot ...
```

Re-check:

```text
Model
Build
Firmware source
Boot image
SHA-256
Active slot
```

---

## Android starts but `su` does not work

Do not perform Direct Install yet.

The temporary validation stage has not succeeded.

---

## Qualcomm CrashDump appears

Return to the last known valid state and re-check the boot image source and exact firmware/build relationship.

Do not randomly replace the image with a boot image from another build.

The reference experience is the reason this guide uses:

```bash
fastboot boot
```

as a test before persistent installation.

---

# 28. Reproducibility checklist

Before starting:

```text
[ ] Model identified
[ ] Device codename identified
[ ] Android version identified
[ ] Exact OxygenOS build identified
[ ] OTA version identified
[ ] Incremental version recorded
[ ] Active slot recorded
[ ] Matching firmware obtained
```

Before Magisk patching:

```text
[ ] boot.img extracted from matching firmware
[ ] SHA-256 calculated
[ ] Stock image preserved
```

Before persistent installation:

```text
[ ] fastboot boot succeeded
[ ] Android booted normally
[ ] su -c id → uid=0(root)
[ ] Magisk works
[ ] Active slot verified
[ ] Stock boot backed up with dd
[ ] Backup copied to PC
[ ] Backup SHA-256 calculated
```

After installation:

```text
[ ] Android boots normally
[ ] su -c id → uid=0(root)
[ ] Magisk version verified
[ ] Active slot verified
[ ] Original stock boot preserved
```

---

# 29. Short version

For users who already understand ADB, Fastboot and Magisk:

```bash
# Identify the device
adb shell getprop ro.product.model
adb shell getprop ro.build.display.id
adb shell getprop ro.build.version.incremental
adb shell getprop ro.build.version.ota
adb shell getprop ro.boot.slot_suffix

# Extract boot from the matching OxygenOS firmware
python3 payload_dumper.py --partitions boot payload.bin

# Patch with Magisk on the phone
# Magisk → Install → Select and Patch a File

# Retrieve the patched image
adb pull /sdcard/Download/magisk_patched-xxxxx.img .

# Test temporarily — DO NOT FLASH
adb reboot bootloader
fastboot getvar current-slot 2>&1
fastboot boot magisk_patched-xxxxx.img

# Verify temporary root
adb shell su -c id

# After temporary root succeeds, back up stock boot
adb shell getprop ro.boot.slot_suffix
adb shell su -c 'dd if=/dev/block/bootdevice/by-name/boot_b of=/sdcard/boot_b-stock.img'
adb pull /sdcard/boot_b-stock.img .

# Only now:
# Magisk → Install → Direct Install (Recommended)

adb reboot

# Verify persistent root
adb shell su -c id
adb shell su -c 'magisk -c'
```

> Replace `boot_b` with the partition corresponding to the **actual active slot**. The `_b` value above is specific to the reference test device.

---

# 30. Contribution scope

This documentation is intended to help OnePlus 7T HD1903 users who cannot find a compatible stock boot image.

The contribution does not require publishing a personal device dump.

The useful contribution is the reproducible method:

```text
Exact build identification
        +
Matching firmware
        +
Stock boot extraction
        +
Magisk patch
        +
Temporary validation
        +
Stock boot backup
        +
Persistent installation
```

This approach makes the procedure easier to reproduce while keeping personal device information out of the public documentation.

---

## Final reference flow

```text
HD1903
  ↓
Identify exact OxygenOS build
  ↓
Find matching firmware
  ↓
Extract stock boot.img
  ↓
Verify build + SHA-256
  ↓
Patch with Magisk
  ↓
fastboot boot
  ↓
Temporary root
  ↓
dd stock boot backup
  ↓
Magisk Direct Install
  ↓
Persistent root
```

**Do not skip the validation stage.**

**Do not assume another boot image is compatible.**

**Do not publish private device identifiers with debugging logs.**
