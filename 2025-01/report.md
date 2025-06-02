# FreeBSD 并没有死，别听信那些夸张的说法

- 原文：[The Report of My Death Was an Exaggeration](https://freebsdfoundation.org/blog/the-report-of-my-death-was-an-exaggeration/)
- 2025-5-20


我来到基金会已经大约三个月了。在这段时间里，我和一些正在使用 FreeBSD 的企业、以及一些有意愿使用 FreeBSD 的企业进行过几次交谈。在不止一次的场合中，有个说法让我不禁挑眉——因为这既不是我所见，也不是我所感受到的。那句话是：“FreeBSD 不是快死了吗？”

等等！什么？你怎么会这么想？

在我仔细拆解这个问题的过程中，我得出了一个惊人的结论，以及一个显而易见的事实。在我们一起踏上这段旅程之前，我先简明地回答这个问题：**不，FreeBSD 没有死，甚至连濒死都谈不上。** 我为什么如此肯定？请往下看……

![别相信搜索引擎！](https://freebsdfoundation.org/wp-content/uploads/2025/05/google-search-is-dead-1024x538.png)

**谷歌搜索正在往你脑子里塞想法**

我们从一些数字说起，或者至少，从谷歌的视角看一些数字。开始输入一个搜索词时，谷歌的自动补全建议就会浮现出一幅图景。你看第七个建议是什么？所以说，谷歌已经开始在你脑海中植入想法了。然而如果我们深入挖掘……

![](https://freebsdfoundation.org/wp-content/uploads/2025/05/search-trend-up-1024x538.png)

**这条线正在上升**

……很清楚地可以看到，这是稳定的上升趋势。我知道你在想什么：“统计数字不过是乐观的真相！”确实，通过谷歌趋势几乎可以构造出任何故事。我之所以选择这个时间范围，是因为当我查看更广的时间线时，谷歌提示他们更改了算法，意味着结果并不完全可比。在上一次算法更改之后，这大概是我能找到的最合适的时间段了。

当然，单独看这一张图表意义不大，那我们来做个对比……

![](https://freebsdfoundation.org/wp-content/uploads/2025/05/search-linux-1024x538.png)

**完全平坦的一张图**

在相同时间段内，“Linux”的搜索量显示出的是平稳趋势。虽然绝对数字可能大不一样，但趋势很明显：Linux 是一条平线，而 FreeBSD 呈缓慢上升趋势。

## 究竟发生了什么？

我有一个假设，解释为什么人们 **认为** FreeBSD 正在衰亡，以及引言中提到的那个令人吃惊的结论。

最近，万能的 YouTube 算法给我推送了一段关于 [Cal Newport](https://youtu.be/v520wFzpAd0?si=QksXOGcX664LTL1f) 的简短访谈。我读过他的一本书，也听过其他访谈，所以自然引起了我的兴趣。在这段视频中，他谈到了他所谓的“伪生产力”——基本上是用工厂式的生产力标准来衡量办公室或知识工作。这导致了大量无意义的忙碌，人们试图看起来很忙碌，却没有做真正有意义的工作。我认为 FreeBSD 的问题正好是这种情况的相反方向。因为它不被频繁提及，人们就认为它在衰亡。这是一个典型的[可得性启发](https://en.wikipedia.org/wiki/Availability_heuristic) 的例子。

那么一个显而易见的问题是，为什么人们不怎么谈论 FreeBSD 呢？

## 令人吃惊的结论

可得性启发是一种迷人的心理捷径。它解释了为什么一些产品名称会变成动词和家喻户晓的词汇。比如“谷歌”（搜索）、“Hoover”（吸尘器）、“Zoom”（视频会议）。这些词达到了某个临界点，人们不再需要多想，直接“谷歌”或者“zoom”。

如今，构建互联网服务时很少会考虑底层系统。随着容器和云平台的普及，开发已经远离了硬件层面。操作系统不再是人们关注的焦点，因此大家默认使用熟悉的东西，而当他们想到操作系统时，通常是 Linux。

然而 FreeBSD 就在那里，[默默支撑着互联网的大量服务](https://www.theregister.com/2025/04/28/freebsd_foundation_25/)，一点声响也没有。使用它的公司呢？他们也不怎么宣传。为什么？因为他们没必要。让我恍然大悟的是，FreeBSD 既是我们所有人的礼物，同时也是它自己的阿喀琉斯之踵，那就是它的[许可证](https://www.freebsd.org/copyright/freebsd-license/)。

不同于 GPL 许可证要求你必须共享衍生作品，BSD 许可证并没有这种要求。你可以拿 FreeBSD 代码，进行开发，然后完全不用回馈任何东西。这使得 FreeBSD 成为构建产品的绝佳基础——但同时也意味着公司几乎没有理由去贡献回馈。

## 来聊聊吧

由于厂商相信 FreeBSD 正在衰亡这一错误观念，FreeBSD 的硬件支持有时比 Linux 更加棘手。但我们只需看看最近在 [Wi-Fi 支持](https://youtu.be/Uic0ksaqOwE) 方面取得的惊人成果，就能明白采用 [Linux 驱动](https://wiki.freebsd.org/LinuxKPI) 是一项切实可行的解决方案。

因此，我们想向使用 FreeBSD 的公司发出呼吁。**请跟我们分享你们的使用场景**。如果你们在硬件支持方面遇到困难，也请告诉我们。哪怕你们的法务团队不希望公开谈论这些使用情况，我们仍然[希望听到你们的声音](https://freebsdfoundation.org/about-us/contact-us/)。这些对话可以只在我们之间进行 😀，但至少我们能够了解 FreeBSD 的使用案例和存在的问题。我们，FreeBSD 基金会，可以成为工业界与软件和硬件厂商之间的纽带。

与此同时，请[持续关注本博客](https://freebsdfoundation.org/feed/)和[我们的 YouTube 频道](https://youtube.com/@freebsdproject?si=Ly9T5JbCjjKfQo_s)。我们即将推出一些精彩内容，介绍基于 FreeBSD 构建的解决方案，并展示适合日常使用的现代笔记本电脑。
