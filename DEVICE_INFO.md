# Prebuilt binary provenance

Both prebuilt binaries in `prebuilt/` were replaced from this fork's P205
starting point with files pulled directly from this exact `SM-P200` unit's own
stock firmware (`P200ZHS4CWA2`, CSC `TGY`, downloaded and verified 2026-07-26
via `samloader`; see `../../../firmware/` and the project's `NOTES.md`), not
carried over from the upstream `wisdom`/P205 tree.

## `prebuilt/Image`

Kernel, extracted from this device's stock `boot.img` with `unpack_bootimg`
(part of the `android-tools` package):

```
sha256: 3af558ee654c66db9f0680636d4ff0d8535d4611c296012437dc480fa63f7a97
size:   23983352 bytes
kernel load address: 0x10008000
page size: 2048
os version: 11.0.0, patch level 2023-01
```

The stock `boot.img` this came from contains live `wacom_i2c_*` driver symbols
(`wacom_i2c_init/send/recv/query`, `wacom_usb_typec_*`) — confirmed via
`strings`, so the S Pen driver is present in this exact kernel build.

Differs in size from the P205 tree's original `prebuilt/Image`
(23762168 bytes) — confirmed these are two distinct kernel builds, not
interchangeable by assumption.

## `prebuilt/recovery_dtbo`

The **full stock DTBO partition image** from this device's firmware (not a
single hand-extracted entry):

```
sha256: 2b986099f6c148ac71bbd4b8a18ff5d20725c9a98c03608092d0b970baf82fe5
size:   8388608 bytes (8 MiB, matches BOARD_DTBOIMG_PARTITION_SIZE)
```

Contains (confirmed via `strings`) the entry:

```
samsung,WISDOMWIFI CHN OPEN 01
Samsung WISDOMWIFI CHN OPEN 01 board based on EXYNOS7904
```

Using the full multi-entry stock image rather than hand-extracting a single
DTBO entry was a deliberate choice: the bootloader already selects the correct
entry from this exact image via `board_id`/`board_rev` every time this unit
boots stock Android, so reusing it whole avoids the risk of extracting the
wrong entry or corrupting the packed-DTBO format by hand.

## Why not just reuse the P205 tree's prebuilt binaries

`wisdom` (P205, LTE) and `wisdomwifi` (P200, Wi-Fi) share the `universal7904`
kernel source tree and (per upstream `xuanyayi/android_kernel_samsung_universal7904`)
the same `wisdom_defconfig` — there's no separate `wisdomwifi_defconfig`. But
they are still different boards with their own DTBO entries, and the actual
compiled kernel Images differ in size as measured above. Since this project's
whole premise is "ground claims in dumped files, not assumption," and we
already have this exact unit's own verified firmware sitting right there, using
it directly is strictly safer than assuming P205's prebuilts are interchangeable.
