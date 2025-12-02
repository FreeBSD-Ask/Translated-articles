# FrankenPad T25 上的 FreeBSD 14.3

- [FreeBSD 14.3 on FrankenPad T25](https://vermaden.wordpress.com/2025/06/26/freebsd-14-3-on-frankenpad-t25/)
- 发布时间：2025/06/26
- 作者：𝚟𝚎𝚛𝚖𝚊𝚍𝚎𝚗

在我看过的许多电影中，我也很喜欢 *No Country for Old Men（老无所依）*（2007）——它与我对电脑/笔记本的偏好以及当时市场上可获得的设备产生了共鸣……以及现在可用的设备。我甚至还分享过我颇为消极的 [**笔记本悼文**](https://vermaden.wordpress.com/2022/02/07/epitaph-to-laptops/)。不久前，我还在使用 [**ThinkPad W520**](https://vermaden.wordpress.com/2022/04/14/freebsd-13-1-on-thinkpad-w520/)，那是 2011 年的笔记本——已有 14 年历史——但它运行 FreeBSD 完全没问题……不过当我因工作离开家时，一个新的机会出现了。

联想当时只生产了大约 5000 台 [**ThinkPad (T25) 25 周年纪念版**](https://news.lenovo.com/pressroom/press-releases/happy-25th-birthday-thinkpad/) 笔记本——所有机器配置相同——搭载了一颗“性能较弱”的 Intel 双核 CPU。我至今仍然遗憾，当时在我所在的波兰中部城市的本地商店里，一台全新的售价约 **$1100**，我没能买到，但过去无法改变。

![](https://vermaden.wordpress.com/wp-content/uploads/2025/06/thinkpad-t25-ansi-keyboard.jpg)

总体来说，**ThinkPad T25** 本质上是 **ThinkPad T470**，只是更换了若干部件——比如掌托和键盘。

在随意查看 EBAY 上 **ThinkPad T25** 的购买方案时，我找到了 **FrankenPad T25** ——价格大约 **$1200**，再加上税费和运送到波兰的费用。我购买了它，并顺利收到了，没有任何意外。经过一番测试后，它运行良好——一切正常——只是我花了大约半年时间才真正从 **ThinkPad W520** 迁移过去。

对于不了解 **FrankenPad** 的人，我来解释一下——它是由各种 ThinkPad 型号混合改装而成的——有时需要 3D 打印部件，有时需要修改 BIOS，有时甚至需要焊接等。过去，你可以自己制作，也可以订购——例如这里：[https://xyte.ch/mods/t25-frankenpad/](https://xyte.ch/mods/t25-frankenpad/)。

**FrankenPad** 最重要的部分当然是经典的 7 排键盘——其余的都只是附加组件。

![](https://vermaden.wordpress.com/wp-content/uploads/2025/06/thinkpad-25-keyboard-comparison-flat.jpg)

白色虚线显示了尺寸差异。

## 独一无二

每次听到或看到 *The One and Only（唯一）* 这个词，我脑海中都会浮现 **反恐精英** 1.x 的 [**HeatoN**](https://youtube.com/watch?v=KwAGylrHbyE) 玩家——看起来 **HeatoN**（本名 Emil Christensen）最近被收入了 **反恐精英** 的 [*Hall of Fame*](https://youtube.com/watch?v=AUztvlHCJAQ)——值得称赞！……我真的很喜欢二十年前我们玩的那些 **反恐精英** 1.x 局域网派对……可惜那些都已经结束了，每个人都太忙，再也无法像以前那样聚会……不过，当我的一个朋友创建了一个名为 *CS:GO* 的 WhatsApp 群组，在线聚会玩游戏时，多少恢复了一点乐趣……我们见面过一次，半年过去了。

回到 FrankenPad——世界上可能至少有几台这样的机器——我的具有以下特征：

– 它本质上是搭载四核 Intel CPU 和 32 GB 内存的 **ThinkPad T480** 笔记本。
– 它拥有 **ThinkPad (T25) 25 周年纪念版** 的掌托/触控板和经典键盘。
– 它还配备了来自 **ThinkPad T490** 的低功耗 FullHD（1920×1080）屏幕。

因此可以说，我的 **FrankenPad T25** 是 **ThinkPad T480/T25/T490** 的混合体。

从缺点来看——它仍然搭载 **Nvidia MX150**，我已在 BIOS 中将其关闭。

![](https://vermaden.wordpress.com/wp-content/uploads/2025/06/frankenpad-t25-freebsd-small.jpg)

你可以看到屏幕边框上仍有 **T480** 标识，而掌托和键盘来自 **T25** 型号。

…… 至于最不重要的消息——上面的截图展示了我大部分文章的写作方式 🙂。

我非常喜欢浏览器 *Epiphany* 的功能——每次保存文件时它都会刷新页面，所以我可以像有“实时”预览一样工作。

![](https://vermaden.wordpress.com/wp-content/uploads/2025/06/frankenpad-t25-top-cover-palm-rest.jpg)

另外——上盖仍然来自 **ThinkPad T480**（黑白色），而掌托来自 **ThinkPad T25**（彩色）。

## ThinkPad W520 的遗产

迁移后会怀念它吗？有些东西会。比如 **ThinkPad W520** 左侧有三个 USB-A 接口，而 **FrankenPad T25** 右侧只有两个 USB-A 接口（我需要使用 90 度角 USB 转接头和延长线，以免干扰鼠标板）。

**FrankenPad T25** 左侧还有两个 USB-C 接口……但一个被电源适配器占用，另一个在 FreeBSD 下无法直接使用，除非我接上一个廉价的 USB 集线器 🙂。

![](https://vermaden.wordpress.com/wp-content/uploads/2025/06/usb-c-hub.jpg)

我在这里买了一个小型 USB 集线器 – [https://aliexpress.com/item/1005007244139650.html](https://aliexpress.com/item/1005007244139650.html) – 在 *Aliexpress* 上。

键盘方面——**ThinkPad T25** 的键盘略微柔软/细腻——打字手感更柔和——很好用——并不是说我不喜欢 **ThinkPad W520** 的键盘，只是我更喜欢 **ThinkPad T25**。

其他优点是——我终于可以使用任何 65W Type-C 线来为它充电——包括我的 **ZMI QB826G 210W 25000mAh** 移动电源 – [Tiny UPS for Tiny NAS Reloaded](https://vermaden.wordpress.com/2024/02/11/tiny-ups-for-tiny-nas-reloaded/)——在这里有介绍。

![](https://vermaden.wordpress.com/wp-content/uploads/2024/02/zmi-top.jpg)

## 硬件

我拿到它后做的第一件事之一就是看看内部硬件。

![](https://vermaden.wordpress.com/wp-content/uploads/2025/06/frankenpad-t25-inside-bottom.jpg)

内部大致就是普通 **ThinkPad T480** 的配置。

……但在拆卸底盖时，我把背部的一些塑料部件弄坏了。

![](https://vermaden.wordpress.com/wp-content/uploads/2025/06/frankenpad-t25-inside-broken.jpg)

这是剩下的零件。这种情况在我过去拆装 **ThinkPad W520** / **X220** / **T520** / **T420s** 时从未发生过。

![](https://vermaden.wordpress.com/wp-content/uploads/2025/06/frankenpad-t25-inside-broken-parts.jpg)

我还订购了一条 SATA 数据线——大约 **$3**——为了能够连接我的 **Samsung 870 QVO 8TB SATA 2.5 SSD**……

![](https://vermaden.wordpress.com/wp-content/uploads/2025/06/thinkpad-t480-sata-cable.jpg)

但后来我以便宜的价格买到了 **Corsair MP600 PRO LPX 8TB M.2 NVMe SSD**，所以最终选择了 NVMe 固态硬盘。

## WiFi

我最初在 **FrankenPad T25** 上安装的是 **FreeBSD 14.2-RELEASE** ——但 WiFi 完全无法使用。

它原装安装的是 **Intel AX210** 网卡。

![](https://vermaden.wordpress.com/wp-content/uploads/2025/06/frankenpad-t25-inside-wifi-card.jpg)

感谢 **Michal Sapka** 在他的文章 [FreeBSD: Fixing ThinkPad X1 WiFi](https://michal.sapka.pl/2023/fixing-thinkpad-x1-wifi-on-freebsd/) 中提供的解决方案，我购买了 **Intel AC 9260** 网卡，并将其安装到我的 **FrankenPad** 笔记本中。

![](https://vermaden.wordpress.com/wp-content/uploads/2025/06/frankenpad-t25-inside-wifi-install.jpg)

上图显示了安装好的 **Intel AC 9260** WiFi 网卡。

现在 —— 无论是 **FreeBSD 14.2-RELEASE** 还是 **FreeBSD 14.3-RELEASE** 都能顺利连接 WiFi 网络。

## 指点杆

有些人喜欢 **ThinkPad** 笔记本，因为它们配备 **指点杆**，但我不是其中之一——至少可以说它非常慢且不够精准。

![](https://vermaden.wordpress.com/wp-content/uploads/2025/06/thinkpad-nub.png)

不过我对此并不反感——虽然它对我不适用，但我相信有很多人依赖它——所以就留给他们吧。

## FreeBSD 系统配置

我非常喜欢 FreeBSD UNIX 的一点是（更多内容见 – [Quare FreeBSD?](https://vermaden.wordpress.com/2020/09/07/quare-freebsd/)），它几乎可以仅通过三个文件完成完整配置。这套配置已经包含了我在 [The Power to Serve – FreeBSD Power Management](https://vermaden.wordpress.com/2018/11/28/the-power-to-serve-freebsd-power-management/) 文章中描述的所有电源管理设置。

像往常一样，我以比较标准的方式安装了 FreeBSD，启用了 GELI 加密，并使用 ZFS 作为文件系统。如果有疑问，安装过程可参考文章 [FreeBSD Desktop – Part 2.1 – Install FreeBSD 12](https://vermaden.wordpress.com/2018/11/20/freebsd-desktop-part-2-1-install-freebsd-12/)。

主要的 FreeBSD 配置文件如下：

* **/etc/rc.conf** – 系统服务配置
* **/etc/sysctl.conf** – 运行时参数
* **/boot/loader.conf** – 启动可配置参数

我还会包括以下文件，因为它们对配置也至关重要：

* **/etc/devfs.rules** – 设备配置
* **/etc/fstab** – 文件系统配置
* **/etc/ttys** – 终端初始化配置
* **/etc/wpa_supplicant.conf** – WiFi 配置
* **/usr/local/etc/automount.conf** – **automount(8)** 配置
* **/usr/local/etc/doas.conf** – **doas(1)** 配置
* 用户组成员信息。

首先是主要的 **/etc/rc.conf** 配置文件。

```ini
F25 % cat /etc/rc.conf
# 静默 # ------------------------------------------------------------------
  rc_startmsgs=NO
  rc_info=NO

# 网络 # ------------------------------------------------------------------
  hostname=f25.local
  background_dhclient=YES
  extra_netfs_types=NFS
  defaultroute_delay=3
  defaultroute_carrier_delay=3
  wlans_iwm0=wlan0                              # // network.sh
  create_args_wlan0="country PL regdomain FCC4" # // network.sh
# ifconfig_wlan0="WPA SYNCDHCP powersave"       # // network.sh
  gateway_enable="YES"
  harvest_mask=351
  rtsol_flags="-i"
  rtsold_flags="-a -i"

# 模块/通用/基本 # ------------------------------------------------------
  kld_list="${kld_list} /boot/modules/i915kms.ko"
  kld_list="${kld_list} fusefs coretemp sem cpuctl ichsmb cuse linux linux64"
  kld_list="${kld_list} urndis"

# 模块/BHYVE/VIRTUALBOX # -------------------------------------------------
  vboxnet_enable=NO
  vm_enable=YES
  vm_dir="zfs:zroot/vm"
  vm_list="poudriere"
  vm_delay="3"
  pf_enable=YES
  dnsmasq_enable=NO

# 电源 # --------------------------------------------------------------------
  performance_cx_lowest=C1
  economy_cx_lowest=Cmax
  powerd_enable=YES
  powerd_flags="-n adaptive -a hiadaptive -b adaptive -m 800 -M 1400"
  powerdxx_enable=NO
  powerdxx_flags="-n adaptive -a hiadaptive -b adaptive -m 800 -M 1400"

# 守护进程 | yes # ------------------------------------------------------------
  zfs_enable=YES
  xdm_enable=YES
  xdm_tty=ttyv4
  nfs_client_enable=YES
  devd_flags="-n"
  moused_enable=YES
  syslogd_flags='-s -s'
  sshd_enable=YES
  local_unbound_enable=NO
  webcamd_DEV=$( usbconfig | grep -i camera | awk -F ':' '{print $1}' )
  webcamd_enable=YES
  webcamd_0_flags="-d ${webcamd_DEV}"
  ubuntu_enable=YES
  rctl_enable=YES
  dbus_enable=YES
  cupsd_enable=YES

# 守护进程/temp | yes # -------------------------------------------------------
  samba_server_enable=NO
  nmbd_enable=NO
  smbd_enable=NO

# 守护进程 | no # -------------------------------------------------------------
  linux_enable=NO
  openssh_enable=NO
  openssh_flags='-4 -p 23'
  sendmail_enable=NO
  sendmail_submit_enable=YES
  sendmail_outbound_enable=NO
  sendmail_msp_queue_enable=NO

# 文件系统 # -----------------------------------------------------------------------
  fsck_y_enable=YES
  clear_tmp_enable=NO
  clear_tmp_X=YES
  growfs_enable=YES

# 其他 # --------------------------------------------------------------------
  keyrate=fast
  keymap=pl.kbd
  virecover_enable=NO
  update_motd=NO
  devfs_system_ruleset=desktop
  hostid_enable=YES
  entropy_file=NO
  savecore_enable=NO
  dumpdev=AUTO

# NFSD # ---------------------------------------------------------------------
  sshd_enable=YES
  nfs_server_enable=YES
  nfsv4_server_only=YES
  nfs_server_flags="-t"

# JAIL # ---------------------------------------------------------------------
  jail_enable=YES
  jail_devfs_enable=YES
  jail_list="minecraft"
```

接下来是运行时参数文件 **/etc/sysctl.conf** 。

```ini
F25 % cat /etc/sysctl.conf
# HARVEST MASK FOR random(4)
kern.random.harvest.mask=33119

# 安全
security.bsd.see_jail_proc=0
security.bsd.unprivileged_proc_debug=0

# 安全性/禁用 Intel CPU MDS 缓解措施
hw.mds_disable=0
machdep.mitigations.mds.disable=0

# 安全/随机 PID
kern.randompid=1

# 令人烦恼的事项
vfs.usermount=1
kern.coredump=0
hw.syscons.bell=0
kern.vt.enable_bell=0

# ZFS 磁盘块大小对齐为 4k
vfs.zfs.min_auto_ashift=12

# ZFS 删除错误导致的 TRIM 问题 (def:64)
vfs.zfs.vdev.trim_max_active=1

# ZFS ARC 调优
vfs.zfs.arc.min=134217728
vfs.zfs.arc.max=536870912

# 禁用 ZFS 严格的 ZVOL 配额执行 (def:1)
vfs.zfs.zvol_enforce_quotas=0

# Jail/允许在 Jail 中升级
security.jail.chflags_allowed=1

# JAILS/允许 RAW SOCKETS
security.jail.allow_raw_sockets=1

# JAILS/允许 fdescfs(5)
security.jail.mount_fdescfs_allowed=1
security.jail.param.allow.mount.fdescfs=1

# 桌面/交互性
kern.sched.preempt_thresh=224

# 桌面量子用于时间共享线程，以 stathz 时钟滴答为单位 (def:12) NomadBSD
kern.sched.slice=3

# 桌面/Iridium/Chromium
kern.ipc.shm_allow_removed=1

# 采样率转换器质量 (0=low .. 4=high) (def:1) NomadBSD
hw.snd.feeder_rate_quality=3

# 性能/所有共享内存段将映射到不可分页内存
kern.ipc.shm_use_phys=1

# VIRTUALBOX aio(4) 设置
vfs.aio.max_buf_aio=8192
vfs.aio.max_aio_queue_per_proc=65536
vfs.aio.max_aio_per_proc=8192
vfs.aio.max_aio_queue=65536

# 允许普通用户使用 idprio(8)
security.bsd.unprivileged_idprio=1

# 网络/不向已关闭端口的报文发送 RST
net.inet.tcp.blackhole=2

# 网络/对拒绝的连接不发送“端口不可达”响应
net.inet.udp.blackhole=1

# 网络/限制 SYN/ACK 重传次数（默认:3）

net.inet.tcp.syncache.rexmtlimit=0

# 网络/如果 syncache 溢出则使用 TCP SYN Cookies（默认:1）

net.inet.tcp.syncookies=0

# 网络/分配随机的 ip_id 值（默认:0）

net.inet.ip.random_id=1

# 网络/启用发送 IP 重定向（默认:1）

net.inet.ip.redirect=0

# 网络/忽略 ICMP 重定向（默认:0）

net.inet.icmp.drop_redirect=1

# 网络/丢弃 SYN+FIN 设置的 TCP 数据包（默认:0）

net.inet.tcp.drop_synfin=1

# 网络/更快回收 FIN_WAIT_2 状态的关闭连接（默认:0）

net.inet.tcp.fast_finwait2_recycle=1

# 网络/某些 ICMP 不可达消息可能会在 SYN_SENT 状态中中断连接（默认:1）

net.inet.tcp.icmp_may_rst=0

# 网络/初始发送/接收套接字缓冲区大小

net.inet.tcp.sendspace=65536
net.inet.tcp.recvspace=65536

# BHYVE

net.link.tap.up_on_open=1
net.link.tap.user_open=1

# 参考: [https://lists.freebsd.org/archives/freebsd-stable/2023-November/001726.html](https://lists.freebsd.org/archives/freebsd-stable/2023-November/001726.html)

vfs.zfs.dmu_offset_next_sync=0

# 声卡音量

hw.snd.vpc_0db=35

# 默认音频输出设备

hw.snd.default_unit=0

# 提高软件时钟精度（默认:5） 2025/02/05

kern.timecounter.alloweddeviation=0

# 禁用 ZFS Deadman —— 有助于 ZFS 在会进入睡眠的 USB 驱动器上运行

vfs.zfs.deadman.enabled=0


# 最大监听套接字待处理连接队列大小（默认: 128）

kern.ipc.soacceptqueue=1024

# 最大监听套接字待处理连接队列大小（兼容模式）（默认: 128）

kern.ipc.somaxconn=1024

# 在慢启动期间 TCP 最大 CWND 增量限制为该段数（默认: 2）

net.inet.tcp.abc_l_var=16

# 启用 TCP Fast Open 服务器功能（默认: 0）

net.inet.tcp.fastopen.server_enable=1

# FIN-WAIT2 超时（默认: 60000）

net.inet.tcp.finwait2_timeout=8000

# 防止共享内存被换出到磁盘

kern.ipc.shm_use_phys=1

# 在挂起时不切换虚拟控制台
# 有时切换到不同 VT 会破坏硬件加速
# 参考: [https://github.com/freebsd/drm-kmod/issues/175](https://github.com/freebsd/drm-kmod/issues/175)

kern.vt.suspendswitch=0
```

接下来是启动参数 **/boot/loader.conf** 文件。

```sh
F25 % cat /boot/loader.conf
# CONSOLE 通用
autoboot_delay=2       # USE '-1' FOR NO WAIT | USE 'NO' FOR INFINITE WAIT
hw.usb.no_boot_wait=1  # DO NOT WAIT FOR USB DEVICES FOR ROOT (/) FILESYSTEM
boot_mute=YES          # LIKE '-m' IN LOADER - MUTE CONSOLE WITH FreeBSD LOGO
loader_logo=none       # POSSIBLE LOGO OPTIONS: fbsdbw beastiebw beastie none
loader_menu_frame="none"
screen.font="6x12"

# CONSOLE 分辨率
kern.vt.fb.default.mode="1920x1080"
efi_max_resolution="1920x1080"
vbe_max_resolution="1920x1080"

# WINE 修复
machdep.max_ldt_segment=2048

# 启动模块
aesni_load=YES
geom_eli_load=YES
cryptodev_load=YES
zfs_load=YES

# 启用帧缓冲压缩以节能

compat.linuxkpi.i915_enable_fbc=1

# 启动时跳过不必要的模式设置

compat.linuxkpi.i915_fastboot=1

# 启用节能显示 C 状态

compat.linuxkpi.i915_enable_dc=2

# 尽可能禁用显示电源井

# compat.linuxkpi.i915_disable_power_well=1

# 启用 Synaptics 支持

hw.psm.synaptics_support=1

# 禁用 /dev/diskid/* 和 /dev/gptid/* 磁盘条目

kern.geom.label.disk_ident.enable=0
kern.geom.label.gptid.enable=0

# 增加 ZFS 事务超时时间以节省电池

vfs.zfs.txg.timeout=10

# RACCT/RCTL 资源限制

kern.racct.enable=1
# ZFS 调优

vfs.zfs.prefetch_disable=1

# 电源管理：关闭没有驱动程序的设备

hw.pci.do_power_nodriver=3

# 电源管理 / 为每个核心单独优化 ISS 时钟

# machdep.hwpstate_pkg_ctrl=0

# 电源管理：每个使用的 AHCI 通道（ahcich 0-7）

hint.ahcich.0.pm_level=5
hint.ahcich.1.pm_level=5
hint.ahcich.2.pm_level=5
hint.ahcich.3.pm_level=5
hint.ahcich.4.pm_level=5
hint.ahcich.5.pm_level=5
hint.ahcich.6.pm_level=5
hint.ahcich.7.pm_level=5

# GELI 线程数

kern.geom.eli.threads=4

# 最大发送队列大小

net.link.ifqmaxlen=2048

# 禁用 USB 数据包过滤

hw.usb.no_pf=1

# 启动和关闭时不等待 USB 设备枚举

hw.usb.no_boot_wait=0
hw.usb.no_shutdown_wait=1

# 禁用 hwpstate_intel(4) 驱动

hint.hwpstate_intel.0.disabled=1
```

如上所示——我已禁用了 **hwpstate_intel(4)** 驱动，因为在保持系统响应性的同时，我无法找到性能、功耗与电池续航的最佳平衡。

现在来看前面提到的 **/etc/devfs.rules** 文件。

```sh
F25 % cat /etc/devfs.rules
[desktop=10]
add path 'acd*'      mode 0660 group operator
add path 'cd*'       mode 0660 group operator
add path 'da*'       mode 0660 group operator
add path 'pass*'     mode 0660 group operator
add path 'xpt*'      mode 0660 group operator
add path 'fd*'       mode 0660 group operator
add path 'md*'       mode 0660 group operator
add path 'uscanner*' mode 0660 group operator
add path 'lpt*'      mode 0660 group cups
add path 'ulpt*'     mode 0660 group cups
add path 'unlpt*'    mode 0660 group cups
add path 'ugen*'     mode 0660 group operator
add path 'usb/*'     mode 0660 group operator
add path 'video*'    mode 0660 group operator
add path 'cuse*'     mode 0660 group operator
```

文件系统及 SWAP 配置。

```sh
F25 % cat /etc/fstab
# SWAP
  /dev/gpt/swap0  none  swap  sw  0 0

# /tmp @ RAM
  tmpfs  /tmp  tmpfs  rw,size=1g,mode=1777  0 0

# FreeBSD PSEUDO
  procfs  /proc  procfs  rw  0 0

# Ubuntu Linux PSEUDO
  linprocfs  /compat/ubuntu/proc     linprocfs  rw,failok,late                    0 0
  linsysfs   /compat/ubuntu/sys      linsysfs   rw,failok,late                    0 0
  devfs      /compat/ubuntu/dev      devfs      rw,failok,late                    0 0
  fdescfs    /compat/ubuntu/dev/fd   fdescfs    rw,failok,late,linrdlnk           0 0
  tmpfs      /compat/ubuntu/dev/shm  tmpfs      rw,failok,late,size=1g,mode=1777  0 0
  /home      /compat/ubuntu/home     nullfs     rw,failok,late                    0 0
  /tmp       /compat/ubuntu/tmp      nullfs     rw,failok,late                    0 0
```

终端配置位于 **/etc/ttys** 文件中。重要部分是 **ttyv4** 条目，它需要与 **/etc/rc.conf** 文件中的 **xdm_tty=ttyv4** 值相对应。

```sh
F25 % grep '^[^#]' /etc/ttys
console	none				unknown	off insecure
ttyv0	"/usr/libexec/getty Pc"		xterm	onifexists secure
ttyv1	"/usr/libexec/getty Pc"		xterm	onifexists secure
ttyv2	"/usr/libexec/getty Pc"		xterm	onifexists secure
ttyv3	"/usr/libexec/getty Pc"		xterm	onifexists secure
ttyu0	"/usr/libexec/getty 3wire"	vt100	onifconsole secure
ttyu1	"/usr/libexec/getty 3wire"	vt100	onifconsole secure
ttyu2	"/usr/libexec/getty 3wire"	vt100	onifconsole secure
ttyu3	"/usr/libexec/getty 3wire"	vt100	onifconsole secure
dcons	"/usr/libexec/getty std.115200"	vt100	off secure
xc0	"/usr/libexec/getty Pc"		xterm	onifconsole secure
rcons	"/usr/libexec/getty std.115200"	vt100	onifconsole secure
```

无线网络配置——作为不同网络类型的示例。如你所见，我没有在 **/etc/rc.conf** 文件中包含任何网络信息——这是因为我使用自己的 **network.sh** 方案来连接各种有线和无线网络——详细描述见 [FreeBSD Network Management with network.sh Script](https://vermaden.wordpress.com/2018/03/24/freebsd-network-management-with-network-sh-script/)。

```ini
F25 # cat /etc/wpa_supplicant.conf
# 通用
eapol_version=2
ap_scan=1
fast_reauth=1

# 开放网络
network={
  key_mgmt=NONE
  priority=0
}

# 隐藏 SSID 的 WIFI
network={
  scan_ssid=1
  ssid="hidden-network"
  psk="12341234"
  priority=0
}

# 命名开放网络
network={
  ssid="Free_Internet"
  key_mgmt=NONE
  priority=0
}

# 普通 WPA/WPA2 加密网络
network={
  ssid="SECURED"
  psk="12345678"
}
The automount(8) config.

F25 % cat /usr/local/etc/automount.conf
  USERUMOUNT=YES
  USER=vermaden
  FM='caja --no-desktop'
  NICENAMES=YES
```

**doas(1)** 配置文件。

```ini
F25 # cat /usr/local/etc/doas.conf
# CORE
  permit nopass keepenv root     as root
  permit nopass keepenv vermaden as root

# THE network.sh SCRIPT
  # pw groupmod network -m YOURUSERNAME
  # cat /usr/local/etc/doas.conf
  permit nopass :network as root cmd /etc/rc.d/netif args onerestart
  permit nopass :network as root cmd /usr/sbin/service args squid onerestart
  permit nopass :network as root cmd dhclient
  permit nopass :network as root cmd ifconfig
  permit nopass :network as root cmd killall args -9 dhclient
  permit nopass :network as root cmd killall args -9 ppp
  permit nopass :network as root cmd killall args -9 wpa_supplicant
  permit nopass :network as root cmd ppp
  permit nopass :network as root cmd route
  permit nopass :network as root cmd tee args -a /etc/resolv.conf
  permit nopass :network as root cmd tee args /etc/resolv.conf
  permit nopass :network as root cmd umount
  permit nopass :network as root cmd wpa_supplicant
```

我所属的用户组。

```sh
F25 % id vermaden | tr ' ' '\n' | tr ',' '\n'
uid=1000(vermaden)
gid=1000(vermaden)
groups=1000(vermaden)
0(wheel)
5(operator)
44(video)
47(realtime)
48(idletime)
69(network)
145(webcamd)
920(vboxusers)
```

我也不依赖“默认”的风扇转速，而是根据 CPU 温度使用 **[acpi-thinkpad-fan.sh](https://github.com/vermaden/scripts/blob/master/acpi-thinkpad-fan.sh)** 脚本自行设置风扇速度。

## 性能差异

为了测试 FrankenPad T25 的性能提升，我使用 **unixbench(1)** 测量 CPU 相关任务，并使用 **blogbench(1)** 测试磁盘读写性能。

图例说明如下：

* **ub1** – **unixbench(1)** 单线程（1 CPU 线程）得分
* **ub8** – **unixbench(1)** 多线程（8 CPU 线程）得分
* **bbR** – **blogbench(1)** 读取性能
* **bbW** – **blogbench(1)** 写入性能
* **diff** – **F25** 相对于 **W520** 提升的速度

测试结果：


|       | ub1  | ub8  | bbR  | bbW  |
|-------|------|------|------|------|
| **W520** | 510  | 1435 | 346k | 1443 |
| **F25**  | 1014 | 2447 | 548k | 2986 |
| **diff** | 2.0x | 1.7x | 2.0x | 1.6x |

如你所见，基于 **T480** 的 **FrankenPad T25** 在计算和 I/O 相关任务上大约比 **ThinkPad W520** 快 **2.0 倍**……老实说，这在日常工作中是明显感觉得到的。

## 电池续航

虽然 **FrankenPad T25**（或为了清晰起见称为改装的 **T480**）配备了两块电池，但它们都能支持约 3 到 4 小时的工作——如下所示，使用我创建的 FreeBSD 电池相关脚本测试。

使用的脚本：

* **[battery-time.sh](https://raw.githubusercontent.com/vermaden/scripts/refs/heads/master/battery-time.sh)**
* **[battery-capacity.sh](https://raw.githubusercontent.com/vermaden/scripts/refs/heads/master/battery-capacity.sh)**
* [**battery-info.sh**](https://raw.githubusercontent.com/vermaden/scripts/refs/heads/master/battery-info.sh)

测试结果如下。

```sh
F25 % battery-time.sh
time: 3:30
bat0: 92%
bat1: 98%

F25 % battery-capacity.sh 0
Battery '0' model '01AV420' has efficiency: 96%

F25 % battery-capacity.sh 1
Battery '1' model '01AV425' has efficiency: 92%

F25 % battery-info.sh 0
         Design capacity: 23940 mWh
      Last full capacity: 23070 mWh
              Technology: secondary (rechargeable)
Battery Swappable Capability: Non-swappable
          Design voltage: 11400 mV
         Capacity (warn): 1153 mWh
          Capacity (low): 200 mWh
             Cycle Count: 37
    Measurement Accuracy: 95%
    Max Average Interval: 1000 ms
    Min Average Interval: 500 ms
    Low/warn granularity: -1 mWh
   Warn/full granularity: -1 mWh
            Model number: 01AV420
           Serial number: 1020
                    Type: LiP
                OEM info: LGC
                   State: high
      Remaining capacity: 100%
          Remaining time: unknown
            Present rate: 0 mW
         Present voltage: 12713 mV

F25 % battery-info.sh 1
         Design capacity: 23990 mWh
      Last full capacity: 22220 mWh
              Technology: secondary (rechargeable)
          Design voltage: 10800 mV
         Capacity (warn): 1111 mWh
          Capacity (low): 200 mWh
             Cycle Count: 10
    Measurement Accuracy: 95%
    Max Average Interval: 1000 ms
    Min Average Interval: 500 ms
    Low/warn granularity: -1 mWh
   Warn/full granularity: -1 mWh
            Model number: 01AV425
           Serial number: 9001
                    Type: LION
                OEM info: SANYO
                   State: high
      Remaining capacity: 98%
          Remaining time: unknown
            Present rate: 0 mW
         Present voltage: 12676 mV
```

就像最初从 *Chernobyl* 核反应堆测得的每小时 3.6 兰氏剂量一样——不算好，也不算糟。

## 桌面环境

### Openbox

至于我使用的“桌面环境”，是我自定义的 **Openbox** 设置，配合 **Tint2** 和 **Dzen2** 等工具——用于最基础的环境。下图截图来自 FreeBSD 11.1，但今天看起来完全一样。

![](https://vermaden.wordpress.com/wp-content/uploads/2019/04/freebsd-desktop-2019-04.jpg?w=960)

我在整个 [FreeBSD Desktop](https://vermaden.wordpress.com/freebsd-desktop/) 系列文章中详细描述了该设置。

### XFCE

我也尝试过 XFCE——我特别喜欢配合 Global Menu appmenu 插件的使用方式。你可以参考这篇 [XFCE Cupertino Way](https://vermaden.wordpress.com/2022/02/09/xfce-cupertino-way/) 的实用指南。

![](https://vermaden.wordpress.com/wp-content/uploads/2022/02/xfce-ghostbsd.jpg?w=960)

## 配件

对于 **ThinkPad W520** 笔记本，有一些非常实用的配件。我将在下文中说明它们。

### 更小的电源适配器

**ThinkPad W520** 需要大块电源砖——官方 **ThinkPad 170W 电源** 或 **ThinkPad 135W 电源**（最初随 **ThinkPad W510** 销售）。而对于 **ThinkPad T480** 或我的 **FrankenPad T25**，使用文章 [More Undervalued Hardware Companions](https://vermaden.wordpress.com/2025/06/21/more-undervalued-hardware-companions/) 中 **Small Powerful USB-C Chargers** 部分所述的 140W 电源更加方便。

![](https://vermaden.wordpress.com/wp-content/uploads/2025/06/more-hardware-smaller-power-adapters-a.jpg)

### 鼠标搭档

在参考了多款鼠标（详见文章 [UNIX Mouse Shootout](https://vermaden.wordpress.com/2021/11/09/unix-mouse-shootout/)）后，我最终选择了 **罗技 Triathlon M720** 鼠标。我将 **联想 USB 接收器** 插入侧边 USB 接口。使用 USB 接收器时可以使用该鼠标，你也可以通过蓝牙连接到其他电脑。这款鼠标有一个专用按钮，可在三台不同电脑间切换。可惜它们之间的复制粘贴功能无法使用 🙂。

![](https://vermaden.wordpress.com/wp-content/uploads/2021/11/mouse-m720.jpg)

## 总结

在使用 **ThinkPad W520** 这么长时间后，没有任何设备是完美的——但 **FrankenPad T25** 是我能迁移到的最“无痛”选择——老实说，使用几个月后，我终于再次有了家的感觉。唯一让我担心的事情是——如果 **F25** 出了问题怎么办？

在大部分时间使用 **ThinkPad W520** 时，我都有备用设备……而这次我没有任何“备份”。

为了对潜在问题保持一定准备，我仍然保留了 **ThinkPad T14 GEN1**，并且还将 **ThinkPad T520** 添加到我的“设备库”中……不过不是普通型号——而是搭载 4C/8T CPU 的型号。在我考虑 **ThinkPad W520** 最重要的部分时，发现是 4C/8T CPU 和 1920×1080 FullHD 屏幕……当然还有传奇的 7 排 ThinkPad 键盘——因此我又入手了一台 **ThinkPad T520** 作为备用笔记本，带 4C/8T CPU，价格约 **$70**——至少在波兰当时能买到。

我计划在下个月左右出售我这两台 **ThinkPad W520**……虽然充满了许多回忆……
