# 苹果的开源基石：macOS 和 iOS 背后的 BSD 传统

- 原文地址：[Apple’s Open Source Roots: The BSD Heritage Behind macOS and iOS](https://thenewstack.io/apples-open-source-roots-the-bsd-heritage-behind-macos-and-ios/)
- 原作者：[Jason Perlow](https://thenewstack.io/author/jason-perlow/)
- 发布时间：2024 年 7 月 8 日凌晨 5:00

认识到 BSD 在苹果成功中的重要性，有助于我们更好地理解开源贡献在塑造我们日常使用的技术中之价值。

想一想：苹果时尚且易用的设备，如强大的 MacBook 和无处不在的 iPhone，它们的可靠性和性能，在很大程度上得益于开源操作系统。那么，苹果究竟使用的是哪款开源操作系统呢？尽管常有人说，苹果的 macOS、iOS、iPadOS、watchOS、visionOS 和 tvOS 直接源自 FreeBSD，但其实这是一种误解。

这些操作系统的真正基石在于苹果较早期的操作系统技术与 NeXTSTEP 的结合，而 NeXTSTEP 本身是 Mach 和 BSD 的混合体——早于 FreeBSD。尽管多年来，FreeBSD 的一些用户空间元素被整合进上述操作系统，但苹果的内核（XNU）并不直接来源于 FreeBSD。然而，它们确实共享一脉共同的 BSD 传统。

![](https://cdn.thenewstack.io/media/2024/07/c2be97c9-openbsd.png)

[图片来源](https://cdn.thenewstack.io/media/2024/07/c2be97c9-openbsd.png)

## 理解 BSD 各版本

要了解苹果操作系统的发展，理解不同版本的 BSD（伯克利软件发行版）及其影响至关重要：

### 伯克利的原始 BSD

BSD 起源于 1970 年代末的加州大学伯克利分校。作为对 AT&T 原始 UNIX 操作系统的增强，BSD 引入了许多成为现代操作系统标准的创新。第一个版本 1BSD 发布于 1977 年，随后是 1978 年的 2BSD。重要版本包括 4.1BSD（1981 年）和 4.2BSD（1983 年），它们引入了快速文件系统（FFS）、TCP/IP 网络和套接字 API，这些基础要素至今仍在使用。

### FreeBSD

FreeBSD 是从原始 BSD 衍生而来的，以其出色的性能、先进的网络功能和广泛的硬件支持而著称。它源自 1993 年的 386BSD 项目，至今广泛应用于服务器、桌面和嵌入式系统。FreeBSD 强大的性能和可靠性使其成为高可用性应用和互联网基础设施的首选。

### NetBSD

NetBSD 以其在多种硬件平台上的可移植性而闻名。其座右铭“Of course it runs NetBSD”（它当然可以运行 NetBSD）体现了它在各种硬件架构上运行的能力，从服务器和桌面到嵌入式设备和大型机。自 1993 年成立以来，NetBSD 一直是 BSD 系列中可移植性和简洁设计的典范。

### OpenBSD

OpenBSD 是于 1995 年，从 NetBSD 复刻出来的，重点关注安全性、正确性和代码简洁性。它以严格的安全实践而闻名，并且是许多安全技术的先驱，如 OpenSSH、PF（包过滤器）和安全内存管理技术。OpenBSD 对安全性的承诺使其成为那些安全至关重要的应用程序的首选。

### 其他衍生 BSD

出现了几款衍生的 BSD，每款变种都有其独特的重点和增强功能。例如，DragonFly BSD 注重性能和可扩展性，而 Darwin 则是苹果 macOS 和 iOS 的内核。

## Darwin 和 XNU：苹果操作系统的内核

macOS 的核心是 XNU 内核，这是一款混合内核，结合了 Mach 微内核、BSD 组件和 I/O Kit（设备驱动的面向对象 API）。这种集成确保了 macOS 受益于 BSD 类 Unix 的强大稳定性，同时利用了 Mach 微内核的灵活性。

Darwin 是苹果操作系统（macOS、iOS、watchOS、tvOS 和 iPadOS）的开源基石，它包括了 XNU 内核、各种 BSD 组件和其他开源项目。Darwin 的起源可以追溯到 NeXT，这是一家由史蒂夫·乔布斯在 1985 年创办的公司，他离开苹果后创办了这家公司。NeXT 开发了 NeXTSTEP，这是一种基于 Mach 微内核和 BSD 的操作系统。它发展成为 OpenStep，并最终在史蒂夫·乔布斯回归苹果时演变成 Darwin，带回了 NeXT 的技术。

2000 年，苹果发布了 Mac OS X 的一些核心组件（现在的 macOS）作为开源项目，采用 Apple 公共源代码许可证（APSL），允许更广泛的社区受益并为其开发做出贡献。这些组件包括 launchd、Grand Central Dispatch 和 Core Foundation，其中一些后来在更宽松的 Apache 许可证下重新授权，以促进更广泛的应用。然而，高级组件如 Cocoa 和 Carbon 框架仍然是专有的，以保持苹果的竞争优势。GPL 和 BSD 许可的组件未重新授权，保持了其原有的[开源许可证](https://thenewstack.io/how-do-open-source-licenses-work-the-ultimate-guide/ "open source licenses")。

### 详细时间线

- **1969 年**：AT&T 贝尔实验室开发了 UNIX 操作系统。
- **1977 年**：加州大学伯克利分校发布了 BSD 的首个版本（1BSD），作为对 AT&T 原始 UNIX 操作系统的增强。
- **1978 年**：发布了 2BSD，继续在 1BSD 的基础上进行改进和工具开发。
- **1980 年**：发布了 3BSD，加入了更先进的特性和改进。
- **1983 年**：发布了 4.2BSD，包含了像快速文件系统（FFS）和 TCP/IP 网络这样的重大创新，这些成了未来操作系统的基础元素。
- **1985 年**：史蒂夫·乔布斯在离开苹果后创办了 NeXT 公司。
- **1989 年**：发布了 NeXTSTEP，它基于 Mach 微内核和 BSD，结合了先进的功能和面向对象设计。
- **1993 年**：FreeBSD 和 NetBSD 从 386BSD 衍生出来，分别专注于性能、安全性和可移植性。
- **1995 年**：OpenBSD 从 NetBSD 分支出来，强调安全性和代码正确性。
- **1996 年**：苹果收购 NeXT，带回了 NeXTSTEP 技术，为 Mac OS X 奠定基础。
- **2000 年**：苹果发布了基于 Darwin OS 的 Mac OS X 第一版。这个操作系统结合了 Mach 微内核和来自 BSD 的组件，成为苹果现代操作系统的核心。
- **2006 年**：OpenDarwin 停止发布，标志着苹果不再提供可独立安装的 Darwin OS 版本。

## BSD 组件在 macOS 中的演变与整合

| **组件**         | **来源**                                             |
| :----------------: | :---------------------------------------------------- |
| **XNU 内核**     | **Mach/NeXTStep/OpenStep**                           |
| **网络栈**       | **FreeBSD/BSD，以及额外的 NIKE 和 IOKIT**            |
| **虚拟文件系统** | **FreeBSD/BSD**                                      |
| **用户空间工具** | **FreeBSD/BSD**                                      |
| **内存管理**     | **Mach/NetBSD**                                      |
| **进程模型**     | **Mach IPC、Mach Security Trailers 和强制访问控制（MAC）机制** |

苹果的 macOS 是一款混合化的操作系统，集成了来自不同 BSD 变种的各种组件，形成了一个强大且多功能的平台。macOS 的内核是 XNU，它是一款混合内核，结合了来自 Mach、NeXTSTEP 和 OpenStep 的元素，并且加入了来自 BSD 的其他组件。这一基础架构利用了每个系统的优势，提供了一款可靠且高性能的操作系统。

**网络栈**：macOS 的网络栈源自 FreeBSD 和其他 BSD 变种，集成了它们可靠和高效的网络功能。早期的网络元素，如 TCP/IP 栈，受 FreeBSD 设计的影响，因其性能和可靠性而闻名。包括最初为 FreeBSD 开发的 kqueue 事件通知接口，也增强了 macOS 处理 I/O 事件的能力。苹果的实现结合了来自 BSD 和 FreeBSD 的代码，但也包含了像网络内核扩展（NKE）、面向对象的设备驱动系统（IOKit）以及磁盘仲裁层等独特机制。这些组件与传统 BSD 实现有所不同。

**虚拟文件系统**：macOS 的虚拟文件系统组件来自 FreeBSD 和其他 BSD 系统，确保了稳定和安全的文件管理系统。

**内存管理**：macOS 的内存管理系统主要来源于 Mach，并受 NetBSD 的影响，特别是在共享/合并缓冲区缓存方面。

**进程模型**：macOS 的进程模型基于 Mach，使用 Mach 系统线程作为进程子系统的基础层。

**Mach IPC 和安全性**：Mach IPC 广泛应用于 macOS 和 iOS 的内核及用户空间，引入了进一步的差异。例如，Mach Security Trailers 是受信任的 IPC 的基础部分。应用程序沙箱中使用的强制访问控制（MAC）机制也与 \*BSD 系统中的机制大相径庭。

**随着时间的推移发生的分歧**：由于 FreeBSD 不愿意整合 UNIX03 兼容性更改，导致了分歧，尽管有一些混合的努力，像 libc 和 libm 这样的库还是出现了复刻。此外，像 NeXTBSD 项目这样的倡议，旨在将像 launchd、Mach IPC 和 Grand Central Dispatch 等苹果的技术带到 FreeBSD，但这些努力未能得到普遍支持。结果，FreeBSD 和 NeXTBSD 的代码库继续出现分歧，尤其是在苹果推进定制芯片支持和优化时，进一步拉大了它们之间的差距。

**已知的集成和贡献**：macOS 在很大程度上受益于 FreeBSD，尤其是在网络概念上。尽管 macOS 并未完全采用 FreeBSD 的网络栈，但它集成了几款 FreeBSD 衍生的组件，经过调整以满足其独特的需求。macOS 中的早期网络元素，如 TCP/IP 栈，受 FreeBSD 设计的影响，以其性能和可靠性而著称。此外，像最初为 FreeBSD 开发的 kqueue 事件通知接口也被集成到 macOS 中，增强了它处理 I/O 事件的能力。

## 对苹果产品的影响

苹果对 BSD 代码的采用遍及其整个产品线。macOS 驱动着 Mac 台式机和笔记本，iOS 运行在 iPhone 上，iPadOS 运行在 iPad 上，watchOS 运行在 Apple Watch 上，visionOS 运行在 AR/VR 设备上，tvOS 运行在 Apple TV 上。每款操作系统都包含了 BSD 组件，突显了 BSD 在苹果生态系统中的广泛影响。

BSD 的稳健性、安全性和性能特点在塑造苹果操作系统的稳定性和效率方面发挥了重要作用。集成 BSD 衍生的网络概念和组件，使得 macOS 能够提供高性能的网络功能，使其成为消费者和企业应用的可靠选择。BSD 的先进内存管理和进程调度也对 macOS 的响应能力和多任务处理能力作出了贡献。

通过将 BSD 的强大特性与其专有技术相结合，苹果创造了一系列稳定、高效、创新且功能强大的操作系统。这种共生关系凸显了开源贡献在苹果软件生态系统持续演变中的重要性。

## 苹果对开源的贡献与利用

苹果公司在高度依赖 BSD 的同时，也积极参与着开源社区。公司通过其[开源网站](https://opensource.apple.com/)和 [GitHub](https://github.com/apple-oss-distributions) 发布了多项 Darwin OS 组件，包括 XNU 内核、各种用户空间工具和库。这些贡献确保了开源社区能够从苹果的创新和改进中受益。

苹果会定期更新其开源项目，发布 Darwin OS 组件的新版本。这些更新通常与新的 macOS 和 iOS 版本发布同步，体现了苹果对开源社区的持续承诺。通过共享其改进和增强功能，苹果推动了[开源软件开发](https://thenewstack.io/open-source/ "open source software development")，为开发者和用户带来益处。

## Darwin OS 的现状

Darwin OS 是苹果操作系统的开源核心，目前已不再作为完整的、可安装的系统发布。苹果将 Darwin 的各个组件分开发布，例如 XNU 内核仓库在 GitHub 上，用户空间工具则发布在苹果的开源网站上。这种分散的发布策略使得从苹果的官方渠道组装完整的 Darwin OS 变得困难。

通过 OpenDarwin 项目，苹果曾使这些组件能够单独安装并作为独立系统存在。然而，由于开发的兴趣有限和分歧逐渐加大，OpenDarwin 于 2006 年停止发布。苹果当时的目标值得称赞，但随着复刻过多，最终无法维持该项目，导致了当前的碎片化分发方式。

尽管从苹果处获得完整的 Darwin OS 存在挑战，开源社区仍尝试将这些组件整合为可安装的操作系统，取得了一定的成功。像 [PureDarwin](http://www.puredarwin.org/)（以及之前的 OpenDarwin）这样的社区驱动项目，旨在通过整合苹果发布的各种开源组件，提供可用的 Darwin OS 版本。这些项目展现了开源开发的合作精神，并证明了 Darwin OS 作为独立操作系统的潜力。

## 苹果与 BSD 的现有关系

苹果与 BSD 代码的关系仍然相当神秘。尽管苹果在开源方面作出了重大贡献，但该公司在操作系统中 BSD 代码的具体使用程度仍然较为模糊。这种封闭的开发过程使得很难准确指出 macOS、iOS 及其他苹果系统中有多少部分仍依赖于 BSD 基石。[BSD 许可证](https://opensource.org/license/bsd-3-clause)的宽松性允许苹果使用并修改代码，而无需公开其使用方式和位置，这与 [GPLv2](https://www.gnu.org/licenses/old-licenses/gpl-2.0.en.html) 的“传染性”要求有所不同，后者要求修改后的代码必须共享。

苹果的开源软件策略随着时间的推移发生了变化。公司将焦点从 CoreOS 的开源组件转向了其他领域，如编程语言和编译器技术。苹果投入大量时间和精力开发项目如 [Swift](https://swift.org/)、[clang](https://clang.llvm.org/) 编译器和 [LLVM](https://llvm.org/) 运行时。这些项目代表了苹果对开源社区的主要贡献，并展示了苹果与 BSD 之间的技术转移，尤其是在编译工具链方面。

此外，随着苹果对原始 BSD 组件的大量修改，传统 BSD 代码与现代改编之间的界限变得模糊。苹果在不断修改和扩展这些组件后，已经很难区分哪些部分是直接继承自 BSD 的，哪些是苹果的创新。这种不透明性限制了我们对 BSD 在苹果产品中全貌的理解，也突出了 BSD 的宽松许可证与 GPL 的传染性要求之间的差异。

有证据表明，苹果仍在其操作系统中使用着当前的 BSD 组件。最近，[2024 年 6 月的提交](https://cgit.freebsd.org/src/commit/?id=7dd39ef4e0d56b213445754a189d204b70a77a00)由 Klara 公司提交，该公司代表多个客户（如 NetApp）向 FreeBSD 项目提交代码，这表明 FreeBSD 代码的持续集成和使用，证明了尽管苹果的开发过程封闭，BSD 代码（尤其是 FreeBSD）仍然是其操作系统的重要组成部分。

## 市场影响与采用

苹果在其产品中使用 BSD 对消费电子行业和创意内容行业产生了重大影响。BSD 的鲁棒性、安全性和性能使得苹果设备对消费者和专业人士极具吸引力。电影、音乐制作、图形设计等创意领域的行业尤其青睐苹果产品，因为它们可靠且功能强大。

BSD 的先进内存管理、进程调度和高性能网络能力确保了苹果的桌面和笔记本电脑为高要求的应用提供所需的性能。这使得 macOS 成为视频编辑、音乐制作和图形设计的优秀选择，在这些领域，稳定性和性能至关重要。

## 其他行业利用 BSD

虽然苹果集成 BSD 对其成功起到了关键作用，但 BSD 的特性也使其成为其他各行各业供应商的首选。例如，NetApp、Netflix、Juniper（均为 FreeBSD 用户）等公司，利用 BSD 的先进功能应用于不同的领域：

- **安全性：** BSD 强大的网络功能和安全措施使其在安全通信和数据保护领域尤为适用。
- **互联网流量管理：** BSD 可靠的 TCP/IP 栈确保了高数据量的有效处理，使其在互联网流量管理中大有作为。
- **嵌入式系统：** BSD 的先进内存管理和进程调度提高了嵌入式系统的效率和可靠性。
- **存储解决方案：** NetApp 等公司利用 BSD 提供强大而高效的存储解决方案。

通过认识到 BSD 的广泛应用，我们可以更好地理解 BSD 对苹果产品以及广泛行业的影响，彰显了这一开源操作系统的多样性和稳健性。

## 产品数量与市场覆盖

苹果发布了多款运行 macOS、iOS、iPadOS、watchOS、tvOS 和 visionOS 的产品。因此，全球许多设备中都包含 BSD 代码。这些设备的出货量和运行数量，包括嵌入式设备、桌面/笔记本电脑，甚至苹果公司数据中心内的服务器计算机，可能与运行 Linux 的设备和系统数量相当，甚至更多。这强有力地证明了 BSD 在苹果生态系统中的广泛采用与影响。

苹果的设备遍布各行各业，满足不同消费者群体的需求。这种广泛的市场覆盖展示了 BSD 对苹果操作系统成功和可靠性的贡献。来自 BSD 的无缝用户体验、强大性能和安全性使得苹果产品成为全球数百万用户的首选。

## 展望

苹果在开发其操作系统（包括 macOS、iOS、iPadOS、watchOS、visionOS 和 tvOS）时，极度依赖 BSD 代码。通过集成 BSD 组件，苹果为创新、稳定性和性能奠定了坚实的基础。尽管苹果系统中 BSD 代码的具体使用程度不透明，但显然，BSD 在苹果产品的设计和功能中起到了重要作用。

展望未来，苹果可能会继续利用 BSD 的优势，并为开源社区作出贡献。BSD 和苹果操作系统的不断演进，暗示了开源合作在技术进步中的重要性。认识到 BSD 在苹果成功中的重要性，有助于我们更好地理解开源贡献在塑造我们日常使用技术中之价值。

参考文献与进一步阅读：

- [Apple Open Source](https://opensource.apple.com/)
- [Apple GitHub](https://github.com/apple-oss-distributions)
- [FreeBSD Project](https://www.freebsd.org/)
- [PureDarwin Project](http://www.puredarwin.org/)

---

**Jason Perlow** 是一位技术专家，拥有超过二十年的经验，曾在财富 500 强公司中整合大型异构多供应商计算环境。他曾担任 Linux 基金会的前编辑总监。目前，他在佛罗里达州科勒尔斯普林斯经营着自己的技术媒体咨询公司——Argonaut Media Communications LLC。
