# Multi-boot for the HiBy R1

**English** · [简体中文](README.zh-CN.md)

> ⚠️ **Alpha release.** Four systems do coexist now, but this can still cause unexpected problems: corrupted data, a device that will not boot, the need to reflash. Back up the card first and make sure you know how to recover (see the end of "Installing").

A multi-boot modification built around several excellent open-source players — **CompásOS**, **SonixOS** and **Rockbox**. Each is the work of its own authors and each makes its own trade-offs. This project **does not replace any of them and has no affiliation with any upstream project**. Because it modifies parts of them, it **may introduce unknown bugs and other problems** — and those are very likely this project's fault. Unless a problem also reproduces after flashing the original system back, **please report it in this repository**.

## What this repository is

An **unofficial, experimental** project that lets multiple systems share one device: flash once, then switch as often as you like. Your one device can now hold **four systems at once** — the stock HiByOS, CompásOS, SonixOS and Rockbox — and switch between them freely.

| Piece | Where the code lives |
| --- | --- |
| HiByOS (stock) | left in mtd2, unmodified |
| CompásOS with the multi-boot menu (vanilla) | [`VangerXiao/compas-player`](https://github.com/VangerXiao/compas-player) branch **`mutiboot`** |
| CompásOS with the CN enhancements | [`VangerXiao/compas-player`](https://github.com/VangerXiao/compas-player) branch **`cn-enhanced`** |
| SonixOS | [`VangerXiao/sonix-player`](https://github.com/VangerXiao/sonix-player) branch **`compas-multiboot`** |
| Rockbox | [`VangerXiao/rockbox`](https://github.com/VangerXiao/rockbox) branch **`rockbox_mutiboot_fix`** |
| Prebuilt images | [Releases](../../releases) of this repository |

`mutiboot` is the plain "upstream plus multi-boot" build: nothing cosmetic, no network changes. `cn-enhanced` is `mutiboot` plus the conveniences listed under "About the CN_Enhanced builds" below.

## Switching systems

Hold **VOL−** while pressing power and the device opens a boot menu. If you just press power next time, the device boots whatever you picked last. With no card, or with the card missing that system's files, it falls back to HiByOS.

Both CompásOS and SonixOS add a **"Reboot to HiByOS"** row to their settings, so getting back to the stock system is one tap.

## What was fixed for Rockbox

- Rockbox now plays **800×480 MPEG1 video**; see [`diy/docs/video-transcode.md`](https://github.com/VangerXiao/rockbox/blob/diy/r1-erosq/diy/docs/video-transcode.md) for the transcoding recipe.
- PictureFlow had its keys fixed: VOL +/− page through covers, play/pause opens the album.
- Under the multi-boot layout Rockbox now powers off by calling `reboot(2)` directly, so a short press on the power key really switches the device off (it used to need a ten-second hold).

## How the switch works

Booting goes `kernel → /linuxrc → the init scripts in rcS`. This project's hook, `S20switch`, runs early in rcS: it mounts the card, reads `boot_target.txt` from its root, and then either stays in HiByOS or chroots into one of the card's squashfs images.

- Every system on the card is a read-only `rootfs.squashfs`, mounted at `/tmp/target` and entered with `chroot`, so they cannot interfere with each other.
- CompásOS keeps its user data on the internal storage (UBI); SonixOS keeps its own under `/sonix/data` on the card, so the two never collide.
- "Reboot to HiByOS" inside a system only writes `stock` into `boot_target.txt` and reboots. A plain reboot writes that system's own name back, so an ordinary reboot never switches systems by accident.

## Installing

1. Prepare a card with `compas/rootfs.squashfs`, `sonix/rootfs.squashfs` and `.rockbox/` in its root.
2. Flash `r1.upt` from the Release — it writes the kernel, the stock root filesystem and the boot hook.
3. On the first boot, hold VOL− to open the menu and pick a system; the menu records the choice in `boot_target.txt`.
4. From then on, pressing power boots whatever you picked last.

> If something goes wrong (or you want a particular system back), **put that system's package in the root of the card and rename it to `r1.upt`.** The factory recovery path (hold VOL+ while powering on) still works.

## Notes

### About the CN_Enhanced builds

The CN_Enhanced build adds a **gh-proxy mirror for GitHub requests**, because GitHub is unreliable from parts of mainland China (**on by default**; write `off` into `/usr/data/.compas/github_mirror.txt` to disable it). It also raises the Quick Drawer caption size a little for legibility (20 px by default) and widens the bundled font coverage. On top of that it adds **pinyin-initial search**: type `zjl` in the library search box to find 周杰伦, or `wyt` for 五月天, and the library and its A-Z strip sort by pinyin initial too. Those features exist only in the CN_Enhanced build.

If you are not in mainland China, prefer the `mutiboot` release.

### About online updates

This image changes how storage is laid out, so **do not use either system's built-in online update**:

- CompásOS's firmware update writes the kernel and root filesystem partitions directly, overwriting the boot hook and putting the stock kernel back, which ends the multi-boot;
- SonixOS's online update replaces the player and files on the card.

When upstream ships a new version, this project re-merges upstream and rebuilds; new images appear under Releases.

## Problems

Please open an [Issue](../../issues) here.

**Once more**: do not report these to the CompásOS, SonixOS or Rockbox authors — anything this project broke is this project's responsibility. Conversely, nothing here is meant to affect upstream; these are downstream branches only.

## Following upstream

This fork keeps following upstream. Our changes are a handful of commits on top of `upstream/main`, so each upstream update is one rebase:

```sh
git remote add upstream https://github.com/Starnished66/compas-player.git
git fetch upstream
git checkout main && git merge --ff-only upstream/main && git push origin main
git checkout mutiboot && git rebase upstream/main && git push --force-with-lease
git checkout cn-enhanced && git rebase mutiboot && git push --force-with-lease
```

(The other repository works the same way, with `Jepl4r/sonix-player` and the `compas-multiboot` branch.)

## Licence

CompásOS and SonixOS are GPLv3; Rockbox is GPLv2. These forks keep their respective licences and notices, and every change is public in the branches listed above, as those licences require. The images in Releases are built from exactly those branches.

## Thanks

- [Starnished66](https://github.com/Starnished66) and the CompásOS community.
- [Jepl4r](https://github.com/Jepl4r) and the SonixOS community.
- The Rockbox project and everyone who worked on its ports.
- Part of this project was written with [DeepSeek Harness](https://github.com/deepseek-ai) (an AI coding assistant).
