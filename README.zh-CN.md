# Compás Player × SonixOS 双系统

两个出色的开源播放器，一台 HiBy R1，可以在它们之间自由切换——不用拆机、不用拔卡、不用反复刷机。

> [English](README.md) · **简体中文**

- **[CompásOS](https://github.com/Starnished66/compas-player)** —— 功能完整的日常播放器：界面精致、格式支持广、有插件生态、设置层级清晰，社区一直在持续打磨。
- **[SonixOS](https://github.com/Jepl4r/sonix-player)** —— 专注而轻量的播放器：界面漂亮、体积小、响应快，把"播放音乐"这件事做得干净利落。

两者都是各自作者的优秀作品，各有各的取舍。本项目不替代任何一方，与两个上游项目**没有任何隶属关系**，也不会往它们的源码里添加任何东西。

## 这个仓库是什么

一个**非官方的爱好者项目**，把两个系统打包成能在同一台设备上共存的样子：

| 内容 | 代码位置 |
|---|---|
| CompásOS + 双启动菜单（原版修改版） | [`VangerXiao/compas-player`](https://github.com/VangerXiao/compas-player) 分支 **`mutiboot`** |
| CompásOS + 国内增强版（GitHub 反代、网易云修复、`v1.4.1CN` 标识） | [`VangerXiao/compas-player`](https://github.com/VangerXiao/compas-player) 分支 **`cn-enhanced`** |
| SonixOS + "重启到 Compás" 电源菜单 | [`VangerXiao/sonix-player`](https://github.com/VangerXiao/sonix-player) 分支 **`compas-multiboot`** |
| 预编译镜像 | 本仓库的 [Releases](../../releases) |

`mutiboot` 是"上游 + 双启动"的纯净版本：不加任何外观改动、不改网络行为。
`cn-enhanced` 在它之上增加了在国内网络环境下更好用的几处修改：可选的 GitHub 反代（用于更新检查和两个商店）、以及修复网易云登录 Cookie 被截断的问题。

## 切换是怎么工作的

CompásOS 留在内部闪存，永远是默认系统；SonixOS 整个住在 SD 卡上。

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

在 SonixOS 里普通重启会**留在 SonixOS**——离开是一个明确的选择，而不是意外。

SonixOS 的设置存放在卡上的 `/sonix/data`，所以一次 SonixOS 会话不可能动到内部存储里的 CompásOS 数据。

## 恢复通道始终可用

真正实现"按住音量+开机、从卡上刷 `r1.upt`"的那两个恢复分区**从头到尾没有被修改过**。请始终在卡根目录保留一份可用的 `r1.upt`——它是从未失手过的救命通道。

**请永远在卡上保留 `r1.upt`。**

## 安装

1. 把 Release 里的 `sonix/rootfs.squashfs` 放到卡上的 `/sonix/rootfs.squashfs`，并在旁边建一个 `/sonix/data/` 目录。
2. 把 Release 里的 `r1.upt` 放到卡根目录。
3. 按住**音量+**、按一下**电源键**、继续按住，直到出现升级界面；等它刷完后设备会重启进入 CompásOS。
4. 设置 → 系统 → **重启到 SonixOS**，即可体验另一个系统。

## 遇到问题

请在本仓库提 [Issue](../../issues)。

**请不要**把这些问题反馈给 CompásOS 或 SonixOS 的作者——凡是本项目改动带来的问题，都由本项目负责。同理，这里的任何改动都不打算合并回上游，只是下游分支。

## 跟随上游

本 fork 持续跟随两个上游。我们的改动只是在 `upstream/main` 之上的少量提交，所以吃到上游更新就是一次 rebase：

```sh
git remote add upstream https://github.com/Starnished66/compas-player.git
git fetch upstream
git checkout main && git merge --ff-only upstream/main && git push origin main
git checkout mutiboot && git rebase upstream/main && git push --force-with-lease
git checkout cn-enhanced && git rebase mutiboot && git push --force-with-lease
```

（另一个仓库同理，把地址换成 `Jepl4r/sonix-player`，分支换成 `compas-multiboot`。）

想给**上游**提 PR 时，请从 `upstream-prs` 分支开始——它永远干净地基于上游 `main`，不会把双启动的改动混进 PR 里。

## 许可

CompásOS 与 SonixOS 均为 GPLv3。这些 fork 保留各自的许可证与声明；按照许可证要求，所有修改都在上面列出的分支里公开。Release 中的镜像正是由这些分支构建的。

## 致谢

- 感谢 [Starnished66](https://github.com/Starnished66) 与 CompásOS 社区，把一个播放器做成了真正的操作系统级体验。
- 感谢 [Jepl4r](https://github.com/Jepl4r) 与 SonixOS，证明了轻量也可以很美。
- 本项目的打包、分支整理与文档由 [DeepSeek Harness](https://github.com/deepseek-ai)（AI 编程助手）协助完成。
