# Minimal Pixel 2 (XL) manifest

## Purpose

Currently, the state of TWRP makes using AnyKernel2 very impractical. Additionally, AOSP can consume a lot of space, even with a `--depth=1` flag. With this manifest, syncing and building are quicker as there are less files for ninja and soong to parse. Win win!

Size comparison (done with android-8.0.0_r33):

| Manifest | `--clone-depth=1`? | Size |
| -------- | ------------------ | ---- |
| AOSP     | No                 | 55GB |
| AOSP     | Yes                | 41GB |
| Minimal  | N/A                | 16GB |

## Usage

1. Sync the manifest
```bash
repo init -u https://github.com/nathanchance/pixel2-manifest -b 8.0.0
repo sync -j$(nproc --all)
```

2. Add your kernel binary (Image.lz4-dtb) to the prebuilts location (device/google/wahoo-kernel). Follow the below steps to do this if you don't know how.

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

## Building the kernel image

The Pixel 2 has two oddities when it comes to building the kernel: Clang and dtbo. Clang is just another compiler like GCC and dtbo stands for device tree blob overlay, which allows Google to keep the kernel source for the two Pixels unified. However, to build it, you need two tools from AOSP (dtc and mkdtimg). Here are the step by step instructions to build the kernel!

1. Sync the manifest using step 1 above.

2. Generate the dtc and mkdtimg files (these are host tools so lunch isn't necessary)
```bash
. build/envsetup.sh
make -j$(nproc --all) dtc mkdtimg
export AOSP_FOLDER=$(pwd)
```

3. Copy these files to somewhere in your path (like /usr/local/bin) or run `export PATH=out/host/linux-x86/bin:${PATH}`

4. Download the kernel source from Google:
```bash
git clone -b android-msm-wahoo-4.4-oreo-dr1 https://android.googlesource.com/kernel/msm ../wahoo-kernel
```

5. Start building the kernel!
```bash
cd ../wahoo-kernel
make -j$(nproc --all) ARCH=arm64 O=out wahoo_defconfig
make -j$(nproc --all) ARCH=arm64 CC=${AOSP_FOLDER}/prebuilts/clang/host/linux-x86/clang-3859424/bin/clang CLANG_TRIPLE=aarch64-linux-gnu- CROSS_COMPILE=${AOSP_FOLDER}/prebuilts/gcc/linux-x86/aarch64/aarch64-linux-android-4.9/bin/aarch64-linux-android- O=out
```

If successful, the kernel will be located at `out/arch/arm64/boot/Image.lz4-dtb`, which you can build and flash using the Usage section above.
