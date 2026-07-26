# TWRP 12.1 Base-Tree Patches

Generic AOSP/TWRP-core fixes inherited from the upstream P205 tree — not
device-specific (nothing in these patches' content references p205/wisdom
hardware), so the filenames are kept as-is rather than renamed for churn's
sake. Apply these patches from the Android source root after syncing the
TWRP 12.1 manifest and cloning this device tree.

```bash
cd ~/twrp
P200_TREE="$PWD/device/samsung/p200"

git -C build/make apply "$P200_TREE/patches/0001-build-make-p205-recovery-props.patch"
git -C system/sepolicy apply "$P200_TREE/patches/0002-system-sepolicy-drop-deprecated-board-plat-policy.patch"
git -C bootable/recovery apply "$P200_TREE/patches/0003-bootable-recovery-p205-twrp12-support.patch"
git -C system/tools/mkbootimg apply "$P200_TREE/patches/0004-mkbootimg-preserve-empty-second-address.patch"
git -C vendor/twrp apply "$P200_TREE/patches/0005-vendor-twrp-export-p205-soong-vars.patch"
```

Patch summary:

- `0001-build-make-p205-recovery-props.patch`: keeps recovery USB properties
  deterministic and allows `device/samsung/p200/ramdisk.prop` to feed the
  recovery ramdisk.
- `0002-system-sepolicy-drop-deprecated-board-plat-policy.patch`: removes legacy
  `BOARD_PLAT_*_SEPOLICY_DIR` compatibility reads that produce Android 12.1
  sepolicy warnings in this tree.
- `0003-bootable-recovery-p205-twrp12-support.patch`: adds the P205 recovery
  runtime changes for Samsung configfs USB handling, encrypted-data ADB
  sideload enumeration, selected language resource copying, sdfat/exfat
  handling, and deterministic `gzip`/`gunzip` symlinks.
- `0004-mkbootimg-preserve-empty-second-address.patch`: adds the
  `--set_empty_second_addr` option required by the Samsung recovery header.
- `0005-vendor-twrp-export-p205-soong-vars.patch`: exports the P205 TWRP
  trimming and language variables to Soong.
