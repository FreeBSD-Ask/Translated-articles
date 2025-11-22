# 使用 Pacemaker 与 Corosync 构建 FreeBSD 集群

- 原文：[FreeBSD Cluster with Pacemaker and Corosync](https://vermaden.wordpress.com/2020/09/03/freebsd-cluster-with-pacemaker-and-corosync/)
- 作者：𝚟𝚎𝚛𝚖𝚊𝚍𝚎𝚗
- 2020/09/03

我一直很想找到适合 FreeBSD 系统的“正统”集群软件。最近我有机会在 Linux 系统上运行几个基于 Pacemaker/Corosync 的集群。我开始思考如何在 FreeBSD 上实现类似的高可用性解决方案，当我发现 Pacemaker 和 Corosync 工具在 FreeBSD Ports 和包中分别可以通过 **net/pacemaker2** 和 **net/corosync2** 获得时，我真的非常震惊。

在本文中，我将检查 Pacemaker 和 Corosync 集群在 FreeBSD 上的运行情况。

![pacemaker](https://vermaden.wordpress.com/wp-content/uploads/2020/09/pacemaker.png?w=960)

关于集群，有很多定义。我最喜欢的一个定义是：集群是即使失去其中一个节点仍保持冗余的系统（仍然是集群）。这意味着根据这个定义，集群的最少节点数是 3 个。两节点集群存在较大问题，因为它们最容易出现脑裂问题。这就是为什么在两节点集群中，通常会添加额外的设备或系统，以确保脑裂不会发生。例如，可以添加第三个节点，不提供任何资源或服务，仅作为“见证者”角色。另一种方式是添加共享磁盘资源，其作用相同，通常是使用 SCSI-3 持久保留机制的原始卷。

# 实验室搭建

和以往一样，整个实验将基于 VirtualBox，并由 3 台主机组成。为了不创建 3 个相同的 FreeBSD 安装，我使用了 FreeBSD 项目直接提供的 12.1-RELEASE 虚拟机镜像：

* [https://download.freebsd.org/ftp/releases/VM-IMAGES/12.1-RELEASE/](https://download.freebsd.org/ftp/releases/VM-IMAGES/12.1-RELEASE/)

有几种格式可选 – **qcow2**/**raw**/**vhd**/**vmdk** – 因为我将使用 VirtualBox，所以选择了 [VMDK](https://download.freebsd.org/ftp/releases/VM-IMAGES/12.1-RELEASE/amd64/Latest/FreeBSD-12.1-RELEASE-amd64.vmdk.xz) 格式。

下面是 GlusterFS 集群的主机列表：

* 10.0.10.111 node1
* 10.0.10.112 node2
* 10.0.10.113 node3

每个 VirtualBox 的 FreeBSD 虚拟机均为默认配置（如 VirtualBox 向导所建议），内存为 512 MB，网络模式为 *NAT Network*，如下图所示。

![machine](https://vermaden.wordpress.com/wp-content/uploads/2020/09/machine-1.jpg?w=960)

下面是 VirtualBox 上 *NAT Network* 的配置。

![nat-network-01](https://vermaden.wordpress.com/wp-content/uploads/2020/09/nat-network-01.jpg?w=960)

![nat-network-02](https://vermaden.wordpress.com/wp-content/uploads/2020/09/nat-network-02.jpg?w=960)

在尝试连接 FreeBSD 主机之前，需要在每台虚拟机内进行最小网络配置。每台 FreeBSD 主机将具有如下示例的最小 **/etc/rc.conf** 文件（以 **node1** 为例）。

```sh
root@node1:~ # cat /etc/rc.conf
hostname=node1
ifconfig_em0="inet 10.0.10.111/24 up"
defaultrouter=10.0.10.1
sshd_enable=YES
```

为了搭建实验环境，我们需要允许 **root** 登录这些 FreeBSD 主机，在 **/etc/ssh/sshd_config** 文件中设置 **PermitRootLogin yes**。更改后，还需要重启 **sshd(8)** 服务。

```sh
root@node1:~ # grep PermitRootLogin /etc/ssh/sshd_config
PermitRootLogin yes

root@node1:~ # service sshd restart
```

通过使用带有 *端口转发* 的 NAT Network，FreeBSD 主机将可以通过本地主机端口访问。例如，**node1** 可以通过端口 **2211** 访问，**node2** 可以通过端口 **2212** 访问，依此类推。如下 **sockstat** 工具输出所示。


![nat-network-03-sockstat](https://vermaden.wordpress.com/wp-content/uploads/2020/09/nat-network-03-sockstat.jpg?w=960)

![nat-network-04-ssh](https://vermaden.wordpress.com/wp-content/uploads/2020/09/nat-network-04-ssh.jpg?w=960)

要从 VirtualBox 主机系统连接到这样的虚拟机，需要使用如下命令：

```
vboxhost % ssh -l root localhost -p 2211
```

# 软件包

现在我们已经可以通过 **ssh(1)** 连接，需要添加所需的软件包。为了让我们的虚拟机能够解析 DNS 查询，还需要做最后一件事。同时，我们将切换 **pkg(8)** 软件包到 “quarterly” 分支。

```
root@node1:~ # echo 'nameserver 1.1.1.1' > /etc/resolv.conf
root@node1:~ # sed -i '' s/quarterly/latest/g /etc/pkg/FreeBSD.conf
```

请记得在 **node2** 和 **node3** 系统上重复执行上述两条命令。

现在我们将安装 Pacemaker 和 Corosync 软件包。

```
root@node1:~ # pkg install pacemaker2 corosync2 crmsh

root@node2:~ # pkg install pacemaker2 corosync2 crmsh

root@node3:~ # pkg install pacemaker2 corosync2 crmsh
```

下面是 **pacemaker2** 和 **corosync2** 的一些需要我们处理的信息。

```sh
Message from pacemaker2-2.0.4:

--
For correct operation, maximum socket buffer size must be tuned
by performing the following command as root :

# sysctl kern.ipc.maxsockbuf=18874368

To preserve this setting across reboots, append the following
to /etc/sysctl.conf :

kern.ipc.maxsockbuf=18874368

======================================================================

Message from corosync2-2.4.5_1:

--
For correct operation, maximum socket buffer size must be tuned
by performing the following command as root :

# sysctl kern.ipc.maxsockbuf=18874368

To preserve this setting across reboots, append the following
to /etc/sysctl.conf :

kern.ipc.maxsockbuf=18874368
```

我们需要修改 **kern.ipc.maxsockbuf** 参数。那就开始修改吧。

```sh
root@node1:~ # echo 'kern.ipc.maxsockbuf=18874368' >> /etc/sysctl.conf
root@node1:~ # service sysctl restart

root@node2:~ # echo 'kern.ipc.maxsockbuf=18874368' >> /etc/sysctl.conf
root@node2:~ # service sysctl restart

root@node3:~ # echo 'kern.ipc.maxsockbuf=18874368' >> /etc/sysctl.conf
root@node3:~ # service sysctl restart
```

我们来查看这些软件包都包含了哪些可执行文件。

```sh
root@node1:~ # pkg info -l pacemaker2 | grep bin
        /usr/local/sbin/attrd_updater
        /usr/local/sbin/cibadmin
        /usr/local/sbin/crm_attribute
        /usr/local/sbin/crm_diff
        /usr/local/sbin/crm_error
        /usr/local/sbin/crm_failcount
        /usr/local/sbin/crm_master
        /usr/local/sbin/crm_mon
        /usr/local/sbin/crm_node
        /usr/local/sbin/crm_report
        /usr/local/sbin/crm_resource
        /usr/local/sbin/crm_rule
        /usr/local/sbin/crm_shadow
        /usr/local/sbin/crm_simulate
        /usr/local/sbin/crm_standby
        /usr/local/sbin/crm_ticket
        /usr/local/sbin/crm_verify
        /usr/local/sbin/crmadmin
        /usr/local/sbin/fence_legacy
        /usr/local/sbin/iso8601
        /usr/local/sbin/pacemaker-remoted
        /usr/local/sbin/pacemaker_remoted
        /usr/local/sbin/pacemakerd
        /usr/local/sbin/stonith_admin

root@node1:~ # pkg info -l corosync2 | grep bin
        /usr/local/bin/corosync-blackbox
        /usr/local/sbin/corosync
        /usr/local/sbin/corosync-cfgtool
        /usr/local/sbin/corosync-cmapctl
        /usr/local/sbin/corosync-cpgtool
        /usr/local/sbin/corosync-keygen
        /usr/local/sbin/corosync-notifyd
        /usr/local/sbin/corosync-quorumtool

root@node1:~ # pkg info -l crmsh | grep bin
        /usr/local/bin/crm
```

# 集群初始化

现在我们将初始化 FreeBSD 集群。

首先需要确保各节点的名称可以通过 DNS 解析。

```sh
root@node1:~ # tail -3 /etc/hosts

10.0.10.111 node1
10.0.10.112 node2
10.0.10.113 node3

root@node2:~ # tail -3 /etc/hosts

10.0.10.111 node1
10.0.10.112 node2
10.0.10.113 node3

root@node3:~ # tail -3 /etc/hosts

10.0.10.111 node1
10.0.10.112 node2
10.0.10.113 node3
```

现在我们将生成 Corosync 密钥。

```sh
root@node1:~ # corosync-keygen
Corosync Cluster Engine Authentication key generator.
Gathering 1024 bits for key from /dev/random.
Press keys on your keyboard to generate entropy.
Writing corosync key to /usr/local/etc/corosync/authkey.

root@node1:~ # echo $?
0

root@node1:~ # ls -l /usr/local/etc/corosync/authkey
-r--------  1 root  wheel  128 Sep  2 20:37 /usr/local/etc/corosync/authkey
```

现在是 Corosync 配置文件的部分。当然，软件包维护者已经提供了一些示例。

```sh
root@node1:~ # pkg info -l corosync2 | grep example
        /usr/local/etc/corosync/corosync.conf.example
        /usr/local/etc/corosync/corosync.conf.example.udpu
```

我们将使用第二个示例作为配置的基础。

```sh
root@node1:~ # cp /usr/local/etc/corosync/corosync.conf.example.udpu /usr/local/etc/corosync/corosync.conf

root@node1:~ # vi /usr/local/etc/corosync/corosync.conf
               /* LOTS OF EDITS HERE */

root@node1:~ # cat /usr/local/etc/corosync/corosync.conf

totem {
  version: 2
  crypto_cipher: aes256
  crypto_hash: sha256
  transport: udpu

  interface {
    ringnumber: 0
    bindnetaddr: 10.0.10.0
    mcastport: 5405
    ttl: 1
  }
}

logging {
  fileline: off
  to_logfile: yes
  to_syslog: no
  logfile: /var/log/cluster/corosync.log
  debug: off
  timestamp: on

  logger_subsys {
    subsys: QUORUM
    debug: off
  }
}

nodelist {

  node {
    ring0_addr: 10.0.10.111
    nodeid: 1
  }

  node {
    ring0_addr: 10.0.10.112
    nodeid: 2
  }

  node {
    ring0_addr: 10.0.10.113
    nodeid: 3
  }

}

quorum {
  provider: corosync_votequorum
  expected_votes: 2
}
```

现在我们需要将 Corosync 密钥和配置文件分发到集群中的各个节点。

可以使用一些专门为此创建的简单工具，例如 net/csync2 集群同步工具，但传统的 net/rsync 也同样可用。

```sh
root@node1:~ # pkg install -y rsync

root@node1:~ # rsync -av /usr/local/etc/corosync/ node2:/usr/local/etc/corosync/
The authenticity of host 'node2 (10.0.10.112)' can't be established.
ECDSA key fingerprint is SHA256:/ZDmln7GKi6n0kbad73TIrajPjGfQqJJX+ReSf3NMvc.
No matching host key fingerprint found in DNS.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'node2' (ECDSA) to the list of known hosts.
Password for root@node2:
sending incremental file list
./
authkey
corosync.conf
service.d/
uidgid.d/

sent 1,100 bytes  received 69 bytes  259.78 bytes/sec
total size is 4,398  speedup is 3.76

root@node1:~ # rsync -av /usr/local/etc/corosync/ node3:/usr/local/etc/corosync/
The authenticity of host 'node2 (10.0.10.112)' can't be established.
ECDSA key fingerprint is SHA256:/ZDmln7GKi6n0kbad73TIrajPjGfQqJJX+ReSf3NMvc.
No matching host key fingerprint found in DNS.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'node3' (ECDSA) to the list of known hosts.
Password for root@node3:
sending incremental file list
./
authkey
corosync.conf
service.d/
uidgid.d/

sent 1,100 bytes  received 69 bytes  259.78 bytes/sec
total size is 4,398  speedup is 3.76
```

现在我们来检查它们是否一致。

```sh
root@node1:~ # cksum /usr/local/etc/corosync/{authkey,corosync.conf}
2277171666 128 /usr/local/etc/corosync/authkey
1728717329 622 /usr/local/etc/corosync/corosync.conf

root@node2:~ # cksum /usr/local/etc/corosync/{authkey,corosync.conf}
2277171666 128 /usr/local/etc/corosync/authkey
1728717329 622 /usr/local/etc/corosync/corosync.conf

root@node3:~ # cksum /usr/local/etc/corosync/{authkey,corosync.conf}
2277171666 128 /usr/local/etc/corosync/authkey
1728717329 622 /usr/local/etc/corosync/corosync.conf
```

一致。

现在我们可以在 **/etc/rc.conf** 文件中添加 **corosync_enable=YES** 和 **pacemaker_enable=YES**。

```sh
root@node1:~ # sysrc corosync_enable=YES
corosync_enable:  -> YES

root@node1:~ # sysrc pacemaker_enable=YES
pacemaker_enable:  -> YES

root@node2:~ # sysrc corosync_enable=YES
corosync_enable:  -> YES

root@node2:~ # sysrc pacemaker_enable=YES
pacemaker_enable:  -> YES

root@node3:~ # sysrc corosync_enable=YES
corosync_enable:  -> YES

root@node3:~ # sysrc pacemaker_enable=YES
pacemaker_enable:  -> YES
```

那就启动这些服务吧。

```sh
root@node1:~ # service corosync start
Starting corosync.
Sep 02 20:55:35 notice  [MAIN  ] Corosync Cluster Engine ('2.4.5'): started and ready to provide service.
Sep 02 20:55:35 info    [MAIN  ] Corosync built-in features:
Sep 02 20:55:35 warning [MAIN  ] interface section bindnetaddr is used together with nodelist. Nodelist one is going to be used.
Sep 02 20:55:35 warning [MAIN  ] Please migrate config file to nodelist.

root@node1:~ # ps aux | grep corosync
root  1695   0.0  7.9 38340 38516  -  S    20:55    0:00.40 /usr/local/sbin/corosync
root  1699   0.0  0.1   524   336  0  R+   20:57    0:00.00 grep corosync
```

在 **node2** 和 **node3** 系统上也执行同样操作。

由于 Pacemaker 还未运行，因此操作会失败。

```sh
root@node1:~ # crm status
Could not connect to the CIB: Socket is not connected
crm_mon: Error: cluster is not available on this node
ERROR: status: crm_mon (rc=102):
```

现在我们将启动它。

```sh
root@node1:~ # service pacemaker start
Starting pacemaker.

root@node2:~ # service pacemaker start
Starting pacemaker.

root@node3:~ # service pacemaker start
Starting pacemaker.
```

需要给它一些启动时间，因为如果你立即执行 **crm status** 命令，会看到如下所示的 **0 nodes configured** 消息。

```sh
root@node1:~ # crm status
Cluster Summary:
  * Stack: unknown
  * Current DC: NONE
  * Last updated: Wed Sep  2 20:58:51 2020
  * Last change:  
  * 0 nodes configured
  * 0 resource instances configured


Full List of Resources:
  * No resources
```

……但过一会儿，所有节点都会被检测到，一切按预期正常工作。

```sh
root@node1:~ # crm status
Cluster Summary:
  * Stack: corosync
  * Current DC: node2 (version 2.0.4-2deceaa3ae) - partition with quorum
  * Last updated: Wed Sep  2 21:02:49 2020
  * Last change:  Wed Sep  2 20:59:00 2020 by hacluster via crmd on node2
  * 3 nodes configured
  * 0 resource instances configured

Node List:
  * Online: [ node1 node2 node3 ]

Full List of Resources:
  * No resources
```

Pacemaker 运行正常。

```sh
root@node1:~ # ps aux | grep pacemaker
root      1716   0.0  0.5 10844   2396  -  Is   20:58     0:00.00 daemon: /usr/local/sbin/pacemakerd[1717] (daemon)
root      1717   0.0  5.2 49264  25284  -  S    20:58     0:00.27 /usr/local/sbin/pacemakerd
hacluster 1718   0.0  6.1 48736  29708  -  Ss   20:58     0:00.75 /usr/local/libexec/pacemaker/pacemaker-based
root      1719   0.0  4.5 40628  21984  -  Ss   20:58     0:00.28 /usr/local/libexec/pacemaker/pacemaker-fenced
root      1720   0.0  2.8 25204  13688  -  Ss   20:58     0:00.20 /usr/local/libexec/pacemaker/pacemaker-execd
hacluster 1721   0.0  3.9 38148  19100  -  Ss   20:58     0:00.25 /usr/local/libexec/pacemaker/pacemaker-attrd
hacluster 1722   0.0  2.9 25460  13864  -  Ss   20:58     0:00.17 /usr/local/libexec/pacemaker/pacemaker-schedulerd
hacluster 1723   0.0  5.4 49304  26300  -  Ss   20:58     0:00.41 /usr/local/libexec/pacemaker/pacemaker-controld
root      1889   0.0  0.6 11348   2728  0  S+   21:56     0:00.00 grep pacemaker
```

我们可以查看 Corosync 如何识别其成员。

```sh
root@node1:~ # corosync-cmapctl | grep members
runtime.totem.pg.mrp.srp.members.1.config_version (u64) = 0
runtime.totem.pg.mrp.srp.members.1.ip (str) = r(0) ip(10.0.10.111) 
runtime.totem.pg.mrp.srp.members.1.join_count (u32) = 1
runtime.totem.pg.mrp.srp.members.1.status (str) = joined
runtime.totem.pg.mrp.srp.members.2.config_version (u64) = 0
runtime.totem.pg.mrp.srp.members.2.ip (str) = r(0) ip(10.0.10.112) 
runtime.totem.pg.mrp.srp.members.2.join_count (u32) = 1
runtime.totem.pg.mrp.srp.members.2.status (str) = joined
runtime.totem.pg.mrp.srp.members.3.config_version (u64) = 0
runtime.totem.pg.mrp.srp.members.3.ip (str) = r(0) ip(10.0.10.113) 
runtime.totem.pg.mrp.srp.members.3.join_count (u32) = 1
runtime.totem.pg.mrp.srp.members.3.status (str) = joined
```

……或者查看 quorum 信息。

```sh
root@node1:~ # corosync-quorumtool
Quorum information
------------------
Date:             Wed Sep  2 21:00:38 2020
Quorum provider:  corosync_votequorum
Nodes:            3
Node ID:          1
Ring ID:          1/12
Quorate:          Yes

Votequorum information
----------------------
Expected votes:   3
Highest expected: 3
Total votes:      3
Quorum:           2  
Flags:            Quorate 

Membership information
----------------------
    Nodeid      Votes Name
         1          1 10.0.10.111 (local)
         2          1 10.0.10.112
         3          1 10.0.10.113
```

Corosync 日志文件中充满了以下信息。

```sh
root@node1:~ # cat /var/log/cluster/corosync.log
Sep 02 20:55:35 [1694] node1 corosync notice  [MAIN  ] Corosync Cluster Engine ('2.4.5'): started and ready to provide service.
Sep 02 20:55:35 [1694] node1 corosync info    [MAIN  ] Corosync built-in features:
Sep 02 20:55:35 [1694] node1 corosync warning [MAIN  ] interface section bindnetaddr is used together with nodelist. Nodelist one is going to be used.
Sep 02 20:55:35 [1694] node1 corosync warning [MAIN  ] Please migrate config file to nodelist.
Sep 02 20:55:35 [1694] node1 corosync notice  [TOTEM ] Initializing transport (UDP/IP Unicast).
Sep 02 20:55:35 [1694] node1 corosync notice  [TOTEM ] Initializing transmit/receive security (NSS) crypto: aes256 hash: sha256
Sep 02 20:55:35 [1694] node1 corosync notice  [TOTEM ] The network interface [10.0.10.111] is now up.
Sep 02 20:55:35 [1694] node1 corosync notice  [SERV  ] Service engine loaded: corosync configuration map access [0]
Sep 02 20:55:35 [1694] node1 corosync info    [QB    ] server name: cmap
Sep 02 20:55:35 [1694] node1 corosync notice  [SERV  ] Service engine loaded: corosync configuration service [1]
Sep 02 20:55:35 [1694] node1 corosync info    [QB    ] server name: cfg
Sep 02 20:55:35 [1694] node1 corosync notice  [SERV  ] Service engine loaded: corosync cluster closed process group service v1.01 [2]
Sep 02 20:55:35 [1694] node1 corosync info    [QB    ] server name: cpg
Sep 02 20:55:35 [1694] node1 corosync notice  [SERV  ] Service engine loaded: corosync profile loading service [4]
Sep 02 20:55:35 [1694] node1 corosync notice  [QUORUM] Using quorum provider corosync_votequorum
Sep 02 20:55:35 [1694] node1 corosync notice  [SERV  ] Service engine loaded: corosync vote quorum service v1.0 [5]
Sep 02 20:55:35 [1694] node1 corosync info    [QB    ] server name: votequorum
Sep 02 20:55:35 [1694] node1 corosync notice  [SERV  ] Service engine loaded: corosync cluster quorum service v0.1 [3]
Sep 02 20:55:35 [1694] node1 corosync info    [QB    ] server name: quorum
Sep 02 20:55:35 [1694] node1 corosync notice  [TOTEM ] adding new UDPU member {10.0.10.111}
Sep 02 20:55:35 [1694] node1 corosync notice  [TOTEM ] adding new UDPU member {10.0.10.112}
Sep 02 20:55:35 [1694] node1 corosync notice  [TOTEM ] adding new UDPU member {10.0.10.113}
Sep 02 20:55:35 [1694] node1 corosync notice  [TOTEM ] A new membership (10.0.10.111:4) was formed. Members joined: 1
Sep 02 20:55:35 [1694] node1 corosync warning [CPG   ] downlist left_list: 0 received
Sep 02 20:55:35 [1694] node1 corosync notice  [QUORUM] Members[1]: 1
Sep 02 20:55:35 [1694] node1 corosync notice  [MAIN  ] Completed service synchronization, ready to provide service.
Sep 02 20:58:14 [1694] node1 corosync notice  [TOTEM ] A new membership (10.0.10.111:8) was formed. Members joined: 2
Sep 02 20:58:14 [1694] node1 corosync warning [CPG   ] downlist left_list: 0 received
Sep 02 20:58:14 [1694] node1 corosync warning [CPG   ] downlist left_list: 0 received
Sep 02 20:58:14 [1694] node1 corosync notice  [QUORUM] This node is within the primary component and will provide service.
Sep 02 20:58:14 [1694] node1 corosync notice  [QUORUM] Members[2]: 1 2
Sep 02 20:58:14 [1694] node1 corosync notice  [MAIN  ] Completed service synchronization, ready to provide service.
Sep 02 20:58:19 [1694] node1 corosync notice  [TOTEM ] A new membership (10.0.10.111:12) was formed. Members joined: 3
Sep 02 20:58:19 [1694] node1 corosync warning [CPG   ] downlist left_list: 0 received
Sep 02 20:58:19 [1694] node1 corosync warning [CPG   ] downlist left_list: 0 received
Sep 02 20:58:19 [1694] node1 corosync warning [CPG   ] downlist left_list: 0 received
Sep 02 20:58:19 [1694] node1 corosync notice  [QUORUM] Members[3]: 1 2 3
Sep 02 20:58:19 [1694] node1 corosync notice  [MAIN  ] Completed service synchronization, ready to provide service.
```

配置如下。

```sh
root@node1:~ # crm configure show
node 1: node1
node 2: node2
node 3: node3
property cib-bootstrap-options: \
        have-watchdog=false \
        dc-version=2.0.4-2deceaa3ae \
        cluster-infrastructure=corosync
```

由于我们不会配置 STONITH 机制，因此将其禁用。

```sh
root@node1:~ # crm configure property stonith-enabled=false
```

禁用 STONITH 后的新配置。

```sh
root@node1:~ # crm configure show
node 1: node1
node 2: node2
node 3: node3
property cib-bootstrap-options: \
        have-watchdog=false \
        dc-version=2.0.4-2deceaa3ae \
        cluster-infrastructure=corosync \
        stonith-enabled=false
```

STONITH 配置超出了本文的范围，但正确配置的 STONITH 如下所示。

![stonith](https://vermaden.wordpress.com/wp-content/uploads/2020/09/stonith.jpg?w=960)

# 第一个服务

现在我们将配置第一个高可用服务——经典示例——一个浮动 IP 地址 :🙂:

```sh
root@node1:~ # crm configure primitive IP ocf:heartbeat:IPaddr2 params ip=10.0.10.200 cidr_netmask="24" op monitor interval="30s"
```

让我们看看它的运行情况。

```sh
root@node1:~ # crm configure show
node 1: node1
node 2: node2
node 3: node3
primitive IP IPaddr2 \
        params ip=10.0.10.200 cidr_netmask=24 \
        op monitor interval=30s
property cib-bootstrap-options: \
        have-watchdog=false \
        dc-version=2.0.4-2deceaa3ae \
        cluster-infrastructure=corosync \
        stonith-enabled=false
```

看起来不错——我们来检查一下集群状态。

```sh
root@node1:~ # crm status
Cluster Summary:
  * Stack: corosync
  * Current DC: node2 (version 2.0.4-2deceaa3ae) - partition with quorum
  * Last updated: Wed Sep  2 22:03:35 2020
  * Last change:  Wed Sep  2 22:02:53 2020 by root via cibadmin on node1
  * 3 nodes configured
  * 1 resource instance configured

Node List:
  * Online: [ node1 node2 node3 ]

Full List of Resources:
  * IP  (ocf::heartbeat:IPaddr2):        Stopped

Failed Resource Actions:
  * IP_monitor_0 on node3 'not installed' (5): call=5, status='complete', exitreason='Setup problem: couldn't find command: ip', last-rc-change='2020-09-02 22:02:53Z', queued=0ms, exec=132ms
  * IP_monitor_0 on node2 'not installed' (5): call=5, status='complete', exitreason='Setup problem: couldn't find command: ip', last-rc-change='2020-09-02 22:02:54Z', queued=0ms, exec=120ms
  * IP_monitor_0 on node1 'not installed' (5): call=5, status='complete', exitreason='Setup problem: couldn't find command: ip', last-rc-change='2020-09-02 22:02:53Z', queued=0ms, exec=110ms
```

糟糕。Linux 思维模式。系统中预期存在 **ip(8)** 命令。但这是 FreeBSD，像所有 UNIX 系统一样，它提供的是 **ifconfig(8)** 命令。

我们必须另想办法。目前我们先删除这个无用的 **IP** 服务。

```sh
root@node1:~ # crm configure delete IP
```

删除后的状态。

```sh
root@node1:~ # crm status
Cluster Summary:
  * Stack: corosync
  * Current DC: node2 (version 2.0.4-2deceaa3ae) - partition with quorum
  * Last updated: Wed Sep  2 22:04:34 2020
  * Last change:  Wed Sep  2 22:04:31 2020 by root via cibadmin on node1
  * 3 nodes configured
  * 0 resource instances configured

Node List:
  * Online: [ node1 node2 node3 ]

Full List of Resources:
  * No resources
```

# 自定义资源

让我们查看默认 Pacemaker 安装提供了哪些资源。

```sh
root@node1:~ # ls -l /usr/local/lib/ocf/resource.d/pacemaker
total 144
-r-xr-xr-x  1 root  wheel   7484 Aug 29 01:22 ClusterMon
-r-xr-xr-x  1 root  wheel   9432 Aug 29 01:22 Dummy
-r-xr-xr-x  1 root  wheel   5256 Aug 29 01:22 HealthCPU
-r-xr-xr-x  1 root  wheel   5342 Aug 29 01:22 HealthIOWait
-r-xr-xr-x  1 root  wheel   9450 Aug 29 01:22 HealthSMART
-r-xr-xr-x  1 root  wheel   6186 Aug 29 01:22 Stateful
-r-xr-xr-x  1 root  wheel  11370 Aug 29 01:22 SysInfo
-r-xr-xr-x  1 root  wheel   5856 Aug 29 01:22 SystemHealth
-r-xr-xr-x  1 root  wheel   7382 Aug 29 01:22 attribute
-r-xr-xr-x  1 root  wheel   7854 Aug 29 01:22 controld
-r-xr-xr-x  1 root  wheel  16134 Aug 29 01:22 ifspeed
-r-xr-xr-x  1 root  wheel  11040 Aug 29 01:22 o2cb
-r-xr-xr-x  1 root  wheel  11696 Aug 29 01:22 ping
-r-xr-xr-x  1 root  wheel   6356 Aug 29 01:22 pingd
-r-xr-xr-x  1 root  wheel   3702 Aug 29 01:22 remote
```

资源不多……我们将尝试将 **Dummy** 服务修改为在 FreeBSD 上的 IP 切换器。

```sh
root@node1:~ # cp /usr/local/lib/ocf/resource.d/pacemaker/Dummy /usr/local/lib/ocf/resource.d/pacemaker/ifconfig

root@node1:~ # vi /usr/local/lib/ocf/resource.d/pacemaker/ifconfig
               /* 输入量真大啊…… */
```

由于 *WordPress* 博客系统的限制，我不得不将这个 **ifconfig** 资源以图片形式发布……但别担心，文本版本也可在这里下载——[ifconfig.odt](https://vermaden.wordpress.com/wp-content/uploads/2020/09/ifconfig.odt)。

此外，第一个版本的效果并不理想……

```sh
root@node1:~ # setenv OCF_ROOT /usr/local/lib/ocf
root@node1:~ # ocf-tester -n resourcename /usr/local/lib/ocf/resource.d/pacemaker/ifconfig
Beginning tests for /usr/local/lib/ocf/resource.d/pacemaker/ifconfig...
* rc=3: Your agent has too restrictive permissions: should be 755
-:1: parser error : Start tag expected, '<' not found
usage: /usr/local/lib/ocf/resource.d/pacemaker/ifconfig {start|stop|monitor}
^
* rc=1: Your agent produces meta-data which does not conform to ra-api-1.dtd
* rc=3: Your agent does not support the meta-data action
* rc=3: Your agent does not support the validate-all action
* rc=0: Monitoring a stopped resource should return 7
* rc=0: The initial probe for a stopped resource should return 7 or 5 even if all binaries are missing
* Your agent does not support the notify action (optional)
* Your agent does not support the demote action (optional)
* Your agent does not support the promote action (optional)
* Your agent does not support master/slave (optional)
* rc=0: Monitoring a stopped resource should return 7
* rc=0: Monitoring a stopped resource should return 7
* rc=0: Monitoring a stopped resource should return 7
* Your agent does not support the reload action (optional)
Tests failed: /usr/local/lib/ocf/resource.d/pacemaker/ifconfig failed 9 tests
```

但在为其添加 755 权限并进行了若干（上百次）修改后，它终于可以使用了。

```sh
root@node1:~ # vi /usr/local/lib/ocf/resource.d/pacemaker/ifconfig
             /* LOTS OF NERVOUS TYPING */
root@node1:~ # chmod 755 /usr/local/lib/ocf/resource.d/pacemaker/ifconfig
root@node1:~ # setenv OCF_ROOT /usr/local/lib/ocf
root@node1:~ # ocf-tester -n resourcename /usr/local/lib/ocf/resource.d/pacemaker/ifconfig
Beginning tests for /usr/local/lib/ocf/resource.d/pacemaker/ifconfig...
* Your agent does not support the notify action (optional)
* Your agent does not support the demote action (optional)
* Your agent does not support the promote action (optional)
* Your agent does not support master/slave (optional)
* Your agent does not support the reload action (optional)
/usr/local/lib/ocf/resource.d/pacemaker/ifconfig passed all tests
```

看起来可以使用。

这是 **ifconfig** 资源。目前它相当有限，而且 IP 地址是硬编码的。

<img width="650" height="1636" alt="image" src="https://github.com/user-attachments/assets/8fd5bab9-9483-48d3-b0b1-efd31f2a6a66" />


让我们尝试向 FreeBSD 集群添加新的 **IP** 资源。

# 测试

```
root@node1:~ # crm configure primitive IP ocf:pacemaker:ifconfig op monitor interval="30"
```

已添加。

我们来看现在 **status** 命令显示的情况。

```sh
root@node1:~ # crm status
Cluster Summary:
  * Stack: corosync
  * Current DC: node2 (version 2.0.4-2deceaa3ae) - partition with quorum
  * Last updated: Wed Sep  2 22:44:52 2020
  * Last change:  Wed Sep  2 22:44:44 2020 by root via cibadmin on node1
  * 3 nodes configured
  * 1 resource instance configured

Node List:
  * Online: [ node1 node2 node3 ]

Full List of Resources:
  * IP  (ocf::pacemaker:ifconfig):       Started node1

Failed Resource Actions:
  * IP_monitor_0 on node3 'not installed' (5): call=24, status='Not installed', exitreason='', last-rc-change='2020-09-02 22:42:52Z', queued=0ms, exec=5ms
  * IP_monitor_0 on node2 'not installed' (5): call=24, status='Not installed', exitreason='', last-rc-change='2020-09-02 22:42:53Z', queued=0ms, exec=2ms
```

糟糕，我忘了将这个新的 **ifconfig** 资源复制到其他节点。现在来修复它。

```sh
root@node1:~ # rsync -av /usr/local/lib/ocf/resource.d/pacemaker/ node2:/usr/local/lib/ocf/resource.d/pacemaker/
Password for root@node2:
sending incremental file list
./
ifconfig

sent 3,798 bytes  received 38 bytes  1,534.40 bytes/sec
total size is 128,003  speedup is 33.37

root@node1:~ # rsync -av /usr/local/lib/ocf/resource.d/pacemaker/ node3:/usr/local/lib/ocf/resource.d/pacemaker/
Password for root@node3:
sending incremental file list
./
ifconfig

sent 3,798 bytes  received 38 bytes  1,534.40 bytes/sec
total size is 128,003  speedup is 33.37
```

现在我们先停止、删除，然后重新添加这个重要的资源。

```sh
root@node1:~ # crm resource stop IP
root@node1:~ # crm configure delete IP
root@node1:~ # crm configure primitive IP ocf:pacemaker:ifconfig op monitor interval="30"
```

祈祷顺利吧。

```sh
root@node1:~ # crm status
Cluster Summary:
  * Stack: corosync
  * Current DC: node2 (version 2.0.4-2deceaa3ae) - partition with quorum
  * Last updated: Wed Sep  2 22:45:46 2020
  * Last change:  Wed Sep  2 22:45:43 2020 by root via cibadmin on node1
  * 3 nodes configured
  * 1 resource instance configured

Node List:
  * Online: [ node1 node2 node3 ]

Full List of Resources:
  * IP  (ocf::pacemaker:ifconfig):       Started node1
```

看起来运行正常。

让我们验证它是否确实在应该的节点上启动。

```sh
root@node1:~ # ifconfig em0
em0: flags=8843 metric 0 mtu 1500
        options=81009b
        ether 08:00:27:2a:78:60
        inet 10.0.10.111 netmask 0xffffff00 broadcast 10.0.10.255
        inet 10.0.10.200 netmask 0xffffff00 broadcast 10.0.10.255
        media: Ethernet autoselect (1000baseT )
        status: active
        nd6 options=29

root@node2:~ # ifconfig em0
em0: flags=8843 metric 0 mtu 1500
        options=81009b
        ether 08:00:27:80:50:05
        inet 10.0.10.112 netmask 0xffffff00 broadcast 10.0.10.255
        media: Ethernet autoselect (1000baseT )
        status: active
        nd6 options=29

root@node3:~ # ifconfig em0
em0: flags=8843 metric 0 mtu 1500
        options=81009b
        ether 08:00:27:74:5e:b9
        inet 10.0.10.113 netmask 0xffffff00 broadcast 10.0.10.255
        media: Ethernet autoselect (1000baseT )
        status: active
        nd6 options=29
```

看起来确实在工作。

现在我们尝试将它移动到集群中的另一台节点。

```sh
root@node1:~ # crm resource move IP node3
INFO: Move constraint created for IP to node3

root@node1:~ # crm status
Cluster Summary:
  * Stack: corosync
  * Current DC: node2 (version 2.0.4-2deceaa3ae) - partition with quorum
  * Last updated: Wed Sep  2 22:47:31 2020
  * Last change:  Wed Sep  2 22:47:28 2020 by root via crm_resource on node1
  * 3 nodes configured
  * 1 resource instance configured

Node List:
  * Online: [ node1 node2 node3 ]

Full List of Resources:
  * IP  (ocf::pacemaker:ifconfig):       Started node3
```

已成功切换到 **node3** 系统。

```sh
root@node3:~ # ifconfig em0
em0: flags=8843 metric 0 mtu 1500
        options=81009b
        ether 08:00:27:74:5e:b9
        inet 10.0.10.113 netmask 0xffffff00 broadcast 10.0.10.255
        inet 10.0.10.200 netmask 0xffffff00 broadcast 10.0.10.255
        media: Ethernet autoselect (1000baseT )
        status: active
        nd6 options=29

root@node1:~ # ifconfig em0
em0: flags=8843 metric 0 mtu 1500
        options=81009b
        ether 08:00:27:2a:78:60
        inet 10.0.10.111 netmask 0xffffff00 broadcast 10.0.10.255
        media: Ethernet autoselect (1000baseT )
        status: active
        nd6 options=29
```

现在我们将 **关闭** **node3** 系统，以检查这个 **IP** 是否真的具备高可用性。

```sh
root@node2:~ # crm status
Cluster Summary:
  * Stack: corosync
  * Current DC: node2 (version 2.0.4-2deceaa3ae) - partition with quorum
  * Last updated: Wed Sep  2 22:49:57 2020
  * Last change:  Wed Sep  2 22:47:29 2020 by root via crm_resource on node1
  * 3 nodes configured
  * 1 resource instance configured

Node List:
  * Online: [ node1 node2 node3 ]

Full List of Resources:
  * IP  (ocf::pacemaker:ifconfig):       Started node3

root@node3:~ # poweroff

root@node2:~ # crm status
Cluster Summary:
  * Stack: corosync
  * Current DC: node2 (version 2.0.4-2deceaa3ae) - partition with quorum
  * Last updated: Wed Sep  2 22:50:16 2020
  * Last change:  Wed Sep  2 22:47:29 2020 by root via crm_resource on node1
  * 3 nodes configured
  * 1 resource instance configured

Node List:
  * Online: [ node1 node2 ]
  * OFFLINE: [ node3 ]

Full List of Resources:
  * IP  (ocf::pacemaker:ifconfig):       Started node1
```

看起来故障切换进行得很顺利。

**crm** 命令还会对其输出的不同部分进行高亮显示。

![failover](https://vermaden.wordpress.com/wp-content/uploads/2020/09/failover.jpg?w=960)

很高兴知道 Pacemaker 和 Corosync 集群在 FreeBSD 上运行良好。

虽然需要一些工作来编写必要的资源文件，但只要有时间和决心，完全可以将 FreeBSD 打造成一个非常强大的高可用集群。


