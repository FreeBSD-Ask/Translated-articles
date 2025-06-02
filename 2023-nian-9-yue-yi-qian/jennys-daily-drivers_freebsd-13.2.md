# JENNY 日常使用的系统：FreeBSD 13.2

- 原地址 <https://hackaday.com/2023/08/01/jennys-daily-drivers-freebsd-13-2/>
- 作者：Jenny List
- 译者：ykla & ChatGPT

![image](https://github.com/FreeBSD-Ask/Translated-articles/assets/10327999/33485271-7297-4391-ae82-5c1d486b2a5e)


上个月我开始了一个新系列，尝试使用不同的操作系统来进行日常的工作，[我选择的是 Slackware 15](https://hackaday.com/2023/07/10/jennys-daily-drivers-slackware-15/)，这是我在 1990 年代中期时尝试的第一个 Linux 发行版的最新版本。我会在适当的时候返回来介绍更多基于 Linux 的操作系统，但这个系列的整个目的是尽可能地广泛涉猎，尝试每个合理的操作系统。因此，今天我将迈出清晰的第一步，尝试一个基于 BSD 的操作系统。对我来说，这是未知的领域，关于选择哪一个存在着相当大的分歧。因此，在阅读了相关资料后，我选择了 FreeBSD，因为它似乎是最易于使用的选择。

## 首先，是一些背景信息

![image](https://github.com/FreeBSD-Ask/Translated-articles/assets/10327999/8cde3a3d-cf16-450b-8ad2-b5dde29e2e32)

一台带有 FreeBSD 引导界面的计算机成功！这是我第一次看到运行中的 FreeBSD 安装界面。大多数读者都知道，[BSD 操作系统可以直接追溯到原始的 AT&T UNIX](https://hackaday.com/2019/11/05/will-the-real-unix-please-stand-up/)，而 GNU/Linux 则是一个相当好的 UNIX 复制品，起源于 1990 年代初的 Linus Torvalds，以及 1980 年代开始的 Richard Stallman 的 GNU 项目。这意味着对于 Linux 用户来说，需要适应于不同的术语。

Linux 是一个围绕其内核构建的操作系统，具有不同实现的用户空间组件，而各种 BSD 操作系统则是独立的操作系统。因此，我们谈论 Slackware 和 Debian 等不同的 Linux 发行版，但相比之下，[NetBSD](https://www.netbsd.org/) 和 FreeBSD 即使有共同的历史，也是不同的操作系统。虽然有一些如 [GhostBSD](https://www.ghostbsd.org/) 这样以 FreeBSD 为核心的 BSD 发行版，但在这个上下文中这样的情况较少见。所以，我用种子下载了 FreeBSD 13.2 的 USB 镜像文件，并将其写入了 U 盘。我拿出了 Hackaday 的测试计算机，开始进行试验。



## 出乎意料地易于安装

安装 FreeBSD 就像从 U 盘引导并运行安装脚本一样简单。基于文本的界面，使用起来感觉非常古老，但整个过程还是相当顺利的。有个选项可以用来自动对磁盘进行分区，然后选择一些基本服务进行安装（如果需要），然后它会进行安装过程。最后，你就拥有了一个正常运行的 FreeBSD 操作系统。

![image](https://github.com/FreeBSD-Ask/Translated-articles/assets/10327999/98af25ab-4530-414b-8c23-27ade8bbc11f)

一个原始的 X 窗口桌面

我已经很久没有看到原始的 X 窗口了。典型的流行的 Linux 发行版安装过程将尝试配置你的系统并安装所需的软件。因此，在设置过程中，你将创建用户，从庞大的软件库中进行选择，或者让它安装许多软件，其中将包括你所需的程序。很可能它还会配置自身以引导到图形桌面，在安装完成后，你只需继续使用它作为桌面机器。

如果你对操作系统的期望就是这样的话，可以说 FreeBSD 并不适合你，因为它的方法是为你提供一个空白的画布，你可以在上面绘就自己的故事。你会得到 FreeBSD，即一个命令提示符，你可以以 root 身份登录，仅此而已。

你的第一个任务是使用 `adduser` 添加一个日常用户，然后在为自己授予 sudo 特权之前，你必须安装 `sudo`。这将是你第一次使用 `pkg` 软件包管理器，作为长期使用类似 Linux 发行版软件包管理器的用户，我发现（FreeBSD）使用起来相当容易。我想要一个桌面环境，所以我再次使用了 pkg 安装 X，然后是 [桌面环境](https://docs.freebsd.org/en/books/handbook/desktop/)（我选择了 [Lumina](https://lumina-desktop.org/)，但有很多其他选择），以及诸如 Firefox、GIMP、OpenSCAD 和 KiCAD 等有用的软件。所有这些都不是很困难的，尽管我确实不得不搜索一些在线指南来配置桌面环境。

确保你的硬件足够新（但不要太新）一张 Nvidia GeForce GT520 显卡，它是不再受支持的古老显卡。

![image](https://github.com/FreeBSD-Ask/Translated-articles/assets/10327999/f8b87661-766f-42e5-ab62-b2994f9957de)


因此，尽管需要对 UNIX 或类似 UNIX 的操作系统有一点了解才能开始，但在 FreeBSD 上获得用于日常使用的桌面计算机还是相当简单的。这意味着我已经准备好写这篇文章了，只有一个例外。我的显卡是 Nvidia GT520，一款相当古老的显卡，曾被放入我的测试计算机中，来替换掉一个已经过时的新显卡。

FreeBSD 并不像一个功能完整的 Linux 发行版那样，从一开始就自带驱动程序，因此如果你有某些不常见的硬件，你需要自己找到并安装驱动程序。因此，在我能安装驱动程序之前，我被困在 VESA 分辨率中。这里我遇到了一个问题。[Nvidia 在他们的显卡支持方面做得很好](https://www.nvidia.com/en-us/drivers/unix/)，但是像这样古老的显卡早就不再得到他们的支持。我找到的最后一个支持它的驱动程序不能正常运行，因此我从未能够发挥出显卡的潜力。这并不是在批评 FreeBSD，而是我的显卡太古老了。

因此，我除了以一个非常复古的 1024 x 768 的 VESA 分辨率来编写这篇文章以外，我对 FreeBSD 的印象还是相当喜欢的。我喜欢它精简的安装，与典型的 Linux 发行版相比，你只需安装所有你需要的东西。我喜欢它的安装过程，这对于像我这样中等水平的 Linux 用户来说相对轻松。我喜欢它的速度，我发现它是一个非常可接受的日常选择。当然，还有一些 Linux 发行版的安装要困难得多。

我相信如果我有一块新一点的受支持的显卡，就能获得完整的分辨率，我甚至可能在另一台配置更好的计算机上安装这个操作系统，继续进行实验。然而，如果我对 FreeBSD 有什么不满，那就是针对新手的文档。我拥有多年的 Linux 使用经验，帮助我找到我需要的东西，但尽管安装过程相对轻松，我发现对我来说可能很难找到少数问题的答案。这绝对是个值得一试的操作系统，但如果你不是 UNIX 专家，你可能需要使用 Google-fu【译者注：指在 Google 上运用高超的搜索技巧来解决问题的能力】来获取帮助。去吧，试一试吧！


>译者注：实际上 freebsd 是支持 GT520 这块显卡的，你只需要 `pkg install nvidia-driver-390`，然后再执行 `sysrc kld_list+="nvidia-modeset"` 即可。
