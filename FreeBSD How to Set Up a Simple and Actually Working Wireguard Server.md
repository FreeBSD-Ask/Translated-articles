# 如何在 FreeBSD 上设置一个简单且实际工作的 WireGuard 服务器

- 原链接：<https://herrbischoff.com/2023/04/freebsd-how-to-set-up-a-simple-and-actually-working-wireguard-server/>
- 发布日期：2023 年 4 月 17 日
- 译者：ykla & ChatGPT

在我努力使 WireGuard 正常运行的过程中，我碰到了许多指南和教程。但它们都缺少关键信息，或者依赖于过时的信息。通常只是微妙的指示出现了问题。当与其他潜在故障点一起出现时，这很快就会变成一个令人沮丧的打地鼠游戏，让人走入很多死胡同。

话虽如此，以下是一个最小化的设置，截止到今天，在运行 FreeBSD 13.2 的具有单个以太网连接的树莓派 3 上有效。

首先，安装所需的软件：

```
pkg install wireguard wireguard-tools libqrencode
```

开启 Wireguard 服务：

```
service wireguard enable
sysrc wireguard_interfaces="wg0"
```

启用 IP 转发并立即激活：

```
sysrc gateway_enable=YES
sysctl -w net.inet.ip.forwarding=1
```

设置防火墙和日志记录：

```
service pf enable
service pflog enable
```

```
# /etc/pf.conf

ext_if="ue0"
wg_net="10.10.10.0/24"
scrub in all
nat on $ext_if from $wg_net to any -> ($ext_if)
pass log all
```

```
service pf start
service pflog start
```

接下来生成服务器密钥：

```
cd /usr/local/etc/wireguard/
umask 077
wg genkey | tee /usr/local/etc/wireguard/server_private.key | wg pubkey | tee /usr/local/etc/wireguard/server_public.key
```

你还需要客户端密钥。这个示例假设使用 `iPhone` 作为名称，但对于任何客户端，流程都是相同的：

```
cd /usr/local/etc/wireguard/
umask 077
wg genkey | tee /usr/local/etc/wireguard/iphone_private.key | wg pubkey | tee /usr/local/etc/wireguard/iphone_public.key
wg genpsk > iphone_preshared.key
```

创建服务器配置：

```
https://herrbischoff.com/2023/04/freebsd-how-to-set-up-a-simple-and-actually-working-wireguard-server/#:~:text=%23%20/usr/local/etc/wireguard/wg0.conf%0A%0A%5BInterface%5D%0AAddress%20%3D%2010.10.10.1/32%20%23%20address%20the%20server%20will%20bind%20to%0AListenPort%20%3D%2051820%0APrivateKey%20%3D%20your%2Dprivate%2Dserver%2Dkey%2Dhere%0A%0A%5BPeer%5D%0AAllowedIPs%20%3D%2010.10.10.2/32%0APreSharedKey%20%3D%20your%2Dpreshared%2Dclient%2Dkey%0APublicKey%20%3D%20your%2Dpublic%2Dclient%2Dkey
```

创建客户端配置：

```
# /usr/local/etc/wireguard/wg_iphone.conf

[Interface]
PrivateKey = your-private-client-key
Address = 10.10.10.2/32
DNS = 9.9.9.9 # optional, useful to avoid DNS leaks

[Peer]
PublicKey = your-public-server-key
PreSharedKey = your-preshared-client-key
AllowedIPs = 0.0.0.0/0 # entire Internet
Endpoint = your-external-ip-or-host:51820
PersistentKeepalive = 30
```

对于额外的客户端，使用不同的文件名重复密钥生成。然后在服务器配置中添加另一个 `[Peer]` 部分，其中包含唯一的 IP 和客户端密钥内容。还要创建相应的客户端配置。在更改了服务器配置之后重新启动 WireGuard。

为了方便传输到 iPhone，在 WireGuard 应用程序中生成一个二维码，并通过摄像头导入：

```
qrencode -t ansi < wg_iphone.conf
```

开启 Wireguard 服务：

```
service wireguard start
```

---

现在，你应该能够成功地使用 iPhone 进行连接，并且能够访问你的内部网络以及互联网了。

几点注意事项：

- 在家中进行此操作时，请不要忘记在路由器中设置端口转发。WireGuard 专门使用 UDP。
- 可以使用本地 DNS 服务器，比如 Adguard Home。我就是这么做的。这样你就可以在任何地方实现全面的广告拦截。
- 如果遇到问题，首先重新启动机器。如果问题仍然存在，请使用 `tcpdump -n -e -i pflog0` 检查连接日志。
- 在几乎所有连接无法工作的情况下，要么你忘记设置 IP 转发，要么有其他干扰的防火墙规则，要么 NAT 规则错误。
- 我发现握手可靠性不稳定。你可能需要多次停止和启动客户端连接才能使其正常工作。
