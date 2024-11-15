# 配置自己的 VPN——基于 OpenBSD、Wireguard、IPv6 和广告拦截

原文链接：<https://it-notes.dragas.net/2023/04/03/make-your-own-vpn-wireguard-ipv6-and-ad-blocking-included/>

VPN 是连接到自己的服务器和设备的基本工具。许多人出于各种原因使用商业 VPN，从不信任其提供商（尤其在连接到公共热点时）到希望使用不同 IP 地址（也许是其他国家的）在互联网上“活跃”。

无论出于何种原因，解决方案都不会匮乏。我一直在设置管理 VPN，以允许服务器、客户端使用安全通道进行通信。最近，我在所有设备上（包括桌面/服务器和移动设备）启用了 IPv6 连接，我需要快速创建一个节点，集中一些网络并允许它们在 IPv6 网络上出站。我使用并将要描述的工具是：

* VPS——在这种情况下，我使用了基本的 Hetzner Cloud VPS，但所有提供 IPv6 连接的提供商都可以：如果您需要 IPv6 的话。
* OpenBSD——一款干净、稳定且安全的操作系统。
* Wireguard——轻量级、安全，同时也不会“唠叨”，因此对移动设备电池也很友好。在没有流量时，它不会传输/接收任何内容。被所有主要的桌面和服务器操作系统以及 Android 和 iOS 设备广泛支持。
* Unbound——可以直接向根域名服务器发出 DNS 查询，而非转发器。还允许插入阻止列表，类似于 Pi-Hole 的结果（即广告拦截）。
* 列表——立即阻断与黑名单用户之间的连接。

第一步是激活 VPS 并安装 OpenBSD。在 Hetzner 云控制台上，没有有预置的 OpenBSD 镜像，只有 Linux 发行版可选。别怕，只需选择其中一个 Linux 并创建 VPS。创建完成后，OpenBSD ISO 镜像将出现在“ISO 映像”中。只需插入虚拟光驱，重启 VPS，OpenBSD 安装程序就会出现在控制台中。

我就不细说了，操作简单明了。唯一的注意事项（对于 Hetzner Cloud VPS）是使用“自动配置”进行 IPv4 设置，但暂时别配置 IPv6。稍后再配置。

使用 `syspatch` 命令安装所有 OpenBSD 更新并重新启动，内核将被重新链接。

在 OpenBSD 上，Wireguard 完全集成到了基本系统中，无需安装外部软件包。这是一个重大优势，因为随着时间的推移，所有与 Wireguard 相关的支持将由主要的 OpenBSD 开发团队直接管理。

第一步是在 VPS 上配置 IPv6。在 Hetzner 下，不幸的是，他们只提供 /64 的地址，因此需要对分配的网络进行分段。在这个例子中，它将被划分为 /72 子网络 - 可以使用[计算器](https://subnettingpractice.com/ipv6-subnet-calculator.html)来查找有效的子类。

文件 `/etc/hostname.vio0` 应该看起来如下：

```fallback
inet autoconf
inet6 2a01:4f8:cafe:cafe::1 72 
!route add -net ::/0 fe80::1%vio0
```

简而言之，保留 Hetzner 分配的基地址，但将子网掩码改为/72 - 从而可以使用其他网络。

```sh
sh /etc/netstart vio0
```

它将重新配置网络接口并允许 IPv6 开始工作。要测试它：

```sh
ping6 google.com
```

若一切配置正确，可执行 ping 命令且 google.com 将回复。

现在需要为 IPv4 和 IPv6 启用转发。在文件 `/etc/sysctl.conf` 中输入以下内容：

```fallback
net.inet.ip.forwarding=1
net.inet6.ip6.forwarding=1
```

要应用这些更改，您可以重新启动或直接输入：

```fallback
sysctl net.inet.ip.forwarding=1
sysctl net.inet6.ip6.forwarding=1
```

配置 Wireguard，需要进行一些步骤。首先，需要创建私钥：

```sh
openssl rand -base64 32
```

会出现类似以下内容：

> `YUkS6cNTyPbXmtVf/23ppVW3gX2hZIBzlHtXNFRp80w=`

现在创建一个名为 `/etc/hostname.wg0` 的新文件：

```fallback
172.14.0.1/24 wgport 51820 wgkey YUkS6cNTyPbXmtVf/23ppVW3gX2hZIBzlHtXNFRp80w=
inet6 2a01:4f8:cafe:cafe:100::1 72
```

正在创建一个新的 Wireguard 接口，名为 wg0。它将拥有 IPv4 地址“172.14.0.1”，Wireguard 将监听端口 51820，并使用稍后创建的私钥。它还将在供应商提供的子类别之一上拥有 IPv6 地址。

保存并激活该接口：

```sh
sh /etc/netstart wg0
```

如果所有内容都正确输入，它应该会启用接口。现在：

```sh
ifconfig wg0
```

将返回类似这样的内容：

```fallback
wg0: flags=80c3<UP,BROADCAST,RUNNING,NOARP,MULTICAST> mtu 1420
	index 5 priority 0 llprio 3
	wgport 51820
	wgpubkey xxxxxxxxxxxxxxx=
	groups: wg
	inet 172.14.0.1 netmask 0xffffff00 broadcast 172.14.0.255
	inet6 2a01:4f8:cafe:cafe:100::1 prefixlen 72
```

记录下 wgpubkey——它将用于配置客户端。

就防火墙而言，OpenBSD 带有基本的 pf 配置。在我的设置中，我倾向于阻断不需要的内容，并允许可能有用的内容。但是，我喜欢把“坏人”挡在门外，所以我使用黑名单。pf 允许在运行时向表中插入和移除元素，因此防火墙可以相应地进行配置。

要下载并应用 Spamhaus 列表，我使用在互联网上找到的一个简单而有效的脚本。

所以在/usr/local/sbin/spamhaus.sh 中创建脚本：

```sh
#!/bin/sh
#
# this is normally run once per day via /etc/daily.local.
#
echo updating Spamhaus DROP lists:
(
  { ftp -o - https://www.spamhaus.org/drop/drop.txt && \
    ftp -o - https://www.spamhaus.org/drop/dropv6.txt ; \
  } 2>/dev/null | sed "s/;/#/" > /var/db/drop.txt
)
pfctl -t spamhaus -T replace -f /var/db/drop.txt
```

使其可执行并运行：

```bash
chmod a+rx /usr/local/sbin/spamhaus.sh
/usr/local/sbin/spamhaus.sh
```

配置 pf 的方法很多。一个相当简单的例子可能是这样的：

这是一个非常简单的配置：它阻止了来自 Spamhaus 下载列表中的所有内容，允许 Wireguard 网络到公共接口的 NAT，允许 IPv6 中的 icmp 流量（这对网络正常运行至关重要），同时阻止了针对 Wireguard IPv6 LAN 的传入流量（请记住，IP 地址将是公共的且可直接到达，因此我们不希望默认情况下公开我们的设备）。允许 Wireguard 接口上的所有流量通过。然后将阻止所有流量并指定异常，即允许 ssh 和 Wireguard 连接（当然）。还将授权允许流量从公共网络接口流出。

重新加载 pf 配置:

```sh
pfctl -f /etc/pf.conf
```

倘若一切正常，防火墙应该已加载新的参数。

现在是配置 Unbound 来实现 DNS 查询缓存和相关广告拦截的时候了。一段时间前，我找到了一个脚本，稍作调整后使用。我记不清楚原始作者是谁了，所以我只在这里贴出来脚本。

创建一个脚本来更新 `/usr/local/sbin/unbound-adhosts.sh` 中的 unbound 广告拦截。

类似地，使脚本可执行并运行它：

```bash
chmod a+rx /usr/local/sbin/unbound-adhosts.sh
/usr/local/sbin/unbound-adhosts.sh
```

现在，可以修改位于 `/var/unbound/etc/unbound.conf` 的 Unbound 配置文件如下：

在启动 unbound 之前，必须给予适当的权限：

```bash
chown -R nobody:nobody /var/unbound
```

现在可以启用并启动 unbound。由于需要加载（长）阻止列表，这将需要几秒钟：

```bash
rcctl enable unbound
rcctl start unbound
```

如果一切都正确，unbound 将能够响应来自各自 LAN 的请求，位于 `172.14.0.1` 和` 2a01:4f8:cafe:cafe:100::1` 上。

现在可以配置 Wireguard 客户端了。每种实现都有其自己的过程（Android、iOS、MikroTik、Linux 等），但基本上只需在服务器和客户端上创建正确的配置即可。例如，在 OpenBSD 服务器上输入“ifconfig wg0”命令可查看服务器的公钥，应将其插入到客户端上将创建的“peer”配置中；而客户端的公钥则将在服务器上这样使用：

重新打开文件 `/etc/hostname.wg0` 添加：

```fallback
172.14.0.1/24 wgport 51820 wgkey YUkS6cNTyPbXmtVf/23ppVW3gX2hZIBzlHtXNFRp80w=
inet6 2a01:4f8:cafe:cafe:100::1 72
wgpeer *client's public key* wgaip 172.14.0.2/32 wgaip 2a01:4f8:cafe:cafe:100::2/128
```

重新加载配置：

```sh
sh /etc/netstart wg0
```

在客户端上，通过在本地 IP 地址中插入“172.14.0.2/32, 2a01:4f8:cafe:cafe:100::2/128”（在主机名为 wg0 的对等配置中输入的）创建一个新配置。将 DNS 服务器地址设置为“172.14.0.1”及/或其对应的 IPv6 地址（在本例中为 2a01:4f8:cafe:cafe:100::1 - 您的将会不同）。在对等端，插入服务器的数据，包括其公钥、IP 地址:port（在示例中，port是 51820），以及允许的地址（设置“0.0.0.0/0, ::0/0”意味着“所有连接将通过 Wireguard 发送” - 所有流量将通过 VPN 传输，无论是 IPv4 还是 IPv6）。

也可以将 VPN 仅用作广告拦截器，只通过其路由 DNS 流量。要实现此结果，只需配置客户端，使得唯一允许的地址是刚配置的 unbound 的地址（在本示例中，172.14.0.1 或 2a01:4f8:cafe:cafe:100::1） - DNS 解析将通过 VPN 进行，但浏览将继续通过主提供商运行。

如果您希望垃圾邮件和广告屏蔽列表能够自动更新，创建/etc/daily.local 文件并添加以下行：

```bash
#!/bin/ksh

/usr/local/sbin/unbound-adhosts.sh
/usr/local/sbin/spamhaus.sh
```

所有这些都可以通过简单安装 OpenBSD 来实现，无需安装任何额外的软件包。这既有助于更新管理，也有助于安全性。
