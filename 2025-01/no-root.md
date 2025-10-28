# FreeBSD 已实现可重现构建并且无需 root 权限

- [FreeBSD now builds reproducibly and without root privilege](https://freebsdfoundation.org/blog/freebsd-now-builds-reproducibly-and-without-root-privilege/)
- 2025 年 10 月 27 日

FreeBSD 基金会很高兴地宣布，FreeBSD 现已完成无需 root 权限即可进行构建的工作。我们已经为所有源代码发布构建实现了无 root 基础设施支持，彻底消除了在 FreeBSD 发布流水线中使用 root 权限的需求。此项工作是 [由 Sovereign Tech Agency 委托的项目](https://freebsdfoundation.org/blog/sovereign-tech-fund-to-invest-e686400-in-freebsd-infrastructure-modernization/) 的一部分。

这些更改目前已在 FreeBSD 开发分支中提供，并在可能的情况下被合并到 FreeBSD 15.0 的发布分支中。

现在，在构建 FreeBSD 发布产物时，无需 root 权限即可创建设备文件、设置正确的文件所有权或挂载文件系统。这一改进提升了安全性，并简化了自动化构建过程。

现在，每个 FreeBSD 发布产物都可以在无 root 权限的情况下完成构建：

* 适用于 USB 闪存盘与 CD/DVD 安装介质的双模式 ISO 镜像
* 适用于可引导 USB 驱动器的 Memstick 镜像
* 适用于虚拟机部署的 VM 镜像
* 适用于 AWS、Azure 等云平台的云磁盘镜像

消除构建流水线中对 root 权限的依赖，减少了攻击面和权限提升的风险。这使得构建环境更加安全与灵活，无论是官方基础设施还是社区贡献者都能受益。

**可重现构建**

与无 root 工作并行，FreeBSD 还引入了多项改进以提升构建的可重现性——确保相同的源代码输入始终能生成逐字节完全一致的二进制输出。这些更改涵盖了操作系统本身、发布工具链以及整个构建过程。

主要改进包括：

* 消除或标准化时间戳
* 文件列表、包元数据等内容的稳定排序
* 一致的构建环境，包括调试路径与区域设置
* 在诸如文件系统镜像创建工具 mkimg(1) 等构建工具中实现可重现产物支持

可重现构建增强了整个软件供应链的完整性与透明度。它能够实现可验证的信任、改进调试与审计流程、简化持续集成，并支持长期可维护性。

现在，FreeBSD 的持续集成系统与自动化构建基础设施可以在无特权容器或受限环境中运行。贡献者们也能够在本地系统上无需提升权限即可完整构建 FreeBSD 发布版本。

FreeBSD 现已能够在无需 root 权限的情况下安全、可重现地完成构建。它更快、更安全、更透明——让任何人、在任何地方，都能以信任的方式构建 FreeBSD。
