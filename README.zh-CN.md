# 针对Hiby R1的多系统启动

[English](README.md) · **简体中文**

这是针对几个优秀的开源播放器——**CompásOS** 和 **SonixOS**,以及**Rockbox**  而开发的多启动修改版。这些都是各自作者的优秀作品，各有各的取舍。本项目**不替代任何一方，并且与上游项目没有任何隶属关系**。由于本项目修改了各自的部分内容，**可能会引入未知的错误和其他问题**；这些问题很可能是本项目导致的——除非刷回原版系统后仍能复现，否则**请在本仓库提交 Issue**。

## 这个仓库是什么

一个非官方的实验性项目，让多个系统能在同一台设备上共存，刷一次即可长期切换，不必反复刷机。现在你的一台设备可以同时存放**四套系统**——原厂的HibyOS,CompásOS、SonixOS，以及Rockbox，并随意切换了!

| 内容                    | 代码位置                                                                                            |
| ----------------------- | --------------------------------------------------------------------------------------------------- |
| CompásOS （原版修改版） | [`VangerXiao/compas-player`](https://github.com/VangerXiao/compas-player) 分支 **`mutiboot`**       |
| CompásOS  CN 增强版     | [`VangerXiao/compas-player`](https://github.com/VangerXiao/compas-player) 分支 **`cn-enhanced`**    |
| SonixOS 修改版          | [`VangerXiao/sonix-player`](https://github.com/VangerXiao/sonix-player) 分支 **`compas-multiboot`** |
| Rockbox 修改版 | [`VangerXiao/rockbox`](https://github.com/VangerXiao/rockbox) 分支 **`rockbox_mutiboot_fix`** |
| 预编译镜像                         | 本仓库的 [Releases](../../releases)                                                                 |

`mutiboot` 是"上游 + 双启动"的纯净版本：不加任何外观改动、不改网络行为。
`cn-enhanced` 在它之上增加了在国内网络环境下更好用的几处修改，见下方"关于 CN_Enhanced 版本"。

## 切换系统
按下音量-,同时按下开机按钮,设备会弹出引导界面.如果下次直接按下电源键开机,设备会记住上次的启动系统.如果缺少TF卡,或者TF卡内缺少对应的系统文件,将会默认启动到hibyOS


## 为rockbox添加了新的修复

rockbox目前已经支持800*480P MPEG1视频的播放.
使用如下视频转换方式:
转码命令见 [`diy/docs/video-transcode.md`](https://github.com/VangerXiao/rockbox/blob/diy/r1-erosq/diy/docs/video-transcode.md)。多启动结构下 Rockbox 的关机也改成了直接调用 `reboot(2)`，短按电源键即可真正关机。
rockbox的picture flow按键映射现在已经得到修复,现在按下音量+/-直接翻页,按下播放/暂停进入专辑.

## 切换是怎么工作的



设备的启动流程是 `内核 → /linuxrc → rcS 里的 init 脚本`。本项目的钩子 `S20switch` 插在 rcS 早期：它挂载 TF 卡、读取卡根目录的 `boot_target.txt`，再决定是留在 HiByOS，还是 chroot 进入卡上的某个 `rootfs.squashfs`。卡上的系统都是只读镜像，挂在 `/tmp/target` 后 chroot 进去，所以彼此互不干扰。
```

SonixOS 的设置存放在卡上的 `/sonix/data`，因此两个系统的用户数据相互隔离，不会冲突。

两个系统的普通重启都回到自己；要回原厂请用设置里的"重启到 HiByOS"。

## 安装

1. 准备 TF 卡，卡根放 `compas/rootfs.squashfs`、`sonix/rootfs.squashfs` 与 `.rockbox/`。
2. 用 Release 里的 `r1.upt` 刷入内核、原厂根文件系统与启动钩子。
3. 开机按住音量− 进菜单选择系统；之后直接按电源键就进上次选中的系统。

> 如果遇到意外(或者想刷回某个系统),**将该系统包放置到根目录并重命名为 `r1.upt`。** 原厂恢复通道（按住音量+开机）仍然生效。

## 注意事项

### 关于 CN_Enhanced 版本

CN_Enhanced 版本针对中国大陆部分地区 GitHub 访问不稳定的问题，为 GitHub 请求增加了 gh-proxy 代理（**默认开启**，可在 `/usr/data/.compas/github_mirror.txt` 中写入 `off` 关闭），并把快捷抽屉的字号稍微调大以增强可读性（默认档位 20 px），拓展了部分字体的覆盖范围。另外新增了**拼音首字母搜索**：在曲库搜索框里输入 `zjl` 即可找到「周杰伦」，输入 `wyt` 找到「五月天」。该功能只存在于 CN_Enhanced 版本。另外新增了**拼音首字母搜索**：在曲库搜索框里输入 `zjl` 即可找到「周杰伦」，输入 `wyt` 找到「五月天」。该功能只存在于 CN_Enhanced 版本。

如果您不在中国大陆地区，请**使用 `mutiboot` 发行版**，或**彻底忽略带有 `CN_Enhanced` 的版本**，否则可能带来未知的网络问题、日文歌名显示错乱或其他问题。

### 关于在线更新

本镜像改变了两个系统的存储方式，**请不要使用任一系统内置的在线更新**：

- CompásOS 的固件更新会直接刷写内核与根文件系统分区，覆盖双启动钩子，并把内核换回原厂，双启动随即失效；
- SonixOS 的在线更新会替换卡上的播放器与文件。

上游发布新版本时，本项目需要重新合并上游代码并重新编译镜像。新镜像会发布在 Releases。双系统的在线更新的实现可能在未来得到解决.

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
- 感谢 [Jepl4r](https://github.com/Jepl4r) 与 SonixOS 社区。
- 本项目部分由 [DeepSeek Harness](https://github.com/deepseek-ai)（AI 编程助手）协助完成。
