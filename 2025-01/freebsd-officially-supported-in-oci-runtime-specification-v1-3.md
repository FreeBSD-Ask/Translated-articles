# FreeBSD 在 OCI 运行时规范 v1.3 中获得正式支持

- 原文：[FreeBSD Officially Supported in OCI Runtime Specification v1.3](https://freebsdfoundation.org/blog/freebsd-officially-supported-in-oci-runtime-specification-v1-3/)
- 发布日期：2025 年 11 月 4 日

FreeBSD 已被纳入 [Open Container Initiative（OCI）运行时规范](https://github.com/opencontainers/runtime-spec/releases/tag/v1.3.0) 的 1.3 版本中，成为官方支持的平台。该版本于 2025 年 11 月 4 日发布。这一里程碑标志着 FreeBSD 的重大成就，体现了社区志愿者多年来为将云原生容器技术引入该平台所做的不懈努力。

## FreeBSD 的重大进步

FreeBSD 纳入了 OCI 运行时规范，是个分水岭式的时刻，使 FreeBSD 成为现代云原生工作负载的一级平台。官方的 OCI 支持意味着 FreeBSD 用户现在可以放心地利用完整的容器工具和编排平台生态系统，因为他们使用的是标准化、厂商无关的规范。对于已经在生产环境中运行 FreeBSD 的组织来说，这为采用符合行业标准的容器化应用部署策略打开了大门，使 FreeBSD 成为云基础设施、边缘计算和企业部署的更具吸引力的选择。

## 基于 FreeBSD 虚拟化传统的扩展

新增的云原生容器支持，进一步强化了 FreeBSD 已经非常成熟的虚拟化能力，尤其是强大的 FreeBSD jail 技术——它已成为 FreeBSD 二十多年来的基石。实际上，FreeBSD 上的 OCI 容器是通过 jail 作为底层隔离机制实现的，将 jail 的安全性和资源管理优势与符合 OCI 标准的容器可移植性和生态系统优势结合在一起。这种技术结合为 FreeBSD 用户带来了两全其美的体验：既有 jail 所具备的轻量、安全隔离，又拥有容器在现代软件开发中广泛普及的标准化工具链和镜像格式。

## 凝聚多年努力的社区成果

这一成就凝聚了由志愿者 Doug Rabson 领导的长期开发工作，并得到了来自 OCI 社区、FreeBSD 项目以及 FreeBSD 基金会众多贡献者的支持与合作。实现官方 OCI 支持的历程包括以下几个关键里程碑：

* 2021 年：Samuel Karp 发布了 *runj*，这是 FreeBSD 的首个 OCI 运行时，并支持 *containerd*，证明了在该平台上运行 OCI 容器的可行性。
* 2022 年：Doug Rabson 为 Buildah 和 Podman 添加了 FreeBSD 支持，这两个关键的容器管理工具的改进要求对 FreeBSD 内核进行重要修改，以满足 OCI 运行时规范的要求，为今天的正式认可奠定了基础。
* 2024 年：Dave Cottlehuber 领导将 FreeBSD 官方 OCI 镜像纳入的工作，首次出现在 FreeBSD 14.2 中，这些镜像可在 Docker Hub 和 GitHub Container Registry 上获取。
* 2025 年：Doug Rabson 领导将 FreeBSD 作为平台添加到 OCI 运行时规范中的工作，该支持在 v1.3 版本中正式提供。

FreeBSD 基金会借此机会感谢 Samuel Karp 与 Doug Rabson 在推动此项计划中所付出的不懈努力，同时向所有为使 FreeBSD 成为完全受支持的 OCI 平台而作出贡献的人员致以诚挚谢意。

## 展望将来

随着官方 OCI 运行时规范支持的确立，FreeBSD 比以往任何时候都更有能力满足现代基础设施与应用开发的需求。我们期待看到社区如何利用这一能力，在 FreeBSD 上构建创新解决方案。

确保最新动态：

* [加入 FreeBSD jail 邮件列表，获取云原生容器相关更新](https://lists.freebsd.org/subscription/freebsd-jail)
* [加入工作组，为 FreeBSD 云原生容器技术的开发作出贡献](https://wiki.freebsd.org/Containers#Working_Groups)
