# iOS 16 XNU port for Xiaomi Mi 9T (`davinci`)

This repository's `gpu-bringup` branch adapts the open-source XNU work from
[ios-sweet](https://github.com/dipa-studio/ios-sweet) for the Xiaomi Mi 9T
(`davinci`, Qualcomm SDM730 / Adreno 618). The iOS userland used by the
upstream project is Apple's iOS 16.7.16 for iPhone X; it is not included here.

## Status

This is experimental bring-up work, not a daily-use operating system.

- The port carries shared SWEET XNU patches and davinci-specific changes for
  the GT9886 touch controller, display controller, and Adreno 618.
- GitHub Actions run [42659894924](https://github.com/devaxto/ios-davinci/actions/runs/42659894924)
  successfully built the XNU kernel through patch 0104. This verifies the
  build, not a stable davinci boot.
- The latest engineering notes record a davinci display/CA test with an
  unresolved DPU scanout issue; the 0104 image was awaiting a device test.
  They do not establish a usable SpringBoard. Results recorded only for
  `sweet` must not be treated as davinci test results.
- Repository validation and the 0104 CI kernel build have passed. The
  verified kernel artifact is in `xnu/out-0104/kernel.development.sweet`;
  that does not establish that the latest image boots to a usable interface.
- Wi-Fi, cellular, audio, camera, sensors, SEP/Face ID, and a working GPU
  remain unavailable or experimental. The inherited port disables or stubs
  parts of iOS security; do not use personal accounts or data.

The Mi 9T's 1080x2340 display and GT9886 touchscreen are distinct from the
Redmi Note 10 Pro's 1080x2400 display and GT9896 touchscreen. Davinci uses an
L6C touch-rail vote; do not substitute the SWEET L11C settings without
hardware verification. The `SWEET` XNU machine-config name remains because
the devices share the SM7150-family kernel configuration and driver code.

## Build and test

The XNU kernel is built by `.github/workflows/xnu.yml` on a macOS runner. Its
`SWEET` machine-config value is intentional; the resulting kernel contains
the shared board code. The davinci-specific Device Tree and patches select
the Mi 9T hardware at runtime.

Generate the Apple Device Tree and payload after obtaining the required build
outputs, the Mi 9T's stock DTB, and the iPhone X `/product` data from firmware
you are authorized to use:

```sh
python3 tools/mkadt.py shim/out/davinci.adt --highmem
python3 tools/mkpayload.py xnu/out-0104/kernel.development.sweet \
    shim/out/davinci.adt shim/out/davinci-payload.bin \
    --cmdline 'debug=0xc serial=3 cpus=6 keepsyms=1 -enable_kprintf_spam -noprogress'
python3 tools/make-boot.py --payload shim/out/davinci-payload.bin \
    --dtb backup/davinci.dtb shim/out/davinci-test.img
```

The generated tree retains `xiaomi,sweet` for existing shared-driver matching
and also identifies the physical board as `xiaomi,davinci`. It keeps the
iPhone X identity required by the iOS userland. Without the local iPhone X
`/product` input, `mkadt.py` warns and omits that node.

Use `fastboot boot` for initial testing; it does not write the boot image to a
partition. Do not flash `boot`, `system`, or recovery as part of this bring-up.
The experimental root image occupies `userdata` and erases its contents.
Only proceed with that step after verifying the image, confirming the target
partition, and obtaining explicit confirmation immediately before the write.

Use only Apple firmware that you are authorized to use. Firmware images,
device dumps, calibration data, and generated root images are not part of this
repository and must remain out of git.

## Validation

Run the repository checks with:

```sh
python3 tools/validate-repo.py
```

For the detailed engineering log and the status of individual patches, see
[NOTES.md](NOTES.md). GPU-specific design and limitations are in
[docs/GPU.md](docs/GPU.md).
