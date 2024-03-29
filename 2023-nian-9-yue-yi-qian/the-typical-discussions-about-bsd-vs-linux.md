# 关于 BSD 与 Linux 的典型讨论

- 原地址：<https://unixsheikh.com/articles/the-typical-discussions-about-bsd-vs-linux.html>
- 译者：ykla & ChatGPT
- 发布日期：2017-10-15

**在这篇文章中，我们将简要讨论关于流行操作系统（如 BSD 和 Linux）之间差异。**

如果你对操作系统有所兴趣，很可能会遇到人们询问类似于“我应该使用 FreeBSD 还是 Ubuntu？”或者“我应该使用 Arch Linux 还是 Debian？”这种问题。你可能还见过有人对他人从一个操作系统迁移到另一个操作系统而感到兴奋。

这些问题和行为是没有意义的。让我用一个例子来解释，这样你会立即明白其中的道理。

“我应该用拖拉机还是汽车？”或者“我应该乘火车还是坐飞机？”

每种交通工具都是为了解决特定问题而开发的。操作系统也是如此。

因此，有人会问是否应该使用操作系统 X 而非 Y，是因为这个人还没有理解他的具体需求或所涉及操作系统之间的差异。而且大多数人实际上并不真正与操作系统打交道，而是更多地与软件包管理器或应用程序打交道。

没有人能够告诉你应该使用什么，这完全取决于你自己的具体情况和需求。当你面临一个具体问题时，你需要考虑用哪个系统最适合解决这个问题，并且当你有多个选择时，你可能需要研究每个系统之间的差异。在某些情况下，你在“拖拉机”和“汽车”之间做选择，但在其他情况下，你在考虑“汽车品牌”和“汽车品牌”之间的选择，而你知道某些品牌和型号非常相似。

所有开源操作系统项目都很优秀，它们各自专注于不同的领域。当有人从某个 Linux 发行版迁移到某个 BSD 版本或反之时，感到高兴是没有意义的。而当有人从像 Microsoft Windows 或 Apple 的 OS-X 这样的专有系统迁移时，感到高兴是有意义的，因为这些系统不仅损害了自由，而且还包含一些有问题的内容。

几乎所有不同的开源项目可以相互帮助、友好合作，终端用户应该只从技术角度而不是个人喜好角度讨论这些问题。当然，进行简单的闲聊和分享个人喜好没有问题，但实际情况并非如此。

当有人说：“我们一直在我们的设备上运行 Linux，但它们最终崩溃了。然后我们决定在它们上面安装 FreeBSD，它们的性能提高了一倍，而且依然表现得很好！”这样的陈述对于他们一开始使用的具体操作系统没有提供任何信息。实际上，这表明这些人很可能一开始对自己在做什么一无所知。

当你理解了自己的具体需求时，你就会寻找解决问题的最佳工具，而在某些情况下，这需要测试和试错的方法。

在某些情况下，你面对的是非常普通的问题，比如需要阅读电子邮件、浏览互联网，并且偶尔写一封信。这种情况可以类比为一个非常普通的运输问题，你只需将一个小箱子运输到相对较近的地方。几乎任何一种运输方式都可以解决你的问题，和你使用什么方式几乎没有关系。嗯，几乎没有关系，因为在这种情况下使用大卡车或飞机是没有意义的。就像使用一个没有二进制软件包，而需要从头编译所有东西的通用目的操作系统一样，是没有意义的。而且，也许你真的喜欢因为不需要将不需要的东西编译到你的应用程序中而获得的微小的速度提升？也许你宁愿开保时捷或法拉利而不是步行，即使你只需要走几米就能到达目的地？谁知道呢，对吧？

当你面对大多数通用情况时，主要取决于个人品味和偏好，而不是技术要求。几乎任何操作系统都可以很好地满足你的需求，但也有一些例外情况（主要是硬件相关的限制）。但是，当你需要调整编译时间选项，因为你需要定制解决方案时，你面对的不是通用问题，而是一个非常具体的问题。

你需要问自己一些问题，比如：

- 你使用了什么硬件？
- 你需要从内核中得到什么？
- 你需要哪种类型的驱动程序？
- 你希望如何安装第三方软件？
- 你需要能够更改编译时间的选项吗？
- 你需要多少系统控制权或喜欢拥有多少控制权？
- 你需要多高的系统安全性？
- 你更喜欢图形界面应用程序还是命令行界面应用程序？
- 这是一个用于重要事务的设备，还是一个用于娱乐的设备？

一些操作系统经过精心设计和详细考虑，比如 OpenBSD，而其他系统则几乎是没有计划地“有机地”拼凑在一起，这极大地影响了安全性、性能和控制权。

举个例子，多年来，FreeBSD 在通信行业一直是首选的操作系统，因为它的网络堆栈经过非常谨慎的设计，而 Linux 则因其网络堆栈混乱而受到指责。当然，这影响了所有运行相同内核的 Linux 发行版。情况已经改变，但了解底层代码的设计方式也有助于理解哪个系统最适合满足你的具体需求。

当这些因素都不重要时，你几乎可以自由选择所有东西。
