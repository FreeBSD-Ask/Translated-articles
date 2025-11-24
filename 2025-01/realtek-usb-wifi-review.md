# Realtek RTL8188CUS – USB 802.11n 无线网卡评测

- 原文：[Realtek RTL8188CUS – USB 802.11n WiFi Review](https://vermaden.wordpress.com/2020/10/30/realtek-usb-wifi-review/)
- 作者：𝚟𝚎𝚛𝚖𝚊𝚍𝚎𝚗
- 发布时间：2020/11/18

在新款笔记本上使用 FreeBSD 时，有时你会发现自带的 WiFi 芯片不受支持……或者在 RELEASE 版本中尚未支持，而支持仅存在于你不想使用的开发版本 CURRENT 中。

这时，芯片 **Realtek RTL8188CUS** 就派上用场了。

![](https://vermaden.wordpress.com/wp-content/uploads/2020/10/realtek.png?w=960)

它被用于许多设备和产品，但我们关心的是它的小型 USB WiFi 版本，非常小巧。

Realtek 公司甚至在 2011 年因其 **802.11b/g/n 2.4GHz 1T1R WLAN 单芯片控制器（RTL8188CE/RTL8188CUS）** 获得了 [*台湾绿能经典奖 2011*](https://www.realtek.com/en/about-realtek/milestones-and-awards)，当时该芯片刚推出。

![](https://vermaden.wordpress.com/wp-content/uploads/2020/10/chip.jpg?w=960)

![](https://vermaden.wordpress.com/wp-content/uploads/2020/10/chip-look.jpg?w=960)

由于仅配备 1×1 天线并支持 802.11n，它的性能并不强——最高 150Mbps，仅单天线。

它体积也非常小，几乎不会从笔记本上突出来。

![](https://vermaden.wordpress.com/wp-content/uploads/2020/10/chip-space.jpg?w=960)

在连接时，它还会发出微弱的灯光。

![](https://vermaden.wordpress.com/wp-content/uploads/2020/10/chip-light.jpg?w=960)

## FreeBSD

下面展示它在 FreeBSD 上的使用情况。本文以 12.2-RELEASE 为例，但三年前在 11.1-RELEASE 上也是同样适用。

我的 *ThinkPad W520* 笔记本已经配备了 **Intel 6300** WiFi 卡，3×3 天线，802.11n 标准，由 **iwn(4)** 驱动。

```sh
# sysctl net.wlan.devices
net.wlan.devices: iwn0
```

现在我们将连接到 **Realtek RTL8188CUS** 芯片，并查看 **dmesg(8)** 命令的输出。

```sh
# dmesg
(...)
ugen2.3:  at usbus2
rtwn0 on uhub4
rtwn0:  on usbus2
rtwn0: MAC/BB RTL8188CUS, RF 6052 1T1R
```

……以及来自 **usbconfig(8)** 命令的更多信息。

```sh
# usbconfig
(...)
ugen2.3:  at usbus2, cfg=0 md=HOST spd=HIGH (480Mbps) pwr=ON (500mA)

# usbconfig -d 2.3 show_ifdrv
ugen2.3:  at usbus2, cfg=0 md=HOST spd=HIGH (480Mbps) pwr=ON (500mA)
ugen2.3.0: rtwn0:
```

它现在被列为 **rtwn0**，因为在 FreeBSD 上由 **rtwn(4)** 驱动。

```sh
# sysctl net.wlan.devices
net.wlan.devices: rtwn0 iwn0
```

现在让我们用这个 Realtek 芯片连接无线网络。我将创建 **wlan1** 设备，因为 **wlan0** 已经被另一张 Intel 6300 卡占用。

```sh
# ifconfig wlan1 create wlandev rtwn0

# ifconfig wlan1
wlan1: flags=8802<broadcast,simplex,multicast> metric 0 mtu 1500
        ether 00:1d:43:21:2d:1c
        groups: wlan
        ssid "" channel 1 (2412 MHz 11b)
        regdomain FCC country US authmode OPEN privacy OFF txpower 30 bmiss 7
        scanvalid 60 wme bintval 0
        parent interface: rtwn0
        media: IEEE 802.11 Wireless Ethernet autoselect (autoselect)
        status: no carrier
        nd6 options=21<performnud,auto_linklocal>

# wpa_passphrase WIFINETWORK PASSWORD >> /etc/wpa_supplicant.conf

# wpa_supplicant -i wlan1 -c /etc/wpa_supplicant.conf
Successfully initialized wpa_supplicant
wlan1: Trying to associate with d8:07:b8:b8:f4:81 (SSID='wireless' freq=2442 MHz)
wlan1: Associated with d8:07:b6:b8:f4:81
wlan1: WPA: Key negotiation completed with d8:07:b6:b8:f4:81 [PTK=CCMP GTK=CCMP]
wlan1: CTRL-EVENT-CONNECTED - Connection to d8:07:b6:b8:f4:81 completed [id=40 id_str=]
^Z // 在这里按 [CTRL]+[Z] 键
zsh: suspended  wpa_supplicant -i wlan1 -c /etc/wpa_supplicant.conf

# bg
[1]  + continued  wpa_supplicant -i wlan1 -c /etc/wpa_supplicant.conf

#

```

此时我们的网络二层（LAYER 2）应该已经连接，**wpa_supplicant(8)** 应在后台运行，**wlan1** 接口应显示 **associated** 状态。

```sh
# ps ax | grep wpa_supplicant
48693  4  S        0:00.43 wpa_supplicant -i wlan1 -c /etc/wpa_supplicant.conf
50687  4  S+       0:00.00 grep --color wpa_supplicant

# ifconfig wlan1
wlan1: flags=8843<up,broadcast,running,simplex,multicast> metric 0 mtu 1500
        ether 00:1d:43:21:2d:1c
        groups: wlan
        ssid wireless channel 7 (2442 MHz 11g ht/20) bssid d8:07:b6:b8:f4:81
        regdomain FCC country US authmode WPA2/802.11i privacy ON
        deftxkey UNDEF AES-CCM 2:128-bit txpower 30 bmiss 7 scanvalid 60
        protmode CTS ht20 ampdulimit 64k ampdudensity 4 shortgi -stbc -ldpc
        -uapsd wme roaming MANUAL
        parent interface: rtwn0
        media: IEEE 802.11 Wireless Ethernet MCS mode 11ng
        status: associated
        nd6 options=29<performnud,ifdisabled,auto_linklocal>
```

现在使用 **dhclient(8)** 命令为网络添加三层（LAYER 3）并获取 IP 地址。

```sh
# dhclient wlan1
DHCPDISCOVER on wlan1 to 255.255.255.255 port 67 interval 3
DHCPOFFER from 10.0.0.1
DHCPREQUEST on wlan1 to 255.255.255.255 port 67
DHCPACK from 10.0.0.1
bound to 10.0.0.9 -- renewal in 3600 seconds.
```

我们刚刚获得了 IP 地址 **10.0.0.9**。

最后一步是配置 DNS，然后使用命令 **ping(8)** 测试连接。

```sh
# echo nameserver 1.1.1.1 > /etc/resolv.conf

# ping -c 3 freebsd.org
PING freebsd.org (96.47.72.84): 56 data bytes
64 bytes from 96.47.72.84: icmp_seq=0 ttl=50 time=119.870 ms
64 bytes from 96.47.72.84: icmp_seq=1 ttl=50 time=119.371 ms
64 bytes from 96.47.72.84: icmp_seq=2 ttl=50 time=119.128 ms

--- freebsd.org ping statistics ---
3 packets transmitted, 3 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 119.128/119.456/119.870/0.309 ms
```

可用。

## FreeBSD 性能测试

接下来，我使用 **thunar(1)** 文件管理器，通过 NFS 大文件传输测试了这块简单的单天线 Realtek 芯片的性能。

![](https://vermaden.wordpress.com/wp-content/uploads/2020/10/not-great-not-terrible.jpg?w=960)

结果不算太差，但也算不上优秀。

从直接连接到 WiFi 路由器的局域网服务器复制文件到我的笔记本，速度约为 **2.9 MB/s**。我当时距离路由器 **5 米**。

```sh
服务器  ==局域网==>  路由器  ==WiFi==>  笔记本  @  2.9 MB/s
```

从笔记本通过 WiFi 向直接连接到 WiFi 路由器的局域网服务器复制文件，速度约为 **2.6 MB/s**。仍然距离路由器约 **5 米**。

```sh
笔记本  ==WiFi==>  路由器  ==局域网==>  服务器  @  2.6 MB/s
```

分别为 **23.2 Mbps** 和 **20.8 Mbps**。远低于单天线 802.11n 理论传输速度 150 Mbps……这很可能是 FreeBSD 无线栈的问题。

我认为这对于上网浏览已经足够，但通过 NFS 使用本地局域网资源可能会很痛苦。

相比之下，我的 Intel 6300 WiFi 卡在笔记本到路由器到服务器的复制速度为 **5.5 MB/s**，在服务器到路由器到笔记本的传输速度为 **10.5 MB/s**，分别为 **44 Mbps** 和 **84 Mbps**，而理论最大值为 450 Mbps。Intel 6300 和我的路由器均为 3×3 天线。

希望这些数字能接近 **30 MB/s**……

## 树莓派

**Realtek RTL8188CUS** 芯片的另一个优点是它在树莓派等小型设备上表现良好。我个人在 **树莓派 2B** 上测试过，效果非常出色。

![](https://vermaden.wordpress.com/wp-content/uploads/2020/10/rpi.jpg?w=960)

## 价格

这款芯片的价格也非常诱人。基于它的产品随处可见。在 *EBAY*、*全球速卖通* 上都有出售，许多情况下价格低至 **$2.50**。

有时运费甚至比产品本身还贵 :🙂:

尽情享用。

## 更新 1 – 中世纪状态

Reddit 用户 [Yaazkal](https://www.reddit.com/r/freebsd/comments/jl04tu/realtek_rtl8188cus_usb_80211n_wifi_review/gamsaog/) 提醒我，FreeBSD 上的 **rtwn(4)** 驱动仍不支持 802.11n 协议。

它仍停留在 802.11g 的“中世纪”传输水平。

## 更新 2 – 意外好处

虽然我很少使用或安装 Windows 系统，但我发现了 **Realtek RTL8188CUS** USB WiFi 适配器的新用途……例如在 Windows 10 全新安装后，原有 WiFi 卡可能“无法使用”，因为没有可用驱动。

这时，这个小巧的设备就非常有用——只需插入，运行 **Windows 更新** 获取所需的驱动和更新，无需查找设备 ID 或从 USB 驱动手动安装。

