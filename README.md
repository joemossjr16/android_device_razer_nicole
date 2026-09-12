# TWRP Device Tree for Razer Edge 5G (nicole / RZ45-0460)

```
#
# Copyright (C) 2026 The LineageOS Project
# Copyright (C) 2026 TeamWin Recovery Project
#
# Licensed under the Apache License, Version 2.0 (the "License");
#
```

## Device Specifications
| Feature | Specification |
| :--- | :--- |
| **SoC** | Qualcomm Snapdragon G3x Gen 1 (SM8350 / lahaina) |
| **CPU** | 8x Kryo cores (up to 3.0 GHz) |
| **GPU** | Adreno 660 |
| **Memory** | 8 GB LPDDR5 |
| **Display** | 6.8" AMOLED 2400 x 1080 (20:9), 144Hz |
| **Storage** | 128 GB UFS 3.1 + MicroSD slot |
| **Battery** | 5000 mAh |
| **Platform** | LineageOS 23.2 (Android 16) |

---

## Features & Working Status
- **Touch & Display**: Landscape orientation, adjusted status bar and margins for rounded corners.
- **Decryption**:
  - Full FBE v2 / metadata encryption decryption supported via Qualcomm Keymaster 4.x.
  - Dynamic security patch extraction from installed ROM (`system/build.prop`) to prevent TrustZone rollback lockouts.
  - Tested and working on LineageOS 23.2 (Android 16).
- **Partitions & Storage**: Internal Storage (`/sdcard`) and external MicroSD supported.
- **A/B Slot Support**: Fastboot bootable (`fastboot boot boot.img`) and flashable zip installer with `addon.d` survival across ROM updates.

---

## Patches Required for TWRP Minimal Tree
The `patches/` folder contains patches that must be applied to the TWRP source tree:
- `bootable_recovery.patch`: UI status bar padding for rounded corners, clean version string, and crypto listener wait loops.
- `system_vold.patch`: FsCrypt and KeyStorage unwrap compatibility fixes for FBE.
- `system_security.patch`: Keystore2 / km_compat translation fixes.
- `device_qcom_twrp-common.patch`: Init triggers for qcom decrypt services.
- `device_qcom_common.patch`: gpt-utils header declarations.
- `hardware_qcom-caf_bootctrl.patch`: CAF bootctrl build flags.

To apply patches from the root of your TWRP build tree:
```bash
git -C bootable/recovery apply < device/razer/nicole/patches/bootable_recovery.patch
git -C system/vold apply < device/razer/nicole/patches/system_vold.patch
git -C system/security apply < device/razer/nicole/patches/system_security.patch
git -C device/qcom/twrp-common apply < device/razer/nicole/patches/device_qcom_twrp-common.patch
git -C device/qcom/common apply < device/razer/nicole/patches/device_qcom_common.patch
git -C hardware/qcom-caf/bootctrl apply < device/razer/nicole/patches/hardware_qcom-caf_bootctrl.patch
```

---

## How to Build

### 1. Initialize TWRP Manifest
```bash
repo init --depth=1 -u https://github.com/minimal-manifest-twrp/platform_manifest_twrp_aosp.git -b twrp-12.1
mkdir -p .repo/local_manifests
cp device/razer/nicole/manifest/nicole.xml .repo/local_manifests/
repo sync -c -j$(nproc --all) --force-sync --no-clone-bundle --no-tags
```

### 2. Apply Patches
Run the patch commands listed above.

### 3. Build Boot Image
```bash
source build/envsetup.sh
lunch twrp_nicole-eng
m bootimage -j$(nproc --all)
```
The resulting TWRP image will be generated at `out/target/product/nicole/boot.img`.

### 4. Create Flashable Installer ZIP (with Survival Script)
```bash
python3 device/razer/nicole/installer/make_installer.py
```
This generates `out/target/product/nicole/twrp-installer-nicole.zip`.

---

## Flashing Instructions

### Temporary Boot
```bash
adb reboot bootloader
fastboot boot boot.img
```

### Permanent Installation
Flash `twrp-installer-nicole.zip` in TWRP:
- Via TWRP GUI: **Install** -> Select `twrp-installer-nicole.zip` -> Swipe to Confirm Flash.
- Via ADB: `adb shell twrp install /sdcard/twrp-installer-nicole.zip`.

This will inject the TWRP recovery ramdisk into both boot slots (`boot_a` and `boot_b`) and place an `addon.d` survival script in `/system/addon.d/` so TWRP survives ROM updates.
