# OpenBSD 太棒了

- 原地址：<https://unixsheikh.com/articles/openbsd-is-fantastic.html>
- 译者：ykla & ChatGPT
- 最后发布日期：2020-11-20

我自 2004 年左右开始专业性的和个人化的使用 OpenBSD，这是一个免费的、多平台的基于 4.4BSD 的类 Unix 操作系统。今天我将分享一些我的经验。

在准备这篇文章时，我意识到很难充分赞扬 OpenBSD 的开发者。这是因为 OpenBSD 非常独特，我认为它相当令人惊叹。它的“辉煌”大部分隐藏在开发者的设计和特定的编码风格中，对于普通用户来说并不可见。你需要理解一些在幕后发生的事情，才能真正的欣赏 OpenBSD！

OpenBSD 安装简单快速，你会惊讶地发现这个系统是多么简单而且设计非常出色。项目从一开始就付出了大量工作，严格遵循了 Unix 哲学。

OpenBSD 在基本系统中附带了许多应用程序，但除了安全功能外，在默认情况下没有启用任何功能，你必须自行启用需要的服务。每个配置文件都遵循相同的语法风格，非常易读，因此非常容易理解和设置。每个参数在手册页中都有很好的文档，并且 OpenBSD 项目认为缺乏文档是“一个 Bug”。这是每个专业程序员都应该采用的做法。

缺乏文档或错误的文档对于具有安全漏洞的运行系统同样危险。原因在于，安全问题有时源于错误的配置。如果你不知道如何设置你的系统，你怎么能确定它是否以容易受到攻击者威胁的方式运行？互联网上的很多垃圾邮件源于配置不当的邮件服务器被攻击者利用而被入侵。

OpenBSD 操作系统内核和基本系统的每一行代码都会由程序员进行安全审计和检查，并且所有代码都遵循一套严格的指导方针和原则，试图消除所有传统的编码错误，因为许多安全漏洞实际上是由程序员的编码错误造成的。

但这还不是全部。让 OpenBSD 变得惊人的另一点是在开发过程中所做的所有安全保护机制，OpenBSD 的开发者在这个领域做了一些非常棒的先进工程！

安全保护机制是一些技术手段，帮助防止攻击者在操作系统上运行恶意代码或利用软件中的安全漏洞或弱点。

如果你正在使用一款软件，比如浏览器，而该浏览器有一个可以被利用的安全漏洞，那么攻击者可能能够访问你的计算机。攻击者在你的计算机上可以造成多大的破坏取决于操作系统的基本安全性。

OpenBSD 在内核和基本系统中内置了许多保护技术，使得攻击者的操作变得非常困难。这意味着攻击者很难通过常规的攻击技术（对于许多其他操作系统如 Microsoft Windows、Linux、Mac OS 等有效的技术）首先获取对你的系统的未经授权访问。这也意味着，即使攻击者尽管这些缓解技术而成功获取对你的系统的访问，他们能够造成的损害也是非常有限的。

这是一些内置在 OpenBSD 操作系统中并默认启用的安全创新功能列表：

- 在 i386/amd64/sparc64 架构中，内核强制执行 W^X。
- 自 6.0 版本起，用户空间强制执行 W^X。
- 默认启用 SROP（sigreturn(2) 导向编程）保护措施。
- 自重定位静态二进制文件的静态 PIE。
- 堆栈保护器。
- 大部分基本系统的特权降级和分离是一种政策，没有这些措施，新的功能不会被启用。
- 仅支持 bcrypt 密码散列，并且根据系统性能自动选择轮次值。
- 默认在基本系统、软件包和 Ports 上启用 PIE。
- 在引导时对 C 共享库进行重新排序，即在引导时重新链接 libc.so，使对象随机排序。
- 系统范围内的沙盒（pledge(2)）适用于大部分用户空间，包括 X 服务器的特权部分和大多数面向网络的守护程序。
- arc4random(3) 支持 rand(3)、random(3) 和 drand48(3)，并且有经过审核的基本/Ports。软件必须选择确定性破碎的 POSIX 行为。

这只是 OpenBSD 的一部分创新，更多的创新可以在 OpenBSD 的 [创新页面](https://www.openbsd.org/innovations.html) 中找到。其中一些创新得益于 OpenBSD 开发者的工作，已被其他操作系统采用和实现。

OpenBSD 是一个强大可靠的操作系统，在设置完成后你可以最小干预地运行它。实际上，它是唯一真正让你可以安心睡眠的操作系统，特别是在运行任何系统关键软件时。

OpenBSD 维护了许多基本系统的便携版本，包括：

- LibreSSL，一个从 OpenSSL 1.0.1g 分支中复刻出来的免费的 Secure Sockets Layer（SSL）和 Transport Layer Security（TLS）协议的实现。
- OpenBGPD，一个免费的 Border Gateway Protocol 4（BGP-4）实现。
- OpenOSPFD，一个免费的 Open Shortest Path First（OSPF）路由协议实现。
- OpenNTPD，一个简单的替代 ntp.org 的网络时间协议（NTP）守护进程。
- OpenSMTPD，一个免费的支持 IPv4/IPv6、PAM、Maildir 和虚拟域的 Simple Mail Transfer Protocol（SMTP）守护进程。
- httpd，一个在 5.6 版本中首次包含的 HTTP 服务器。
- OpenSSH，一个免费的 Secure Shell（SSH）协议实现。
- OpenIKED，一个免费的 Internet Key Exchange（IKEv2）协议实现。
- Common Address Redundancy Protocol（CARP），一个免费的替代 Cisco 专利的 HSRP/VRRP 服务器冗余协议。
- PF，一个带有 NAT、PAT、QoS 和流量标准化支持的 IPv4/IPv6 有状态防火墙。
- Unbound，一个 DNS 验证解析器。
- dhcpd，一个动态主机配置协议（DHCP）服务器。
- pfsync，一个用于 PF 防火墙的防火墙状态同步协议，使用 CARP 实现高可用性支持。
- spamd，一个带有灰名单支持的垃圾邮件过滤器，旨在与 PF 防火墙进行互操作。
- sndio，一个紧凑的音频和 MIDI 框架。
- Xenocara，一个定制的 X.Org 构建基础设施。
- cwm，一个堆叠式窗口管理器。
- tmux，一个虚拟控制台复用器。
- X.Org 服务器。
- Clang 编译器。
- GNU 编译器套件。
- Perl 编程语言。
- NSD DNS 服务器。
- Ncurses 终端处理库。
- GNU Binutils 工具集。
- GNU Debugger 调试器。
- Awk 文本处理工具。

所有这些都包含在操作系统的基本系统中，并且是 OpenBSD 标准安装的一部分。所有基本系统的部分都包含了 OpenBSD 特定的补丁、改进和增强措施，以提高安全性。

除了上述功能之外，截至撰写本文时，OpenBSD 还通过 OpenBSD [软件包管理器](https://www.openbsd.org/faq/faq15.html) 提供了超过 9700 个可安装的应用程序。然而，需要注意的是，尽管建议使用预编译的软件包而不是手动从 Ports 构建软件，但 OpenBSD 的“release”和“stable”分支的软件包不会得到更新。这意味着软件包的安全更新只能通过 Port 在“stable”分支运行时获得。

当在 Ports 中的应用程序中发现严重的错误或安全漏洞时，它们将在 Port 的“stable”分支中修复。与基本系统相反，“stable”Port 只会为最新发布提供安全后移。这意味着，如果你使用第三方应用程序，你需要检查正确的 Port 分支，并手动构建软件。可以使用 CVS 来保持 Port 的最新状态，并可以订阅 ports-changes 邮件列表以接收与 Port 中应用程序相关的安全公告。

另一个非常有效的解决方案是运行 OpenBSD 的“current”分支。 “current”分支是活跃开发发生的地方，但开发人员非常谨慎，不会引入可能会导致系统出现问题的新功能。 “current”分支有点类似于“滚动发布”模型。

由于 Ports 与第三方供应商提供的软件相关，因此它不会经过与 OpenBSD 基本系统执行的同样全面的安全审计。OpenBSD 项目没有足够的资源来确保与基本系统相同级别的 Port 健壮性和安全性。

欲了解更多信息，请访问 [OpenBSD 项目网站](https://www.openbsd.org/)。
