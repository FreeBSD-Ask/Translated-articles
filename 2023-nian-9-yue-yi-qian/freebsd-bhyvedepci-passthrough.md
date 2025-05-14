# FreeBSD Bhyve 上的 PCI 直通

- 原地址：<https://qiita.com/yshdsnd/items/0b8c38425a6a3bae289b>
- 作者：yshdsnd
- 最后发布日期：2023 年 1 月 27 日
- 译者：ykla & ChatGPT

## 1、开始

我想在 Bhyve 的 Windows 虚拟机中使用主机设备，所以尝试进行了 PCI 直通的设置。作为管理软件，我使用了 vm-bhyve，因此在这个前提下我会提供以下说明。关于 vm-bhyve 的内容在搜索时可以找到相当多的解释，请参考那些资料。

硬件是我去年购买的，与这篇文章中所写的内容相同。[https://qiita.com/yshdsnd/items/e8ba8d417851ae56f2fc](https://qiita.com/yshdsnd/items/e8ba8d417851ae56f2fc)不过值得注意的是，内存已经增加到了 128GB。我正在的是使用 FreeBSD 13.1-STABLE。

## 2. 物理机（硬件）设置

PCI 直通需要使用 VT-d 功能。由于默认情况下通常被禁用，因此请进入 UEFI 菜单并启用 VT-d。

![image](https://github.com/FreeBSD-Ask/Translated-articles/assets/10327999/c905e5b1-2890-41e3-80af-0917bce19556)

虽然本次不会使用，但我们也一并启用了 SR-IOV。

## 3. 物理机（软件）设置

首先，我们需要查找所需设备的 ID。虽然你可以使用 `pciconf`，但是 vm-bhyve 的 `vm passthru` 命令更加简便。

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

这里的最后一个 NIC（Killer E3000 2.5GbE 控制器）和 xhci1 设备我想在 Windows 中使用。

然后，在/boot/loader.conf 中，将要预留为直通设备的设备进行记录。

`/boot/loader.conf`：

```
kern.geom.label.disk_ident.enable="0"
kern.geom.label.gptid.enable="0"

zfs_load="YES"
vmm_load="YES"
pptdevs="3/0/0 4/0/0"
```

在 pptdevs 中，写入之前使用 `vm passthru` 显示的 ID。同时，在这里必须明确加载 vmm 模块。这是因为如果不这样做，xhci 驱动程序会首先识别设备，导致无法将其预留为直通设备。如果你只想使用没有 FreeBSD 驱动程序的 Killer E3000 2.5GbE 控制器，则无需显式写入 vmm 模块。它会在启动 bhyve 时自动加载。

编辑 `loader.conf` 后，重新启动操作系统，然后再次运行 `vm passthru`。

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

指定的设备已被注册为 ppt0、ppt1 等直通设备。

## 4. 虚拟机启动设置

将直通设备注册到 vm-bhyve 的配置文件中。
由于已经创建了名为"windows"的虚拟机，因此可以通过执行`vm config windows`来编辑配置文件。

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

使用`passthruX="BUS/SLOT/FUNC"`的格式进行记录。只需将 vm passthru 显示的内容直接复制到这里即可。虽然与 passthru 无关，但是在最近的-STABLE 版本中，添加了通过 VNC 连接时指定键盘映射的选项（-K），这样即使通过日语键盘使用，也不再出现无法正确输入符号的情况。这个选项已经被添加到了`bhyve_options`中。

## 5. 尝试启动虚拟机

上述步骤完成后，只需启动即可。

```
# vm start windows
```

确认在 Windows 的设备管理器中是否已添加。如果需要，安装所需的驱动程序。

如果一切顺利，你将看到以下情况：ASMedia USB 3.1 eXtensible Host Controller 是直通的设备，下面的 Intel xhci 控制器是 Bhyve 的虚拟设备。再往下的 Killer E3100G 也是直通的设备。

![image](https://github.com/FreeBSD-Ask/Translated-articles/assets/10327999/d199ff5a-427d-43cf-9540-c4a3c7e88115)

## 6. 但是遇到了问题

在 Windows 侧安装驱动程序并重新启动后，bhyve 进程异常终止了。无论尝试多少次都无效。查看日志，显示为 status 4 终止。

日志内容如下：

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

从配置文件中删除 passthru 后，可以顺利启动。此外，如果重新启动主机后再启动，也可以正常工作。然而，如果重新启动虚拟机，它又无法启动了。

我认为可能是由于需要重新初始化直通设备，但由于某种问题而未能成功。尽管如此，我仍无法找到解决方法。如果有人知道解决方案，我希望能得到指导。每次需要重新启动主机才能重新启动虚拟机，这对虚拟机来说并不是很有意义，让人感到困扰...如果有人知道解决方法，请告诉我。

## ※2023 年 1 月 27 日 更新

由于 13-STABLE 的源代码对 bhyve 命令和内核的 vmm 相关部分进行了重大更改，因此我尝试了一下更新到最新的 13-STABLE 版本。令人欣慰的是，现在虚拟机可以正常重新启动了。现在可以放心大胆地使用了。
