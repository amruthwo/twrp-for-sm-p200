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

**Correction, 2026-07-26**: an earlier version of this file used the raw
8388608-byte stock DTBO *partition* dump directly, reasoning that reusing the
whole thing was safer than hand-parsing the packed-DTBO format. That reasoning
was based on a false premise — the file wasn't actually a multi-entry image at
all. Its own header (`dt_table_header`, see AOSP `system/libufdt/include/dt_table.h`)
declares `dt_entry_count: 1` and `total_size: 88976` bytes; everything from
byte 88976 to the 8MB partition-size boundary is Samsung's own AVB signing
footer (`SignerVer02`, `avbtool 1.1.0`, build/CSC strings — confirmed via
`strings`), completely irrelevant to a build with `BOARD_AVB_ENABLE := false`.
This was discovered by comparing byte-for-byte against `xuanyayi/twrp-for-sm-p205`'s
actual release artifact (`v3.7.1_12-0-p205-20260614-150146`), whose
`recovery_dtbo` is 88644 bytes — nearly identical real size once padding is
removed. The 8MB-vs-88KB gap accounted for **8299964 of the 8529920-byte total
gap** between our first failing build and their working one; the recovery
ramdisk (ICU included) was essentially the same size on both sides the whole
time.

Now uses `head -c 88976 dtbo.img`, i.e. the real DTBO content with the
irrelevant trailing partition padding/AVB footer dropped, not a hand-extracted
or reconstructed entry:

```
sha256: 2e9db0776d0a6211b86695d46b2447dca9ddd1e50ca34ab527935db63fb3ea8f
size:   88976 bytes
```

Contains (confirmed via `strings`) the entry:

```
samsung,WISDOMWIFI CHN OPEN 01
Samsung WISDOMWIFI CHN OPEN 01 board based on EXYNOS7904
```

## Why not just reuse the P205 tree's prebuilt binaries

`wisdom` (P205, LTE) and `wisdomwifi` (P200, Wi-Fi) share the `universal7904`
kernel source tree and (per upstream `xuanyayi/android_kernel_samsung_universal7904`)
the same `wisdom_defconfig` — there's no separate `wisdomwifi_defconfig`. But
they are still different boards with their own DTBO entries, and the actual
compiled kernel Images differ in size as measured above. Since this project's
whole premise is "ground claims in dumped files, not assumption," and we
already have this exact unit's own verified firmware sitting right there, using
it directly is strictly safer than assuming P205's prebuilts are interchangeable.
