# 新型 ZFS 启动环境工具

- [Upgrade FreeBSD with ZFS Boot Environments](https://vermaden.wordpress.com/2021/02/23/upgrade-freebsd-with-zfs-boot-environments/)
- 作者：𝚟𝚎𝚛𝚖𝚊𝚍𝚎𝚗
- 发布时间：2021/02/23


我以强烈支持 **ZFS 启动环境** 而闻名……而这并非没有理由。我已经多次说明了支持的原因，但大多数（或全部）内容在这里被浓缩了——[https://is.gd/BECTL](https://is.gd/BECTL)——在我关于此的演示中。即将发布的 FreeBSD 13.0-RELEASE 看起来非常有希望。在许多测试中，它的速度几乎是 12.2-RELEASE 的两倍。哎呀！
![](https://vermaden.wordpress.com/wp-content/uploads/2021/02/phoronix-mean.png?w=960)

详细测试可以在 [phoronix.com](https://www.phoronix.com/scan.php?page=article&item=freebsd-13-beta1) 网站上查看。由于我已经安装了 12.2-RELEASE，我想测试 13.0-BETA*，看看对我来说重要的功能——比如正常工作的挂起/恢复——在新版本中是否如宣传般正常工作。这正是使用 *ZFS Boot Environments* 可以完美实现的任务。

在下面的示例中，我们将创建一个全新的 **ZFS 启动环境**，克隆我们当前的 12.2-RELEASE 系统，并在该 BE 中升级到 13.0-BETA3 版本……这样重启时只需一次，而不像典型的 **freebsd-update(8)** 升级流程需要三次重启。

我假设你已经安装了带 ZFS 的 FreeBSD 12.2-RELEASE（FreeBSD 默认 ZFS 安装），并且系统以 UEFI 或 UEFI+BIOS 模式安装。以下是所需的步骤。

```sh
(host) # beadm create 13                        # 创建新的 '13' ZFS Boot Environment
       Created successfully
(host) # beadm mount 13 /var/tmp/BE-13          # 将新的 '13' BE 挂载到某个目录
       Mounted successfully on '/var/tmp/BE-13'
(host) # chroot /var/tmp/BE-13                  # 切换到该目录的 chroot(8) 环境
  (BE) # mount -t devfs devfs /dev              # 在该 BE 中挂载 devfs(8)
  (BE) # rm -rf /var/db/freebsd-update          # 删除任何旧补丁
  (BE) # mkdir /var/db/freebsd-update           # 为补丁创建新的目录
  (BE) # freebsd-update upgrade -r 13.0-BETA3   # 获取升级所需的补丁
  (BE) # freebsd-update install                 # 安装内核及内核模块
  (BE) # freebsd-update install                 # 安装用户空间程序/二进制文件/库
  (BE) # pkg upgrade                            # 使用 pkg(8) 升级所有软件包
  (BE) # freebsd-update install                 # 移除旧的库和文件
  (BE) # exit                                   # 退出 chroot(8) 环境
(host) # umount -f /var/tmp/BE-13/dev           # 卸载该 BE 中的 devfs(8)
(host) # beadm activate 13                      # 激活新的 '13' BE
       Activated successfully
```

我在这个过程中使用的是我的 [**sysutils/beadm**](https://github.com/vermaden/beadm)，但你也可以使用 FreeBSD 基础系统自带的 **bectl(8)**。

我们还需要更新新的 FreeBSD **loader(8)** —— 通过这种方式可以更新 —— 感谢 [@JeffSipek](https://twitter.com/JeffSipek) 的指点。

在我的系统中，FreeBSD 安装在 **ada1** 设备上。

```sh
(host) # gpart show -p ada1 | grep efi                # 查找 UEFI msdosfs(5) 分区
               40     409600  ada1p1  efi  (200M)     # <-- 就是这一项
(host) # mount_msdosfs /dev/ada1p1 /mnt               # 将其挂载到 /mnt 下
(host) # find /mnt                                    # 显示其内容
       /mnt
       /mnt/efi
       /mnt/efi/boot
       /mnt/efi/boot/bootx64.efi                      # 更新 bootx64.efi 文件
(host) # cp /boot/boot1.efi /mnt/efi/boot/bootx64.efi # 从 /boot/boot1.efi 文件复制
(host) # umount -f /mnt                               # 卸载 /mnt 文件系统
```

有小概率你无法挂载 **efi** 分区。即使使用 **fsck(8)** 也无法解决此问题。

一些用户遇到的典型错误如下：

```sh
(host) # mount_msdosfs /dev/ada1p1 /mnt # 尝试挂载 EFI 分区时出错
       mount_msdosfs: /dev/ada1p1: Invalid argument

(host) # fsck_msdosfs -y /dev/ada1p1    # 尝试对该分区执行 fsck(8) 时出错
       ** /dev/ada1p1
       Invalid signature in boot block: 0b6a
```

如果你遇到这个问题，首先将当前的 **efi** 分区备份，例如备份到 **/BACKUP.ada1p1** 文件。


```sh
(host) # dd < /dev/ada1p1 > /BACKUP.ada1p1 bs=1m
```

现在我们将从零开始创建全新的 **efi** 分区。

```sh
(host) # newfs_msdos -F 32 -c 1 /dev/ada0p1            # 创建新的 FAT32 分区
(host) # mount_msdosfs /dev/ada0p1 /mnt                # 挂载到 /mnt 下
(host) # mkdir -p /mnt/efi/boot                        # 创建所需目录
(host) # cp /boot/loader.efi /mnt/efi/boot/bootx64.efi # 从 /boot/loader.efi 复制文件
(host) # umount -f /mnt                                # 卸载 /mnt 文件系统
```

现在你应该已经有了新的“可用” **efi** 分区。

最后一步是 **reboot(8)** 进入新的 13.0-BETA3 系统。

```sh
(host) # reboot
```

如果你发现新的引导加载程序无法加载你的新 FreeBSD，也可以选择将 **/boot/boot1.efi** 而不是 **/boot/loader.efi** 复制到 **/mnt/efi/boot/bootx64.efi**。

请记住，如果你从 **geli(8)** 加密系统启动，那么 **/boot/loader.efi** 是必需的，如果使用 **/boot/boot1.efi** 文件将无法启动。

完成。

现在你应该能看到全新的 FreeBSD **loader(8)**：🙂

![](https://vermaden.wordpress.com/wp-content/uploads/2021/02/fancy-loader.png?w=960)

你现在可以享受最新的 FreeBSD 13.0-BETA3 安装。

同样的步骤将适用于即将发布的 FreeBSD 13.0-RC*（RC1/RC2/RC3）版本的更新，最终希望在 2021 年 3 月发布 FreeBSD 13.0-RELEASE。

## 更新 1 – 如果一切顺利

你现在拥有最先进的 FreeBSD 系统，运行速度应该比 12.2-RELEASE 更快，同时你仍保留旧的 12.2-RELEASE Boot Environment，如果在 13.0 版本中遇到问题，可以回退。

在我的系统中，它看起来是这样的：

```sh
(host) # beadm list
       BE   Active Mountpoint Space Created
       12.2 -      -           6.5G 2021-02-12 10:15
       13   NR     /          18.8G 2021-02-13 11:32
```

**Space** 列有点容易误导，因为它会计算例如快照占用的空间。要获取每个 Boot Environment 占用的精确信息，请使用 **-D** 选项。这样你可以分别查看每个 Boot Environment 的空间信息。

```sh
(host) # beadm list -D
       BE   Active Mountpoint  Space Created
       12.2 -      -            9.8G 2021-02-12 10:15
       13   NR     /            9.6G 2021-02-13 11:32
```

我会暂时保留 12.2-RELEASE Boot Environment —— 可能在 13.0-RELEASE 发布后一个月左右删除它，但如果你测试过所有需求，并且发现 13.0 能像 12.2-RELEASE 一样或更好地满足需求，那么你可以使用下面的命令删除旧的 Boot Environment。

```sh
(host) # beadm destroy 12.2
```

## 更新 2 – 如果出现问题

一般来说，如果新 BE 名为 ‘**13**’ 无法启动（或在启动时挂起），只需选择你在升级前使用的旧 Boot Environment —— 也就是包含 12.2-RELEASE 的那个。

这样你就可以回到升级前可正常工作的系统。

如果这也失败了（或者引导加载程序损坏），那么下载 [FreeBSD-13.0-BETA3-amd64-memstick.img](http://ftp.freebsd.org/pub/FreeBSD/releases/ISO-IMAGES/13.0/FreeBSD-13.0-BETA3-amd64-memstick.img) 镜像，并用 **dd(8)** 命令写入 U 盘。

```sh
# dd if=FreeBSD-13.0-BETA3-amd64-memstick.img of=/dev/da0 bs=1M status=progress
```

现在你有了 FreeBSD 13.0-BETA3 的 U 盘，可以从它启动并修复你的安装。启动后选择 **LiveCD**，在 **login:** 提示符下输入 **root** 并按 **[ENTER]** 留空密码。

接下来可以执行的操作取决于具体损坏情况，我无法预估每种可能的错误和修复方案。如果在升级过程中遇到任何问题，请通过你方便的方式联系我，我们会一起解决。

## 更新 3 – 使用新版本 beadm(8) 更快升级

今天（2022/05/06）我发布了新版本 **beadm(8)** 1.3.5，带来了新的 **chroot(8)** 功能。它已经提交到 FreeBSD Ports 树的 [263805](https://bugs.freebsd.org/bugzilla/show_bug.cgi?id=263805) PR 中，所以预计很快就能获取到软件包。

你也可以直接这样更新 **beadm(8)**：

```sh
# fetch -o /usr/local/sbin/beadm https://raw.githubusercontent.com/vermaden/beadm
```

现在讲更快的升级流程 —— 以下说明根据你使用的 shell 而定。

* **ZSH / CSH**

```sh
# beadm create 13.1-RC6
# beadm chroot 13.1-RC6
BE # zsh || csh
BE # yes | freebsd-update upgrade -r 13.1-RC6
BE # 重复执行 3 次 freebsd-update install
BE # exit
# beadm activate 13.1-RC6
# reboot
```

* **SH / BASH / FISH / KSH**

```sh
# beadm create 13.1-RC6
# beadm chroot 13.1-RC6
BE # sh || bash || fish || ksh
BE # yes | freebsd-update upgrade -r 13.1-RC6
BE # seq 3 | xargs -I- freebsd-update install
BE # exit
# beadm activate 13.1-RC6
# reboot
```

祝升级顺利 :🙂:
