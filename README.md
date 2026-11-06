# Compás Player with SonixOS Multi-boot

**English** · [简体中文](README.zh-CN.md)

A multi-boot modification for two excellent open-source players — **CompásOS**
and **SonixOS**. Both are the work of their own authors and both are very good at
what they do. This project **does not replace either of them and has no
affiliation with either upstream project**. Because it modifies parts of both
systems, it **may introduce unknown bugs and other problems** — and those are
very likely this project's fault. Unless a problem also reproduces after flashing
the original system back, **please report it in this repository**.

## What this repository is

An **unofficial, experimental** project that lets the two systems share one
device: flash once, then switch as often as you like. Your one device can now
hold **three systems at once** — CompásOS, SonixOS, and Rockbox, which CompásOS's
own boot selector can start — and switch between them freely!

| Piece | Where the code lives |
|---|---|
| CompásOS with the dual-boot menu (vanilla) | [`VangerXiao/compas-player`](https://github.com/VangerXiao/compas-player) branch **`mutiboot`** |
| CompásOS with the CN enhancements | [`VangerXiao/compas-player`](https://github.com/VangerXiao/compas-player) branch **`cn-enhanced`** |
| SonixOS with the "reboot to Compás" pill | [`VangerXiao/sonix-player`](https://github.com/VangerXiao/sonix-player) branch **`compas-multiboot`** |
| Prebuilt images | [Releases](../../releases) of this repository |

`mutiboot` is the plain "upstream plus dual boot" build: nothing cosmetic, no
network changes. `cn-enhanced` is `mutiboot` plus the conveniences listed under
"About the CN_Enhanced builds" below.

## How the switch works

CompásOS stays on the internal flash and is always the default. SonixOS lives
entirely on the SD card.

```
power on
  |
  +-- CompásOS boots normally
  |     Settings -> System -> "Reboot to SonixOS"
  |        writes /sonix/boot, reboots
  |
  +-- CompásOS's boot hook sees /sonix/boot
        mounts /sonix/rootfs.squashfs, bind-mounts /sonix/data over /usr/data,
        chroots into SonixOS
              |
              +-- power menu -> "Slide to reboot to Compás"
                    writes /usr/data/boot_compas, reboots
                    hook clears both files and CompásOS comes up
```

SonixOS's settings live in `/sonix/data` on the card, so the two systems' user
data stay separate and cannot collide.

A plain restart inside SonixOS **keeps SonixOS**; coming back to CompásOS takes
the explicit "Slide to reboot to Compás". Likewise, an ordinary restart in
CompásOS never switches systems either.

## Installing

1. Put the release's `r1.upt` in the **card root** (the name must be exactly
   `r1.upt`, lower case).
2. Unzip the release's Sonix archive into the card root, so that a `sonix` folder
   exists there.
3. Hold **volume-up**, press **power**, keep holding until the upgrade screen
   appears. Wait for it to finish; the device reboots into CompásOS. (If the
   device is already running any system — the stock HiByOS, CompásOS or SonixOS
   — you can also install it through that system's "install update from the SD
   card" entry.)
4. Settings -> System -> **Reboot to SonixOS** to switch over.
5. In SonixOS, open the power menu and slide **"Reboot to Compás"** to come back.

> If something goes wrong — or you want to flash a particular system back — **put
> that system's image in the card root and rename it to `r1.upt`.** The factory
> recovery path (hold volume-up while powering on) still works.

## Notes

### About the CN_Enhanced builds

The CN_Enhanced builds address unstable GitHub access from parts of mainland
China: GitHub requests are routed through a gh-proxy mirror (**on by default**;
write `off` into `/usr/data/.compas/github_mirror.txt` to disable it). They also
rework some of the network logic (chiefly the NetEase plugin's login cookie being
truncated), size the quick drawer's captions a little larger for readability
(20 px on the default font-size tier), and widen the coverage of the bundled
fonts.This also add pinyin initial search: typing `zjl` in the library search box
finds 周杰伦, `wyt` finds 五月天. This exists only in the CN_Enhanced builds.

If you are not in mainland China, please **use a `mutiboot` release without the
`CN_Enhanced` label**, or **ignore the `CN_Enhanced` builds entirely** —
otherwise they may bring unexpected network ,Japanese song titles messed up，or other problems.

### About online updates

This image changes how both systems store their data, so **do not use either
system's built-in online update**:

- CompásOS's firmware update flashes the kernel and root filesystem partitions
  directly, which overwrites the dual-boot hook and puts the stock kernel back —
  dual boot stops working.
- SonixOS's online update replaces the player and files on the card.

When upstream releases a new version, this project has to merge it and rebuild
the images. New images are published in Releases. Making online updates work
with dual boot may be solved in the future.

## Reporting problems

Open an [issue](../../issues) here.

**One more time:** please do not report these problems to the CompásOS or SonixOS
authors — anything that goes wrong with the modifications in this project is this
project's responsibility, not theirs. Likewise, nothing here is intended to
affect upstream; these are downstream branches only.

## Keeping up with upstream

This fork follows both upstreams. The branches are kept as a small set of commits
on top of upstream `main`, so picking up new upstream work is a rebase:

```sh
git remote add upstream https://github.com/Starnished66/compas-player.git
git fetch upstream
git checkout main && git merge --ff-only upstream/main && git push origin main
git checkout mutiboot && git rebase upstream/main && git push --force-with-lease
git checkout cn-enhanced && git rebase mutiboot && git push --force-with-lease
```

(the same, with `Jepl4r/sonix-player` and `compas-multiboot`, for the other
repository).

## Licence

CompásOS and SonixOS are both GPLv3. These forks keep their licences and their
notices; the modifications are in the branches linked above, as the licence
requires. Release images are built from exactly those branches.

## Acknowledgements

- Thanks to [Starnished66](https://github.com/Starnished66) and the CompásOS
  community.
- Thanks to [Jepl4r](https://github.com/Jepl4r) and the SonixOS community.
- Parts of this project were built with assistance from
  [DeepSeek Harness](https://github.com/deepseek-ai), an AI coding assistant.
