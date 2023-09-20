# 第二次企业工作组会议回顾

- 原文：[Recap of Second Enterprise Working Group Meeting](https://freebsdfoundation.org/blog/recap-of-second-enterprise-working-group-meeting/)
- 发布日期：2023.9.18
- 译者：ykla & ChatGPT

- **会议日期:** 2023 年 9 月 4 日当周
- **会议录像链接:** [Meeting Recording](https://youtu.be/rCRi57tdHns)

TL;DR 以下表格总结了优先项目的状态。蓝色行似乎是我们可以立即采取行动的项目：

| 项目                                | 明白下一步怎么做 | 项目负责人           | 用户/测试者       | 开发者         |
| ----------------------------------- | :----------: | :--------------------: | :----------------: | :--------------: |
| **云原生 / OCI 运行时**     | **是**         | **Greg W**      | **是**               | **是**             |
| AD / DNS                            | 是         | Michael O            | 是               | 否             |
| NVIDIA GPU                          | 是         | Greg W               | 是               | 否             |
| **Samba**              | **是**         | **Greg W**              | **是**               | **是**            |
| Definitive manager bhyve/VM/Jail   | 是         | ?                    | 是               | 否             |
| Zero Trust Build                    | 是         | FreeBSD 基金会   | 是               | 否             |
| Ports Automation                    | 否         | FreeBSD 基金会   | 是               | 否             |
| **Enterprise CA**                    | **是**         | **Greg W**               | **是**         | **是**     |


鉴于这个工作组在“Zero Trust Builds”和“Ports Automation”项目上可能取得的进展有限，我们建议将它们放到一旁，而不是将更好的证书机构支持纳入范围，因为社区似乎可以采取明确的下一步行动。

我们回顾了功能和基础设施优先级和难度调查的结果，并开始制定下一步计划，以解决最高优先级的问题。

以下是工作组为通用企业调查用例的功能和基础设施需求优先级的排名：

![image](https://github.com/FreeBSD-Ask/Translated-articles/assets/10327999/3d5b6a72-49f5-43f3-bc7d-0d54cf0b8aed)
![image](https://github.com/FreeBSD-Ask/Translated-articles/assets/10327999/21d03053-e490-483d-903e-26e64e9ebf28)


然后，我们逐个讨论了每个类别中的前三个问题以及如何继续。

### Cloud Native / OCI runtime

这被认为是最重要的缺失部分，差距很大。令人高兴的是，已经有正在进行的工作。

以下是我所了解到的现有项目：


| 项目               | 说明                 | 链接                                 |
|:----------------------:| --------------------------------------- |:---------------------------------------------:|
| Planternetes                         | 将 K8s v1.28.0 移植到 illumos，并为 FreeBSD 和 OpenBSD 编译了二进制文件。                                                                                                                                                                                  | [链接](https://medium.com/@norlin.t/by-the-way-planternetes-kubernetes-v1-28-0-for-illumos-freebsd-and-openbsd-5d57026d6a25) |
| runj                                 | 一个新的实验性、概念验证的 OCI 兼容运行时，用于 FreeBSD jail。                                                                                                                                                                                            | [链接](https://samuel.karp.dev/blog/2021/03/runj-a-new-oci-runtime-for-freebsd-jails/)                                       |
| ocijail                              | 实验性、概念验证的 OCI 兼容运行时，用于 jail。                                                                                                                                                                                                            | [链接](https://www.freshports.org/sysutils/ocijail/)                                                                         |
| Podman port to FreeBSD               | 在 FreeBSD 上使用 Podman 来管理 Pod、容器和容器镜像。                                                                                                                                                                                                     | [链接](https://www.freshports.org/sysutils/podman/)                                                                          |
| FreeBSD containers on macOS          | Podman 支持在流行的客户端操作系统（macOS、Windows）上运行 Linux 容器，以便人们可以在这些系统上工作并部署到云中。它通过生成一个 Linux 虚拟机并从外部连接到它来实现这一点。该项目旨在为 FreeBSD 服务器带来与 Linux 服务器用户相同的 podman 桌面/笔记本体验。 | [链接](https://www.linkedin.com/pulse/freebsd-containers-macos-david-chisnall/)                                              |
| XC                                   | 用于 FreeBSD 的正在进行中的容器引擎，可运行 Linux 和 FreeBSD 容器。它可以支持 OCI 兼容的镜像注册表，如 DockerHub 或 Azure 容器注册表，用于镜像分发。                                                                                                       | [链接](https://github.com/michael-yuji/xc)                                                                                   |
| 在 Kubelet 上加入基本的 FreeBSD 支持 | 用于在 FreeBSD 上启动 kubelet 并与 CRI 运行时（在我这里是 containerd+runj）通信以运行（目前是）FreeBSD 容器的第一个基础构件。                                                                                                                              | [链接](https://github.com/kubernetes/kubernetes/pull/115870)                                                                 |                                                                       |


## 接下来的步骤：

- 我们同意完成一个非常轻量级的 PRD（产品需求文档），以便我们就最终目标达成一致。
  - 需要志愿者。
- 开发测试和文档协议，并根据 PRD 评估上述列出的方法。
  - 需要 3 名志愿者或更多。
- 确定哪种方法最适合要求，然后确定如何将其推向生产就绪状态。
  - 需要 2 名志愿者。
- 我已经创建了一个 Wrike 项目，包括上述三项任务，并将邀请志愿者加入。

### AD / DNS 集成

正在寻找志愿者协助。下一步是进行电话会议，了解现有状态和所需工作。

### NVIDIA GPU / CUDA

这个问题引发了一些很好的讨论。在 FreeBSD 中需要支持这些芯片和驱动程序，以用于高性能计算和人工智能工作负载。这些市场的许多用户喜欢 FreeBSD，因为它具有安全性、稳定性和性能，但需要更多的 NVIDIA 本机支持。讨论的一个使用模型是在私有数据上训练模型，然后在 FreeBSD 上托管，以获得诸如 ZFS、安全性等功能的好处。保护数据模型至关重要，并且在 FreeBSD 上托管是实现这一目标的好方法。

我提供了两个链接，以提供特性请求给 Nvidia：

- [Will CUDA work with FreeBSD?](https://forums.developer.nvidia.com/t/will-cuda-work-with-freebsd/926/4)
- [UDA and /(nv_(un|)register|os_(un|)lock)_user_pages/](https://forums.developer.nvidia.com/t/cuda-and-nv-un-register-os-un-lock-user-pages/174678)

如果社区可以持续请求对 FreeBSD 的支持，希望能够得到 NVIDIA 的一些帮助。

工作组中的其他两名成员表示愿意提供帮助。感谢他们！

- Zoran，来自 Supermicro，可以测试并提供错误以帮助支持驱动程序开发。
- Jason 可以在 Azure 上提供帮助。
- 我将有机会在下周与 NVIDIA 的一些人交谈，并将提出这个讨论并报告回应。

### Samba

在上周早些时候，我们与一位有着与 Samba 团队合作经验的开发者进行了交谈。他们将通过电子邮件联系 Samba 团队，并提供帮助将补丁（回馈给）上游。

Samba 团队和其他项目要求的一个事项是使用 FreeBSD 进行 GitHub Actions，以帮助自动化工作。FreeBSD 基金会将联系 GitHub 和 Microsoft 的人，尝试将这个功能引入。

我已经创建了一个 Wrike 项目，包括上述三项任务，并将邀请志愿者加入。

### bhyve/VM/Jail 的 Definitive manager

现在有 5-6 个 VM 管理器，大部分相同。

工作组似乎同意最好的下一步是组织一个电话会议或某种讨论，与各种工具的维护者讨论是否可以通过整合来提高效率。

通过这个过程，希望有一个默认的配置。

正在寻找志愿者协助。

### 企业 CA（推进的原因是似乎需要的工作量较小，而“Zero Trust Builds”和“Ports Automation”目前对工作组无效）

与 AD/DNS 类似，这里的下一步是进行快速电话会议——查看目前的情况，还需要做什么，需要有人可以提交补丁。

Greg 将采取行动，与 Joe/Ed/其他人一起确定谁是最适合提供帮助的人。

这是第二次企业工作组会议的回顾摘要，详细信息可在会议录像中找到。
