# FreeBSD Bhyve PCI 直通

- <https://qiita.com/yshdsnd/items/0b8c38425a6a3bae289b>


* [ 自由开放操作系统](https://qiita.com/tags/freebsd)
* [ BHyVe 虚拟机](https://qiita.com/tags/bhyve)
* [ PCI 透传](https://qiita.com/tags/pci-passthrough)

最后更新于 2023-01-27 发布于 2022-12-18

## 1.首先

我想在 Bhyve 的 Windows 宾客中使用主机设备，因此尝试设置 PCI 直通。我使用 vm-bhyve 作为管理软件，基于这一前提，我将进行以下说明。关于 vm-bhyve，您可以在搜索时找到相当多的解释，请参考那些解释。

这些硬件是我去年购买的，就是这篇文章中提到的硬件。https://qiita.com/yshdsnd/items/e8ba8d417851ae56f2fc 但是，内存已扩展到 128GB。我正在使用 FreeBSD 13.1-STABLE。

## 2. 主机端(硬件)设置

PCI passthrough 需要使用 VT-d 功能。由于通常情况下默认是禁用的，所以请进入 UEFI 菜单启用 VT-d。

![IMG_2153.jpg](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F373056%2F3584500f-cfd5-bec1-20ff-ffbf1ab05af3.jpeg?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=aadf1ec7e0872ffc6cb5141398a25278)

虽然本次不使用，但同时也启用了 SR-IOV。

## 3. 主机端（软件）设置

首先查找要使用的设备 ID。 尽管可以使用 pciconf，但 vm-bhyve 的 vm passthru 命令更简单。

```
# vm passthru
DEVICE     BHYVE ID     READY        DESCRIPTION
hostb0     0/0/0        No           12th Gen Core Processor Host Bridge/DRAM Registers
pcib1      0/1/0        No           12th Gen Core Processor PCI Express x16 Controller
vgapci0    0/2/0        No           AlderLake-S GT1
pcib2      0/6/0        No           12th Gen Core Processor PCI Express x4 Controller
none0      0/8/0        No           12th Gen Core Processor Gaussian & Neural Accelerator
xhci0      0/20/0       No           Alder Lake-S PCH USB 3.2 Gen 2x2 XHCI Controller
none1      0/20/2       No           Alder Lake-S PCH Shared SRAM
ig4iic0    0/21/0       No           Alder Lake-S PCH Serial IO I2C Controller
none2      0/22/0       No           Alder Lake-S PCH HECI Controller
ahci0      0/23/0       No           Alder Lake-S PCH SATA Controller [AHCI Mode]
pcib3      0/28/0       No           -
pcib4      0/29/0       No           -
isab0      0/31/0       No           Z690 Chipset LPC/eSPI Controller
hdac0      0/31/3       No           Alder Lake-S HD Audio Controller
ichsmb0    0/31/4       No           Alder Lake-S PCH SMBus Controller
none3      0/31/5       No           Alder Lake-S PCH SPI Controller
ix0        1/0/0        No           Ethernet Controller 10-Gigabit X540-AT2
ix1        1/0/1        No           Ethernet Controller 10-Gigabit X540-AT2
nvme0      2/0/0        No           E18 PCIe4 NVMe Controller
none0       3/0/0       No           Killer E3000 2.5GbE Controller
xhci1       4/0/0       No           -
```

→ 我想在 Windows 上使用最后的这个 NIC（Killer E3000 2.5GbE 控制器）和 xhci1 设备。

然后在/boot/loader.conf 中写下要作为传递设备保留的设备。

/boot/loader.conf：

```

kern.geom.label.disk_ident.enable="0"
kern.geom.label.gptid.enable="0"

zfs_load="YES"
vmm_load="YES"
pptdevs="3/0/0 4/0/0"
```

pptdevs 现在会记录刚才在 vm passthru 中显示的 ID。此外，必须在此处明确加载 vmm 模块。否则，xhci 驱动程序将首先识别设备，导致设备无法被预留为透传设备。如果只想使用没有 FreeBSD 驱动程序的 Killer E3000 2.5GbE Controller，则无需明确编写 vmm 模块。它将在启动 bhyve 时自动加载。

编辑 loader.conf 后，重新启动操作系统后再次运行 vm passthru。

```

# vm passthru
DEVICE     BHYVE ID     READY        DESCRIPTION
hostb0     0/0/0        No           12th Gen Core Processor Host Bridge/DRAM Registers
pcib1      0/1/0        No           12th Gen Core Processor PCI Express x16 Controller
vgapci0    0/2/0        No           AlderLake-S GT1
pcib2      0/6/0        No           12th Gen Core Processor PCI Express x4 Controller
none0      0/8/0        No           12th Gen Core Processor Gaussian & Neural Accelerator
xhci0      0/20/0       No           Alder Lake-S PCH USB 3.2 Gen 2x2 XHCI Controller
none1      0/20/2       No           Alder Lake-S PCH Shared SRAM
ig4iic0    0/21/0       No           Alder Lake-S PCH Serial IO I2C Controller
none2      0/22/0       No           Alder Lake-S PCH HECI Controller
ahci0      0/23/0       No           Alder Lake-S PCH SATA Controller [AHCI Mode]
pcib3      0/28/0       No           -
pcib4      0/29/0       No           -
isab0      0/31/0       No           Z690 Chipset LPC/eSPI Controller
hdac0      0/31/3       No           Alder Lake-S HD Audio Controller
ichsmb0    0/31/4       No           Alder Lake-S PCH SMBus Controller
none3      0/31/5       No           Alder Lake-S PCH SPI Controller
ix0        1/0/0        No           Ethernet Controller 10-Gigabit X540-AT2
ix1        1/0/1        No           Ethernet Controller 10-Gigabit X540-AT2
nvme0      2/0/0        No           E18 PCIe4 NVMe Controller
ppt0       3/0/0        Yes          Killer E3000 2.5GbE Controller
ppt1       4/0/0        Yes          -
```

指定的设备已被注册为 ppt0、ppt1 并作为透传设备。

# 4. 客人的启动设定

在 vm-bhyve 的设定文件中注册透传设备。因为已经创建了名为 Windows 的客人，所以使用 vm config windows 来编辑设定文件。

```

loader="uefi"
graphics="yes"
xhci_mouse="yes"
cpu=4
cpu_sockets=1
cpu_cores=4
cpu_threads=1
memory=16G
ahci_device_limit="8"
network0_type="virtio-net"
network0_switch="lan0"

disk0_type="ahci-hd"
disk0_name="disk0"
disk0_dev="sparse-zvol"

graphics_port=5902
graphics_res="1920x1080"

# windows expects the host to expose localtime by default, not UTC
utctime="no"

# JP keyboard
bhyve_options="-K jp"
uuid="xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx"
network0_mac="58:9c:fc:xx:xx:xx"

passthru0="3/0/0"
passthru1="4/0/0"
```

以 "passthruX="BUS/SLOT/FUNC"" 的形式进行记录。只需直接记录 vm passthru 显示的内容即可。 虽然与透传无关，但最近的-STABLE 版本增加了通过 VNC 连接时指定键盘映射的选项（-K），因此即使通过日本键盘使用，也不会出现无法正确输入符号的情况。我已将该选项指定为 bhyve_options。

## 5. 尝试启动客户端

 到这一步只需启动即可。

在 Windows 设备管理器中确认是否已添加。如有需要，请安装驱动程序。

如果一切正常识别就是这样。ASMedia USB 3.1 eXtensible Host Controller 是透传设备，其下的 Intel xhci 控制器是 Bhyve 的虚拟设备。再往下的 Killer E3100G 也是透传设备。

![截图2022-12-16 190045.png](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F373056%2F95619590-f66e-8023-88b3-4e35373460b7.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=ece585965d94c9f96f571b9b0c216bf0)

## 6. 但是问题是...

在 Windows 端安装驱动并重启后，bhyve 进程异常终止。无论启动多少次都不行。查看日志显示，以 status 4 结束。

 在日志中如下所示。

```
12月 16 19:13:17: bhyve exited with status 0
12月 16 19:13:17: restarting
12月 16 19:13:17:  [bhyve options: -c 4,sockets=1,cores=4,threads=1 -m 16G -Hwl bootrom,/usr/local/share/uefi-firmware/BHYVE_UEFI.fd -K jp -U 0e7bf333-96b9-11ea-bb29-e8611f133073 -S]
12月 16 19:13:17:  [bhyve devices: -s 0,hostbridge -s 31,lpc -s 4:0,ahci,hd:/dev/zvol/zroot/data/vm/windows/disk0 -s 5:0,virtio-net,tap4,mac=58:9c:fc:06:57:ac -s 6:0,passthru,3/0/0 -s 7:0,passthru,4/0/0 -s 8:0,fbuf,tcp=0.0.0.0:5902,w=1920,h=1080 -s 9:0,xhci,tablet]
12月 16 19:13:17:  [bhyve console: -l com1,/dev/nmdm-windows.1A]
12月 16 19:13:17: starting bhyve (run 2)
12月 16 19:13:18: bhyve exited with status 4
12月 16 19:13:18: destroying network device tap4
12月 16 19:13:18: stopped
```

从配置文件中删除 passthru 后，可以正常启动。此外，重新启动主机然后启动可以正常运行。然而，如果重新启动来宾，则无法再次启动。

我认为可能有某种问题导致无法重新初始化 passthrough 设备，无法解决。每次重新启动来宾都需要重启主机，来宾重启没有意义，让我很困惑...如果有人知道解决方案，请告诉我。

***添加于2023年1月27日**

由于 13-STABLE 源代码树中 bhyve 命令和内核的 vmm 相关部分有重大更改，因此我怀疑并尝试更新到最新的 13-STABLE 后，成功地使客户机可以重新启动。现在可以放心地充分使用了。
