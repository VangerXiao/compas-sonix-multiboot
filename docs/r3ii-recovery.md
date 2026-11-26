# 海贝 R3II 2025 线刷恢复指南

**简体中文** · [English](#english)

> **这份文档只针对 HiBy R3II 2025。** 它讲的是设备已经进不了任何系统（俗称变砖）时，如何用君正官方的 Cloner 工具通过 USB 把原厂固件写回去。如果你只是想切换系统或升级固件，请回到 [README](../README.md)。

## 什么时候需要线刷

满足下列任一情况，卡刷（「通过 SD 卡升级」）就已经帮不上忙了，只能线刷：

- 开机后屏幕不亮、卡在开机 logo，且拔卡也无法进原厂系统
- 进不了原厂系统的设置菜单

原因是结构性的：HiByOS 的卡刷入口在「设置」里，**需要先启动一个系统**，由它把包写进 flash；系统起不来，就没有任何东西能执行写入。

## 准备工作

| 项目 | 说明 |
| --- | --- |
| Windows 电脑 | Windows 10 / 11 均可 |
| USB-C 数据线 | **必须能传数据**，只能充电的线不行 |
| 君正 Cloner 工具 | `cloner-2.5.80.2-windows_alpha`，压缩包内附带 USB 驱动 |
| 板级配置 | 本仓库的 `hiby_x1600e_ota_sfc_nand_lpddr2_linux.cfg`，与本文件同目录 |
| 完整固件包 | 社区验证可用的线刷包（见下方「下载地址」），或官方 `HiBy_R3II_2025_1.3_*.ingenic` |

### 下载地址

- **Cloner 工具**：君正官方发布页的 `cloner-2.5.80.2-windows_alpha.zip`
- **社区线刷包（实测可用）**：`r3ii_cloner.zip` v1.1 —— 链接待补充
- **官方固件**：HiBy 官网发布页的 `HiBy_R3II_2025_1.3_*.ingenic`

> ⚠️ **官方 `.ingenic` 用 2.5.80.2 版 Cloner 直接刷，实测会报 `uboot INIT ERR`**：官方固件里带的是与新工具不匹配的 loader/uboot。社区线刷包 `r3ii_cloner.zip` 正是为此准备的（带可用的 spl/uboot 与配置），**优先用它**。

## 操作步骤

### 1. 安装驱动

解压 Cloner 压缩包，进入 `driver/` 目录（不同版本可能叫 `usb_driver`），按系统位数运行安装程序。安装后设备管理器中应能看到「Ingenic USB Device」，或至少不再有未知设备。

### 2. 放入板级配置

把本目录下的 `hiby_x1600e_ota_sfc_nand_lpddr2_linux.cfg` 复制到 Cloner 目录下的：

```
configs\x1600\hiby_x1600e_ota_sfc_nand_lpddr2_linux.cfg
```

Cloner 启动时会扫描这个目录，**必须重启 Cloner 才能在下拉框里看到它**。

### 3. 进入线刷模式

1. 让设备**完全关机**（长按电源键，确认屏幕熄灭）
2. **按住「下一曲」键不放**
3. 保持按住，把 USB 线插上电脑
4. 松开按键

此时电脑应发出设备连接提示音。Cloner 界面显示连接就绪，说明进入成功。

### 4. 开始刷写

1. 打开 Cloner，板级配置选择 `hiby_x1600e_ota_sfc_nand_lpddr2_linux`
2. 固件文件选择社区包里的 `.ingenic`（或官方同名文件）
3. 点击 `Start`，**期间不要拔线、不要锁屏、不要动电脑**
4. 进度条走完大约 1–3 分钟，出现 `Succeeded`（完成）即结束
5. 拔线，长按电源键开机

## 常见问题

| 现象 | 原因 | 处理 |
| --- | --- | --- |
| `uboot INIT ERR` | 固件包里的 loader 与工具版本不匹配（官方 `.ingenic` 常见） | 换社区线刷包 `r3ii_cloner.zip` |
| 一直「等待设备连接」 | 驱动没装好，或没进入线刷模式 | 重装驱动；确认是**完全关机**后按住「下一曲」再插线 |
| 下拉框里没有板级配置 | 配置文件放错目录 / 没重启 Cloner | 确认路径是 `configs\x1600\`，然后重启工具 |
| 刷到一半失败 | 线材接触不良、USB 口供电不足 | 换线、换后置 USB 口，重新进入线刷模式再刷一次 |
| 刷完仍然不开机 | 固件包不完整 | 重新下载（注意校验文件大小），或改用官方包 |

## 刷完之后

线刷会把**整个 flash 恢复成原厂单系统状态**：多启动钩子与卡上的系统镜像都不再被使用，开机就是原厂 HiByOS。

想回到多启动：

1. 按 [README](../README.md) 的说明，把 `r3ii.upt` 放到卡根，用「通过 SD 卡升级」刷入
2. 再解压对应版本的卡上系统包（`r3ii_mutiboot.zip` 或 `r3ii_cnenhanced.zip`）
3. 开机按住「播放/暂停」进入菜单选择系统

---

## English

> **This page is about the HiBy R3II 2025 only.** It covers recovering a device that cannot start any system ("bricked") by writing the stock firmware back over USB with Ingenic's Cloner. To switch systems or update firmware, see the [README](../README.md).

### When you need this

Card updates cannot help in either case:

- the device does not boot, or hangs on the boot logo even without a card;
- you cannot reach the stock system's settings menu.

The reason is structural: HiByOS performs a card update **from its settings menu**, so it needs a running system to write the image into flash. No system, nothing to do the writing.

### What you need

| Item | Notes |
| --- | --- |
| Windows PC | Windows 10 or 11 |
| USB-C cable | a **data** cable; charge-only cables do not work |
| Ingenic Cloner | `cloner-2.5.80.2-windows_alpha`; its USB driver is in the archive |
| Board config | this directory's `hiby_x1600e_ota_sfc_nand_lpddr2_linux.cfg` |
| Full firmware | the community package that is known to work (below), or HiBy's `HiBy_R3II_2025_1.3_*.ingenic` |

### Where to download

- **Cloner**: `cloner-2.5.80.2-windows_alpha.zip` from Ingenic's release page;
- **Community package (verified working)**: `r3ii_cloner.zip` v1.1 — link to be added;
- **Stock firmware**: `HiBy_R3II_2025_1.3_*.ingenic` from HiBy's release page.

> ⚠️ **HiBy's own `.ingenic` fails on Cloner 2.5.80.2 with `uboot INIT ERR`**: it carries a loader/uboot pair that version does not accept. The community package `r3ii_cloner.zip` exists for exactly this reason (it ships a working spl/uboot and config); **use it first**.

### Steps

1. **Install the driver** from the Cloner archive (`driver/`, or `usb_driver/` in some versions).
2. **Copy the board config** to `configs\x1600\hiby_x1600e_ota_sfc_nand_lpddr2_linux.cfg` inside the Cloner folder. Cloner scans that folder at startup, so **restart it** to see the entry.
3. **Enter flash mode**: power the device fully off, hold **next track**, plug the USB cable in while holding, then release.
4. **Flash**: pick `hiby_x1600e_ota_sfc_nand_lpddr2_linux` as the board config, choose the `.ingenic` file, press `Start`. Do not unplug, lock the screen or touch the machine during the 1–3 minutes it takes. Unplug and power on when it reports success.

### Troubleshooting

| Symptom | Cause | Fix |
| --- | --- | --- |
| `uboot INIT ERR` | the loader in the package does not match the tool version (common with HiBy's `.ingenic`) | use the community `r3ii_cloner.zip` |
| Stuck at "waiting for device" | driver missing, or the device is not in flash mode | reinstall the driver; power off completely, hold **next track**, then plug in |
| Board config missing from the list | wrong folder, or Cloner was not restarted | check `configs\x1600\`, restart the tool |
| Fails part way through | bad cable or weak USB port | change the cable/port, enter flash mode again and retry |
| Still will not boot after flashing | incomplete firmware package | download it again (check the file size) or use HiBy's package |

### After flashing

The flash is the stock single-system layout again: the multi-boot hook and the card images are unused, and the device boots HiByOS.

To return to multi-boot:

1. put `r3ii.upt` in the card root and flash it with **Settings → Firmware update → via SD card**;
2. unzip the card archive for the variant you want (`r3ii_mutiboot.zip` or `r3ii_cnenhanced.zip`) into the card root;
3. hold **PLAY/PAUSE** while powering on to reach the boot menu.
