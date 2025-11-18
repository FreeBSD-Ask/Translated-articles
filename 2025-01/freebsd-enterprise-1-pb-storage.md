# FreeBSD 企业级 1 PB 存储

- [FreeBSD Enterprise 1 PB Storage](https://vermaden.wordpress.com/2019/06/19/freebsd-enterprise-1-pb-storage/)
- 作者：𝚟𝚎𝚛𝚖𝚊𝚍𝚎𝚗
- 2019/06

今天 FreeBSD 操作系统迎来了 26 周岁生日。6 月 19 日是 **国际 FreeBSD 日**。所以今天我准备了一些特别的内容 🙂。如何在真实硬件上使用 FreeBSD 构建企业级存储解决方案？这正是 FreeBSD 发挥其所有存储特性（包括 ZFS）优势的地方。

今天我将展示我如何基于 FreeBSD 系统构建所谓的企业级存储，同时提供逾 1 PB（拍字节）的原始存储容量。

我曾基于 FreeBSD 构建过各种存储相关系统：

* [FreeBSD 上基于 Minio 的分布式对象存储](https://vermaden.wordpress.com/2018/04/16/distributed-object-storage-with-minio-on-freebsd/)
* [FreeBSD 上的 GlusterFS 集群，结合 Ansible 和 GNU Parallel](https://vermaden.wordpress.com/2019/01/07/glusterfs-cluster-on-freebsd-with-ansible-and-gnu-parallel/)
* [静音无风扇 FreeBSD 服务器 —— 冗余备份](https://vermaden.wordpress.com/2019/04/03/silent-fanless-freebsd-server-redundant-backup/)

这个项目有所不同。一台 4U 服务器最多能提供多少存储空间？事实证明，非常多！绝对大于 1 PB（1024 TB）的原始存储空间。

## 硬件

这些是 4U 服务器，带有 90-100 个 3.5 寸硬盘位，可以安装 1260-1400 TB 数据（使用 14 TB 硬盘）。此类系统示例有：

* [TYAN Thunder SX FA100](https://www.tyan.com/Barebones_FA100B7118_B7118F100V100HR)（100 个硬盘位）
* [Supermicro SuperStorage 6048R-E1CR90L](https://www.supermicro.com/products/system/4U/6048/SSG-6048R-E1CR90L.cfm)（90 个硬盘位）

我会选择第一款 —— 简称 TYAN FA100。

![logo-tyan.png](https://vermaden.wordpress.com/wp-content/uploads/2019/06/logo-tyan.png?w=960)

虽然之前的 *GlusterFS* 和 *Minio* 集群是在虚拟硬件（甚至 FreeBSD Jail 容器）上搭建的，但这个项目使用了真实物理硬件。

该系统的规格如下：

```
2 x 10 核 Intel Xeon Silver 4114 CPU @ 2.20GHz
4 x 32 GB DDR4 内存（总计 128 GB）
2 x Intel SSD DC S3500 240 GB（系统盘）
90 x Toshiba HDD MN07ACA12TE 12 TB（数据盘）
2 x Broadcom SAS3008 控制器
2 x Intel X710 DA-2 10GE 网卡
2 x 电源
```

整套系统价格约为 **$65,000**（含硬盘）。外观如下：

![tyan-fa100-small.jpg](https://vermaden.wordpress.com/wp-content/uploads/2019/06/tyan-fa100-small.jpg?w=960)

你需要长达 1200 mm 的机架机柜来放置这台巨兽 :🙂:

## 管理界面

所谓的 *Lights Out* 管理界面非常优秀。界面简洁、组织良好、响应迅速。可以创建多个独立用户账户，也可以连接外部用户服务，如 LDAP/AD/RADIUS。

![n01.png](https://vermaden.wordpress.com/wp-content/uploads/2019/06/n01.png?w=960)

登录后，简洁的 *Dashboard* 在欢迎我们。

![n02.png](https://vermaden.wordpress.com/wp-content/uploads/2019/06/n02.png?w=960)

可以获取各种 *传感器* 信息，包括系统组件温度。

![n03](https://vermaden.wordpress.com/wp-content/uploads/2019/06/n03.png?w=960)

还可以查看 *系统库存* 信息，了解已安装硬件。

![n04.png](https://vermaden.wordpress.com/wp-content/uploads/2019/06/n04.png?w=960)

有独立的 *设置* 菜单，用于各种配置选项。

![n05.png](https://vermaden.wordpress.com/wp-content/uploads/2019/06/n05.png?w=960)

虽然是 2019 年，但只需 HTML5 的 *远程控制*（远程控制台），无需任何第三方插件如 Java/Silverlight/Flash 等，非常方便，也运行良好。

![n06.png](https://vermaden.wordpress.com/wp-content/uploads/2019/06/n06.png?w=960)

![n07.png](https://vermaden.wordpress.com/wp-content/uploads/2019/06/n07.png?w=960)

当然可以远程开关机或循环重启主机。

![n08.png](https://vermaden.wordpress.com/wp-content/uploads/2019/06/n08.png?w=960)

*维护* 菜单用于 BIOS 更新。

![n09.png](https://vermaden.wordpress.com/wp-content/uploads/2019/06/n09.png?w=960)

## BIOS/UEFI

进入 BIOS/UEFI 后，可以选择从哪些硬盘启动。截图中显示为两块固态系统盘。

![nas01.png](https://vermaden.wordpress.com/wp-content/uploads/2019/06/nas01.png?w=960)

BIOS/UEFI 界面显示两个 *Enclosures*，实际上是两块 Broadcom SAS3008 控制器。一些硬盘连接到第一个 SAS 控制器，其余连接到第二个，他们称之为 *Enclosures* 而非控制器，原因不明。

![nas05.png](https://vermaden.wordpress.com/wp-content/uploads/2019/06/nas05.png?w=960)

## FreeBSD 系统

我选择了最新的 FreeBSD 12.0-RELEASE 来进行安装。安装过程多“默认”，系统盘使用两块 SSD 做 ZFS 镜像，没有特别配置。

![logo-freebsd.jpg](https://vermaden.wordpress.com/wp-content/uploads/2019/06/logo-freebsd.jpg?w=960)

该安装当然支持 [ZFS Boot Environments](https://vermaden.wordpress.com/2018/11/15/zfs-boot-environments-reloaded-at-nluug-autumn-conference-2018/) 功能，可实现安全升级和系统变更。

```sh
# zpool list zroot
NAME    SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP  HEALTH  ALTROOT
zroot   220G  3.75G   216G        -         -     0%     1%  1.00x  ONLINE  -

# zpool status zroot
  pool: zroot
 state: ONLINE
  scan: none requested
config:

        NAME        STATE     READ WRITE CKSUM
        zroot       ONLINE       0     0     0
          mirror-0  ONLINE       0     0     0
            da91p4  ONLINE       0     0     0
            da11p4  ONLINE       0     0     0

errors: No known data errors

# df -g
Filesystem              1G-blocks Used  Avail Capacity  Mounted on
zroot/ROOT/default            211    2    209     1%    /
devfs                           0    0      0   100%    /dev
zroot/tmp                     209    0    209     0%    /tmp
zroot/usr/home                209    0    209     0%    /usr/home
zroot/usr/ports               210    0    209     0%    /usr/ports
zroot/usr/src                 210    0    209     0%    /usr/src
zroot/var/audit               209    0    209     0%    /var/audit
zroot/var/crash               209    0    209     0%    /var/crash
zroot/var/log                 209    0    209     0%    /var/log
zroot/var/mail                209    0    209     0%    /var/mail
zroot/var/tmp                 209    0    209     0%    /var/tmp

# beadm list
BE      Active Mountpoint  Space Created
default NR     /            2.4G 2019-05-24 13:24
```

## 硬盘准备

在所有 90 块 12 TB 硬盘的可能方案中，我选择使用 RAID60 —— 当然这是 ZFS 的等效方案。每个 RAID6（raidz2）组使用 12 块硬盘，总共会有 7 个这样的组 —— 这样 ZFS 池将使用 84 块硬盘，剩下 6 块作为备用（SPARE）硬盘 —— 对我来说非常合适。硬盘分布大致如下。

```sh
DISKS  CONTENT
   12  raidz2-0
   12  raidz2-1
   12  raidz2-2
   12  raidz2-3
   12  raidz2-4
   12  raidz2-5
   12  raidz2-6
    6  spares
   90  TOTAL
```

下面是 FreeBSD 系统通过命令 **camcontrol(8)** 看到的这些硬盘情况。按连接的 SAS 控制器 —— **scbus(4)** 排序。


```sh
# camcontrol devlist | sort -k 6
(AHCI SGPIO Enclosure 1.00 0001)   at scbus2 target 0 lun 0 (pass0,ses0)
(ATA TOSHIBA MG07ACA1 0101)        at scbus3 target 50 lun 0 (pass1,da0)
(ATA TOSHIBA MG07ACA1 0101)        at scbus3 target 52 lun 0 (pass2,da1)
(ATA TOSHIBA MG07ACA1 0101)        at scbus3 target 54 lun 0 (pass3,da2)
(ATA TOSHIBA MG07ACA1 0101)        at scbus3 target 56 lun 0 (pass5,da4)
(ATA TOSHIBA MG07ACA1 0101)        at scbus3 target 57 lun 0 (pass6,da5)
(ATA TOSHIBA MG07ACA1 0101)        at scbus3 target 59 lun 0 (pass7,da6)
(ATA TOSHIBA MG07ACA1 0101)        at scbus3 target 60 lun 0 (pass8,da7)
(ATA TOSHIBA MG07ACA1 0101)        at scbus3 target 66 lun 0 (pass9,da8)
(ATA TOSHIBA MG07ACA1 0101)        at scbus3 target 67 lun 0 (pass10,da9)
(ATA TOSHIBA MG07ACA1 0101)        at scbus3 target 74 lun 0 (pass11,da10)
(ATA INTEL SSDSC2KB24 0100)        at scbus3 target 75 lun 0 (pass12,da11)
(ATA TOSHIBA MG07ACA1 0101)        at scbus3 target 76 lun 0 (pass13,da12)
(ATA TOSHIBA MG07ACA1 0101)        at scbus3 target 82 lun 0 (pass14,da13)
(ATA TOSHIBA MG07ACA1 0101)        at scbus3 target 83 lun 0 (pass15,da14)
(ATA TOSHIBA MG07ACA1 0101)        at scbus3 target 85 lun 0 (pass16,da15)
(ATA TOSHIBA MG07ACA1 0101)        at scbus3 target 87 lun 0 (pass17,da16)
(Tyan B7118 0500)                  at scbus3 target 88 lun 0 (pass18,ses1)
(ATA TOSHIBA MG07ACA1 0101)        at scbus3 target 89 lun 0 (pass19,da17)
(ATA TOSHIBA MG07ACA1 0101)        at scbus3 target 90 lun 0 (pass20,da18)
(ATA TOSHIBA MG07ACA1 0101)        at scbus3 target 91 lun 0 (pass21,da19)
(ATA TOSHIBA MG07ACA1 0101)        at scbus3 target 92 lun 0 (pass22,da20)
(ATA TOSHIBA MG07ACA1 0101)        at scbus3 target 93 lun 0 (pass23,da21)
(ATA TOSHIBA MG07ACA1 0101)        at scbus3 target 94 lun 0 (pass24,da22)
(ATA TOSHIBA MG07ACA1 0101)        at scbus3 target 95 lun 0 (pass25,da23)
(ATA TOSHIBA MG07ACA1 0101)        at scbus3 target 96 lun 0 (pass26,da24)
(ATA TOSHIBA MG07ACA1 0101)        at scbus3 target 97 lun 0 (pass27,da25)
(ATA TOSHIBA MG07ACA1 0101)        at scbus3 target 98 lun 0 (pass28,da26)
(ATA TOSHIBA MG07ACA1 0101)        at scbus3 target 99 lun 0 (pass29,da27)
(ATA TOSHIBA MG07ACA1 0101)        at scbus3 target 100 lun 0 (pass30,da28)
(ATA TOSHIBA MG07ACA1 0101)        at scbus3 target 101 lun 0 (pass31,da29)
(ATA TOSHIBA MG07ACA1 0101)        at scbus3 target 102 lun 0 (pass32,da30)
(ATA TOSHIBA MG07ACA1 0101)        at scbus3 target 103 lun 0 (pass33,da31)
(ATA TOSHIBA MG07ACA1 0101)        at scbus3 target 104 lun 0 (pass34,da32)
(ATA TOSHIBA MG07ACA1 0101)        at scbus3 target 105 lun 0 (pass35,da33)
(ATA TOSHIBA MG07ACA1 0101)        at scbus3 target 106 lun 0 (pass36,da34)
(ATA TOSHIBA MG07ACA1 0101)        at scbus3 target 107 lun 0 (pass37,da35)
(ATA TOSHIBA MG07ACA1 0101)        at scbus3 target 108 lun 0 (pass38,da36)
(ATA TOSHIBA MG07ACA1 0101)        at scbus3 target 109 lun 0 (pass39,da37)
(ATA TOSHIBA MG07ACA1 0101)        at scbus3 target 110 lun 0 (pass40,da38)
(ATA TOSHIBA MG07ACA1 0101)        at scbus4 target 48 lun 0 (pass41,da39)
(ATA TOSHIBA MG07ACA1 0101)        at scbus4 target 49 lun 0 (pass42,da40)
(ATA TOSHIBA MG07ACA1 0101)        at scbus4 target 51 lun 0 (pass43,da41)
(ATA TOSHIBA MG07ACA1 0101)        at scbus4 target 53 lun 0 (pass44,da42)
(ATA TOSHIBA MG07ACA1 0101)        at scbus4 target 55 lun 0 (da43,pass45)
(ATA TOSHIBA MG07ACA1 0101)        at scbus4 target 59 lun 0 (pass46,da44)
(ATA TOSHIBA MG07ACA1 0101)        at scbus4 target 64 lun 0 (pass47,da45)
(ATA TOSHIBA MG07ACA1 0101)        at scbus4 target 67 lun 0 (pass48,da46)
(ATA TOSHIBA MG07ACA1 0101)        at scbus4 target 68 lun 0 (pass49,da47)
(ATA TOSHIBA MG07ACA1 0101)        at scbus4 target 69 lun 0 (pass50,da48)
(ATA TOSHIBA MG07ACA1 0101)        at scbus4 target 73 lun 0 (pass51,da49)
(ATA TOSHIBA MG07ACA1 0101)        at scbus4 target 76 lun 0 (pass52,da50)
(ATA TOSHIBA MG07ACA1 0101)        at scbus4 target 77 lun 0 (pass53,da51)
(Tyan B7118 0500)                  at scbus4 target 80 lun 0 (pass54,ses2)
(ATA TOSHIBA MG07ACA1 0101)        at scbus4 target 81 lun 0 (pass55,da52)
(ATA TOSHIBA MG07ACA1 0101)        at scbus4 target 82 lun 0 (pass56,da53)
(ATA TOSHIBA MG07ACA1 0101)        at scbus4 target 83 lun 0 (pass57,da54)
(ATA TOSHIBA MG07ACA1 0101)        at scbus4 target 84 lun 0 (pass58,da55)
(ATA TOSHIBA MG07ACA1 0101)        at scbus4 target 85 lun 0 (pass59,da56)
(ATA TOSHIBA MG07ACA1 0101)        at scbus4 target 86 lun 0 (pass60,da57)
(ATA TOSHIBA MG07ACA1 0101)        at scbus4 target 87 lun 0 (pass61,da58)
(ATA TOSHIBA MG07ACA1 0101)        at scbus4 target 88 lun 0 (pass62,da59)
(ATA TOSHIBA MG07ACA1 0101)        at scbus4 target 89 lun 0 (da63,pass66)
(ATA TOSHIBA MG07ACA1 0101)        at scbus4 target 90 lun 0 (pass64,da61)
(ATA TOSHIBA MG07ACA1 0101)        at scbus4 target 91 lun 0 (pass65,da62)
(ATA TOSHIBA MG07ACA1 0101)        at scbus4 target 92 lun 0 (da60,pass63)
(ATA TOSHIBA MG07ACA1 0101)        at scbus4 target 94 lun 0 (pass67,da64)
(ATA TOSHIBA MG07ACA1 0101)        at scbus4 target 97 lun 0 (pass68,da65)
(ATA TOSHIBA MG07ACA1 0101)        at scbus4 target 98 lun 0 (pass69,da66)
(ATA TOSHIBA MG07ACA1 0101)        at scbus4 target 99 lun 0 (pass70,da67)
(ATA TOSHIBA MG07ACA1 0101)        at scbus4 target 100 lun 0 (pass71,da68)
(Tyan B7118 0500)                  at scbus4 target 101 lun 0 (pass72,ses3)
(ATA TOSHIBA MG07ACA1 0101)        at scbus4 target 102 lun 0 (pass73,da69)
(ATA TOSHIBA MG07ACA1 0101)        at scbus4 target 103 lun 0 (pass74,da70)
(ATA TOSHIBA MG07ACA1 0101)        at scbus4 target 104 lun 0 (pass75,da71)
(ATA TOSHIBA MG07ACA1 0101)        at scbus4 target 105 lun 0 (pass76,da72)
(ATA TOSHIBA MG07ACA1 0101)        at scbus4 target 106 lun 0 (pass77,da73)
(ATA TOSHIBA MG07ACA1 0101)        at scbus4 target 107 lun 0 (pass78,da74)
(ATA TOSHIBA MG07ACA1 0101)        at scbus4 target 108 lun 0 (pass79,da75)
(ATA TOSHIBA MG07ACA1 0101)        at scbus4 target 109 lun 0 (pass80,da76)
(ATA TOSHIBA MG07ACA1 0101)        at scbus4 target 110 lun 0 (pass81,da77)
(ATA TOSHIBA MG07ACA1 0101)        at scbus4 target 111 lun 0 (pass82,da78)
(ATA TOSHIBA MG07ACA1 0101)        at scbus4 target 112 lun 0 (pass83,da79)
(ATA TOSHIBA MG07ACA1 0101)        at scbus4 target 113 lun 0 (pass84,da80)
(ATA TOSHIBA MG07ACA1 0101)        at scbus4 target 114 lun 0 (pass85,da81)
(ATA TOSHIBA MG07ACA1 0101)        at scbus4 target 115 lun 0 (pass86,da82)
(ATA TOSHIBA MG07ACA1 0101)        at scbus4 target 116 lun 0 (pass87,da83)
(ATA TOSHIBA MG07ACA1 0101)        at scbus4 target 117 lun 0 (pass88,da84)
(ATA TOSHIBA MG07ACA1 0101)        at scbus4 target 118 lun 0 (pass89,da85)
(ATA TOSHIBA MG07ACA1 0101)        at scbus4 target 119 lun 0 (pass90,da86)
(ATA TOSHIBA MG07ACA1 0101)        at scbus4 target 120 lun 0 (pass91,da87)
(ATA TOSHIBA MG07ACA1 0101)        at scbus4 target 121 lun 0 (pass92,da88)
(ATA TOSHIBA MG07ACA1 0101)        at scbus4 target 122 lun 0 (pass93,da89)
(ATA TOSHIBA MG07ACA1 0101)        at scbus4 target 123 lun 0 (pass94,da90)
(ATA INTEL SSDSC2KB24 0100)        at scbus4 target 124 lun 0 (pass95,da91)
(ATA TOSHIBA MG07ACA1 0101)        at scbus4 target 125 lun 0 (da3,pass4)
```

有人可能会问，当硬盘出现故障时如何识别是哪一块……这时 FreeBSD 的 **<sesutil(8)>** 命令就非常有用了。

```sh
# sesutil locate all off
# sesutil locate da64 on
```

第一条 **sesutil(8)** 命令关闭机箱中所有硬盘的位置指示灯。第二条命令则点亮了 **da64** 硬盘的识别灯。

我还会确保不会使用每块硬盘的全部容量。这个想法看似无意义，但设想以下情况：五块 12 TB 硬盘在三年后同时损坏，你无法获得相同型号的硬盘，只能用其他 12 TB 硬盘替代，甚至可能来自不同厂商。

```sh
# grep da64 /var/run/dmesg.boot
da64 at mpr1 bus 0 scbus4 target 93 lun 0
da64:  Fixed Direct Access SPC-4 SCSI device
da64: Serial Number 98G0A1EQF95G
da64: 1200.000MB/s transfers
da64: Command Queueing enabled
da64: 11444224MB (23437770752 512 byte sectors)
```

单块 12 TB 硬盘共有 **23437770752** 个 **512** 字节的扇区，总原始容量为 **12000138625024** 字节。

```sh
# expr 23437770752 \* 512
12000138625024
```

现在假设这些来自其他厂商的 12 TB 硬盘每块比原来的少 4 字节 —— ZFS 将不允许使用它们，因为容量不足。

因此我会将每块硬盘的容量设置为 **11175 GB**，大约比其总容量 **11176 GB** 少 1 GB。

下面是可以对所有 90 块硬盘执行此操作的命令。

```sh
# camcontrol devlist \
    | grep TOSHIBA \
    | awk '{print $NF}' \
    | awk -F ',' '{print $2}' \
    | tr -d ')' \
    | while read DISK
      do
        gpart destroy -F                   ${DISK} 1> /dev/null 2> /dev/null
        gpart create -s GPT                ${DISK}
        gpart add -t freebsd-zfs -s 11175G ${DISK}
      done

# gpart show da64
=>         40  23437770672  da64  GPT  (11T)
           40  23435673600     1  freebsd-zfs  (11T)
  23435673640      2097072        - free -  (1.0G)
```

## 配置 ZFS 池

接下来，我们需要创建 ZFS 池，这可能是我执行过的最长的 **zpool** 命令了 :🙂:

由于东芝 12 TB 硬盘使用 4k 扇区，我们需要将 **vfs.zfs.min_auto_ashift** 设置为 12 来强制使用该扇区大小。

```sh
# sysctl vfs.zfs.min_auto_ashift=12
vfs.zfs.min_auto_ashift: 12 -> 12

# zpool create nas02 \
    raidz2  da0p1  da1p1  da2p1  da3p1  da4p1  da5p1  da6p1  da7p1  da8p1  da9p1 da10p1 da12p1 \
    raidz2 da13p1 da14p1 da15p1 da16p1 da17p1 da18p1 da19p1 da20p1 da21p1 da22p1 da23p1 da24p1 \
    raidz2 da25p1 da26p1 da27p1 da28p1 da29p1 da30p1 da31p1 da32p1 da33p1 da34p1 da35p1 da36p1 \
    raidz2 da37p1 da38p1 da39p1 da40p1 da41p1 da42p1 da43p1 da44p1 da45p1 da46p1 da47p1 da48p1 \
    raidz2 da49p1 da50p1 da51p1 da52p1 da53p1 da54p1 da55p1 da56p1 da57p1 da58p1 da59p1 da60p1 \
    raidz2 da61p1 da62p1 da63p1 da64p1 da65p1 da66p1 da67p1 da68p1 da69p1 da70p1 da71p1 da72p1 \
    raidz2 da73p1 da74p1 da75p1 da76p1 da77p1 da78p1 da79p1 da80p1 da81p1 da82p1 da83p1 da84p1 \
    spare  da85p1 da86p1 da87p1 da88p1 da89p1 da90p1

# zpool status
  pool: nas02
 state: ONLINE
  scan: scrub repaired 0 in 0 days 00:00:05 with 0 errors on Fri May 31 10:26:29 2019
config:

        NAME        STATE     READ WRITE CKSUM
        nas02       ONLINE       0     0     0
          raidz2-0  ONLINE       0     0     0
            da0p1   ONLINE       0     0     0
            da1p1   ONLINE       0     0     0
            da2p1   ONLINE       0     0     0
            da3p1   ONLINE       0     0     0
            da4p1   ONLINE       0     0     0
            da5p1   ONLINE       0     0     0
            da6p1   ONLINE       0     0     0
            da7p1   ONLINE       0     0     0
            da8p1   ONLINE       0     0     0
            da9p1   ONLINE       0     0     0
            da10p1  ONLINE       0     0     0
            da12p1  ONLINE       0     0     0
          raidz2-1  ONLINE       0     0     0
            da13p1  ONLINE       0     0     0
            da14p1  ONLINE       0     0     0
            da15p1  ONLINE       0     0     0
            da16p1  ONLINE       0     0     0
            da17p1  ONLINE       0     0     0
            da18p1  ONLINE       0     0     0
            da19p1  ONLINE       0     0     0
            da20p1  ONLINE       0     0     0
            da21p1  ONLINE       0     0     0
            da22p1  ONLINE       0     0     0
            da23p1  ONLINE       0     0     0
            da24p1  ONLINE       0     0     0
          raidz2-2  ONLINE       0     0     0
            da25p1  ONLINE       0     0     0
            da26p1  ONLINE       0     0     0
            da27p1  ONLINE       0     0     0
            da28p1  ONLINE       0     0     0
            da29p1  ONLINE       0     0     0
            da30p1  ONLINE       0     0     0
            da31p1  ONLINE       0     0     0
            da32p1  ONLINE       0     0     0
            da33p1  ONLINE       0     0     0
            da34p1  ONLINE       0     0     0
            da35p1  ONLINE       0     0     0
            da36p1  ONLINE       0     0     0
          raidz2-3  ONLINE       0     0     0
            da37p1  ONLINE       0     0     0
            da38p1  ONLINE       0     0     0
            da39p1  ONLINE       0     0     0
            da40p1  ONLINE       0     0     0
            da41p1  ONLINE       0     0     0
            da42p1  ONLINE       0     0     0
            da43p1  ONLINE       0     0     0
            da44p1  ONLINE       0     0     0
            da45p1  ONLINE       0     0     0
            da46p1  ONLINE       0     0     0
            da47p1  ONLINE       0     0     0
            da48p1  ONLINE       0     0     0
          raidz2-4  ONLINE       0     0     0
            da49p1  ONLINE       0     0     0
            da50p1  ONLINE       0     0     0
            da51p1  ONLINE       0     0     0
            da52p1  ONLINE       0     0     0
            da53p1  ONLINE       0     0     0
            da54p1  ONLINE       0     0     0
            da55p1  ONLINE       0     0     0
            da56p1  ONLINE       0     0     0
            da57p1  ONLINE       0     0     0
            da58p1  ONLINE       0     0     0
            da59p1  ONLINE       0     0     0
            da60p1  ONLINE       0     0     0
          raidz2-5  ONLINE       0     0     0
            da61p1  ONLINE       0     0     0
            da62p1  ONLINE       0     0     0
            da63p1  ONLINE       0     0     0
            da64p1  ONLINE       0     0     0
            da65p1  ONLINE       0     0     0
            da66p1  ONLINE       0     0     0
            da67p1  ONLINE       0     0     0
            da68p1  ONLINE       0     0     0
            da69p1  ONLINE       0     0     0
            da70p1  ONLINE       0     0     0
            da71p1  ONLINE       0     0     0
            da72p1  ONLINE       0     0     0
          raidz2-6  ONLINE       0     0     0
            da73p1  ONLINE       0     0     0
            da74p1  ONLINE       0     0     0
            da75p1  ONLINE       0     0     0
            da76p1  ONLINE       0     0     0
            da77p1  ONLINE       0     0     0
            da78p1  ONLINE       0     0     0
            da79p1  ONLINE       0     0     0
            da80p1  ONLINE       0     0     0
            da81p1  ONLINE       0     0     0
            da82p1  ONLINE       0     0     0
            da83p1  ONLINE       0     0     0
            da84p1  ONLINE       0     0     0
        spares
          da85p1    AVAIL
          da86p1    AVAIL
          da87p1    AVAIL
          da88p1    AVAIL
          da89p1    AVAIL
          da90p1    AVAIL

errors: No known data errors

# zpool list nas02
NAME    SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP  HEALTH  ALTROOT
nas02   915T  1.42M   915T        -         -     0%     0%  1.00x  ONLINE  -

# zfs list nas02
NAME    USED  AVAIL  REFER  MOUNTPOINT
nas02    88K   675T   201K  none
```

## ZFS 设置

由于该存储的主要用途是存放文件，我将使用 **recordsize** 的最大值之一 —— 1 MB —— 以获得更好的压缩比。

…但它也将作为 iSCSI 目标使用，在 iSCSI 中我们会使用原生的 4k 块，因此 iSCSI 设置为 4096 字节。

```sh
# zfs set compression=lz4         nas02
# zfs set atime=off               nas02
# zfs set mountpoint=none         nas02
# zfs set recordsize=1m           nas02
# zfs set redundant_metadata=most nas02
# zfs create                      nas02/nfs
# zfs create                      nas02/smb
# zfs create                      nas02/iscsi
# zfs set recordsize=4k           nas02/iscsi
```

另外说明一下参数 **redundant_metadata** ，因为它不是很直观。引用 **zfs(8)** 手册中的说明。

```sh
# man zfs
(...)
redundant_metadata=all | most
  Controls what types of metadata are stored redundantly.  ZFS stores
  an extra copy of metadata, so that if a single block is corrupted,
  the amount of user data lost is limited.  This extra copy is in
  addition to any redundancy provided at the pool level (e.g. by
  mirroring or RAID-Z), and is in addition to an extra copy specified
  by the copies property (up to a total of 3 copies).  For example if
  the pool is mirrored, copies=2, and redundant_metadata=most, then ZFS
  stores 6 copies of most metadata, and 4 copies of data and some
  metadata.

  When set to all, ZFS stores an extra copy of all metadata.  If a
  single on-disk block is corrupt, at worst a single block of user data
  (which is recordsize bytes long can be lost.)

  When set to most, ZFS stores an extra copy of most types of metadata.
  This can improve performance of random writes, because less metadata
  must be written.  In practice, at worst about 100 blocks (of
  recordsize bytes each) of user data can be lost if a single on-disk
  block is corrupt.  The exact behavior of which metadata blocks are
  stored redundantly may change in future releases.

  The default value is all.

控制存储哪些类型的元数据为冗余。
ZFS 会存储额外的元数据副本，以便当单个块损坏时，用户数据的丢失量有限。
这个额外副本是在池级别提供的冗余（例如镜像或 RAID-Z）之外的，并且是在 **copies** 属性指定的额外副本之外（最多可达三份）。
例如，如果池是镜像，copies=2，且 redundant_metadata=most，那么 ZFS 会为大部分元数据存储 6 份副本，为数据和部分元数据存储 4 份副本。

当设置为 **all** 时，ZFS 会为所有元数据存储额外副本。
如果单个磁盘块损坏，最坏情况下只会丢失一块用户数据（大小为 recordsize 字节）。

当设置为 **most** 时，ZFS 会为大部分类型的元数据存储额外副本。
这可以提升随机写入性能，因为需要写入的元数据更少。
实际上，如果单个磁盘块损坏，最坏情况下可能丢失大约 100 块用户数据（每块大小为 recordsize 字节）。
具体哪些元数据块会被冗余存储，未来版本可能会有所变化。

默认值为 **all**。

(...)
```

从上面的文字可以看出，它主要在单设备池中有用，因为当我们基于 RAIDZ2（RAID6 等价）提供冗余时，就不需要保留额外的元数据副本。

这样做能提升写入性能。

为了记录——iSCSI 的 ZFS zvol 是通过如下命令创建的——作为稀疏文件，也称为 *薄配置（Thin Provisioning）* 模式。

```sh
# zfs create -s -V 16T nas02/iscsi/test
```

由于我们有备用（SPARE）磁盘，我们还需要通过在 **/etc/rc.conf** 文件中添加 **zfsd_enable=YES** 来启用 **zfsd(8)** 守护进程。

我们还需要为池启用 **autoreplace** 属性，因为默认情况下它是 **off**。

```sh
# zpool get autoreplace nas02
NAME   PROPERTY     VALUE    SOURCE
nas02  autoreplace  off      default

# zpool set autoreplace=on nas02

# zpool get autoreplace nas02
NAME   PROPERTY     VALUE    SOURCE
nas02  autoreplace  on       local
```

其他 ZFS 设置位于 **/boot/loader.conf** 文件中。由于该系统有 128 GB 内存，我们将允许 ZFS 使用其中的 50% 到 75% 作为 ARC。

```sh
# grep vfs.zfs /boot/loader.conf
  vfs.zfs.prefetch_disable=1
  vfs.zfs.cache_flush_disable=1
  vfs.zfs.vdev.cache.size=16M
  vfs.zfs.arc_min=64G
  vfs.zfs.arc_max=96G
  vfs.zfs.deadman_enabled=0
```

## 配置网络

这正是我非常喜欢 FreeBSD 的地方。要设置 LACP 链路聚合，你只需在 **/etc/rc.conf** 文件中写 5 行。在 RHEL 上，你可能需要多个文件，每个文件还要写很多行。

```sh
# head -5 /etc/rc.conf
  defaultrouter="10.20.30.254"
  ifconfig_ixl0="up"
  ifconfig_ixl1="up"
  cloned_interfaces="lagg0"
  ifconfig_lagg0="laggproto lacp laggport ixl0 laggport ixl1 10.20.30.2/24 up"

# ifconfig lagg0
lagg0: flags=8843 metric 0 mtu 1500
        options=e507bb
        ether a0:42:3f:a0:42:3f
        inet 10.20.30.2 netmask 0xffffff00 broadcast 10.20.30.255
        laggproto lacp lagghash l2,l3,l4
        laggport: ixl0 flags=1c
        laggport: ixl1 flags=1c
        groups: lagg
        media: Ethernet autoselect
        status: active
        nd6 options=29
```

**Intel X710 DA-2** 10GE 网卡在 FreeBSD 下由 **ixl(4)** 驱动完美支持。

![intel-x710-da-2.jpg](https://vermaden.wordpress.com/wp-content/uploads/2019/06/intel-x710-da-2.jpg?w=960)

### 配置 Cisco Nexus 

这是启用 LACP 聚合所需的 *Cisco Nexus* 配置。

首先是端口设置。

```sh
NEXUS-1  Eth1/32  NAS02_IXL0  connected 3  full  a-10G  SFP-H10GB-A
NEXUS-2  Eth1/32  NAS02_IXL1  connected 3  full  a-10G  SFP-H10GB-A
```

… 现在进行聚合配置。

```ini
interface Ethernet1/32
  description NAS02_IXL1
  switchport
  switchport access vlan 3
  mtu 9216
  channel-group 128 mode active
  no shutdown
!
interface port-channel128
  description NAS02
  switchport
  switchport access vlan 3
  mtu 9216
  vpc 128
```

… 在第二台 *Cisco Nexus* **NEXUS-2** 交换机上也进行相同/类似配置。

## FreeBSD 配置

这些是在 FreeBSD 系统上最重要的三个配置文件。

现在我将发布我在这台存储系统上使用的所有设置。

**/etc/rc.conf** 文件。

```ini
# cat /etc/rc.conf
# 网络
  hostname="nas02.local"
  defaultrouter="10.20.30.254"
  ifconfig_ixl0="up"
  ifconfig_ixl1="up"
  cloned_interfaces="lagg0"
  ifconfig_lagg0="laggproto lacp laggport ixl0 laggport ixl1 10.20.30.2/24 up"

# 内核模块
  kld_list="${kld_list} aesni"

# 守护进程 | 启用
  zfs_enable=YES
  zfsd_enable=YES
  sshd_enable=YES
  ctld_enable=YES
  powerd_enable=YES

# 守护进程 | NFS 服务
  nfs_server_enable=YES
  nfs_client_enable=YES
  rpc_lockd_enable=YES
  rpc_statd_enable=YES
  rpcbind_enable=YES
  mountd_enable=YES
  mountd_flags="-r"

# 其他
  dumpdev=NO
```

**/boot/loader.conf** 文件。

```ini
# cat /boot/loader.conf
# 启动参数
  autoboot_delay=3
  kern.geom.label.disk_ident.enable=0
  kern.geom.label.gptid.enable=0

# 禁用英特尔超线程
  machdep.hyperthreading_allowed=0

# 在内核加载前更新英特尔处理器微码
  cpu_microcode_load=YES
  cpu_microcode_name=/boot/firmware/intel-ucode.bin

# 模块
  zfs_load=YES
  aio_load=YES

# RACCT/RCTL 资源限制
  kern.racct.enable=1

# 启动时禁用内存测试
  hw.memtest.tests=0

# 管道 KVA 限制 | 320 MB
  kern.ipc.maxpipekva=335544320

# IPC
  kern.ipc.shmseg=1024
  kern.ipc.shmmni=1024
  kern.ipc.shmseg=1024
  kern.ipc.semmns=512
  kern.ipc.semmnu=256
  kern.ipc.semume=256
  kern.ipc.semopm=256
  kern.ipc.semmsl=512

# 大页映射
  vm.pmap.pg_ps_enabled=1

# ZFS 调优
  vfs.zfs.prefetch_disable=1
  vfs.zfs.cache_flush_disable=1
  vfs.zfs.vdev.cache.size=16M
  vfs.zfs.arc_min=64G
  vfs.zfs.arc_max=96G

# ZFS 禁用对过期 I/O 的 panic
  vfs.zfs.deadman_enabled=0

# NEWCONS 挂起
  kern.vt.suspendswitch=0
```


**/etc/sysctl.conf** 文件

```ini
# cat /etc/sysctl.conf
# ZFS 对齐大小
  vfs.zfs.min_auto_ashift=12

# 安全
  security.bsd.stack_guard_page=1

# 安全性：Intel MDS（微架构数据采样）缓解措施
  hw.mds_disable=3

# 禁用恼人的功能
  kern.coredump=0
  hw.syscons.bell=0

# IPC
  kern.ipc.shmmax=4294967296
  kern.ipc.shmall=2097152
  kern.ipc.somaxconn=4096
  kern.ipc.maxsockbuf=5242880
  kern.ipc.shm_allow_removed=1

# 网络
  kern.ipc.maxsockbuf=16777216
  kern.ipc.soacceptqueue=1024
  net.inet.tcp.recvbuf_max=8388608
  net.inet.tcp.sendbuf_max=8388608
  net.inet.tcp.mssdflt=1460
  net.inet.tcp.minmss=1300
  net.inet.tcp.syncache.rexmtlimit=0
  net.inet.tcp.syncookies=0
  net.inet.tcp.tso=0
  net.inet.ip.process_options=0
  net.inet.ip.random_id=1
  net.inet.ip.redirect=0
  net.inet.icmp.drop_redirect=1
  net.inet.tcp.always_keepalive=0
  net.inet.tcp.drop_synfin=1
  net.inet.tcp.fast_finwait2_recycle=1
  net.inet.tcp.icmp_may_rst=0
  net.inet.tcp.msl=8192
  net.inet.tcp.path_mtu_discovery=0
  net.inet.udp.blackhole=1
  net.inet.tcp.blackhole=2
  net.inet.tcp.hostcache.expire=7200
  net.inet.tcp.delacktime=20
```


## 目的

为什么要构建这种设备？因为它比购买“品牌”设备便宜得多。以 *Dell EMC Data Domain* 为例——而且不是“普通”的 *Data Domain*，而是几乎最高端的型号——至少是 *Data Domain DD9300*。它的价格至少要高十倍……容量更小，而且占用机架空间不是 4U，而是接近 14U，需要三个 *DS60* 扩展器。

但你实际上可以让这个 *FreeBSD 企业存储* 的行为类似于 *Dell EMC Data Domain*……或者比如他们的 *Dell EMC Elastic Cloud Storage*。

*Dell EMC CloudBoost* 可以部署在你的 *VMware* 环境中，以提供 DDBoost 去重功能。然后你还需要 *OpenStack Swift*，因为它是支持的后端设备之一。

![emc-cloudboost-swift-cover.png](https://vermaden.wordpress.com/wp-content/uploads/2019/06/emc-cloudboost-swift-cover.png?w=960)

![emc-cloudboost-swift-support.png](https://vermaden.wordpress.com/wp-content/uploads/2019/06/emc-cloudboost-swift-support.png?w=960)

FreeBSD 上的 OpenStack Swift 包大约落后现实 4-5 年（版本 2.2.2）（**译注：目前版本已经更新了**），所以这里你需要使用 Bhyve。

```sh
# pkg search swift
(...)
py27-swift-2.2.2_1             Highly available, distributed, eventually consistent object/blob store
(...)
```

在这台 *FreeBSD 企业存储* 上可以创建 Bhyve 虚拟机，例如安装 CentOS 7.6 系统，然后在其中设置 *Swift*，这样完全可行。利用 20 个物理核心和 128 GB 内存，你几乎感觉不到虚拟机的存在。

这样，你可以使用 *Dell EMC Networker*，而存储成本却降低到了原来的十分之一还多。

以前我也写过关于 [IBM Spectrum Protect (TSM)](https://vermaden.wordpress.com/2018/08/23/ibm-tsm-spectrum-protect-on-veritas-cluster-server/) 的文章，这种 FreeBSD 企业存储对它也非常有利。我实际上也将这个基于 FreeBSD 的存储用作 *IBM Spectrum Protect (TSM)* 容器池目录的空间。通过 iSCSI 导出效果非常好。

你还可以将这个 *FreeBSD 企业存储* 与其他存储设备进行比较，比如 *iXsystems TrueNAS* 或 *EXAGRID*。

## 性能

你肯定想知道这个 *FreeBSD 企业存储* 的性能水平 :🙂:

我会分享我收集到的所有性能数据。

### 网络性能

首先是网络性能。

我使用 **iperf3** 作为基准测试工具。

我在 FreeBSD 端启动服务器：

```sh
# iperf3 -s
```

然后在 *Windows Server 2016* 机器上启动客户端：

```sh
C:\iperf-3.1.3-win64>iperf3.exe -c nas02 -P 8
(...)
[SUM]   0.00-10.00  sec  10.8 GBytes  9.26 Gbits/sec                  receiver
(..)
```

这是在 MTU 1500 下的结果——不使用 Jumbo 帧 :😦:

不幸的是，这套系统只有一个物理 10GE 接口，但我也做了其他测试。使用两台单 10GE 接口的服务器，可以很好地饱和 FreeBSD 端的双 10GE LACP。

我还将 NFS 和 iSCSI 导出到 *RHEL* 系统。单个 10GE 接口下网络性能约为 500-600 MB/s。在 LACP 聚合下可以达到 1000-1200 MB/s。

### 磁盘子系统性能

现在是磁盘子系统。

先用一些简单的测试，使用 FreeBSD 自带工具 **diskinfo(8)**。

```sh
# diskinfo -ctv /dev/da12
/dev/da12
        512             # sectorsize
        12000138625024  # mediasize in bytes (11T)
        23437770752     # mediasize in sectors
        4096            # stripesize
        0               # stripeoffset
        1458933         # Cylinders according to firmware.
        255             # Heads according to firmware.
        63              # Sectors according to firmware.
        ATA TOSHIBA MG07ACA1    # Disk descr.
        98H0A11KF95G    # Disk ident.
        id1,enc@n500e081010445dbd/type@0/slot@c/elmdesc@ArrayDevice11   # Physical path
        No              # TRIM/UNMAP support
        7200            # Rotation rate in RPM
        Not_Zoned       # Zone Mode

I/O command overhead:
        time to read 10MB block      0.067031 sec       =    0.003 msec/sector
        time to read 20480 sectors   2.619989 sec       =    0.128 msec/sector
        calculated command overhead                     =    0.125 msec/sector

Seek times:
        Full stroke:      250 iter in   5.665880 sec =   22.664 msec
        Half stroke:      250 iter in   4.263047 sec =   17.052 msec
        Quarter stroke:   500 iter in   6.867914 sec =   13.736 msec
        Short forward:    400 iter in   3.057913 sec =    7.645 msec
        Short backward:   400 iter in   1.979287 sec =    4.948 msec
        Seq outer:       2048 iter in   0.169472 sec =    0.083 msec
        Seq inner:       2048 iter in   0.469630 sec =    0.229 msec

Transfer rates:
        outside:       102400 kbytes in   0.478251 sec =   214114 kbytes/sec
        middle:        102400 kbytes in   0.605701 sec =   169060 kbytes/sec
        inside:        102400 kbytes in   1.303909 sec =    78533 kbytes/sec
```

现在我们已经知道单块磁盘的速度了。

接下来，让我们在 ZFS zvol 设备上重复同样的测试。

```sh
# diskinfo -ctv /dev/zvol/nas02/iscsi/test
/dev/zvol/nas02/iscsi/test
        512             # sectorsize
        17592186044416  # mediasize in bytes (16T)
        34359738368     # mediasize in sectors
        65536           # stripesize
        0               # stripeoffset
        Yes             # TRIM/UNMAP support
        Unknown         # Rotation rate in RPM

I/O command overhead:
        time to read 10MB block      0.004512 sec       =    0.000 msec/sector
        time to read 20480 sectors   0.196824 sec       =    0.010 msec/sector
        calculated command overhead                     =    0.009 msec/sector

Seek times:
        Full stroke:      250 iter in   0.006151 sec =    0.025 msec
        Half stroke:      250 iter in   0.008228 sec =    0.033 msec
        Quarter stroke:   500 iter in   0.014062 sec =    0.028 msec
        Short forward:    400 iter in   0.010564 sec =    0.026 msec
        Short backward:   400 iter in   0.011725 sec =    0.029 msec
        Seq outer:       2048 iter in   0.028198 sec =    0.014 msec
        Seq inner:       2048 iter in   0.028416 sec =    0.014 msec

Transfer rates:
        outside:       102400 kbytes in   0.036938 sec =  2772213 kbytes/sec
        middle:        102400 kbytes in   0.043076 sec =  2377194 kbytes/sec
        inside:        102400 kbytes in   0.034260 sec =  2988908 kbytes/sec
```

接近 3 GB/s —— 不错。

接下来进行更传统的测试 —— 不朽的 **dd(8)** 命令。

这是在 **compression=off** 设置下进行的测试。

单进程运行。

```sh
# dd if=/dev/zero of=FILE bs=128m status=progress
26172456960 bytes (26 GB, 24 GiB) transferred 16.074s, 1628 MB/s
202+0 records in
201+0 records out
26977763328 bytes transferred in 16.660884 secs (1619227644 bytes/sec)
```

四个并发进程运行。

```sh
# dd if=/dev/zero of=FILE${X} bs=128m status=progress
80933289984 bytes (81 GB, 75 GiB) transferred 98.081s, 825 MB/s
608+0 records in
608+0 records out
81604378624 bytes transferred in 98.990579 secs (824365101 bytes/sec)
```

八个并发进程运行。

```sh
# dd if=/dev/zero of=FILE${X} bs=128m status=progress
174214610944 bytes (174 GB, 162 GiB) transferred 385.042s, 452 MB/s
1302+0 records in
1301+0 records out
174617264128 bytes transferred in 385.379296 secs (453104943 bytes/sec)
```

总结这些数据。

```sh
1 STREAM(s) ~ 1600 MB/s ~ 1.5 GB/s
4 STREAM(s) ~ 3300 MB/s ~ 3.2 GB/s
8 STREAM(s) ~ 3600 MB/s ~ 3.5 GB/s
```

所以磁盘子系统在顺序写入时能够达到约 3.5 GB/s 的持续速度。这意味着如果想要完全饱和网络，需要再添加两个 10GE 接口。

磁盘的压力测试仅达到约 55%，这可以通过 FreeBSD 的另一个实用工具 **gstat(8)** 命令看到。

![n10.png](https://vermaden.wordpress.com/wp-content/uploads/2019/06/n10.png?w=960)

接下来进行更“智能”的测试——**blogbench** 测试。

首先禁用压缩进行测试。

```sh
# time blogbench -d .
Frequency = 10 secs
Scratch dir = [.]
Spawning 3 writers...
Spawning 1 rewriters...
Spawning 5 commenters...
Spawning 100 readers...
Benchmarking for 30 iterations.
The test will run during 5 minutes.
(...)
Final score for writes:          6476
Final score for reads :        660436

blogbench -d .  280.58s user 4974.41s system 1748% cpu 5:00.54 total
```

接下来设置压缩为 LZ4 进行第二轮测试。

```sh
# time blogbench -d .
Frequency = 10 secs
Scratch dir = [.]
Spawning 3 writers...
Spawning 1 rewriters...
Spawning 5 commenters...
Spawning 100 readers...
Benchmarking for 30 iterations.
The test will run during 5 minutes.
(...)
Final score for writes:          7087
Final score for reads :        733932

blogbench -d .  299.08s user 5415.04s system 1900% cpu 5:00.68 total
```

压缩效果不大，但确实有一定帮助。

为了对比，我们将在系统 ZFS 池上运行相同的测试——两个 Intel SSD DC S3500 240 GB 镜像驱动，其性能如下：

**Intel SSD DC S3500 240 GB 驱动特性：**

* 顺序读取（最大）**500 MB/s**
* 顺序写入（最大）**260 MB/s**
* 随机读取（100% 范围）**75000 IOPS**
* 随机写入（100% 范围）**7500 IOPS**

```sh
# time blogbench -d .
Frequency = 10 secs
Scratch dir = [.]
Spawning 3 writers...
Spawning 1 rewriters...
Spawning 5 commenters...
Spawning 100 readers...
Benchmarking for 30 iterations.
The test will run during 5 minutes.
(...)
Final score for writes:          6109
Final score for reads :        654099

blogbench -d .  278.73s user 5058.75s system 1777% cpu 5:00.30 total
```

现在进行 **randomio** 测试。这是一个多线程磁盘 I/O 微基准测试。

使用方法如下：

```sh
usage: randomio filename nr_threads write_fraction_of_io fsync_fraction_of_writes io_size nr_seconds_between_samples

filename                    Filename or device to read/write.
write_fraction_of_io        What fraction of I/O should be writes - for example 0.25 for 25% write.
fsync_fraction_of_writes    What fraction of writes should be fsync'd.
io_size                     How many bytes to read/write (multiple of 512 bytes).
nr_seconds_between_samples  How many seconds to average samples over.
```

使用 **4k 块** 进行的 **随机 I/O** 测试。

```sh
# zfs create -s -V 1T nas02/iscsi/test
# randomio /dev/zvol/nas02/iscsi/test 8 0.25 1 4096 10
  total |  read:         latency (ms)       |  write:        latency (ms)
   iops |   iops   min    avg    max   sdev |   iops   min    avg    max   sdev
--------+-----------------------------------+----------------------------------
54137.7 |40648.4   0.0    0.1  575.8    2.2 |13489.4   0.0    0.3  405.8    2.6
66248.4 |49641.5   0.0    0.1   19.6    0.3 |16606.9   0.0    0.2   26.4    0.7
66411.0 |49817.2   0.0    0.1   19.7    0.3 |16593.8   0.0    0.2   20.3    0.7
64158.9 |48142.8   0.0    0.1  254.7    0.7 |16016.1   0.0    0.2  130.4    1.0
48454.1 |36390.8   0.0    0.1  542.8    2.7 |12063.3   0.0    0.3  507.5    3.2
66796.1 |50067.4   0.0    0.1   24.1    0.3 |16728.7   0.0    0.2   23.4    0.7
58512.2 |43851.7   0.0    0.1  576.5    1.7 |14660.5   0.0    0.2  307.2    1.7
63195.8 |47341.8   0.0    0.1  261.6    0.9 |15854.1   0.0    0.2  361.1    1.9
67086.0 |50335.6   0.0    0.1   20.4    0.3 |16750.4   0.0    0.2   25.1    0.8
67429.8 |50549.6   0.0    0.1   21.8    0.3 |16880.3   0.0    0.2   20.6    0.7
^C
```

… 使用 **512 字节扇区** 进行测试。


```sh
# zfs create -s -V 1T nas02/iscsi/test
# randomio /dev/zvol/nas02/iscsi/TEST 8 0.25 1 512 10
  total |  read:         latency (ms)       |  write:        latency (ms)
   iops |   iops   min    avg    max   sdev |   iops   min    avg    max   sdev
--------+-----------------------------------+----------------------------------
58218.9 |43712.0   0.0    0.1  501.5    2.1 |14506.9   0.0    0.2  272.5    1.6
66325.3 |49703.8   0.0    0.1  352.0    0.9 |16621.4   0.0    0.2  352.0    1.5
68130.5 |51100.8   0.0    0.1   24.6    0.3 |17029.7   0.0    0.2   24.4    0.7
68465.3 |51352.3   0.0    0.1   19.9    0.3 |17112.9   0.0    0.2   23.8    0.7
54903.5 |41249.1   0.0    0.1  399.3    1.9 |13654.4   0.0    0.3  335.8    2.2
61259.8 |45898.7   0.0    0.1  574.6    1.7 |15361.0   0.0    0.2  371.5    1.7
68483.3 |51313.1   0.0    0.1   22.9    0.3 |17170.3   0.0    0.2   26.1    0.7
56713.7 |42524.7   0.0    0.1  373.5    1.8 |14189.1   0.0    0.2  438.5    2.7
68861.4 |51657.0   0.0    0.1   21.0    0.3 |17204.3   0.0    0.2   21.7    0.7
68602.0 |51438.4   0.0    0.1   19.5    0.3 |17163.7   0.0    0.2   23.7    0.7
^C
```

两个 **随机 I/O** 测试都是在启用 LZ4 压缩的情况下运行的。

接下来是 **bonnie++** 基准测试。同样是在启用 LZ4 压缩的情况下运行的。


```sh
# bonnie++ -d . -u root
Using uid:0, gid:0.
Writing a byte at a time...done
Writing intelligently...done
Rewriting...done
Reading a byte at a time...done
Reading intelligently...done
start 'em...done...done...done...done...done...
Create files in sequential order...done.
Stat files in sequential order...done.
Delete files in sequential order...done.
Create files in random order...done.
Stat files in random order...done.
Delete files in random order...done.
Version  1.97       ------Sequential Output------ --Sequential Input- --Random-
Concurrency   1     -Per Chr- --Block-- -Rewrite- -Per Chr- --Block-- --Seeks--
Machine        Size K/sec %CP K/sec %CP K/sec %CP K/sec %CP K/sec %CP  /sec %CP
nas02.local 261368M   139  99 775132  99 589190  99   383  99 1638929  99 12930 2046
Latency             60266us    7030us    7059us   21553us    3844us    5710us
Version  1.97       ------Sequential Create------ --------Random Create--------
nas02.local         -Create-- --Read--- -Delete-- -Create-- --Read--- -Delete--
              files  /sec %CP  /sec %CP  /sec %CP  /sec %CP  /sec %CP  /sec %CP
                 16 +++++ +++ +++++ +++ 12680  44 +++++ +++ +++++ +++ 30049  99
Latency              2619us      43us     714ms    2748us      28us      58us
```

最后但同样重要的是 **fio** 基准测试。同样启用了 LZ4 压缩。

```sh
# fio --randrepeat=1 --direct=1 --gtod_reduce=1 --name=test --filename=random_read_write.fio --bs=4k --iodepth=64 --size=4G --readwrite=randrw --rwmixread=75
test: (g=0): rw=randrw, bs=(R) 4096B-4096B, (W) 4096B-4096B, (T) 4096B-4096B, ioengine=psync, iodepth=64
fio-3.13
Starting 1 process
Jobs: 1 (f=1): [m(1)][98.0%][r=38.0MiB/s,w=12.2MiB/s][r=9735,w=3128 IOPS][eta 00m:05s]
test: (groupid=0, jobs=1): err= 0: pid=35368: Tue Jun 18 15:14:44 2019
  read: IOPS=3157, BW=12.3MiB/s (12.9MB/s)(3070MiB/248872msec)
   bw (  KiB/s): min= 9404, max=57732, per=98.72%, avg=12469.84, stdev=3082.99, samples=497
   iops        : min= 2351, max=14433, avg=3117.15, stdev=770.74, samples=497
  write: IOPS=1055, BW=4222KiB/s (4323kB/s)(1026MiB/248872msec)
   bw (  KiB/s): min= 3179, max=18914, per=98.71%, avg=4166.60, stdev=999.23, samples=497
   iops        : min=  794, max= 4728, avg=1041.25, stdev=249.76, samples=497
  cpu          : usr=1.11%, sys=88.64%, ctx=677981, majf=0, minf=0
  IO depths    : 1=100.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued rwts: total=785920,262656,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=64

Run status group 0 (all jobs):
   READ: bw=12.3MiB/s (12.9MB/s), 12.3MiB/s-12.3MiB/s (12.9MB/s-12.9MB/s), io=3070MiB (3219MB), run=248872-248872msec
  WRITE: bw=4222KiB/s (4323kB/s), 4222KiB/s-4222KiB/s (4323kB/s-4323kB/s), io=1026MiB (1076MB), run=248872-248872msec
```

不知道你怎么看，我对性能是非常满意的 :🙂:

## FreeNAS

最初我确实想在这些服务器上使用 *FreeNAS*，甚至还安装了 *FreeNAS*。它运行得不错，但…… *FreeNAS* 的安全部分并不是最佳。

这是 **pkg audit** 命令的输出，非常吓人。

```sh
root@freenas[~]# pkg audit -F
Fetching vuln.xml.bz2: 100%  785 KiB 804.3kB/s    00:01
python27-2.7.15 is vulnerable:
Python -- NULL pointer dereference vulnerability
CVE: CVE-2019-5010
WWW: https://vuxml.FreeBSD.org/freebsd/d74371d2-4fee-11e9-a5cd-1df8a848de3d.html

curl-7.62.0 is vulnerable:
curl -- multiple vulnerabilities
CVE: CVE-2019-3823
CVE: CVE-2019-3822
CVE: CVE-2018-16890
WWW: https://vuxml.FreeBSD.org/freebsd/714b033a-2b09-11e9-8bc3-610fd6e6cd05.html

libgcrypt-1.8.2 is vulnerable:
libgcrypt -- side-channel attack vulnerability
CVE: CVE-2018-0495
WWW: https://vuxml.FreeBSD.org/freebsd/9b5162de-6f39-11e8-818e-e8e0b747a45a.html

python36-3.6.5_1 is vulnerable:
Python -- NULL pointer dereference vulnerability
CVE: CVE-2019-5010
WWW: https://vuxml.FreeBSD.org/freebsd/d74371d2-4fee-11e9-a5cd-1df8a848de3d.html

pango-1.42.0 is vulnerable:
pango -- remote DoS vulnerability
CVE: CVE-2018-15120
WWW: https://vuxml.FreeBSD.org/freebsd/5a757a31-f98e-4bd4-8a85-f1c0f3409769.html

py36-requests-2.18.4 is vulnerable:
www/py-requests -- Information disclosure vulnerability
WWW: https://vuxml.FreeBSD.org/freebsd/50ad9a9a-1e28-11e9-98d7-0050562a4d7b.html

libnghttp2-1.31.0 is vulnerable:
nghttp2 -- Denial of service due to NULL pointer dereference
CVE: CVE-2018-1000168
WWW: https://vuxml.FreeBSD.org/freebsd/1fccb25e-8451-438c-a2b9-6a021e4d7a31.html

gnupg-2.2.6 is vulnerable:
gnupg -- unsanitized output (CVE-2018-12020)
CVE: CVE-2017-7526
CVE: CVE-2018-12020
WWW: https://vuxml.FreeBSD.org/freebsd/7da0417f-6b24-11e8-84cc-002590acae31.html

py36-cryptography-2.1.4 is vulnerable:
py-cryptography -- tag forgery vulnerability
CVE: CVE-2018-10903
WWW: https://vuxml.FreeBSD.org/freebsd/9e2d0dcf-9926-11e8-a92d-0050562a4d7b.html

perl5-5.26.1 is vulnerable:
perl -- multiple vulnerabilities
CVE: CVE-2018-6913
CVE: CVE-2018-6798
CVE: CVE-2018-6797
WWW: https://vuxml.FreeBSD.org/freebsd/41c96ffd-29a6-4dcc-9a88-65f5038fa6eb.html

libssh2-1.8.0,3 is vulnerable:
libssh2 -- multiple issues
CVE: CVE-2019-3862
CVE: CVE-2019-3861
CVE: CVE-2019-3860
CVE: CVE-2019-3858
WWW: https://vuxml.FreeBSD.org/freebsd/6e58e1e9-2636-413e-9f84-4c0e21143628.html

git-lite-2.17.0 is vulnerable:
Git -- Fix memory out-of-bounds and remote code execution vulnerabilities (CVE-2018-11233 and CVE-2018-11235)
CVE: CVE-2018-11235
CVE: CVE-2018-11233
WWW: https://vuxml.FreeBSD.org/freebsd/c7a135f4-66a4-11e8-9e63-3085a9a47796.html

gnutls-3.5.18 is vulnerable:
GnuTLS -- double free, invalid pointer access
CVE: CVE-2019-3836
CVE: CVE-2019-3829
WWW: https://vuxml.FreeBSD.org/freebsd/fb30db8f-62af-11e9-b0de-001cc0382b2f.html

13 problem(s) in the installed packages found.

root@freenas[~]# uname -a
FreeBSD freenas.local 11.2-STABLE FreeBSD 11.2-STABLE #0 r325575+95cc58ca2a0(HEAD): Mon May  6 19:08:58 EDT 2019     root@mp20.tn.ixsystems.com:/freenas-releng/freenas/_BE/objs/freenas-releng/freenas/_BE/os/sys/FreeNAS.amd64  amd64

root@freenas[~]# freebsd-version -uk
11.2-STABLE
11.2-STABLE

root@freenas[~]# sockstat -l4
USER     COMMAND    PID   FD PROTO  LOCAL ADDRESS         FOREIGN ADDRESS
root     uwsgi-3.6  4006  3  tcp4   127.0.0.1:9042        *:*
root     uwsgi-3.6  3188  3  tcp4   127.0.0.1:9042        *:*
nobody   mdnsd      3144  4  udp4   *:31417               *:*
nobody   mdnsd      3144  6  udp4   *:5353                *:*
www      nginx      3132  6  tcp4   *:443                 *:*
www      nginx      3132  8  tcp4   *:80                  *:*
root     nginx      3131  6  tcp4   *:443                 *:*
root     nginx      3131  8  tcp4   *:80                  *:*
root     ntpd       2823  21 udp4   *:123                 *:*
root     ntpd       2823  22 udp4   10.49.13.99:123       *:*
root     ntpd       2823  25 udp4   127.0.0.1:123         *:*
root     sshd       2743  5  tcp4   *:22                  *:*
root     syslog-ng  2341  19 udp4   *:1031                *:*
nobody   mdnsd      2134  3  udp4   *:39020               *:*
nobody   mdnsd      2134  5  udp4   *:5353                *:*
root     python3.6  236   22 tcp4   *:6000                *:*
```

我甚至试图了解为什么 *FreeNAS* 在其最新版本中会包含如此过时且不安全的包——[FreeNAS 11.2-U3 Vulnerabilities](https://www.ixsystems.com/community/threads/freenas-11-2-u3-vulnerabilities.75353/)——这是我在他们论坛上发起的讨论贴。

不幸的是，这反映了他们的政策，可以总结为“只要能用，就不要动/更改版本”——至少我得出了这个印象。

因为这些安全漏洞，我无法推荐使用 *FreeNAS*，于是我转向了原生的 FreeBSD 系统。

另一个有趣的情况是，在我安装 FreeBSD 后，我想导入 *FreeNAS* 创建的 ZFS pool。这是我执行 **zpool import** 命令后的结果。

```sh
# zpool import
   pool: nas02_gr06
     id: 1275660523517109367
  state: ONLINE
 status: The pool was last accessed by another system.
 action: The pool can be imported using its name or numeric identifier and
        the '-f' flag.
   see: http://illumos.org/msg/ZFS-8000-EY
 config:

        nas02_gr06  ONLINE
          raidz2-0  ONLINE
            da58p2  ONLINE
            da59p2  ONLINE
            da60p2  ONLINE
            da61p2  ONLINE
            da62p2  ONLINE
            da63p2  ONLINE
            da64p2  ONLINE
            da26p2  ONLINE
            da65p2  ONLINE
            da23p2  ONLINE
            da29p2  ONLINE
            da66p2  ONLINE
            da67p2  ONLINE
            da68p2  ONLINE
        spares
          da69p2

   pool: nas02_gr05
     id: 5642709896812665361
  state: ONLINE
 status: The pool was last accessed by another system.
 action: The pool can be imported using its name or numeric identifier and
        the '-f' flag.
   see: http://illumos.org/msg/ZFS-8000-EY
 config:

        nas02_gr05  ONLINE
          raidz2-0  ONLINE
            da20p2  ONLINE
            da30p2  ONLINE
            da34p2  ONLINE
            da50p2  ONLINE
            da28p2  ONLINE
            da38p2  ONLINE
            da51p2  ONLINE
            da52p2  ONLINE
            da27p2  ONLINE
            da32p2  ONLINE
            da53p2  ONLINE
            da54p2  ONLINE
            da55p2  ONLINE
            da56p2  ONLINE
        spares
          da57p2

   pool: nas02_gr04
     id: 2460983830075205166
  state: ONLINE
 status: The pool was last accessed by another system.
 action: The pool can be imported using its name or numeric identifier and
        the '-f' flag.
   see: http://illumos.org/msg/ZFS-8000-EY
 config:

        nas02_gr04  ONLINE
          raidz2-0  ONLINE
            da44p2  ONLINE
            da37p2  ONLINE
            da18p2  ONLINE
            da36p2  ONLINE
            da45p2  ONLINE
            da19p2  ONLINE
            da22p2  ONLINE
            da33p2  ONLINE
            da35p2  ONLINE
            da21p2  ONLINE
            da31p2  ONLINE
            da47p2  ONLINE
            da48p2  ONLINE
            da49p2  ONLINE
        spares
          da46p2

   pool: nas02_gr03
     id: 4878868173820164207
  state: ONLINE
 status: The pool was last accessed by another system.
 action: The pool can be imported using its name or numeric identifier and
        the '-f' flag.
   see: http://illumos.org/msg/ZFS-8000-EY
 config:

        nas02_gr03  ONLINE
          raidz2-0  ONLINE
            da81p2  ONLINE
            da71p2  ONLINE
            da14p2  ONLINE
            da15p2  ONLINE
            da80p2  ONLINE
            da16p2  ONLINE
            da88p2  ONLINE
            da17p2  ONLINE
            da40p2  ONLINE
            da41p2  ONLINE
            da25p2  ONLINE
            da42p2  ONLINE
            da24p2  ONLINE
            da43p2  ONLINE
        spares
          da39p2

   pool: nas02_gr02
     id: 3299037437134217744
  state: ONLINE
 status: The pool was last accessed by another system.
 action: The pool can be imported using its name or numeric identifier and
        the '-f' flag.
   see: http://illumos.org/msg/ZFS-8000-EY
 config:

        nas02_gr02  ONLINE
          raidz2-0  ONLINE
            da84p2  ONLINE
            da76p2  ONLINE
            da85p2  ONLINE
            da8p2   ONLINE
            da9p2   ONLINE
            da78p2  ONLINE
            da73p2  ONLINE
            da74p2  ONLINE
            da70p2  ONLINE
            da77p2  ONLINE
            da11p2  ONLINE
            da13p2  ONLINE
            da79p2  ONLINE
            da89p2  ONLINE
        spares
          da90p2

   pool: nas02_gr01
     id: 1132383125952900182
  state: ONLINE
 status: The pool was last accessed by another system.
 action: The pool can be imported using its name or numeric identifier and
        the '-f' flag.
   see: http://illumos.org/msg/ZFS-8000-EY
 config:

        nas02_gr01  ONLINE
          raidz2-0  ONLINE
            da91p2  ONLINE
            da75p2  ONLINE
            da0p2   ONLINE
            da82p2  ONLINE
            da1p2   ONLINE
            da83p2  ONLINE
            da2p2   ONLINE
            da3p2   ONLINE
            da4p2   ONLINE
            da5p2   ONLINE
            da86p2  ONLINE
            da6p2   ONLINE
            da7p2   ONLINE
            da72p2  ONLINE
        spares
          da87p2
```

看起来 *FreeNAS* 对 ZFS 的处理方式略有不同，他们为每个 RAIDZ2 目标创建了单独的 pool，并配置了专用的备用盘。有趣……

## 更新 1 – BSD Now 305

[FreeBSD Enterprise 1 PB Storage](https://vermaden.wordpress.com/2019/06/19/freebsd-enterprise-1-pb-storage/) 文章出现在 [BSD Now 305 – Changing Face of Unix](https://www.bsdnow.tv/305) 节目中。

感谢提及！

## 更新 2 – 数据中心实景照片

有些读者希望看到这台“怪兽”的实景照片。下面是数据中心拍摄的几张图片。

机箱正面及布线。

![tyan-real-01.jpg](https://vermaden.wordpress.com/wp-content/uploads/2019/06/tyan-real-01.jpg?w=960)

机箱正面另一角度。

![tyan-real-09.jpg](https://vermaden.wordpress.com/wp-content/uploads/2019/06/tyan-real-09.jpg?w=960)

机箱背面及布线。

![tyan-real-02.jpg](https://vermaden.wordpress.com/wp-content/uploads/2019/06/tyan-real-02.jpg?w=960)

带硬盘的顶部视角。

![tyan-real-03](https://vermaden.wordpress.com/wp-content/uploads/2019/06/tyan-real-03.jpg?w=960)

顶部另一视角。

![tyan-real-07.jpg](https://vermaden.wordpress.com/wp-content/uploads/2019/06/tyan-real-07.jpg?w=960)

硬盘位特写。

![tyan-real-08.jpg](https://vermaden.wordpress.com/wp-content/uploads/2019/06/tyan-real-08.jpg?w=960)

固态硬盘与机械硬盘。

![tyan-real-06.jpg](https://vermaden.wordpress.com/wp-content/uploads/2019/06/tyan-real-06.jpg?w=960)
