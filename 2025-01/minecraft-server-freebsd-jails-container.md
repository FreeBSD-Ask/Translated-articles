# FreeBSD Jail 容器中的 Minecraft 服务器

- 原文：[Minecraft Server in FreeBSD Jails Container](https://vermaden.wordpress.com/2025/04/05/minecraft-server-freebsd-jails-container/)
- 发布时间：2025/04/05
- 作者：𝚟𝚎𝚛𝚖𝚊𝚍𝚎𝚗

今天——应我儿子的要求——我们来讲讲如何在 FreeBSD Jail 容器中运行 Minecraft 服务器。

![](https://vermaden.wordpress.com/wp-content/uploads/2025/04/minecraft-logo.png)

这有点像 Linux 上的 Docker/Podman，但安全性更高。

## 准备工作

首先我们将在 **/jail** 路径下创建环境，并获取所需的 FreeBSD **基本系统** 版本。

今天我们将使用 FreeBSD 14.2-RELEASE 版本。

```sh
host # mkdir -p /jail/minecraft /jail/BASE

host # VER=$( freebsd-version | awk -F '-' '{print $1 "-" $2}' )

host # fetch -o /jail/BASE/${VER}-base.txz https://download.freebsd.org/releases/amd64/14.2-RELEASE/base.txz
```

接下来我们将创建专用的 Minecraft FreeBSD Jail，并为其填充独立的 **基本系统** 内容。

我们还会复制 **/var/run/dmesg.boot**，因为 Minecraft 服务器需要使用它。

```sh
host # tar -C /jail/minecraft --unlink -xvf /jail/BASE/14.2-RELEASE-base.txz

host # cp /var/run/dmesg.boot /jail/minecraft/var/run/
```

## 创建

现在我们将设置基础 FreeBSD Jail 配置。

这些配置将作为所有其他 Jail 的默认值，除非我们重新定义它们。

```ini
host # cat /etc/jail.conf
exec.start      = "/bin/sh /etc/rc";
exec.stop       = "/bin/sh /etc/rc.shutdown";
exec.consolelog = "/var/log/jail_console_${name}.log";
exec.clean;
mount.devfs;
```

现在配置我们的 Minecraft Jail。

我们将使用局域网接口 **em0** 和 IP 地址 **10.0.0.210**。

```ini
host # cat /etc/jail.conf.d/minecraft.conf
  minecraft {
  # 全局
    exec.start = "/bin/sh /etc/rc";
    exec.stop  = "/bin/sh /etc/rc.shutdown";
    exec.consolelog = "/var/log/jail_console_${name}.log";
    exec.clean;
    mount.devfs;
    host.hostname = ${name};
    path = /jail/${name};

  # 自定义
    ip4.addr = 10.0.0.210;
    interface = em0;
    allow.raw_sockets;
    allow.sysvipc;
    devfs_ruleset=210;
    allow.mount;
    enforce_statfs=1;
    allow.mount.devfs;
    allow.mount.procfs;
    allow.mount.fdescfs;
  }
```

下面你还会看到宿主机上的规则集 **/etc/devfs.rules**。

```sh
host # grep -A 4 minecraft /etc/devfs.rules
[minecraft=210]
add include $devfsrules_jail
add path 'fd*' unhide
```

现在我们可以启动我们的 Jail 了。

```sh
host # service jail onestart minecraft
Starting jails: minecraft.

host # jls
   JID  IP Address      Hostname                      Path
     1  10.0.0.210      minecraft                     /jail/minecraft
```

你也可以使用我提供的工具 [**jmore(8)**](https://github.com/vermaden/jmore)。

```sh
host # jmore
JAIL       JID  TYPE  VER     DIR              IFACE   IP(s)
----       ---  ----  ---     ---              -----   -----
classic    -    std   13.2-R  /jail/classic    em0     10.0.0.199
ctld-two   -    vnet  13.2-R  /jail/ctld-two   ${if}b  -
ctld       -    vnet  13.2-R  /jail/ctld       ${if}b  -
fbsdjail   -    std   13.1-R  /jail/fbsdjail   wlan0   10.0.0.43
iscsi      -    vnet  13.2-R  /jail/iscsi      ${if}b  -
minecraft  1    std   14.2-R  /jail/minecraft  em0     10.0.0.210
minio      -    std   14.0-R  /jail/minio      em0     10.0.0.133
nfsd       -    vnet  14.1-R  /jail/nfsd       ${if}b  -
other      -    std   14.1-R  /jail/other      em0     10.0.0.199
sambajail  -    vnet  14.1-R  /jail/sambajail  ${if}b  -
unfs3      -    vnet  14.1-R  /jail/unfs3      ${if}b  -
```

为了让 Minecraft Jail 在启动时自动运行，需要在 **host** 系统的 **/etc/rc.conf** 文件中添加如下内容：

```ini
host # grep jail /etc/rc.conf
jail_enable=YES
jail_devfs_enable=YES
jail_list="minecraft"
```

## 配置 FreeBSD Jail

现在我们进入 Minecraft Jail。

使用 **jmore minecraft c** 相当于执行了知名的命令：

```sh
env PS1='minecraft # ' jexec minecraft /bin/sh
```

示例：

```sh
host # jmore minecraft c
minecraft #
```

接下来进行一些基本配置，例如设置 DNS 或将 **pkg(8)** FreeBSD 包管理器切换到 **latest** 分支：

```sh
minecraft # echo nameserver 1.1.1.1 > /etc/resolv.conf
minecraft # mkdir -p /usr/local/etc/pkg/repos
minecraft # sed -e 's|quarterly|latest|g' /etc/pkg/FreeBSD.conf > /usr/local/etc/pkg/repos/FreeBSD.conf
minecraft # pkg search -o minecraft
games/minecraft-client         Client for the block building game
```

现在安装其他所需的包，因为 Minecraft 服务器需要通过 FreeBSD Ports 构建：

```sh
minecraft # pkg install gitup bsddialog ccache portconfig openjdk21 tmux jless
```

由于需要通过 FreeBSD Ports 构建 Minecraft 服务器，并且许可需要手动接受（或忽略）（**译注：并不需要**），因此接下来使用 **gitup** 工具获取 FreeBSD Ports 树。

在 **make config** 阶段选择选项 **DAEMON** 。

```sh
minecraft # gitup ports
(...)
#
# Please review the following file(s) for important changes.
#       /usr/ports/UPDATING
#       /usr/ports/mail/dspam/files/UPDATING
#
# Done.

minecraft # cd /usr/ports/games/minecraft-server

minecraft # make config

 +------------|minecraft-server-1.21.4|--------------+
 | 'F1' for Ports Collection help.                   |  
 | +---------- RUN [select at least one] ----------+ |
 | | new (*) DAEMON     Run as a service           | |
 | | new ( ) STANDALONE Run the .jar file directly | |
 | +-----------------------------------------------+ |
 |                [  OK  ]     [Cancel]              |
 +---------------------------------------------------+
```

## 构建

接下来我们将构建 Minecraft 服务器。

```sh
minecraft # echo DISABLE_LICENSES=yes >> /etc/make.conf

minecraft # env BATCH=yes make build install clean
(...)
When you first run minecraft-server, it will populate the file
/usr/local/etc/minecraft-server/eula.txt

It is required to read the EULA, and then set eula=true

- Configuration files can be found in /usr/local/etc/minecraft-server/
- Log and debug output files can be found in /var/log/minecraft-server/
- World files can be found in /var/db/minecraft-server/

Without daemon option:
- To run the server, run /usr/local/bin/minecraft-server
- To edit java's parameters, edit /usr/local/etc/minecraft-server/java-args.txt
- To run with a specific version of Java, set environment variable JAVA_VERSION,
  for example:
    export JAVA_VERSION=22
    /usr/local/bin/minecraft-server
  or:
    JAVA_VERSION=22 /usr/local/bin/minecraft-server

With daemon option:
- The service has been installed with the name 'minecraft'
- To adjust maximum memory usage (-Xmx), use minecraft_memx= in /etc/rc.conf
- To adjust initial memory usage (-Xms), use minecraft_mems= in /etc/rc.conf
- To add other java parameters, use minecraft_args= in /etc/rc.conf
- To run with a specific version of Java, use minecraft_java_version= in /etc/rc.conf
- To see the interactive console, type service minecraft console

===>  Cleaning for minecraft-server-1.21.4
```

## Minecraft 服务器配置

按照 **pkg-message** 中的建议，我们将在 **/etc/fstab** 文件中添加额外的虚拟文件系统。

同时，我们还要确保这些文件系统在 Jail 启动时通过 **/etc/rc.local** 文件被挂载。

```sh
minecraft # cat << FSTAB >> /etc/fstab
        fdesc   /dev/fd         fdescfs         rw      0       0
        proc    /proc           procfs          rw      0       0
FSTAB

minecraft # echo 'mount -a' >> /etc/rc.local

minecraft # mount -a

minecraft # mount
zroot/jail on / (zfs, local, noatime, nfsv4acls)
devfs on /dev (devfs)
fdescfs on /dev/fd (fdescfs)
procfs on /proc (procfs, local)
devfs on /dev (devfs)
```

我们不会修改 Minecraft Jail 的主 **/etc/rc.conf** 配置文件来添加所需的 Minecraft 服务器选项。

同时，我们将“接受”EULA，并在 **/usr/local/etc/minecraft-server/server.properties** 文件中创建基本的 Minecraft 服务器配置。

你也可以在 **/usr/local/etc/minecraft-server/java-args.txt** 文件中配置额外的 Java 参数。如果默认值对你的情况太小，请自行增大。

```sh
minecraft # cat << RC >> /etc/rc.conf
minecraft_enable=YES
minecraft_mems=1024M
minecraft_memx=1024M
RC

minecraft # echo eula=true > /usr/local/etc/minecraft-server/eula.txt

minecraft # cat << MINECRAFT > /usr/local/etc/minecraft-server/server.properties
enable-jmx-monitoring=false
rcon.port=25575
level-seed=
gamemode=survival
enable-command-block=false
enable-query=false
generator-settings={}
enforce-secure-profile=true
level-name=world
motd=FreeBSD Minecraft Server
query.port=25565
pvp=true
generate-structures=true
max-chained-neighbor-updates=1000000
difficulty=easy
network-compression-threshold=256
max-tick-time=60000
require-resource-pack=false
use-native-transport=true
max-players=20
online-mode=false
enable-status=true
allow-flight=false
initial-disabled-packs=
broadcast-rcon-to-ops=true
view-distance=10
server-ip=
resource-pack-prompt=
allow-nether=true
server-port=25565
enable-rcon=false
sync-chunk-writes=true
resource-pack-id=
op-permission-level=4
prevent-proxy-connections=false
hide-online-players=false
resource-pack=
entity-broadcast-range-percentage=100
simulation-distance=10
rcon.password=
player-idle-timeout=0
force-gamemode=false
rate-limit=0
hardcore=false
white-list=false
broadcast-console-to-ops=true
spawn-npcs=true
spawn-animals=true
log-ips=true
function-permission-level=2
initial-enabled-packs=vanilla
level-type=minecraft\:normal
text-filtering-config=
spawn-monsters=true
enforce-whitelist=false
spawn-protection=16
resource-pack-sha1=
max-world-size=29999984
MINECRAFT
```

## 启动

现在是时候启动已安装并配置好的 Minecraft 服务器了。

```sh
minecraft # service minecraft start
Starting minecraft.

minecraft # service minecraft status
minecraft is running.

minecraft # sockstat -l4
USER     COMMAND    PID   FD  PROTO  LOCAL ADDRESS         FOREIGN ADDRESS      
mcserver java       33227 103 tcp4   10.0.0.210:25565      *:*
root     syslogd     7809 5   udp4   10.0.0.210:514        *:*
```

看起来服务器运行正常——但如果不正常，可以使用以下命令进行调试。

```sh
minecraft # su mcserver -c '/usr/local/bin/java -Xmx1024M -Xms1024M -jar /usr/local/minecraft-server/server.jar nogui'
```

## 连接 Minecraft 客户端

首先 – 确保你的客户端版本与 Minecraft 服务器版本一致 – 我这里是 **1.21.4**。

我的情况是，Minecraft 客户端是在某台随机的 Windows 电脑上启动的，如下所示。

![](https://vermaden.wordpress.com/wp-content/uploads/2025/04/minecraft-client-1.png)

点击 **多人游戏** 按钮。

![](https://vermaden.wordpress.com/wp-content/uploads/2025/04/minecraft-client-2.jpg)

然后点击 **添加服务器** 按钮 – 我们将添加基于 FreeBSD 的 Minecraft 服务器。

输入你喜欢的 Minecraft 服务器名称和 IP 地址。

![](https://vermaden.wordpress.com/wp-content/uploads/2025/04/minecraft-client-3.jpg)

稍等片刻，我们的基于 FreeBSD 的 Minecraft 服务器将出现在列表中。

![](https://vermaden.wordpress.com/wp-content/uploads/2025/04/minecraft-client-4.jpg)

现在点击 **加入服务器** 按钮加入服务器。

![](https://vermaden.wordpress.com/wp-content/uploads/2025/04/minecraft-client-5.jpg)

我们会看到 **正在连接服务器…** 的提示。

……片刻之后，我们就成功加入了 Minecraft 服务器。

![](https://vermaden.wordpress.com/wp-content/uploads/2025/04/minecraft-client-6.png)

## 用户

由于我不是 Minecraft 专家，我花了一些时间才找到在该服务器上定义“管理员”的方法。

幸运的是，我有一个可行的解决方案 🙂：

当我在这里写关于我自己的 **jmore(8)** 工具时 [New jmore(8) FreeBSD Jails List/Manage Tool](https://vermaden.wordpress.com/2024/11/22/new-jless-freebsd-jails-list-manage-tool/)，我最初把它叫作 **jless(8)**，但有人提醒我这个名字已经被一个处理 JSON 的工具占用了。我们稍后会用到它 🙂。

当有人连接到我们的服务器时，他的 **名字** 和 **uuid** 会被添加到 **/usr/local/etc/minecraft-server/usercache.json** 文件中，如下所示。

```json
host # cat /jail/minecraft/usr/local/etc/minecraft-server/usercache.json | tr ',' '\n'
[{"name":"antuan"
"uuid":"0d61326c-dfd1-3fa8-ba9d-249d402fb700"
"expiresOn":"2025-05-05 14:04:15 +0000"}
{"name":"antek"
"uuid":"4b520bac-4b31-3c41-8f9b-2781763e5c88"
"expiresOn":"2025-05-05 09:16:08 +0000"}]

host # jless /jail/minecraft/usr/local/etc/minecraft-server/usercache.json | cat
[
  {
    "name": "antuan",
    "uuid": "0d61326c-dfd1-3fa8-ba9d-249d402fb700",
    "expiresOn": "2025-05-05 14:04:15 +0000"
  },
  {
    "name": "antek",
    "uuid": "4b520bac-4b31-3c41-8f9b-2781763e5c88",
    "expiresOn": "2025-05-05 09:16:08 +0000"
  }
]
```

用 **bat(1)** 命令显示颜色后看起来效果更好。

![](https://vermaden.wordpress.com/wp-content/uploads/2025/04/minecraft-jless.png)

现在 – 要让这些用户成为“管理员”，我们需要将他们添加到 **/usr/local/etc/minecraft-server/ops.json** 文件中，并且重启 Minecraft 服务器才能生效。

```json
host # jless /jail/minecraft/usr/local/etc/minecraft-server/ops.json | cat
[
  {
    "uuid": "4b520bac-4b31-3c41-8f9b-2781763e5c88",
    "name": "antek",
    "level": 4,
    "bypassesPlayerLimit": false

  },
  {
    "uuid": "0d61326c-dfd1-3fa8-ba9d-249d402fb700",
    "name": "antuan",
    "level": 4,
    "bypassesPlayerLimit": false
  }
]
```


权限等级 **4** 是记录中最高的权限等级。

![](https://vermaden.wordpress.com/wp-content/uploads/2025/04/minecraft-jless-admins.png)

现在我儿子拥有了他需要使用的所有命令，例如 **/gamemode** 或 **/time** 🙂。

## 总结

欢迎随意分享你对个人 Minecraft 服务器其他所需配置的想法。

## 更新 1 – FreeBSD Jail 与 Linux Podman

我没想到第一句话会成为评论的焦点，因此在这里补充一些细节。我们来讨论一下 FreeBSD Jails 和 Linux Podman 容器在安全性上的差异。

**隔离性：** 对于无 root Podman 来说，如果启用了 SELinux/AppArmor，它的隔离性似乎与 Jail 在同一水平。但如果没有 SELinux/AppArmor，Jail 提供了更好的隔离性。当你在 Podman 中启用 SELinux/AppArmor 并再添加 MAC 框架（如 **mac_sebsd** / **mac_jail** / **mac_bsdextended** / **mac_portacl**）时，Jail 的隔离性更高。

**内核系统调用暴露面：** 即使是无 root Podman，除非通过 **seccomp**（SELinux）限制，否则仍有“完全”的系统调用访问权限。Jail 对系统调用的使用有限制，不需要额外工具就能实现；在 FreeBSD 上结合 MAC 框架还可以进一步缩小系统调用的可用范围。

**防火墙：** 你无法在无 root Podman 容器内运行防火墙。而在 VNET Jail 中，你可以运行完整的网络栈和任何防火墙（如 PF 或 IPFW），独立于宿主机运行，这意味着安全性更高。

**总结：** FreeBSD Jail 通常在默认情况下比 Podman 容器更安全，如果花时间添加额外的安全层，其安全性会更高。

市场存在时间也是一个重要因素。

Jail 自 1999/2000 年引入以来已经投入生产环境，已有 25 年历史，久经考验。Docker 从 2014 年开始流行，时间短约 10 年，但我们要比较的是 Jail 与 Podman。Podman 的无 root 支持首次出现在 2019 年晚期（1.6 版本），因此在市场上的时间不足 6 年。

这意味着 Jail 是所有方案中最经受考验的。
