# 在 FreeBSD 上部署 Syncthing

- [Syncthing on FreeBSD](https://vermaden.wordpress.com/2018/08/21/syncthing-on-freebsd/)
- 作者：𝚟𝚎𝚛𝚖𝚊𝚍𝚎𝚗
- 2018/08

本文将向你说明怎样在 FreeBSD 系统上配置 Syncthing。

![syncthing-logo.png](https://vermaden.wordpress.com/wp-content/uploads/2018/08/syncthing-logo.png?w=960)

>**警告**
>
>请记住，Syncthing 配置文件实际上是 XML 文件。

就我个人的大部分备份需求，我一般使用 **rsync(1)**，但在像手机或平板这样的有限设备上 rsync 实在很麻烦。因此，对于从这些设备自动导入照片和其他文件，我更倾向于使用 Syncthing 工具。

如果你还没听说过，我引用 Syncthing 官网 [https://syncthing.net/](https://syncthing.net/) 的话：**“Syncthing 替代了专有的同步和云服务，提供开源、可靠和去中心化的方案。你的数据完全属于你自己，你有权选择数据存储位置、是否与第三方共享以及在互联网上的传输方式。”** ……以及 [维基百科](https://en.wikipedia.org/wiki/Syncthing) 的描述：**“Syncthing 是一款免费、开源的点对点文件同步应用，适用于 Windows、Mac、Linux、Android、Solaris、Darwin 和 BSD。它可以在局域网设备间同步文件，或通过互联网在远程设备间同步。数据安全和数据保护是软件设计的一部分。”**

有人可能会问，它与 Nextcloud 有何不同。其实，Nextcloud 提供了几乎完整的云服务堆栈以及定制应用，而 Syncthing 仅是设备间的同步工具，仅此而已。

最初，我像设置 [**FreeBSD 上的 Nextcloud**](https://vermaden.wordpress.com/2018/04/04/nextcloud-13-on-freebsd/) 一样，打算在 FreeBSD Jail 中完成全部设置。问题是，我尝试了几个小时后发现 Syncthing 无法在 FreeBSD Jail 虚拟环境中正常工作。管理界面可以访问并正常工作，但 Android 手机上的 Syncthing 无法与 FreeBSD Jail 中的 Syncthing 实例连接和同步。当然，我可以从手机连接到 Syncthing 管理界面，但仍无法使用 Syncthing 协议进行任何备份。了解了这个不足后，你有三种选择：

* 在 FreeBSD 主机上像其他服务一样设置 Syncthing。
* 使用 FreeBSD Bhyve 虚拟化运行 Syncthing 实例。
* 使用 VirtualBox 软件包/Port 运行 Syncthing 实例。

我选择了第一种方案。Bhyve 和 VirtualBox 实际上也是类似，但需要额外处理虚拟化层。我将以基于 Android 的手机作为 Syncthing 客户端示例，但你也可以在计算机之间同步数据。

还有一点，Syncthing 没有服务器端和客户端的区分。所有 Syncthing 实例/安装都是一样的，你只需添加或移除设备及目录来进行同步。我上文中使用“客户端”一词，只是为了说明我将自动化从手机向 FreeBSD 服务器上运行的 Syncthing 实例复制文件，仅此而已。

## 主机

以下是在 FreeBSD 主机上我所做的一些基本步骤，包括别名数据库、时区、DNS 以及 FreeBSD 基本设置，这些都在其 **/etc/rc.conf** 关键文件中配置。

```sh
# newaliases -v
/etc/mail/aliases: 29 aliases, longest 10 bytes, 297 bytes total

# ln -s /usr/share/zoneinfo/Europe/Warsaw /etc/localtime

# date
Fri Aug 17 22:05:18 CEST 2018

# echo nameserver 1.1.1.1 > /etc/resolv.conf

# ping -c 3 freebsd.org
PING freebsd.org (96.47.72.84): 56 data bytes
64 bytes from 96.47.72.84: icmp_seq=0 ttl=51 time=117.918 ms
64 bytes from 96.47.72.84: icmp_seq=1 ttl=51 time=115.169 ms
64 bytes from 96.47.72.84: icmp_seq=2 ttl=51 time=115.392 ms

--- freebsd.org ping statistics ---
3 packets transmitted, 3 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 115.169/116.160/117.918/1.247 ms
```

……以及主要的 FreeBSD 配置文件。

```sh
# cat /etc/rc.conf
# 网络
  hostname=blackbox.local
  ifconfig_re0="inet 10.0.0.100/24 up"
  defaultrouter="10.0.0.1"

# 守护进程 | 启用
  zfs_enable=YES
  sshd_enable=YES
  ntpd_enable=YES
  syncthing_enable=YES
  syslogd_flags="-s -s"

# 守护进程 | 禁用
  sendmail_enable=NONE
  sendmail_submit_enable=NO
  sendmail_outbound_enable=NO
  sendmail_msp_queue_enable=NO

# 其他
  dumpdev=NO
  update_motd=NO
  virecover_enable=NO
  clear_tmp_enable=YES
```

## 安装

首先，为了获取最新的软件包，我们将从 **pkg(8)** 分支 *quarterly* 分支切换到 *latest*。

```sh
# grep url: /etc/pkg/FreeBSD.conf
  url: "pkg+http://pkg.FreeBSD.org/${ABI}/quarterly",

# sed -i '' s/quarterly/latest/g /etc/pkg/FreeBSD.conf

# grep url: /etc/pkg/FreeBSD.conf
  url: "pkg+http://pkg.FreeBSD.org/${ABI}/latest",
```

现在我们将引导安装 **pkg(8)**，然后将其数据库更新到最新可用版本。

```sh
# env ASSUME_ALWAYS_YES=yes pkg update -f
Bootstrapping pkg from pkg+http://pkg.FreeBSD.org/FreeBSD:11:amd64/latest, please wait...
Verifying signature with trusted certificate pkg.freebsd.org.2013102301... done
[syncthing.local] Installing pkg-1.10.5_1...
[syncthing.local] Extracting pkg-1.10.5_1: 100%
Updating FreeBSD repository catalogue...
pkg: Repository FreeBSD load error: access repo file(/var/db/pkg/repo-FreeBSD.sqlite) failed: No such file or directory
[syncthing.local] Fetching meta.txz: 100%    944 B   0.9kB/s    00:01    
[syncthing.local] Fetching packagesite.txz: 100%    6 MiB 352.7kB/s    00:19    
Processing entries: 100%
FreeBSD repository update completed. 32388 packages processed.
All repositories are up to date.
```

……然后从 **pkg(8)** 软件包中安装 Syncthing。

```sh
# pkg install -y syncthing 
Updating FreeBSD repository catalogue...
FreeBSD repository is up to date.
All repositories are up to date.
The following 1 package(s) will be affected (of 0 checked):

New packages to be INSTALLED:
        syncthing: 0.14.48

Number of packages to be installed: 1

The process will require 88 MiB more space.
15 MiB to be downloaded.
[1/1] Fetching syncthing-0.14.48.txz: 100%   15 MiB 525.3kB/s    00:29    
Checking integrity... done (0 conflicting)
[1/1] Installing syncthing-0.14.48...
===> Creating groups.
Creating group 'syncthing' with gid '983'.
===> Creating users
Creating user 'syncthing' with uid '983'.
[1/1] Extracting syncthing-0.14.48: 100%
Message from syncthing-0.14.48:

WARNING: This version is not backwards compatible with 0.13.x, 0.12.x, 0.11.x
nor 0.10.x releases!

For more information, please read:

https://forum.syncthing.net/t/syncthing-v0-14-0/7806
https://github.com/syncthing/syncthing/releases/tag/v0.13.0
https://forum.syncthing.net/t/syncthing-v0-11-0-release-notes/2426
https://forum.syncthing.net/t/syncthing-syncthing-v0-12-0-beryllium-bedbug/6026
```

Syncthing 软件包为我们创建了用户和用户组 **syncthing**。

```sh
# id syncthing
uid=983(syncthing) gid=983(syncthing) groups=983(syncthing)
```

看看 Syncthing 的体积有多小，这些都是软件包 **net/syncthing** 安装的所有文件。

```sh
# pkg info -l syncthing
syncthing-0.14.48:
        /usr/local/bin/stbench
        /usr/local/bin/stcli
        /usr/local/bin/stcompdirs
        /usr/local/bin/stdisco
        /usr/local/bin/stdiscosrv
        /usr/local/bin/stevents
        /usr/local/bin/stfileinfo
        /usr/local/bin/stfinddevice
        /usr/local/bin/stgenfiles
        /usr/local/bin/stindex
        /usr/local/bin/strelaypoolsrv
        /usr/local/bin/strelaysrv
        /usr/local/bin/stsigtool
        /usr/local/bin/sttestutil
        /usr/local/bin/stvanity
        /usr/local/bin/stwatchfile
        /usr/local/bin/syncthing
        /usr/local/etc/rc.d/syncthing
        /usr/local/etc/rc.d/syncthing-discosrv
        /usr/local/etc/rc.d/syncthing-relaypoolsrv
        /usr/local/etc/rc.d/syncthing-relaysrv
        /usr/local/share/doc/syncthing/AUTHORS
        /usr/local/share/doc/syncthing/LICENSE
        /usr/local/share/doc/syncthing/README.md
```

## 配置

如上所示，我们已经在 **/etc/rc.conf** 文件中添加了 **syncthing_enable=YES**。


```sh
# /usr/local/etc/rc.d/syncthing rcvar
# syncthing
#
syncthing_enable="NO"
#   (default: "")

# grep syncthing_enable /etc/rc.conf
  syncthing_enable=YES
```

此外，你还可以从 Syncthing 的 **rc(8)** 启动脚本中查看其他启动选项。

```sh
# less -N /usr/local/etc/rc.d/syncthing
(...)
      9 # Add the following lines to /etc/rc.conf.local or /etc/rc.conf
     10 # to enable this service:
     11 #
     12 # syncthing_enable (bool):      Set to NO by default.
     13 #                               Set it to YES to enable syncthing.
     14 # syncthing_home (path):        Directory where syncthing configuration
     15 #                               data is stored.
     16 #                               Default: /usr/local/etc/syncthing
     17 # syncthing_log_file (path):    Syncthing log file
     18 #                               Default: /var/log/syncthing.log
     19 # syncthing_user (user):        Set user to run syncthing.
     20 #                               Default is "syncthing".
     21 # syncthing_group (group):      Set group to run syncthing.
     22 #                               Default is "syncthing".
(...)
```

Syncthing 需要日志文件 **/var/log/syncthing.log**。让我们创建该文件，并为其设置正确的所有者和权限。

```sh
# ls /var/log/syncthing.log
ls: /var/log/syncthing.log: No such file or directory

# :> /var/log/syncthing.log

# chown syncthing:syncthing /var/log/syncthing.log

# ls -l /var/log/syncthing.log
-rwxr-xr-x  1 syncthing  syncthing  0 2018.08.19 01:06 /var/log/syncthing.log
```

由于我们将使用该日志文件，还需要管理日志轮转，我们将使用 FreeBSD 内置的守护进程 **newsyslog(8)** 来实现。

```sh
# cat > /etc/newsyslog.conf.d/syncthing.conf << __EOF
# logfilename              [owner:group]     mode  count  size  when  flags [/pid_file]
/var/log/syncthing.log  syncthing:syncthing  640   7      100   *     JC
__EOF

# cat /etc/newsyslog.conf.d/syncthing.conf
# logfilename              [owner:group]     mode  count  size  when  flags [/pid_file]
/var/log/syncthing.log  syncthing:syncthing  640   7      100   *     JC

# newsyslog -v | grep syncthing
Processing /etc/newsyslog.conf.d/syncthing
/var/log/syncthing.log : size (Kb): 0 [100] --> skipping
```

让我们尝试初次启动 Syncthing。

```sh
# service syncthing start
Starting syncthing.
daemon: pidfile ``/var/run/syncthing.pid'': Permission denied
/usr/local/etc/rc.d/syncthing: WARNING: failed to start syncthing
```

看来 **rc(8)** 启动 Syncthing 并不会自动创建 PID 文件，那么我们现在手动创建它。

```sh
 
# :> /var/run/syncthing.pid

# chown syncthing:syncthing /var/run/syncthing.pid

# ls -l /var/run/syncthing.pid
-rwxr-xr-x  1 syncthing  syncthing  0 2018.08.19 01:08 /var/run/syncthing.pid
```

现在让我们再次尝试启动 Syncthing。

```sh
# service syncthing start
Starting syncthing.
```

好多了。让我们看看它使用了哪些端口。

```sh
# sockstat -l -4 | grep syncthing
syncthing syncthing 27499 9  tcp46  *:22000               *:*
syncthing syncthing 27499 10 udp4   *:18876               *:*
syncthing syncthing 27499 13 udp4   *:21027               *:*
syncthing syncthing 27499 20 tcp4   127.0.0.1:8384        *:*
```

……并查看它的日志文件。

```sh
# cat /var/log/syncthing.log
[start] 01:08:40 INFO: Generating ECDSA key and certificate for syncthing...
[MPN4S] 01:08:40 INFO: syncthing v0.14.48 "Dysprosium Dragonfly" (go1.10.3 freebsd-amd64) root@111amd64-default-job-12 2018-08-08 09:19:19 UTC [noupgrade]
[MPN4S] 01:08:40 INFO: My ID: MPN4S65-UQWC5SP-3LR2XDB-T5JNYET-VQEQC3X-DSAUI27-BQQKZQE-BWQ3NAO
[MPN4S] 01:08:41 INFO: Single thread SHA256 performance is 131 MB/s using minio/sha256-simd (89 MB/s using crypto/sha256).
[MPN4S] 01:08:41 INFO: Default folder created and/or linked to new config
[MPN4S] 01:08:41 INFO: Default config saved. Edit /usr/local/etc/syncthing/config.xml to taste or use the GUI
[MPN4S] 01:08:42 INFO: Hashing performance is 112.85 MB/s
[MPN4S] 01:08:42 INFO: Updating database schema version from 0 to 2...
[MPN4S] 01:08:42 INFO: Updated symlink type for 0 index entries and added 0 invalid files to global list
[MPN4S] 01:08:42 INFO: Finished updating database schema version from 0 to 2
[MPN4S] 01:08:42 INFO: No stored folder metadata for "default": recalculating
[MPN4S] 01:08:42 WARNING: Creating directory for "Default Folder" (default): mkdir /Sync/: permission denied
[MPN4S] 01:08:42 WARNING: Creating folder marker: folder path missing
[MPN4S] 01:08:42 INFO: Ready to synchronize "Default Folder" (default) (readwrite)
[MPN4S] 01:08:42 INFO: Overall send rate is unlimited, receive rate is unlimited
[MPN4S] 01:08:42 INFO: Rate limits do not apply to LAN connections
[MPN4S] 01:08:42 INFO: Using discovery server https://discovery-v4.syncthing.net/v2/?nolookup&id=LYXKCHX-VI3NYZR-ALCJBHF-WMZYSPK-QG6QJA3-MPFYMSO-U56GTUK-NA2MIAW
[MPN4S] 01:08:42 INFO: Using discovery server https://discovery-v6.syncthing.net/v2/?nolookup&id=LYXKCHX-VI3NYZR-ALCJBHF-WMZYSPK-QG6QJA3-MPFYMSO-U56GTUK-NA2MIAW
[MPN4S] 01:08:42 INFO: Using discovery server https://discovery.syncthing.net/v2/?noannounce&id=LYXKCHX-VI3NYZR-ALCJBHF-WMZYSPK-QG6QJA3-MPFYMSO-U56GTUK-NA2MIAW
[MPN4S] 01:08:42 INFO: TCP listener ([::]:22000) starting
[MPN4S] 01:08:42 INFO: Relay listener (dynamic+https://relays.syncthing.net/endpoint) starting
[MPN4S] 01:08:42 WARNING: Error on folder "Default Folder" (default): folder path missing
[MPN4S] 01:08:42 INFO: Failed initial scan of readwrite folder "Default Folder" (default)
[MPN4S] 01:08:42 INFO: Device MPN4S65-UQWC5SP-3LR2XDB-T5JNYET-VQEQC3X-DSAUI27-BQQKZQE-BWQ3NAO is "blackbox.local" at [dynamic]
[MPN4S] 01:08:42 INFO: Loading HTTPS certificate: open /usr/local/etc/syncthing/https-cert.pem: no such file or directory
[MPN4S] 01:08:42 INFO: Creating new HTTPS certificate
[MPN4S] 01:08:42 INFO: GUI and API listening on 127.0.0.1:8384
[MPN4S] 01:08:42 INFO: Access the GUI via the following URL: http://127.0.0.1:8384/
[MPN4S] 01:08:55 INFO: Joined relay relay://11.12.13.14:443
[MPN4S] 01:09:02 INFO: Detected 1 NAT service
```

这里有几条关于默认 **/Sync** 目录的 **WARNING** 警告信息。让我们来修复它们。

```sh
# service syncthing stop
Stopping syncthing.
Waiting for PIDS: 27498.
```

在第一次启动 Syncthing 时，**rc(8)** 启动脚本创建了目录及其配置文件 **/usr/local/etc/syncthing**。

```sh
# find /usr/local/etc/syncthing
/usr/local/etc/syncthing
/usr/local/etc/syncthing/https-cert.pem
/usr/local/etc/syncthing/https-key.pem
/usr/local/etc/syncthing/cert.pem
/usr/local/etc/syncthing/key.pem
/usr/local/etc/syncthing/config.xml
/usr/local/etc/syncthing/index-v0.14.0.db
/usr/local/etc/syncthing/index-v0.14.0.db/MANIFEST-000000
/usr/local/etc/syncthing/index-v0.14.0.db/LOCK
/usr/local/etc/syncthing/index-v0.14.0.db/000001.log
/usr/local/etc/syncthing/index-v0.14.0.db/LOG
/usr/local/etc/syncthing/index-v0.14.0.db/CURRENT
```

现在让我们回过头来修复 **/Sync** 目录的 **WARNING** 警告。

```sh
# grep '/Sync' /usr/local/etc/syncthing/config.xml
    <folder id="default" label="Default Folder" path="//Sync" type="readwrite" rescanIntervalS="3600" fsWatcherEnabled="true" fsWatcherDelayS="10" ignorePerms="false" autoNormalize="true">

# ls /Sync
ls: /Sync: No such file or directory
```

现在让我们为 Syncthing 实例创建专用目录，并在 **/usr/local/etc/syncthing/config.xml** 配置文件中进行设置。

```sh
# mkdir /syncthing

# chown syncthing:syncthing /syncthing

# chmod 750 /syncthing

# vi /usr/local/etc/syncthing/config.xml

# grep '/syncthing' /usr/local/etc/syncthing/config.xml
    <folder id="default" label="Default Folder" path="/syncthing" type="readwrite" rescanIntervalS="3600" fsWatcherEnabled="true" fsWatcherDelayS="10" ignorePerms="false" autoNormalize="true">
```

我们还将禁用 *Relay* 和 *Global Announce Server*，但保持启用 *Local Announce Server*。

```sh
# grep -i relay /usr/local/etc/syncthing/config.xml
        <relaysEnabled>true</relaysEnabled>
        <relayReconnectIntervalM>10</relayReconnectIntervalM>

# vi /usr/local/etc/syncthing/config.xml

# grep -i relay /usr/local/etc/syncthing/config.xml
        <relaysEnabled>false</relaysEnabled>
        <relayReconnectIntervalM>10</relayReconnectIntervalM>

# grep globalAnnounce /usr/local/etc/syncthing/config.xml
        <globalAnnounceServer>default</globalAnnounceServer>
        <globalAnnounceEnabled>true</globalAnnounceEnabled>

# vi /usr/local/etc/syncthing/config.xml

# grep globalAnnounce /usr/local/etc/syncthing/config.xml
        <globalAnnounceServer>default</globalAnnounceServer>
        <globalAnnounceEnabled>false</globalAnnounceEnabled>
```

在重启 Syncthing 之前，让我们清空 **/var/log/syncthing.log** 文件，以清除不再需要的信息。

```sh
# service syncthing stop
Stopping syncthing.

# :> /var/log/syncthing.log

# service syncthing start
Starting syncthing.
```

让我们看看现在日志中记录了哪些内容。

```sh
# cat /var/log/syncthing.log
[MPN4S] 01:13:38 INFO: syncthing v0.14.48 "Dysprosium Dragonfly" (go1.10.3 freebsd-amd64) root@111amd64-default-job-12 2018-08-08 09:19:19 UTC [noupgrade]
[MPN4S] 01:13:38 INFO: My ID: MPN4S65-UQWC5SP-3LR2XDB-T5JNYET-VQEQC3X-DSAUI27-BQQKZQE-BWQ3NAO
[MPN4S] 01:13:39 INFO: Single thread SHA256 performance is 131 MB/s using minio/sha256-simd (89 MB/s using crypto/sha256).
[MPN4S] 01:13:40 INFO: Hashing performance is 112.97 MB/s
[MPN4S] 01:13:40 INFO: Ready to synchronize "Default Folder" (default) (readwrite)
[MPN4S] 01:13:40 INFO: Overall send rate is unlimited, receive rate is unlimited
[MPN4S] 01:13:40 INFO: Rate limits do not apply to LAN connections
[MPN4S] 01:13:40 INFO: Device MPN4S65-UQWC5SP-3LR2XDB-T5JNYET-VQEQC3X-DSAUI27-BQQKZQE-BWQ3NAO is "blackbox.local" at [dynamic]
[MPN4S] 01:13:40 INFO: TCP listener ([::]:22000) starting
[MPN4S] 01:13:40 INFO: Completed initial scan of readwrite folder "Default Folder" (default)
[MPN4S] 01:13:40 INFO: GUI and API listening on 127.0.0.1:8384
[MPN4S] 01:13:40 INFO: Access the GUI via the following URL: http://127.0.0.1:8384/
```

我们可以看到管理界面监听的是 HTTP 而非 HTTPS，因为 **tls** 选项被设置为 **false**。我们还需要将管理界面的地址从本地主机 (**127.0.0.1**) 切换到我们的 IP 地址 (**10.0.0.100**)。

```sh
# grep -B 1 -A 3 127.0.0.1 /usr/local/etc/syncthing/config.xml
    <gui enabled="true" tls="false" debugging="false">
        <address>127.0.0.1:8384</address>
        <apikey>2jU5aR4zTJLGdEuSLLmdRGgfCgJaUpUv</apikey>
        <theme>default</theme>
    </gui>

# vi /usr/local/etc/syncthing/config.xml

# grep -B 1 -A 3 10.0.0.100 /usr/local/etc/syncthing/config.xml
    <gui enabled="true" tls="true" debugging="false">
        <address>10.0.0.100:8384</address>
        <apikey>2jU5aR4zTJLGdEuSLLmdRGgfCgJaUpUv</apikey>
        <theme>default</theme>
    </gui>
```

现在让我们验证所做的更改。

```sh
# service syncthing stop
Stopping syncthing.

# :> /var/log/syncthing.log

# service syncthing start
Starting syncthing.

# cat /var/log/syncthing.log
[MPN4S] 01:16:20 INFO: syncthing v0.14.48 "Dysprosium Dragonfly" (go1.10.3 freebsd-amd64) root@111amd64-default-job-12 2018-08-08 09:19:19 UTC [noupgrade]
[MPN4S] 01:16:20 INFO: My ID: MPN4S65-UQWC5SP-3LR2XDB-T5JNYET-VQEQC3X-DSAUI27-BQQKZQE-BWQ3NAO
[MPN4S] 01:16:21 INFO: Single thread SHA256 performance is 131 MB/s using minio/sha256-simd (89 MB/s using crypto/sha256).
[MPN4S] 01:16:22 INFO: Hashing performance is 113.07 MB/s
[MPN4S] 01:16:22 INFO: Ready to synchronize "Default Folder" (default) (readwrite)
[MPN4S] 01:16:22 INFO: Overall send rate is unlimited, receive rate is unlimited
[MPN4S] 01:16:22 INFO: Rate limits do not apply to LAN connections
[MPN4S] 01:16:22 INFO: TCP listener ([::]:22000) starting
[MPN4S] 01:16:22 INFO: Completed initial scan of readwrite folder "Default Folder" (default)
[MPN4S] 01:16:22 INFO: Device MPN4S65-UQWC5SP-3LR2XDB-T5JNYET-VQEQC3X-DSAUI27-BQQKZQE-BWQ3NAO is "blackbox.local" at [dynamic]
[MPN4S] 01:16:22 INFO: GUI and API listening on 10.0.0.100:8384
[MPN4S] 01:16:22 INFO: Access the GUI via the following URL: https://10.0.0.100:8384/
[MPN4S] 01:16:42 INFO: Detected 1 NAT service
```

日志现在已经“干净”了，我们可以继续在浏览器中访问 **[https://10.0.0.100:8384](https://10.0.0.100:8384/)** 管理界面，进行 Syncthing 的剩余配置。浏览器当然会提示我们 HTTPS 证书不受信任。

![syncthing-01.png](https://vermaden.wordpress.com/wp-content/uploads/2018/08/syncthing-01.png?w=960)

Syncthing 会询问我们是否同意共享统计数据。你可自行选择。

![syncthing-02.png](https://vermaden.wordpress.com/wp-content/uploads/2018/08/syncthing-02.png?w=960)

Syncthing 仪表盘会显示一个大红色警告，提示远程管理允许在没有密码的情况下访问。我们稍后会修复它，点击警告中的 **Settings** 按钮。

![syncthing-03](https://vermaden.wordpress.com/wp-content/uploads/2018/08/syncthing-03.png?w=960)

保持第一个 **General** 标签页不修改。

![syncthing-04.png](https://vermaden.wordpress.com/wp-content/uploads/2018/08/syncthing-04.png?w=960)

在 **GUI** 标签页中，我们为 Syncthing 管理界面创建用户 **admin**，密码为 **SYNCTHINGPASSWORD**。这里可以使用更合理的密码 🙂。

![syncthing-05.png](https://vermaden.wordpress.com/wp-content/uploads/2018/08/syncthing-05.png?w=960)

我没有修改 **Connections** 标签页中的设置。点击 **Save** 继续。

![syncthing-06.png](https://vermaden.wordpress.com/wp-content/uploads/2018/08/syncthing-06.png?w=960)

除了设置用户及其密码，我没有更改或设置其他选项。

现在 Syncthing 已无错误。系统会提示你输入刚设置的用户和密码。接着我们将删除 **Default Folder**，因为不需要它。点击其 **Edit** 按钮。

![syncthing-07.png](https://vermaden.wordpress.com/wp-content/uploads/2018/08/syncthing-07.png?w=960)

然后点击底部的 **Remove** 按钮。

![syncthing-08.png](https://vermaden.wordpress.com/wp-content/uploads/2018/08/syncthing-08.png?w=960)

……并点击 **Yes** 确认。

![syncthing-09.png](https://vermaden.wordpress.com/wp-content/uploads/2018/08/syncthing-09.png?w=960)

“空白”的 Syncthing 仪表盘。

![syncthing-10.png](https://vermaden.wordpress.com/wp-content/uploads/2018/08/syncthing-10.png?w=960)

接下来，我们将在 Android 手机上下载、安装并配置 Syncthing。根据你的喜好，可使用 **F-Droid** 仓库、**Google Play** 仓库，或直接从任意来源 APK 文件安装。安装后的 Syncthing 应用如下图，约 50 MB。

![syncthing-11](https://vermaden.wordpress.com/wp-content/uploads/2018/08/syncthing-11.png?w=960)

启动应用，你会看到 Syncthing 的 *Welcome* 欢迎界面。

![syncthing-12](https://vermaden.wordpress.com/wp-content/uploads/2018/08/syncthing-12.png?w=960)

根据 Android 版本，手机可能会要求你授权 Syncthing 各种权限，选择同意。

![syncthing-13](https://vermaden.wordpress.com/wp-content/uploads/2018/08/syncthing-13.png?w=960)

与之前一样，Syncthing 会询问你是否同意共享统计数据。我同样留给你选择。

![syncthing-14](https://vermaden.wordpress.com/wp-content/uploads/2018/08/syncthing-14.png?w=960)

Syncthing 现在需要重启，点击 **RESTART NOW** 继续。

![syncthing-15](https://vermaden.wordpress.com/wp-content/uploads/2018/08/syncthing-15.png?w=960)

默认情况下，**Camera** 目录已预配置为 **/storage/emulated/0/DCIM**，用于存放手机拍摄的照片和截图。我将使用它。点击 Syncthing [汉堡菜单](https://en.wikipedia.org/wiki/Hamburger_button) 按钮。

![syncthing-19](https://vermaden.wordpress.com/wp-content/uploads/2018/08/syncthing-19.png?w=960)

选择 **Web GUI** 选项。

![syncthing-20](https://vermaden.wordpress.com/wp-content/uploads/2018/08/syncthing-20.png?w=960)

你将看到 Android 手机上 Syncthing 的管理界面，向下滚动，在 **Remote Devices** 区域添加 FreeBSD 上的 **blackbox.local** Syncthing 实例。

![syncthing-21](https://vermaden.wordpress.com/wp-content/uploads/2018/08/syncthing-21.png?w=960)

在 **Remote Devices** 区域点击 **Add Remote Device** 按钮。

![syncthing-22](https://vermaden.wordpress.com/wp-content/uploads/2018/08/syncthing-22.png?w=960)

记住之前我们启用的 *Local Announce* 服务吗？此时它派上用场。FreeBSD 上的 Syncthing 实例 ID 会自动在网络中显示。

![syncthing-23](https://vermaden.wordpress.com/wp-content/uploads/2018/08/syncthing-23.png?w=960)

点击显示的 ID，并输入 **blackbox.local** 主机名。

除了输入（点击）ID 和主机名，我没有设置其他选项。点击 **Save**。

![syncthing-24](https://vermaden.wordpress.com/wp-content/uploads/2018/08/syncthing-24.png?w=960)

**blackbox.local** 现在已添加到 **Remote Devices** 列表中。

![syncthing-25](https://vermaden.wordpress.com/wp-content/uploads/2018/08/syncthing-25.png?w=960)

以下是 **Camera** 目录属性。记得选择 **blackbox.local** 作为允许的主机（小黄滑块）。

![syncthing-26](https://vermaden.wordpress.com/wp-content/uploads/2018/08/syncthing-26.png?w=960)

……以及 **blackbox.local** 设备属性。

![syncthing-27](https://vermaden.wordpress.com/wp-content/uploads/2018/08/syncthing-27.png?w=960)

现在返回 FreeBSD 上的 Syncthing 管理界面，你会被提示将 Android 手机上的 Syncthing（在我这里是 **SM-A320FL**）添加到设备列表。点击绿色 **Add Device** 按钮。

![syncthing-28.png](https://vermaden.wordpress.com/wp-content/uploads/2018/08/syncthing-28.png?w=960)

无需添加其他选项，直接点击 **Save**。

![syncthing-29.png](https://vermaden.wordpress.com/wp-content/uploads/2018/08/syncthing-291.png?w=960)

Android 手机 **SM-A320FL** 设备现在在 **Remote Devices** 区域可见。

![syncthing-30.png](https://vermaden.wordpress.com/wp-content/uploads/2018/08/syncthing-301.png?w=960)

系统会提示 **SM-A320FL** 设备想要共享 **Camera** 目录，点击绿色 **Add** 按钮。

![syncthing-31.png](https://vermaden.wordpress.com/wp-content/uploads/2018/08/syncthing-31.png?w=960)

输入 **SM-A320FL** 作为文件夹标签，**/syncthing/SM-A320FL** 作为 FreeBSD Syncthing 实例上的目录名。确保在底部 **Share With Devices** 区域选择了 **SM-A320FL**。

![syncthing-32.png](https://vermaden.wordpress.com/wp-content/uploads/2018/08/syncthing-32.png?w=960)

现在 **SM-A320FL** 设备及其 **SM-A320FL** 文件夹已配置完成。你将首先看到 **Out of Sync** 状态。同步将开始，其进度可在手机和 FreeBSD Syncthing 管理界面中观察。

![syncthing-33.png](https://vermaden.wordpress.com/wp-content/uploads/2018/08/syncthing-33.png?w=960)

**SM-A320FL** 文件夹状态切换为 **Syncing**，显示同步进度。

![syncthing-34.png](https://vermaden.wordpress.com/wp-content/uploads/2018/08/syncthing-34.png?w=960)

在 Android 手机上也会看到类似状态。

![syncthing-36](https://vermaden.wordpress.com/wp-content/uploads/2018/08/syncthing-36.png?w=960)

一段时间后，**SM-A320FL** 文件夹状态显示 **Up to Date**，表示 **Camera** 目录中的所有文件已同步到 FreeBSD Syncthing 实例。

![syncthing-35](https://vermaden.wordpress.com/wp-content/uploads/2018/08/syncthing-35.png?w=960)

在 FreeBSD Syncthing 实例上，来自 Android 手机的已创建/同步目录如下所示。

```sh
# find /syncthing -type d
/syncthing
/syncthing/SM-A320FL
/syncthing/SM-A320FL/Camera
/syncthing/SM-A320FL/Camera/.AutoPortrait
/syncthing/SM-A320FL/Screenshots
/syncthing/SM-A320FL/.thumbnails
/syncthing/SM-A320FL/.stfolder
```

现在，你的 Camera 文件已经同步完成，可作为备份使用。

FreeBSD 实例上的完整 Syncthing 配置文件可在此获取：[**/usr/local/etc/syncthing/config.xml**](https://vermaden.wordpress.com/wp-content/uploads/2018/08/config-xml.key "config.xml")。下载后，将文件从 ***.xml.key** 重命名为 ***.xml**（**WordPress** 限制所致）。

## 更新 1

[FreeBSD 上的 Syncthing](https://vermaden.wordpress.com/2018/08/21/syncthing-on-freebsd/) 文章曾在 [BSD Now 262 – OpenBSD Surfacing](https://www.jupiterbroadcasting.com/127006/openbsd-surfacing-bsd-now-262/) 节目中提及。

感谢分享！
