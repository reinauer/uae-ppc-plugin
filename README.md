# UAE PPC plugin

This repository contains the patch deck and helper script used to build the
QEMU-UAE PowerPC plugin for UAE-based Amiga emulators.

The original repository is <https://github.com/reinauer/qemu-uae>.

The helper downloads the upstream QEMU 11.0.1 source archive, verifies its
SHA-256 checksum, applies the ordered patches from `patches/`, and builds
`qemu-uae.so`.

## Requirements

Install the normal QEMU build dependencies for the host platform, plus:

- `bash`
- `curl` or `wget`
- `tar`
- `patch`
- `sha256sum` or `shasum`
- `ninja`
- GLib and libslirp development files

On macOS, the script also honors `WINUAE_MACOS_DEPLOYMENT_TARGET` and defaults
to deployment target `13.0` when no target is set.

If a sibling `../winuae-macos-deps` tree exists, the script automatically uses
its `pkg-config` files. You can also point at a dependency prefix explicitly:

```sh
QEMU_UAE_DEPS_PREFIX=/path/to/deps ./build-qemu-uae-plugin.sh
```

## Build

Run:

```sh
./build-qemu-uae-plugin.sh
```

By default this creates:

- `build/downloads/qemu-11.0.1.tar.xz`
- `build/qemu-11.0.1-uae/`
- `build/qemu-uae.so`

Useful options:

```sh
./build-qemu-uae-plugin.sh --clean
./build-qemu-uae-plugin.sh -j 8
./build-qemu-uae-plugin.sh --output /path/to/qemu-uae.so
./build-qemu-uae-plugin.sh -- --disable-debug-info
```
