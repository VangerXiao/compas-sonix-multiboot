# Compás Player with SonixOS Multi-boot

Two excellent open-source players, one HiBy R1, and a way to switch between them
without opening a case, pulling the card, or reflashing.

- **[CompásOS](https://github.com/Starnished66/compas-player)** — the polished
  everyday player: a clean interface, broad codec support, plugins, a real
  settings tree, and a community that keeps shipping improvements.
- **[SonixOS](https://github.com/Jepl4r/sonix-player)** — a focused, minimal
  player with a beautiful interface and a refreshingly small footprint.

Both are the work of their own authors and both are very good at what they do.
This project does not replace either of them, is not affiliated with either of
them, and adds nothing to their source trees.

## What this repository is

An **unofficial hobby project** that packages the two systems so they can share
one device:

| Piece | Where the code lives |
|---|---|
| CompásOS with the dual-boot menu (vanilla) | [`VangerXiao/compas-player`](https://github.com/VangerXiao/compas-player) branch **`mutiboot`** |
| CompásOS with the CN enhancements (GitHub mirror, NetEase fixes, `v1.4.1CN` label) | [`VangerXiao/compas-player`](https://github.com/VangerXiao/compas-player) branch **`cn-enhanced`** |
| SonixOS with the "reboot to Compás" pill | [`VangerXiao/sonix-player`](https://github.com/VangerXiao/sonix-player) branch **`compas-multiboot`** |
| Prebuilt images | [Releases](../../releases) of this repository |

`mutiboot` is the plain "upstream plus dual boot" build: nothing cosmetic, no
network changes. `cn-enhanced` is `mutiboot` plus the conveniences that make the
player comfortable to use from mainland China — an optional GitHub mirror for the
update check and the two stores, and a fix for NetEase login cookies being
truncated.

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

A plain restart inside SonixOS keeps SonixOS — leaving is an explicit choice.

SonixOS's settings live in `/sonix/data` on the card, so a SonixOS session can
never disturb the CompásOS data in the internal volume.

## The recovery path is untouched

The factory recovery partitions (the ones that implement "hold volume-up, press
power, flash `r1.upt` from the card") are never modified. Keep a known-good
`r1.upt` in the card root at all times; it is the escape hatch that has never
failed.

**Always keep `r1.upt` on the card.**

## Installing

1. Copy `sonix/rootfs.squashfs` from a release to `/sonix/rootfs.squashfs` on the
   card, and create `/sonix/data/` next to it.
2. Copy the release's `r1.upt` to the card root.
3. Hold **volume-up**, press **power**, keep holding until the upgrade screen
   appears. Wait for it to finish; the device reboots into CompásOS.
4. Settings -> System -> **Reboot to SonixOS** to try the other side.

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
