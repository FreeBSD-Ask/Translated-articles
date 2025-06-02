# systemd 在任何地方都不安全

- 原地址：<https://unixsheikh.com/articles/systemd-isnt-safe-to-run-anywhere.html>
- 译者：ykla & ChatGPT
- 最后发布日期：2021-02-10

自从 2010 年 systemd 发布以来，该项目一直在不断地增加新功能和扩展性能。代码行数超过了 130 万行，其中 Lennart Poettering 刚刚将其个人 systemd-homed git 树合并到 systemd 中，添加了另外 2 万行代码，而持续存在的开放问题数约为 1400 个，新问题和漏洞不断涌现。因此，systemd 应被视为实验性的，不适合在任何地方运行。

即使像 OpenBSD 开发者这样细致且注重安全的编码者，代码审查非常严格，[错误和安全问题仍然时有发生](https://www.openbsd.org/errata.html)。

考虑到 systemd 的开发方式，代码甚至没有经过（据我所知）单一的代码审查，而且开发者不断增加新功能而不是专注于解决 [漏洞](https://github.com/systemd/systemd/issues?q=is%3Aissue+is%3Aopen+sort%3Acreated-asc+label%3A%22bug+%F0%9F%90%9B%22)、安全和稳定性，这个项目不适合出现在任何 GNU/Linux 发行版中，除非是“测试阶段”。

即使是 Debian GNU/Linux，在其“testing”和“unstable”版本分支中进行了大量的测试，也不适用于生产系统，因为 systemd 仍然有许多可追溯至 2015 年的漏洞。

添加一个与现有代码库非常匹配的简单功能是一回事，但合并具有成千上万行代码的项目则完全不同。这等同于添加了一个全新的应用程序。

在 Lennart Poettering 于 2013 年 1 月的博客文章 [《The Biggest Myths》](http://0pointer.de/blog/projects/the-biggest-myths.html)（最大的误会）中，他试图反驳将 systemd 称为“单体化”（monolith）的说法，而许多人都认为它就是。Lennart 表示：

>一个包含 69 个单独二进制文件的软件包很难称为单体化。然而，与以前的解决方案不同的是，我们将更多组件捆绑在一个单一的压缩包中，并在单个存储库中维护它们，具有统一的发布周期。

然而，问题是，许多这些所谓的单独二进制文件在没有其他 systemd 组件的情况下将无法工作。只需举一个例子，如果查看 [systemd-networkd](https://www.freedesktop.org/software/systemd/man/systemd.network.html) 的 man 页面，明确说明如果将 UseDNS 选项定义为 `true`，*则将使用从 DHCP 服务器接收到的 DNS 服务器，并优先于任何静态配置的 DNS 服务器。这对应于 `resolv.conf` 中的 `nameserver` 选项。* 但它忽略了这个设置（和多个其他设置）在没有 systemd-resolved 的情况下是无效的。systemd 的其他组件也同样紧密集成。

某些类型的应用程序非常难以按照 Unix 哲学设计和构建，例如现代网络浏览器、电子游戏等，但 systemd 显然并不属于这类应用程序。它在许多方面违反了 Unix 哲学，以至于很难跟上，这不是因为开发者需要这样做，而是因为他们想这样做，他们根本不在乎。虽然他们可以不在乎，并继续将更多功能添加到 systemd 中，但是现在一些大型 GNU/Linux 发行版是时候开始认真关注了。

我们所熟悉的 GNU/Linux 正慢慢变成一场灾难，一个充满安全问题的操作系统末日，等待爆发。这只是因为每个人都希望有一个新的 init 系统，但最终却得到了除了新内核以外的一切。现在唯一缺少的就是 systemd-linux。
