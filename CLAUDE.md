# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

TWRP (Team Win Recovery Project) device tree for the **Motorola edge 30 neo** (codename: `miami`). It lives at `device/motorola/miami` inside a full TWRP AOSP source tree synced from the minimal-manifest-twrp manifest (`twrp-12.1` branch).

This repository is not standalone — it must be placed inside a synced TWRP source tree to build.

## Build

```bash
# One-time setup (from AOSP root)
. build/envsetup.sh

# Build recovery/boot image
lunch twrp_miami-eng
mka bootimage
```

Output: `out/target/product/miami/boot.img`

## Flash / Test

```bash
# Boot temporarily without flashing
fastboot boot out/target/product/miami/boot.img

# To persist: boot TWRP via the above, then use Advanced → Flash Current TWRP
```

## Architecture

### Key configuration files

| File | Purpose |
|---|---|
| `BoardConfig.mk` | Hardware flags: CPU, kernel, partitions, AVB, TWRP UI settings, crypto |
| `device.mk` | Product packages: A/B OTA, boot control HAL, fastbootd, QCOM decrypt |
| `twrp_miami.mk` | Product identity (name, brand, model) and top-level inheritance |
| `AndroidProducts.mk` | Registers `twrp_miami-eng` as a lunch target |
| `recovery.fstab` | Partition mount table used by recovery |
| `system.prop` / `vendor.prop` | Runtime properties for charger, crypto, USB, storage |

### Hardware platform

- SoC: Qualcomm **holi** (SM6375), GPU: Adreno 619
- Architecture: arm64 (primary), arm (secondary)
- Boot header version 3; kernel is **prebuilt** (`prebuilt/Image`)

### A/B (VAB) specifics

The device uses **Virtual A/B** (`virtual_ab_ota.mk`). `BOARD_USES_RECOVERY_AS_BOOT := true` / `TARGET_NO_RECOVERY := true` means the recovery ramdisk is embedded in the boot image — there is no separate recovery partition.

### Encryption / decryption

QCOM FBE decryption is enabled (`BOARD_USES_QCOM_FBE_DECRYPTION := true`). The `vendor.prop` properties configure inline crypto and dm-default-key. `qcom_decrypt` and `qcom_decrypt_fbe` packages are included.

`PLATFORM_SECURITY_PATCH` is set to `2099-12-31` to prevent Android version checks from blocking decryption of newer userdata partitions.

### Prebuilt kernel modules

All `.ko` files in `prebuilt/` are loaded at recovery startup via `TW_LOAD_VENDOR_MODULES` and are copied to `recovery/root/vendor/lib/modules/1.1/`. When updating the kernel (`prebuilt/Image`), also update the matching kernel modules.

### Custom components

- `bootctrl/` — Boot control HAL implementation for the holi platform (wraps `gpt-utils`)
- `gpt-utils/` — GPT partition utilities used by boot control to manage A/B slot metadata

### TWRP UI settings (BoardConfig.mk)

- Theme: `portrait_hdpi`
- Display offsets: `TW_H_OFFSET=-115`, `TW_Y_OFFSET=115` (notch compensation)
- Brightness: path `/sys/class/backlight/panel0-backlight/brightness`, default 1024, max 3072
- CPU temp: `/sys/devices/virtual/thermal/thermal_zone50/temp`
- exFAT handled by kernel module (`exfat.ko`), not FUSE (`TW_NO_EXFAT_FUSE := true`)
