# 升级到 FreeBSD 13.2 的十大理由

- 原文地址：<https://freebsdfoundation.org/blog/top-ten-reasons-to-upgrade-to-freebsd-13-2/>
- 译者：ykla & ChatGPT

在软件升级方面，世界上看起来有两种类型的人：一类是当新版本发布就立刻升级的人，另一类是一直推迟升级直到绝对必要时才进行的人。说实话，我可能属于后一类，但是对于 FreeBSD 来说，升级到最新版本将为你提供更好的安全性，更多的功能和更加流畅的体验。FreeBSD 13.2 版本于四月份发布，对于那些还没有升级的用户，我们为你列出了十大升级理由。

1. 对数百个 Bug 的修复和改进。
2. 多个第三方软件和驱动程序的版本升级，包括 OpenSSH、libarchive、LLVM 和 Clang 编译器。
3. 针对 Intel 第 12 代和第 13 代混合型 CPU 的缺陷提供了解决方案——该错误可能导致 UFS 和 MSDOSFS 文件系统的损坏，以及可能的其他内存损坏。慢速核心（E-cores）自动使用更慢的页面无效化方法来解决此问题。
4. 在默认情况下启用了 64 位可执行文件的地址空间布局随机化（ASLR），提供了更高的安全性。
5. WireGuard VPN 驱动程序（wg(4)）已重新集成到内核，可提供更快且更安全的 VPN 体验。
6. 内核 TLS（KTLS）已添加对 TLS 1.3 的接收卸载支持。
7. 更新了 Intel WiFi 驱动程序，并添加了 Realtek WiFi 的驱动程序。
8. 现在可以在启用日志软更新的 UFS 文件系统上进行快照，实现后台备份。
9. FreeBSD 虚拟化管理程序 bhyve 支持更多虚拟 CPU。
10. 为具有大型路由表的系统添加了 DPDK 路由模块。

当然，这只是一个有关十大升级理由的列表。FreeBSD 的美妙之处在于你可以根据自己的需求进行自定义。确保查看完整的 [发行说明](https://www.freebsd.org/releases/13.2R/relnotes/)，了解还有哪些更新可以让你的生活更加便利。请记住，拖延症“专家”从未让任何人取得进步。立刻 [升级](https://www.freebsd.org/where/) 到 13.2 版本吧！
