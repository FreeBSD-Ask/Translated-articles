# FreeBSD Jail 的安全性

- [FreeBSD Jail Security](https://vermaden.wordpress.com/2025/04/11/freebsd-jails-security/)
- 作者：vermaden
- 发布时间：2025/04/11


我认为这个话题在网络上讨论得并不充分——而且常常伴随着多重误解。

![](https://vermaden.wordpress.com/wp-content/uploads/2025/04/containers-security-title.jpg)

似乎普遍存在一种观点，认为 **Linux** 上的 Podman 与 **FreeBSD Jail** 一样安全……接下来我们试着深入探讨这个问题。

下面我将尝试揭示 **FreeBSD Jail** 与 **Linux Podman 容器** 在安全性上的所有差异。

## 基本概念

![](https://vermaden.wordpress.com/wp-content/uploads/2025/04/containers-security-hier.png)

### Podman

通常，当创建新容器时，需要从注册中心下载某个镜像——例如 **registry.fedoraproject.org/fedora-minimal:30**——然后根据需要添加所需文件和软件包。

另一种方法是使用 [scratch](https://docs.docker.com/build/building/base-images/#create-a-minimal-base-image-using-scratch) 作为“空”镜像来创建 Podman 容器，仅包含所需的二进制文件。

### Jail

在历史上，Jail 被用作内部较完整的 FreeBSD 系统——其版本等于或低于宿主机版本。

概念很简单——你创建一个目录（或 ZFS 数据集）作为根目录，并在其中解压 FreeBSD **基本系统**。

```sh
freebsd # mkdir -p /jail/test
freebsd # fetch https://download.freebsd.org/releases/amd64/14.2-RELEASE/base.txz
freebsd # tar -C /jail/test --unlink -xf base.txz
```

然后在 **/etc/jail.conf** 中进行一些简单的配置，或者作为单独文件 **/etc/jail.conf.d/test**，就可以启动并使用 Jail。

大多数人不知道，Jail 完全可以像 Podman 或 Docker 那样使用——根本不需要最小系统镜像。甚至可以创建只包含单个文件的 Jail——下面我给你演示一下。

```sh
freebsd # mkdir -p /jail/minimal/dev
freebsd # cp /rescue/sh /jail/minimal
```

甚至不需要配置文件 **/etc/jail.conf**、**/etc/jail.conf.d/minimal**。

只需在命令行传入必要选项直接启动 Jail——执行 **jail(8)** 命令后，你就会进入这个 **minimal** Jail 环境。

```sh
freebsd # jail -n minimal            \
            -c path=/jail/minimal    \
               host.hostname=minimal \
               ip4.addr=10.0.0.111   \
               mount.devfs           \
               command=/sh

minimal # /sh
Cannot read termcap database;
using dumb terminal settings.

minimal # for I in 1 2 3; do echo ${I}; done
1
2
3

minimal # echo /*
/dev /sh
```

可以在宿主系统的另一个终端检查，确认该 Jail 是否正在运行。

```sh
freebsd # jls
   JID  IP Address      Hostname                      Path
     1  10.0.0.111      minimal                      /jail/minimal
```

### 基本概念与安全总结

有句 UNIX 老话：“少即是多”……同时也意味着更安全。各类安全指南和最佳实践也强调同样的原则——服务中包含的内容越少，解决方案就越安全，潜在攻击面就越小。

Jail 和 Podman 都无需任何最小系统或镜像即可启动——只需为你的服务准备必要的文件/可执行程序（当然包括依赖库）。

## Root 用户

![](https://vermaden.wordpress.com/wp-content/uploads/2025/04/containers-security-root.png)

这一点很简单。‘rootless’ 模式下的 Podman 不使用宿主系统的 **root** 用户——Jail 也一样，它们有一个虚拟 **root**，与宿主 **root** 完全无关。

## 隔离层

![](https://vermaden.wordpress.com/wp-content/uploads/2025/04/containers-security-onion.png)

### 纯 Podman 与 Jail

所谓的 ‘rootless’ Podman 被宣传为与 Jail 具有相同级别的隔离——但前提是你必须在 Podman 上正确配置并启用 SELinux/AppArmor。没有 SELinux/AppArmor 时，Jail 提供的隔离远比 ‘rootless’ Podman 更强。

### 配置额外安全层后的 Podman 与 Jail

当你在 Podman 上正确配置 SELinux/AppArmor 后，如果在 FreeBSD 上添加 MAC Framework 模块，如：

* **mac_sebsd**
* **mac_jail**
* **mac_bsdextended**
* **mac_portacl**

在添加这些后，Jail 的隔离能力再次优于 Podman。

### 隔离层安全总结

单独运行时，FreeBSD Jail 提供的隔离性就比 Podman 更强；即便在 Podman 上添加了额外的隔离层，Jail 仍然更加安全。

## 内核系统调用表面

![](https://vermaden.wordpress.com/wp-content/uploads/2025/04/containers-security-syscall.png)

### Podman

即便是 ‘rootless’ Podman，也可以完全访问所有 Linux 内核系统调用——除非通过 **seccomp** 命令/方案为任务应用默认配置文件，或自定义生成的配置文件进行限制。

### Jail

Jail 默认就限制了 FreeBSD 内核系统调用的使用——无需任何额外工具——如果需要，还可以通过 MAC Framework 进一步缩小范围。相反，在 FreeBSD 上可以通过 **devfs(8)** 命令/方案增加额外系统调用。

### 内核系统调用安全总结

在系统调用攻击面上，Jail 比 Podman 更安全——因为 Podman 需要额外工具（和配置文件）来限制系统调用范围，而 FreeBSD Jail 默认就将系统调用暴露降至最小。

## 专用网络接口

![](https://vermaden.wordpress.com/wp-content/uploads/2025/04/containers-security-network.png)

### Podman

你无法将任何物理接口专用于 ‘rootless’ Podman 容器。

不过有一些变通方法：

* 使用 **root** 权限的辅助方案，如 **macvlan**——但它在 **root** 上下文/权限下运行。
* 降低安全性，改用 ‘rootful’ Podman，此时容器中的 **root** 与宿主系统 **root** 完全相同。

### Jail

FreeBSD Jail 可以配置以下网络选项：

* 零网络接口。
* 虚拟 **tap(4)** 网络接口。
* 虚拟 **tun(4)** 网络接口。
* 虚拟 **epair(4)** 网络接口。
* 虚拟 **ng_ether(4)** Netgraph 网络接口。
* 来自宿主的任意专用物理网络接口。
* 来自宿主的 SR-IOV 物理网络接口的任意部分/片段。

这意味着你可以创建一个 Jail，该 Jail 甚至无法从宿主系统访问，只能通过网络访问。

### 专用网络接口安全总结

Podman 在这一点上劣势明显。无法使用任何宿主物理网络接口，极大限制了安全可能性。

## 容器内专用防火墙

![](https://vermaden.wordpress.com/wp-content/uploads/2025/04/containers-security-firewall.png)

### Podman

你无法在 ‘rootless’ Podman 容器内运行防火墙。

### Jail

你可以在 Jail 内运行完整专用的（与宿主隔离的）网络栈——并且可以在 VNET Jail 中独立运行任何防火墙，如 **pf(4)** 或 **ipfw(4)**，完全独立于宿主。

### 容器内专用防火墙安全总结

在每个 Jail 中运行专用防火墙，能够显著提升其安全级别。

## 实战验证程度

![](https://vermaden.wordpress.com/wp-content/uploads/2025/04/containers-security-battle.jpg)

### Podman

Docker——Podman 的前身——自 2014 年起开始使用，也就是比 Jail 晚 **约 15 年**，而 Jail 最早在 1999 年开发。

Podman 最安全的变体——‘rootless’ 模式——首次出现在 2019 年末（1.6 版本发布），也就是投入使用时间刚刚超过 **5 年**。

### Jail

Jail 自 1999/2000 年推出以来就进入生产环境——已有 **25 年**，久经考验。

这意味着，Jail 是所有方案中经过实战验证最充分的。

**总结**：与 Podman 容器相比，FreeBSD Jail 开箱即用时通常更安全；如果再添加额外安全层，安全性更高。

## CVE 漏洞

![](https://vermaden.wordpress.com/wp-content/uploads/2025/04/containers-security-cve.jpg)

有官方的 CVE 漏洞数据库，大家可以查询感兴趣的项目中发现了多少安全漏洞。

### Podman

我们先统计一下已知的 Linux 相关 CVE 数量。

```sh
% links -dump -width 512 'https://app.opencve.io/cve/?vendor=linux' | grep -i TOTAL
   Total 10064 CVE
``

已知 CVE 数量是 **147**（截至最近公开数据）。

目前与 Podman 相关的：

```sh
% links -dump -width 512 'https://app.opencve.io/cve/?search=podman' | grep -i TOTAL
   Total 24 CVE
```

在过去 **5 年** 的市场使用期间，Podman 出现了 **24** 个 CVE——平均每年约 **5 个** CVE。

### Jail

现在轮到 FreeBSD 系统了。

```sh
% links -dump -width 512 'https://app.opencve.io/cve/?vendor=freebsd' | grep -i TOTAL
   Total 557 CVE

% links -dump -width 512 https://www.freebsd.org/security/advisories/ | grep -c BSD-SA
649
```

看起来 FreeBSD 项目发布的 *Security Advisories*（**649**）比已存在的 CVE 数量（**557**）还要多 🙂

现在让我们来查看 Jail 的漏洞记录。

```sh
% links -dump -width 512 https://freebsd.org/security/advisories/ | grep -i jail 
   2021-04-06 FreeBSD-SA-21:10.jail_mount            
   2021-02-24 FreeBSD-SA-21:05.jail_chdir            
   2021-02-24 FreeBSD-SA-21:04.jail_remove           
   2020-03-19 FreeBSD-SA-20:08.jail                  
   2010-05-27 FreeBSD-SA-10:04.jail                  
   2007-01-11 FreeBSD-SA-07:01.jail                  
   2004-06-07 FreeBSD-SA-04:12.jailroute             
   2004-02-25 FreeBSD-SA-04:03.jail                  

% links -dump -width 512 https://freebsd.org/security/advisories/ | grep -c jail
8
```

在 FreeBSD 项目中，已知与 Jail 相关的安全问题有 **8 个**。

接下来，我们将其与 CVE 数据库中的记录进行对比。

```sh
% links -dump -width 512 'https://app.opencve.io/cve/?search=jail&vendor=freebsd' | grep -i TOTAL
   Total 18 CVE
```

在 25 年多的时间里，总共只有 **18** 个 CVE——也就是说每年不到一个 CVE，准确来说是 **0.7 个**。

### CVE 安全总结

Linux 系统已知安全 CVE 为 **10064**，而 FreeBSD 只有 **557 个**，相比 **649 个** 这个数量少了一个数量级以上。

Jail 每年大约 **0.7 个** CVE 安全问题，而 Podman 每年约 **5 个** CVE 安全问题。

赢家显而易见——再次是 FreeBSD Jail。

## 总结

众所周知且有文档记录，FreeBSD Jail 在安全性和灵活性上远超 Linux 上的 Podman（即便是 ‘rootless’ 模式）。

欢迎大家参与讨论并发表自己的看法。

## 更新 1 – 评论后的感想

### 失望

当我只写 FreeBSD 相关内容——不涉及 Linux 时——大家似乎都很喜欢；但只要我深入 Linux 领域进行对比、分析优劣，就会收到更多反对意见。正如那句众所周知的格言所说：**“你有敌人？很好。那意味着在你一生中的某个时刻，你曾经站出来维护过某样东西。”（英国首相丘吉尔）**——这大概是我在发布最近三篇文章后的最佳总结。顺便列出文章记录：

* [Minecraft Server in FreeBSD Jail Container](https://vermaden.wordpress.com/2025/04/05/minecraft-server-freebsd-jails-container/)
* [Are FreeBSD Jail a Containers?](https://vermaden.wordpress.com/2025/04/08/are-freebsd-jails-containers/)
* [FreeBSD Jail Security](https://vermaden.wordpress.com/2025/04/11/freebsd-jails-security/)

它们的核心在于，将 FreeBSD Jail 讨论置于 **容器** 与 **容器安全** 的语境下。

有人指责我不公平或不理解情况……但要深入了解 Podman 容器的内部机制确实非常困难。

举个例子——当 Podman 容器启动时，会运行 **seccomp** 命令（使用默认策略，也就是参数 **--seccomp-policy=POLICY**）来限制/管理内核系统调用……但具体限制哪些调用？你试过在线查找这个信息吗？我的意思是——是的，我可以从头安装所有东西，然后查看软件包里的文件……但官方文档在哪里？面向何种 Linux 发行版？因为发行版太多了。

这只是其中一个例子。

再举一个例子，我提到 VNET Jail 可以为 FreeBSD Jail 提供独立的网络栈，并能在其中运行专用防火墙，比如 **pf(4)** 或 **ipfw(4)**。有人评论说，Linux 下同样可以通过 **CAP_NET_ADMIN** 权限（使用 **--cap-add=NET_ADMIN** 参数）实现……好，但我查阅 Linux 的 **capabilities(7)** 手册页，它是这样描述的：

```sh
linux # man 7 capabilities | grep -m 1 firewall
- administration of IP firewall, masquerading, and accounting;
```

很好。但这个防火墙是在宿主的防火墙上，还是在 Podman 容器内部、与宿主隔离的防火墙里呢？……即便是在 Podman 容器内，它是和宿主一样的防火墙，还是可以独立为不同防火墙？

……而如果我去问“官方文档里具体描述在哪里”，最常见的回答大概是：**“在源代码里”**。

### 无知与现实

我甚至和一个朋友——我们称他为 *Marian*（化名）——聊过这件事，因为我不想透露他的真实身份。他过去使用过 FreeBSD 和 OpenBSD（也包括 Solaris/AIX/HP-UX 等），在最近几个工作岗位中担任 SRE，使用 AWS（亚马逊云）提供解决方案。他说，他非常喜欢我最近写的 *FreeBSD Jail* 系列文章，而在他所接触的云相关 Linux/容器圈子里，几乎没人会深入去了解这些差异或实际安全性。

他们只是在某个 AWS 托管的 K8S 服务中使用容器，确保所需服务提供“构建块”，仅此而已。没有人真正关注这种级别的细节。没人关心。

### OCI 容器

灵感来源于常见的指责——有人说 FreeBSD Jail 不是容器。在我看来，FreeBSD Jail 是可用的容器解决方案之一。

但如果你想要基于 FreeBSD Jail 提供 OCI 兼容的容器解决方案，可以结合 **pot** 或 **nomad** 使用，这样就可以得到基于 FreeBSD Jail 的 OCI 兼容容器。
