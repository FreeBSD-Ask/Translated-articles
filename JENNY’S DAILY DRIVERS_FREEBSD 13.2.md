# JENNY 日常使用的系统：FreeBSD 13.2

- 原地址 <https://hackaday.com/2023/08/01/jennys-daily-drivers-freebsd-13-2/>
- 作者：Jenny List
- 译者：ykla & ChatGPT

![image](https://github.com/FreeBSD-Ask/Translated-articles/assets/10327999/33485271-7297-4391-ae82-5c1d486b2a5e)


上个月我开始了一个系列，尝试使用不同的操作系统来进行日常工作，[我选择的是 Slackware 15](https://hackaday.com/2023/07/10/jennys-daily-drivers-slackware-15/)，这是我在 1990 年代中期尝试的第一个 Linux 发行版的最新版本。我会在适当的时候回来介绍更多基于 Linux 的操作系统，但这个系列的整个目的是尽可能地广泛漫游，尝试每个合理的操作系统。因此，今天我将迈出一个显而易见的第一步，尝试一个基于 BSD 的操作系统。对我来说，这是未知的领域，关于选择哪一个存在着相当大的选择。因此，在阅读了相关资料后，我选择了 FreeBSD，因为它似乎是最易于访问的选项。

## 首先，是一些背景信息

一台带有 FreeBSD 引导界面的计算机成功！我第一次看到一个工作中的 FreeBSD 安装。大多数读者都会知道，[BSD 操作系统可以直接追溯到原始的 AT&T UNIX](https://hackaday.com/2019/11/05/will-the-real-unix-please-stand-up/)，而 GNU/Linux 则是一个相当好的 UNIX 克隆，起源于 1990 年代初的 Linus Torvalds，以及 1980 年代开始的 Richard Stallman 的 GNU 项目。这意味着对于 Linux 用户来说，需要适应不同的术语。

Linux 是一个围绕其核心构建的操作系统，具有不同实现的用户空间组件，而各种 BSD 操作系统则是独立的操作系统。因此，我们谈论 Slackware 和 Debian 等不同的 Linux 发行版，但相比之下，[NetBSD](https://www.netbsd.org/) 和 FreeBSD 即使有共同的历史，也是不同的操作系统。虽然有一些如 [GhostBSD](https://www.ghostbsd.org/) 这样以 FreeBSD 为核心的 BSD 发行版，但在这个上下文中这样的情况较少见。所以，我从种子下载了 FreeBSD 13.2 的 USB 镜像文件，并将其写入了 USB 闪存驱动器。拿出 Hackaday 的测试计算机，开始进行试验。

![image](https://github.com/FreeBSD-Ask/Translated-articles/assets/10327999/8cde3a3d-cf16-450b-8ad2-b5dde29e2e32)

## 出乎意料地易于安装

安装 FreeBSD 就像从活动的 USB 驱动器引导并运行安装脚本一样简单。使用基于文本的界面，感觉非常古老，但整个过程还是相当顺利的。有一个选项可以自动分区磁盘，然后选择一些基本服务进行安装（如果需要），然后它会进行安装过程。最后，你就拥有了一个工作中的 FreeBSD 操作系统。

![image](https://github.com/FreeBSD-Ask/Translated-articles/assets/10327999/98af25ab-4530-414b-8c23-27ade8bbc11f)

一个原始的 X 窗口桌面

我已经很久没有看到原始的 X 窗口了。一个典型的流行 Linux 发行版安装将尝试配置您的系统并安装所需的软件。因此，在设置过程中，您将创建用户，从庞大的软件库中进行选择，或者让它安装许多软件，其中将包括您所需的程序。很可能它还会配置自身以引导到图形桌面，在安装完成后，您只需继续使用它作为桌面机器。

如果您对操作系统的期望就是这样的话，可以说 FreeBSD 不适合您，因为它的方法是为您提供一个空白的画布，您可以在上面书写自己的故事。您会得到 FreeBSD，一个命令提示符，您可以以 root 身份登录，仅此而已。

您的第一个任务是使用 adduser 添加一个日常用户，然后在为自己授予 sudo 特权之前，您必须安装 sudo。这将是您第一次使用 pkg 软件包管理器，作为长期使用类似 Linux 发行版软件包管理器的用户，我发现使用起来相当容易。我想要一个桌面环境，所以我再次使用 pkg 安装 X，然后是[桌面环境](https://docs.freebsd.org/en/books/handbook/desktop/)（我选择了 [Lumina](https://lumina-desktop.org/)，但有很多选择），以及诸如 Firefox、GIMP、OpenSCAD 和 KiCAD 等有用的应用程序。所有这些都不是特别具有挑战性的，尽管我确实不得不搜索一些在线指南来配置桌面环境。

确保您的硬件足够新（但不要太新）
一张 Nvidia GeForce GT520 显卡

![image](https://github.com/FreeBSD-Ask/Translated-articles/assets/10327999/f8b87661-766f-42e5-ab62-b2994f9957de)

不再支持的古老显卡

因此，尽管需要对 UNIX 或类似 UNIX 的操作系统有一点了解才能开始，但在 FreeBSD 中获得用于日常使用的桌面计算机还是相当简单的。这意味着我已经准备好写这篇文章了，只有一个例外。我的显卡是 Nvidia GT520，一款相当古老的 GPU，曾被放入我的测试计算机中，以替换掉一个已经过时的年轻卡。

FreeBSD 并不像一个功能完整的 Linux 发行版那样，从一开始就自带驱动程序，因此如果您有某些不寻常的设备，您需要自己找到并安装驱动程序。因此，在我能安装驱动程序之前，我被困在 VESA 分辨率中。这里我遇到了一个问题。[Nvidia 在支持他们的显卡方面做得很好](https://www.nvidia.com/en-us/drivers/unix/)，但是像这样古老的显卡早就不再得到他们的支持。我找到的最后一个支持它的驱动程序不想合作，因此我从未能够发挥出显卡的潜力。这并不是在批评 FreeBSD，而是显卡太古老了。

因此，除了以一个
