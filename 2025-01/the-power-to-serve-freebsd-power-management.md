# 服务之力：FreeBSD 电源管理

- 原文：[The Power to Serve – FreeBSD Power Management](https://vermaden.wordpress.com/2018/11/28/the-power-to-serve-freebsd-power-management/)
- 2018/11
- 作者：𝚟𝚎𝚛𝚖𝚊𝚍𝚎𝚗

这是 FreeBSD 操作系统的格言 —— “The Power to Serve（服务之力）” —— 也非常契合本文的主题。十年前（是的，光阴荏苒）我甚至还制作了一张带有这个格言的壁纸 —— 仍可在 DeviatArt 页面上获取。

![](https://vermaden.wordpress.com/wp-content/uploads/2018/11/freebsd_the_power_to_serve_small1.jpg?w=960)

是时候写一篇关于 FreeBSD 电源管理特性的文章了。它也适用于 *FreeBSD 桌面* 系列，但不限于此。流行的观点似乎是 FreeBSD 如此偏向服务器，以至于没有任何电源管理机制。事实完全不是这样。虽然在桌面上不那么重要（但仍能减少你的电费）或服务器上不那么关键，但在笔记本电脑上正确配置电源管理是非常必要的，这样它们会拥有更长的电池寿命，并运行得更安静。

我写这篇文章，是因为 *FreeBSD 手册* 在 [11.13. Power and Resource Management](https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/acpi-overview.html) 章节中并未涉及全部这些信息。*FreeBSD on Laptops* 文章的 [4. Power Management](https://docs.freebsd.org/doc/10.1-RELEASE/usr/local/share/doc/freebsd/en_US.ISO8859-1/articles/laptop/power-management.html) 部分来自 FreeBSD 10.1-RELEASE 的远古时代。FreeBSD Wiki 页面上也有一些信息，但部分已过时。

FreeBSD 在电源管理领域有许多机制：

* 关闭没有附加驱动的设备
* 调整 CPU 频率和功耗
* 支持 CPU 睡眠状态（C1 / C1E / C2 / C3 / …）
* 启用/禁用大多数 CPU 可用的睿频
* 每个 USB 设备的电源管理方案
* SATA / AHCI 通道/控制器电源管理
* 挂起/恢复支持（包括用笔记本盖子执行）
* 支持厂商专有工具帮助测量电源管理
* 工具与 ACPI 支持风扇速度控制
* 工具与 ACPI 支持屏幕亮度设置
* 电池容量状态与运行时间估计
* 网络接口的节能方案
* 支持 AMD PowerNow!
* 支持 Intel (Enhanced) SpeedStep
* 支持 Intel Speed Shift
* 支持 AMD Turbo Core
* 支持 Intel 睿频

关于 FreeBSD 系统中不同设置文件的一句话说明：

* **/etc/rc.conf** —— 无需重启，只需重新加载守护进程
* **/etc/sysctl.conf** —— 无需重启 —— 可在运行时设置
* **/boot/loader.conf** —— 这些设置 **需要重启**


## 信息

让我们先从如何获取所需信息开始，例如当前 CPU 速度、已使用的 C 状态、USB 设备当前的电源管理模式、电池容量和剩余时间等。

### 电池

要获取电池信息，你可以使用工具 **acpiconf(8)**。以下是 **acpiconf(8)** 在我的主电池（ThinkPad T420s 笔记本）接通电源源时的输出。

```sh
% acpiconf -i 0
Design capacity:        44000 mWh
Last full capacity:     37930 mWh
Technology:             secondary (rechargeable)
Design voltage:         11100 mV
Capacity (warn):        1896 mWh
Capacity (low):         200 mWh
Low/warn granularity:   1 mWh
Warn/full granularity:  1 mWh
Model number:           45N1037
Serial number:          28608
Type:                   LION
OEM info:               SANYO
State:                  high
Remaining capacity:     100%
Remaining time:         unknown
Present rate:           0 mW
Present voltage:        12495 mV
```

……在接入电源适配器后：

```sh
% acpiconf -i 0
Design capacity:        44000 mWh
Last full capacity:     37930 mWh
Technology:             secondary (rechargeable)
Design voltage:         11100 mV
Capacity (warn):        1896 mWh
Capacity (low):         200 mWh
Low/warn granularity:   1 mWh
Warn/full granularity:  1 mWh
Model number:           45N1037
Serial number:          28608
Type:                   LION
OEM info:               SANYO
State:                  high
Remaining capacity:     100%
Remaining time:         2:31
Present rate:           0 mW
Present voltage:        12492 mV
```

现在，当从笔记本电脑上拔掉电源时，**Remaining time:** 字段会显示该单块电池的剩余时间估计，这里显示为 **2:31**（两小时三十一分钟）。

下面是我的第二块电池的 **acpiconf(8)** 输出（位于 ThinkPad T420s 的 ultrabay，替代了 DVD 驱动器）。

```sh
% acpiconf -i 1
Design capacity:        31320 mWh
Last full capacity:     24510 mWh
Technology:             secondary (rechargeable)
Design voltage:         10800 mV
Capacity (warn):        1225 mWh
Capacity (low):         200 mWh
Low/warn granularity:   1 mWh
Warn/full granularity:  1 mWh
Model number:           45N1041
Serial number:            260
Type:                   LiP
OEM info:               SONY
State:                  high
Remaining capacity:     100%
Remaining time:         unknown
Present rate:           0 mW
Present voltage:        12082 mV
```

……在接入电源适配器后：


```sh
% acpiconf -i 1
Design capacity:        31320 mWh
Last full capacity:     24510 mWh
Technology:             secondary (rechargeable)
Design voltage:         10800 mV
Capacity (warn):        1225 mWh
Capacity (low):         200 mWh
Low/warn granularity:   1 mWh
Warn/full granularity:  1 mWh
Model number:           45N1041
Serial number:            260
Type:                   LiP
OEM info:               SONY
State:                  discharging
Remaining capacity:     98%
Remaining time:         1:36
Present rate:           14986 mW
Present voltage:        11810 mV
```

在拔掉电源的情况下，它会将第二块电池的 **Remaining time:** 显示为 **1:36**。

因此，总电池续航时间估计为 **4:07**。同样的时间，以分钟表示（**247**），会显示在名为 **sysctl(8)** 值 **hw.acpi.battery.time** 中，如下所示。

```sh
% sysctl hw.acpi.battery.time
hw.acpi.battery.time: 247
```

你还可以通过 **sysctl(8)** **hw.acpi.battery**，获取更“完整”的电池信息。

```sh
% sysctl hw.acpi.battery
hw.acpi.battery.info_expire: 5
hw.acpi.battery.units: 2
hw.acpi.battery.state: 1
hw.acpi.battery.time: 247
hw.acpi.battery.life: 99
```

如果连接了电源，**hw.acpi.battery.time** 将显示为 **-1**。

```sh
% sysctl hw.acpi.battery
hw.acpi.battery.info_expire: 5
hw.acpi.battery.units: 2
hw.acpi.battery.state: 0
hw.acpi.battery.time: -1
hw.acpi.battery.life: 100
```

### 电池损耗

随着时间的推移，电池会失去其“设计”容量。经过一到两年，这类电池的实际效率可能只剩下原来的 70% 或更低。

检查这些信息所需的全部数据，都可以通过 **acpiconf(8)** 命令中的 **Design capacity:** 和 **Last full capacity:** 获取。我写了个 **battery-capacity.sh** 脚本，可以告诉你当前电池的效率。下面是该脚本运行时的效果示例。

```sh
% battery-capacity.sh 0
Battery '0' model '45N1037' has efficiency: 86%

% battery-capacity.sh 1
Battery '1' model '45N1041' has efficiency: 78%
```

下面是 **battery-capacity.sh** 脚本。


```sh
#! /bin/sh

if [ ${#} -ne 1 ]
then
  echo "usage: ${0##*/} BATTERY"
  exit
fi

if acpiconf -i ${1} 1> /dev/null 2> /dev/null
then
  DATA=$( acpiconf -i ${1} )
  MAX=$( echo "${DATA}" | grep '^Design\ capacity:'     | awk -F ':' '{print $2}' | tr -c -d '0-9' )
  NOW=$( echo "${DATA}" | grep '^Last\ full\ capacity:' | awk -F ':' '{print $2}' | tr -c -d '0-9' )
  MOD=$( echo "${DATA}" | grep '^Model\ number:'        | awk -F ':' '{print $2}' | awk '{print $1}' )
  echo -n "Battery '${1}' model '${MOD}' has efficiency: "
  printf '%1.0f%%\n' $( bc -l -e "scale = 2; ${NOW} / ${MAX} * 100" -e quit )
else
  echo "NOPE: Battery '${1}' does not exists on this system."
  echo "INFO: Most systems has only '0' or '1' batteries."
  exit 1
fi
```

### CPU

要获取当前 CPU 的信息，你需要使用 **dev.cpu**，或者使用 **dev.cpu.0** 来获取第一个物理 CPU 核心的信息。

```sh
% sysctl dev.cpu.0
dev.cpu.0.cx_method: C1/hlt C2/io
dev.cpu.0.cx_usage_counters: 412905 0
dev.cpu.0.cx_usage: 100.00% 0.00% last 290us
dev.cpu.0.cx_lowest: C1
dev.cpu.0.cx_supported: C1/1/1 C2/3/104
dev.cpu.0.freq_levels: 2501/35000 2500/35000 2200/29755 2000/26426 1800/23233 1600/20164 1400/17226 1200/14408 1000/11713 800/9140
dev.cpu.0.freq: 800
dev.cpu.0.%parent: acpi0
dev.cpu.0.%pnpinfo: _HID=none _UID=0
dev.cpu.0.%location: handle=\_PR_.CPU0
dev.cpu.0.%driver: cpu
dev.cpu.0.%desc: ACPI CPU
```

如果你使用 **kldload(8)** 命令加载内核模块 **coretemp(4)**，就能获得额外的温度信息。

下面是加载内核模块 **coretemp(4)** 后，相同的 **sysctl(8) dev.cpu.0** 输出。

```sh
% sysctl dev.cpu.0
dev.cpu.0.temperature: 49.0C
dev.cpu.0.coretemp.throttle_log: 0
dev.cpu.0.coretemp.tjmax: 100.0C
dev.cpu.0.coretemp.resolution: 1
dev.cpu.0.coretemp.delta: 51
dev.cpu.0.cx_method: C1/hlt C2/io
dev.cpu.0.cx_usage_counters: 16549 0
dev.cpu.0.cx_usage: 100.00% 0.00% last 1489us
dev.cpu.0.cx_lowest: C1
dev.cpu.0.cx_supported: C1/1/1 C2/3/104
dev.cpu.0.freq_levels: 2501/35000 2500/35000 2200/29755 2000/26426 1800/23233 1600/20164 1400/17226 1200/14408 1000/11713 800/9140
dev.cpu.0.freq: 800
dev.cpu.0.%parent: acpi0
dev.cpu.0.%pnpinfo: _HID=none _UID=0
dev.cpu.0.%location: handle=\_PR_.CPU0
dev.cpu.0.%driver: cpu
dev.cpu.0.%desc: ACPI CPU
```

让我说下最有用的值。

CPU 核心温度。

**dev.cpu.0.temperature: 49.0C**

CPU 支持的 C 状态（此 CPU 支持 C1 和 C2）。

**dev.cpu.0.cx_supported: C1/1/1 C2/3/104**

CPU C -states 使用统计（仅使用了 C1 状态）。

**dev.cpu.0.cx_usage_counters: 16549 0
dev.cpu.0.cx_usage: 100.00% 0.00% last 1489us**

CPU 启用的最最深的 C 状态。

**dev.cpu.0.cx_lowest: C1**

CPU 支持的频率级别及其功耗（‘/’ 后面的数字表示功耗）。例如 **2500/35000** 可理解为 2.5 GHz 频率，功耗 35 W，而 **2501** 表示 Turbo 模式。最低频率为 800 MHz，功耗约 9 W。

**dev.cpu.0.freq_levels: 2501/35000 2500/35000 2200/29755 2000/26426 1800/23233 1600/20164 1400/17226 1200/14408 1000/11713 800/9140**

CPU 当前频率（使用守护进程 **powerd(8)** 或 **powerdxx(8)** 时会变化）。

**dev.cpu.0.freq: 800**

**hw.acpi.thermal.tz0.temperature** 也会显示当前热区温度。


```sh
% sysctl hw.acpi.thermal.tz0.temperature
hw.acpi.thermal.tz0.temperature: 49.1C
```

要检查你有多少颗 CPU 核心，可以使用以下命令。

```sh
% grep FreeBSD/SMP /var/run/dmesg.boot
FreeBSD/SMP: Multiprocessor System Detected: 2 CPUs
FreeBSD/SMP: 1 package(s) x 2 core(s)

% sysctl kern.smp.cpus
kern.smp.cpus: 2
```

如果我的说明不够直观，你还能用 **sysctl(8)** 命令的参数 **-d**，如下所示。

```sh
% sysctl -d dev.cpu.0.freq
dev.cpu.0.freq: Current CPU frequency
```

### lscpu(1)

还有个第三方工具 **lscpu(8)**，可以显示你的 CPU 特性和型号。你需要从软件包中安装它。


```sh
# pkg install lscpu
```

要使 **lscpu(8)** 正常工作，需要加载内核模块 **cpuctl(4)**。

下面是我的双核 CPU 使用 **lscpu(8)** 的显示效果。

```sh
# kldload cpuctl
# lscpu
Architecture:            amd64
Byte Order:              Little Endian
Total CPU(s):            2
Thread(s) per core:      2
Core(s) per socket:      2
Socket(s):               0
Vendor:                  GenuineIntel
CPU family:              6
Model:                   42
Model name:              Intel(R) Core(TM) i5-2520M CPU @ 2.50GHz
Stepping:                7
L1d cache:               32K
L1i cache:               32K
L2 cache:                256K
L3 cache:                3M
Flags:                   fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 cflsh ds acpi mmx fxsr sse sse2 ss htt tm pbe sse3 pclmulqdq dtes64 monitor ds_cpl vmx smx est tm2 ssse3 cx16 xtpr pdcm pcid sse4_1 sse4_2 x2apic popcnt tsc_deadline aes xsave osxsave avx syscall nx rdtscp lm lahf_lm
```

### dmesg(8)

此外，**dmesg(8)** 命令（或长时间运行后查看 **/var/run/dmesg.boot** 文件）也可以显示你的 CPU 型号和特性信息。

```sh
% grep CPU /var/run/dmesg.boot
CPU: Intel(R) Core(TM) i5-2520M CPU @ 2.50GHz (2491.97-MHz K8-class CPU)
FreeBSD/SMP: Multiprocessor System Detected: 2 CPUs
cpu0:  on acpi0
coretemp0:  on cpu0
```

## CPU 调频

对于 CPU 调频功能，你可以使用 FreeBSD 基本系统提供的守护进程 **powerd(8)** ，或者使用来自 FreeBSD Ports 或软件包的 **powerdxx(8)**。守护进程 **powerdxx(8)** 旨在更好地调频多核系统，在系统负载适中时不会将所有核心都调到高状态，但有些人可能更喜欢那种方法，以便在执行所有操作时都拥有全部性能，而在闲置时节省功耗。因此，**powerd(8)** 并不比 **powerdxx(8)** 更好，反之亦然。它们只是不同的实现，为你的需求提供了更多选择。

无论选择何者，都必须在文件 **/etc/rc.conf** 中进行配置。

### powerd(8)

以下是守护进程 **powerd(8)** 的选项。

```sh
powerd_enable=YES
powerd_flags="-n adaptive -a hiadaptive -b adaptive -m 800 -M 1600"
```

**-n** 选项用于未知状态——当 **powerd(8)** 无法确定当前是使用电源还是电池供电时使用。**-a** 用于电源，**-b** 用于电池供电。**adaptive** 设置较“温和”，更有利于延长电池续航时间。**hiadaptive** 设置更激进，适合在电源供电时使用。**-m** 选项设置最低 CPU 频率，**-M** 设置最大 CPU 频率，单位均为 MHz。更多细节请参见 **powerd(8)** 手册页。

### powerdxx(8)

首先，你需要安装它。

```sh
# pkg install powerdxx
```

然后，它的选项与守护进程 **powerd(8)** 的选项完全相同。

```sh
powerdxx_enable=YES
powerdxx_flags="-n adaptive -a hiadaptive -b adaptive -m 800 -M 1600"
```

请查看上文 **powerdxx(8)** 部分获取标志/参数说明。

十年前，FreeBSD 上的 CPU 调频不像现在这么“简单”，你可以看看我 2008 年的老文章 [HOWTO: FreeBSD CPU Scaling and Power Saving](https://forums.freebsd.org/threads/howto-freebsd-cpu-scaling-and-power-saving.172/)。

### C 状态

可以在 **/etc/rc.conf** 文件中用以下选项配置 C 状态。

* **performance_cx_lowest**
* **economy_cx_lowest**

**economy_cx_lowest** 参数用于电池供电时，**performance_cx_lowest** 参数用于电源供电时。两者都通过 **rc(8)** 子系统使用的 **/etc/rc.d/power_profile** 脚本进行设置。该脚本会设置 **hw.acpi.cpu.cx_lowest** 参数，从而控制所有 **dev.cpu.*.cx_lowest** 的值。当你连接或断开电源时，也可以在 **/var/log/messages** 文件中跟踪这些变化。

```sh
% tail -f /var/log/messages
Nov 28 13:14:42 t420s power_profile[48231]: changed to 'economy'
Nov 28 13:14:46 t420s power_profile[56835]: changed to 'performance'
```

通常我只是使用这些值。

```sh
performance_cx_lowest=C1
economy_cx_lowest=Cmax
```

上述设置对于大多数系统来说通常已经足够。要检查你的 CPU 支持哪些 C 状态，请看看 **dev.cpu.0.cx_supported** 的值。

```sh
% sysctl dev.cpu.0.cx_supported
dev.cpu.0.cx_supported: C1/1/1 C2/3/104
```

我的 CPU 仅支持 C1 和 C2，但你的 CPU 可能支持更多。我记得曾经使用一台老旧的 Core 2 Duo 笔记本时，从 C1（运行）状态返回到 C2（休眠）状态会有相当“明显”的延迟，因此需要如下设置。此时不使用参数 **performance_cx_lowest** 和 **economy_cx_lowest**。你可以将第一个核心设置为 C1，其他所有核心设置为 C2。这样即使在电池供电时，你的系统也能完全响应，而其他核心则可以休眠以节省能源。

例如，如果你有 4 个核心，并且最深支持的 C 状态为 C3，那么你可以将如下内容添加到 **/etc/sysctl.conf** 文件中。

```sh
% grep cx_lowest /etc/sysctl.conf
dev.cpu.0.cx_lowest=C1
dev.cpu.1.cx_lowest=C3
dev.cpu.2.cx_lowest=C3
dev.cpu.3.cx_lowest=C3
```

### CPU 睿频

启用睿频模式有两种方式。一种是通过设置守护进程 **powerd(8)** 或 **powerdxx(8)**，将最大频率设置高于 CPU 标称速度。例如，如果你的 CPU 描述为双核 2.3 GHz，则可以使用参数 **-M** 将最大速度设置为 **4000**（即 4 GHz）。如果你不使用 CPU 调频守护进程，则可以使用参数 **dev.cpu.0.freq**，将其设置为 **dev.cpu.0.freq_levels** 中的最高（第一个）值。

下面是我系统上支持的 CPU 频率级别。

```sh
% sysctl dev.cpu.0.freq_levels 
dev.cpu.0.freq_levels: 2501/35000 2500/35000 2200/29755 2000/26426 1800/23233 1600/20164 1400/17226 1200/14408 1000/11713 800/9140
```

最高值（左侧）为 **2501/35000**，因此我需要将 **dev.cpu.0.freq** 参数设置为该值以启用睿频模式。你只需要使用“频率”部分的值，因为如果连同功耗描述一起设置，会导致失败。

```sh
# sysctl dev.cpu.0.freq=2501/35000
sysctl: invalid integer '2501/35000'
```

如下所示为正确的使用方法。


```sh
# sysctl dev.cpu.0.freq=2501
dev.cpu.0.freq: 800 -> 2501
```

## USB 设备

要列出已连接的 USB 设备，请使用工具 **usbconfig(8)** 。


```sh
% usbconfig
ugen1.1:  at usbus1, cfg=0 md=HOST spd=SUPER (5.0Gbps) pwr=SAVE (0mA)
ugen2.1:  at usbus2, cfg=0 md=HOST spd=HIGH (480Mbps) pwr=SAVE (0mA)
ugen0.1:  at usbus0, cfg=0 md=HOST spd=HIGH (480Mbps) pwr=SAVE (0mA)
ugen2.2:  at usbus2, cfg=0 md=HOST spd=HIGH (480Mbps) pwr=SAVE (0mA)
ugen0.2:  at usbus0, cfg=0 md=HOST spd=HIGH (480Mbps) pwr=SAVE (0mA)
ugen0.3:  at usbus0, cfg=0 md=HOST spd=FULL (12Mbps) pwr=ON (100mA)
ugen2.3:  at usbus2, cfg=0 md=HOST spd=HIGH (480Mbps) pwr=SAVE (0mA)
```

你会看到 **pwr** 参数（即 `power`，电源的缩写）显示当前电源设置，可以为：

* **ON**
* **OFF**
* **SAVE**

要为 **ugen1.1** 设备设置新的 USB 电源选项，也可以使用 **usbconfig(8)** 工具，并使用参数 **power_save**，方法如下。

```sh
# usbconfig -u 1 -a 1 power_save
```

FreeBSD 的 USB 电源管理没有专用的配置文件，因此我们将设置放入通用的 **/etc/rc.local** 文件中，该文件在 **rc(8)** 子系统管理的启动过程结束时运行。下面是添加的内容，唯一的例外是我的无线鼠标 “**Lenovo USB Receiver 联想 USB 接收器**”。

```sh
% grep -A 10 POWER /etc/rc.local
# POWER SAVE USB DEVICES
usbconfig \
  | grep -v 'Lenovo USB Receiver' \
  | grep -v 'Unifying Receiver Logitech' \
  | awk '{print $1}' \
  | sed 's|ugen||'g \
  | tr -d : \
  | awk -F '.' '{print $1 " " $2 }' \
  | while read U A
    do
      usbconfig -u ${U} -a ${A} power_save 2> /dev/null
    done
```

对于鼠标和触控设备，不保存电源是个好主意，因为每次使用时都需要等待约一秒钟，这会很烦人。我使用一个 for 循环为除无线 USB 鼠标（识别为 “**Lenovo USB Receiver**” 设备）之外的所有 USB 设备设置节能模式。

## SATA/AHCI 电源管理

FreeBSD 通过 **acpich(4)** 驱动提供 AHCI 通道电源管理。这些电源管理设置可以在启动时通过 **/boot/loader.conf** 文件中的 **hint.ahcich.*.pm_level** 参数进行配置。我为多达 8 个通道进行了配置，尽管我实际上只有三个通道。

```sh
% grep ahcich /var/run/dmesg.boot
ahcich0:  at channel 0 on ahci0
ahcich1:  at channel 1 on ahci0
ahcich4:  at channel 4 on ahci0
ada0 at ahcich0 bus 0 scbus0 target 0 lun 0
```

这是因为对不存在的设备进行设置是无害的，不会显示任何错误信息，而且你也不必为不同系统使用不同设置，从而节省时间。下面是 **ahci(4)** 手册页中关于 **hint.ahcich.*.pm_level** 的说明。

```sh
  hint.ahcich.X.pm_level

    controls SATA interface Power Management for the specified channel,
    allowing some power to be saved at the cost of additional command latency.

    Some controllers, such as ICH8, do not implement modes 2 and 3 with NCQ
    used. Because of artificial entering latency, performance degradation in
    modes 4 and 5 is much smaller then in modes 2 and 3.

控制指定通道的 SATA 接口电源管理，可以在牺牲额外命令延迟的情况下节省一些电量。
一些控制器（例如 ICH8）在用 NCQ 时不支持模式 2 和 3。
由于人工进入延迟，模式 4 和 5 的性能下降远小于模式 2 和 3。

```

可能的电源管理选项如下：

* **0** – 禁用接口电源管理（默认）
* **1** – 允许设备主动发起 PM 状态改变，主机被动
* **2** – 每次端口空闲时，主机发起 PARTIAL PM 状态转换
* **3** – 每次端口空闲时，主机发起 SLUMBER PM 状态转换
* **4** – 端口空闲 1 毫秒后，驱动发起 PARTIAL PM 状态转换
* **5** – 端口空闲 125 毫秒后，驱动发起 SLUMBER PM 状态转换

下面是我在 **/boot/loader.conf** 文件中的设置。


```sh
# AHCI POWER MANAGEMENT FOR EVERY USED CHANNEL (ahcich 0-7)
  hint.ahcich.0.pm_level=5
  hint.ahcich.1.pm_level=5
  hint.ahcich.2.pm_level=5
  hint.ahcich.3.pm_level=5
  hint.ahcich.4.pm_level=5
  hint.ahcich.5.pm_level=5
  hint.ahcich.6.pm_level=5
  hint.ahcich.7.pm_level=5
```

## 无驱动的设备

FreeBSD 提供了对未附加驱动的设备不供电的节能选项。该选项称为 **hw.pci.do_power_nodriver**，可以在 **/boot/loader.conf** 文件中设置。下面是 **pci(4)** 手册页中的说明。

```sh
  hw.pci.do_power_nodriver (Defaults to 0)

    Place devices into a low power state (D3) when
    a suitable device driver is not found.

当未找到合适的设备驱动时，将设备置于低功耗状态（D3）。
```

它可以设置为以下值之一：

* **0** – 所有设备保持完全供电（默认）。
* **1** – 类似于 ‘2’，但存储控制器仍保持供电。
* **2** – 关闭大多数设备电源（显示器/内存/外设不关闭）。
* **3** – 关闭所有没有驱动程序的 PCI 设备电源。

下面是我在 **/boot/loader.conf** 文件中的设置。

```sh
# POWER OFF DEVICES WITHOUT ATTACHED DRIVER
  hw.pci.do_power_nodriver=3
```

**pciconf(8)** 工具可以显示系统中的设备以及附加到它们的驱动程序。如果设备没有附加驱动，你会看到 **none*@**，例如下面的 **none0@**。你还可以查看大多数驱动的手册页，例如 **em(4)** 查看 **em0** 设备，或 **xhci(4)** 查看 **xhci0** 设备。

```sh
% pciconf -l
hostb0@pci0:0:0:0:      class=0x060000 card=0x21d217aa chip=0x01048086 rev=0x09 hdr=0x00
vgapci0@pci0:0:2:0:     class=0x030000 card=0x21d217aa chip=0x01268086 rev=0x09 hdr=0x00
none0@pci0:0:22:0:      class=0x078000 card=0x21d217aa chip=0x1c3a8086 rev=0x04 hdr=0x00
em0@pci0:0:25:0:        class=0x020000 card=0x21ce17aa chip=0x15028086 rev=0x04 hdr=0x00
ehci0@pci0:0:26:0:      class=0x0c0320 card=0x21d217aa chip=0x1c2d8086 rev=0x04 hdr=0x00
hdac0@pci0:0:27:0:      class=0x040300 card=0x21d217aa chip=0x1c208086 rev=0x04 hdr=0x00
pcib1@pci0:0:28:0:      class=0x060400 card=0x21d217aa chip=0x1c108086 rev=0xb4 hdr=0x01
pcib2@pci0:0:28:1:      class=0x060400 card=0x21d217aa chip=0x1c128086 rev=0xb4 hdr=0x01
pcib3@pci0:0:28:3:      class=0x060400 card=0x21d217aa chip=0x1c168086 rev=0xb4 hdr=0x01
pcib4@pci0:0:28:4:      class=0x060400 card=0x21d217aa chip=0x1c188086 rev=0xb4 hdr=0x01
ehci1@pci0:0:29:0:      class=0x0c0320 card=0x21d217aa chip=0x1c268086 rev=0x04 hdr=0x00
isab0@pci0:0:31:0:      class=0x060100 card=0x21d217aa chip=0x1c4f8086 rev=0x04 hdr=0x00
ahci0@pci0:0:31:2:      class=0x010601 card=0x21d217aa chip=0x1c038086 rev=0x04 hdr=0x00
ichsmb0@pci0:0:31:3:    class=0x0c0500 card=0x21d217aa chip=0x1c228086 rev=0x04 hdr=0x00
iwn0@pci0:3:0:0:        class=0x028000 card=0x11118086 chip=0x42388086 rev=0x3e hdr=0x00
sdhci_pci0@pci0:5:0:0:  class=0x088000 card=0x21d217aa chip=0xe8221180 rev=0x07 hdr=0x00
xhci0@pci0:13:0:0:      class=0x0c0330 card=0x01941033 chip=0x01941033 rev=0x04 hdr=0x00
```

你也可以使用参数 **-v** 获取更详细的信息。

```sh
% pciconf -l -v
(...)
xhci0@pci0:13:0:0:      class=0x0c0330 card=0x01941033 chip=0x01941033 rev=0x04 hdr=0x00
    vendor     = 'NEC Corporation'
    device     = 'uPD720200 USB 3.0 Host Controller'
    class      = serial bus
    subclass   = USB
```

### Nvidia Optimus

如果由于某种原因你的 BIOS/UEFI 固件无法禁用 Nvidia 独显，你可以使用此脚本将其禁用，从而避免消耗系统电源。该操作需要内核模块 **acpi_call(4)**，该模块由软件包 **acpi_call** 提供。

```sh
# mkdir /root/bin
# cd /root/bin
# fetch https://people.freebsd.org/~xmj/turn_off_gpu.sh
# pkg install acpi_call
# kldload acpi_call
# chmod +x /root/bin/turn_off_gpu.sh
# /root/bin/turn_off_gpu.sh
```

你可以在 **/etc/rc.local** 文件中，将其添加到 USB 节能选项之后，使用如下条目。

```sh
# DISABLE NVIDIA CARD
  /root/bin/turn_off_gpu.sh
```

它会将有效的 ACPI 调用存储在 **/root/.gpu_method** 文件中，并在每次启动时执行。

## 挂起与恢复

挂起/恢复机制的最大敌人是硬件 BIOS/UEFI 固件中的错误。例如，有时禁用蓝牙会有所帮助——这是 ThinkPad T420s 中的一个方案。要检查系统支持哪些挂起模式，请查看 **sysctl(8)** 中的 **hw.acpi.supported_sleep_state**。

```sh
% sysctl hw.acpi.supported_sleep_state
hw.acpi.supported_sleep_state: S3 S4 S5
```

要进入 ACPI S3 睡眠状态（挂起），你可以使用 **acpiconf(8)** 工具或 **zzz(8)** 工具。

```sh
# zzz
```

……或者使用 **acpiconf(8)** 工具。

```sh
# acpiconf -s 3
```

这与手册页 **zzz(8)** 中的说明完全相同。

你还可以设置 **sysctl(8)** 值，使每次关闭笔记本盖时系统进入睡眠状态。为此，将 **hw.acpi.lid_switch_state=S3** 添加到 **/etc/sysctl.conf** 文件中。无论是通过命令还是关闭盖子让硬件进入睡眠状态，打开盖子后笔记本都会恢复。当然，如果在执行 **zzz(8)** 命令后没有关闭盖子，你需要关闭并重新打开盖子，或按下电源按钮以恢复。当然，你也可以对桌面或甚至备份服务器执行挂起/恢复操作，这并不仅限于笔记本。

此外，不同厂商的 ACPI 子系统还有专用内核模块，如下：

* **/boot/kernel/acpi_asus_wmi.ko**
* **/boot/kernel/acpi_asus.ko**
* **/boot/kernel/acpi_dock.ko**
* **/boot/kernel/acpi_fujitsu.ko**
* **/boot/kernel/acpi_hp.ko**
* **/boot/kernel/acpi_ibm.ko**
* **/boot/kernel/acpi_panasonic.ko**
* **/boot/kernel/acpi_sony.ko**
* **/boot/kernel/acpi_toshiba.ko**
* **/boot/kernel/acpi_video.ko**
* **/boot/kernel/acpi_wmi.ko**

例如，如果你使用的是 IBM/联想 ThinkPad，则需使用内核模块 **acpi_ibm.ko**。

```sh
# kldload acpi_ibm
```

加载所需模块后，你会得到新的 **sysctl(8)** 值供使用，例如与风扇速度、键盘背光或屏幕亮度相关的值。下面是在加载 **acpi_ibm(4)** 内核模块后，**sysctl(8)** 中新增的 **dev.acpi_ibm** 部分。

```sh
% sysctl dev.acpi_ibm
dev.acpi_ibm.0.handlerevents: NONE
dev.acpi_ibm.0.mic_led: 0
dev.acpi_ibm.0.fan: 0
dev.acpi_ibm.0.fan_level: 0
dev.acpi_ibm.0.fan_speed: 0
dev.acpi_ibm.0.wlan: 1
dev.acpi_ibm.0.bluetooth: 0
dev.acpi_ibm.0.thinklight: 0
dev.acpi_ibm.0.mute: 0
dev.acpi_ibm.0.volume: 0
dev.acpi_ibm.0.lcd_brightness: 0
dev.acpi_ibm.0.hotkey: 1425
dev.acpi_ibm.0.eventmask: 134217727
dev.acpi_ibm.0.events: 1
dev.acpi_ibm.0.availmask: 134217727
dev.acpi_ibm.0.initialmask: 2060
dev.acpi_ibm.0.%parent: acpi0
dev.acpi_ibm.0.%pnpinfo: _HID=LEN0068 _UID=0
dev.acpi_ibm.0.%location: handle=\_SB_.PCI0.LPC_.EC__.HKEY
dev.acpi_ibm.0.%driver: acpi_ibm
dev.acpi_ibm.0.%desc: IBM ThinkPad ACPI Extras
dev.acpi_ibm.%parent:
```

这里是一些更有趣的项的说明。

控制麦克风静音按钮上的 LED 灯。

**dev.acpi_ibm.0.mic_led**

选择是否管理 CPU 风扇（**0**）或使用厂商默认设置（**1**）。

**dev.acpi_ibm.0.fan**

如果启用 CPU 风扇，设置其转速。

**dev.acpi_ibm.0.fan_level**

显示 CPU 风扇转速（单位 RPM）。

**dev.acpi_ibm.0.fan_speed**

启用/禁用 WiFi（前提是在 BIOS 中启用）。

**dev.acpi_ibm.0.wlan**

启用/禁用蓝牙（前提是在 BIOS 中启用）。

**dev.acpi_ibm.0.bluetooth**

启用/禁用 ThinkLight。

**dev.acpi_ibm.0.thinklight**

静音/取消静音扬声器。

**dev.acpi_ibm.0.mute**

扬声器音量。

**dev.acpi_ibm.0.volume**

屏幕亮度。

**dev.acpi_ibm.0.lcd_brightness**

在大多数情况下，你可能无需直接使用这些参数，因为通常会使用厂商定义的键盘快捷键（可能需要 **Fn** 键）或厂商特定的专用按键。但有时你可能希望创建或使用自己的设置，或者需要自定义快捷键，或者想根据 CPU 温度以不同于厂商预设的方式控制风扇转速。这时，这些专用的 ACPI 内核模块就非常有用。

例如，我最近觉得我的 CPU 风扇声音似乎有些大，所以创建了基于 **cron(8)** 的自定义 **acpi-thinkpad-fan.sh** 脚本，在 CPU 温度足够低时使用较低或更安静的风扇转速。简单来说，它的工作逻辑是：CPU 温度低于 50°C 时关闭风扇，温度在 50°C 到 60°C 之间时设置为级别 ‘1’，温度超过 60°C 时设置为级别 ‘3’。


```sh
#! /bin/sh

if ! kldstat | grep -q acpi_ibm.ko
then
  doas kldload acpi_ibm
fi

doas sysctl dev.acpi_ibm.0.fan=0 1> /dev/null 

TEMP=$( sysctl -n hw.acpi.thermal.tz0.temperature | awk -F'.' '{print $1}' )

if [ ${TEMP} -lt 50 ]
then
  doas sysctl dev.acpi_ibm.0.fan_level=0 1> /dev/null
  exit 0
fi

if [ ${TEMP} -lt 60 ]
then
  doas sysctl dev.acpi_ibm.0.fan_level=1 1> /dev/null
  exit 0
fi

if [ ${TEMP} -ge 60 ]
then
  doas sysctl dev.acpi_ibm.0.fan_level=3 1> /dev/null
  exit 0
fi
```

……下面是它在 **crontab(5)** 中的条目：

```sh
% crontab -l
# ACPI/IBM/FAN
* * * * * ~/scripts/acpi-thinkpad-fan.sh
```

## 网卡

如果驱动支持节能功能，**ifconfig(8)** 也要相应选项，称为 **powersave**，用法如下：

```
# ifconfig wlan0 powersave
```

我在自己的 **network.sh** 网络管理脚本中使用了它，脚本在文章 [FreeBSD Network Management with network.sh](https://vermaden.wordpress.com/2018/03/24/freebsd-network-management-with-network-sh-script/) 中有详细介绍。

## 厂商工具

FreeBSD 上也有一些厂商提供的工具，例如 **powermon(8)**。请注意，它需要内核模块 **cpuctl(4)** 才能工作。

```sh
# pkg install powermon
# kldload cpuctl
# powermon
                  Intel(R) Core(TM) i5-2520M CPU @ 2.50GHz
                      (Arch: Sandy Bridge, Limit: 44W)



   5.11W [=======>                                                           ]



 Package:           Uncore:             x86 Cores:          GPU:
 Current: 5.11W     Current: 3.17W      Current: 1.73W      Current: 0.21W
 Total: 98.33J      Total: 60.86J       Total: 33.49J       Total: 3.98J
```
## DTrace

动态追踪框架（像 ZFS 一样由 Solaris/Illumos 引入 FreeBSD）在延长电池使用时间方面也可能是款有用的工具。

首先安装软件包 **dtrace-toolkit**：

```
# pkg install dtrace-toolkit
```

系统停止节能或唤醒 CPU，通常是因为有任务需要执行。你大多数情况下会使用 **ps(1)** 或 **top(1)** 来查看运行情况，但这些工具无法显示具体启动了哪些命令或命令的执行频率。这时 DTrace 就派上用场了。

我们将使用软件包 **dtrace-toolkit** 中的 **/usr/share/dtrace/toolkit/execsnoop** 脚本。它会打印系统中执行的每一个命令及其所有参数。当没有命令执行时，它将保持静默，请注意。

下面是我在更新 **dzen2** 工具栏时的示例输出。

```sh
# /usr/local/share/dtrace-toolkit/execsnoop 
  UID    PID   PPID ARGS
 1000  97748  97509 /usr/local/bin/zsh -c ~/scripts/dzen2-update.sh > ~/.dzen2-fifo
 1000  97748      1 /bin/sh /home/vermaden/scripts/dzen2-update.sh
 1000  99157  97748 sysctl -n kern.smp.cpus
 1000    311  97748 ps ax -o %cpu,rss,command -c
 1000   3118   1521 awk -v SMP=200 /\ idle$/ {printf("%.1f%%",SMP-$1)}
 1000   4462  97748 date +%Y/%m/%d/%a/%H:%M
 1000   4801  97748 sysctl -n dev.cpu.0.freq
 1000   6009  97748 sysctl -n hw.acpi.thermal.tz0.temperature
 1000   6728  97748 sysctl -n vm.stats.vm.v_inactive_count
 1000   7043  97748 sysctl -n vm.stats.vm.v_free_count
 1000   7482  97748 sysctl -n vm.stats.vm.v_cache_count
 1000  10363   8568 bc -l
 1000  10863  10363 dc -x
 1000  13143   7773 grep --color -q ^\.
 1000  13798  97748 /bin/sh /home/vermaden/scripts/__conky_if_ip.sh
 1000  15089  14235 ifconfig -u
 1000  16439  14235 grep -v 127.0.0.1
 1000  17738  14235 grep -c inet 
 1000  19069  18612 ifconfig -l -u
 1000  19927  18612 sed s/lo0//g
 1000  20772  13798 ifconfig wlan0
 1000  23388  21410 grep ssid
 1000  24588  13798 grep -q "
 1000  25965  25282 awk /ssid/ {print $2}
 1000  27917  27217 awk /inet / {print $2}
 1000  29941  97748 /bin/sh /home/vermaden/scripts/__conky_if_gw.sh
 1000  32808  31412 route -n -4 -v get default
 1000  34012  31412 awk END{print $2}
 1000  34895  97748 /bin/sh /home/vermaden/scripts/__conky_if_dns.sh
 1000  36118  34895 awk /^nameserver/ {print $2; exit} /etc/resolv.conf
 1000  37628  97748 /bin/sh /home/vermaden/scripts/__conky_if_ping.sh dzen2
 1000  38829  37628 ping -c 1 -s 0 -t 1 -q 9.9.9.9
 1000  42079  41566 mixer -s vol
 1000  42177  41566 awk -F : {printf("%s",$2)}
 1000  44434  43254 zfs list -H -d 0 -o name,avail
 1000  45866  43254 awk {printf("%s/%s ",$1,$2)}
 1000  47004  97748 /bin/sh /home/vermaden/scripts/__conky_battery_separate.sh dzen2
 1000  48282  47004 sysctl -n hw.acpi.battery.units
 1000  49494  47004 sysctl -n hw.acpi.battery.life
 1000  49948  47004 sysctl -n hw.acpi.acline
 1000  52073  51441 acpiconf -i 0
 1000  53055  51441 awk /^State:/ {print $2}
 1000  53981  53186 acpiconf -i 0
 1000  55354  53186 awk /^Remaining capacity:/ {print $3}
 1000  55968  55631 acpiconf -i 1
 1000  57187  55631 awk /^State:/ {print $2}
 1000  58405  57471 acpiconf -i 1
 1000  59201  57471 awk /^Remaining capacity:/ {print $3}
 1000  60961  59252 bsdgrep -v -E (COMMAND|idle)$
 1000  63534  59252 head -3
 1000  62194  59252 sort -r -n
 1000  64629  59252 awk {printf("%s/%d%%/%.1fGB ",$3,$1,$2/1024/1024)}
 1000  64634  93198 tail -1 /home/vermaden/.dzen2-fifo
```

屏幕顶部的信息更新会启动许多进程。这就是为什么我只每 5 分钟刷新一次 **dzen2** 信息，如果我想获得当前时刻的精确信息和系统状态，我只需点击 **dzen2** 工具栏，它就会执行所有这些命令并刷新自己。

通过这种方式使用 DTrace，你可以知道是否有不必要的进程消耗了宝贵的电池时间。你可以在我的文章 [FreeBSD Desktop – Part 13 – Configuration – Dzen2](https://vermaden.wordpress.com/2018/07/05/freebsd-desktop-part-13-configuration-dzen2/) 中找到相关 **dzen2** 配置。

## 其他

### ZFS

默认情况下，ZFS 每 5 秒提交一次事务组，这对于 **vfs.zfs.txg.timeout** 参数是一个良好的默认设置。如果需要，你可以稍微增加该值，例如设为 10。我之所以提到这个参数，是因为很多指南出于性能原因建议将其设置为 1，但请记住，将其设置为 1 会阻止你的磁盘（和 CPU）进入休眠，从而消耗更多电池寿命。

如果你想调整 **vfs.zfs.txg.timeout** 值，可以在 **/boot/loader.conf** 文件中设置。

### 应用程序

延长电池使用时间时，所使用的应用程序也至关重要。例如，Thunar 比 Caja 或 Nautilus 占用更少的 CPU 时间。Geany 文本编辑器比 Scite 或 Gedit 占用更少的 CPU 资源和内存，即便是 GVim 也会占用更多资源。更不用说基于自定义 Openbox/Fluxbox/**${YOUR_FAVORITE_WM}** 的窗口管理器设置，其 CPU 占用远低于整个 Gnome 或 Mate 环境。

# 硬件

有时可以通过硬件本身获得更多电池时间。例如，当你为笔记本购买新的固态硬盘时，可以选择速度不是最快但最节能的型号。你可能感受不到性能差异，但会明显获得更长的电池使用时间。

大多数内存模块的电压为 1.5V，但你的笔记本可能支持 LPDDR 模块（1.35V），从而延长电池续航。此外，每根内存条大约消耗 0.5–1.0W 功率，因此使用单根 8 GB 内存条比使用两根 4 GB 内存条能拥有更多电池时间。当然这也有性能上的缺点，因为单条内存无法使用双通道技术，会限制内存速度。一些笔记本甚至有四条内存槽（例如 ThinkPad W520），因此为了更长的电池寿命，可以选择使用两根 8 GB 内存条而不是四根 4 GB 内存条，而不会损失性能。

有时你可以将 DVD 光驱替换为内部辅助电池。例如 Dell Latitude D630、ThinkPad T420s 或 ThinkPad T500/W500。有些厂商提供整块贴合笔记本底部的扩展电池，例如 ThinkPad X220、T420/T520/W520 或第一代 ThinkPad X1 笔记本的扩展电池。

希望这些信息能帮助你在 FreeBSD 上延长一些电池使用时间（或至少节省一些电量） :🙂:

## 更新 1 – 显卡节能

如果你安装了 **graphics/drm-kmod** 包，你可能使用的是最新的内核模块 **i915kms.ko** 。

要为 Intel 核显设置最大化电源管理，请将以下内容添加到 **/boot/loader.conf** 文件中：

```sh
# 使用 graphics/drm-kmod 包的 INTEL DRM（新）
# 启动时跳过不必要的模式设置
  compat.linuxkpi.fastboot=1
# 使用信号量进行环间同步
  compat.linuxkpi.semaphores=1
# 启用渲染 C 状态 6 节能
  compat.linuxkpi.enable_rc6=7
# 启用显示 C 状态节能
  compat.linuxkpi.enable_dc=2
# 启用帧缓冲压缩以节能
  compat.linuxkpi.enable_fbc=1
```

过去使用的旧设置如下，但现在已不再使用：

```sh
# 使用 graphics/drm-kmod 包的 INTEL DRM（旧）
  drm.i915.enable_rc6=7
  drm.i915.semaphores=1
  drm.i915.intel_iommu_enabled=1
```

## 更新 2 – AMD CPU 温度

对于 Intel CPU 使用 **coretemp(4)** 内核模块，而 **amdtemp(4)** 内核模块可提供 AMD CPU 的额外温度信息。

## 更新 3 – Suspend/Resume 提示

Suspend/resume 子系统最大的敌人是 BIOS/UEFI 固件中的 BUG。有时禁用 *Bluetooth* 有助于解决问题——例如 *Lenovo ThinkPad T420s* 的做法。在 *Lenovo ThinkPad X240* 上，需要禁用 *TPM*（*Trusted Platform Module*）。

## 更新 4 – Intel Speed Shift

我在较老的 2011 年 *ThinkPad W520* 上运行 FreeBSD，因此这个选项对我来说仍然是个谜——但对于运行更新系统的用户可能会有帮助。

随着 *Intel Skylake*（第六代）CPU 的引入，FreeBSD 能够利用 *Intel Speed Shift* 技术。你可以在 [**hwpstate_intel(4)**](https://man.freebsd.org/hwpstate_intel) 手册页中了解更多信息。要启用该功能，请在 **/boot/loader.conf** 文件中添加以下行：

```ini
machdep.hwpstate_pkg_ctrl=0
```

然后在 **/etc/sysctl.conf** 文件中，你需要为每个 CPU 线程（而非核心）都添加一行，使用你期望的设置：

```ini
dev.hwpstate_intel.N.epp=Y
```

这里的 **N** 代表线程编号。对于双核四线程 CPU，你将有 4 行这样的设置；对于八核 CPU，你将有 8 行。

**Y** 的取值含义如下：

* **0** – 最大性能
* **50** – 平衡（默认）
* **100** – 最大节能

若希望获得最大化节能，可以为所有线程使用 **100**，如下所示：

```ini
dev.hwpstate_intel.0.epp=100
dev.hwpstate_intel.1.epp=100
dev.hwpstate_intel.2.epp=100
dev.hwpstate_intel.3.epp=100
dev.hwpstate_intel.4.epp=100
dev.hwpstate_intel.5.epp=100
dev.hwpstate_intel.6.epp=100
dev.hwpstate_intel.7.epp=100
```

当然，你也可以为单个线程使用全性能，将一个线程设置为平衡，其余线程节能，如下所示：

```ini
dev.hwpstate_intel.0.epp=0
dev.hwpstate_intel.1.epp=50
dev.hwpstate_intel.2.epp=100
dev.hwpstate_intel.3.epp=100
dev.hwpstate_intel.4.epp=100
dev.hwpstate_intel.5.epp=100
dev.hwpstate_intel.6.epp=100
dev.hwpstate_intel.7.epp=100
```

… 当你接入电源而非使用电池时，你可能希望全部线程都使用无限制性能，将所有值设为 **0**。

![](https://vermaden.wordpress.com/wp-content/uploads/2018/11/unlimited-power.gif?w=960)

以下是具体设置示例。

```ini
dev.hwpstate_intel.0.epp=0
dev.hwpstate_intel.1.epp=0
dev.hwpstate_intel.2.epp=0
dev.hwpstate_intel.3.epp=0
dev.hwpstate_intel.4.epp=0
dev.hwpstate_intel.5.epp=0
dev.hwpstate_intel.6.epp=0
dev.hwpstate_intel.7.epp=0
```

理想情况下，当使用电源时，你会使用 *performance*（性能）设置，而在使用电池时则运行 *power save*（节能）模式。你可以通过下面这个简单脚本实现，并让它在 **cron(8)** 守护进程中运行。

```sh
#! /bin/sh

case $( sysctl -n hw.acpi.acline ) in

  (0) # BATTERY
    doas sysctl dev.hwpstate_intel.0.epp=100 1> /dev/null 2> /dev/null
    doas sysctl dev.hwpstate_intel.1.epp=100 1> /dev/null 2> /dev/null
    doas sysctl dev.hwpstate_intel.2.epp=100 1> /dev/null 2> /dev/null
    doas sysctl dev.hwpstate_intel.3.epp=100 1> /dev/null 2> /dev/null
    doas sysctl dev.hwpstate_intel.4.epp=100 1> /dev/null 2> /dev/null
    doas sysctl dev.hwpstate_intel.5.epp=100 1> /dev/null 2> /dev/null
    doas sysctl dev.hwpstate_intel.6.epp=100 1> /dev/null 2> /dev/null
    doas sysctl dev.hwpstate_intel.7.epp=100 1> /dev/null 2> /dev/null
    ;;

  (1) # AC
    doas sysctl dev.hwpstate_intel.0.epp=0   1> /dev/null 2> /dev/null
    doas sysctl dev.hwpstate_intel.1.epp=50  1> /dev/null 2> /dev/null
    doas sysctl dev.hwpstate_intel.2.epp=100 1> /dev/null 2> /dev/null
    doas sysctl dev.hwpstate_intel.3.epp=100 1> /dev/null 2> /dev/null
    doas sysctl dev.hwpstate_intel.4.epp=100 1> /dev/null 2> /dev/null
    doas sysctl dev.hwpstate_intel.5.epp=100 1> /dev/null 2> /dev/null
    doas sysctl dev.hwpstate_intel.6.epp=100 1> /dev/null 2> /dev/null
    doas sysctl dev.hwpstate_intel.7.epp=100 1> /dev/null 2> /dev/null
    ;;

esac
```

… 下面是 **crontab(5)** 的条目。我这里假设你已经具备所需的 **doas(1)** 权限。如果你使用 **sudo(8)**，请相应修改脚本。

```sh
% crontab -l
# INTEL/SPEED/SHIFT
  * * * * * ~/scripts/acpi-intel-speed-shift.sh
```

遗憾的是，我目前没有搭载 Intel Speed Shift CPU 的笔记本，所以不确定为前两个线程分别设置 **0** 和 **50** 是否最佳。也许将所有线程都设为 **100** 以节省更多电力会更好。当我将来拥有这样的笔记本时，会再做更新 🙂


