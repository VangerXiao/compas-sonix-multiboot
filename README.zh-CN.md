# Compás Player × SonixOS 双系统

[English](README.md) · **简体中文**

这是针对两个优秀的开源播放器——**CompásOS** 和 **SonixOS**——而开发的多启动修改版。两者都是各自作者的优秀作品，各有各的取舍。本项目**不替代任何一方，并且与两个上游项目没有任何隶属关系**。由于本项目修改了双方各自的部分内容，**可能会引入未知的错误和其他问题**；这些问题很可能是本项目导致的——除非刷回原版系统后仍能复现，否则请在本仓库提交 Issue。

## 这个仓库是什么

一个非官方的爱好者项目，让两个系统能在同一台设备上共存，刷一次即可长期切换，不必反复刷机。实测一台设备可以同时存放**三套系统**——CompásOS、SonixOS，以及通过 CompásOS 自带引导器启动的 Rockbox，并随意切换。

| 内容 | 代码位置 |
|---|---|
| CompásOS + 双启动（原版修改版） | [`VangerXiao/compas-player`](https://github.com/VangerXiao/compas-player) 分支 **`mutiboot`** |
| CompásOS + CN 增强版 | [`VangerXiao/compas-player`](https://github.com/VangerXiao/compas-player) 分支 **`cn-enhanced`** |
| SonixOS + "重启到 Compás" 电源菜单 | [`VangerXiao/sonix-player`](https://github.com/VangerXiao/sonix-player) 分支 **`compas-multiboot`** |
| 预编译镜像 | 本仓库的 [Releases](../../releases) |

`mutiboot` 是"上游 + 双启动"的纯净版本：不加任何外观改动、不改网络行为。
`cn-enhanced` 在它之上增加了在国内网络环境下更好用的几处修改，见下方"关于 CN_Enhanced 版本"。

## 切换是怎么工作的

CompásOS 放在内置存储，SonixOS 整个放在 TF 卡上。

```
开机
  |
  +-- 正常进入 CompásOS
  |     设置 → 系统 → "重启到 SonixOS"
  |        写入 /sonix/boot，重启
  |
  +-- CompásOS 的引导钩子看到 /sonix/boot
        挂载 /sonix/rootfs.squashfs，把 /sonix/data 绑定到 /usr/data，
        chroot 进入 SonixOS
              |
              +-- 电源菜单 → "滑动重启到 Compás"
                    写入 /usr/data/boot_compas，重启
                    钩子清除两个文件，CompásOS 启动
```

SonixOS 的设置存放在卡上的 `/sonix/data`，因此两个系统的用户数据相互隔离，不会冲突。

在 SonixOS 里**普通重启仍然回到 SonixOS**；要回 CompásOS 必须显式使用"滑动重启到 Compás"，离开是一个明确的选择。

## 安装

1. 把 Release 里的 `r1.upt` 放到 **TF 卡根目录**（文件名必须正好是 `r1.upt`，全小写）。
2. 把 Release 里的 Sonix 根文件系统放到卡上的 `/sonix/rootfs.squashfs`，并确保存在 `/sonix/data/` 目录（用来存放 SonixOS 的设置，可以先留空）。
3. 按住**音量+**，按一下**电源键**，继续按住，直到出现升级界面；等待刷写完成，设备会自动重启进入 CompásOS。
4. 设置 → 系统 → **重启到 SonixOS**，切换到 SonixOS。
5. 在 SonixOS 中呼出电源菜单，滑动 **"重启到 Compás"**，即可回到 CompásOS。

> **请始终在卡根目录保留一份可用的 `r1.upt`。** 原厂恢复通道（按住音量+开机）从未失手，它是唯一的救命通道。

## 注意事项

### 关于 CN_Enhanced 版本

CN_Enhanced 版本针对中国大陆部分地区 GitHub 访问不稳定的问题，为 GitHub 请求增加了 gh-proxy 代理（**默认开启**，可在 `/usr/data/.compas/github_mirror.txt` 中写入 `off` 关闭），另外修复了网易云插件登录 Cookie 被截断的问题，并把快捷抽屉的字号调整到与界面其它文字一致（默认档位 20 px）。

需要说明的是：**中文字体的显示能力来自上游 CompásOS 的官方实现**，不是本项目的贡献；本项目只调整了抽屉里的字号。

如果您不在中国大陆地区，请**使用不带 CN_Enhanced 标签的发行版**，否则 GitHub 访问可能会因为走代理而变慢。

### 关于在线更新

本镜像改变了两个系统的存储方式，**请不要使用任一系统内置的在线更新**：

- CompásOS 的固件更新会直接刷写内核与根文件系统分区，覆盖双启动钩子，并把内核换回原厂，双启动随即失效；
- SonixOS 的在线更新会替换卡上的播放器与文件。

上游发布新版本时，本项目需要重新合并上游代码并重新编译镜像。新镜像会发布在 Releases。

## 遇到问题

请在本仓库提 [Issue](../../issues)。

**请再次注意**，不要把这些问题反馈给 CompásOS 或 SonixOS 的作者——凡是本项目改动带来的问题，都由本项目负责。同理，这里的任何改动都不打算影响上游，只是下游分支。

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
- 感谢 [Jepl4r](https://github.com/Jepl4r) 与 SonixOS 社区。
- 本项目部分由 [DeepSeek Harness](https://github.com/deepseek-ai)（AI 编程助手）协助完成。
