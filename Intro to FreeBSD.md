# FreeBSD 简介| BSD.pw 研讨会

## 必要条件

- VirtualBox（5+ 是我们使用的版本，但任何版本都应该可以。（在 2020 年使用 6.1 版，仍然可以正常工作）此程序适用于大多数电脑，请确保在您的电脑 BIOS 中启用 VT-x）
- 有网络接入

## 研讨会

### 上午

首先，我们可以简单地说：

- 在工作电脑上访问 [FreeBSD.org](https://FreeBSD.org)
- 在小恶魔的 logo 旁边寻找 `Download FreeBSD` 这个大按钮。
- 进入下载页面后，点击 Installer 镜像下的 `amd64`。
- 接下来，在列表中点击 `FreeBSD-13.1-RELEASE-amd64-disc1.iso` 这个镜像文件 。
  - 因为它是最新版的 FreeBSD

我强烈建议你关注 `CHECKSUM.SHA512-FreeBSD-13.1-RELEASE-amd64` 这个文件，并在你下载完文件后自己进行 SHA512 校验，看它们是否匹配。 只有这样，你才能知道你下载的文件是否能正常工作 . 安装镜像文件下载完成后，就可以用虚拟机来启动了。

### 如果你不想或者不能使用 VirtualBox ，那么你可以用 [Vultr](https://www.vultr.com/?ref=9386224) 或者 [Digital Ocean](https://m.do.co/c/42ca15dcb41e)

在 VirtualBox 中，单击“新建”按钮启动向导，该向导将引导您创建虚拟机。需要注意的重要选项如下：

- 操作系统类型为 `FreeBSD 64 bit`,
- 以及用于虚拟机的内存大小。使用大于 512MB 的内存，小于这个数值会使 virtualbox 向导“滑块”变红。将您的“滑块”保持在绿色中，您就可以为虚拟机提供足够的内存。
- 接受其它的默认设置。

向导完成后，即可启动虚拟机。此时将出现另一个向导，要求您将 VirtualBox 指向新虚拟机的安装程序镜像。如果您想避免第二个向导，只需在首次启动前编辑虚拟机设置即可：

- 点击 `存储` 链接,
- 然后点击 `空白 Disc` ,
- 然后你就可以挂在这个虚拟镜像了.

在 VirtualBox 中，所有这些步骤都相当于组装硬件和插入安装光盘。在 VirtualBox 中按“开始”键就像按“电源”键启动一台真正的机器。第一次启动安装光盘时， FreeBSD 会显示欢迎您，并询问您是否要开始安装。

按键盘上的“回车”键选择 `Install` 。此时，您会注意到 VirtualBox 试图解释它正在捕获您的鼠标和键盘以便在虚拟机中使用。请忽略这些信息。要再次在虚拟机外使用鼠标和键盘，您需要按下主机键。VirtualBox 会在窗口右下方显示当前的主机键。该安装显示 "Right Ctrl" ，这意味着您需要先按键盘上的右控制键，以从虚拟机中重新获得对鼠标和键盘的控制。

#### 用户账号

为系统的 root 账户设置密码。当输入 root 密码时没有出现 `****` 时不要着急。请放心，您的键盘输入都被记录下来了，只是没有显示在屏幕上而已。

继续执行安装程序，注意以下事项：

在安装过程中，我们将继续使用默认的 US 键盘布局，将计算机命名为 `getting-started-with-freebsd`，并在 `Distribution Select` 屏幕上选择默认设置（如果有的话，这取决于你下载的.iso 版本），只需按下 `ok` 即可。 在 `Network Configuration` 部分，我们在虚拟机上使用 `em0` 接口，因为我们知道 VirtualBox 使用的是 intel 网络驱动程序，它有 `em0` 的名称。在某些机器上，你会发现 `re0` 接口是 Realtek 网络驱动程序。

是，配置 IPv4。是，使用 DHCP 。是，使用 IPv6 。是，尝试 SLAAC 。解析器配置中应至少填入一个 DNS 值。使用键盘上的 Tab 键移动到选择 "OK " 继续。对于 "镜像配置"，我们使用 "主站点"。接下来，我们需要决定如何对磁盘分区。在 "分区 "屏幕上，有 "自动（ZFS）"和 "自动（UFS）"两个选项。选择自动 (ZFS)，选择条纹作为 ZFS 池类型，用空格键选择用于池的磁盘，为池命名，然后继续安装。最后，选择 "Yes " 以同意安装 FreeBSD。

在 "时区 "部分，我们选择了 "美国 - 北部和南部，美利坚合众国"、"山区（大部分地区）"、"时间和日期 "屏幕上的 "跳过"。在 "系统配置 "部分，我们取消选择 "sshd" ，选择 "powerd"，然后选择 "OK"。在下一个屏幕中，我们选择了 "9 secure_console"，以便在试图进入系统控制台的单用户模式时提示 root 密码。对添加新用户的提示说 "是"。使用全小写的用户名。输入用户的 "全名"。通过回答问题 "登录组是......邀请......进入其他组？[]: ，然后按 Enter 键。wheel "组的用户可以运行 "特权 "命令。操作员 "组允许用户执行关机和重启计算机等操作，而无需调用 wheel 组的特殊权限。将打算使用 FreeBSD 作为桌面的用户加入 `video` 组是个好主意。我们选择 `` 作为 shell ，其他字段使用默认值，最后提供一个账户密码。如果一切正常，则无需再创建任何用户帐户。

在 "最终配置 "部分，选择 "手册"。我们使用默认的 "en " 并选择 "ok" 。回到配置界面后，选择 "退出 "完成配置，选择 "是 "继续手动配置。我们需要做的唯一手动配置就是关闭机器，以便取出 CD 。在虚拟机上，为了在取出 CD 时不出错，需要关闭电源，因此我们将发出下面的关机命令来关闭计算机。输入 `init 0` 或 `shutdown -p now` 关闭计算机。要从 VirtualBox 中的虚拟机中取出 CD ，请单击虚拟机名称，单击 "设置"，单击 "存储"，然后单击小 CD 图标。注意在 "光驱 "部分旁边还有一个小 CD 图标，下面有一个向下的小箭头。使用第二个 CD 图标 "从虚拟驱动器删除磁盘"。单击 "确定"。

#### 克隆一个 VirtualBox 虚拟机

使用 VirtualBox 中的 "快照 "菜单来 "克隆 "虚拟机。选中 "重新初始化所有网卡的 MAC 地址 "复选框。我们可以选择 "完全克隆 "或 "链接克隆"。我们将使用 "完全克隆 "选项。稍后我们将使用该克隆虚拟机。

### 休息片刻后

启动第一个虚拟机，使用 "Start" （开始）按钮启动机器，并以 root 用户身份登录。

#### 软件包安装/文本配置

在虚拟机上键入以下命令:

```
pkg
```

如果之前没有安装手册，请用 "y " 回复提示引导 pkg 。我们正在安装 Xfce 。你可以轻松安装 3 种桌面环境，即 lumina、xfce 和 kde。

```
pkg install -y xorg sudo xfce virtualbox-ose-additions firefox vim-x11
sysrc vboxguest_enable=YES
sysrc vboxservice_enable=YES
sysrc moused_enable=YES
visudo (this command launches the vi text editor).
```

输入以下内容编辑一行文本，然后退出编辑器：

```
/wheel
```

输入回车键或点击回车键，然后输入

```
j0xxZZ (once you hit the first search term, j goes down, 0 goes to the beginning of the line, xx deletes the comment, ZZ saves and quits vi)
```

```
sysrc dbus_enable=YES
dbus-uuidgen > /etc/machine-id
reboot
```

机器重启后，再次登录，这次使用创建的普通用户账户。然后键入以下命令。

```
vim .xinitrc
```

这次我们使用 vim 编辑器 VI iMproved，开始键入时需要处于 "插入模式"，必须按 `i` 键才能进入插入模式并开始键入文本。键入以下单行配置：

#### XFCE 桌面的 .xinitrc 设置

```
exec startxfce4
```

#### Lumina 桌面的 .xinitrc 设置

```
exec start-lumina-desktop
```

#### KDE Plasma 桌面的 .xinitrc 设置

```
exec startplasma-x11
```

按键盘上的 "Esc " 键退出 vim，然后键 入`ZZ` 或 `:wq`，接着键入 `Enter/Return`。此时，可以使用以下命令启动虚拟机桌面：

```
startx
```

##### 已知的问题

- 在 OSX 上，不要使用绿色按钮来放大窗口，而是从窗口的角落拖动来增大窗口。
- 如果发现自己的鼠标有点慢，或者发现自己的屏幕很小，无法填满 VirtualBox 窗口，或者 startx 无法正常工作，或者加载了桌面但被冻结，请使用 "Machine" （机器）菜单关闭机器，然后选择 "ACPI Shutdown" （ ACPI 关闭）选项，然后执行以下操作：
  - 打开虚拟机的 "设置"->"显示 "选项卡
  - 确认 `VBoxVGA` 被选中，忽略无效设置提示，因为我们需要使用该特定显示驱动程序
  - 单击 "确定"，重新启动机器。

#### Lumina 桌面配置

在桌面上单击右键 > `首选项' > `所有桌面设置' 在 "外观 "下单击 "主题"，然后从侧边栏中选择 "图标"，再选择 "材质设计（浅色）"，然后单击 "应用"。关闭 "主题设置 "窗口。

返回 "桌面设置 "窗口并选择 "窗口管理器"。在 "窗口主题 "下拉菜单中，选择 "两次"。单击 "保存"。关闭 "窗口管理器设置 "窗口。返回 "桌面设置 "窗口并选择 "应用程序"。在这里，我们可以设置用于打开特定文件类型的默认应用程序。如果将 "电子邮件客户端 "设置为 Firefox，就可以使用 Firefox 内置的 "mailto " 处理程序，在点击电子邮件地址链接时自动打开基于 Web 的电子邮件客户端，如 Gmail 或 Yahoo Mail 。有关其他支持的网络邮件平台和其他可用配置设置，请参阅 Firefox 文档。在 "虚拟终端 "部分，将默认设置设为 Sakura。关闭所有设置窗口。

右键点击桌面，选择 "首选项"，然后选择 "壁纸"，点击 "+"，选择 "纯色"。我们选择红色。点击 "好"，然后点击 "保存"。

#### FreeBSD 手册

下面介绍如何打开我们在 "最终配置 "中安装的手册的本地副本：

- 在应用程序菜单中查找 Firefox
- 在 XFCE 中:
  - 点击 `应用程序` > `网络` > `Firefox 网络浏览器`
- 在 Lumina 中:
  - 右键点击: `桌面` > `应用程序` > `网络` > `Firefox 网络浏览器`
- 打开火狐浏览器并导航至以下 URL: `file:///usr/local/share/doc/freebsd/en/books/handbook/book.html`

这是 FreeBSD 手册的本地拷贝，由于不需要互联网连接，因此可以随时使用。这就是安装 FreeBSD 的全部过程。下一步建议选择并配置防火墙。如果您购买了《BSD Hacks》一书，您将会对 `` shell 更加熟悉，并掌握一些新的技能。

### 返回到幻灯片

## FreeBSD 更新和软件包

### 更新

```
sudo -i
env PAGER=cat freebsd-update fetch install
```

我们使用 env PAGER=cat 来修改 Freebsd-update shell 脚本使用的 PAGER 变量的值。如果打开 shell 脚本，你会看到 PAGER 的默认值被设置为 `less`。输入 `which freebsd-update` 可以找到 freebsd-update shell 脚本的位置。将 PAGER 的值设置为 `cat` 后，`freebsd-update` 就不会停下来显示要更新的文件列表，相反， `cat` 命令只会显示文件列表，而不会停下来等你读取。如果你只键入 `freebsd-update fetch install`，你需要使用下面的命令来通过提示：

向下翻页和 q 应该可以让你通过提示。如果屏幕显示 "END" （结束），请按 q ，然后在其他屏幕上使用 "Page Down" （向下翻页）。也可以在每个屏幕上都使用 q 。

既然 FreeBSD 本身是最新的，我们也应该更新软件包。

```
pkg update
pkg upgrade -y
```

### Jails

#### IOCAGE

安装 iocage

```
pkg install -y py39-iocage
```

以下命令假定您的 zpool 名称为 zroot ，必要时可进行调整。输入 `zpool status` 查看 zpool 的当前名称。

```
iocage activate zroot
iocage list
```

现在我们应该看到 iocage list 输出的是一个空表，下面我们将做一些额外的内务处理，以提高 iocage 的性能。要优化各种应用程序的性能，你可以做很多事情。学习新技巧和窍门的方法之一是阅读 https://MWL.io 上的 FreeBSD Mastery 系列丛书（事实上，下面的性能优化窍门就在 FreeBSD Mastery 一书中）： Jails）

```
mount -t fdescfs null /dev/fd
vi /etc/fstab
```

在文件中添加以下一行，完成后退出。键入 `j` 直到文件底部，键入 `o` 打开新行并开始键入，ESC 然后键入 `ZZ` 保存并退出文件。在 fstab 文件的每个字段之间按 TAB 键，而不是使用空格来分隔字段。

```
fdescfs /dev/fd fdescfs rw 0 0
```

首先下载 FreeBSD 的发行版分支

```
iocage fetch
```

我们使用 [Enter] 来获取 iocage 希望提供的默认版本。使用以下命令验证下载

```
iocage list -r
```

查看帮助文档

```
iocage --help
```

任何子命令都可以附加 --help 标记

```
iocage create --help
```

创建一个基础 jail

```
iocage create -T -n JAILNAME ip4_addr="10.0.2.16" -r 13.1-RELEASE
```

列出当前的 jails

```
iocage list
```

启动 jail-one 并跳转到控制台

```
iocage console JAILNAME
```

如果 jail 尚未启动，控制台命令将启动 jail 。要手动启动 jail ，请使用

```
iocage start JAILNAME
```

同样地停止一个有问题的 jail

```
iocage stop JAILNAME
```

验证网络是否正常运行

```
pkg
```

对 pkg bootstrap 提示点 "是"。
使用 `ctrl + d` 退出 jail 控制台。

删除一个 jail

```
iocage destroy JAILNAME
```

# Poudriere

以下命令可以在我们目前使用的虚拟机上运行，但编译所有软件包需要很长时间。

我们将使用之前创建的克隆虚拟机来探索 Poudriere 。

启动克隆虚拟机。

以 root 用户身份执行的命令：

```
pkg install -y poudriere vim
mkdir /usr/ports/distfiles
mkdir /usr/local/poudriere
vim /usr/local/etc/poudriere.conf
```

`poudriere.conf` 要修改的内容

```
ZPOOL=zroot
FREEBSD_HOST=https://download.freebsd.org
```

继续执行：

```
poudriere jail -c -j amd64-13-1 -v 13.1-RELEASE
poudriere jail -l
pkg install -y git
poudriere ports -cp head
poudriere ports -l
```

凑出一个要构建的软件包列表。我在笔记本电脑上运行了 `pkg query -e '%a=0' %o`，结果出现了一个很大的列表，而在我的虚拟机上运行的 pkg query 要短得多。要了解有关 `pkg query` 的更多信息，请执行命令 `pkg help query` 查看查询子命令的帮助文本。大多数命令都支持某种形式的帮助（-h、--help 或使用 `man PROGRAM` 阅读手册）。将系统上当前安装的软件包导出到名为 pkglist 的文件的命令是

```
pkg query -e '%a=0' %o | tee pkglist
```

```
editors/vim
```

**注意**: `tee` 会将输出写入文件，也会写入 `stdout`。另外，以下两点信息对于本教程来说是可选的，目的是向你展示如何定制软件包。

为软件包配置自定义选项有两种方法：手动和使用正常的 make 配置屏幕，
仅供参考：手动操作看起来像这样

```
vim /usr/local/etc/poudriere.d/amd64-13-1-make.conf
```

```
OPTIONS_UNSET+= DOCS NLS X11 EXAMPLES IPV6 KERBEROS
DEFAULT_VERSIONS+= ssl=openssl
```

要自定义软件包，可以使用正常的软件包配置屏幕，该屏幕将在发出以下命令时显示

```
poudriere options -j amd64-13-1 -p head -f pkglist
```

在本教程中，我们将使用默认选项实际构建软件包，以便掌握 Poudriere 的使用方法，方法如下

```
poudriere bulk -j amd64-13-1 -p head -f pkglist
cd /usr/local/poudriere
```

该目录包含与 Poudriere 生成相关的日志。在 `/data/logs/bulk/.html/` 中甚至还生成了几个不同的网站文件，其中包含有关构建的信息。你可以在火狐浏览器中使用网址`file://data/logs/bulk/.html/index.html` 打开网站，查看当前 Poudriere 构建的状态。你也可以在命令行中按下 `ctrl + t` 来向 Poudriere 发送 SIGINFO ，这样 Poudriere 就会在命令处理过程中打印出当前的 Poudriere 编译状态。这个小技巧也适用于许多其他命令，可以向你显示状态。例如，如果你正在使用 `scp` 下载一个大文件，并想知道下载进行到哪一步了，SIGINFO 就会在命令行上打印出状态信息。另一个例子是，如果你正在解压一个压缩文件，并想知道解压过程的状态。

一旦 Poudriere 完成，所构建的软件包可从以下网址获取

```
/usr/local/poudriere/data/packages/amd64-13-1-head
```

创建一个 pkg 仓库

```
mkdir -p /usr/local/etc/pkg/repos
```

创建一个文件来覆盖 FreeBSD 软件源文件 `/etc/pkg/FreeBSD.conf` 中的默认设置，从而禁用主 FreeBSD 软件源。

```
vim /usr/local/etc/pkg/repos/FreeBSD.conf
```

键入以下配置，并将启用设置为 no

```
FreeBSD: {
    enabled: no
}
```

为本地 Poudriere 软件包仓库新建一个配置文件，类似于 FreeBSD.conf 文件，用于保存仓库配置

```
vim /usr/local/etc/pkg/repos/amd64-13-1.conf
```

添加以下配置

```
amd64-13-1: {
    url: “file:///usr/local/poudriere/data/packages/amd64-13-1-head”,
    enabled: yes,
}
```

使用新的软件源重新安装所有软件包

```
pkg upgrade -fy
```

## 升级 Poudriere

注意: 一个好的系统管理做法是在配置文件中添加注释，让未来的你知道配置选项的作用，以及为什么要把它们放在那里。这样，当你在一段时间后重新查看配置文件时，就会知道为什么要选择这些选项。

```
vim /usr/local/etc/poudriere.conf
```

```
# Poudriere builds all packages in the pkglist by default, these two lines tell Poudriere to look at each package individually to see if the currently built package is already at the latest version, if so, Poudriere will skip building the package again. This way only packages that need updates are being rebuilt, speeing up the overall build time for your package repository as you issue updates going forward.

CHECK_CHANGED_OPTIONS=verbose
CHECK_CHANGED_DEPS=yes
```

```
poudriere jail -j amd64-13-1 -u
poudriere ports -p head -u
poudriere bulk -j amd64-13-1 -p head -f pkglist
```

现在您可以安装更新的软件包

```
pkg update
```

# Ansible

Ansible 是一种配置管理系统，允许创建可重复使用的代码结构，在服务器组或单台服务器上执行多种操作。Ansible 将任务组织到 Ansible Playbooks 中，并支持将服务器分成多个称为角色的组。你可以将某些任务分配给各自的角色。Ansible 使用 `ssh` 从控制机器向 Ansible 主机文件中列出的机器发送命令。控制机器就是需要安装 Ansible 的地方。远程机器只需要安装 `ssh`。

我们将使用 Ansible 在远程服务器上安装 Poudriere ，该服务器比我的本地机器强大许多倍，然后构建 pkglist 文件，最后将生成的软件包同步到本地机器上。

安装 Ansible

```
pkg install -y py39-ansible
```

获取研讨会的 Ansible 操作手册，并提取 Ansible 压缩文件的内容

```
fetch http://bsd.pw/ansible-directory.tar.gz
tar -xzvf ansible-directory.tar.gz
```

创建 SSH 密钥对，在出现提示时为密钥添加口令，然后使用 `ssh-agent` 将新密钥加载到内存中，此时会提示你输入我们刚刚在密钥上设置的口令，以便将其加载到内存中。这样，我们在每个终端会话中只需输入一次 SSH 密钥的口令。

```
cd ansible
ssh-keygen -t ed25519 -f ansible
ssh-agent
ssh-add ansible
```

将 ansible.pub 密钥的内容复制到 Vultr 的 SSH 密钥部分

```
cat ansible.pub
```

创建另一台机器，选择新的 SSH 密钥，然后复制新机器的 IP 地址

用新机器 IP 更新主机文件

```
vim hosts
```

在运行游戏程序之前，请注意以下几点：

ansible/roles/poudriere/tasks/poudriere.yml "中最后一项名为 "下载编译好的软件包 "的任务被设置为下载到"/home/roller/packages"，所以你可能需要修改一下。

您需要在本地计算机上安装 rsync，才能使用 syncronize 模块并同步软件包目录

你可以告诉 ansible 从某个特定任务开始。例如，使用 `--start-at-task="下载编译好的软件包"`，就会从最后一个任务开始，只使用 rsync 将软件包目录同步到本地机器上

你可以用 `--step` 命令 ansible 逐步执行每个任务，并询问你想做什么。`y` 将执行当前任务，`n` 将跳过该任务，`c` 将继续执行所有剩余任务

### 运行游戏手册

```
ansible-playbook -vv playbook.yml
```

### VNC

如果使用 Vultr 等软件在远程服务器上创建一个 FreeBSD 实例，只需再执行几个步骤即可访问桌面。以普通用户身份登录控制台，然后执行以下命令。确保你已经如上所述编辑了你的 `.xinitrc`，以使 `startx` 启动你喜欢的桌面。

```
sudo pkg install -y x11vnc
startx
```

然后在本地计算机上安装 RealVNC 或其他 VNC 客户端的 VNC Viewer，并使用以下命令创建 ssh 通道。

```
ssh -t -L 5900:localhost:5900 roller@94.8(whatever your IP Address is use that here)
x11vnc -localhost -shared -forever -ultrafilexfer -usepw -display :0
```

系统会要求你设置 VNC 密码，然后按 Y 键保存密码文件。现在就可以使用 VNC 客户端进行连接了，只需将其指向 localhost:5900，并使用刚才设置的 VNC 密码进行连接即可。尽情享受吧
