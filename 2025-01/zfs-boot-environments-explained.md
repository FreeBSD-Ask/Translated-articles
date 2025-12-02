# ZFS 启动环境详解

- [ZFS Boot Environments Explained](https://vermaden.wordpress.com/2025/11/25/zfs-boot-environments-explained/)
- 发布时间：2025/11/25
- 作者：𝚟𝚎𝚛𝚖𝚊𝚍𝚎𝚗


这篇文章不会尝试给出 ZFS 的通用解释——我们只会专注于 *ZFS 启动环境*。关于它们存在许多误解和误会——有些人不知道哪些内容在 ZFS BE 之内、哪些内容在 ZFS BE 之外——但更糟的是——有些人完全不理解它们。

我假设读者确实了解什么是 ZFS 文件系统的概念和特性 —— 比如 ZFS pool 或 ZFS dataset 之类的东西对读者来说是已知的……并且读者能够解释 ZFS 和 LVM 之间的区别。

## ZFS 启动环境里有什么

这是首先经常被误解的主题……大概是因为没有理解 **canmount=off** 这个 ZFS 属性 —— 你可以在 [**zfsprops(7)**](https://man.freebsd.org/zfsprops/7) man page 中阅读更多相关内容。

这是我过去一次关于 “ZFS 启动环境” 的演讲中的幻灯片 —— 可以在链接 [https://is.gd/BECTL](https://is.gd/BECTL) 获得 —— 那是在 **2018 NLUUG** 会议上做的……颜色并不是随便选的……它们是特地准备成那样，以致敬 NLUUG 会议举办的国家 —— 荷兰。

![](https://vermaden.wordpress.com/wp-content/uploads/2025/11/nluug-zfs-canmount.png)

对 **/var** 和 **/usr** 使用 **canmount=off** 背后的想法是这样的：

**/var** 和 **/usr** 这些 ZFS datasets 的内容确实位于 ZFS BE 中 —— 位于安装程序创建的 **zroot/ROOT/default** 这个 ZFS BE 中 —— 而其余像下面这些这样的内容：

```sh
zroot/tmp
zroot/usr/home
zroot/usr/ports
zroot/usr/src
zroot/var/audit
zroot/var/crash
zroot/var/log
zroot/var/mail
zroot/var/tmp
```

这些内容 **被排除在这个 ZFS 启动环境之外**。

因此，默认情况下，一切都包含在 ZFS 启动环境中 —— 如果你想从 **/usr** 或 **/var** 中排除某些部分 —— 你就为这些需要排除的路径添加相应的 ZFS datasets。

下面是你在 [**bsdinstall(8)**](https://man.freebsd.org/bsdinstall/8) 安装程序中选择 **Auto（ZFS）** 选项后，FreeBSD 默认的 ZFS 布局。

```sh
root@freebsd:~ # zfs set mountpoint=none zroot

root@freebsd:~ # zfs list
NAME                 USED  AVAIL  REFER  MOUNTPOINT
zroot                385M  18.5G    96K  none
zroot/ROOT           383M  18.5G    96K  none
zroot/ROOT/default   383M  18.5G   383M  /
zroot/home            96K  18.5G    96K  /home
zroot/tmp             96K  18.5G    96K  /tmp
zroot/usr            288K  18.5G    96K  /usr
zroot/usr/ports       96K  18.5G    96K  /usr/ports
zroot/usr/src         96K  18.5G    96K  /usr/src
zroot/var            600K  18.5G    96K  /var
zroot/var/audit       96K  18.5G    96K  /var/audit
zroot/var/crash       96K  18.5G    96K  /var/crash
zroot/var/log        120K  18.5G   120K  /var/log
zroot/var/mail        96K  18.5G    96K  /var/mail
zroot/var/tmp         96K  18.5G    96K  /var/tmp

root@freebsd:~ # zfs get canmount
NAME                PROPERTY  VALUE     SOURCE
zroot               canmount  on        default
zroot/ROOT          canmount  on        default
zroot/ROOT/default  canmount  noauto    local
zroot/home          canmount  on        default
zroot/tmp           canmount  on        default
zroot/usr           canmount  off       local
zroot/usr/ports     canmount  on        default
zroot/usr/src       canmount  on        default
zroot/var           canmount  off       local
zroot/var/audit     canmount  on        default
zroot/var/crash     canmount  on        default
zroot/var/log       canmount  on        default
zroot/var/mail      canmount  on        default
zroot/var/tmp       canmount  on        default
```

关键点在于，**zroot/usr** 和 **zroot/var** 这两个 ZFS dataset 都具有 **canmount=off** 这个 ZFS 属性。这意味着 **/usr** 文件系统 **并不存放在** **zroot/usr** 这个 ZFS dataset 中……它是存放在 **zroot/ROOT/default** 这个 ZFS 启动环境里的。

你可以用传统的 [**df(1)**](https://man.freebsd.org/df/1) 命令像这样进行验证。

```sh
root@freebsd:~ # df -g /usr
Filesystem         1G-blocks Used Avail Capacity  Mounted on
zroot/ROOT/default        18    0    18     2%    /

root@freebsd:~ # df -g /var
Filesystem         1G-blocks Used Avail Capacity  Mounted on
zroot/ROOT/default        18    0    18     2%    /
```

如你所见，**/usr** 和 **/var** 都只是 **zroot/ROOT/default** 这个 ZFS dataset 中的目录。数据并 **不** 保存在 **zroot/usr** 或 **zroot/var** 这些 ZFS dataset 中……再次强调，这是因为它们具有 **canmount=off** 这个 ZFS 属性。

现在，这个 FreeBSD 默认设置的主要理念 / 概念是：所有内容都保存在 **zroot/ROOT/default** 这个 ZFS dataset 中，而所有其他 ZFS dataset 都是从它中 **排除（EXCLUDE）** 出去的。比如 **zroot/var/audit** 具有 **canmount=on** 这个 ZFS 属性……其他所有的也都是完全相同的情况。

说实话，为了让事情变得傻瓜式地简单，我会把 **zroot/usr** 和 **zroot/var** 这两个 ZFS dataset 换成一个能准确说明用途的名称 —— **zroot/exclude**，因为在它下面的所有 ZFS dataset，本质上都是被排除在 **ZFS 启动环境** 之外的。

```sh
root@freebsd:~ # zfs list
NAME                 USED  AVAIL  REFER  MOUNTPOINT
zroot                385M  18.5G    96K  none
zroot/ROOT           383M  18.5G    96K  none
zroot/ROOT/default   383M  18.5G   383M  /
zroot/home            96K  18.5G    96K  /home
zroot/tmp             96K  18.5G    96K  /tmp
zroot/usr            288K  18.5G    96K  /usr
zroot/usr/ports       96K  18.5G    96K  /usr/ports
zroot/usr/src         96K  18.5G    96K  /usr/src
zroot/var            600K  18.5G    96K  /var
zroot/var/audit       96K  18.5G    96K  /var/audit
zroot/var/crash       96K  18.5G    96K  /var/crash
zroot/var/log        120K  18.5G   120K  /var/log	
zroot/var/mail        96K  18.5G    96K  /var/mail
zroot/var/tmp         96K  18.5G    96K  /var/tmp

root@freebsd:~ # zfs create -o canmount=off -o mountpoint=none zroot/exclude

root@freebsd:~ # zfs rename -u zroot/usr  zroot/exclude/usr
root@freebsd:~ # zfs rename -u zroot/var  zroot/exclude/var
root@freebsd:~ # zfs rename -u zroot/home zroot/exclude/home
root@freebsd:~ # zfs rename -u zroot/tmp  zroot/exclude/tmp

root@freebsd:~ # zfs list
NAME                      USED  AVAIL  REFER  MOUNTPOINT
zroot                     385M  18.5G    96K  none
zroot/ROOT                383M  18.5G    96K  none
zroot/ROOT/default        383M  18.5G   383M  /
zroot/exclude            1.16M  18.5G    96K  none
zroot/exclude/home         96K  18.5G    96K  /home
zroot/exclude/tmp          96K  18.5G    96K  /tmp
zroot/exclude/usr         288K  18.5G    96K  /usr
zroot/exclude/usr/ports    96K  18.5G    96K  /usr/ports
zroot/exclude/usr/src      96K  18.5G    96K  /usr/src
zroot/exclude/var         612K  18.5G    96K  /var
zroot/exclude/var/audit    96K  18.5G    96K  /var/audit
zroot/exclude/var/crash    96K  18.5G    96K  /var/crash
zroot/exclude/var/log     132K  18.5G   132K  /var/log
zroot/exclude/var/mail     96K  18.5G    96K  /var/mail
zroot/exclude/var/tmp      96K  18.5G    96K  /var/tmp
```

现在更容易理解了吗？我希望是的……即使在上面进行了所有这些重命名，ZFS 的挂载点也 **没有任何变化**，因为 **mountpoint** 是一个独立的 ZFS 属性——所以即使我们对 ZFS dataset 的命名逻辑进行了重新组织，只要没有从上层继承，这个 ZFS **mountpoint** 属性在逻辑重组后依然保持不变。

之前使用 **df(1)** 的测试结果与之前完全相同。

```sh
root@freebsd:~ # df -g /usr
Filesystem         1G-blocks Used Avail Capacity  Mounted on
zroot/ROOT/default        18    0    18     2%    /

root@freebsd:~ # df -g /var
Filesystem         1G-blocks Used Avail Capacity  Mounted on
zroot/ROOT/default        18    0    18     2%    /
```

**/usr** 和 **/var** 的数据都保存在 **default** ZFS 启动环境内。

我们甚至可以移除 **/usr** 和 **/var** 的挂载点，这样会更加清晰。

```sh
root@freebsd:~ # zfs set -u mountpoint=/var/tmp   zroot/exclude/var/tmp
root@freebsd:~ # zfs set -u mountpoint=/var/mail  zroot/exclude/var/mail
root@freebsd:~ # zfs set -u mountpoint=/var/log   zroot/exclude/var/log
root@freebsd:~ # zfs set -u mountpoint=/var/crash zroot/exclude/var/crash
root@freebsd:~ # zfs set -u mountpoint=/var/audit zroot/exclude/var/audit
root@freebsd:~ # zfs set -u mountpoint=none       zroot/exclude/var

root@freebsd:~ # zfs set -u mountpoint=/usr/src   zroot/exclude/usr/src
root@freebsd:~ # zfs set -u mountpoint=/usr/ports zroot/exclude/usr/ports
root@freebsd:~ # zfs set -u mountpoint=none       zroot/exclude/usr

root@freebsd:~ # zfs list
NAME                      USED  AVAIL  REFER  MOUNTPOINT
zroot                     385M  18.5G    96K  none
zroot/ROOT                383M  18.5G    96K  none
zroot/ROOT/default        383M  18.5G   383M  /
zroot/exclude            1.16M  18.5G    96K  none
zroot/exclude/home         96K  18.5G    96K  /home
zroot/exclude/tmp          96K  18.5G    96K  /tmp
zroot/exclude/usr         288K  18.5G    96K  none
zroot/exclude/usr/ports    96K  18.5G    96K  /usr/ports
zroot/exclude/usr/src      96K  18.5G    96K  /usr/src
zroot/exclude/var         612K  18.5G    96K  none
zroot/exclude/var/audit    96K  18.5G    96K  /var/audit
zroot/exclude/var/crash    96K  18.5G    96K  /var/crash
zroot/exclude/var/log     132K  18.5G   132K  /var/log
zroot/exclude/var/mail     96K  18.5G    96K  /var/mail
zroot/exclude/var/tmp      96K  18.5G    96K  /var/tmp

root@freebsd:~ # df -g
Filesystem              1G-blocks Used Avail Capacity  Mounted on
/dev/gpt/efiboot0               0    0     0     0%    /boot/efi
devfs                           0    0     0     0%    /dev
zroot/ROOT/default             18    0    18     2%    /
zroot/exclude/tmp              18    0    18     0%    /tmp
zroot/exclude/home             18    0    18     0%    /home
zroot/exclude/var/log          18    0    18     0%    /var/log
zroot/exclude/var/tmp          18    0    18     0%    /var/tmp
zroot/exclude/var/mail         18    0    18     0%    /var/mail
zroot/exclude/var/crash        18    0    18     0%    /var/crash
zroot/exclude/var/audit        18    0    18     0%    /var/audit
zroot/exclude/usr/src          18    0    18     0%    /usr/src
zroot/exclude/usr/ports        18    0    18     0%    /usr/ports
```

够清楚了吗？

## 新建 ZFS 启动环境时会发生什么

另一个经常被误解的谜题……或者说没有被清楚理解的部分。

通常，*ZFS 启动环境* 是由某个时间点创建的 ZFS **快照** 克隆出来的可写 ZFS **clone**。

让我用我最喜欢的 *Enterprise Architect ASCII Edition* 软件来可视化一下。

```sh
                                 |
                    ZFS DATASET  |   ZFS MOUNTPOINT => WHY
                                 |
          +-------------------+  |
          | ZFS 'zroot' pool  |  |  /sys            => # zfs set mountpoint=/sys sys
          |     (dataset)     |  |  canmount:on     => # zfs set canmount=on     sys
          +---------+---------+  |
                    |            |
            +-------+-------+    |
            |     ROOT      |    |  (none)          => # zfs set mountpoint=none sys/ROOT
            |   (dataset)   |    |  canmount:on     => # zfs set canmount=on     sys/ROOT
            +-------+-------+    |
                    |            |
              +-----+-----+      |
              |  default  |      |  /               => # zfs set mountpoint=/    sys/ROOT/${DATASET}
              | (dataset) |      |  canmount:noauto => # zfs set canmount=noauto sys/ROOT/${DATASET}
           +--+-----------+      |
           |                     |
           +- @2025-11-11@10:10  |  point-in-time   => # zfs snapshot            sys/ROOT/${DATASET}@2025-11-11@10:10
              | (snapshot)       |  (read only)        # beadm create            sys/ROOT/${DATASET}@2025-11-11@10:10
              |                  |
              +- safe            |  clone           => # zfs clone               sys/ROOT/${DATASET}@2025-11-11@10:10 sys/ROOT/safe
              | (clone)          |  (writable)         # beadm create -e default@2025-11-11@10:10 safe
              |                  |
              +- test            |  clone           => # zfs clone               sys/ROOT/${DATASET}@2025-11-11@10:10 sys/ROOT/test
                (clone)          |  (writable)         # beadm create -e default@2025-11-11@10:10 test
                                 |
```

关于上面的 ASCII 图，**beadm(8)** 命令的输出将显示类似的信息。

```sh
root@freebsd:~ # beadm list
BE                  Active Mountpoint  Space Created
default             NR     /           24.0G 2025-10-08 01:42
safe                -      -            1.3G 2025-06-10 09:47
test                -      -            6.4G 2025-08-22 23:02

root@freebsd:~ # zfs list -r -t all zroot/ROOT
NAME                                                 USED  AVAIL  REFER  MOUNTPOINT
zroot/ROOT                                          29.8G   154G    96K  none
zroot/ROOT/default                                   748K   154G  19.5G  /
zroot/ROOT/default@2025-11-11@10:10                 16.1G      -  24.2G  -
zroot/ROOT/safe                                        8K   154G  20.5G  /
zroot/ROOT/test                                      810M   154G  20.9G  /

root@freebsd:~ # beadm list -a
BE/Dataset/Snapshot                Active Mountpoint  Space Created

default
  zroot/ROOT/default               NR     /           24.0G 2025-10-08 01:42

safe
  zroot/ROOT/safe                  -      -          748.0K 2025-06-10 09:47
    default@2025-11-11@10:10       -      -            1.3G 2025-10-08 01:42

test
  zroot/ROOT/test                  -      -          810.0M 2025-08-22 23:02
    default@2025-11-11@10:10        -      -            5.6G 2025-08-22 23:02


root@freebsd:~ # zfs get origin zroot/ROOT/default
NAME                PROPERTY  VALUE   SOURCE
zroot/ROOT/default  origin    -       -

root@freebsd:~ # zfs get origin zroot/ROOT/safe
NAME             PROPERTY  VALUE                                SOURCE
zroot/ROOT/safe  origin    zroot/ROOT/default@2025-11-11@10:10  -

root@freebsd:~ # zfs get origin zroot/ROOT/test
NAME             PROPERTY  VALUE                                SOURCE
zroot/ROOT/test  origin    zroot/ROOT/default@2025-11-11@10:10  -

root@freebsd:~ # zpool get bootfs
NAME   PROPERTY  VALUE                SOURCE
zdata  bootfs    -                    default
zroot  bootfs    zroot/ROOT/default   local
```

现在……有趣的部分来了——你可以创建任意数量的额外 *ZFS 启动环境*，或者使用 ZFS 的 **send|recv** 功能。

下面是这种设置的示例。

```sh
                                        |
                       ZFS DATASET      |       MOUNTPOINT => WHY
                                        |
         +-----------------------+      |
         |    ZFS 'zroot' pool   |      |  /sys            => # zfs set mountpoint=/sys sys
         |     (ZFS dataset)     |      |  canmount:on     => # zfs set canmount=on     sys
         +-----------+-----------+      |
                     |                  |
           +---------+---------+        |
           |       ROOT        |        |  (none)          => # zfs set mountpoint=none sys/ROOT
           |   (ZFS dataset)   |        |  canmount:on     => # zfs set canmount=on     sys/ROOT
           +---------+---------+        |
                     |                  |
      +--------------+----------+       |
      |              |          |       |
 +----+----+ +-------+---+ +----+----+  |
 | default | |  15.0-RC1 | |  12.2   |  |  /               => # zfs set mountpoint=/    sys/ROOT/${DATASET}
 |(dataset)| | (dataset) | |(dataset)|  |  canmount:noauto => # zfs set canmount=noauto sys/ROOT/${DATASET}
 +---------+ +-----------+ +---------+  |
                                        |
```

……并且你可以从每一个这些 ZFS dataset 创建额外的 ZFS 启动环境。

## 在系统之间传输 ZFS 启动环境

……而且这些系统可以是 **任意** 系统。你可以在笔记本之间传输 ZFS 启动环境，或者从笔记本传到服务器……甚至从 Bhyve VM 传到服务器……随你喜欢。

下面你将看到一些示例，展示如何将现有的 ZFS 启动环境通过网络从一个 FreeBSD UNIX 系统复制到另一个系统。

这里的 **mbuffer(1)** 并非关键——它只是通过确保随时有数据可发送来加快传输过程。

```sh
root@freebsd:~ # beadm export 14.3 | mbuffer | ssh 10.26 doas beadm import 14.3.w520
summary: 30.3 GiByte in 34min 04.8sec - average of 15.2 MiB/s
```

上面演示的是将我在 **ThinkPad W520** 上的 **14.3** ZFS 启动环境发送到 **ThinkPad T14** 系统——这也是为什么我想在另一端将其命名为 **14.3.w520**，以确保我记得它的来源。

你也可以像我在文章 [Other FreeBSD Version in ZFS Boot Environment](https://vermaden.wordpress.com/2021/10/19/other-freebsd-version-in-zfs-boot-environment/) 中描述的那样，从零创建新的 ZFS 启动环境。

## 总结

我希望现在 *ZFS 启动环境* 的主要原理更加清晰了。

……如果还有不明白的地方，欢迎提出你的疑问。
