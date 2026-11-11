# Multi-boot state — 2026-11-11

Where the work stopped, what works, and what to pick up next. Written after a
session that got HiByOS, CompasOS and SonixOS booting off one device with a
shared kernel, plus a Rockbox entry.

## Layout on the device

| Region | Holds | Notes |
|---|---|---|
| mtd0 | uboot/SPL | untouched |
| mtd1 | SonixOS kernel (`xImage-R1`, `4.4.94-sonix`) | one kernel serves every system |
| mtd2 | **HiByOS v2.0 rootfs** | modules patched to `4.4.94-sonix` |
| mtd3 | factory recovery kernel | untouched — this is the way back |
| mtd4 | factory recovery rootfs | untouched |
| mtd5 | ota marker + theme byte at `0x20000` | untouched |
| mtd6 | userdata (UBI, `/usr/data`) | untouched |
| SD | `compas/rootfs.squashfs`, `sonix/rootfs.squashfs`, `.rockbox/rockbox.r1`, `boot_target.txt` | |

The HiByOS rootfs carries the selector: `/etc/init.d/S20switch` runs before
`S21mount_ubifs`, mounts the card, reads `boot_target.txt` (`stock` / `compas` /
`sonix` / `rockbox`) and chroots into the named system's squashfs. Holding a
key at boot runs `/usr/bin/bootmenu`, which writes the same marker.

Everything falls back to HiByOS: no card, unknown marker, or a missing image all
leave the stock system running.

## What works

- Three systems plus Rockbox selectable from the boot menu (hold VOL- or NEXT).
- Menu: pre-rendered frames blitted to `/dev/fb0`, countdown drawn from small
  strips, first key press freezes it and blanks the line.
- CompasOS "Reboot to HiByOS" and "Reboot to SonixOS" rows write the marker.
- SonixOS keeps its data in `sonix/data` on the card; HiByOS owns the internal
  UBI volume, so the two do not fight over it.
- No-card boot is fast: the hook gives up after ~1 s when `/sys/block/mmcblk0`
  never appears.
- Gold theme-2 splash replaced by the default artwork (`logo2.jpeg` only).
- `S99shutdown_logo` repaints the logo while HiByOS shuts down.

## Known problem: Rockbox power-off

After Rockbox, the device does not power off — it needs a ten-second power-key
hold. This happens both when the hook starts Rockbox and when CompasOS's own
selector does, and it started when CompasOS moved from mtd2 to the card.

Working theory: the running root filesystem is a loop-mounted squashfs whose
backing file lives on the card. The shutdown path (`release_sd_card()` in
CompasOS, and our own `sync`/`umount` in the hook) has to flush and release that
same card, which cannot complete while it is the root; the power-off syscall is
then never reached.

Tried and did not help: skipping `sync`/`umount` in the hook and calling
`poweroff -f` straight after Rockbox exits.

Untried options, cheapest first:

1. Put CompasOS back in mtd2 and move HiByOS to the card. The root filesystem
   then lives in NAND and the old, working behaviour comes back. Costs the
   "pull the card and still get the stock system" property.
2. Patch `compas_loader` to skip `release_sd_card()` when its root is a loop
   mount backed by the card. Needs the loader rebuilt, which needs the Rockbox
   source (`scripts/build/bootloader/build_rockbox_bootloader.sh` clones it).
3. Change Rockbox's own power-off code. Needs the Rockbox source for this port.

## Where things live

- `D:\r1fw\out\expC_stock_mtd2_switch.upt` — the current image
  (`r1.upt` on the card, sha256 `a0a68c4f…`).
- `D:\r1fw\work\packaging\S20switch_v2` — the boot selector installed as
  `S20switch`. **Keep it LF-only**: a CRLF edit once made it unparseable and
  killed multi-boot silently; the build now runs `sh -n` and a CRLF check.
- `D:\r1fw\work\packaging\bootmenu3.c` — the menu; frames and countdown strips
  come from `D:\r1fw\gen_bootmenu3.py`.
- `D:\r1fw\build_expC3.sh` — builds the whole image from the stock v2.0 rootfs.
- `D:\r1fw\pack_any.sh` — packs any rootfs tree plus kernel into a `.upt`.
- `compas-player` branches: `mutiboot` (base) and `cn-enhanced` (adds pinyin
  search, ordering, drawer captions, the boot-handover rows).

## Next time

1. Fix or work around the Rockbox power-off (see above).
2. Merge upstream CompasOS into `mutiboot`, then rebase `cn-enhanced` onto it;
   bump `APP_RELEASE_LABEL` and the Sonix `OS_version` again.
3. Move the HiByOS-row commit (`7ac4747`) onto `mutiboot` — it is a dual-boot
   feature, not a Chinese-build one.
4. Publish the `mutiboot` project (repo + release notes in the style of v1.5.0,
   no implementation details).
