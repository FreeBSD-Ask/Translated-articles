# 新型 ZFS 启动环境工具

- [New ZFS Boot Environments Tool](https://vermaden.wordpress.com/2018/08/24/new-zfs-boot-environments-tool/)
- 作者：𝚟𝚎𝚛𝚖𝚊𝚍𝚎𝚗
- 2018/08/24

大约一个月前，我有幸在 [**PBUG**](https://vermaden.wordpress.com/2018/07/30/zfs-boot-environments-at-pbug/) 上做了关于 **ZFS 启动环境** 的演讲。在演讲的最后，我提到了在 FreeBSD 上管理 **ZFS 启动环境** 的工具发展历史。

![zfs-boot-environments-history.png](https://vermaden.wordpress.com/wp-content/uploads/2018/08/zfs-boot-environments-history.png?w=960)

Pawel Jakub Dawidek——他也参加了 PBUG #3 会议——建议我应该尝试将 **beadm** 纳入 FreeBSD 基本系统。我也从很多 **beadm** 用户那里听到过这个想法，他们反复询问为什么 FreeBSD 基本系统没有内置 **beadm**。因此，在 PBUG #3 结束后，我正是做了这件事。我创建了新的 PR——[Bug 230323 – Idea/Feature Request – include beadm in the base](https://bugs.freebsd.org/bugzilla/show_bug.cgi?id=230323)——令我（惊喜地）发现，他们将新工具 **bectl** 内置到了 FreeBSD 基本系统！现在，我们的 **ZFS 启动环境** 工具家族中新增了一员——工具 **bectl**。

当然，我仍会维护和更新工具 **beadm**，它仍可通过 FreeBSD Ports 中的 [sysutils/beadm](https://freshports.org/sysutils/beadm) 获取。因为该工具是用 POSIX **/bin/sh** 编写的，所以调试快速且易于修改。简单来说（TLDR）：**bectl** 是用 C 语言实现的 **beadm**，它已引入到了 FreeBSD 基本系统，这意味着它将成为 FreeBSD 12.0-RELEASE 的一部分。目前 **bectl** 已经可以在 12.0-ALPHA2 镜像中使用。

## 对比

新的 **bectl** 工具还处于非常早期的阶段，目前（尚未）能完全替代工具 **beadm**。下面是 **bectl** 和 **beadm** 工具使用信息的简单对比。

```sh
root@fbsd12:~ # beadm
usage:
  beadm activate 
  beadm create [-e nonActiveBe | -e beName@snapshot] 
  beadm create 
  beadm destroy [-F] 
  beadm list [-a] [-s] [-D] [-H]
  beadm rename  
  beadm mount  [mountpoint]
  beadm { umount | unmount } [-f] 
  beadm version
```

……以及新的 **bectl** 工具。

```sh
root@fbsd12:~ # bectl
missing command
usage:  bectl ( -h | -? | subcommand [args...] )
        bectl activate [-t] beName
        bectl create [-e nonActiveBe | -e beName@snapshot] beName
        bectl create beName@snapshot
        bectl destroy [-F] beName | beName@snapshot⟩
        bectl export sourceBe
        bectl import targetBe
        bectl jail [ -o key=value | -u key ]... bootenv
        bectl list [-a] [-D] [-H] [-s]
        bectl mount beName [mountpoint]
        bectl rename origBeName newBeName
        bectl { ujail | unjail } ⟨jailID | jailName | bootenv)
        bectl { umount | unmount } [-f] beName
```

例如，**bectl** 目前无法重命名正在使用/已挂载的启动环境，而 **beadm** 可以。

```sh
root@fbsd12:~ # bectl rename safe new
boot environment is already mounted
failed to rename bootenv safe to new
```

可以通过命令 **zfs rename -u ...** 重命名挂载到 **/** 的 ZFS 数据集（这正是 **beadm** 在底层所做的操作），作为 **bectl** 工具的替代方案。

```sh
root@fbsd12:~ # bectl list
BE      Active Mountpoint Space Created
safe    NR     /          188K  2018-08-18 02:32
default -      -          427M  2018-08-18 02:26

root@fbsd12:~ # zfs list | grep safe
zroot/ROOT/safe      108K  6.85G   427M  /

root@fbsd12:~ # zfs rename -u zroot/ROOT/safe zroot/ROOT/new
```

随后它会像平常一样以新名字显示在 **bectl** 下，如下所示：

```sh
root@fbsd12:~ # bectl list
BE      Active Mountpoint Space Created
new     NR     /          188K  2018-08-18 02:32
default -      -          427M  2018-08-18 02:26
```

**bectl** 相较于 **beadm** 的一个很棒的新功能是可以在指定的启动环境中动态创建 FreeBSD Jail。

下面展示 **bectl** 创建 FreeBSD Jail 的实际操作。

```sh
root@fbsd12:~ # bectl list
BE      Active Mountpoint Space Created
new     NR     /          188K  2018-08-18 02:32
default -      -          427M  2018-08-18 02:26

root@fbsd12:~ # bectl jail default
# pwd
/
# ls /
.cshrc          bin             entropy         libexec         net             root            usr
.profile        boot            etc             media           proc            sbin            var
COPYRIGHT       dev             lib             mnt             rescue          tmp             zroot
# exit
root@fbsd12:~ # jls
   JID  IP Address      Hostname                      Path
     1                  default                       /tmp/be_mount.OnRc

root@fbsd12:~ # mount | grep default
zroot/ROOT/default on /tmp/be_mount.OnRc (zfs, local, noatime, nfsv4acls)

root@fbsd12:~ # bectl unjail default

root@fbsd12:~ # jls
   JID  IP Address      Hostname                      Path
```

如果你从 **beadm** 迁移到 **bectl**，你也需要更加小心，因为 **bectl** 不会让你进行确认 :🙂:

例如，**beadm** 工具会在销毁指定的启动环境前询问你是否确认。而 **bectl** 工具则会直接删除，不会在屏幕上出现任何提示。

```sh
root@fbsd12:~ # bectl list
BE      Active Mountpoint Space Created
new     NR     /          188K  2018-08-18 02:32
default -      -          427M  2018-08-18 02:26

root@fbsd12:~ # beadm destroy safe
Are you sure you want to destroy 'safe'?
This action cannot be undone (y/[n]): n

root@fbsd12:~ # bectl destroy safe

root@fbsd12:~ # bectl list
BE      Active Mountpoint Space Created
new     NR     /          188K  2018-08-18 02:32
```

**bectl** 缺少的功能之一是 *Ansible* 插件支持。**beadm** 支持 *Ansible* 插件，因此如果你希望使用该配置管理工具，那么使用 **bectl** 时就需要退回到 **原生** *Ansible* 模块 :🙂:

好消息是 **beadm** 和 **bectl** 可以在同一台主机上共存，所以你不必二选一。你仍然可以用 **beadm** 工具处理日常任务（或用于 *Ansible* 模块），而用 **bectl** 来处理 **jail**/**unjail** 等选项。

我认为随着时间推移，**bectl** 会增加所需功能，而将这样的工具纳入 FreeBSD 基本系统无疑是个受欢迎的补充。

## 更新 1

[New ZFS Boot Environments Tool](https://vermaden.wordpress.com/2018/08/24/new-zfs-boot-environments-tool/) 文章被收录在 [BSD Now 262 – OpenBSD Surfacing](https://www.jupiterbroadcasting.com/127006/openbsd-surfacing-bsd-now-262/) 集中。

感谢分享！

## 更新 2

最后，我有时间在更新的 FreeBSD-12.0-ALPHA6 版本中再次测试新的 **bectl** 命令，看是否有改进。

现在在无参数调用时，**bectl** 不会显示缺少命令的提示。

可以使用 **bectl** 命令重命名当前正在使用的启动环境。

我注意到的最后一点是，**bectl jail** 命令在退出后不会保持 Jail 处于启用/运行状态，虽然只是外观上的变化，但很重要。

最后，最简单的迁移路径是创建一个简单别名：

```sh
# alias beadm=bectl
```

… 或针对 (T)CSH shell：

```sh
# alias beadm bectl
```
