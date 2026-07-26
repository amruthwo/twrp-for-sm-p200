# TWRP for Samsung Galaxy Tab A 8.0 with S Pen (SM-P200)

This device tree builds TWRP 12.1 for the Samsung Galaxy Tab A 8.0 with S Pen
Wi-Fi (`SM-P200`). The Samsung codename for this device is `wisdomwifi`
(product `wisdomwifizh`). The LTE sibling is `SM-P205`, codename `wisdom`.

This tree is forked from `xuanyayi/twrp-for-sm-p205` (itself based on the
upstream SM-P205 TWRP device tree from `topser9/twrp_device_samsung_p205`),
adapted here for the Wi-Fi-only `SM-P200`. The prebuilt kernel (`prebuilt/Image`)
and recovery DTBO (`prebuilt/recovery_dtbo`) are pulled directly from this
exact device's stock firmware (`P200ZHS4CWA2`, CSC `TGY`), not carried over
from the P205 tree — see `DEVICE_INFO.md` for how those were extracted and
verified.

## Build Environment

Use a Linux build host. Ubuntu 20.04/22.04 or WSL2 Ubuntu is recommended.

Install common Android build packages:

```bash
sudo apt update
sudo apt install -y \
  bc bison build-essential ccache curl flex g++-multilib gcc-multilib \
  git git-lfs gnupg gperf imagemagick lib32ncurses5-dev lib32readline-dev \
  lib32z1-dev libelf-dev liblz4-tool libncurses5 libncurses5-dev \
  libsdl1.2-dev libssl-dev libxml2 libxml2-utils lzop pngcrush \
  rsync schedtool squashfs-tools unzip xsltproc zip zlib1g-dev
```

Install the `repo` tool if it is not already available:

```bash
mkdir -p ~/.bin
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/.bin/repo
chmod a+x ~/.bin/repo
export PATH="$HOME/.bin:$PATH"
```

For WSL2, build inside the Linux filesystem, for example under `~/twrp`.
Building under `/mnt/c` or another Windows-mounted path is much slower and can
cause file permission issues.

## Sync TWRP Source

```bash
mkdir -p ~/twrp
cd ~/twrp

repo init --depth=1 \
  -u https://github.com/minimal-manifest-twrp/platform_manifest_twrp_aosp.git \
  -b twrp-12.1

repo sync -c --no-tags --no-clone-bundle --optimized-fetch --fail-fast -j"$(nproc)"
```

If the sync fails because of network instability, rerun the same `repo sync`
command. On very unstable networks, use `-j1`.

## Clone Device Tree

Clone this repository into the Android source tree exactly at
`device/samsung/p200`:

```bash
cd ~/twrp
rm -rf device/samsung/p200
git clone git@github.com:amruthwo/twrp-for-sm-p200.git device/samsung/p200
```

The expected device tree files include:

```text
device/samsung/p200/AndroidProducts.mk
device/samsung/p200/BoardConfig.mk
device/samsung/p200/twrp_p200.mk
device/samsung/p200/prebuilt/Image
device/samsung/p200/prebuilt/recovery_dtbo
```

## Apply Required Patches

This device tree depends on a few small TWRP 12.1 base-tree patches (generic
AOSP/TWRP-core fixes, not device-specific — inherited as-is from the P205
tree) for the recovery runtime. Apply them from the Android source root before
building:

```bash
cd ~/twrp
P200_TREE="$PWD/device/samsung/p200"

git -C build/make apply "$P200_TREE/patches/0001-build-make-p205-recovery-props.patch"
git -C system/sepolicy apply "$P200_TREE/patches/0002-system-sepolicy-drop-deprecated-board-plat-policy.patch"
git -C bootable/recovery apply "$P200_TREE/patches/0003-bootable-recovery-p205-twrp12-support.patch"
git -C system/tools/mkbootimg apply "$P200_TREE/patches/0004-mkbootimg-preserve-empty-second-address.patch"
git -C vendor/twrp apply "$P200_TREE/patches/0005-vendor-twrp-export-p205-soong-vars.patch"
```

If you want to verify first without changing files:

```bash
cd ~/twrp
P200_TREE="$PWD/device/samsung/p200"

git -C build/make apply --check "$P200_TREE/patches/0001-build-make-p205-recovery-props.patch"
git -C system/sepolicy apply --check "$P200_TREE/patches/0002-system-sepolicy-drop-deprecated-board-plat-policy.patch"
git -C bootable/recovery apply --check "$P200_TREE/patches/0003-bootable-recovery-p205-twrp12-support.patch"
git -C system/tools/mkbootimg apply --check "$P200_TREE/patches/0004-mkbootimg-preserve-empty-second-address.patch"
git -C vendor/twrp apply --check "$P200_TREE/patches/0005-vendor-twrp-export-p205-soong-vars.patch"
```

If a patch reports that it is already applied, skip that patch.

## Build

```bash
cd ~/twrp
export ALLOW_MISSING_DEPENDENCIES=true
export BUILD_DATETIME=1781100000
export BUILD_NUMBER=p200-repro-20260726
export BUILD_USERNAME=p200
export BUILD_HOSTNAME=repro
source build/envsetup.sh
lunch twrp_p200-eng
mka recoveryimage
```

The output image is:

```text
out/target/product/p200/recovery.img
```

For a clean rebuild:

```bash
cd ~/twrp
rm -rf out/target/product/p200 out/soong/build_number.txt

export ALLOW_MISSING_DEPENDENCIES=true
export BUILD_DATETIME=1781100000
export BUILD_NUMBER=p200-repro-20260726
export BUILD_USERNAME=p200
export BUILD_HOSTNAME=repro
source build/envsetup.sh
lunch twrp_p200-eng
mka recoveryimage
```

## Odin Package

To flash with Odin, pack `recovery.img` into a tar archive:

```bash
cd /path/to/twrp-12.1/out/target/product/p200
tar --sort=name \
  --mtime='@1781100000' \
  --owner=0 --group=0 --numeric-owner \
  -H ustar \
  -cf twrp-3.7.1_12-0-p200.tar \
  recovery.img
```

Flash `twrp-3.7.1_12-0-p200.tar` with Odin's AP slot.

In Odin, disable `Auto Reboot` before flashing. After Odin reports `PASS`, do
not let the tablet boot Android first. Hold `Power + Volume Down` to leave
Download Mode; as soon as the screen turns off, switch to `Power + Volume Up`
and keep holding until recovery starts.

## Known Build Notes

- Correct manifest branch: `twrp-12.1`
- Correct device path: `device/samsung/p200`
- Correct lunch target: `twrp_p200-eng`
- Recovery kernel: `prebuilt/Image` — extracted from this device's own stock
  `boot.img` (`P200ZHS4CWA2`), not carried over from the P205 tree
- Recovery DTBO: `prebuilt/recovery_dtbo` — the full stock DTBO partition
  image from this device's own firmware (contains the `wisdomwifizh`/
  `wisdomwifi_chn_open` entries; the bootloader selects the matching one by
  `board_id`/`board_rev` the same way it does for normal Android boot)
- Recovery partition size: `39845888` bytes
- `ALLOW_MISSING_DEPENDENCIES=true` is expected for this minimal recovery tree.
- Reproducible release builds use `BUILD_DATETIME=1781100000`,
  `BUILD_NUMBER=p200-repro-<date>`, `BUILD_USERNAME=p200`, and
  `BUILD_HOSTNAME=repro`.
- Optional TWRP extras are trimmed to keep `recovery.img` inside the stock
  recovery partition size.
- The inherited P205 bootable/recovery patch includes a forced USB disconnect
  and minadbd handoff so ADB sideload enumerates on Windows even when `/data`
  is encrypted and MTP is not running. Not P205-specific in practice — this is
  a generic AOSP/TWRP-core fix.

## Common Problems

### `lunch twrp_p200-eng` is not listed

Check that the device tree is in the correct path:

```bash
ls device/samsung/p200/AndroidProducts.mk
```

If the file is missing, clone this repository again into `device/samsung/p200`.

### `vendor/twrp/config/common.mk` not found

The wrong manifest was synced. Reinitialize and sync the TWRP 12.1 AOSP
manifest:

```bash
repo init --depth=1 \
  -u https://github.com/minimal-manifest-twrp/platform_manifest_twrp_aosp.git \
  -b twrp-12.1
repo sync -c --no-tags --no-clone-bundle --optimized-fetch --fail-fast -j"$(nproc)"
```

### Patch application fails

Make sure the patches are applied from the Android source root, not from inside
`device/samsung/p200`:

```bash
cd ~/twrp
```

If the source tree was synced from a different TWRP branch, re-sync `twrp-12.1`.

### `recovery.img` is larger than the recovery partition

Do not enable extra languages, NTFS/exFAT userspace tools, repack tools, bash,
nano, tzdata, or zip unless you also verify the final image size. The stock
recovery partition limit for this device is `39845888` bytes.

Check the image size with:

```bash
stat -c '%n %s bytes' out/target/product/p200/recovery.img
```

### `mka: command not found`

Load the Android build environment first:

```bash
source build/envsetup.sh
```

Then run `lunch twrp_p200-eng` again before building.

## Device Notes

- Device: Samsung Galaxy Tab A 8.0 with S Pen (Wi-Fi)
- Model: `SM-P200`
- Samsung codename: `wisdomwifi` (product `wisdomwifizh`)
- Platform: Exynos 7904 / universal7904
- Device tree path: `device/samsung/p200`
- Recovery variant: TWRP 12.1
- TWRP version from this branch: `3.7.1_12-0`
