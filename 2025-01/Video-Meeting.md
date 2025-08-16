# 如何安装和配置 Galene 视频会议服务器

- 原文：[How To Install and Configure the Galene Video Meeting Server](https://freebsdfoundation.org/blog/how-to-install-and-configure-the-galene-video-meeting-server-on-freebsd/)
- 2025 年 7 月 7 日
- 作者：Benedict Reuschling

## 一点背景

来自 [其官网](https://galene.org/)：

> Galene（或 Galène）是一款视频会议服务器（“SFU”），易于部署，对服务器资源需求非常适中。它最初设计用于讲座、会议和学生辅导，但后来发现同样适用于传统会议。Galene 已在两所主要大学（巴黎大学和索邦大学）投入生产，用于讲座、实验课、研讨会和员工会议。

## 要求

- 较新的 FreeBSD 系统（本文写作时为版本 14.3）
- 可以不用 ZFS，但强烈建议使用

## FreeBSD 基础设置

首先，确保你已经安装了最新的 FreeBSD 系统更新。以 root 身份运行以下命令（由 `#` 表示）：

```sh
# freebsd-update fetch install
```

接着，将 pkg 仓库更新为 `latest` 分支。编辑 `/etc/pkg/Freebsd.conf`，把字符串 `quarterly` 改成 `latest`。然后运行以下命令更新 pkg 数据库：

```sh
# pkg update
```

## 安装 Galene

在从包安装 Galene 之前，我们可以在 `/var/db/galene` 下添加几个 ZFS 数据集来保存视频会议数据。这一步是可选的，Galene 在 UFS2 上同样能正常工作。这里使用 ZFS 是因为它提供了额外的特性，在生产环境中可能会很有用。

假设我们的池叫做 `videostar`。我们在 `/var/db` 下为 Galene 创建数据集。

### 为 Galene 创建 ZFS 数据集

```sh
# zfs create -p videostar/var/db/galene/data
# zfs create videostar/var/db/galene/groups
# zfs create videostar/var/db/galene/recordings
```

通过这些命令，我们在 `galene` 子目录下创建了一个独立数据集，并在其下创建了三个数据集：`recordings`、`data` 和 `groups`。我们稍后将填充这些内容。

### 包安装

FreeBSD 的软件包非常易用，并包含了预构建好的 Galene 包。我们接下来安装它：

```sh
# pkg install galene
```

安装过程中，`/var/db/galene` 下的目录会被创建。由于我们提前用 ZFS 数据集创建了它们，管理起来就更灵活了。例如，我们可以为 `recordings` 数据集设置配额。对于长时间的视频会议，录制文件会变得非常大。设置配额能防止 Galene 占满池中剩余的磁盘空间。

### Galene 配置文件

在首次启动 Galene 前，我们需要定义有哪些 group（群组）。这些群组就是视频会议房间，允许多个用户加入同一个房间，或在不同房间举行会议而互不干扰。同时，也需要定义用户的权限和密码。

一个在 `/var/db/galene/groups` 下的基础示例文件如下：

```ini
{
    "users":
    {
        "bob":
        {
            "password": "secret",
            "permissions": "op"
        }
    }
}
```

这里我们定义了一个名为 `bob` 的用户、一个密码以及房间中的操作员权限。房间本身叫做 `videostar`。

## 添加有效的 SSL 证书

虽然我们没有将其加入 [Ansible playbook](https://github.com/FreeBSDFoundation/blog/tree/main/how-to-install-and-configure-the-galene-video-meeting-server-on-freebsd)，但添加一份来自 `letsencrypt.org` 的有效 SSL 证书相对简单。简要步骤如下：

```sh
pkg install py311-certbot
certbot certonly -d YOURHOSTSFQDN --standalone
cp /usr/local/etc/letsencrypt/live/meet.fortasse.cloud/fullchain.pem /var/db/galene/data/cert.pem
cp /usr/local/etc/letsencrypt/live/meet.fortasse.cloud/privkey.pem /var/db/galene/data/key.pem
chown galene:galene /var/db/galene/data/*
service galene restart
```

## Galene 启动配置

Galene 包在安装二进制文件的同时也安装了启动脚本，位于 `/usr/local/etc/rc.d`。要让 Galene 随系统启动，在 `/etc/rc.conf` 中加入以下内容：

```sh
# service galene enable
```

之后启动服务：

```sh
# service galene start
```

检查服务是否在运行：

```sh
# service galene status
```

若显示 Galene 正在运行并附带进程 ID（PID），说明启动成功。如果没有，请重新检查上面的步骤，并查看 `/var/log/messages` 获取更多提示。

## 测试 Galene

我们可以在 `sockstat -l`（监听套接字列表）的输出中找到正在运行的 Galene 进程 PID：

```sh
# sockstat -l|grep galene
```

在同一行里，可以看到 Galene 默认监听的端口 8443。假设我们的主机名是 `videostar.example`，在浏览器地址栏中输入：

```sh
https://videostar.example:8443
```

此时会打开一个网页，询问你要加入哪个 group。输入 `videostar`（我们在配置文件中定义的那个），点击 `Join` 按钮。在下一页输入用户名和密码，选择允许使用的设备（摄像头、麦克风），然后点击 `Connect` 按钮。如果一切顺利，你就进入了拥有完整权限的视频会议房间。之后只需在 `videostar.json` 文件中添加更多用户并重启 galene 进程，就能把这个网址分享给他人。祝贺你，视频会议愉快！

![](https://freebsdfoundation.org/wp-content/uploads/2025/07/Galene-1024x790.webp "Galene")
