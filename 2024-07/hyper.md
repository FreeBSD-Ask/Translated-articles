# Hyper-V FreeBSD 13 安装感想

<https://qiita.com/nanorkyo/items/d33e1befd4eb9c004fcd>

* [FreeBSD](https://qiita.com/tags/freebsd)
* [Hyper-V](https://qiita.com/tags/hyper-v)

最后更新于 2023-06-12 发布于 2021-04-07

# 开始

在 Windows 10 Pro 20H2 上使用 Hyper-V。在 Hyper-V 上安装 FreeBSD 13。

有关各种 FreeBSD 安装事项，请参考该部分，同时假定已完成了 Hyper-V 相关设置事项。

尤其是有关设置方式的部分将被省略。这次我把重点放在与物理环境和其他虚拟环境不同的方面。

由于这次设置没有考虑到目的和持久性，相当大一部分是“随意的”。请按需分配资源。

## 前提条件

* 使用 FreeBSD 13（ FreeBSD 13.0 ）的 iso 进行镜像安装
* Windows 10（ Windows 10 Pro 20H2），安装了 Hyper-V
* 创建虚拟机时，根据 FreeBSD 兼容设备进行配置
  * 虚拟机代际：第二代（支持 UEFI 环境和 64 位操作系统）
  * 内存：随意分配并开启使用动态内存
  * 网络配置：连接到 Default Switch
  * 硬盘：随意分配虚拟硬盘
  * 安装参数：从启动光盘／光盘安装操作系统，并指定要安装 ISO 镜像文件
* 对创建的虚拟机进行额外设置
  * 安全性：禁用安全启动
  * 处理器：随意更改虚拟处理器的数量
  * 自动启动 & 自动停止：根据用途进行设置
* 为创建的虚拟机进行附加设置 :🆕: ※需要 CUI 设置和确认
  * 串行控制台：命名管道设置

# Hyper-V 与 FreeBSD 的规范调整

## CMOS 时钟时间（BIOS 时间）

在硬件（虚拟机）上采用 UTC。由于没有自定义项，因此假设 CMOS 时钟时间设置为 UTC。

 具体以下步骤确认。

* 没有 /etc/wall_cmos_clock 这个文件。如果有，则删除。
* 执行 sysctl machdep.wall_cmos_clock ，结果值为 `0`。如果不为 `0`，则置为 `0`。

现在的安装程序不会执行该设置，因此建议，即使在物理环境下，也使用 UTC 时间。

## PS/2 旧设备

在第二代虚拟机环境中，所谓的 PS/2 旧设备已经没有了。这意味着可以使用非标准键盘和鼠标设备驱动程序。这反而浪费时间去查找再禁用，不如将以下设置写入 `/boot/loader.conf` 以禁用它。

`/boot/loader.conf`：

```sh
hint.atkbdc.0.disabled="1"
hint.atkbd.0.disabled="1"
hint.psm.0.disabled="1"
```

键盘输入由设备 `hvkbd0` 处理。遗憾的是，目前不支持鼠标（HID）的使用。截至 2023/04/01，已经集成到 13-STABLE，但未包含在 13.2-RELEASE 中（参见 <https://bugs.freebsd.org/bugzilla/show_bug.cgi?id=221074>）。

对我个人来说，我关注的 hvkbd0 尚未集成到 kbdmux0 中，以及 kbd0 尚未完全封装等问题。虽然它在运行中没有问题...

```sh
 :
WARNING: Device "kbd" is Giant locked and may be deleted before FreeBSD 14.0.
kbd0 at kbdmux0
 :
hvkbd0: <Hyper-V KBD> on vmbus0
 :
```

## 存储设备和网络接口

* 存储设备（连接至 SCSI 控制器的 HDD）可以被引用为 daｎ （n≥0）。
* 网络接口（连接至网络适配器的 NIC）可以被引用为 hnｎ （n≥0）。

关于这一点没有特别说明，但 if_hn(4) 设备可能看起来很陌生，仅供参考。

```sh
# ifconfig hn0
hn0: flags=8843<UP,BROADCAST,RUNNING,SIMPLEX,MULTICAST> metric 0 mtu 1500
        options=8051b<RXCSUM,TXCSUM,VLAN_MTU,VLAN_HWTAGGING,TSO4,LRO,LINKSTATE>
        ether 00:15:5d:xx:xx:xx
        inet 172.30.226.166 netmask 0xfffff000 broadcast 172.30.239.255
        media: Ethernet autoselect (10Gbase-T <full-duplex>)
        status: active
        nd6 options=29<PERFORMNUD,IFDISABLED,AUTO_LINKLOCAL>
```

再者， daｎ 处于虚拟化主机总线适配器（ hv_storvsc(4) 驱动程序）之上，因此已 CAM 化。

```sh
# camcontrol identify da0
camcontrol: ATA ATA_IDENTIFY via pass_16 failed
camcontrol: ATA ATAPI_IDENTIFY via pass_16 failed
# camcontrol tags da0 -v
(pass0:storvsc0:0:0:0): dev_openings  255
(pass0:storvsc0:0:0:0): dev_active    0
(pass0:storvsc0:0:0:0): allocated     0
(pass0:storvsc0:0:0:0): queued        0
(pass0:storvsc0:0:0:0): held          0
(pass0:storvsc0:0:0:0): mintags       2
(pass0:storvsc0:0:0:0): maxtags       255
```

临时支持标签队列，因此要充分利用（以下是在 ZFS 中的设置示例）。

```sh
sysctl vfs.zfs.vdev.async_read_max_active=140
sysctl vfs.zfs.vdev.async_write_max_active=108
```

在调整这些参数时，在执行 gstat 命令期间进行一些负载操作，监视 L(q) 值（队列数）、 r/s （读 IOPS）、 w/s （写 IOPS）。同时观察 ms/r （每读操作时间）、 ms/w （每写操作时间）以及整个系统的负载，以便在不影响主要性能的情况下进行调整。

在某些微妙的地方很难判断，因为指标会随着是 HDD 还是 SSD 而变化，所以不能一概而论。

## TRIM/UNMAP 支持

Hyper-V 的规格书中也写道，执行 sysctl kern.geom.disk.daｎ.flags 后会显示 CANDELETE ，表明支持 TRIM/UNMAP。在此前提下进行设置。

```
# sysctl kern.geom.disk.da0.flags
kern.geom.disk.da0.flags: be<OPEN,CANDELETE,CANFLUSHCACHE,UNMAPPEDBIO,DIRECTCOMPLETION,CANZONE>
```

## ＴＰＭ未対応

所谓的 TPM 2.0。遗憾的是，目前还不支持。由于不支持安全启动，因此不涉及此类功能。

```
tpmcrb0: <Trusted Platform Module 2.0, CRB mode> iomem 0xfed40000-0xfed40fff on acpi0
device_attach: tpmcrb0 attach returned 6
```

## 事件计时器

这里也嵌入了虚拟事件驱动程序，但选择了 LAPIC（ kern.eventtimer.timer 设置）。 这不是什么大不了的事，但不清楚是否可以这样。 因此，不再深入研究。

```
kern.eventtimer.choice: Hyper-V(1000) LAPIC(100) RTC(0)
kern.eventtimer.et.Hyper-V.quality: 1000
kern.eventtimer.et.Hyper-V.frequency: 10000000
kern.eventtimer.et.Hyper-V.flags: 6
kern.eventtimer.et.RTC.quality: 0
kern.eventtimer.et.RTC.frequency: 32768
kern.eventtimer.et.RTC.flags: 17
kern.eventtimer.et.LAPIC.quality: 100
kern.eventtimer.et.LAPIC.frequency: 100000740
kern.eventtimer.et.LAPIC.flags: 15
kern.eventtimer.periodic: 0
kern.eventtimer.timer: LAPIC
kern.eventtimer.idletick: 0
kern.eventtimer.singlemul: 4
```

## 计时器计数器

```
kern.timecounter.tsc_shift: 1
kern.timecounter.smp_tsc_adjust: 0
kern.timecounter.smp_tsc: 0
kern.timecounter.invariant_tsc: 0
kern.timecounter.fast_gettime: 1
kern.timecounter.tick: 1
kern.timecounter.choice: ACPI-fast(900) Hyper-V-TSC(3000) TSC-low(-100) Hyper-V(2000) dummy(-1000000)
kern.timecounter.hardware: Hyper-V-TSC
kern.timecounter.alloweddeviation: 5
kern.timecounter.timehands_count: 2
kern.timecounter.stepwarnings: 0
kern.timecounter.tc.ACPI-fast.quality: 900
kern.timecounter.tc.ACPI-fast.frequency: 3579545
kern.timecounter.tc.ACPI-fast.counter: 913986421
kern.timecounter.tc.ACPI-fast.mask: 4294967295
kern.timecounter.tc.Hyper-V-TSC.quality: 3000
kern.timecounter.tc.Hyper-V-TSC.frequency: 10000000
kern.timecounter.tc.Hyper-V-TSC.counter: 1667097865
kern.timecounter.tc.Hyper-V-TSC.mask: 4294967295
kern.timecounter.tc.TSC-low.quality: -100
kern.timecounter.tc.TSC-low.frequency: 1651207337
kern.timecounter.tc.TSC-low.counter: 1948344295
kern.timecounter.tc.TSC-low.mask: 4294967295
kern.timecounter.tc.Hyper-V.quality: 2000
kern.timecounter.tc.Hyper-V.frequency: 10000000
kern.timecounter.tc.Hyper-V.counter: 1667098332
kern.timecounter.tc.Hyper-V.mask: 4294967295
```

计时器计数器实际上选择了 Hyper-V-TSC（ kern.timecounter.hardware 的设置）。

就个人感觉而言，虚拟环境的计时器精度不佳，在 1/100 秒级别会出现抖动（波动）。始终存在大约±3％/秒的误差，最大可达±10％/秒。在物理环境中，精度约为 1/1000 秒级别，通常为±0.1％/秒以下，最大为±3％/秒左右。

## 关于 DefaultSwitch 的处理

根据规范，这是一个使用 NAT 封闭环境，可以使用 DHCP 的尽善尽美的环境。如果不考虑太复杂的事情，让我们在 /etc/rc.conf 写下以下设置来操作。

/etc/rc.conf

```
ifconfig_hn0="SYNCDHCP"
```

暂时不讨论 IP 地址的固定方式等。此外，从运行 Hyper-V 的机器（在笔记本电脑上验证）可以通过 SSH 命令连接，当然，还可以使用诸如 PuTTY 等终端软件连接到 FreeBSD（客户操作系统）。

内部到外部的连接没有问题，但外部（PC 外部）到内部的通信方法尚未验证。

## 主机电源状态监控

在检查ｄｍｅｓｇ时注意到，主机的电源状态可以从虚拟机（客户机）中看到。在像 Azure 这样的虚拟化平台上运行时可能无关紧要，但在笔记本电脑上运行时，这可能是一个参考。例如，执行ｔｏｐ命令时，在头部显示当前的充电状态（ battery: 49% ・表示维护充电，正常显示）。此时，由于显示列数的限制，ｔｏｐ命令需要在至少 91 列的终端仿真器上运行。

```
last pid:   898;  load averages:  0.17,  0.13,  0.06; battery: 49% up 0+00:11:54  00:43:03
16 processes:  1 running, 15 sleeping
CPU:  0.0% user,  0.0% nice,  0.0% system,  0.0% interrupt,  100% idle
Mem: 26M Active, 184M Inact, 873M Wired, 2148K Buf, 1981M Free
ARC: 419M Total, 54M MFU, 231M MRU, 4108K Header, 130M Other
     214M Compressed, 332M Uncompressed, 1.55:1 Ratio
Swap: 5120M Total, 5120M Free
```

同样，可以使用ｓｙｓｃｔｌ命令检查电源状态。ｓｙｓｃｔｌ名称、值及其状态如下所示对应。

| sysctl 名称 | １         | ０         | 备考           |
| ------------- | ------------ | ------------ | ---------------- |
| `hw.acpi.acline`            | 电源连接   | 电源未连接 | 电源线连接状态 |
| `hw.acpi.battery.state`            | 电池放电中 | 电池充电中 | 电池充放电状态 |

最近的硬件特征是，在连接电源时也可能会放电电池，因此需要注意。此外，在电池模式下，可以通过 hw.acpi.battery.time 获取剩余运行时间（单位：分钟）。

确实无法获取 CPU 运行频率和 CPU 温度。

### 充电模式

```
# sysctl hw.acpi.battery
hw.acpi.battery.info_expire: 5
hw.acpi.battery.units: 1
hw.acpi.battery.state: 0
hw.acpi.battery.rate: 0
hw.acpi.battery.time: -1
hw.acpi.battery.life: 49
```

### 放电模式

```
hw.acpi.battery.info_expire: 5
hw.acpi.battery.units: 1
hw.acpi.battery.state: 1
hw.acpi.battery.rate: 498
hw.acpi.battery.time: 296
hw.acpi.battery.life: 49
```

# 串口设置

在第二代 VM 中，无法在管理控制台上设置或引用串行，而是需要在以管理员权限启动的终端（PowerShell）上执行。要确认当前设置，请运行 Get-VMComPort -VMName "仮想マシン名" 以检查状态。

```
PS C:\Users\nork> Get-VMComPort -VMName "FreeBSD 13"

VMName     Name  Path
------     ----  ----
FreeBSD 13 COM 1
FreeBSD 13 COM 2

PS C:\Users\nork> 
```

这时可以发现不会在任何地方输出 COM 控制台。

要进行设置，请决定一个合适的命名管道名称，并进行设置。我个人选择了类似 Hyper-V-仮想マシン名-ComPort-ポート番号 的命名规则。然后使用 Set-VMCOmPort 命令进行设置。

```
PS C:\Users\nork> Set-VMComPort -VMName "FreeBSD 13" -Number 1 -Path "\\.\pipe\Hyper-V-FreeBSD13-ComPort-1"
PS C:\Users\nork> 
```

再次检查设置，可以看到已设置路径。

```
PS C:\Users\nork> Get-VMComPort -VMName "FreeBSD 13"

VMName     Name  Path
------     ----  ----
FreeBSD 13 COM 1 \\.\pipe\Hyper-V-FreeBSD13-ComPort-1
FreeBSD 13 COM 2


PS C:\Users\nork> 
```

请注意，设置的命名管道只在虚拟机运行时可见，在停止后将不可见。

```
PS C:\Users\nork> Get-ChildItem -Path "\\.\pipe\" -Filter Hyper-V*

    ディレクトリ: \\.\pipe

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
------        1601/01/01      9:00              1 Hyper-V-FreeBSD13-ComPort-1

PS C:\Users\nork>
```

实际上，访问此命名管道的方法并不是很好找到合适的工具，这也是一个问题。

# 设备信息

## dmesg

```
Copyright (c) 1992-2021 The FreeBSD Project.
Copyright (c) 1979, 1980, 1983, 1986, 1988, 1989, 1991, 1992, 1993, 1994
        The Regents of the University of California. All rights reserved.
FreeBSD is a registered trademark of The FreeBSD Foundation.
FreeBSD 13.0-RELEASE-p4 #0: Tue Aug 24 07:33:27 UTC 2021
    root@amd64-builder.daemonology.net:/usr/obj/usr/src/amd64.amd64/sys/GENERIC amd64
FreeBSD clang version 11.0.1 (git@github.com:llvm/llvm-project.git llvmorg-11.0.1-0-g43ff75f2c3fe)
SRAT: Ignoring memory at addr 0xc7800000
SRAT: Ignoring memory at addr 0x100000000
SRAT: Ignoring memory at addr 0x1000000000
SRAT: Ignoring memory at addr 0x10000000000
SRAT: Ignoring memory at addr 0x20000000000
SRAT: Ignoring memory at addr 0x40000000000
SRAT: Ignoring memory at addr 0x80000000000
VT(efifb): resolution 1024x768
Hyper-V Version: 10.0.19041 [SP1]
  Features=0x2e7f<VPRUNTIME,TMREFCNT,SYNIC,SYNTM,APIC,HYPERCALL,VPINDEX,REFTSC,IDLE,TMFREQ>
  PM Features=0x0 [C2]
  Features3=0xbed7b2<DEBUG,XMMHC,IDLE,NUMA,TMFREQ,SYNCMC,CRASH,NPIEP>
Timecounter "Hyper-V" frequency 10000000 Hz quality 2000
CPU: 11th Gen Intel(R) Core(TM) i7-11375H @ 3.30GHz (3302.42-MHz K8-class CPU)
  Origin="GenuineIntel"  Id=0x806c1  Family=0x6  Model=0x8c  Stepping=1
  Features=0x1f83fbff<FPU,VME,DE,PSE,TSC,MSR,PAE,MCE,CX8,APIC,SEP,MTRR,PGE,MCA,CMOV,PAT,PSE36,MMX,FXSR,SSE,SSE2,SS,HTT>
  Features2=0xfeda3203<SSE3,PCLMULQDQ,SSSE3,FMA,CX16,PCID,SSE4.1,SSE4.2,MOVBE,POPCNT,AESNI,XSAVE,OSXSAVE,AVX,F16C,RDRAND,HV>
  AMD Features=0x2c100800<SYSCALL,NX,Page1GB,RDTSCP,LM>
  AMD Features2=0x121<LAHF,ABM,Prefetch>
  Structured Extended Features=0xf1bf27a9<FSGSBASE,BMI1,AVX2,SMEP,BMI2,ERMS,INVPCID,NFPUSG,AVX512F,AVX512DQ,RDSEED,ADX,SMAP,AVX512IFMA,CLFLUSHOPT,CLWB,AVX512CD,SHA,AVX512BW,AVX512VL>
  Structured Extended Features2=0x405f42<AVX512VBMI,AVX512VBMI2,GFNI,VAES,VPCLMULQDQ,AVX512VNNI,AVX512BITALG,AVX512VPOPCNTDQ,RDPID>
  Structured Extended Features3=0xbc000010<FSRM,IBPB,STIBP,L1DFL,ARCH_CAP,SSBD>
  XSAVE Features=0xf<XSAVEOPT,XSAVEC,XINUSE,XSAVES>
  IA32_ARCH_CAPS=0x67<RDCL_NO,IBRS_ALL,RSBA,MDS_NO>
Hypervisor: Origin = "Microsoft Hv"
real memory  = 3347054592 (3192 MB)
avail memory = 3204317184 (3055 MB)
Event timer "LAPIC" quality 100
ACPI APIC Table: <VRTUAL MICROSFT>
FreeBSD/SMP: Multiprocessor System Detected: 4 CPUs
FreeBSD/SMP: 1 package(s) x 2 core(s) x 2 hardware threads
random: registering fast source Intel Secure Key RNG
random: fast provider: "Intel Secure Key RNG"
random: unblocking device.
ioapic0 <Version 1.1> irqs 0-23
Launching APs: 3 1 2
KTLS: Initialized 4 threads
Timecounter "Hyper-V-TSC" frequency 10000000 Hz quality 3000
random: entropy device external interface
[ath_hal] loaded
WARNING: Device "kbd" is Giant locked and may be deleted before FreeBSD 14.0.
kbd0 at kbdmux0
000.000056 [4354] netmap_init               netmap: loaded module
mlx5en: Mellanox Ethernet driver 3.6.0 (December 2020)
nexus0
efirtc0: <EFI Realtime Clock>
efirtc0: registered as a time-of-day clock, resolution 1.000000s
cryptosoft0: <software crypto>
aesni0: <AES-CBC,AES-CCM,AES-GCM,AES-ICM,AES-XTS,SHA1,SHA256>
acpi0: <VRTUAL MICROSFT>
atrtc0: <AT realtime clock> port 0x70-0x71 irq 8 on acpi0
atrtc0: registered as a time-of-day clock, resolution 1.000000s
Event timer "RTC" frequency 32768 Hz quality 0
Timecounter "ACPI-fast" frequency 3579545 Hz quality 900
acpi_timer0: <32-bit timer at 3.579545MHz> port 0x408-0x40b on acpi0
cpu0: <ACPI CPU> on acpi0
acpi_syscontainer0: <System Container> on acpi0
vmbus0: <Hyper-V Vmbus> on acpi_syscontainer0
uart0: <16550 or compatible> port 0x3f8-0x3ff irq 4 flags 0x10 on acpi0
uart1: <16550 or compatible> port 0x2f8-0x2ff irq 3 on acpi0
vmgenc0: <VM Generation Counter> on acpi0
vmbus_res0: <Hyper-V Vmbus Resource> irq 5 on acpi0
battery0: <ACPI Control Method Battery> on acpi0
acpi_acad0: <AC Adapter> on acpi0
Timecounters tick every 10.000 msec
ZFS filesystem version: 5
ZFS storage pool version: features support (5000)
usb_needs_explore_all: no devclass
vmbus0: version 4.0
hvet0: <Hyper-V event timer> on vmbus0
Event timer "Hyper-V" frequency 10000000 Hz quality 1000
hvheartbeat0: <Hyper-V Heartbeat> on vmbus0
hvkvp0: <Hyper-V KVP> on vmbus0
hvshutdown0: <Hyper-V Shutdown> on vmbus0
hvtimesync0: <Hyper-V Timesync> on vmbus0
hvtimesync0: RTT
hvvss0: <Hyper-V VSS> on vmbus0
storvsc0: <Hyper-V SCSI> on vmbus0
hvkbd0: <Hyper-V KBD> on vmbus0
hn0: <Hyper-V Network Interface> on vmbus0
hn0: Ethernet address: 00:15:5d:XX:XX:XX
hn0: link state changed to UP
da0 at storvsc0 bus 0 scbus0 target 0 lun 0
da0: <Msft Virtual Disk 1.0> Fixed Direct Access SPC-3 SCSI device
da0: 300.000MB/s transfers
da0: Command Queueing enabled
da0: 130048MB (266338304 512 byte sectors)
```

## devinfo -rv

```
nexus0
  efirtc0
  cryptosoft0
  aesni0
  ram0
      I/O memory addresses:
          0x0-0x9efff
          0x100000-0xc6593fff
          0xc6595000-0xc66cbfff
          0xc671f000-0xc779afff
          0xc77ff000-0xc77fffff
  apic0
  acpi0
      Interrupt request lines:
          0x9
    acpi_syscontainer0 pnpinfo _HID=ACPI0004 _UID=0 _CID=none at handle=\_SB_.VMOD
      vmbus0
          I/O memory addresses:
              0xf8000000-0xf87fffff
        hvet0
        unknown pnpinfo classid=f8e65716-3cb3-4a06-9a60-1889c5cccab5 deviceid=99221fa0-24ad-11e2-be98-001aa01bbf6e
        unknown pnpinfo classid=34d14be3-dee4-41c8-9ae7-6b174977c192 deviceid=eb765408-105f-49b6-b4aa-c123b64d17d4
        hvheartbeat0 pnpinfo classid=57164f39-9115-4e78-ab55-382f3bd5422d deviceid=fd149e91-82e0-4a7d-afa6-2a4166cbd7c0
        hvkvp0 pnpinfo classid=a9a0f4e7-5a45-4d96-b827-8a841e8c03e6 deviceid=242ff919-07db-4180-9c2e-b86cb68c8c55
        hvshutdown0 pnpinfo classid=0e0b6031-5213-4934-818b-38d90ced39db deviceid=b6650ff7-33bc-4840-8048-e0676786f393
        hvtimesync0 pnpinfo classid=9527e630-d0ae-497b-adce-e80ab0175caf deviceid=2dd1ce17-079e-403c-b352-a1921ee207ee
        hvvss0 pnpinfo classid=35fa2e29-ea23-4236-96ae-3a6ebacba440 deviceid=2450ee40-33bf-4fbd-892e-9fb06e9214cf
        storvsc0 pnpinfo classid=ba6163d9-04a1-4d29-b605-72e2ffb1dc7f deviceid=140cd59f-fb5d-4dfd-976a-d54ef05992fb
        unknown pnpinfo classid=525074dc-8985-46e2-8057-a307dc18a502 deviceid=1eccfd72-4b41-45ef-b73a-4a6e44c12924
        unknown pnpinfo classid=cfa8b69e-5b4a-4cc0-b98b-8ba1a1f3f95a deviceid=58f75a6d-d949-4320-99e1-a2a2576d581c
        hvkbd0 pnpinfo classid=f912ad6d-2b17-48ea-bd65-f927a61c7684 deviceid=d34b2567-b9b6-42b9-8778-0a4ec0b955bf
        unknown pnpinfo classid=da0a7802-e377-4aac-8e77-0558eb1073f8 deviceid=5620e0c7-8062-4dce-aeb7-520c7ef76171
        unknown pnpinfo classid=3375baf4-9e15-4b30-b765-67acb10d607b deviceid=4487b255-b88c-403f-bb51-d1f69cf17f87
        unknown pnpinfo classid=276aacf4-ac15-426c-98dd-7521ad3f01fe deviceid=f5bee29c-1741-4aad-a4c2-8fdedb46dcc2
        hn0 pnpinfo classid=f8615163-df3e-46c5-913f-f2d2f965ed0e deviceid=5e4f5b37-61e9-4ba5-beff-64dbfdddc551
    unknown pnpinfo _HID=PNP0003 _UID=0 _CID=none at handle=\_SB_.VMOD.APIC
        I/O memory addresses:
            0xfec00000-0xfec00fff
            0xfee00000-0xfee00fff
    vmbus_res0 pnpinfo _HID=VMBUS _UID=0 _CID=none at handle=\_SB_.VMOD.VMBS
    battery0 pnpinfo _HID=PNP0C0A _UID=0 _CID=VIRTUAL BATTERY at handle=\_SB_.VMOD.BAT1
    acpi_acad0 pnpinfo _HID=ACPI0003 _UID=0 _CID=VIRTUAL AC ADAPTER at handle=\_SB_.VMOD.AC1_
    uart0 pnpinfo _HID=PNP0501 _UID=1 _CID=none at handle=\_SB_.UAR1
        Interrupt request lines:
            0x4
        I/O ports:
            0x3f8-0x3ff
    uart1 pnpinfo _HID=PNP0501 _UID=2 _CID=none at handle=\_SB_.UAR2
        Interrupt request lines:
            0x3
        I/O ports:
            0x2f8-0x2ff
    vmgenc0 pnpinfo _HID=HYPER_V_GEN_COUNTER_V1 _UID=0 _CID=VM_GEN_COUNTER at handle=\_SB_.GENC
    atrtc0 pnpinfo _HID=PNP0B00 _UID=0 _CID=none at handle=\_SB_.RTC0
        Interrupt request lines:
            0x8
        I/O ports:
            0x70-0x71
    cpu0 pnpinfo _HID=ACPI0007 _UID=1 _CID=none at handle=\P001
    cpu1 pnpinfo _HID=ACPI0007 _UID=2 _CID=none at handle=\P002
    cpu2 pnpinfo _HID=ACPI0007 _UID=3 _CID=none at handle=\P003
    cpu3 pnpinfo _HID=ACPI0007 _UID=4 _CID=none at handle=\P004
    acpi_timer0 pnpinfo unknown
        I/O ports:
            0x408-0x40b
```

从这个结果可以看出，半虚拟化设备被放置在 acpi_syscontainer0 上。由于没有 PCI（PCIe）桥，因此 pciconf 命令不会显示任何内容。存在 UART，但正在调查如何使用它。如果完全无头运行，使用 UART 会非常方便。

# 参考文献

* [在安装在 Windows 上的 Hyper-v 的 Linux 和 FreeBSD 虚拟机是受支持的。](https://docs.microsoft.com/ja-jp/windows-server/virtualization/hyper-v/supported-linux-and-freebsd-virtual-machines-for-hyper-v-on-windows)
* [HYPER-V 支持的 FreeBSD 虚拟机](https://docs.microsoft.com/ja-jp/windows-server/virtualization/hyper-v/supported-freebsd-virtual-machines-on-hyper-v)
* [Hyper-v 上的 Linux 和 FreeBSD 虚拟机功能说明](https://docs.microsoft.com/ja-jp/windows-server/virtualization/hyper-v/feature-descriptions-for-linux-and-freebsd-virtual-machines-on-hyper-v)
* [在 HYPER-V 上运行 FreeBSD 的最佳实践](https://docs.microsoft.com/ja-jp/windows-server/virtualization/hyper-v/best-practices-for-running-freebsd-on-hyper-v)
* [ 内核调试添加 COM 端口](https://learn.microsoft.com/ja-jp/windows-server/virtualization/hyper-v/plan/should-i-create-a-generation-1-or-2-virtual-machine-in-hyper-v#add-a-com-port-for-kernel-debugging)
* [虚拟机 Gen 2 安装没有鼠标](https://bugs.freebsd.org/bugzilla/show_bug.cgi?id=221074)
* [ FreeBSD 时间](https://qiita.com/yamori813/items/d9b0a451f1e115de893f)
* [ 时间计数器(4)](https://www.freebsd.org/cgi/man.cgi?timecounters(4))
* [ 事件定时器(4)](https://www.freebsd.org/cgi/man.cgi?eventtimers(4))
