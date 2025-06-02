# OpenBSD 与 FreeBSD：有何不同，哪个更好？

——是否对选择下一个项目中使用 OpenBSD 还是 FreeBSD 感到困惑？我们将比较这两种流行的基于 BSD 的操作系统。

- 原文链接：[OpenBSD vs. FreeBSD: What Is the Difference, Which Is Best?](https://www.makeuseof.com/openbsd-vs-freebsd-what-is-the-difference/)
- 作者：[DAVID DELONY](https://www.makeuseof.com/author/david-delony/)
- 发布时间：2023.9.22
- 译者：ykla & ChatGPT

>**主要要点**
>
>**OpenBSD 和 FreeBSD 在安装过程上有所不同，OpenBSD 假定用户具备更多的计算机经验，（安装）界面较为简约，而 FreeBSD 则具有更为精致的安装程序。**
>
>**OpenBSD 专注于安全性，并以其对代码正确性的承诺而闻名。另一方面，FreeBSD 被定位为通用系统，有着支持点对点互联网基础设施的历史。**
>
>**FreeBSD 拥有更好的文档，包括一本写得很好的手册，既可以作为专家的参考，也可以作为初学者的入门指南。而 OpenBSD 的文档更面向专家，并在视觉上根植于上世纪 90 年代。**

[OpenBSD](https://www.makeuseof.com/what-is-openbsd/) 和 [FreeBSD](https://redirect.viglink.com/?format=go&jsonp=vglnk_169555652387021&key=eac202ea7a96cf485281d6c4ffa2069e&libId=lmxdwmxz0103es17000ULaqhkd35z&loc=https%3A%2F%2Fwww.makeuseof.com%2Fopenbsd-vs-freebsd-what-is-the-difference%2F&ccpaConsent=1YNY&v=1&opt=true&optExText=false&out=https%3A%2F%2Fwww.freebsd.org%2F&ref=https%3A%2F%2Fwww.google.com%2F&title=OpenBSD%20vs.%20FreeBSD%3A%20What%20Is%20the%20Difference%2C%20Which%20Is%20Best%3F&txt=FreeBSD) 都是原始的服务器专用的伯克利软件发行版（BSD）的后继者，该发行版在 20 世纪 70 年代末至 90 年代由加州大学伯克利分校（UC Berkeley）开发。

作为流行的开源项目，它们有着忠实的用户群，被视为 Linux 的替代选择。它们有哪些不同之处，哪一个可能更适合你呢？

## 安装过程：OpenBSD 与 FreeBSD

OpenBSD 和 FreeBSD 都有安装程序，可让你在计算机上分区并安装这些系统，安装过程类似于 Linux 发行版。然而，安装这两个操作系统的体验是非常不同的。

![image](https://github.com/FreeBSD-Ask/Translated-articles/assets/10327999/4f8917f3-6b27-4030-ab98-7416822c8dd2)

OpenBSD 假定用户具备丰富的计算机经验，其（安装）界面相对简约。你需要从其网站下载安装镜像，就像在下载 Linux 时一样，然后将其 [提取到存储设备](https://www.makeuseof.com/tag/extract-iso-files-linux/) 中，并启动计算机。

当你的计算机启动时，你会看到一个控制台界面。与主要的 Linux 发行版不同，OpenBSD 没有图形化的安装程序和 live CD。你需要从终端运行安装程序，并回答有关键盘类型和区域的问题。

如果你犯了错误，你唯一能做的就是按下 **Ctrl + C**，然后重新开始。这个安装程序类似于专注于专家用户的发行版——如 Arch 或 Gentoo。

之后，你将选择要包含在系统中的“文件集”。开发人员建议初次使用者使用默认设置。然后，你将配置 root 密码以及其他用户。接下来，关键时刻来临，你将启动新的操作系统。

![image](https://github.com/FreeBSD-Ask/Translated-articles/assets/10327999/3d37befc-ff49-410d-86d0-94a35a74ac1e)

FreeBSD 的安装程序类似，但（安装）界面更为精致。不过，它仍然是基于文本的。如果你在上世纪 90 年代曾在 MS-DOS 系统上安装过游戏，那么 FreeBSD 的安装程序会让你感到熟悉。

它还会引导你完成 FreeBSD 机器的设置，如格式化分区（FreeBSD 称之为 "slices"）、选择软件、设置互联网连接以及设置用户和时区。

尽管它看起来更友好，但 FreeBSD 的安装程序也假定用户熟悉类 Unix 操作系统，就像 OpenBSD 一样。

与 OpenBSD 相比，FreeBSD 在具有更直观的安装程序方面具有优势，如果你以前安装过操作系统，你可以在没有手册的情况下完成安装。

## 用途和应用领域

OpenBSD 和 FreeBSD 都源自 386BSD 项目，该项目旨在将 BSD 源代码移植到 Intel 80386 处理器上，但它们面向两个不同的市场。

OpenBSD 由 Theo De Raadt 创建，此前他与其他 NetBSD 开发者发生了多次分歧。NetBSD 本身是 386BSD 的另一个分支。

OpenBSD 以其对安全性的关注而闻名。截至 2023 年 9 月，该项目的官方网站声称在默认安装中已经在很长一段时间内只发现了两个远程漏洞。

在上世纪 90 年代的互联网繁荣时期，OpenBSD 最初因在小型 ISP 中使用二手零件从头开始构建路由器和网关而广受欢迎。随着互联网服务的集中化和专业化，专用硬件变得更加普遍，但 OpenBSD 通过强调代码正确性来保持对安全性的关注。

OpenBSD 对技术质量的承诺很可能是其组件被移植到其他系统（如 OpenSSH 和 tmux）并在 OpenBSD 生态系统之外广泛流行的原因。甚至在 Windows 10 和 11 上，都默认安装了 OpenSSH。

另一方面，尽管开发者侧重于服务器用途，FreeBSD 被定位为通用系统。

与 OpenBSD 一样，在上世纪 90 年代的互联网繁荣时期，FreeBSD 是驱动互联网基础设施的流行操作系统。当时，雅虎以广泛依赖 FreeBSD 而闻名，Netflix 的 Open Connect 内容交付网络处理了许多用户的连续观看会话。

## 文档质量

OpenBSD 和 FreeBSD 都维护有关其系统的文档。

![image](https://github.com/FreeBSD-Ask/Translated-articles/assets/10327999/5045b374-f63d-476d-838d-f212e6f7d8fc)

OpenBSD 的文档与其余系统一样，简洁而面向专家。从视觉上看，该项目的网站根植于上世纪 90 年代，尽管 OpenBSD 通常会有富有幽默感的发布主题。

除了 man 页之外，OpenBSD 还维护着 ["FAQ" 部分](https://redirect.viglink.com/?format=go&jsonp=vglnk_169555659403722&key=eac202ea7a96cf485281d6c4ffa2069e&libId=lmxdwmxz0103es17000ULaqhkd35z&loc=https%3A%2F%2Fwww.makeuseof.com%2Fopenbsd-vs-freebsd-what-is-the-difference%2F&ccpaConsent=1YNY&v=1&opt=true&optExText=false&out=https%3A%2F%2Fwww.openbsd.org%2Ffaq%2Findex.html&ref=https%3A%2F%2Fwww.google.com%2F&title=OpenBSD%20vs.%20FreeBSD%3A%20What%20Is%20the%20Difference%2C%20Which%20Is%20Best%3F&txt=%22FAQ%22%20sections)，实际上也可以作为手册使用。这些内容涵盖了一些细节，如安装和系统安全性。

![image](https://github.com/FreeBSD-Ask/Translated-articles/assets/10327999/7dcb3392-c258-46d9-8985-c714f1c94860)

FreeBSD 的文档方式更加精致。FreeBSD 有写得很好的 man 页，但该系统最出色的特点可能是“[Handbook](https://redirect.viglink.com/?format=go&jsonp=vglnk_169555660350123&key=eac202ea7a96cf485281d6c4ffa2069e&libId=lmxdwmxz0103es17000ULaqhkd35z&loc=https%3A%2F%2Fwww.makeuseof.com%2Fopenbsd-vs-freebsd-what-is-the-difference%2F&ccpaConsent=1YNY&v=1&opt=true&optExText=false&out=https%3A%2F%2Fdocs.freebsd.org%2Fen%2Fbooks%2Fhandbook%2F&ref=https%3A%2F%2Fwww.google.com%2F&title=OpenBSD%20vs.%20FreeBSD%3A%20What%20Is%20the%20Difference%2C%20Which%20Is%20Best%3F&txt=Handbook)”（手册）。它提供了足够详尽的信息，可作为专家用户的参考，同时也解释了足够基本的概念，适合那些对类 Unix 系统经验不太丰富的用户作为入门材料。

在文档质量方面，FreeBSD 胜过 OpenBSD。

## 防火墙实现

OpenBSD 和 FreeBSD 都强调安全性，它们提供了防火墙作为提高安全性的一种方式。

忠实于 OpenBSD 对安全性的强调，该项目开发了自己的防火墙程序，即 Packet Filter（PF）。与其他组件一样，PF 已经广泛移植到其他系统中。PF 是 macOS 的一部分，而 macOS 的一部分又是基于 FreeBSD 的。

与 OpenBSD 一样，FreeBSD 也将 PF 作为主要的防火墙程序之一，但还提供了 IPFW 和 IPFILTER。FreeBSD 手册为 PF 提供了最多的空间，但警告说它们的移植版本与 OpenBSD 的有显著差异。

在两个系统上配置防火墙以使其充当临时路由器需要一些时间和专业知识。对于那些决心构建 DIY 路由器的人来说，OpenBSD 因其对安全性的极端关注而具有优势。

## 桌面环境

尽管 OpenBSD 和 NetBSD 主要是面向服务器开发的，但也可以将它们用作桌面系统。

![image](https://github.com/FreeBSD-Ask/Translated-articles/assets/10327999/f5d28a60-db86-4693-99d3-5af922706e2a)

OpenBSD 可以安装 X 服务器和基本的 FVWM 窗口管理器环境【译者注：这里表述有点问题，OpenBSD 的文件集是有 X 的，不需要也不能通过包管理器安装】。从视觉上看，与其他内容一样，它也带有上世纪 90 年代的风格。你可以通过软件包管理器安装其他桌面环境。

![image](https://github.com/FreeBSD-Ask/Translated-articles/assets/10327999/95846a9f-0233-4c92-b86e-3d756743a80c)

FreeBSD 提供了许多与 Linux 发行版相同的窗口管理器和桌面环境。

在这两个系统上安装图形用户界面（GUI）需要一些较为复杂的步骤，类似于在 Arch 或 Gentoo 上安装 GUI。在这方面，FreeBSD 可能是赢家，因为你可以安装像 TrueOS【译者注：2020 年该系统就已经停止开发了】或 MidnightBSD 这样的完整桌面系统，它们已经默认包含了桌面环境。

## 硬件支持：OpenBSD 与 FreeBSD

如果你觉得在 Linux 上 [开源和专有硬件驱动程序的支持](https://www.makeuseof.com/closed-source-vs-open-source-hardware-drivers/) 令人沮丧，那么在 [基于 BSD 的操作系统上](https://www.makeuseof.com/best-bsd-based-operating-systems/)，你的选择会更有限，因为至少在桌面领域，它们比 Linux 更为小众。

与 Linux 一样，最大的挑战在于显卡和 Wi-Fi。

OpenBSD 支持 AMD 和 Intel 芯片组，但不支持 Radeon，因为该公司未向开发者提供任何技术信息。然而，有许多 Wi-Fi 驱动程序可用。

FreeBSD 支持主要的显卡芯片制造商以及 Wi-Fi。

与许多现代 Linux 发行版一样，X 在这两个系统上运行时需要很少或根本不需要配置。由于它们主要面向服务器，它们默认在控制台模式下运行。连接到 Wi-Fi 也较为复杂，但有线连接通常可以直接使用。

## FreeBSD 和 OpenBSD 中的包管理器

OpenBSD 和 FreeBSD 都提供了软件包管理，以简化软件安装，类似于现代的 Linux 发行版。在这两个系统中，你可以从源代码编译 "ports"，但也可以选择快速安装二进制软件包。后一种方法在这两个系统中越来越常见。

![image](https://github.com/FreeBSD-Ask/Translated-articles/assets/10327999/2cda586a-8b75-40b1-bbbc-4c6c5e9222f0)

OpenBSD 使用 `pkg_add` 和 `pkg_info` 程序来安装和搜索软件包。

![image](https://github.com/FreeBSD-Ask/Translated-articles/assets/10327999/7af6923b-163f-4d98-b9a9-3dd5a829a6f4)

FreeBSD 的软件包命令称为 "pkg"，并且所有操作都在一个程序中执行。后一种方法似乎更加简单。

## 哪个更安全？OpenBSD 还是 FreeBSD

OpenBSD 和 FreeBSD 都强调它们对安全性的承诺。FreeBSD 是一个更通用的系统，但显然可以看出，OpenBSD 在编码和系统设计方面非常专注。那些真正关心安全性的人可能会选择后者。

## 知名度

尽管仅仅因为流行度而选择操作系统不应该是唯一的考虑因素，但它将影响到寻找软件和支持的能力。虽然 OpenBSD 在开源社区中以其对安全性的忠诚以及在发布主题方面的俏皮一面而声名远扬，但 FreeBSD 似乎得到了更广泛的支持。

很难准确衡量这两个系统在实际中有多广泛的使用，但截至 2023 年 9 月，根据对其网页的点击量，FreeBSD 在 [distrowatch.com](https://redirect.viglink.com/?format=go&jsonp=vglnk_169555670553524&key=eac202ea7a96cf485281d6c4ffa2069e&libId=lmxdwmxz0103es17000ULaqhkd35z&loc=https%3A%2F%2Fwww.makeuseof.com%2Fopenbsd-vs-freebsd-what-is-the-difference%2F&ccpaConsent=1YNY&v=1&opt=true&optExText=false&out=https%3A%2F%2Fdistrowatch.com%2F&ref=https%3A%2F%2Fwww.google.com%2F&title=OpenBSD%20vs.%20FreeBSD%3A%20What%20Is%20the%20Difference%2C%20Which%20Is%20Best%3F&txt=distrowatch.com) 上的排名高于 OpenBSD。

## 现在，你可以根据工作的性质选择适合的 BSD

在 OpenBSD 和 FreeBSD 之间做出决策可能令人生畏，但决策很可能取决于你希望系统有多么安全。如果你需要一个安全、坚不可摧的操作系统，最好选择 OpenBSD。如果需要一个更通用的 BSD 系统，可以选择 FreeBSD 或 NetBSD。
