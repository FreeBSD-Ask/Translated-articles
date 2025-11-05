# FreeBSD OCI 容器简介

- 原文 [An introduction to OCI Containers on FreeBSD](https://freebsdfoundation.org/blog/oci-containers-on-freebsd/)
- 发布日期：2025 年 10 月 31 日

《在 FreeBSD 上运行 Linux 容器！》<https://www.youtube.com/watch?v=HV-wUUzRCMo>

我想你在过去十年里大概没有与世隔绝，所以我就不解释什么是 [容器](https://en.wikipedia.org/wiki/Containerization_%28computing%29) 了。不过我会提到什么是开放容器倡议（Open Container Initiative，简称 OCI），因为 FreeBSD 刚刚成为了它的一部分。引用 [他们自己网站](https://opencontainers.org/) 上的话：

> “开放容器倡议（OCI）是一款轻量级、开放治理结构，其明确目的是围绕容器格式和运行时创建开放的行业标准。”

很好。为了在这个框架内工作，我们需要一款工具。登场的是 Podman：

> “Podman 是一款功能齐全的容器引擎，是一款简单的无守护进程工具。Podman 提供了与 Docker CLI 可比的命令行，使从其他容器引擎的迁移变得容易，并可管理 pod、容器和镜像。”

规则已经设定。让我们看看 Podman 所谓的“简单”，是否真的名副其实。

## 安装与设置

我这里从全新的 FreeBSD 15.0-BETA3 安装开始，但我也在 CURRENT、STABLE、14.3-RELEASE 上进行了测试，结果完全一致。让我们安装所需包：

```sh
# pkg install -y podman-suite
```

Podman 默认会把容器存储在 `/var/db/containers`，所以你可能想为它创建专用的 ZFS 文件系统：

```sh
# zfs create -o mountpoint=/var/db/containers zroot/containers
```

如果我们希望访问运行在容器中的任何服务，就需要使用 pf 防火墙。podman 有个示例配置文件，让事情变得简单：

```sh
# cp /usr/local/etc/containers/pf.conf.sample /etc/pf.conf
# sysctl net.pf.filter_local=1
```

**别忘了在启用并启动 pf 服务之前，更新复制文件中的接口变量。** 当然，如果你已经在运行 pf，并且已经有配置文件，请不要直接覆盖。只需从示例文件中取出相关部分，添加到现有配置中即可。

```sh
# service pf enable
# service pf start
```

如果我们想使用任何基于 Linux 的容器，还需要启用 Linux 服务：

```sh
# service linux enable
# service linux start
```

准备就绪。我们来试试 FreeBSD 容器（这里我们是普通用户，所以需要使用权限提升，因为目前 podman 需要 root——我个人偏好使用 doas；它小巧而简单）：

```sh
$ doas podman run ghcr.io/freebsd/freebsd-runtime:14.3 freebsd-version
14.3-RELEASE
```

非常简单！我们再测试一个 Linux 容器：

```sh
$ doas podman run --rm --os=linux docker.io/alpine cat /etc/os-release | head -1
NAME="Alpine Linux"
```

同样非常简单。

你可以在 [GitHub](https://github.com/orgs/freebsd/packages) 和 [Docker Hub](https://hub.docker.com/u/freebsd) 上找到 FreeBSD OCI 镜像。不过我们可以想象，许多人更希望使用 Linux 镜像。

## 测试一些有用的东西

我很早前就听说过 Caddy，当时是在与一家基于 FreeBSD 的公司讨论大型多人在线游戏服务时（那是另一个有趣的故事，以后再说）。简单了解后我发现，它可能是一个在容器中安全地提供网站服务的极其简单的方法。于是我用 Hugo 生成的静态网站进行了测试——仅仅是提供 `public` 文件夹。

```sh
$ doas podman run --os=linux -p 80:80 -v $PWD/website:/usr/share/caddy -v caddy_data:/data docker.io/caddy
```

![](https://freebsdfoundation.org/wp-content/uploads/2025/10/running-caddy-1024x721.png "运行 Caddy")

哇，太简单了！

## 我能得到什么？

除了让现有的 FreeBSD 用户能使用容器之外，Podman 还将把一大批新用户带到 FreeBSD 世界。我们已经看到，越来越多的新用户访问 [YouTube 频道](https://youtube.com/@freebsdproject) 和 [Subreddit](https://reddit.com/r/freebsd)，希望在 FreeBSD 以简洁和可靠著称的基础上构建他们的数字世界。ZFS 万岁 💪

因此，我决定尝试一个有点奇特的挑战。很多年前，我在 Docker Hub 上发布过一个容器：以 CentOS 为基础的 Python Pandas。那我能否在我崭新的 OCI FreeBSD 服务器上运行它？

```sh
doas podman run -it --os=linux docker.io/phips/pandas:v3 /bin/bash
```

![](https://freebsdfoundation.org/wp-content/uploads/2025/10/running-old-pandas-1024x721.png "运行旧版 Pandas")

可以！太棒了！这意味着可以将旧的工作负载迁移到新平台上。

### 快速又简单

就是这样。在 FreeBSD 上启动并运行容器既简单又快捷。同时，它也非常容易使用旧的容器——为将底层平台迁移到一个精心设计、坚如磐石的操作系统提供了路径。

FreeBSD 手册很快将更新，加入 OCI 容器的正式文档，但与此同时，你可以查看核心成员 Dave Cottlehuber 的[工作文档](https://docs.skunkwerks.at/s/fUiAmi4pE)。

我们会定期发布新的技术主题文章和视频，因此请务必订阅 [YouTube 频道](https://youtube.com/@freebsdproject)，并在你喜欢的 RSS 阅读器中关注[此订阅源](https://freebsdfoundation.org/our-work/latest-updates/)。如果你希望通过电子邮件接收更新，也可以订阅[我们的通讯](https://mailchi.mp/freebsdfoundation.org/newsletter-sign-up)。

我们也希望这个内容系列是互动的——你希望我们讨论什么？有哪些 FreeBSD 问题可以帮你解决？请[联系我们](https://freebsdfoundation.org/about-us/contact-us/)，告诉我们你的想法。
