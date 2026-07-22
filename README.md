# Raven OS

![License](https://img.shields.io/badge/license-GPLv2-blue.svg)
![Status](https://img.shields.io/badge/status-active%20development-yellow.svg)
![Kernel](https://img.shields.io/badge/kernel-6.6%20LTS-brightgreen.svg)
![Toolchain](https://img.shields.io/badge/toolchain-musl%20%2B%20GCC%2016-orange.svg)

**linux** · **operating-system** · **kernel** · **musl** · **busybox** · **squashfs** · **overlayfs** · **privacy** · **from-scratch** · **osdev** · **live-boot**

A minimal, privacy-focused Linux distribution built from scratch custom musl cross-toolchain, self-configured kernel, static BusyBox userspace, and a live-boot architecture with a read-only squashfs root and a RAM-only overlay (nothing persists across reboots by design).

This is a from scratch build in LFS (Linux From Scratch) tradition: no distro packages, no prebuilt kernel, no prebuilt userspace. Every binary in the boot chain toolchain, kernel, init, shell was compiled from source for this project.

## Why

Most "build your own Linux" projects stop at booting a kernel. Raven's goal is a distributable ISO people can put on a USB stick and boot on real hardware, with an actual design point: a minimal, hardened, and schizo private amnesiac system in the spirit of Tails/OpenBSD, not just a proof that Linux can boot.

## Architecture
Custom musl cross-toolchain (x86_64-linux-musl-*)
GCC 16.1.0 + binutils 2.44, built via musl-cross-make
|
v
Linux kernel 6.6 (bzImage)
Custom config: squashfs, overlayfs, minimal drivers
|
v
BusyBox 1.36.1 (static)
ash shell, coreutils, init utilities in one binary
|
v
raven.squashfs
Compressed, read-only root filesystem the OS payload
|
v
Tiny bootstrap initramfs
Mounts squashfs read-only, layers tmpfs overlay, switch_root

**Boot flow:** GRUB loads the kernel and a small bootstrap initramfs. The initramfs mounts the squashfs image read-only, creates a tmpfs-backed overlay on top (overlayfs), and switch_roots into the combined filesystem. Any writes during a session land in RAM only — a reboot returns the system to its exact shipped state.

## Toolchain notes

Building a musl cross-toolchain with a bleeding-edge GCC (16.1.0) surfaced a handful of real bugs, not just config mistakes:

- **GCC 16 defaults to `-std=c23`**, which reserves `bool`/`true`/`false` as keywords — the Linux 6.6 headers (written pre-C23) collide with this. Fixed by forcing `-std=gnu11` at the compiler level.
- **The custom-built binutils 2.44 shipped a broken `nm`** — segfaulted on the kernel's realmode object files. Worked around by wrapping the cross-nm to call the host system's nm instead (symbol table reading doesn't need cross-compiled tooling).
- **Same binutils build's `ar` also crashed** (SIGSEGV while archiving arch/x86/lib). Same fix pattern wrapped to call host ar.
- **GCC's `--enable-default-pie` broke the kernel's 16-bit real-mode trampoline code**, which cannot be position-independent. Fixed by forcing `-fno-pie`/`-no-pie` through a compiler/linker wrapper.

These are documented here because they're the kind of bug that's easy to lose an afternoon to and hard to find written up anywhere hopefully useful to the next person who tries a GCC 16 + musl-cross-make + Linux 6.x combination.

## Status

- [x] Custom musl cross-toolchain
- [x] Kernel builds to bzImage
- [x] Static BusyBox userspace
- [x] Boots to an interactive shell (QEMU)
- [x] squashfs + overlayfs live-boot architecture
- [x] Bootable ISO (GRUB + xorriso)
- [x] Basic networking (DHCP via udhcpc)
- [x] raven-install: interactive installer (partition/format/copy tested working; GRUB-in-Raven still pending)
- [x] caw: custom package manager with real hosted packages (musl cross-compiled)
- [x] First real package: fastfetch, cross-compiled for musl, Raven-branded logo
<<<<<<< HEAD
- [ ] Real hardware boot test
- [x] Persistent /etc across reboots (persistent partition + init script bind-copy) - tested working
=======
- [ ] Persistent /etc across reboots (persistent partition + init script bind-mount)
>>>>>>> 5c05f5648a32d8ffee1d1f2c8f65d4a2291e3f77
- [ ] GRUB cross-compiled for musl (needed for raven-install to fully work)
- [ ] Tor-forcing firewall
- [ ] runit as PID 1 (currently BusyBox's built-in init)
- [ ] Desktop environment
- [ ] Hardening pass (sysctl, module restrictions, setuid audit)
- [ ] Real hardware boot test
## Build environment

- Host: Arch Linux (VM)
- Kernel: Linux 6.6 (LFS)
- Toolchain: musl-cross-make, GCC 16.1.0, binutils 2.44
- Userspace: BusyBox 1.36.1 (static)

## License

GPLv2 — see [LICENSE](LICENSE)
