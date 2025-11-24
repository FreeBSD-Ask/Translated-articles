# FreeBSD 上的高可用 DHCP 服务器

- [Highly Available DHCP Server on FreeBSD](https://vermaden.wordpress.com/2018/08/12/highly-available-dhcp-server-on-freebsd/)
- 作者：𝚟𝚎𝚛𝚖𝚊𝚍𝚎𝚗
- 2018/08


今天我想分享在 FreeBSD 系统上搭建高可用 DHCP 服务器的方法，但在其他 UNIX 和类 UNIX 系统上也应该一样简单。我将在这里使用最显而易见的选择——*Internet Systems Consortium* 的实现——ISC DHCP server——它也可以在 FreeBSD Ports 和软件包中获得。

![ISC](https://vermaden.wordpress.com/wp-content/uploads/2018/08/isc.png?w=960)

因为一些时间的推进，ISC 正在开发一款新的 DHCP 服务器——Kea——他们打算最终在大多数服务器实现中用它替代 ISC DHCP。他们也建议新的实现者考虑使用 Kea 而不是 ISC DHCP，仅在 Kea 无法满足需求时才实现 ISC DHCP。例如，Kea 当前不包含客户端和中继。也许我以后会对这篇文章进行更新或者单独再写一篇文章。

同时 Kea 在一个月前刚拥有了高可用模式，因此如果我更早写这篇文章，那么这样的设置在 Kea 上将不可行。这也显示了 Kea 实现是多么年轻，所以目前我会坚持使用 ISC DHCP server，并在后续“观察”Kea 的发展。

## 架构

下面是我们的 ISC DHCP 设置的 [POOR MAN’S ASCII 架构图](https://en.wikipedia.org/wiki/Enterprise_Architect_%28software%29) 图。

```sh
+-------------+              +-------------+
  | {primary}   |              | {secondary} |
  | DHCPs1      | ==== HA ==== | DHCPs2      |
  | 10.0.10.251 |              | 10.0.10.252 |
  +-------------+              +-------------+
                 \            /
  +------------------------------------------+
  | ADDRESS POOL  10.0.10.x/24  ADDRESS POOL |
  +------------------------------------------+
              \                  /
               +----------------+
               | {DHCP CLIENTS} |
               +----------------+
```

每个 DHCP 服务器节点的设置都非常简单。它的 FreeBSD 11.2-RELEASE 安装在 4 GB 的 GPT 分区上，/ 文件系统是 UFS，如下所示只使用了 666 MB。

```sh
root@DHCPs1:/ # uname -v
FreeBSD 11.2-RELEASE #0 r335510: Fri Jun 22 04:32:14 UTC 2018     root@releng2.nyi.freebsd.org:/usr/obj/usr/src/sys/GENERIC 

root@DHCPs1:/ # gpart show
=>     40  8388528  ada0  GPT  (4.0G)
       40     1024     1  freebsd-boot  (512K)
     1064  8386560     2  freebsd-ufs  (4.0G)
  8387624      944        - free -  (472K)

root@DHCPs1:/ # du -smc * | sort -n
0       sys
1       COPYRIGHT
1       dev
1       entropy
1       libexec
1       media
1       mnt
1       net
1       proc
1       root
1       tmp
2       bin
4       etc
7       sbin
8       var
10      rescue
12      lib
128     boot
499     usr
666     total
```

对于少量客户端来说 128 MB 的内存已经足够。仍然有 32 MB 的空闲内存，以及 32 MB 的 Inactive 和 Buffered 内存可以被交换出去。更不用说每个 `getty` 进程大约占用 2 MB 内存，而你只需要其中 1 个，而不是 8 个。换言之，即使只有 64 MB 的内存，你也能跑起来。

```sh
root@DHCPs1:~ # top -b -o res
last pid: 15205;  load averages:  0.13,  0.25,  0.29  up 0+07:39:11    20:03:48
16 processes:  2 running, 14 sleeping

Mem: 1688K Active, 30M Inact, 26M Wired, 3800K Buf, 32M Free
Swap:


  PID USERNAME    THR PRI NICE   SIZE    RES STATE    TIME    WCPU COMMAND
38897 dhcpd         1  20    0 16424K 10724K select   0:00   0.00% dhcpd
30199 root          1  20    0 13160K  8036K RUN      0:00   0.00% sshd
15106 root          1  28    0 12848K  7136K select   0:00   0.00% sshd
53100 root          1  20    0  9180K  5040K select   0:02   0.00% devd
31079 root          1  20    0  7412K  3640K pause    0:00   0.00% csh
15205 root          1  20    0  7916K  3060K RUN      0:00   0.00% top
15960 root          1  20    0  6464K  2480K nanslp   0:00   0.00% cron
69084 root          1  20    0  6412K  2364K select   0:01   0.00% syslogd
28412 root          1  52    0  6408K  2124K ttyin    0:00   0.00% getty
28188 root          1  52    0  6408K  2124K ttyin    0:00   0.00% getty
28504 root          1  52    0  6408K  2124K ttyin    0:00   0.00% getty
28972 root          1  52    0  6408K  2124K ttyin    0:00   0.00% getty
29736 root          1  52    0  6408K  2124K ttyin    0:00   0.00% getty
29080 root          1  52    0  6408K  2124K ttyin    0:00   0.00% getty
30106 root          1  52    0  6408K  2124K ttyin    0:00   0.00% getty
29392 root          1  52    0  6408K  2124K ttyin    0:00   0.00% getty
```

DHCP 节点 `DHCPs1` 和 `DHCPs2` 的 `/etc/rc.conf` 文件是相同的（除了主机名和地址不同）。

``sh
root@DHCPs1:/ # cat /etc/rc.conf
hostname=DHCPs1
ifconfig_em0="inet 10.0.10.251/24 up"
sshd_enable=YES
sendmail_enable=NONE
clear_tmp_enable=YES
syslogd_flags="-ss"
dumpdev=NO

```

`/etc/sysctl.conf` 和 `/boot/loader.conf` 文件不需要修改。

现在你需要安装 ISC DHCP 服务器，由于当前版本是 4.4.x，软件包名相应为 `isc-dhcp44-server`，我们使用命令 `pkg(8)` 来安装。

```sh
root@DHCPs1:/ # pkg update -f -y
The package management tool is not yet installed on your system.
Bootstrapping pkg from pkg+http://pkg.FreeBSD.org/FreeBSD:11:amd64//quarterly, please wait...
Verifying signature with trusted certificate pkg.freebsd.org.2013102301... done
[nextcloud] Installing pkg-1.10.5...
[nextcloud] Extracting pkg-1.10.5: 100%
Updating FreeBSD repository catalogue...
pkg: Repository FreeBSD load error: access repo file(/var/db/pkg/repo-FreeBSD.sqlite) failed: No such file or directory
[nextcloud] Fetching meta.txz: 100%    944 B   0.9kB/s    00:01
[nextcloud] Fetching packagesite.txz: 100%    6 MiB 530.8kB/s    00:12
Processing entries: 100%
FreeBSD repository update completed. 31134 packages processed.
All repositories are up to date.
root@DHCPs1:/ # echo ?
0
root@DHCPs1:/ #
```

现在让我们安装软件包 `isc-dhcp44-server`。


```sh
root@DHCPs1:/ # pkg install isc-dhcp44-server
Updating FreeBSD repository catalogue...
FreeBSD repository is up to date.
All repositories are up to date.
Checking integrity... done (0 conflicting)
The following 1 package(s) will be affected (of 0 checked):

New packages to be INSTALLED:
        isc-dhcp44-server: 4.4.1_3 [FreeBSD]

Number of packages to be installed: 1

The process will require 6 MiB more space.

Proceed with this action? [y/N]: y
[1/1] Installing isc-dhcp44-server-4.4.1_3...
===> Creating groups.
Creating group 'dhcpd' with gid '136'.
===> Creating users
Creating user 'dhcpd' with uid '136'.
[1/1] Extracting isc-dhcp44-server-4.4.1_3: 100%
Message from isc-dhcp44-server-4.4.1_3:

****  To setup dhcpd, please edit /usr/local/etc/dhcpd.conf.

****  This port installs the dhcp daemon, but doesn't invoke dhcpd by default.
      If you want to invoke dhcpd at startup, add these lines to /etc/rc.conf:

            dhcpd_enable="YES"                          # dhcpd enabled?
            dhcpd_flags="-q"                            # command option(s)
            dhcpd_conf="/usr/local/etc/dhcpd.conf"      # configuration file
            dhcpd_ifaces=""                             # ethernet interface(s)
            dhcpd_withumask="022"                       # file creation mask

****  If compiled with paranoia support (the default), the following rc.conf
      options are also supported:

            dhcpd_chuser_enable="YES"           # runs w/o privileges?
            dhcpd_withuser="dhcpd"              # user name to run as
            dhcpd_withgroup="dhcpd"             # group name to run as
            dhcpd_chroot_enable="YES"           # runs chrooted?
            dhcpd_devfs_enable="YES"            # use devfs if available?
            dhcpd_rootdir="/var/db/dhcpd"       # directory to run in
            dhcpd_includedir=""       # directory with config-
                                                  files to include

****  WARNING: never edit the chrooted or jailed dhcpd.conf file but
      /usr/local/etc/dhcpd.conf instead which is always copied where
      needed upon startup.
```

现在更新 `pkg(8)` 仓库数据，并在 `DHCPs2` 节点上安装 `isc-dhcp44-server`。

配置使用单一网络段 10.0.10.0/24，为客户端分配最后一段在 10-250 的地址。参数 `split 128` 会将负载在 DHCP 服务器节点之间平均分配。由于这只是示例，我们将使用 1.1.1.1 和 9.9.9.9 作为 DNS 服务器，以及 `domain.com` 作为域名。需要说明的是，仅在 `primary` 节点（在我们的例子中为 `DHCPs1`）上设置参数 `split 128`。正如 `man dhcpd.conf` 页面所建议的，我们将 **“为两个服务器使用相同的主配置文件，并有一个单独的文件包含对等声明以及主文件。”**，这样做 **“有助于避免配置不一致。”**

```sh
root@DHCPs1:/ # cat /usr/local/etc/dhcpd.conf
# CORE
failover peer "ha-dhcp" {
  primary;
  address 10.0.10.251;
  port 678;
  peer address 10.0.10.252;
  peer port 678;
  max-response-delay 60;
  max-unacked-updates 10;
  mclt 3600;
  split 128;
  load balance max seconds 3;
}

include "/usr/local/etc/dhcpd.conf.SHARED";
```

```sh
root@DHCPs1:/ # cat /usr/local/etc/dhcpd.conf.SHARED
# CLIENTS
subnet 10.0.10.0 netmask 255.255.255.0 {
  default-lease-time         604800;
  max-lease-time             604800;
  option routers             10.0.10.254;
  option broadcast-address   10.0.10.255;
  option subnet-mask         255.255.255.0;
  option domain-search       "domain.com";
  option domain-name-servers 1.1.1.1,9.9.9.9;

  pool {
    failover peer "ha-dhcp";
    range 10.0.10.10 10.0.10.250;
  }
}
```

……secondary 节点

```sh
root@DHCPs2:~ # cat /usr/local/etc/dhcpd.conf
# CORE
failover peer "ha-dhcp" {
  secondary;
  address 10.0.10.252;
  port 678;
  peer address 10.0.10.251;
  peer port 678;
  max-response-delay 60;
  max-unacked-updates 10;
  mclt 3600;
  load balance max seconds 3;
}

include "/usr/local/etc/dhcpd.conf.SHARED";
```

```sh
root@DHCPs2:/ # cat /usr/local/etc/dhcpd.conf.SHARED
# CLIENTS
subnet 10.0.10.0 netmask 255.255.255.0 {
  default-lease-time         604800;
  max-lease-time             604800;
  option routers             10.0.10.254;
  option broadcast-address   10.0.10.255;
  option subnet-mask         255.255.255.0;
  option domain-search       "domain.com";
  option domain-name-servers 1.1.1.1,9.9.9.9;

  pool {
    failover peer "ha-dhcp";
    range 10.0.10.10 10.0.10.250;
  }
}
```

`/usr/local/etc/dhcpd.conf.SHARED` 文件在两个节点上是相同的。

现在让我们在两个节点上启动 DHCP 服务器。

```sh
root@DHCPs1:~ # sysrc dhcpd_enable=YES
dhcpd_enable:  -> YES
root@DHCPs1:~ # service isc-dhcpd start
Starting dhcpd.
Internet Systems Consortium DHCP Server 4.4.1
Copyright 2004-2018 Internet Systems Consortium.
All rights reserved.
For info, please visit https://www.isc.org/software/dhcp/
Config file: /usr/local/etc/dhcpd.conf
Database file: /var/db/dhcpd/dhcpd.leases
PID file: /var/run/dhcpd/dhcpd.pid
Wrote 122 leases to leases file.
Listening on BPF/em0/08:00:27:3c:ab:c8/10.0.10.0/24
Sending on   BPF/em0/08:00:27:3c:ab:c8/10.0.10.0/24
Sending on   Socket/fallback/fallback-net
failover peer ha-dhcp: I move from normal to startup
```

……在 secondary 节点上也是一样。

```sh
root@DHCPs2:~ # sysrc dhcpd_enable=YES
dhcpd_enable:  -> YES
root@DHCPs2:~ # service isc-dhcpd onestart
Starting dhcpd.
Internet Systems Consortium DHCP Server 4.4.1
Copyright 2004-2018 Internet Systems Consortium.
All rights reserved.
For info, please visit https://www.isc.org/software/dhcp/
Config file: /usr/local/etc/dhcpd.conf
Database file: /var/db/dhcpd/dhcpd.leases
PID file: /var/run/dhcpd/dhcpd.pid
Wrote 122 leases to leases file.
Listening on BPF/em0/08:00:27:de:9b:3d/10.0.10.0/24
Sending on   BPF/em0/08:00:27:de:9b:3d/10.0.10.0/24
Sending on   Socket/fallback/fallback-net
failover peer ha-dhcp: I move from communications-interrupted to startup
```

现在，由于高可用 DHCP 服务器的两个节点都已启动，让我们在 DHCP 客户端——在我们的示例中为 **`DHCPc`**——上尝试获取 DHCP 租约。

```sh
root@DHCPc:~ # dhclient em0
DHCPREQUEST on em0 to 255.255.255.255 port 67
DHCPREQUEST on em0 to 255.255.255.255 port 67
DHCPACK from 10.0.10.251
bound to 10.0.10.131 -- renewal in 302119 seconds.
```

```sh
root@DHCPc:~ # ifconfig em0
em0: flags=8843 metric 0 mtu 1500
        options=9b
        ether 08:00:27:d9:45:96
        hwaddr 08:00:27:d9:45:96
        inet 10.0.10.131 netmask 0xffffff00 broadcast 10.0.10.255
        nd6 options=29
        media: Ethernet autoselect (1000baseT )
        status: active
```

我们可以看到 DHCP 客户端 **`DHCPc`** 获得了 10.0.10.131 的地址。

当然，我们可以在 `/usr/local/etc/dhcpd.conf.SHARED` 配置文件中使用 host 选项为它设置固定地址，如下所示。

所需的“额外配置”如下。

``` ini
group
  {
    host DHCPc {
      hardware ethernet 08:00:27:d9:45:96;
      fixed-address 10.0.10.9;
    }
  }
```

需要在两个节点的 `/usr/local/etc/dhcpd.conf.SHARED` 配置文件中都添加，新的共享配置文件如下所示。

```sh
root@DHCPs1:~ # cat /usr/local/etc/dhcpd.conf.SHARED
# CLIENTS
subnet 10.0.10.0 netmask 255.255.255.0 {
  default-lease-time         604800;
  max-lease-time             604800;
  option routers             10.0.10.254;
  option broadcast-address   10.0.10.255;
  option subnet-mask         255.255.255.0;
  option domain-search       "domain.com";
  option domain-name-servers 1.1.1.1,9.9.9.9;

  group
  {
    host DHCPc {
      hardware ethernet 08:00:27:d9:45:96;
      fixed-address 10.0.10.9;
    }
  }

  pool {
    failover peer "ha-dhcp";
    range 10.0.10.10 10.0.10.250;
  }
}
```

现在将 `/usr/local/etc/dhcpd.conf.SHARED` 文件复制到第二个节点。

让我们再次尝试从同一 DHCP 客户端获取地址。

```sh
root@DHCPs1:~ # cat /usr/local/etc/dhcpd.conf.SHARED
# CLIENTS
subnet 10.0.10.0 netmask 255.255.255.0 {
  default-lease-time         604800;
  max-lease-time             604800;
  option routers             10.0.10.254;
  option broadcast-address   10.0.10.255;
  option subnet-mask         255.255.255.0;
  option domain-search       "domain.com";
  option domain-name-servers 1.1.1.1,9.9.9.9;

  group
  {
    host DHCPc {
      hardware ethernet 08:00:27:d9:45:96;
      fixed-address 10.0.10.9;
    }
  }

  pool {
    failover peer "ha-dhcp";
    range 10.0.10.10 10.0.10.250;
  }
}
```

现在将 `/usr/local/etc/dhcpd.conf.SHARED` 文件复制到第二个节点。

让我们再次尝试从同一 DHCP 客户端获取地址。

```sh
root@DHCPc:~ # pkill dhclient
root@DHCPc:~ # service netif restart
root@DHCPc:~ # dhclient em0
DHCPREQUEST on em0 to 255.255.255.255 port 67
DHCPREQUEST on em0 to 255.255.255.255 port 67
DHCPACK from 10.0.10.252
bound to 10.0.10.131 -- renewal in 1665 seconds.
DHCPREQUEST on em0 to 255.255.255.255 port 67
DHCPREQUEST on em0 to 255.255.255.255 port 67
DHCPNAK from 10.0.10.252
DHCPDISCOVER on em0 to 255.255.255.255 port 67 interval 3
DHCPOFFER from 10.0.10.251
DHCPOFFER from 10.0.10.252
DHCPOFFER already seen.
DHCPREQUEST on em0 to 255.255.255.255 port 67
DHCPACK from 10.0.10.252
bound to 10.0.10.9 -- renewal in 302400 seconds.
```

```sh
root@DHCPc:~ # ifconfig em0
em0: flags=8843 metric 0 mtu 1500
options=9b
ether 08:00:27:d9:45:96
hwaddr 08:00:27:d9:45:96
inet 10.0.10.9 netmask 0xffffff00 broadcast 10.0.10.255
nd6 options=29
media: Ethernet autoselect (1000baseT )
status: active
```

现在我们已经获得了固定地址 10.0.10.9。

你现在可以在 `/etc/rc.conf` 文件中尝试修改以下值：

- `dhcpd_flags`
- `dhcpd_ifaces`
- `dhcpd_withumask`
- `dhcpd_chuser_enable`
- `dhcpd_withuser`
- `dhcpd_withgroup`
- `dhcpd_chroot_enable`
- `dhcpd_devfs_enable`
- `dhcpd_rootdir`
- `dhcpd_includedirnclude`

……以及 `man dhcpd.conf` 页面上所有其他可能的选项 🙂

