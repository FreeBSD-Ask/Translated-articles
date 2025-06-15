# 2023 年 7 月软件开发项目进展报道

- 原地址：<https://freebsdfoundation.org/blog/july-2023-software-development-projects-update/>
- 发布日期：2023 年 8 月 16 日
- 译者：ykla & ChatGPT

衡量进展的方式有很多种，比如提交的速度、开发者的数量、FreeBSD Port 的数量等等。根据基金会资助的承包商数量这一衡量标准，我们正处于一个繁荣时期。截至撰写本文时，FreeBSD 基金会已经为 12 个不同的项目开展了合同。其中一些项目我们在过去的文章中有过介绍，另一些则是最近才开始的。接下来是这些承包工作的摘要，以及基金会员工的一些值得注意的进展。

**承包的工作**

## 文档和测试实习

从 7 月开始，Yan-Hao Wang 开始在基金会进行夏季实习，从事各种任务。首先，Yan-Hao 将专注于改进我们的文档工具。其中一个可交付成果是构建了一个在线编辑器，让创建和编辑 man 页面变得尽可能简单。第二个是开发一个针对 FreeBSD man 页面和其他文档的“专家系统”。这个最佳努力和概念验证的交付成果将涉及将 FreeBSD 文档（如 man 页面和手册）导入向量数据库，以便诸如 ChatGPT 之类的大型语言模型在查询 FreeBSD 问题时可以“阅读”它们，从而提供更好的答案。改进我们的测试框架是夏季工作的第二个总体目标。例如，Yan-Hao 将为 `/bin`、`/sbin`、`/usr/bin` 和 `/usr/sbin` 以及 libxo 下的许多用户空间工具添加缺失的测试用例。最后，一些杂项的可交付成果包括更新 FreeBSD Jeknins tinderbox，并为树莓派 4 支持制定开发路线图。

## 解决 Port OpenSSL 3 / LLVM 16 的问题

发布 FreeBSD 14.0 的一个主要障碍，也是原定发布计划的延迟原因，是基本系统中的 OpenSSL 版本问题。由于计划于 2023 年 9 月终止对 OpenSSL 1.1.1 的支持，有必要在发布之前升级到 OpenSSL 3。基金会团队成员 Pierre Pronchery 与 FreeBSD 开发社区的其他成员基本上完成了更新，然而基本系统和 Ports 中依赖于 OpenSSL 的许多代码需要进行修复，以使其适用于这个新的主要版本。在更新基本系统中的 OpenSSL 的同时，LLVM 也被更新到了版本 16，这也引入了一些破坏性的变化，特别是在 Ports 中。大部分与 OpenSSL 3 和 LLVM 15 相关的关键问题已经得到解决，但是在 LLVM 16 中，大约有 800 个额外的 Port 无法构建，导致额外的 2800 个依赖的 Port 在完整的 Ports 构建中被跳过。Muhammad Moinur（Moin）Rahman 将完成这一耗时且繁琐的工作，修复与更新 OpenSSL 3 和 LLVM 16 相关的所有 Port 问题。

## 基于 AMD64 架构的 SIMD 增强型 libc 库

现代计算机架构提供了 SIMD（单指令多数据）指令集扩展，可以同时处理多个数据。诸如 SSE、AVX 和 NEON 等 SIMD 指令集扩展在现代计算机上无处不在，并且为许多应用程序提供了性能优势。该项目的目标是为常见的 libc 函数提供 SIMD 增强版本，主要是那些在 string(3) 中描述的函数，以加速大多数 C 程序的执行。

SIMD 技术通常用于数值应用，如视频编解码器、图形渲染和科学计算，同时在基本数据处理任务中也有助于 libc 函数实现的任务。虽然其他 libc 实现已经为标准 libc 函数提供了 SIMD 增强版本，但 FreeBSD libc 在很大程度上没有提供。Robert Clausecker 的这个项目的目标是提供这些相关 libc 库函数的 SIMD 增强版本，从而提高与之链接的软件性能。由于这些 libc 函数被大多数适用于 FreeBSD 的软件使用，这些增强预计将广泛受益于各种程序。该项目的主要重点是 amd64 架构，旨在根据 x86_64 psABI 定义的体系结构级别生成 SIMD 优化的实现。

针对每个被优化的函数，将提供多达四个实现：

- 针对 amd64 进行优化的标量实现，但不使用任何 SIMD。
- 基准实现，使用 SSE 和 SSE2，或者使用 x86-64-v2 实现，使用 SSE 扩展，包括 SSE4.2。
- 使用 AVX 和 AVX2 的 x86-64-v3 实现。
- 使用 AVX-512F/BW/CD/DQ 的 x86-64-v4 实现。

用户可以通过设置环境变量 `AMD64_ARCHLEVEL` 来选择要使用的 SIMD 增强级别。虽然当前项目仅涉及 amd64 架构，但未来可能会扩展到其他架构，如 arm64。

## 网络暑期实习

Naman Sood 是 FreeBSD 基金会的暑期实习生，他一直在从事与网络相关的任务，如：

- 完成 Luiz Amaral 开始的工作，允许 pfsync 流量通过 IPv6 传输。
- 更新工作，以实现 pf full cone NAT 的 RFC 4787 要求 1 和 3。

Naman 还修复了 pw(8) 和 du(1) 中的错误，并已开始探索由 Klaus P. Ohrhallinger 开始的 tcp 检查点和故障转移工作的完成。

## 使用 ktrace(1) 进行安全沙箱化

从 6 月开始，Jake Freeland 开始在基金会实习，从事与 Capsicum 相关的项目。Capsicum 是用于限制应用程序和库所赋予的功能的工具。其核心思想很直观；进入能力模式后，资源获取和外部通信将受到限制。围绕这个原则设计程序相对容易，但是当不是设计为沙箱化的应用程序需要在此环境中运行时就会出现问题。很难确定哪些操作会引起 Capsicum 违规，并且无法预先打开尚未被请求或命名的资源。此外，开发人员在实施 Capsicum 功能之前需要对程序有深入了解。由于这些挑战，近年来围绕该框架的进展和开发逐渐减少。

这个实习将涉及一系列项目，总体目标是复兴 Capsicum。主要目标是增强和简化希望将其现有程序转换为 Capsicum 模式的开发人员的体验。Capsicum 的最大敌人是其陡峭的学习曲线。将程序重构为支持能力模式通常要求开发人员知道什么会引起 Capsicum 违规，并知道如何重新组织给定程序以避免违规。有时这个过程很简单，但是更大的程序通常需要按需使用资源，找出如何满足这些需求可能会很困难。为开发人员提供更多便捷的程序 Capsicum 化工具将平缓学习曲线。如果 Capsicum 化变得容易，那么更多的开发人员将采用它。

在进行 Capsicum 化的第一个逻辑步骤是确定你的程序何时引发能力违规。你可以通过查看源代码并删除与 Capsicum 不兼容的代码来解决此问题，但这可能很繁琐，并且需要开发人员熟悉在能力模式中不允许的所有内容。手动查找违规的替代方法是使用 ktrace(1)，它可以记录指定进程的内核活动。虽然 ktrace 可以记录并返回有关程序违规的额外信息，但这可能不方便，因为 ktrace 在程序终止之前只报告第一个能力违规。

由 Jake 编写的新 ktrace 扩展可以在程序不处于能力模式时记录违规。这意味着能力违规跟踪可以在未经修改的程序上运行，而且由于不需要在能力模式下运行程序，即使在发生能力违规时，它仍将获取资源并正常执行。

完成 ktrace 扩展后，Jake 将开始对各种实用程序进行 Capsicum 化，包括 syslogd(8)、NFS 守护程序、ggatec(8)/ggated(8)、tftpd(8)、ntpd(8) 和 libarchive(8)。

## 无线实习

在 2022 年谷歌代码之夏的贡献者 En-Wei Wu，于 2023 年初开始在 FreeBSD 基金会实习，从事 FreeBSD 无线驱动程序和工具方面的工作。这项工作分为三个组成部分。

- 将通过扩展 wtap，增加对目前仅支持的 802.11b 之外的更多 802.11 物理层的支持。wtap 的其他工作将包括添加 WPA/WPA2/WPA3 支持，以便可以测试 wpa supplicant(8) 和 hostapd(8)。
- 将在 hostapd(8) 中添加对 WPA2 预身份验证的支持。WPA2 是 IEEE 802.11i 规范的一部分定义的身份验证协议。现在，这个协议常用于对无线站点进行认证以访问接入点。该协议的一部分是能够在一个或多个接入点上对站点进行预认证，以便快速进行漫游。FreeBSD 在用于构建启用 WPA 的接入点的 hostapd 程序中缺乏对该协议的支持。此任务将移植现有的 Linux 代码，以在 hostapd 中支持预身份验证。这主要涉及重写一些用户模式的多播代码并测试结果。对于 FreeBSD 外部托管的第三方源代码的修改应在适用时上游到相应的项目。
- 将完成对 802.11 驱动程序的工作。通过完成 Adrian Chadd 在驱动程序上开始的工作，将 port ath10k 驱动程序。还将通过协助 Bjoern Zeeb 开发和测试诸如 rtw88 和 rtw89 之类的 Realtek 驱动程序来提供帮助。

## 增强持续集成

FreeBSD 基于 Jenkins 的持续集成基础设施当前在开发人员将提交推送到 FreeBSD 源代码库时启动作业。虽然这对于识别新引入的问题很有价值，但更好的方法是在将其推送到主分支之前轻松地识别这些问题。可以通过在预提交环境中使 CI 测试对开发人员更可用，并且通过拥有私有的 FreeBSD 运行程序，流行的 git 托管服务可以利用这些程序为推送到私有分支的人们创建 CI 基础设施来实现这一目标。为此，Muhammad Moinur Rahman 的目标是采取由 Li-Wen Hsu 编写的 CI 脚本，并将其作为构建系统的一部分提供给开发人员。类似于 make universe 或 make tinderbox 可以构建所有支持的体系结构，make ci 将为所有支持的构建实现类似的功能。另一个目标是使开发人员在调试问题时能够运行单个 CI 作业。希望这种灵活性还将使其他人能够将这些构建/脚本集成到其他 CI 工具中，如从 Github 运行的 Cirrus CI 等。

## 改进 kinst DTrace provider

DTrace 是一个框架，使管理员和内核开发人员能够实时观察内核行为。DTrace 具有称为“provider”的内核模块，它使用“探针”在内核中执行特定的仪器操作。kinst 是由 Christos Margiolis 和 Mark Johnston 于 GSOC 2022 的一部分创建的新 DTrace 提供程序，它允许进行指令级跟踪，即对于给定的内核函数，用户可以跟踪其中的每个指令。该提供程序在 FreeBSD 14.0 中可用。

kinst 探针采用 `kinst::<function>:<offset>` 的形式，其中 `<function>` 是要跟踪的内核函数，`<offset>` 是特定的指令。可以使用 kgdb(1) 从函数的反汇编中获取这些偏移量。例如，以下命令将在每次执行 vm_fault() 时跟踪其第一条指令：［ 注：原文如此 ］

这个项目于 7 月下旬完成，实现了内联函数跟踪。为了实现这个功能，使用了 DWARF 调试标准，以便能够检测内联调用并相应地处理它们。利用了 DWARF 和 kinst 的功能，解决了 FBT 的一些缺点，例如尾部调用优化问题（DTrace 手册的第 20.4 章）和内联跟踪能力的缺失。

项目交付成果包括：

- 添加 kinst 的入口和返回探针，类似于内联跟踪所需的 FBT
- 扩展 kinst，通过使用 FreeBSD 的 dwarf(3)，能够跟踪内联调用
- 添加一个“locals”结构，用于存储跟踪函数的局部变量。例如，对于 kinst:: foo: <x>，我们可以在 D 脚本中使用 print(locals-> bar) 来打印局部变量 bar
- 添加一个新的 dtrace(1) 参数，在 dt_sugar 应用转换后转储 D 程序。这对于调试 dt_sugar 本身非常有用。
- 将 kinst 移植到 riscv / arm64。

## FreeBSD 作为一级 cloud-init 平台

cloud-init 是在云中配置服务器的标准方式。不幸的是，除了 Linux 以外的操作系统对 cloud-init 的支持相当有限，而且缺乏对 FreeBSD 的 cloud-init 支持阻碍了将 FreeBSD 作为一级平台提供的云提供商这一愿望。为了解决这个问题，FreeBSD 基金会与 Mina Galić 签订合同，使 FreeBSD 的 cloud-init 支持与 Linux 支持保持一致。项目交付成果包括完成对特定网络类的提取，实现 ifconfig(8) 和 login.conf(5) 解析器，实现 IPv6 配置，为 Azure 创建 devd 规则，并编写有关将 FreeBSD 投入生产的手册文档。

在上个季度完成了一些项目里程碑：

- 临时网络类已被重写并变得与平台无关。

这些类被多个云提供商用于在检索实际配置之前初始化临时网络。

- cloud-init 在 Vultr 上成功测试通过。

希望在下一个 cloud-init 发布时，可以说服 Vultr 将其 FreeBSD 镜像切换到 cloud-init。

- 扩展了 BSD 的 rsyslog 支持。
- 添加了一个用于 cloud-init 的 ds-identify 的 rc 脚本，这应该使零配置设置的速度提高数个数量级。

ds-identify 首先运行，非常快速地猜测机器正在运行的云提供商。然后，cloud-init 仅使用该猜测，而不是通过完整的可能的云提供商列表进行迭代和失败。构建自定义镜像的人可以轻松地禁用它（通过删除“/usr/local/etc/rc.d/dsidentify”），并自己提供特定的列表，从引导时间中减少几毫秒。

接下来的步骤将是继续进行网络重构任务，并为 FreeBSD 添加 LXD 支持，以便将其包含在 CI 测试中。后者将涉及对 [LXD](https://github.com/canonical/lxd/pull/11761) 的工作，以及对 [FreeBSD virtio](https://bugs.freebsd.org/bugzilla/show_bug.cgi?id=271793) 子系统的工作。

## 在 FreeBSD 上的 OpenStack

OpenStack 是一个开源的云操作系统，可以用于多种资源，包括虚拟机、容器和裸金属服务器。OpenStack 的控制平面主要针对 Linux，而 FreeBSD 只被非官方地支持作为客户操作系统。用户可以在开放的云平台上创建 FreeBSD 实例，但目前管理员或运营商无法在 FreeBSD 主机上设置运行 OpenStack 部署。鉴于基于云的部署的日益重要性，以及 OpenStack 在各种云提供商中的流行度，FreeBSD 基金会已经与 Chin-Hsin Chang 签订合同，将 OpenStack 组件移植到 FreeBSD 主机上，使整个系统可以在 FreeBSD 主机上运行。

要移植到 FreeBSD/amd64 的基于 Linux 的 OpenStack 组件包括但不限于：

- Keystone（身份和服务目录）
- Ironic（裸金属供应）
- Nova（实例生命周期管理）
- Neutron（覆盖网络管理）。

截至 2023 年第二季度，一些重要的成就包括：

- 解决实例内部的网络连接问题
- 添加了同时生成多个实例的功能
- 从 Python 3.8 移植到 3.9
- 完成了 Keystone 的移植
- nova-compute 现在为每个创建的实例（虚拟机）生成不同的控制台

已经开始移植 nova-novncproxy 和 nova-serialproxy，这将增加访问实例控制台的方式。为了降低那些想要测试工作的人的门槛，开发环境也已从物理实例迁移到虚拟实例。有兴趣进行测试的任何人都应该知道，在 Linux KVM 之上运行 bhyve 虚拟机仍然存在问题。有关该问题的详细说明可以在 <https://hackmd.io/@starbops/SkdJON2un> 找到。未来计划的其他工作包括改进控制台代理服务，使整体工作流更加流畅。构建概念验证站点的逐步文档可以在 <https://github.com/openstack-on-freebsd/docs> 找到。每个 OpenStack 组件的修补版本位于同一 GitHub 组织下。

## 使用日志软更新进行文件系统快照

UFS/FFS 文件系统具有进行快照的能力。由于快照能力是在写入软更新之后添加的，因此它们已与软更新完全集成。然而，在 2010 年添加了日志软更新之后，它们从未与快照集成。因此，在运行日志软更新的文件系统上无法使用快照。仍然需要 UFS 快照的两个情况是：它们允许对活动文件系统进行可靠的转储，从而避免可能的停机时间；它们允许运行后台 fsck。与 ZFS 中需要进行校验类似，需要定期运行 fsck 以查找未检测到的磁盘故障。快照允许在活动文件系统上运行 fsck，而不需要安排停机时间。这项工作需要对 UFS/FFS 软更新和快照内核代码以及 fsck_ffs 实用程序进行广泛的更改。

该工作分为两个里程碑。里程碑 1 于 2022 年底完成，可以在运行日志软更新的情况下进行快照。这些快照可用于对活动文件系统进行后台转储。里程碑 2 涉及将 fsck_ffs 扩展为能够使用在运行日志软更新的文件系统上的快照进行后台检查。截至 8 月中旬，内核和 fsck_ffs 所需的更改已经完成，并且测试进展顺利。

与此项目相关，Kirk 还解决了一些最近报告的文件系统 Panic 问题以及一些较旧的 UFS 错误。其中包括可能触发 UFS 超级块完整性检查中的虚假损坏警告的错误。所有修复都在 FreeBSD 14 中进行了测试，并与 FreeBSD 13 相关的内容也被合并到主线。

## WIFI

FreeBSD WiFi 栈需要持续维护和开发，以跟上新标准和设备的发展。基金会资助 Bjoern Zeeb 将支持当前一代英特尔 WiFi 设备的功能集成到 Linux 内核的双重许可的上游驱动程序中。在合同下，Bjoern 还将承担相关的无线工作，例如开发 802.11 LinuxKPI 和集成其他无线驱动程序，比如来自 Realtek 的驱动程序。

在最新的季度中，Bjoern 几乎所有的时间都用于为 iwlwifi(4) 更新做准备的 LinuxKPI 更新，以及对其他驱动程序（如 rtw88、rtw89、mt76 和 ath10k）的更新，以及对 ath11k 和 ath12k 的准备。这项工作的大部分已经在代码库中，并将很快与构建连接，供人们测试。

## 其他基金会工作

正如我们在 2023 年第二季度状态报告中所报道的，339 个 src，155 个 ports 和 20 个 doc tree 的提交将 FreeBSD 基金会标识为赞助商。其中一部分工作包括：

- 修复 man: fsck_ffs [8] 的错误
- 修复 man: killpg [2] 的错误
- 改进 hwpmc
- 改进 vmm
- 更新 libfido2 至 1.9.0 版本
- 各种 LinuxKPI 802.11 的改进
- 各种 riscv 的改进
- 从版本 4.9.3 到版本 4.99.4 的 tcpdump 供应商导入和更新

基金会技术团队的成员在 5 月 17 日至 18 日在加拿大渥太华举行的开发者峰会上发表了演讲。这包括主持谷歌代码之夏、[FreeBSD 基金会](https://wiki.freebsd.org/DevSummit/202305?action=AttachFile&do=view&target=FreeBSD_Foundation_Devsummit_Spring_2023_Day_2.pdf) [技术审查](https://wiki.freebsd.org/DevSummit/202305?action=AttachFile&do=view&target=FreeBSD_Foundation_Devsummit_Spring_2023_Day_2_part1.pdf) 和 [工作流](https://docs.google.com/presentation/d/e/2PACX-1vSnEW5Z0ttQOAeqEEY8KHkfiRGeFUm4i8XrYsfY8TNYD--yx1P6MUu2_u-mCcpe6PMMITjeDIgT31CC/pub) 工作组会议。基金会团队成员 Pierre Pronchery 讨论了 [BSD 之间的驱动器协调性](https://www.bsdcan.org/events/bsdcan_2023/schedule/speaker/89-pierre-pronchery/)，而 En-Wei Wu 则讨论了在基金会合同下完成的 [wtap 工作](https://www.bsdcan.org/events/bsdcan_2023/schedule/session/139-add-operating-modes-to-wtap4/)。

## 基本系统中的 OpenSSL 3 更新

正如先前提到的，上游对 OpenSSL 1.1 分支的支持即将移除，这是发布 FreeBSD 14.0 的主要障碍。新的基金会团队成员 Pierre Pronchery，以及 Ed Maste 和 FreeBSD 开发社区的其他成员开始更新我们的基本系统中的 OpenSSL 到版本 3。

OpenSSL 是一个用于通用加密和安全通信的库。它提供了 SSL 和 TLS 网络协议的开源实现，这些协议广泛用于应用程序，如电子邮件、即时通讯、Voice over IP（VoIP）或更重要的是全球 Web（即 HTTPS）。假设 Apache 和 nginx Web 服务器使用 OpenSSL，它们的网络流量的合并市场份额超过 50%，这将巩固 OpenSSL 作为互联网基础设施的领导地位和关键重要性。

自 2016 年 8 月首次发布以来，OpenSSL 1.1 分支已被大多数 Linux 和 BSD 系统采用，并通过长期支持政策得到上游维护人员的支持。然而，官方支持计划在今年 9 月中旬结束，因此迫切需要考虑采用其继任者 3.0 分支来进行长期支持。

OpenSSL 在过去的几年中已经发展壮大，现在有超过 50 万行的代码分布在 2000 多个文件中。或许作为其结果，它的构建系统相对复杂，通常需要 Perl，而自从 5.0-RELEASE 以来，Perl 已从 FreeBSD 的基本系统中移除。然而，幸运的是，可以按照 FreeBSD 的方式导入和设置 OpenSSL 的 3.0.9 版本，并且现在已经成为计划中的 FreeBSD 14.0 RELEASE 的一部分。

可以说，OpenSSL 3 是一个新的重大发布。首先，它有问题的许可模型终于得到了解决，完全转向了 Apache License 2.0。然后，OpenSSL 3 引入了提供程序模块的概念。虽然过时的加密算法已被隔离到“legacy”模块中，但也可以将实现限制为 [FIPS](https://en.wikipedia.org/wiki/Federal_Information_Processing_Standards) 标准的部分，其中包括 fips 模块。然后，后者可以从专用的认证流程中获益，并且可以得到官方的验证（例如在撰写这些内容时的 3.0.8 版本）。

此外，更新的库带有版本升级，因为使用 OpenSSL 1.1 的应用程序需要重新编译以使用 3.0。许多 API 函数已被弃用，并用更新、更通用的替代方法取而代之，但仍然可以明确地请求旧的 API，并让 OpenSSL 3 相应地公开它们。FreeBSD 已经利用了这个可能性来帮助过渡，其中一些库和应用程序简单地被配置为请求 OpenSSL 1.1 的 API。这些组件将逐步更新，以便在未来的时间内使用 OpenSSL 3 的本地 API。

虽然与更新相关的一个已知的性能影响是在使用小输入块大小时，但在处理 1KB 及以上大小的块时，其影响较小。另一个挑战在于 FIPS 提供程序模块，目前需要一些手动步骤才能使其正常工作。我们目前正在寻找一种解决方案，以便默认情况下将 FreeBSD 与功能性的 FIPS 提供程序一起交付。

## 将 amd64 的最大 CPU 数从 256 增加到 1024

默认的 amd64 和 arm64 FreeBSD 内核配置当前支持最多 256 个 CPU。可以通过设置内核参数 MAXCPU 来构建支持更大核心数量的自定义内核。然而，拥有超过 256 个 CPU 的商品系统正在变得越来越普遍，并且在 FreeBSD 14 的支持生命周期中将越来越常见。我们希望将默认的最大 CPU 数增加到 1024，以便在 FreeBSD 14 上支持这些系统。基金会团队成员 Ed Maste 和其他开发人员已经进行了许多更改，以支持更大的默认 MAXCPU，包括将 cpuset_t 的用户空间最大值修复为 1024。还进行了一些更改，以避免静态的 MAXCPU 大小的数组，将其替换为按需内存分配。还需要额外的工作来继续减少由 MAXCPU 大小的静态分配和解决非常高核心数系统上的可扩展性瓶颈，但目标是在支持大型 CPU 数的情况下发布具有稳定 ABI 和 KBI 的 FreeBSD 14。

## 谷歌代码之夏

2023 年版的谷歌代码之夏的编码刚刚过了中点，七个 FreeBSD 项目的进展情况已在 WIKI 上有所记录。基金会团队成员担任我们的组织管理员，并且有三个项目的导师与基金会有关联。

## 引导加载程序的 CI 测试工具

FreeBSD 支持多种体系结构、文件系统和磁盘分区方案。为此，Sudhanshu Mohan Kashyap <smk@FreeBSD.org> 正在编写一个 Lua 脚本，以允许对所有这些组合进行引导加载程序的测试，适用于一级和二级架构。如果时间允许，还可以进一步探索将脚本集成到现有的构建基础设施（无论是 Jenkins 还是 Github Actions）中，以生成测试结果的综合摘要。

目前，任何开发人员的更改可能会阻止操作系统在某些特定环境中启动的能力。这些脚本确保更改不会导致经过测试的环境出现退化。这些脚本设计得非常高效，成本远低于当前所需的完整 make universe。这些属性使开发人员可以定期使用脚本，并允许将其集成到 CI 流程中，而不会产生不必要的成本。

## 用于 FreeBSD 内核的物理内存压缩

大多数现代 CPU 架构通过支持大于标准页面大小的页面来提供性能提升。不幸的是，由于高度的物理内存碎片化，分配此类页面可能会失败。这项工作实现了物理内存压缩，作为主动减少运行中系统中碎片的一种方法。这个谷歌代码之夏的目标是向虚拟内存子系统添加各种物理内存抗碎片措施。

Differential [D40575](https://reviews.freebsd.org/D40575) 实现了用于量化物理内存碎片程度的众所周知的度量标准。Differential [D40772](https://reviews.freebsd.org/D40772) 实现了物理内存压缩，并添加了一个守护程序，该守护程序监视系统并在需要时执行压缩。

计划未来的工作包括设计适当的基准测试套件，运行测试，并根据评审和测试结果调整代码。这仍然是一个正在进行中的工作，因此任何测试、评审和反馈都将不胜感激。

## 将 mfsBSD 集成到发布构建工具中

mfsBSD 是由 Martin Matuška 创建的工具集，用于创建基于 mfsroot 的 FreeBSD 分发的小型但功能齐全的发行版。也就是说，它通过内存文件系统（MFS）从存储设备加载文件并将它们存储在内存中。作为这个项目的一部分，Soobin Rho <soobinrho@FreeBSD.org> 将在 src/release makefile 中为 -current 和 -stable 版本的 mfsBSD 镜像创建为每周快照的附加目标。目前，只能生成 RELEASE 版本的 mfsBSD 镜像，这意味着它们往往会与基础工具不同步。该项目旨在解决这个问题。

## FreeBSD 在 Microsoft HyperV 和 Azure 上的支持

FreeBSD Azure Release Engineering 团队的成员 Li-Wen Hsu、Wei Hu <whu@FreeBSD.org> 以及 Microsoft FreeBSD Integration Services 团队的成员 Souradeep Chakrabarti <schakrabarti@microsoft.com> 等人一直在努力将 FreeBSD ARM64 架构支持添加到 Azure 平台。这项工作的一部分涉及将镜像发布到 Azure 社区库。在项目的测试公共库中有一些测试镜像，名为 `FreeBSDCGTest-d8a43fa5-745a-4910-9f71-0c9da2ac22bf`：

- FreeBSD-CURRENT-testing
- FreeBSD-CURRENT-gen2-testing
- FreeBSD-CURRENT-arm64-testing

要使用这些镜像，在创建虚拟机时：

1. 在 *Select an Image* 步骤中，在“*Other items*”中选择“*Community Images (PREVIEW)*”。
2. 搜索 *FreeBSD*。

正在进行的工作包括：

- 自动化镜像构建和发布过程，并合并到 `src/release/`。
- 构建并发布基于 ZFS 的镜像到 Azure Marketplace：
  - 所需的所有代码都已合并到主分支，可以通过指定 `VMFS=zfs` 来创建基于 ZFS 的镜像。
  - 未来的任务包括改进构建自动化，并与发布工程合作开始生成快照。
- 构建并发布 Hyper-V gen2 VM 镜像到 Azure Marketplace。
- 构建并发布快照版本到 Azure 社区库。


以上任务由 FreeBSD 基金会赞助，资源由微软提供。

来自 Wei Hu 和 Souradeep Chakrabarti 的工作涉及多项由微软赞助的任务，包括：

- 将 Hyper-V 客户端支持移植到 aarch64 架构
  - <https://bugs.freebsd.org/267654>
  - <https://bugs.freebsd.org/272461>

待处理任务：

- 在 Microsoft Learn 上更新与 FreeBSD 相关的文档
- 在 Azure Pipelines 中支持 FreeBSD
- 将 Azure 代理 Port 更新到最新版本
- 提交 Azure 代理的本地修改内容给上游
