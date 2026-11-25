# HiBy R1 与 R3II 2025 的多系统启动

[English](README.md) · **简体中文**

这是针对几个优秀的开源播放器——**CompásOS**、**SonixOS**，以及 **Rockbox**——而开发的多启动修改版。这些都是各自作者的优秀作品，各有各的取舍。本项目**不替代任何一方，并且与上游项目没有任何隶属关系**。由于本项目修改了各自的部分内容，**可能会引入未知的错误和其他问题**；这些问题很可能是本项目导致的——除非刷回原版系统后仍能复现，否则**请在本仓库提交 Issue**。

| 设备 | 可启动的系统 | 进入菜单 |
| --- | --- | --- |
| **HiBy R1** | HiByOS、CompásOS、SonixOS、Rockbox | 音量− + 电源 |
| **HiBy R3II 2025** | HiByOS、CompásOS、SonixOS | 播放/暂停 + 电源 |

## ⚠️ 风险提示（仅 R3II 2025）

海贝官方所述的「按下播放键并开机进入恢复模式」，经分析原厂固件与实机验证，**均不可行**，目前尚未发现能进入官方 Updater 的组合按键。

Release 中的固件都经过了我的实机测试，但仍可能在极端情况下启动失败。**一旦失败且无法进入任何一个系统，设备就会变砖。** 开发过程中我的 R3II 确实变砖过，最终是用君正官方的 Cloner 工具线刷恢复的——但这个操作远比 R1 那种「进内置 Updater 刷回」麻烦得多。

因此，如果你是 R3II 2025 用户，请自行衡量风险；必要时请先学会用线刷工具刷回原厂系统。**HiBy R1 用户可以忽略这条提示**：R1 有可用的内置恢复通道。

## 这个仓库是什么

一个非官方的**实验性**项目，让多个系统能在同一台设备上共存，刷一次即可长期切换，不必反复刷机。现在你的一台设备可以同时存放**四套系统**——原厂的 HiByOS、CompásOS、SonixOS，以及 Rockbox（仅 HiBy R1）——并随意切换。

| 内容 | 代码位置 |
| --- | --- |
| CompásOS（原版修改版） | [`VangerXiao/compas-player`](https://github.com/VangerXiao/compas-player) 分支 **`mutiboot`** |
| CompásOS CN 增强版 | [`VangerXiao/compas-player`](https://github.com/VangerXiao/compas-player) 分支 **`cn-enhanced`** |
| SonixOS 修改版 | [`VangerXiao/sonix-player`](https://github.com/VangerXiao/sonix-player) 分支 **`compas-multiboot`** |
| Rockbox 修改版 | [`VangerXiao/rockbox`](https://github.com/VangerXiao/rockbox) 分支 **`rockbox_mutiboot_fix`** |
| 预编译镜像 | 本仓库的 [Releases](../../releases) |

`mutiboot` 是「上游 + 多启动」的纯净版本：不加任何外观改动、不改网络行为。
`cn-enhanced` 在它之上增加了在国内网络环境下更好用的几处修改，见下方「关于 CN_Enhanced 版本」。

## 安装

1. 准备 TF 卡，卡根放 `compas/rootfs.squashfs`、`sonix/rootfs.squashfs`（R1 另需 `.rockbox/`）。
2. 用 Release 里对应设备的 `*.upt` 刷入内核、原厂根文件系统与启动钩子。
3. 开机时按住对应按键进入菜单选择系统；之后直接按电源键就会进入上次选中的系统。

> 遇到意外（或想刷回某个系统）时，**把该系统包放到卡根并按下面命名**：
> - HiBy R1：重命名为 `r1.upt`，然后按住音量+ 开机进入内置 Updater；
> - HiBy R3II 2025：重命名为 `r3ii_2025.upt`，用系统内的「通过 SD 卡升级」刷入。
>
> R3II 没有可用的按键入口，请务必保留一个能启动的系统，或准备好线刷工具。

## 切换系统

| 设备 | 进入菜单 | 菜单内操作 |
| --- | --- | --- |
| **HiBy R1** | 按住「音量−」再按电源 | 音量 +/− 切换，任意其它键确认 |
| **HiBy R3II 2025** | 按住「播放/暂停」再按电源 | 滚轮切换，播放/暂停确认 |

菜单会把选择写入卡上的 `boot_target.txt`，下次直接按电源键就进入该系统。缺少 TF 卡、或卡内缺少对应系统文件时，会默认启动 HiByOS。

## 为 Rockbox 添加的修复（仅 HiBy R1）

- 支持 **800×480 MPEG1 视频**播放，转码方式见 [`diy/docs/video-transcode.md`](https://github.com/VangerXiao/rockbox/blob/diy/r1-erosq/diy/docs/video-transcode.md)。
- PictureFlow 按键映射已修复：音量 +/− 翻页，播放/暂停进入专辑。
- 多启动结构下，Rockbox 的关机改为直接调用 `reboot(2)`，**短按电源键即可真正关机**。

## 切换是怎么工作的

设备的启动流程是 `内核 → /linuxrc → rcS 里的 init 脚本`。本项目的钩子 `S20switch` 插在 rcS 早期：它挂载 TF 卡、读取卡根的 `boot_target.txt`，再决定是留在 HiByOS，还是 chroot 进入卡上的某个 `rootfs.squashfs`。卡上的系统都是只读镜像，挂在 `/tmp/target` 后 chroot 进去，所以彼此互不干扰。

各系统的普通重启都回到自己；要回原厂，用设置里的「重启到 HiByOS」。

钩子的每一个失败路径都会**退回原厂启动**（卡不在、镜像缺失、标记非法、挂载失败都会 `exit 0`），因此不存在「钩子坏了就进不了任何系统」的情况。

## 数据存放在哪

| 系统 | 系统本体 | 用户数据 |
| --- | --- | --- |
| 原厂 HiByOS | 设备内置存储 | 设备内置存储 |
| CompásOS | TF 卡 | 设备内置存储（与 HiByOS 共用） |
| SonixOS | TF 卡 | **TF 卡**（`sonix/data`，与其它系统隔离） |
| Rockbox（R1） | TF 卡 `.rockbox/` | TF 卡 |

CompásOS 是 HiByOS 的分支，读写的是同一套数据格式，所以和原厂共用（这与「把 Compás 刷到原厂机器上」完全等价）；SonixOS 是另一条产品线，且它的挂载程序在不认识现有数据格式时会**重建用户数据分区**，因此单独放在卡上。

## 关于 CN_Enhanced 版本

CN_Enhanced 版本针对中国大陆部分地区 GitHub 访问不稳定的问题，为 GitHub 请求增加了 gh-proxy 代理（**默认开启**，可在 `/usr/data/.compas/github_mirror.txt` 中写入 `off` 关闭），并把快捷抽屉的字号稍微调大以增强可读性（默认档位 20 px），拓展了部分字体的覆盖范围。

另外新增了**拼音首字母搜索**：在曲库搜索框里输入 `zjl` 即可找到「周杰伦」，输入 `wyt` 找到「五月天」；曲库与 A-Z 栏也按拼音首字母排序。该功能只存在于 CN_Enhanced 版本。

如果您不在中国大陆地区，请**使用 `mutiboot` 发行版**，或**彻底忽略带有 `CN_Enhanced` 的版本**，否则可能带来未知的网络问题、日文歌名显示错乱或其他问题。

## 关于在线更新

本镜像改变了系统的存储方式，**请不要使用任一系统内置的在线更新**：

- CompásOS 的固件更新会直接刷写内核与根文件系统分区，覆盖多启动钩子，并把内核换回原厂，多启动随即失效；
- SonixOS 的在线更新会替换卡上的播放器与文件。

上游发布新版本时，本项目需要重新合并上游代码并重新编译镜像，新镜像会发布在 Releases。在线更新与多启动共存的问题可能在未来得到解决。

## 遇到问题

请在本仓库提 [Issue](../../issues)。

**请再次注意**，不要把这些问题反馈给 CompásOS 或 SonixOS 的作者——凡是本项目改动带来的问题，都应当由本项目负责。同理，这里的任何改动都不打算影响上游，只是下游分支。

## 跟随上游

本 fork 持续跟随两个上游。我们的改动只是在 `upstream/main` 之上的少量提交，所以每次上游更新可以执行一次 rebase：

```sh
git remote add upstream https://github.com/Starnished66/compas-player.git
git fetch upstream
git checkout main && git merge --ff-only upstream/main && git push origin main
git checkout mutiboot && git rebase upstream/main && git push --force-with-lease
git checkout cn-enhanced && git rebase mutiboot && git push --force-with-lease
```

（另一个仓库同理，把地址换成 `Jepl4r/sonix-player`，分支换成 `compas-multiboot`。）

## 许可

CompásOS 与 SonixOS 均为 GPLv3。这些 fork 保留各自的许可证与声明；按照许可证要求，所有修改都在上面列出的分支里公开。Release 中的镜像正是由这些分支构建的。

## 致谢

- 感谢 [Starnished66](https://github.com/Starnished66) 与 CompásOS 社区。
- 感谢 [Jepl4r](https://github.com/Jepl4r) 与 SonixOS 社区。原仓库已无法访问，Sonix 社区现由 [Tartarus6/sonix-player](https://github.com/Tartarus6/sonix-player) 继续维护；HiBy R3II 2025 的移植来自 [unrau/sonix-player](https://github.com/unrau/sonix-player)，一并感谢。
- 本项目部分由 AI 协助完成（[DeepSeek Harness](https://github.com/deepseek-ai)）。
