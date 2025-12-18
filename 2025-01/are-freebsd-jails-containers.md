# FreeBSD Jail 是容器吗？

- 原文：[Are FreeBSD Jail a Containers?](https://vermaden.wordpress.com/2025/04/08/are-freebsd-jails-containers/)
- 作者：vermaden
- 发布时间：2025/04/08

我相信这不是第一次——当我发布某篇文章，而它在同一句话里同时包含 **FreeBSD Jail** 和 **容器** 时——就会引发激烈的讨论。

至少有以下这些：

- [FreeBSD Jail Container 中的 Minecraft 服务器](https://vermaden.wordpress.com/2025/04/05/minecraft-server-freebsd-Jail-container/)
- [安全的容器化浏览器](https://vermaden.wordpress.com/2021/12/15/secure-containerized-browser/)
- [FreeBSD Jail Containers](https://vermaden.wordpress.com/2023/06/28/freebsd-Jail-containers/)
- [FreeBSD Containers 上的 RabbitMQ 集群](https://vermaden.wordpress.com/2019/06/05/rabbitmq-cluster-on-freebsd-containers/)

有些人认同我的观点——认为 **FreeBSD Jail** 就是 **容器**——而有些人则完全否认。

今天我将尝试为这个问题找到最终且唯一的答案。

## 支持 Jail 是容器的理由

依我看，**容器** 这个术语并不专属于 **Linux** 的 **Podman**/**Docker** 方案。在 **Docker** 或 **Podman** 出现之前，**容器** 这一概念早已存在于 **BSD/UNIX** 世界中——而现在却让这些后来者来定义什么是 **容器**？

在 2000 年，**FreeBSD Jail** 就被作为一种 **容器** 技术引入——甚至有人称 Jail 为增强版的 **chroot(8)**。

众所周知，不仅是在 **FreeBSD** 社区，**[Allan Jude](http://allanjude.com/)** 最近在 [Changelog Interviews – 574 – Lets Talk FreeBSD](https://changelog.com/podcast/574) 中也谈到了以下内容：

>“FreeBSD 在 2000 年率先实现了 **容器** 的实用化，而我们在 **Linux** 上则是在很久以后才看到 **容器** 和 **命名空间（namespaces）**。自那以后可能在其上构建了更多功能，但这一概念最初来自 **BSD**。”

……在该采访中他还提及：

>“FreeBSD 的 **容器** 概念被称为 **Jail**。**Jail** 基本上是 **chroot(8)**，包含了预装好的不同的操作系统，可以安装其他软件包等等。它们对可使用的 IP 地址有约束，可以加以限制。后来又增加了许多功能，比如类似于 **Linux** 的网络 **命名空间**，让它几乎可以像轻量级 VM 一样运行。它拥有自己的网络栈、文件系统等……这就是我在 **FreeBSD** ZFS 机器上运行 Plex 的方式：我创建 **Jail**，然后在其中用 **FreeBSD** 的包管理器安装 Plex。”

……更进一步：

>“这个概念不断发展，Sun 看到了这一概念，并在此基础上构建了自己的 **Zones**……这也是为什么 ZFS 支持将数据集委托给 **容器**，因为 **Solaris** 借鉴了 **FreeBSD** 的思想。最终，ZFS 的这些特性又回到了 **FreeBSD** 和 **Linux**。”

……还有：

>“它还有个概念 **VNET**，你可以拥有完全隔离的 **FreeBSD** 网络栈副本。在这些 **容器** 中，它们可以拥有完整的网络栈；这意味着它们可以有自己的防火墙。每个 **容器** 都可以有独立防火墙和规则，甚至可以使用冲突的 IP 地址。只要不互联，它们互不干扰，也不会冲突。”


换句话说——**Allan Jude** 一直把 **FreeBSD Jail** 称作 *容器*。

2004 年，Sun 推出了 [**Solaris 容器**](https://en.m.wikipedia.org/wiki/Solaris_Containers)（也称 **Solaris Zones**）——比 **FreeBSD Jail** 整整晚了四年，它们都是完全相同的 **操作系统级虚拟化** 技术。

HP 的 **HP-UX** 系统（最后一次官方发布为 **11i v3**，B.11.31，2007 年，之后有一些补丁）也有 **HP-UX 虚拟分区**，官方称为 [**vPars 容器**](https://docs.microfocus.com/SA/10.51/Content/V12N/hpux_v12n/ManagingHP-UXVirtualServers.htm)，同时还有 [**HP-UX 容器 (SRP)**](https://support.hpe.com/connect/s/product?language=en_US&kmpmoid=4164838) 和 [**HP 9000 容器**](https://serviceitdirect.com/modernizing-legacy-hp-ux-servers-and-application/) 方案。

只有 IBM 在 **AIX** 系统中将其 **操作系统级虚拟化** 方案命名为 **WPAR**——Workload Partition（工作负载分区），而不是 **容器**，可能是出于潜在的法律考虑……

看起来即便是微软在 **Windows** 系统中也使用 **容器** 这个术语——我以前并不知道，他们称其为 [**Windows 容器**](https://techtarget.com/searchitoperations/tip/Windows-Server-containers-and-Hyper-V-containers-explained)。

打个比方，就像汽车一样……例如 **Ford Mustang（福特野马）** 被定义为 **Pony Car（小马车）**，搭载 1964 年起的 2.8L–4.7L I6–V8 大排量发动机。到了 1990 年，**本田** 开始把其 1.4L I4 发动机的 **Civic（本田思域）** 称作 *Pony Car（小马车）*，并且宣称 60/70 年代原本被称为 *Pony Cars* 的车型现在要改叫 *Trucks*……我并不认同这种篡改历史的做法。

另一个来自 **[johnklos](https://lobste.rs/~johnklos)** 的引用：

>“那么，流行用法的 **容器** 是否胜过其正确定义？**Linux** 的流行用法是否就能凌驾于其正确定义之上？”

还有来自 [**satanist**](https://lobste.rs/~satanist) 的引用：

>“我认为 **Docker**/**Podman**/…… 并不是 **容器** （方案）。它是 **容器** 管理工具（或类似东西）。是的，当有人解释如何在 **Linux** 上用 **cgroups** 和 **命名空间** 构建类似 **Jail** 的东西时，我也会接纳标题中使用容器。我也接受其他文章在使用时采用 **行业定义**。”

同样来自 [**satanist**](https://lobste.rs/~satanist) 的另一段话：

>“我可以理解，当你只知道 **OCI 容器** 这个概念时，听到有人说 **Jail** 是 **容器** 会感到惊讶。但在 **FreeBSD** 上，你早就拥有构建类似 **Docker** 的所有工具——早在 **Docker**（或 **OCI**） 出现之前。然而直到 **Podman** 移植之前，还没人写过或移植过类似 **Docker** 的方案。但在 **FreeBSD** 上，你完全可以管理大型基础设施，你只需要多了解一些，而不仅仅用 **OCI** 的视角。”

关于 **容器**，你可以像使用 **Docker**/**Podman** 一样，创建单进程 **FreeBSD Jail**：

- [FreeBSD Jail Containers 中的单进程 Jail](https://vermaden.wordpress.com/2023/06/28/freebsd-Jail-containers/#single-process-Jail)

你也可以用 **Bastillefile**，就像在 *BastilleBSD* 中使用 **Dockerfile** 一样：

- [Bastille 默认模板与自定义](https://bastillebsd.org/blog/2021/01/06/bastille-default-templates-and-customization/)

来自 **[satanist](https://lobste.rs/~satanist)** 的更多观点：

>“再解释得清楚一些：当我使用 **Podman** 时，我会得到或创建某个应用镜像（或镜像描述）并交给 **Podman**。**Podman** 现在会创建 **容器** 并在其中启动应用。在 **Linux** 上，这依赖 **cgroups** 和 **命名空间** 的组合。而在 **FreeBSD** 上，只需要 **Jail** 就够了。因此我会称 **Jail** 为 **容器** 。

>区别在于，在 **Linux** 上手动创建自己的 **容器** 并不轻松。你大多数情况下需要 **Docker**、**Podman** 等 **容器** 部署方案。而在 **FreeBSD** 上，你可以很轻松地手动操作自定义的 **容器** 。

>这也是我讨厌 **Docker** 和所有现代 **容器** 方案的原因。它暗示你必须依赖 **Docker** 类的 **容器** 部署来在大系统中使用软件。这导致 **Docker** 成为你的主要操作系统。我知道 DevOps 很喜欢这种方式，但我认为 DevOps 是一个大错误。”

我个人不接受那些起源或以 **Linux** 为导向的人，可以“夺走”在 UNIX 系统中被使用数十年、作为 **操作系统级虚拟化** 技术别名的术语：**容器**。

## 反对 Jail 是容器的理由

有人告诉我，当 **FreeBSD Jail** 几乎不具备 **OCI 容器** 的特性时，就不要把它们称作 **容器**。

我认为大多数反对意见来自 [**David Chisnall**](https://lobste.rs/~david_chisnall)，例如：

>“**容器** 拥有镜像抽象，让你可以通过对 CoW 快照应用增量生成文件系统。这在分发和部署模型中是构建模块。**容器** 将镜像与每次部署的状态分离（状态可以是临时的、作为附加层构建的，或作为跨多次 **容器** 调用可共享的卷）。

>**Jail** 是实现 **容器** 模型的 **FreeBSD** 特性之一。ZFS 也是。**RACCT** 和 **pf(4)** 也是其他手段。

>仅使用 **Jail** 无法构建完整的 **容器** 部署，它只是 **Jail** 的部署。这对某些使用场景有用，但每次你发布类似内容，声称它是 **容器** ，就会让 **Linux** 用户认为 **FreeBSD** 没有 **容器** 解决方案，从而降低他们切换的可能性——这是不准确的（**containerd**/**Podman**、**xc** 和 **potluck** 都是 **FreeBSD** 上可行的 **容器** 系统，尽管我认为只有前三者符合 **OCI** 标准）。”

……还有其他评论中提到的观点：

>“对于管理大规模 **容器** 部署的人来说，**容器** 首要是用于打包、分发、部署和编排的模型。它们依赖隔离技术（有多种选择，包括 **Linux** 上的 **Kata 容器**，创建非常轻量的 **Firecracker VM**，与主机 OS 共享文件系统），但这是实现细节。

>换个角度，如果有人写教程讲如何设置 **cgroups** 和 **命名空间**，然后称其为 **容器** 教程，你会觉得合理吗？还是觉得此人根本不理解 **容器** ？

>再想象如果 **FreeBSD** 是主流开源平台，每个人都用 **Docker** 或 **Podman** 做 **容器** 部署。你看到一篇文章讲如何在 **Linux** 上设置 **命名空间** 并用 **cgroups** 限制它们，然后安装程序运行。你的反应会是‘哦，Linux 有不错的 **容器** 方案’还是‘Linux 人根本不了解 **容器** ’？”

……还有更多观点：

>“我不反对有人写教程讲如何在 **FreeBSD** 上使用 **Jail**。它们是自包含抽象，可与或不与 **容器** 一起使用，这是有价值的。写文章说明‘**FreeBSD** 有几个很棒的 **容器** 方案，但它们都使用这个核心原语进行隔离，你甚至可以在没有完整 **容器** 基础设施的情况下使用它，这里是方法’——这是很好的 **FreeBSD** 宣传。

>但当业界对 ‘**容器**’ 已有共识，而你写了标题含 ‘**容器**’ 却没有涉及这些内容的文章，这就是反对 **FreeBSD** 的好营销。”

互联网上可能还有其他类似的声音——如果我漏掉了重要观点，你知道怎么提醒我。

## 容器与 OCI 容器

这似乎是一切误解的根源。

依我看，**FreeBSD Jail** 是 **容器** 技术——但单独使用它们并不是 **OCI 容器**……而现在很多人把 **容器** 等同于 **OCI 容器**。

我认为这正是问题的根源，也是一切误解的来源。

