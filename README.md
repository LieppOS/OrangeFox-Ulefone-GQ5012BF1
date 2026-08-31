# OrangeFox Recovery for Ulefone Armor 29 Pro Thermal

Unofficial OrangeFox Recovery for the **Ulefone Armor 29 Pro Thermal (GQ5012BF1)**.

## Device

| | |
|---|---|
| Device | Ulefone Armor 29 Pro Thermal |
| Product | `GQ5012BF1` |
| SoC | MediaTek MT6878 |
| Architecture | ARM64 |
| Stock Android | Android 15 |
| Tested OS | Android 16 GSI |
| Recovery | OrangeFox 14.1 |
| Boot layout | A/B + Virtual A/B |
| Recovery location | `vendor_boot` v4 |

## Status

### Working

- Boot
- Touch
- Display
- Brightness
- Vibration
- SELinux enforcing
- ADB
- MTP
- Fastbootd
- FBE decryption
- PIN decryption
- Internal storage
- External SD / exFAT
- Battery percentage
- Charging status
- Battery temperature
- Dynamic partition mounts
- Slot detection
- Backup
- RTC / logs
- Bootloader reboot target
- Power-off target
- System reboot target


### Not fully tested

- USB OTG
- ADB sideload
- Restore
- Screenshot


## Download

Flashable builds are available under:

**[GitHub Releases](https://github.com/LieppOS/OrangeFox-Ulefone-GQ5012BF1/releases)**

Current release:

**Build36**

Always verify the SHA256 checksum before flashing.

## Installation

> [!WARNING]
> This device does **not** have a standalone recovery partition.
> OrangeFox is installed inside `vendor_boot_a`.

Boot into fastbootd:

```bash
adb reboot fastboot
```

Wait until the device enters fastbootd and verify it is detected:

```bash
fastboot devices
```

Flash OrangeFox:

```bash
fastboot flash vendor_boot_a vendor_boot_a-orangefox-FULL64M-BUILD36.img
```

Reboot directly into recovery:

```bash
fastboot reboot recovery
```

Done! Enjoy.

## Building

### 1. Prepare OrangeFox 14.1 source

Create a working directory:

```bash
mkdir -p ~/android/fox_14.1
cd ~/android/fox_14.1
```

Initialize and sync the OrangeFox 14.1 source tree using the appropriate OrangeFox manifest for this branch.

> A complete OrangeFox source checkout is required. This repository only contains the device-specific files.

### 2. Clone the device tree

```bash
cd ~/android/fox_14.1

git clone https://github.com/LieppOS/android_device_ulefone_gq5012bf1.git \
    device/ulefone/gq5012bf1
```

### 3. Prepare the vendor_boot template

This device boots recovery from `vendor_boot` v4.

The generated Android build output cannot be flashed directly because the stock:

- PLATFORM vendor ramdisk
- DTB

must be preserved.

You therefore need a stock-derived repack template:

```text
vendor_boot-repack-template-raw.img
```

Set its location:

```bash
export GQ_TEMPLATE=/absolute/path/to/vendor_boot-repack-template-raw.img
```

Optionally choose where packaged builds are written:

```bash
export GQ_ARTIFACTS=$HOME/android/gq5012bf1-artifacts
```

### 4. Initialize the build environment

```bash
cd ~/android/fox_14.1

unset OUT OUT_DIR OUT_DIR_COMMON_BASE
unset LEX YACC M4 BISON FLEX

export OUT_DIR="$PWD/out"

source build/envsetup.sh
lunch twrp_gq5012bf1-ap2a-eng
```

### 5. Build and package

```bash
./device/ulefone/gq5012bf1/build-gq5012bf1.sh 37 full
```

The helper:

1. builds the OrangeFox recovery ramdisk
2. inserts it into a full vendor_boot v4 image
3. preserves the stock PLATFORM ramdisk
4. preserves the stock DTB
5. adds the AVB hash footer
6. verifies the final image

### 6. Expected output

The final image will be written under your artifacts directory, for example:

```text
~/android/gq5012bf1-artifacts/build37/
vendor_boot_a-orangefox-FULL64M-BUILD37.img
```

The final image must be exactly:

```text
67108864 bytes
```

The helper should also verify:

```text
stock PLATFORM SHA256:
9201a4e5c1b7cb1fc0ce35375af10a3d966dac8b84615a226f98b7d7be2aec00

stock DTB SHA256:
bc156c29c33d8226230f07888df0a3d7a1e9c4b85c5fd550a4c4bd1a3134c0d4
```

### 7. Flash the result

From Android or recovery:

```bash
adb reboot fastboot
```

Wait for fastbootd:

```bash
fastboot devices
```

Then flash:

```bash
fastboot flash vendor_boot_a \
    ~/android/gq5012bf1-artifacts/build37/vendor_boot_a-orangefox-FULL64M-BUILD37.img

fastboot reboot recovery
```

## Disclaimer

This is an unofficial OrangeFox port.

You are responsible for anything you flash to your device.

Keep a copy of the original stock `vendor_boot.img` before installing or testing custom recovery.
