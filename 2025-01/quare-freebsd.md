# 为什么是 FreeBSD？

- 原文：[Quare FreeBSD?](https://vermaden.wordpress.com/2020/09/07/quare-freebsd/)
- 作者：𝚟𝚎𝚛𝚖𝚊𝚍𝚎𝚗
- 2020/09/07

我本来真的想把这篇文章写得不那么长……但我彻底失败了。但我至少尽力把它组织得井井有条，这样读者在略读之后可以回过头来复习，因为它并不是一堂速成课。我本想将它命名为 *Why FreeBSD?*，但当你在自己喜欢的 搜索引擎 **duck.com**中输入这个标题时，会出现很多类似的文章。我希望它有个独特且与众不同的名字，于是我使用了拉丁语中的“为什么”——*quare*。

<img width="650" height="650" alt="image" src="https://github.com/user-attachments/assets/d40aa447-366c-4646-978a-90cd8d8facdb" />

FreeBSD 能为你提供哪些在其他操作系统无法提供的东西？在我使用过的所有操作系统中，我发现 FreeBSD 的缺点最少。这篇文章并不是为了说服你使用和尝试 FreeBSD——这需要你自己去体验。本文将展示 FreeBSD 的价值所在，以及为何它是其他操作系统的更好替代方案，并且 FreeBSD 绝对没有走向大败局。

## 基本系统

当你安装 Linux 系统时，它只是一些 RPM 或 DEB 软件包的集合。例如，如果你安装 CentOS 7.8 的 *Minimal* 版本，最终会安装几百个 RPM 软件包。经过一周乃至一个月，其中许多软件包会收到更新，有时会导致这个 CentOS 系统无法使用，甚至无法启动（例如最近的 GRUB *Boothole* 问题）。而 FreeBSD 提供了 **基本系统** 的概念。这意味着当你安装 FreeBSD 时，你安装的是完整的最小系统。没有需要单独更新的软件包或子系统。只是完整的 **基本系统**。这意味着 **/boot /bin /sbin /usr /etc /lib /libexec /rescue** 目录不受其他软件包影响。当你决定安装软件包（或使用 *FreeBSD Ports* 构建它们）时，它们都会安装在路径 **/usr/local** 下。这意味着配置文件位于 **/usr/local/etc**，可执行文件位于 **/usr/local/bin** 和 **/usr/local/sbin**，库文件位于 **/usr/local/lib** 和 **/usr/local/libexec**，依此类推。FreeBSD **基本系统** 的内核模块与内核一起存放在目录 **/boot/kernel**。为了保持整洁，由软件包提供的所有内核模块都会放在 **/boot/modules** 目录中。所有内容都有其位置并且互不干扰。

这就是 **基本系统** 二进制文件（位于 **/bin /sbin /usr/bin /usr/sbin** 目录）与由 **pkg(8)** 维护的第三方软件包（位于 **/usr/local/bin** 和 **/usr/local/sbin** 目录）之间的分离。我们都知道 **bin**（普通用户）与 **sbin**（root）二进制文件的区别，但在 FreeBSD 中还有另一种与 UFS 相关的分离。当 FreeBSD 世界中只有 UFS 文件系统时，**/bin** 和 **/sbin** 二进制文件在根文件系统（**/**）挂载后、**/usr** 文件系统挂载前即可使用——这是个历史性的（在 UFS 配置中仍然有用）区分，可追溯到早期 UNIX 时代。在 ZFS 配置中，这不再重要，因为所有文件都在 ZFS 池中。

下面是 **基本系统** 的 **/** 目录示意图，以及可选的第三方应用程序目录 **/usr/local**。

<img width="850" height="750" alt="image" src="https://github.com/user-attachments/assets/4c8b3846-9613-43d8-97aa-2e205a64336c" />

FreeBSD **基本系统** 的分离还有另一个好处——如果某个软件包产生“伟大”的想法，去安装新的 **cc** 编译器并覆盖默认系统编译器，或者以某种方式添加库和头文件，使得系统恢复工作变得非常困难，你也不必担心。如果某个随机的 FreeBSD 软件包将 **libc.so** 添加到 **/usr/local/lib** 目录，你仍然可以正常运行程序，因为 FreeBSD 系统二进制文件是链接到 **/usr/lib** 下的内容。这也是 UNIX 系统（包括 FreeBSD）中存在 **PATH** 变量的原因，用于设置首先搜索哪些目录的二进制文件。在 FreeBSD 中，默认设置是先搜索 **基本系统** 的二进制文件目录，然后再搜索第三方软件包目录。

你可以使用命令 **freebsd-update(8)** 在 RELEASE 系统下单独更新（或不更新）**基本系统**，或者在 STABLE/CURRENT 系统下通过 **make buildworld** 和 **make installworld** 命令重新编译更新。而软件包可以通过 **pkg(8)** 工具更新，或者在 **/usr/ports** 目录下使用 **portmaster** 从 *FreeBSD Ports* 树构建更新。这意味着软件包的更新不会影响你的 FreeBSD **基本系统**。例如，当你搞砸了已编译的 ports 和软件包（我在 FreeBSD 初期就这样做过），如果你想重新开始，你只需要删除 **/usr/local**、**/boot/modules** 和 **/var/db/pkg** 目录。就这样，你就回到了 **基本系统**，可以重新开始。这在 Linux 系统中是无法做到的。即使是很多概念基于 FreeBSD 的 Gentoo，也没有 **基本系统** 的特性。

**基本系统** 还有个额外的优点。由于它与软件包分离，没有人阻止你在 2012 年的 FreeBSD 9.0 上运行最新的 Firefox 80 或 LibreOffice 7.0。而你却无法在 2012 年的 Ubuntu 上安装最新的 Firefox……

有人可能担心，这样独立于安装软件包的 **基本系统** 会占用更多空间，但事实完全相反。Fresh 安装的 FreeBSD 12.1 系统占用不到 1 GB 磁盘空间，运行 **sshd(8)** 时使用不到 75 MB 内存。相比之下，Fresh 安装的 CentOS 7.8（选择 Minimal 集）占用 1.1 GB 磁盘空间，运行 **sshd(8)** 时使用超过 100 MB 内存。这样的 CentOS 系统非常精简，实际使用还需要安装更多软件包，而 FreeBSD 依靠 **基本系统** 本身就更强大且功能丰富，例如自带最新版本的 LLVM/CLANG 编译器套件。

从 FreeBSD 15.0 开始，可以使用 PKGBASE 特性安装 FreeBSD，而不是传统的发行版集合——这允许 **pkg(8)** 命令管理 FreeBSD **基本系统** 的软件包集。这与 Linux 系统不同，因为在 Linux 上所有内容都在一个平面仓库中。而 FreeBSD 使用 PKGBASE 仍然保留软件包集、核心软件包，并为 **基本系统** 提供独立仓库。如果 FreeBSD 与 Linux 相同，所有软件包——包括 **基本系统** 和第三方软件包——都在一个仓库中，没有 PKGBASE 软件包集，也没有 **基本系统** 核心软件包。而你仍然可以将 FreeBSD **基本系统** 软件包与第三方软件包分开升级，只需向 **pkg(8)** 命令指定 **-r REPOSITORY** 参数即可。

关于 **基本系统** 主题的更多资料：

* [https://www.over-yonder.net/~fullermd/rants/bsd4linux/03](https://www.over-yonder.net/~fullermd/rants/bsd4linux/03)
* [https://freebsd.org/handbook/dirstructure.html](https://freebsd.org/handbook/dirstructure.html)
* [https://man.freebsd.org/hier](https://man.freebsd.org/hier)
* [https://man.freebsd.org/freebsd-version](https://man.freebsd.org/freebsd-version)
* [https://man.freebsd.org/portmaster](https://man.freebsd.org/portmaster)

## ZFS 启动环境

我已经多次谈到过这一点，可能次数还不够，因为 Linux 世界仍然忽视这一“福音”。拥有 **ZFS 启动环境** 是一个彻底改变游戏规则的特性，一旦你发觉它的强大，你就再也不想使用不支持 ZFS 的系统。其核心思想是，你能在系统运行的任意时刻创建快照，如果出现问题，可重启回到该时刻（或快照）。这是升级或系统变更的完美解决方案。FreeBSD 系统本身已经很好地“防护”了更新软件包后可能产生的问题，但 **ZFS 启动环境** 将这种保护提升到了一个全新的水平。

![groundhog](https://vermaden.wordpress.com/wp-content/uploads/2020/09/groundhog.gif?w=960)

就像电影 [*土拨鼠之日 (1993)*](https://www.imdb.com/title/tt0107048/) 中的情节一样，借助 **ZFS 启动环境**，你将拥有无限次机会来把事情处理好。即使 **基本系统** 的更新和变更，也受其保护。你甚至可以使用 **zfs send** 和 **zfs recv** 命令将该 **启动环境** 传输到其他系统，或者在多个系统间分发它。你可以基于它创建 Jails 容器……或者在新的 **启动环境** 中安装 FreeBSD 新版本并重启进入，同时保留旧的“生产”系统不受影响。

关于 **ZFS 启动环境** 的更多资料：

* [https://is.gd/BECTL](https://is.gd/BECTL)
* [UFS 启动环境](https://vermaden.wordpress.com/2021/04/02/ufs-boot-environments/)
* [ARM 平台的 UFS 启动环境](https://vermaden.wordpress.com/2021/04/05/ufs-boot-environments-for-arm/)
* [使用 ZFS 启动环境升级 FreeBSD](https://vermaden.wordpress.com/2021/02/23/upgrade-freebsd-with-zfs-boot-environments/)
* [https://man.freebsd.org/beadm](https://man.freebsd.org/beadm)
* [https://man.freebsd.org/bectl](https://man.freebsd.org/bectl)

## 救援

当你把系统搞得一团糟，甚至 **基本系统** 概念或 **ZFS 启动环境** 功能都无法阻止你破坏 FreeBSD 安装时，还有一个更高级别的救援手段——**rescue 救援** 子系统。

![rescue](https://vermaden.wordpress.com/wp-content/uploads/2020/09/rescue.jpg?w=960)

你可以使用大约 150 个静态链接的二进制文件来执行 FreeBSD 安装的救援任务。你可能会想，这么多二进制文件是不是占用了大量空间……事实完全相反。它实际上是一个静态二进制文件，通过硬链接实现，并且只占用 11 MB 的磁盘空间。

```sh
FreeBSD # ls -lh /rescue | head -5
total 1118446
-r-xr-xr-x  146 root  wheel    11M 2020.02.19 21:10 [
-r-xr-xr-x  146 root  wheel    11M 2020.02.19 21:10 bectl
-r-xr-xr-x  146 root  wheel    11M 2020.02.19 21:10 bsdlabel
-r-xr-xr-x  146 root  wheel    11M 2020.02.19 21:10 bunzip2
```

**救援** 子系统甚至包含一些二进制文件，例如用于 **ZFS 启动环境** 管理的 **bectl(8)**，以及用于 ZFS 文件系统的 **zfs(8)** 和 **zpool(8)** 命令。以下是这些二进制文件的完整列表。

```sh
FreeBSD # ls /rescue
[           dd               fsck_ffs      init       mdmfs          ping      rtsol        unlink
bectl       devfs            fsck_msdosfs  ipf        mkdir          ping6     savecore     unlzma
bsdlabel    df               fsck_ufs      iscsictl   mknod          pkill     sed          unxz
bunzip2     dhclient         fsdb          iscsid     more           poweroff  setfacl      unzstd
bzcat       dhclient-script  fsirand       kenv       mount          ps        sh           vi
bzip2       disklabel        gbde          kill       mount_cd9660   pwd       shutdown     whoami
camcontrol  dmesg            geom          kldconfig  mount_msdosfs  rcorder   sleep        xz
cat         dump             getfacl       kldload    mount_nfs      rdump     spppcontrol  xzcat
ccdconfig   dumpfs           glabel        kldstat    mount_nullfs   realpath  stty         zcat
chflags     dumpon           gpart         kldunload  mount_udf      reboot    swapon       zdb
chgrp       echo             groups        ldconfig   mount_unionfs  red       sync         zfs
chio        ed               gunzip        less       mt             rescue    sysctl       zpool
chmod       ex               gzcat         link       mv             restore   tail         zstd
chown       expr             gzip          ln         nc             rm        tar          zstdcat
chroot      fastboot         halt          ls         newfs          rmdir     tcsh         zstdmt
clri        fasthalt         head          lzcat      newfs_msdos    route     tee          
cp          fdisk            hostname      lzma       nextboot       routed    test         
csh         fsck             id            md5        nos-tun        rrestore  tunefs       
date        fsck_4.2bsd      ifconfig      mdconfig   pgrep          rtquery   umount
```

关于 **rescue 救援** 主题的更多资料：

* [https://man.freebsd.org/rescue](https://man.freebsd.org/rescue)
* [https://docs.hetzner.com/robot/dedicated-server/operating-systems/freebsd-rescue-system/](https://docs.hetzner.com/robot/dedicated-server/operating-systems/freebsd-rescue-system/)
* [http://wiki.euserv.com/index.php/Manual_FreeBSD_Rescue_System/en](http://wiki.euserv.com/index.php/Manual_FreeBSD_Rescue_System/en)

## 音频

许多人可能不会期望 FreeBSD 在音频方面表现突出，但它在这方面已经表现卓越，并且这种优势不是最近才有，而是持续了数十年。记得 Linux 摒弃了旧的 OSS 子系统（单声道）并提出“伟大”的 ALSA 思路吗？我记得，因为那时我用 Linux。用“灾难”来形容当时的 Linux 音频栈都算礼貌……后来 PulseAudio 出现，整个 Linux 音频系统变得更烂。当时，由于只有 OSS 单通道而 ALSA 有许多通道，这意味着只有一个使用 OSS 后端的应用程序（例如 WINE）可以输出声音。如果另一个应用想用 OSS 发声，而 WINE 已经占用了唯一 OSS 通道，就会没有声音。而当时 ALSA 的实现非常糟糕，以至于 KDE 或 GNOME 各自创建了用户空间的声音守护进程来混合音频，这些守护进程之间又不兼容。这意味着如果你同时使用 KDE 和 GNOME 应用，可能 GNOME 应用有声音，而 KDE 应用没有，反之亦然。那真是 Linux 上的音频地狱。

![audio](https://vermaden.wordpress.com/wp-content/uploads/2020/09/audio.jpg?w=960)

回到 FreeBSD 音频，FreeBSD 提供了什么？惊人的 256 个 OSS 通道在内核中实时混音，实现低延迟。所有音频功能开箱即用——至今依然如此。你可以让 WINE 或 KDE/GNOME 的声音后端绑定到它们的 OSS 通道，同时 ALSA 应用也能正常使用声音设备。即使你插入一个 5.1 环绕音响系统，FreeBSD 也能开箱即用，应用程序能够立即使用。FreeBSD 的音频优势至今依然存在，而 Linux 中 PulseAudio 在用户空间混音虽然能用，但相比内核中的低延迟 FreeBSD 混音，会有较大延迟。

同事 [**meka**](https://twitter.com/meka_floss) 提到，FreeBSD 也是唯一拥有 **virtual_oss** 的操作系统，它能在用户空间进行混音/重采样/压缩，并能把蓝牙耳机和 USB 麦克风表示为单一声卡。

关于 **音频** 的更多资料：

* [https://freebsd.org/doc/en/books/arch-handbook/oss.html](https://freebsd.org/doc/en/books/arch-handbook/oss.html)
* [https://freebsd.org/handbook/sound-setup.html](https://freebsd.org/handbook/sound-setup.html)
* [https://man.freebsd.org/sound](https://man.freebsd.org/sound)
* [https://papers.freebsd.org/2019/fosdem/mekic-audio_studio/](https://papers.freebsd.org/2019/fosdem/mekic-audio_studio/)
* [https://wiki.freebsd.org/Sound](https://wiki.freebsd.org/Sound)

## Jail

**FreeBSD Jail** 是最早的 [操作系统级虚拟化](https://en.wikipedia.org/wiki/OS-level_virtualization) 实现之一，可追溯到 1999 年。即便是 Solaris 的 Zones/Containers，也要到 2004 年才问世，比 Jail 晚了五年。

![containers](https://vermaden.wordpress.com/wp-content/uploads/2020/09/containers.jpg?w=960)

在 Linux 引入 Docker 之后，*操作系统级虚拟化* 这一术语逐渐被 *容器*（Containers）取代，现在 **FreeBSD Jail** 与 Solaris Zones/Containers 被称为初代容器。但这种命名上的变化并没有降低 **FreeBSD Jail** 的强大功能。它们使用起来也非常简单。你只需要一个目录——例如 **/jail/nextcloud**——将所需版本的 FreeBSD **基本系统** 解压到该目录，例如从 12.1-RELEASE 获取 **[base.txz](http://ftp.freebsd.org/pub/FreeBSD/releases/amd64/12.1-RELEASE/base.txz)**，并在 **/etc/jail.conf** 文件中创建 Jail 配置，如下所示。

```sh
FreeBSD # mkdir -p /jail/nextcloud
FreeBSD # fetch -o - http://ftp.freebsd.org/pub/FreeBSD/releases/amd64/12.1-RELEASE/base.txz | tar --unlink -xpJf - -C /jail/nextcloud
FreeBSD # cat /etc/jail.conf
nextcloud {
  host.hostname = nextcloud.local;
  ip4.addr = 10.0.0.100;
  path = /jail/nextcloud;
}
```

现在你可以立即启动你的 Jail。

```bash
FreeBSD # service jail onestart nextcloud
Starting jails: nextcloud.
````

瞧！你的 FreeBSD Jail 已经跑起来了。

```bash
FreeBSD # jls
   JID  IP Address      Hostname                      Path
     1  10.0.0.100      nextcloud.local               /jail/nextcloud
```

当然，如果需要，你可以在 Jail 中使用精简版的 FreeBSD **基本系统**。ZFS 文件系统在这里也非常有用，因为使用 **zfs clone** 时，只有你的“基础” Jail 会占用空间，而从它创建的 Jails 只会记录你的修改。借助 FreeBSD 的另一个子系统——**Linux 二进制兼容层**，你还可以创建 Linux Jail，例如运行 Devuan 或 Ubuntu Jail。

**FreeBSD Jail** 也非常轻量。在单台 4 GB 内存的 FreeBSD 系统上，你可以启动并使用约 1000 个 FreeBSD Jail。

与纯 Docker 相比，它们也更容易调试和排错——更别提 Kubernetes，需要整支高技能团队来维护。

可以仅通过 **基本系统** 工具管理 **FreeBSD Jail**，如 **jls(8)**/**jexec(8)**，当然你也可以选择许多第三方 Jail 管理框架。在所有可用选项中，我会推荐 [**BastilleBSD**](https://bastillebsd.org/)，因为其现代化的方法和大量可用模板覆盖各种使用场景。

如果你想了解 Jail 的所有功能，可以参考文章 [FreeBSD Jail 容器](https://vermaden.wordpress.com/2023/06/28/freebsd-jails-containers/) 。

更多关于 **Jail** 的资料：

* [https://vermaden.wordpress.com/2023/06/28/freebsd-jails-containers/](https://vermaden.wordpress.com/2023/06/28/freebsd-jails-containers/)
* [https://freebsd.org/handbook/jails.html](https://freebsd.org/handbook/jails.html)
* [https://man.freebsd.org/jail](https://man.freebsd.org/jail)
* [https://man.freebsd.org/jail.conf](https://man.freebsd.org/jail.conf)
* [https://man.freebsd.org/jls](https://man.freebsd.org/jls)
* [https://man.freebsd.org/jexec](https://man.freebsd.org/jexec)
* [https://bastillebsd.org](https://bastillebsd.org/)
* [https://web.archive.org/web/20170126214625/http://ivoras.sharanet.org/blog/tree/2009-10-20.the-night-of-1000-jails.html](https://web.archive.org/web/20170126214625/http://ivoras.sharanet.org/blog/tree/2009-10-20.the-night-of-1000-jails.html)
* [https://forums.freebsd.org/threads/setting-up-a-debian-linux-jail-on-freebsd.68434/](https://forums.freebsd.org/threads/setting-up-a-debian-linux-jail-on-freebsd.68434/)

## Bhyve

FreeBSD——和 Linux 或其他受人尊敬的操作系统一样——自带其自研的虚拟化 hypervisor。在 FreeBSD 世界中，这个原生解决方案被称为 *Bhyve*（拼写为 **bee hive**）。

![bhyve-logo](https://vermaden.wordpress.com/wp-content/uploads/2023/08/bhyve-logo.png?w=960)

在功能方面，它类似于 KVM/XEN/VMware/VirtualBox。

它唯一缺少的是 *Live Migration 热迁移* 功能。这个功能正在开发中，因为 *Save*/*Resume* 功能正在测试中，很快将被合并。

除此之外——它的性能与前述竞争产品不相上下或相似。归根结底——它们都使用相同的 AMD/Intel CPU 功能，这些功能支持 x86 平台的虚拟化。

如果你想全面了解 Bhyve hypervisor 的全部特性——可以查看 **[FreeBSD Bhyve Virtualization](https://vermaden.wordpress.com/2023/08/18/freebsd-bhyve-virtualization/)** 文章。

另外，如果 *Bhyve* 不适合你的需求，FreeBSD 也支持 *XEN*（作为 **dom0**）和 *VirtualBox* hypervisor。

更多关于 *Bhyve* 的信息：

* [https://vermaden.wordpress.com/2023/08/18/freebsd-bhyve-virtualization/](https://vermaden.wordpress.com/2023/08/18/freebsd-bhyve-virtualization/)
* [https://freebsd.org/handbook/virtualization-host-bhyve.html](https://freebsd.org/handbook/virtualization-host-bhyve.html)
* [https://wiki.freebsd.org/bhyve](https://wiki.freebsd.org/bhyve)
* [https://man.freebsd.org/bhyve/8](https://man.freebsd.org/bhyve/8)

## FreeBSD Ports 基础设施

这又是一个例子，说明为什么 FreeBSD 如此出色。当你安装某个版本的 Ubuntu 或 CentOS 时，很可能得到的不是最新版本的包，而是该发行版发布时比较新的版本。这一点在 CentOS 世界（以及其上游企业源系统 Red Hat）中尤其明显——在 **.0**（点零）版本发布时，包几乎都是最新的，但当该版本的 **.8** 或 **.9** 版本可用时，包已经严重过时。更不用提像 Firefox 这样每个月都有新版本发布的情况了……

![packages](https://vermaden.wordpress.com/wp-content/uploads/2020/09/packages.jpg?w=960)

正如我之前在说明 FreeBSD *基础系统* 时所说，*FreeBSD Ports*（以及通过 **pkg(8)** 命令可用的由其构建的包）是独立的。这意味着来自 *FreeBSD Ports* 的第三方软件几乎总是最新的（或非常接近最新）。你甚至可以在 **[repology.org](https://repology.org/repositories/statistics/newest)** 网站上查看详细信息。下面是本文撰写时 **[repology.org](https://repology.org/repositories/statistics/newest)** 统计的“快照”。在线表格非常长，这里仅复制了与本文相关的系统部分。

![repology](https://vermaden.wordpress.com/wp-content/uploads/2020/09/repology-1.png?w=960)

*FreeBSD Ports* 的另一个优势是，它提供了极其庞大的软件量——本文撰写时已有 40354 个 port，并且数量仍在增加。可直接安装的包数量略少，32000 余个可用包。

我曾在 2009 年短暂迁移到 OpenSolaris，在我的 Dell Latitude D630 笔记本上，因为我非常喜欢 Solaris 的所有功能（包括当时 FreeBSD 上尚不可用的 ZFS 和 *ZFS Boot Environments*），OpenSolaris 基于 GNOME 的桌面也相当不错，即使在 Nautilus 文件管理器中提供 ZFS 快照的 Time Slider 功能。当时我成功连接了 WiFi，声音正常，总体而言笔记本上的一切都得到支持并正常运行……但软件很少。当然，“大”项目如 GIMP 或 OpenOffice 在默认 **pkg(8)** 仓库中可用，但其他不多。当时 OpenSolaris 的可用包不到 4000 个，而 FreeBSD 的包大约有 25000 个，如果我没记错的话。

你也可以通过网页轻松浏览可用的 *FreeBSD Ports*（及其选项），使用 [**https://freshports.org/**](https://freshports.org/) 页面。

![ports-later](https://vermaden.wordpress.com/wp-content/uploads/2020/09/ports-later.png?w=960)

*FreeBSD Ports* 的数量是一方面，功能则是另一方面。无论你使用哪种 Linux 发行版，总会遇到某些软件在编译和发布时没有加上你急需的标志。如果在 FreeBSD 上遇到类似情况，“痛苦”只会持续片刻，因为你可以非常轻松地使用所需选项重新编译该软件，并用自己的版本替换掉“默认”包。例如，FreeBSD 项目因为现有的 MP3 专利而不敢提供 Lame 包（**译注：在 2020 年[已经移除](https://svnweb.freebsd.org/changeset/ports/554970)该限制及下述限制**），所以 **multimedia/ffmpeg** 包默认是没有 MP3 支持的（使用 **--disable-libmp3lame** 标志）。这就是为什么我有自己的 **audio/lame** 和 **multimedia/ffmpeg** 包，都是按照我的 **configure** 选项构建的，而且实现非常简单。你只需进入 **/usr/ports/multimedia/ffmpeg** 目录，输入 **make config**，在 ncurses 对话框中选择 **[x] LAME**。你选择的选项会保存到普通文件 **/var/db/ports/multimedia_ffmpeg/options**。如果删除该文件（或输入 **make rmconfig**），这些自定义选项将重置为默认值。然后输入 **make build deinstall install clean**，你的 port 就会以新选项构建并安装为包，无需其他操作。你甚至可以使用 **pkg(8)** 的 **pkg lock -y ffmpeg** 命令锁定该包，防止之后被窜改，但更好的做法是每次执行 **pkg upgrade** 时重新构建此类包，以应对库版本升级和变化。虽然使用这些命令创建脚本实现自动化非常容易快捷，你也可以使用 *FreeBSD Ports* 基础设施的其他部分——比如 Poudriere（或 Synth）——在下一部分会详细介绍。

你也不必每个 port 都这样配置（对于大量 port 来说会很麻烦），可以只在 **/etc/make.conf** 文件中全局指定所需的 (**OPTIONS_SET**) 或不需要的 (**OPTIONS_UNSET**) 参数。你还可以指定软件的默认版本，例如使用 Apache 2.2 而不是 2.4，使用 PHP 7.0 而不是 7.2。所有默认版本信息可以在 **/usr/ports/Mk/bsd.default-versions.mk** 文件中找到。只要设置好这些选项，你可以使用 **portmaster(8)** 工具从 FreeBSD Ports 构建/重建或更新包。类似 Gentoo Linux 的 **USE** 标志，但这才是原版。Gentoo 的大部分思想都是从 FreeBSD 系统及其 Ports 基础设施中借鉴的。

Poudriere 是一款构建框架，它使用 *FreeBSD Ports* 和 *FreeBSD Jails* 以干净、可复现的方式构建所需包。你可以为 **pkg(8)** 命令创建全新的二进制包仓库。我提到 Synth，是因为 Poudriere 通常用于生成整个包仓库，而 Synth 通常仅用于重建几个不符合你需求的包。

关于 *FreeBSD Ports*，有一点常被新手误解。Ports 和通过 **pkg(8)** 工具获取并安装的包有什么区别？很简单：软件包只是预构建并安装的 port，仅此而已。当你使用 **pkg(8)** 命令安装二进制包时，你使用的是某个人（在 FreeBSD 的情况下就是 FreeBSD 项目）在某个时间点从 *FreeBSD Ports* 构建的包。虽然 FreeBSD 努力保持构建包尽可能更新，但 *FreeBSD Ports* 的本质是总比构建好的包更新。这就是为什么你可以自己使用 *FreeBSD Ports* 构建并安装所需包的新版本。考虑安全漏洞时，这种使用方式尤其有意义。当某些本地执行的命令（如 **file(1)**）存在安全漏洞时，可能不必急于更新，因为漏洞对你可能无害；但当 Firefox 的新版本修复了重要安全漏洞时，最好尽快从 *FreeBSD Ports* 更新，因为等待两天构建包（与其他包一起）可能太长。

更多关于 *FreeBSD Ports* 的信息：

* [https://freebsd.org/handbook/ports-using.html](https://freebsd.org/handbook/ports-using.html)
* [https://freebsd.org/handbook/ports-poudriere.html](https://freebsd.org/handbook/ports-poudriere.html)
* [https://man.freebsd.org/ports](https://man.freebsd.org/ports)
* [https://man.freebsd.org/build](https://man.freebsd.org/build)
* [https://freshports.org/](https://freshports.org/)
* [https://repology.org/repositories/statistics/newest](https://repology.org/repositories/statistics/newest)

## 从源码更新/构建

虽然 *FreeBSD Ports* 基础设施是为第三方软件设计的，FreeBSD *基本系统*（或其部分）也可以轻松方便地从源码构建。FreeBSD 内核配置也非常小巧且简单。相比之下，Linux 内核配置包含数千个选项——例如在默认 CentOS 8.2 安装中有 **4432** 个选项，而 FreeBSD 的 GENERIC 配置的选项数量约只有前者的二十分之一——只有 **260** 个选项。但这还未触及主题的核心。你可以从最小化的 FreeBSD 内核配置开始，其仅指定 **75** 个选项。

```sh
Linux # grep -c '^CONFIG' /boot/config-$( uname -r )
4432

FreeBSD # grep -c -E '^(device|options)' /usr/src/sys/amd64/conf/GENERIC
260

FreeBSD # grep -c -E '^(device|options)' /usr/src/sys/amd64/conf/MINIMAL
75
```

……而且这不仅仅是选项数量更少的问题。你能说说重建 CentOS 或 Ubuntu（例如去掉蓝牙支持）需要多少步骤（以及哪些步骤是必须的）吗？

![code](https://vermaden.wordpress.com/wp-content/uploads/2020/09/code.jpg?w=960)

相反，在 FreeBSD 上操作非常简单（且快速）。虽然 **/etc/make.conf** 文件用于启用/禁用 Ports 选项，但 **/etc/src.conf** 文件用于在从源码构建 FreeBSD *基本系统* 时启用/禁用选项。要构建不含 Bluetooth 支持的 FreeBSD，只需在 **/etc/src.conf** 文件中添加 **WITHOUT_BLUETOOTH=yes**，然后输入以下命令进行构建：

```sh
FreeBSD # beadm create safe
FreeBSD # cd /usr/src
FreeBSD # make buildworld kernel
FreeBSD # reboot
FreeBSD # cd /usr/src
FreeBSD # etcupdate -p      # // 以前是: mergemaster -p
FreeBSD # make installworld
FreeBSD # etcupdate -B      # // 以前是: mergemaster -iU
FreeBSD # reboot
```

瞧！现在你拥有了不含蓝牙支持的 FreeBSD……而且如果任何步骤失败，或者由于你的经验/专业知识不足导致 FreeBSD 系统无法启动或损坏，你可以使用 **/rescue** 中的工具尝试修复（或至少找出问题所在）；如果你不想处理这些问题，只需在 FreeBSD **loader(8)** 中选择 **safe** *ZFS Boot Environment*，即可启动到你开始构建修改版 FreeBSD 之前的系统。是的，这里你几乎“刀枪不入”。在拥有 294 个 **WITHOUT_X** 选项和 125 个 **WITH_X** 选项的情况下，你真的可以将 FreeBSD *基本系统* 调整到完全符合你的需求。

```sh
FreeBSD # zgrep -c WITHOUT_ /usr/share/man/man5/src.conf.5.gz
294

FreeBSD # zgrep -c WITH_ /usr/share/man/man5/src.conf.5.gz
125
```

通过源码更新 FreeBSD 的一大缺点是，你无法使用 **freebsd-update** 工具……但这并不妨碍你创建自己的 FreeBSD 更新服务器，这样你就可以通过使用 CURRENT 或 STABLE 系统（而不是 RELEASE）添加更新来使用 **freebsd-update**。这一过程在官方 FreeBSD 文档的 [**Build Your Own FreeBSD Update Server**](https://www.freebsd.org/doc/en/articles/freebsd-update-server/article.html) 文章中有详细描述。

更多关于 *FreeBSD 源码更新/构建* 的信息：

* [https://freebsd.org//handbook/kernelconfig.html](https://freebsd.org//handbook/kernelconfig.html)
* [https://freebsd.org/handbook/makeworld.html](https://freebsd.org/handbook/makeworld.html)
* [https://freebsd.org/doc/en/articles/freebsd-update-server/](https://freebsd.org/doc/en/articles/freebsd-update-server/)
* [https://man.freebsd.org/build](https://man.freebsd.org/build)
* [https://man.freebsd.org/src.conf](https://man.freebsd.org/src.conf)

## 存储

存储是 FreeBSD 真正出彩的部分之一。很多人因为 FreeBSD 对 ZFS 文件系统的良好集成而喜爱它，这确实不假。FreeBSD 中的 ZFS 一直是一级公民。最近，OpenZFS 2.0 也已从 FreeBSD 与 Linux 的联合上游仓库集成。越来越多的 FreeBSD 功能和解决方案正在利用 ZFS 的特性。

![openzfs](https://vermaden.wordpress.com/wp-content/uploads/2020/09/openzfs.jpg?w=960)

大多数喜欢 FreeBSD 集成 ZFS 的人并不了解 FreeBSD 的 GEOM 模块化磁盘转换框架，它提供了各种存储相关功能和工具，例如软件 RAID0/RAID1/RAID10/RAID3/RAID5 配置，或使用 GELI/GDBE 对底层设备进行透明加密（类似 Linux 上的 LUKS）。它还允许为任何文件系统提供透明的文件系统日志功能（GJOURNAL，甚至适用于 FAT32 或 exFAT），或通过 GEOM GATE 设备将块设备导出到网络上（类似块设备的 NFS）。

![storage](https://vermaden.wordpress.com/wp-content/uploads/2020/09/storage.jpg?w=960)

FreeBSD 还拥有自己的 FUSE 实现，使所有基于 FUSE 的文件系统能够在 FreeBSD 上原生工作。虽然很多 Linux 用户了解 DRBD，但很少有人知道 FreeBSD 自带类似 DRBD 的解决方案 HAST——其功能完全相同。尽管 ZFS 功能丰富，FreeBSD 仍然维护并开发快速且内存占用小的 UFS 文件系统，该文件系统今天根据使用场景可使用 *Soft Updates* (SU) 或 *Journaled Soft Updates* (SUJ)。例如，在 UFS 文件系统上使用 *Journaled Soft Updates* (SUJ) 管理 10 TB 数据，**fsck(8)** 检查大约只需 1 分钟。这些存储解决方案仅依赖 FreeBSD *基本系统* 即可实现。*FreeBSD Ports* 提供更多功能，包括分布式文件系统解决方案，如 CEPH、LeoFS、LizardFS 或兼容 Amazon S3 的 Minio 存储。

更多关于 *存储* 的信息：

* [https://is.gd/bsdstg](https://is.gd/bsdstg)
* [https://freebsd.org/handbook/disks.html](https://freebsd.org/handbook/disks.html)
* [https://freebsd.org/handbook/geom.html](https://freebsd.org/handbook/geom.html)
* [https://freebsd.org/handbook/zfs.html](https://freebsd.org/handbook/zfs.html)
* [https://freebsd.org/handbook/filesystems.html](https://freebsd.org/handbook/filesystems.html)
* [https://man.freebsd.org/geom](https://man.freebsd.org/geom)
* [https://man.freebsd.org/geli](https://man.freebsd.org/geli)
* [https://man.freebsd.org/fusefs](https://man.freebsd.org/fusefs)
* [https://man.freebsd.org/newfs](https://man.freebsd.org/newfs)
* [https://man.freebsd.org/hast.conf](https://man.freebsd.org/hast.conf)

## Init 系统

FreeBSD 提供了非常简单但功能强大的 init 系统。它在 **/etc/rc.conf** 文件中有系统级配置，你可以通过条目 **service_enable=YES** 和 **service_enable=NO** 启用/禁用所需服务。你甚至不需要打开 **vi(1)** 来手动添加，只需输入 **sysrc service_enable=YES**，条目就会被添加到 **/etc/rc.conf** 文件中。系统还提供了默认值和默认启用的服务，你可以在 **/etc/defaults/rc.conf** 文件中找到它们，以及大量注释。每个 FreeBSD 服务文件都有 **PROVIDE**/**REQUIRE** 条目，用于自动确定服务启动顺序。可以并行运行的服务会同时启动以节省时间。例如，没有网络启动 **sshd(8)** 守护进程是毫无意义的。要启动或停止服务，只需输入 **service sshd start** 或 **service sshd stop** 命令。但当某个服务未在 **/etc/rc.conf** 文件中启用时，需要使用 **onestart** 和 **onestop**。*基本系统* 的分离在这里仍然保持：FreeBSD *基本系统* 服务位于 **/etc/rc.d** 目录，而来自 ports/packages 的第三方应用位于 **/usr/local** 前缀下，即 **/usr/local/etc/rc.d** 目录。

使用 **systemd(1)** 时，你无法预测服务启动顺序，每次可能都不同，完全没有确定性。而在 FreeBSD 上，你可以精确知道哪些服务会启动，因为它们总是根据 **PROVIDE**/**REQUIRE** 条目以相同顺序排列。FreeBSD 还提供了工具 **rcorder(8)**，可用于查看所有服务、*基本系统* 服务或第三方服务的精确启动顺序。同时，**service -r** 命令可以显示启动时的实际顺序。

```sh
FreeBSD # rcorder /etc/rc.d/* | head
/etc/rc.d/growfs
/etc/rc.d/sysctl
/etc/rc.d/hostid
/etc/rc.d/zvol
/etc/rc.d/dumpon
/etc/rc.d/ddb
/etc/rc.d/geli
/etc/rc.d/gbde
/etc/rc.d/ccd
/etc/rc.d/swap

FreeBSD # rcorder /usr/local/etc/rc.d/* | tail
/usr/local/etc/rc.d/hald
/usr/local/etc/rc.d/git_daemon
/usr/local/etc/rc.d/fscd
/usr/local/etc/rc.d/cupsd
/usr/local/etc/rc.d/cups_browsed
/usr/local/etc/rc.d/clamav-clamd
/usr/local/etc/rc.d/clamav-milter
/usr/local/etc/rc.d/clamav-freshclam
/usr/local/etc/rc.d/avahi-dnsconfd
/usr/local/etc/rc.d/aria2

FreeBSD # rcorder /etc/rc.d/* /usr/local/etc/rc.d/* 2> | grep -C 3 sshd
/etc/rc.d/ubthidhci
/etc/rc.d/syscons
/etc/rc.d/swaplate
/etc/rc.d/sshd
/etc/rc.d/cron
/etc/rc.d/jail
/etc/rc.d/localpkg
```

在 FreeBSD 中添加新服务也非常简单，因为新服务的模板非常小巧且易于理解。

```sh
#!/bin/sh

. /etc/rc.subr

name=dummy
rcvar=dummy_enable

start_cmd="${name}_start"
stop_cmd=":"

load_rc_config $name
: ${dummy_enable:=no}
: ${dummy_msg="Nothing started."}

dummy_start()
{
	echo "$dummy_msg"
}

run_rc_command "$1"
```

如果你觉得还不够简单，FreeBSD 官方有一篇关于如何编写服务的文章——[**Practical rc.d Scripting in BSD**](https://www.freebsd.org/doc/en_US.ISO8859-1/articles/rc-scripting/)。

更多关于 *Init 系统* 的信息：

* [https://freebsd.org/handbook/configtuning-rcd.html](https://freebsd.org/handbook/configtuning-rcd.html)
* [https://freebsd.org/doc/en/articles/rc-scripting/](https://www.freebsd.org/doc/en/articles/rc-scripting/)
* [https://man.freebsd.org/rc](https://man.freebsd.org/rc)
* [https://man.freebsd.org/rc.conf](https://man.freebsd.org/rc.conf)
* [https://man.freebsd.org/rc.local](https://man.freebsd.org/rc.local)

## Linux 二进制兼容层

虽然 Linux 不能成为 FreeBSD，但 FreeBSD 可以运行 Linux——而且这并非低速模拟，而是通过对 Linux 系统调用的实现来完成的。过去企业常使用 FreeBSD 的 *Linux 二进制兼容层* 来运行仅适用于 Linux 的应用（当时 FreeBSD 上不可用），因为这样比在 Linux 上原生运行更快——[**FreeBSD 曾用于生成惊艳的特效**](https://www.freebsd.org/news/press-rel-1.html)，这是官方 FreeBSD 新闻稿，介绍 FreeBSD 曾用于为经典影片 *[黑客帝国 (1999)](https://www.imdb.com/title/tt0133093/)* 生成视觉特效。

![matrix](https://vermaden.wordpress.com/wp-content/uploads/2020/09/matrix.jpg?w=960)

如今，**LINUX_COMPAT** 同样原生高效，能运行 Linux 应用——甚至可以在 X11 下硬件加速运行 Linux 游戏。可以将其理解为针对 Linux 应用的 WINE，它位于 **/compat/linux** 目录下。它甚至实现了 Linux 的 **/proc** 虚拟文件系统，可挂载在 **/compat/linux/proc**，但并非强制要求。对于任何没有源码、仅能在 Linux 上运行的软件，*Linux 二进制兼容层* 都能派上用场。例如 [**f.lux**](https://justgetflux.com/) 项目。在了解 Redshift 之前，我曾使用 **LINUX_COMPAT** 运行 f.lux Linux 二进制版本，在 FreeBSD 屏幕上抑制蓝光。*Linux 二进制兼容层* 子系统还可以用于运行基于 Linux 的 *FreeBSD Jails*，例如 Devuan。

更多关于 *Linux 二进制兼容层* 的信息：

* [https://freebsd.org/handbook/linuxemu.html](https://freebsd.org/handbook/linuxemu.html)
* [https://freebsd.org/handbook/linuxemu-lbc-install.html](https://freebsd.org/handbook/linuxemu-lbc-install.html)
* [https://freebsd.org/doc/en/articles/linux-emulation/](https://freebsd.org/doc/en/articles/linux-emulation/)

## 简洁性

FreeBSD 很简单，但并不粗糙或难以驾驭。例如，和 Linux 一样，FreeBSD 系统也支持虚拟文件系统 **/proc**，但在 FreeBSD 上它是可选的，默认不启用，而 Linux 则离不开它。然而，Linux 在强制使用 **/proc** 的同时，还拥有另一个位于 **/sys** 的虚拟文件系统……但为什么 Linux 需要两个目的类似的虚拟文件系统？为什么不能把所有内容都放在已经存在的 **/proc** 下呢？这对我的理智来说简直是个谜。

而 **/sys** 并不是这场混乱的终点，它仅仅是开始。

那么这些呢？

* **securityfs**
* **devpts**
* **cgroup**
* **pstore**
* **bpf**
* **configfs**
* **selinuxfs**
* **systemd-1**
* **mqueue**
* **debugfs**
* **hugetlbfs**

看看 FreeBSD 在默认 ZFS 安装后的 **mount(8)** 输出，就能体会不同的简洁设计。

```sh
FreeBSD # mount
zroot/ROOT/12.1 on / (zfs, local, noatime, nfsv4acls)
devfs on /dev (devfs, local, multilabel)
zroot/tmp on /tmp (zfs, local, noatime, nosuid, nfsv4acls)
zroot/var/mail on /var/mail (zfs, local, nfsv4acls)
zroot/usr/home on /usr/home (zfs, local, noatime, nfsv4acls)
zroot/var/crash on /var/crash (zfs, local, noatime, noexec, nosuid, nfsv4acls)
zroot/var/log on /var/log (zfs, local, noatime, noexec, nosuid, nfsv4acls)
zroot/var/audit on /var/audit (zfs, local, noatime, noexec, nosuid, nfsv4acls)
zroot/var/tmp on /var/tmp (zfs, local, noatime, nosuid, nfsv4acls)
zroot/usr/src on /usr/src (zfs, local, noatime, nfsv4acls)
zroot/usr/ports on /usr/ports (zfs, local, noatime, nosuid, nfsv4acls)
```

几个 ZFS 数据集，以及一个用于 **/dev** 目录的虚拟 **devfs** 文件系统。如果安装在 UFS 上，则类似，只是会挂载几个 UFS 分区来替代 ZFS 数据集。

再看看 CentOS 8.2 的安装情况——只有一个物理根（/）XFS 文件系统。

```sh
[root@centos8 ~]# mount
sysfs on /sys type sysfs (rw,nosuid,nodev,noexec,relatime,seclabel)
proc on /proc type proc (rw,nosuid,nodev,noexec,relatime)
devtmpfs on /dev type devtmpfs (rw,nosuid,seclabel,size=919388k,nr_inodes=229847,mode=755)
securityfs on /sys/kernel/security type securityfs (rw,nosuid,nodev,noexec,relatime)
tmpfs on /dev/shm type tmpfs (rw,nosuid,nodev,seclabel)
devpts on /dev/pts type devpts (rw,nosuid,noexec,relatime,seclabel,gid=5,mode=620,ptmxmode=000)
tmpfs on /run type tmpfs (rw,nosuid,nodev,seclabel,mode=755)
tmpfs on /sys/fs/cgroup type tmpfs (ro,nosuid,nodev,noexec,seclabel,mode=755)
cgroup on /sys/fs/cgroup/systemd type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,xattr,release_agent=/usr/lib/systemd/systemd-cgroups-agent,name=systemd)
pstore on /sys/fs/pstore type pstore (rw,nosuid,nodev,noexec,relatime,seclabel)
bpf on /sys/fs/bpf type bpf (rw,nosuid,nodev,noexec,relatime,mode=700)
cgroup on /sys/fs/cgroup/cpuset type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,cpuset)
cgroup on /sys/fs/cgroup/memory type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,memory)
cgroup on /sys/fs/cgroup/blkio type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,blkio)
cgroup on /sys/fs/cgroup/hugetlb type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,hugetlb)
cgroup on /sys/fs/cgroup/net_cls,net_prio type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,net_cls,net_prio)
cgroup on /sys/fs/cgroup/cpu,cpuacct type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,cpu,cpuacct)
cgroup on /sys/fs/cgroup/freezer type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,freezer)
cgroup on /sys/fs/cgroup/perf_event type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,perf_event)
cgroup on /sys/fs/cgroup/rdma type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,rdma)
cgroup on /sys/fs/cgroup/pids type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,pids)
cgroup on /sys/fs/cgroup/devices type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,devices)
configfs on /sys/kernel/config type configfs (rw,relatime)
/dev/sda1 on / type xfs (rw,relatime,seclabel,attr2,inode64,noquota)
selinuxfs on /sys/fs/selinux type selinuxfs (rw,relatime)
systemd-1 on /proc/sys/fs/binfmt_misc type autofs (rw,relatime,fd=34,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=17309)
mqueue on /dev/mqueue type mqueue (rw,relatime,seclabel)
debugfs on /sys/kernel/debug type debugfs (rw,relatime,seclabel)
hugetlbfs on /dev/hugepages type hugetlbfs (rw,relatime,seclabel,pagesize=2M)
tmpfs on /run/user/0 type tmpfs (rw,nosuid,nodev,relatime,seclabel,size=187088k,mode=700)

[root@centos8 ~]# mount | awk '{print $5}' | sort -u
autofs
bpf
cgroup
configfs
debugfs
devpts
devtmpfs
hugetlbfs
mqueue
proc
pstore
securityfs
selinuxfs
sysfs
tmpfs
xfs
```

天哪。那里甚至很难找到任何真正的文件系统……幸运的是，我们可以请求仅显示 XFS 文件系统。

```sh
[root@centos7 ~]# mount -t xfs
/dev/sda1 on / type xfs (rw,relatime,seclabel,attr2,inode64,noquota)
```

现在我们来谈谈网络。假设你想在一台物理服务器上建立标准企业级网络配置，将两个网卡聚合成一个高可用接口 **bond0**（在 FreeBSD 上为 **lagg0**），然后在该 VLAN 上配置 VLAN 标签和 IP 地址。CentOS 7.x/8.x 的安装程序（Anaconda）在面对这种情况时会让你应接不暇。

```sh
[root@centos7 ~]# ls -1 /etc/sysconfig/network-scripts/ifcfg-*
ifcfg-Bond_connection_1
ifcfg-eno49
ifcfg-eno49-1
ifcfg-eno50
ifcfg-eno50-1
ifcfg-VLAN_connection_1

[root@centos7 ~]# cat /etc/sysconfig/network-scripts/ifcfg-Bond_connection_1
DEVICE=bond0
BONDING_OPTS="miimon=1 updelay=0 downdelay=0 mode=active-backup"
TYPE=Bond
BONDING_MASTER=yes
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=dhcp
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_PRIVACY=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME="Bond connection 1"
UUID=ca85417f-8852-43bf-96ee-5bd3f0f83648
ONBOOT=yes

[root@centos7 ~]# cat /etc/sysconfig/network-scripts/ifcfg-eno49
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=dhcp
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=eno49
UUID=2f60f50b-38ad-492a-b90a-ba736acf6792
DEVICE=eno49
ONBOOT=no

[root@centos7 ~]# cat /etc/sysconfig/network-scripts/ifcfg-eno49-1
HWADDR=xx:xx:xx:xx:xx:xx
TYPE=Ethernet
NAME=eno49
UUID=342b8494-126d-4f3a-b749-694c8c922aa1
DEVICE=eno49
ONBOOT=yes
MASTER=bond0
SLAVE=yes

[root@centos7 ~]# cat /etc/sysconfig/network-scripts/ifcfg-eno50
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=dhcp
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=eno50
UUID=4fd36e24-1c6d-4a65-a316-7a14e9a92965
DEVICE=eno50
ONBOOT=no

[root@centos7 ~]# cat /etc/sysconfig/network-scripts/ifcfg-eno50-1
HWADDR=xx:xx:xx:xx:xx:xx
TYPE=Ethernet
NAME=eno50
UUID=a429b697-73c2-404d-9379-472cb3c35e06
DEVICE=eno50
ONBOOT=yes
MASTER=bond0
SLAVE=yes

[root@centos7 ~]# cat/etc/sysconfig/network-scripts/ifcfg-VLAN_connection_1
VLAN=yes
TYPE=Vlan
PHYSDEV=ca85417f-8852-43bf-96ee-5bd3f0f83648
VLAN_ID=601
REORDER_HDR=yes
GVRP=no
MVRP=no
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=none
IPADDR=10.20.30.40
PREFIX=24
GATEWAY=10.20.30.1
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_PRIVACY=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME="VLAN connection 1"
UUID=90f7a9bb-1443-4adf-a3eb-86a03b23ecfb
ONBOOT=yes
```

说明一下——我选择了“STATIC”（静态）IPv4 地址，但安装程序却让这些接口同时使用 DHCP 和那个 STATIC 地址。这可能是一个 bug，不过我们直接说重点。

经过手动使用 **vi(1)** 修正（花费大约一小时）后，它应该是这样的。

```sh
[root@centos7 ~]# cat /etc/sysconfig/network
GATEWAY=10.20.30.1
NOZEROCONF=yes

[root@centos7 ~]# ls -1 /etc/sysconfig/network-scripts/ifcfg-*
ifcfg-bond0
ifcfg-bond0.601
ifcfg-eno49
ifcfg-eno50

[root@centos7 ~]# cat /etc/sysconfig/network-scripts/ifcfg-bond0
DEVICE=bond0
BONDING_OPTS="miimon=1 updelay=0 downdelay=0 mode=active-backup"
TYPE=Bond
BONDING_MASTER=yes
BOOTPROTO=none
IPV4_FAILURE_FATAL=no
IPV6INIT=no
ONBOOT=yes

[root@centos7 ~]# cat /etc/sysconfig/network-scripts/ifcfg-bond0.601
VLAN=yes
TYPE=Vlan
VLAN_ID=601
DEVICE=bond0.601
REORDER_HDR=yes
GVRP=no
MVRP=no
BOOTPROTO=none
IPADDR=10.20.30.40
PREFIX=24
IPV4_FAILURE_FATAL=no
IPV6INIT=no
ONBOOT=yes

[root@centos7 ~]# cat /etc/sysconfig/network-scripts/ifcfg-eno49
BOOTPROTO=none
IPV4_FAILURE_FATAL=no
IPV6INIT=no
TYPE=Ethernet
NAME=eno49
DEVICE=eno49
ONBOOT=yes
MASTER=bond0
SLAVE=yes

[root@centos7 ~]# cat /etc/sysconfig/network-scripts/ifcfg-eno50
BOOTPROTO=none
IPV4_FAILURE_FATAL=no
IPV6INIT=no
TYPE=Ethernet
NAME=eno50
DEVICE=eno50
ONBOOT=yes
MASTER=bond0
SLAVE=yes
```

更好了……但仍然占用大量空间，需要多个文件来完成这个相对简单的配置。更不用说其复杂度高、容易出错的特点。而在 FreeBSD 上，同样的配置只需在单个 **/etc/rc.conf** 文件中写 7 行即可，如下所示。

```sh
FreeBSD # cat /etc/rc.conf
ifconfig_fxp0="up"
ifconfig_fxp1="up"
cloned_interfaces="lagg0"
ifconfig_lagg0="laggproto failover laggport fxp0 laggport fxp1"
vlans_lagg0="601"
ifconfig_lagg0_601="inet 10.20.30.40/24"
defaultrouter="10.20.30.1"
```

但这还不是所有的复杂之处。过去，你可以使用简单的 **service network restart** 命令重启 RHEL/CentOS/Rocky/... 的网络配置……现在不行了。你需要输入两个命令：先 **nmcli networking off**，再 **nmcli networking on**。为了确保第二条命令能够执行，并且在执行前不会丢失网络连接，你必须在 **screen(1)** 或 **tmux(1)** 终端复用器中一个接一个地输入它们。简直方便得不得了 🙂。

启动过程呢？FreeBSD 可以直接从 ZFS 根分区启动，只需一个小的 512 KB 不可挂载分区。无需单独的 **/boot** 设备。而 Linux 总是需要单独的 **/boot** 分区，并且里面充满了 GRUB 模块。不管是 ZFS 还是 LVM，这都是必须的。这也是为什么在 Linux 上实现 *ZFS Boot Environments* 非常复杂——即使你在 Linux 系统上使用 ZFS 根分区，仍然存在未受保护的 **/boot** 文件系统，无法用 ZFS 快照，只能用传统方式保护，这几乎摧毁了 *ZFS Boot Environments* 的理念。

FreeBSD 真的是一个简单且设计周到的操作系统，同时也是被严重低估的系统。

## 改革而非革命

有多少 Linux 工具或子系统被废弃或被新工具取代？为什么没有为 **ifconfig(8)** 增补新选项，而是引入了全新的 **ip(8)** 命令？同样，**netstat(8)** 被 **ss(8)** 替代，**arp(8)**/**iwconfig**/**route(8)** 等也如此。整个 init 系统呢？Linux 世界已经被 **systemd(1)** 占领，无论你喜欢与否。即便像 Ubuntu 这样拥有成熟 init 系统（Upstart）的发行版，也完全转向 **systemd(1)**。如今，未采纳 systemd 的发行版非常少，被视为小众。

![evolution](https://vermaden.wordpress.com/wp-content/uploads/2020/09/evolution.jpg?w=960)

而在 FreeBSD 世界中，仅在无法通过其他方式实现新功能时才会进行重大变更。这是 FreeBSD 最不希望发生的情况。FreeBSD 的发展注重稳定性和向后兼容性。用户空间工具会随着新选项逐步更新，而不是反复重写。更不用说，将一个工具换成另一个工具会带来多少新的 bug。

更多关于 *改革而非革命* 的信息：

* [https://duck.com/?q=deprecated+linux+commands](https://duck.com/?q=deprecated+linux+commands)
* [https://duck.com/?q=deprecated+linux+subsystems+-windows](https://duck.com/?q=deprecated+linux+subsystems+-windows)

## 文档

拥有一款几乎能做任何事情的系统，但不知道如何操作，这样的系统几乎没用（或者至少使用起来非常麻烦）。FreeBSD 提供了无与伦比的文档，这些文档都是持续维护和更新的。除了其传奇的 [**FreeBSD 手册**](https://freebsd.org/handbook/) 和 [**FreeBSD FAQ**](https://freebsd.org/faq/)，FreeBSD 项目还提供官方 [**FreeBSD 文章**](https://www.freebsd.org/docs/books.html)，涵盖各种 FreeBSD 主题。[**手册页**](https://man.freebsd.org/man) 也非常详细，并包含大量示例。还有 [**FreeBSD Wiki**](https://wiki.freebsd.org/) 页面，用于记录正在进行的文档和与 FreeBSD 开发相关的想法。如果你在使用 FreeBSD 时遇到问题或有疑问，也可以访问官方 [**FreeBSD 论坛**](https://forums.freebsd.org/) 和老派的 [**邮件列表**](https://www.freebsd.org/community/mailinglists.html)。

![documentation](https://vermaden.wordpress.com/wp-content/uploads/2020/09/documentation.jpg?w=960)

以上仅是官方项目的知识来源，此外还有大量 FreeBSD 书籍。你也可以参考我专门的 **[FreeBSD 书籍](https://vermaden.wordpress.com/2022/02/04/books-about-freebsd/)** 文章，深入了解可用的 FreeBSD 书籍。以下是最佳且最新的书籍：

* **Absolute FreeBSD – Complete Guide to FreeBSD** – 第三版（2019）
* **Beginning Modern Unix**（2018）
* **Book of PF** – 第三版（2015）
* **Design and Implementation of FreeBSD 11 Operating System（FreeBSD 操作系统设计与实现）** – 第二版（2015）
* **FreeBSD Device Drivers**（2012）
* **FreeBSD Mastery – ZFS**（2015）
* **FreeBSD Mastery – Advanced ZFS**（2016）
* **FreeBSD Mastery – Storage Essentials**（2014）
* **FreeBSD Mastery – Specialty Filesystems**（2015）
* **FreeBSD Mastery – Jails**（2019）

还有两本专门面向 BSD 和 FreeBSD 系统的杂志，均免费，涵盖大量 FreeBSD 有趣话题：

* **[BSD Magazine](https://bsdmag.org/)**
* [**FreeBSD 期刊**](https://freebsdfoundation.org/our-work/journal/)

拥有如此丰富的知识和支持，要在 FreeBSD 系统上实现所需功能几乎没有难度。

## 社区

最后但同样重要，我认为社区甚至比优秀的文档更重要（FreeBSD 的文档非常棒）。使用 FreeBSD 的人通常是有意识选择的，并且往往在 FreeBSD 领域以及其他 UNIX 系统相关领域都有经验。他们往往经历了从 Linux 系统开始使用的漫长过程，最终选择 FreeBSD，或者在日常仍然从事 Linux 管理工作，但在休息时使用更合理、理智的 FreeBSD 解决方案。我总是觉得 FreeBSD 社区非常乐于助人且友好，尤其对新人非常耐心。即便你试图在不公或有争议的讨论中“逼迫” FreeBSD 用户“争辩”，他们也会以尊严和技术论据回应，而不是对你大喊大叫。

FreeBSD 项目甚至专门为 Linux 新手（有时被称为 **systemd(1)** 难民）制作了若干文章和 Handbook 章节：

* [**For People New to Both FreeBSD and UNIX®**](https://freebsd.org/doc/en/articles/new-users/index.html)
* [**FreeBSD Quickstart Guide for Linux® Users**](https://freebsd.org/doc/en/articles/linux-users/)
* [**Explaining BSD**](https://freebsd.org/doc/en/articles/explaining-bsd/)

## 结语

我尽力避免将其写成对 Linux 的抱怨，但有些人可能会有这种感觉——如果是这样，请记住，这并非我的本意。FreeBSD 和 Linux 以及任何其他操作系统一样，都有其优缺点。希望我向你展示了 FreeBSD 中最有趣的部分。将来我可能会在这里无预警地增加新章节 🙂。

## 外部讨论

来自“外部”来源的讨论和评论可在以下链接查看：

* [**Lobsters**](https://lobste.rs/s/4egn7o/quare_freebsd)
* [**Hacker News**](https://news.ycombinator.com/item?id=31664952)
* [**Reddit**](https://www.reddit.com/r/freebsd/comments/inwclo/quare_freebsd/)
