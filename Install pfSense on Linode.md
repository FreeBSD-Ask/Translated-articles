# 在 Linode 上安装 pfSense

 - 原文链接：<https://github.com/possnfiffer/bsd-pw/blob/gh-pages/docs/Install_pfSense_on_Linode.md>
 - 作者：Roller Angel
 - 译者：罗胜(superluosheng)
 - 校对整理：ykla 【】部分为 ykla 注解。

【Linode】一个 VPS 提供商

【pfSense】一个基于 FreeBSD 的开源防火墙/路由器

使用 <https://manager.linode.com/> 上的旧界面，在你喜欢的数据中心创建你的 Linode 。我不知道如何在新界面上进行操作。为了本教程的目的，我们建议关闭 Lassie，以防止看门狗在没有你的输入的情况下重新启动你的 Linode 。你可以在 Linode 管理器的设置选项卡中关闭看门狗下禁用 Lassie 。

创建两个磁盘镜像；他们应该都是 RAW 格式的。

```
第一个是 1024MB 大小的镜像，标记为安装程序。
第二个使用 Linode 的剩余空间。标记为 pfSense。
```
用以下设置创建两个配置文件（profile）。在每个配置文件中，你将需要禁用文件系统/引导助手下的所有选项。

Installer profile 配置文件：
```
Label: Installer
Kernel: Direct Disk
/dev/sda: pfSense disk image.
/dev/sdb: Installer disk image.
root / boot device: Standard /dev/sdb
```
Boot profile 配置文件：
```
Label: pfSense
Kernel: Direct Disk
/dev/sda: pfSense disk image.
root / boot device: Standard /dev/sda
```

在安装盘挂载到 `/dev/sda` 的情况下启动进入救援模式（Rescue Mode），并通过 SSH 从 Linode 管理器的远程访问选项卡中使用 Lish 访问你的Linode。

```
curl -O -k https://nyifiles.pfsense.org/mirror/downloads/pfSense-CE-memstick-2.4.4-RELEASE-amd64.img.gz
gunzip -c pfSense-CE-memstick-2.4.4-RELEASE-amd64.img.gz | dd of=/dev/sda bs=512k
```

当命令完成后，重新启动到你的安装程序配置文件 Installer profile。

转到 Linode 管理器中的远程访问标签。使用 Glish 访问你的 Linode，开始安装。注意，必须使用 Glish 来完成 pfSense 的安装。

## 安装 pfSense

---

安装（Install）-- 继续选择 ZFS -- da0 条带（Stripe of da0）--继续安装进行

在最后修改：

```
vi /boot/loader.conf
```

增加配置

```
boot_multicons="YES"
boot_serial="YES"
comconsole_speed="115200"
console="comconsole,vidconsole"
```

退出 Glish，返回到 Linode 管理器，重新启动到 pfSense 配置文件。

打开 lish -- 回答 n 到"VLAN 设置" -- vtnet0 为广域网（WAN）--输入其余部分，直到你看到启动完成。-- 关闭列表 -- 打开 Glish -- 从控制台输入 13 来更新，输入 Y 来更新
 --工作正在进行中。
