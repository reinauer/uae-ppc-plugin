# UAE PPC plugin

This repository contains the patch deck and helper script used to build the
QEMU-UAE PowerPC plugin for UAE-based Amiga emulators.

The source branch for the QEMU-UAE patch deck is
[`qemu-v11.1.3-uae` in `reinauer/qemu`](https://github.com/reinauer/qemu/tree/qemu-v11.1.3-uae).

The helper downloads the upstream QEMU source archive, verifies its SHA-256
checksum, applies the ordered version-specific patches, and builds
`qemu-uae.so` on Unix/macOS or `qemu-uae.dll` on Windows. By default it builds
QEMU 11.1.3.

QEMU 11 no longer supports system emulator builds on 32-bit hosts, so this
repository builds the Windows 64-bit and Windows ARM64 plugins from QEMU
11.1.3, while the Windows 32-bit workflow builds from QEMU 9.2.4 with the
self-contained deck in `patches-qemu-9.2.4/`. The helper selects it
automatically; no extra patch directory is needed.

The Windows CI artifacts use WinUAE's per-architecture loader names so all
three DLLs can be unpacked into the same plugin directory:

- `qemu-uae.dll` for 32-bit WinUAE
- `qemu-uae_x64.dll` for 64-bit WinUAE
- `qemu-uae_arm64.dll` for ARM64 WinUAE

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

- `build/downloads/qemu-11.1.3.tar.xz`
- `build/qemu-11.1.3-uae/`
- `build/qemu-uae.so` or `build/qemu-uae.dll`

Windows builds pass QEMU's `--static` option so GLib, zlib, winpthread, and
other MSYS2-supplied runtime libraries are linked into `qemu-uae.dll` instead
of being required as sidecar DLLs.

Set `QEMU_UAE_PDB=1` on Windows clang/lld builds to create `qemu-uae.pdb`
next to `qemu-uae.dll`. The hosted MINGW32 CI lane uses GCC because
`setup-msys2` does not currently provide a CLANG32 environment.

Useful options:

```sh
./build-qemu-uae-plugin.sh --clean
./build-qemu-uae-plugin.sh -j 8
./build-qemu-uae-plugin.sh --output /path/to/qemu-uae.so
./build-qemu-uae-plugin.sh -- --disable-debug-info
```

## Release candidate and older sources

QEMU 11.1.3 is the default release. The separate
[`qemu-v11.2.0-rc1-uae` branch](https://github.com/reinauer/qemu/tree/qemu-v11.2.0-rc1-uae)
and `patches-qemu-11.2.0-rc1/` deck track the release candidate for testing:

```sh
QEMU_UAE_QEMU_VERSION=11.2.0-rc1 ./build-qemu-uae-plugin.sh
```

`QEMU_UAE_QEMU_VERSION=11.0.1` still selects the older source and deck.
All supported versions include the memory-region ownership fix from
[Amiberry PR #4](https://github.com/BlitterStudio/amiberry-qemu-uae/pull/4).
The existing atomic JIT-flush request and vCPU pause fixes are retained.

Patches apply with zero fuzz. A complete deck is checked in reverse order
on temporary copies to recognize an already-patched source tree. If a
partial or locally modified tree cannot be replayed, use a separate work
directory or `--clean` (which removes the selected source directory).
