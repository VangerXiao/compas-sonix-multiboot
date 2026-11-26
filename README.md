# Multi-boot for the HiBy R1 and R3II 2025

**English** · [简体中文](README.zh-CN.md)

A multi-boot modification built around several excellent open-source players — **CompásOS**, **SonixOS** and **Rockbox**（Hiby R1 only）. Each is the work of its own authors and each makes its own trade-offs. This project **does not replace any of them and has no affiliation with any upstream project**. Because it modifies parts of them, it **may introduce unknown bugs and other problems** — and those are very likely this project's fault. Unless a problem also reproduces after flashing the original system back, **please report it in this repository**.

| Device | Systems you can boot | Open the menu with |
| --- | --- | --- |
| **HiBy R1** | HiByOS, CompásOS, SonixOS, Rockbox | VOL− + power |
| **HiBy R3II 2025** | HiByOS, CompásOS, SonixOS | PLAY/PAUSE + power |

## ⚠️ Risk notice (R3II 2025 only)

HiBy documents "hold play/pause while powering on to enter recovery". Reading the stock firmware and testing on the device shows that **it does not work**, and no key combination that reaches HiBy's own updater has been found.

Every firmware in Releases has been tested on real hardware, but a boot can still fail in bad cases — and **if it fails in a way that no system can start, the device is bricked**. Mine did brick during development, and the way back was Ingenic's official Cloner over USB. That is far more involved than the R1 route of reflashing from the built-in updater.

So if you own an R3II 2025, weigh the risk, and be ready to use the cloner to flash the stock system back. **HiBy R1 owners can ignore this section**: the R1 keeps a usable built-in recovery path.

## What this repository is

An **unofficial, experimental** project that lets multiple systems share one device: flash once, then switch as often as you like. Your one device can now hold **four systems at once** — the stock HiByOS, CompásOS, SonixOS and (on the R1) Rockbox — and switch between them freely.

| Piece | Where the code lives |
| --- | --- |
| HiByOS (stock) | left in place, unmodified |
| CompásOS with the multi-boot menu (vanilla) | [`VangerXiao/compas-player`](https://github.com/VangerXiao/compas-player) branch **`mutiboot`** |
| CompásOS with the CN enhancements | [`VangerXiao/compas-player`](https://github.com/VangerXiao/compas-player) branch **`cn-enhanced`** |
| SonixOS | [`VangerXiao/sonix-player`](https://github.com/VangerXiao/sonix-player) branch **`compas-multiboot`** |
| Rockbox | [`VangerXiao/rockbox`](https://github.com/VangerXiao/rockbox) branch **`rockbox_mutiboot_fix`** |
| Prebuilt images | [Releases](../../releases) of this repository |

`mutiboot` is the clean "upstream + multi-boot" build: no visual changes, no network changes.
`cn-enhanced` adds a few things that work better on networks in mainland China; see "About CN_Enhanced" below.

## Installing

1. Prepare a TF card with `compas/rootfs.squashfs` and `sonix/rootfs.squashfs` in its root (plus `.rockbox/` on the R1).
2. Flash the `*.upt` for your device from Releases: it carries the kernel, the stock root filesystem and the boot hook.
3. Hold the menu key while powering on to pick a system; afterwards a plain power press starts whatever you picked last.

> To recover, or to flash a particular system back, **put that system's package in the card root and rename it**:
> - HiBy R1: rename it to `r1.upt`, then hold VOL+ while powering on to enter the built-in updater;
> - HiBy R3II 2025: rename it to `r3ii_2025.upt`, then use **Settings → Firmware update → via SD card**.
>
> The R3II has no key-based entry point, so always keep at least one bootable system, or have the cloner ready.

## Switching systems

| Device | Open the menu | Inside the menu |
| --- | --- | --- |
| **HiBy R1** | hold VOL−, then press power | VOL +/− moves, any other key confirms |
| **HiBy R3II 2025** | hold PLAY/PAUSE, then press power | the wheel moves, PLAY/PAUSE confirms |

The menu writes the choice to `boot_target.txt` on the card, so the next plain power press starts that system. With no card, or with the image for that system missing, the device starts HiByOS.

## Fixes for Rockbox (R1 only)

- **800×480 MPEG1 video** playback; see [`diy/docs/video-transcode.md`](https://github.com/VangerXiao/rockbox/blob/diy/r1-erosq/diy/docs/video-transcode.md) for the transcode command.
- PictureFlow key mapping is fixed: VOL +/− pages, PLAY/PAUSE opens an album.
- Under multi-boot, Rockbox shuts down by calling `reboot(2)` directly, so **a short power press really powers it off**.

## How switching works

Boot is `kernel → /linuxrc → the init scripts in rcS`. The hook, `S20switch`, sits early in rcS: it mounts the TF card, reads `boot_target.txt` from its root, and decides whether to stay in HiByOS or to chroot into one of the card's `rootfs.squashfs` images. Those images are read-only and mounted at `/tmp/target` before the chroot, so the systems cannot disturb each other.

A normal restart returns each system to itself; to get back to the stock system, use "Reboot to HiByOS" in the settings.

Every failure path in the hook **falls back to the stock boot** (no card, missing image, unparsable marker, failed mount all `exit 0`), so a broken hook can never leave the device without a system.

## Where the data lives

| System | Its filesystem | Its user data |
| --- | --- | --- |
| Stock HiByOS | internal storage | internal storage |
| CompásOS | TF card | internal storage (shared with HiByOS) |
| SonixOS | TF card | **TF card** (`sonix/data`, isolated from the others) |
| Rockbox (R1) | TF card `.rockbox/` | TF card |

CompásOS is a fork of HiByOS and reads and writes the same data layout, so sharing with the stock system is exactly what installing CompásOS over it would do. SonixOS comes from a different lineage, and its mount helper **re-creates the userdata volume** when it does not recognise the layout — which is why its data sits on the card instead.

## About CN_Enhanced

CN_Enhanced exists because GitHub is unreliable from parts of mainland China: it routes GitHub requests through a gh-proxy mirror (**on by default**; write `off` to `/usr/data/.compas/github_mirror.txt` to disable it), enlarges the quick-drawer captions a little for readability (20 px as shipped) and widens the coverage of a few fonts.

It also adds **pinyin-initial search**: type `zjl` in the library search box to find 周杰伦, `wyt` to find 五月天, and the library and A–Z bar sort by pinyin initial. These exist only in CN_Enhanced.

Outside mainland China, **use the `mutiboot` build**, or **ignore anything labelled `CN_Enhanced`** — otherwise you may see unexplained network problems, garbled Japanese titles or other issues.

## About in-system updates

This image changes how the systems are stored, so **do not use either system's built-in updater**:

- CompásOS's firmware update flashes the kernel and root filesystem partitions, which overwrites the multi-boot hook and puts the stock kernel back — multi-boot is gone;
- SonixOS's online update replaces the player and its files on the card.

When upstream ships a new version, this project has to merge it and rebuild the images; new images appear in Releases. Making the in-system updaters aware of multi-boot may be solved later.

## Problems

Please open an [Issue](../../issues) in this repository.

**Once more**: do not report these problems to the CompásOS or SonixOS authors. Anything this project breaks is this project's responsibility, and none of the changes here are meant to affect upstream — they are downstream branches only.

## Following upstream

This fork tracks both upstreams. Our changes are a small set of commits on top of `upstream/main`, so an upstream release means one rebase:

```sh
git remote add upstream https://github.com/Starnished66/compas-player.git
git fetch upstream
git checkout main && git merge --ff-only upstream/main && git push origin main
git checkout mutiboot && git rebase upstream/main && git push --force-with-lease
git checkout cn-enhanced && git rebase mutiboot && git push --force-with-lease
```

(The other repository works the same way; swap the URL for `Jepl4r/sonix-player` and the branch for `compas-multiboot`.)

## Licence

CompásOS and SonixOS are GPLv3. These forks keep their licences and notices, and every modification is published in the branches listed above, as the licences require. The images in Releases are built from those branches.

## Thanks

- To [Starnished66](https://github.com/Starnished66) and the CompásOS community.
- To [Jepl4r](https://github.com/Jepl4r) and the SonixOS community. The original repository is no longer reachable; the community continues at [Tartarus6/sonix-player](https://github.com/Tartarus6/sonix-player), and the HiBy R3II 2025 port came from [unrau/sonix-player](https://github.com/unrau/sonix-player). Thanks to all of them.
- Parts of this project were written with AI assistance ([DeepSeek Harness](https://github.com/deepseek-ai)).

[![](https://img.shields.io/badge/powered_by-dsh-4D6BFE?style=flat-square&logo=deepseek&logoColor=white)](https://github.com/deepseek-ai/deepseek-harness)
