# Minimal Pixel 2 (XL) manifest

## Purpose

Currently, the state of TWRP makes using AnyKernel2 very impractical. Additionally, AOSP can consume a lot of space, even with a `--depth=1` flag. With this manifest, syncing and building are quicker as there are less files for ninja and soong to parse. Win win!

Size comparison (done with android-8.0.0_r33):

| Manifest | `--clone-depth=1`? | Size |
| -------- | ------------------ | ---- |
| AOSP     | No                 | 55GB |
| AOSP     | Yes                | 41GB |
| Minimal  | N/A                | 18GB |

## Usage

1. Sync the manifest
```bash
repo init -u https://github.com/nathanchance/pixel2-manifest -b 8.0.0
repo sync -j$(nproc --all)
```

2. Add your kernel binary (Image.lz4-dtb) to the prebuilts location (device/google/wahoo-kernel).

3. Lunch and build!

Pixel 2:
```bash
. build/envsetup.sh
lunch aosp_walleye-user
make -j$(nproc --all) clean
make -j$(nproc --all) bootimage
```

Pixel 2 XL:
```bash
. build/envsetup.sh
lunch aosp_taimen-user
make -j$(nproc --all) clean
make -j$(nproc --all) bootimage
```
