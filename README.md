# Compás Player with SonixOS Multi-boot

**English** · [简体中文](README.zh-CN.md)

A multi-boot modification for two excellent open-source players — **CompásOS**
and **SonixOS**. Both are the work of their own authors and both are very good at
what they do. This project **does not replace either of them and has no
affiliation with either upstream project**. Because it modifies parts of both
systems, it **may introduce unknown bugs and other problems** — and those are
very likely this project's fault. Unless a problem also reproduces after flashing
the original system back, please report it here, not upstream.

## What this repository is

An **unofficial hobby project** that lets the two systems share one device: flash
once, then switch as often as you like. A single device can hold **three systems
at once** — CompásOS, SonixOS, and Rockbox, which CompásOS's own boot selector can
start — and switch between them freely.

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

A plain restart inside SonixOS **keeps SonixOS** — coming back to CompásOS is an
explicit choice, not an accident.

## Installing

1. Put the release's `r1.upt` in the **card root** (the name must be exactly
   `r1.upt`, lower case).
2. Put the release's Sonix root filesystem at `/sonix/rootfs.squashfs`, and make
   sure `/sonix/data/` exists next to it (SonixOS keeps its settings there; it
   may start empty).
3. Hold **volume-up**, press **power**, keep holding until the upgrade screen
   appears. Wait for it to finish; the device reboots into CompásOS.
4. Settings -> System -> **Reboot to SonixOS** to switch over.
5. In SonixOS, open the power menu and slide **"Reboot to Compás"** to come back.

> **Always keep a working `r1.upt` in the card root.** The factory recovery path
> (hold volume-up while powering on) has never failed, and it is the only way
> back if something goes wrong.

## Notes

### About the CN_Enhanced builds

The CN_Enhanced builds address unstable GitHub access from parts of mainland
China: GitHub requests are routed through a gh-proxy mirror (**on by default**;
write `off` into `/usr/data/.compas/github_mirror.txt` to disable it). They also
fix the NetEase plugin's login cookie being truncated, and size the quick
drawer's captions like the rest of the interface (20 px on the default font-size
tier).

The Chinese text rendering itself comes from **upstream CompásOS**, not from this
project; only the drawer caption size is adjusted here.

If you are not in mainland China, please use a release **without** the
CN_Enhanced label — otherwise GitHub traffic may go through the proxy and be
slower than it needs to be.

### About online updates

This image changes how both systems store their data, so **do not use either
system's built-in online update**:

- CompásOS's firmware update flashes the kernel and root filesystem partitions
  directly, which overwrites the dual-boot hook and puts the stock kernel back —
  dual boot stops working.
- SonixOS's online update replaces the player and files on the card.

When upstream releases a new version, this project has to merge it and rebuild
the images. New images are published in Releases.

## Reporting problems

Open an [issue](../../issues) here.

Please **do not** report problems with these builds to the CompásOS or SonixOS
authors — anything that goes wrong with the modifications in this project is this
project's responsibility, not theirs. Likewise, nothing here is proposed for
upstream: these branches are a downstream fork and are not intended to be merged
back.

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
