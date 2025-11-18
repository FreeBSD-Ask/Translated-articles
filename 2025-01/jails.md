# 简单而轻松的 FreeBSD Jail

- 原文：[FreeBSD Jails are Simple and Easy](https://freebsdfoundation.org/blog/freebsd-jails-are-simple-and-easy/)
- 2025 年 8 月 25 日
- 作者：FreeBSD 基金会

在 FreeBSD 上运行常见服务很简单。但有时你想在同一个操作系统实例上运行多个服务，并且让每个服务都彼此安全地“隔离”。如果不提供互联网连接，web 服务就没多大意义，但这同时也可能带来了安全漏洞。很显然，我们不希望这样。如果某个服务有可能因为安全漏洞而被攻破，那么尽可能减少系统的暴露会更好，不是吗？

这就是 jail 的用武之地。Jail 是容器化的轻量化答案，早在 Docker 变得流行之前，[很久以前](https://en.wikipedia.org/wiki/FreeBSD_jail#History) 它就已存在。

## 设置 Jail

同科技行业中的所有事情一样，至少有一只手的方法能实现相同的结果。在 FreeBSD 上有许多工具可以用来创建和管理 jail，但我更倾向于 **保持简单**，所以选择使用默认安装系统中自带的工具。

手册中有 [关于 jail 的优秀文档](https://docs.freebsd.org/en/books/handbook/jails/)，我基本遵循它，仅做了一些细微的调整。

我从 [我们最近 YouTube 视频](https://youtu.be/CWuZLJkUBfw) 中创建的基础虚拟机开始，因为接下来的设置使用 ZFS。

这就是我设置 jail 的过程。如果你有五分钟，我们一起来看看。

## 配置宿主机

启用 jail：

```sh
sysrc jail_enable="YES" && sysrc jail_parallel_start="YES"
```

然后创建一些基本的 zfs 文件系统：

```sh
zfs create -o mountpoint=/jails zroot/jails
zfs create zroot/jails/media
zfs create zroot/jails/_base
```

手册中有其他挂载方式，你也可以选择使用。我更喜欢稍微简单一些的“base”和模板文件系统，然后通过快照来生成新的 jail。我们稍后会用到。

## 创建模板 jail

下载 FreeBSD 用户态文件用作我们的模板：

```sh
fetch https://download.freebsd.org/ftp/releases/$(uname -m)/$(freebsd-version)/base.txz -o /jails/media/$(freebsd-version)-base.txz
```

解压文件，修补模板，然后创建快照：

```sh
tar -xf /jails/media/$(freebsd-version)-base.txz -C /jails/_base/ --unlink
cp /etc/resolv.conf /jails/_base/etc/resolv.conf
cp /etc/localtime /jails/_base/etc/localtime
freebsd-update -b /jails/_base/ fetch install
pkg -c /jails/_base install -y pkg python zsh
zfs snapshot zroot/jails/_base@$(freebsd-version)-$(date +%d-%b-%y)
```

现在我们有了“模板” jail，可以用来创建新的 jail。我不会运行这个 jail，而且通常会在模板里添加常用包，这样每个后续克隆都会包含它们。把你喜欢的工具都放进去 🙂 值得注意的是：如果你打算用 Ansible 来管理 jail 中的包，你也需要在里面安装 pkg。之前为了保持简单，我没有这么做，而是直接在宿主机上用命令 `pkg -j JAILNAME install BLAH` 来安装。我想这算是个哲学问题——你是把 jail 看作是扩展的 chroot，还是微型的虚拟机？

差不多快完成了。我们需要 `/etc/jail.conf` ——我选择使用统一的“全局”文件，其中的选项适用于所有 jail，然后在 `/etc/jail.conf.d/` 中为每个 jail 单独创建配置文件：

`/etc/jail.conf` 模板：

```sh
# https://man.freebsd.org/jail.conf
# https://man.freebsd.org/jail
#
ip4 = inherit;
ip6 = inherit;

# 默认应用于所有 jail
exec.start = "/bin/sh /etc/rc";
exec.stop = "/bin/sh /etc/rc.shutdown jail";
exec.clean;

# 文件系统
mount.devfs;
devfs_ruleset=4;
enforce_statfs=1;
securelevel=2;
#allow.mount.zfs;
#allow.mount;

# 禁用所有花哨的东西
allow.mount.nodevfs;
allow.mount.nofdescfs;
allow.mount.nolinprocfs;
allow.mount.nonullfs;
allow.mount.noprocfs;
allow.mount.notmpfs;
allow.nochflags;

# 但这些总是有用的
allow.raw_sockets;
allow.reserved_ports;
allow.sysvipc=1;

# 杂项
allow.nomlock;
allow.noquotas;
allow.noread_msgbuf;
allow.noset_hostname;
allow.nosocket_af;
allow.nosysvipc;
sysvmsg=disable;
sysvsem=disable;
children.max=0;
.include "/etc/jail.conf.d/*.conf";
```

## 创建新的 jail

每当我需要新的 jail 时，只需 zfs 克隆模板，在 `/etc/jail.conf.d/` 中创建一个配置文件，并在 `/etc/rc.conf` 的 `jail_list` 中添加新 jail 名称。我写了个简单的 shell 脚本来实现这一点：

```sh
#!/bin/sh
#
set -eo pipefail

SNAP=${2:-$(zfs list -t snapshot -H -o name | grep "jails/_base" | cut -f3 -d/)}

if [ -z $1 ]; then
    echo "pass new jail name"
    exit 1
else
    new=$1
fi

zfs clone zroot/jails/${SNAP} zroot/jails/${new}

test -d /etc/jail.conf.d || mkdir -p /etc/jail.conf.d
cat > /etc/jail.conf.d/${new}.conf <<EOF
${new} {
    host.hostname = "${new}.jail";
    path = "/jails/${new}";
}
EOF

sysrc jail_list+=${new}
```

## 扩展 Jail

这只是有关关于如何快速启动 jail 的简单介绍。手册文档里有更多细节，包括如何 [管理资源限制](https://docs.freebsd.org/en/books/handbook/jails/#jail-resource-limits)（比如限制内存和 CPU）。如果你想进一步扩展，运行更多的 jail，你可能需要看看手册中列出的一些工具，或者参考 [社区的 Ansible 剧本](https://git.sr.ht/~dch/ansible-jails)。

正如我们博客文章中常做的那样，我们把本文用到的一些资源放在了 [我们的 GitHub 仓库](https://github.com/FreeBSDFoundation/blog/tree/main/easy-jail-setup)，并制作了视频：

<https://www.youtube.com/embed/vVpd34bWCJA>
