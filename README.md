# OrangeFox recovery — Ulefone Armor 29 Pro Thermal (GQ5012BF1)

Export taken 2026-08-31 from device tree commit `2d9765f`.

```text
Device      Ulefone Armor 29 Pro Thermal
Product     GQ5012BF1
SoC         MediaTek MT6878
Recovery    OrangeFox 14.1 (Android 14 base)
Vendor      stock Android 15
Active OS   Android 16 GSI
Upstream    git@github.com:LieppOS/android_device_ulefone_gq5012bf1.git (master)
```

---

## Contents

```text
images/            flashable vendor_boot images + repack template + SHA256SUMS.txt
device_tree/       full device tree at commit 407d8ec (clean git export)
docs/              README, TODO, reports, Findings
tools/             vendor_boot pack/unpack + platform patches (reference copies)
stock_reference/   stock TrustKernel HAL binaries and rc files
```

The canonical build helper is `device_tree/build-gq5012bf1.sh`, which expects
`tools/` beside it. Use that one, not a copy moved elsewhere, or the helper
paths will not resolve.

---

## Flashing

**Flash `vendor_boot_a` only.** Never flash
`out/target/product/gq5012bf1/vendor_boot.img` from a build tree — it lacks the
stock PLATFORM ramdisk fragment and the stock DTB and will not boot.

From recovery:

```bash
adb reboot fastboot
# wait until 'fastboot devices' lists the device
fastboot flash vendor_boot_a images/vendor_boot_a-orangefox-FULL64M-BUILD36.img
fastboot reboot recovery
```

Never erase partitions, never switch to slot B, and do not touch vbmeta.

### Which image

| image | notes |
|---|---|
| **BUILD36** | current. Everything below plus working vibration. |
| BUILD34 | rollback. Same as 36 without the vibrator patch. |
| BUILD32 | rollback. Predates MTP; use if USB composition ever misbehaves. |

Checksums are in `images/SHA256SUMS.txt`. Every image is exactly 67108864 bytes
with an AVB `NONE` footer on partition `vendor_boot`.

---

## What works

Verified on hardware: cold boot; SELinux **enforcing**; Android 16 `/data`
decrypting from a single PIN with no ADB intervention; battery percentage,
charging state and temperature updating live; touch; display; ADB shell, push
and pull; MTP alongside ADB; fastbootd; internal storage; external SD (exFAT);
read-only mounts of `system`, `system_ext`, `product`, `vendor`; slot display;
backup; logs; RTC; vibration.

## What is NOT verified

USB OTG, ADB sideload, brightness slider, screenshot, restore, and the System /
Bootloader / Power off reboot targets. Zip flashing, Magisk, `/data`
backup-restore and theming were never in scope.

There are also three known cosmetic or structural defects. **Read
`docs/TODO.md` before trusting any feature not listed as verified.**

Of note: USB drops for roughly 40 seconds immediately after the PIN is entered.
This is expected — adding the MTP function to a live USB gadget requires
rebinding the UDC, so the host re-enumerates. It recovers on its own.

---

## Rebuilding

### This folder is NOT self-sufficient for building

It contains the device-specific half. You also need, and it is **not** included
because of its size:

- **A full OrangeFox 14.1 source tree** (roughly 100 GB synced, plus build
  output). That is the actual build system; this folder only supplies the parts
  unique to this device.

Everything device-specific *is* here: the device tree, the prebuilt kernel, DTB
and dtbo, all four platform patches, the pack/unpack tools, and the repack
template.

### Steps

Place the device tree at `device/ulefone/gq5012bf1` inside an OrangeFox 14.1
tree, then:

```bash
cp -r device_tree /path/to/fox_14.1/device/ulefone/gq5012bf1
cd /path/to/fox_14.1
./device/ulefone/gq5012bf1/build-gq5012bf1.sh 37 full
```

If the artifacts directory from the original machine is absent, point the helper
at the template shipped here, otherwise packaging fails with
`template vendor_boot not found`:

```bash
export GQ_TEMPLATE=/path/to/orangefox_recovery/images/vendor_boot-repack-template-raw.img
export GQ_ARTIFACTS=/path/to/wherever/you/want/output
```

The template is a full vendor_boot v4 carrying the stock PLATFORM ramdisk
fragment and stock DTB. Only its RECOVERY fragment is replaced during packaging,
which is how those two stock blobs survive byte for byte.

The former `PRODUCT_SHIPPING_API_LEVEL := 35` conflict is not present in this
snapshot; Build36 compiled successfully. Keep ROM-only shipping API settings
outside `twrp_%` products if they are reintroduced later.

The helper builds the recovery fragment, splices it into a full vendor_boot v4
preserving the stock PLATFORM fragment and DTB, adds the AVB footer, verifies
both invariants and the exact size, and prints the artifact path and SHA256. It
never flashes.

`vendorsetup.sh` applies the platform patches in `tools/patches/` automatically:

```text
system_sepolicy/0001    recovery reads the vold metadata key
bootable_recovery/0001  ramdisk requires vendor property contexts
bootable_recovery/0002  MTP skips legacy USB when FunctionFS is present
bootable_recovery/0003  vibrate supports a brightness-only LED vibrator
```

### Invariants

```text
partition size    67108864
stock PLATFORM    9201a4e5c1b7cb1fc0ce35375af10a3d966dac8b84615a226f98b7d7be2aec00
stock DTB         bc156c29c33d8226230f07888df0a3d7a1e9c4b85c5fd550a4c4bd1a3134c0d4
AVB               algorithm NONE, partition vendor_boot
```

---

## Background

The hard problem was FBE. Three things had to be true at once: TrustKernel needed
an SELinux `link` permission that `create_file_perms` does not grant, it needed
to traverse its own persistent-store mount roots, and the Gatekeeper and KeyMint
trusted-application sessions had to be **serialised** rather than started
together. Full analysis is in `docs/CLAUDE_FBE_REPORT.md`, with bring-up history
in `docs/Findings.md`.
