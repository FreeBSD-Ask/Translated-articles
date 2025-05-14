# 将 FreeBSD 上的 Ansible 主机和客户机都安装在 Jail 中

- 原文链接：<https://github.com/possnfiffer/bsd-pw/blob/gh-pages/docs/Install_Ansible_host_and_guests_on_FreeBSD_all_in_Jails.md>
- 作者：Roller Angel
- 译者：飞鱼
- 校对整理：ykla


## 目前，我正在努力将其转化为 `Ansible Playbook =D` ，我也刚刚从 mwl.io 订购了 *FreeBSD Mastery Jails* 一书，这将是非常有用的

### 在我的实验室里，我使用 OPNSense 防火墙，它也是我的 DHCP 服务器，默认网关设置为 172.16.28.1，它还充当我的 DNS。下面我们将使用 `zroot` 作为 zpool 的名称


在主机上执行：

```
vi /etc/sysctl.conf
```

添加内容：

```
# Allow jail raw sockets
security.jail.allow_raw_sockets=1

# Allow upgrades in jail
security.jail.chflags_allowed=1
```

执行以下内容：

```
sysctl security.jail.allow_raw_sockets=1
```

```
sysctl security.jail.chflags_allowed=1
```

```
vi /boot/loader.conf
```

添加内容：

```
# RACCT/RCTL Resource limits
kern.racct.enable=1
```

运行以下内容：

```
zfs create -o mountpoint=/jail zroot/jail
zfs create -o mountpoint=/jail/ansible0 zroot/jail/ansible0
```

```
fetch -o - http://ftp.freebsd.org/pub/FreeBSD/releases/amd64/12.0-RELEASE/base.txz | tar --unlink -xpJf - -C /jail/ansible0
ls /jail/ansible0
```

```
vi /etc/jail.conf
```

添加以下配置：

```
ansible0 {
  host.hostname = ansible0.lab.bsd.pw;
  ip4.addr = 172.16.28.10;
  interface = em0;
  path = /jail/ansible0;
  exec.start = "/bin/sh /etc/rc";
  exec.stop = "/bin/sh /etc/rc.shutdown";
  exec.clean;
  mount.devfs;
  allow.raw_sockets;
  sysvsem = new;
  sysvshm = new;
  sysvmsg = new;
}
```

运行以下内容：

```
sysrc jail_enable=YES
sysrc jail_ansible0_mount_enable=YES
service jail restart ansible0
jexec 1 tcsh
vi /etc/resolv.conf
```

添加以下 DNS 配置：

```
nameserver 172.16.28.1
```

执行以下内容：

```
ping -c 3 bsd.pw
adduser
```

以下是我敲入的内容：

```
root@ansible0:/ # adduser
Username: ansible
Full name: Ansible user
Uid (Leave empty for default): 
Login group [ansible]: 
Login group is ansible. Invite ansible into other groups? []: wheel
Login class [default]: 
Shell (sh csh tcsh nologin) [sh]: tcsh
Home directory [/home/ansible]: 
Home directory permissions (Leave empty for default): 
Use password-based authentication? [yes]: 
Use an empty password? (yes/no) [no]: 
Use a random password? (yes/no) [no]: 
Enter password: 
Enter password again: 
Lock out the account after creation? [no]: 
Username   : ansible
Password   : *****
Full Name  : Ansible user
Uid        : 1001
Class      : 
Groups     : ansible wheel
Home       : /home/ansible
Home Mode  : 
Shell      : /bin/tcsh
Locked     : no
OK? (yes/no): yes
```

```
pkg install -y sudo python37
visudo
/wheel (press Enter)
j0xxZZ (on first search of wheel, j goes down a line, 0 to the start of a line, xx to delete comment, ZZ to save and quit)
su ansible
cd
sudo -H python3.7 -m ensurepip
sudo -H pip3.7 install pipenv
mkdir ansible
cd ansible
vi hosts
```

添加以下配置：

```
[local]
127.0.0.1 ansible_python_interpreter=/usr/local/bin/python3.7 ansible_connection=local
```

运行以下内容：

```
pipenv install ansible
pipenv shell
ansible local -i hosts -m setup
ansible local -i hosts -m pkgng -a "name=git-lite state=present" -bK
```

奇怪的是......上面的操作在我的 jail 机器里失败了，但当我在真实的硬件设置上设置同样的东西时，它可以正常工作并添加软件包...... jail 得到一个错误，说“Could not update catalogue”（无法更新目录）。

我应该把我的 jail 设置成 Ansible 的 target：

```
sudo -i
sysrc sshd_enable=YES
service sshd start
vi /etc/ssh/sshd_config
```

将 ListenAddress 改为 jaill 的 IP。同时在文件末尾添加一个新行，显示以下配置：

```
AllowUsers ansible
```

执行以下内容：

```
service sshd restart
exit
mkdir .ssh
vi .ssh/authorized_keys
```

`authorized_keys` 应该包括 `cat ansible.pub` 的输出（不管你给你的 ssh 密钥对起什么名字）在它自己的一行中。

从这里开始，我就在我的普通 FreeBSD 笔记本上执行命令：

```
vi ansible.cfg
```

添加以下内容：

```
[defaults]
inventory = hosts
remote_user = ansible
```

不需要一直输入 `-i hosts`。

回到 jail 中（我们可能不再需要把 `ansible` 放在 `wheel` 组中......这需要考虑。）

```
pw groupadd sudo
pw groupmod sudo -m ansible
visudo
```

添加以下配置：

```
%sudo ALL=(ALL) NOPASSWD: ALL
```

不需要一直输入  `-K` 和 sudo 密码，只需使用按键即可。

我在我的主机上添加了一个 `[jails]` 部分，其中有 jail 的 IP，像以下这样：

```
[jails]
172.16.28.10

[jails:vars]
ansible_python_interpreter=/usr/local/bin/python3.7
```

现在我可以成为 sudo，而不必输入密码了：

```
ansible jails -m package -a "name=git-lite state=present" -b
```
