# TWRP Device Tree for Motorola edge 30 neo (miami).

## Setup repo tool
Setup repo tool from here https://source.android.com/setup/develop#installing-repo

## Compile

Sync TWRP manifest:

```
repo init --depth=1 -u https://github.com/minimal-manifest-twrp/platform_manifest_twrp_aosp.git -b twrp-12.1
```

Make a directory named `local_manifest` under the existing subdirectory `.repo`, and create a new manifest file, for example `local_manifests.xml`
and then paste the following

```xml
<?xml version="1.0" encoding="UTF-8"?>
<manifest>
<project path="device/motorola/miami"
	name="ast261/twrp_device_motorola_miami"
	remote="github"
	revision="android-12.1" />
</manifest>
```
You might need to pick few patches from gerrit.twrp.me to get some stuff working.

Sync the sources with

```
repo sync -j$(nproc --all)
```

To build, execute this command:

```
. build/envsetup.sh; lunch twrp_miami-eng; mka bootimage
```

To test it:

```
# Temporarily boot TWRP (loaded into RAM only, gone after a reboot).
# This is non-destructive and leaves the installed OS bootable, so it is
# the recommended way to use TWRP on this device.
fastboot boot out/target/product/miami/boot.img
```

### Making TWRP persistent (and why you usually shouldn't)

This device is **recovery-as-boot** (`BOARD_USES_RECOVERY_AS_BOOT`,
`TARGET_NO_RECOVERY`): there is no separate `recovery` partition. The TWRP
ramdisk lives inside the **boot image**, which is the same partition that
holds the OS (e.g. LineageOS) boot image.

If you boot TWRP and use **Advanced → Flash Current TWRP**, it writes the
running TWRP image to the **`boot` partition of the active slot**. Because
boot and recovery share that partition, this **overwrites the OS boot
image**: the device will then boot into TWRP instead of the OS, and you must
re-flash the OS `boot.img` to boot the system again.

So on this device there is no way to keep a persistent standalone TWRP
without sacrificing normal OS boot on that slot. Prefer `fastboot boot`
above; only use Flash Current TWRP if you deliberately want the device to
land in TWRP and you keep the OS `boot.img` handy to restore it.

## Copyright

```
#
# Copyright (C) 2024-2026 The Android Open Source Project
#
# SPDX-License-Identifier: Apache-2.0
#
