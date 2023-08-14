# 在 FreeBSD Jail 中使用 OpenRC 启动并运行 Devuan GNU+Linux 系统

- 原文地址：<https://weblog.antranigv.am/posts/2023/08/freebsd-jail-devuan-linux-openrc/>
- 作者：Antranig Vartanian
- 译者：basebit

两年前，我写了一篇名为 ["VoidLinux in FreeBSD Jail; with init"](https://weblog.antranigv.am/posts/2021/08/2021-08-21-00-37/) 的博文，在文中我们在一个 FreeBSD Jail 环境里安装并“启动”了 VoidLinux 。我认为现在是时候对那篇博文进行修订了。

这次，我们将使用 [Devuan GNU+Linux](https://www.devuan.org/) 以及用 [OpenRC](https://wiki.gentoo.org/wiki/Project:OpenRC) 来启动系统，并在 Linux Jail 中放置一些本地的 FreeBSD 二进制程序。

以下是我这次运行的操作系统版本：

```
root@srv0:~ # uname -v
FreeBSD 13.2-RELEASE releng/13.2-n254617-525ecfdad597 GENERIC
```

为了引导启动 Devuan 系统，我们需要使用一个叫 [debootstrap](https://wiki.debian.org/Debootstrap) 的软件，确切地说，是 [Devuan Chimaera 版本的 debootstrap 工具](https://pkginfo.devuan.org/cgi-bin/package-query.html?c=package&q=debootstrap=1.0.123+devuan3) 。我们可以从 ports/packages 中安装 `debootstrap`，然后再进行其余的修改。

```
pkg install -y debootstrap
```

现在我们需要获取 Devuan 的 `debootstrap` 工具，并将其解压缩，将一些文件放入我们的 `debootstrap` 中，并设置一些符号链接。

```
# 路径可能会随着时间变化，请访问 https://pkginfo.devuan.org/ 获取确切的链接。

fetch http://deb.devuan.org/merged/pool/DEVUAN/main/d/debootstrap/debootstrap_1.0.123+devuan3_all.deb

# .deb 文件很凌乱，建议创建一个目录。

mkdir debootstrap_devuan
mv debootstrap_1.0.123+devuan3_all.deb debootstrap_devuan/
cd debootstrap_devuan/
tar xf debootstrap_1.0.123+devuan3_all.deb
tar xf data.tar.gz

# 我们需要 chimaera（最新版本，符号链接）和 ceres（源始版本）。

cp usr/share/debootstrap/scripts/ceres usr/share/debootstrap/scripts/chimaera /usr/local/share/debootstrap/scripts/
```

现在我们可以引导启动我们的系统了。我将使用 ZFS 文件系统，但这也可以在不使用 ZFS 文件系统的情况下完成。

请记住，我的 Jail 路径将是 `/usr/local/jails/devuan0` ，请根据需要修改此路径。🙂

```
zfs create zroot/jails/devuan0
debootstrap --no-check-gpg --arch=amd64 chimaera /usr/local/jails/devuan0/ http://pkgmaster.devuan.org/merged/
```

现在安装过程应该开始了，但在某个阶段，我们会遇到以下错误：

```
I: Configuring libpam-runtime...
I: Configuring login...
I: Configuring util-linux...
I: Configuring mount...
I: Configuring sysvinit-core...
W: Failure while configuring required packages.
W: See /usr/local/jails/devuan0/debootstrap/debootstrap.log for details (possibly the package package is at fault)
```

不要慌张！这没事儿。🙂 我们只需要进入 chroot 环境，手动修复这个问题，并安装 OpenRC 程序。

```
chroot /usr/local/jails/devuan0 /bin/bash

# 修复基本包

dpkg --force-depends -i /var/cache/apt/archives/*.deb

# 设置 Cache-Start

echo "APT::Cache-Start 251658240;" > /etc/apt/apt.conf.d/00chroot

# 安装 OpenRC

apt update
apt install openrc
```

我们几乎已经准备好了所有东西。我们只需要创建一个 password 数据库文件，供 jail(8) 命令在内部使用。

```
cd /usr/local/jails/devuan0/etc/
echo "root::0:0::0:0:Charlie &:/root:/bin/bash" > master.passwd
pwd_mkdb -d ./ -p master.passwd

# 存储 Linux passwd 文件

cp passwd- passwd
```

我们还可以将静态链接的 FreeBSD 二进制程序移入 Linux Jail 中，这样我们在需要时就可以使用它们。

```
cp -a /rescue /usr/local/jails/devuan0/native
```

现在我们还需要一个 Jail 配置文件，我们可以将其放在 `/etc/jail.conf.d/devuan0.conf` 中。（假设你的网络配置类似于[“VNET Jail HowTo Part 2: Networking”](https://weblog.antranigv.am/posts/2021/04/2021-04-20-07-02/)）

```
# vim: set syntax=sh:

exec.clean;
allow.raw_sockets;
mount.devfs;

devuan0 {

# ID == epair index :)

  $id             = "0";
  $bridge         = "bridge0";

# Set a domain :)

  $domain         = "bsd.am";
  vnet;
  vnet.interface = "epair${id}b";

  mount.fstab     = "/etc/jail.conf.d/${name}.fstab";

  exec.prestart   = "ifconfig epair${id} create up";
  exec.prestart  += "ifconfig epair${id}a up descr vnet-${name}";
  exec.prestart  += "ifconfig ${bridge} addm epair${id}a up";

  exec.start      = "/sbin/openrc default";

  exec.stop       = "/sbin/openrc shutdown";

  exec.poststop   = "ifconfig ${bridge} deletem epair${id}a";
  exec.poststop  += "ifconfig epair${id}a destroy";

  host.hostname   = "${name}.${domain}";
  path            = "/usr/local/jails/devuan0";

# Maybe mkdir this path :)

  exec.consolelog = "/var/log/jail/${name}.log";

  persist;
  allow.socket_af;
}
```

正如你所猜到的，我们还需要一个 fstab 文件，应该放在 `/etc/jail.conf.d/devuan0.fstab` 中。

```
devfs       /usr/local/jails/devuan0/dev      devfs     rw                   0 0
tmpfs       /usr/local/jails/devuan0/dev/shm  tmpfs     rw,size=1g,mode=1777 0 0
fdescfs     /usr/local/jails/devuan0/dev/fd   fdescfs   rw,linrdlnk          0 0
linprocfs   /usr/local/jails/devuan0/proc     linprocfs rw                   0 0
linsysfs    /usr/local/jails/devuan0/sys      linsysfs  rw                   0 0
tmpfs       /usr/local/jails/devuan0/tmp      tmpfs     rw,mode=1777         0 0
```

最后，我们加载一些内核模块（以防它们尚未加载）。

```
service linux enable
service linux start
kldload netlink
```

让我们启动 Jail 吧！

```
jail -c -f /etc/jail.conf.d/devuan0.conf
```

它正在运行吗？

```
# jls -N

JID             IP Address      Hostname                      Path
devuan0                         devuan0.bsd.am                /usr/local/jails/devuan0
```

是的，它已经在运行了！

现在我们可以使用 `jexec` 进入其中并运行一些命令！

```
root@srv0:~ # jexec -l devuan0 /bin/bash
root@devuan0:~# uname -a
Linux devuan0.bsd.am 4.4.0 FreeBSD 13.2-RELEASE releng/13.2-n254617-525ecfdad597 GENERIC x86_64 GNU/Linux
```

进程树看起来也很整洁！

```
root@devuan0:~# ps f
  PID TTY      STAT   TIME COMMAND
74682 pts/1    S      0:00 /bin/bash
78212 pts/1    R+     0:00  \_ ps f
48412 ?        Ss     0:00 /usr/sbin/cron
41190 ?        Ss     0:00 /usr/sbin/rsyslogd
```

让我们进行一些网络设置吧！设置网络并安装 OpenSSH。（假设你的网络配置类似于[“VNET Jail HowTo Part 2: Networking”](https://weblog.antranigv.am/posts/2021/04/2021-04-20-07-02/)）

```
# 设置网卡

/native/ifconfig lo0 inet 127.0.0.1/8 up
/native/ifconfig epair0b inet 10.0.0.10/24 up
/native/route add default 10.0.0.1

# 安装并启动 OpenSSH 服务器

apt-get --no-install-recommends install openssh-server
rc-service ssh start
```

现在你应该能够 ping 通其他主机了。

```
~# ping -n -c 1 bsd.am
ping: WARNING: setsockopt(ICMP_FILTER): Protocol not available
PING  (37.252.73.34) 56(84) bytes of data.
64 bytes from 37.252.73.34: icmp_seq=1 ttl=55 time=2.60 ms

---  ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 2.603/2.603/2.603/0.000 ms
```

为了使网络配置持久化，我们可以使用 `rc.local` 文件，在 OpenRC 启动时该文件会被执行。

```
chmod +x /etc/rc.local
echo '/native/ifconfig lo0 inet 127.0.0.1/8 up' >> /etc/rc.local
echo '/native/ifconfig epair0b inet 10.0.0.10/24 up' >> /etc/rc.local
echo '/native/route add default 10.0.0.1' >> /etc/rc.local
```

你知道这意味着什么吗？这意味着你现在可以在 Linux 上使用正确的 ZFS、DTrace 和 pf 防火墙了。恭喜你，现在你拥有了一片净土。

就是这样了，朋友们……

PS：我想感谢我的导师 [norayr](http://norayr.am/) ，他向我展示了如何手动启动/停止 OpenRC，还要感谢 [#devuan](https://www.devuan.org/os/community) 社区里那些了不起的人们，感谢他们的帮助。
