# FreeBSD 与 HiFi 音频设置：比特完美、均衡器、实时处理 

- 原文：[FreeBSD and hi-fi audio setup: bit-perfect, equalizer, real-time](https://m4c.pl/blog/freebsd-audio-setup-bitperfect-equalizer-realtime/)
- 作者：Marcin Szewczyk-Wilgan

译者注：

- HiFi：High-Fidelity，高保真音效
- bit-perfect：比特完美，指进入 DAC 解码器的数字信号为原始数据。
- filters：滤镜，指可以对原始音频进行处理以产生特殊效果（如添加回声）

完整指南：将 FreeBSD 配置为发烧友音频服务器 —— 设置系统及音频子系统参数、实时操作、比特完美信号处理，以及启用和调节系统图形均衡器（equalizer）和使用 FFmpeg 滤镜实现高质量音频均衡的最佳方法。Linux 用户也能从中获得有用信息，尤其是在配置和个性化 MPD 播放器及滤镜方面。

出于对音乐的热爱，我对数字音频处理解决方案有着 20 多年的业余兴趣。我曾使用 Linux 及其底层工具作为外部数模转换器（以下简称 DAC）的解码器、播放器和传输工具。直到我开始认真关注 FreeBSD。


<img width="1200" height="800" alt="image" src="https://github.com/user-attachments/assets/b2043bf8-da18-4d0e-9390-1b6164ef6eb0" />

## 作为 HiFi 发烧友音频系统的一部分的 FreeBSD

在我看来，使 FreeBSD 优于 Linux 的，正是它那精确追踪音频设备参数、系统内核参数以及对其进行修改的能力。当然，我们这里讨论的仅限于操作系统层面对音频的处理（即在 FreeBSD 中由 OSS/[`sound(4)`](https://man.freebsd.org/cgi/man.cgi?sound%284%29) 驱动完成，Linux 中由 ALSA 完成），并以一种方式配置系统，使得音频数据只在听音设备的硬件层面以 **比特完美** 模式处理，也就是说：在传输到外部 DAC 设备或声卡的过程中，不进行重采样、路由或声道混合。因此，我自然省略了诸如 Jack 服务器、PulseAudio 或 RedHat 旗下令人头疼的 PipeWire 这类额外音频层的使用和配置。当然，软件信号解码器的问题仍存在，后文会详细论述。

使用 FreeBSD 的另一个优势是，它对[实时操作系统](https://en.wikipedia.org/wiki/Real-time_operating_system)（RTOS）和实时程序的支持更好。虽然“更好”并不意味着完美，但在这方面 Linux 远远落后。直到 2024 年，实时 Linux（PREEMPT\_RT）才成为内核及其主线软件的正式一部分。

## 什么是实时系统以及它在音频处理中的重要性

实时操作系统（RTOS）是一类专门设计用来以高精度和高可靠性处理时间敏感任务的操作系统。与通用操作系统不同，实时系统旨在在最严格、最短的时间内响应事件并处理数据。

>RTOS 的关键特征是其可预测的运行，这意味着关键任务能够保证在规定的时间内完成。**人们常误以为实时系统运行更快，实际上并非如此，实时系统关注的是可靠性和可预测性。** 正是这些特性使得 RTOS 广泛应用于时间至关重要的领域，比如工业自动化系统、航天设备、医疗器械，当然也包括音频处理领域。

以实时系统模式运行音频服务器和播放器，能够让我们对处理质量有更强的掌控感和信心。因此，如果我们希望构建一款不妥协的发烧友音频系统，FreeBSD 将是个极佳的选择。在我们的方案中，所指的是作为传输端或立体声播放器的系统，而非专业录音室应用——后者涉及混音声道或与 MIDI 接口交互，使用 RTOS 的目的则有所不同。

## FreeBSD 与 Linux 的音质比较

我进行了大量的听音测试，结果显示，在比特完美模式下配置的 FreeBSD 和 Linux 系统在音频设备性能方面表现相同。然而，FreeBSD 的优势在于它能够精确控制操作系统层面硬件处理流程中的所有参数，并且可以对其进行调整，这赋予了我们几乎极致的掌控和了解能力。

Linux 的音频处理系统 ALSA 大多自动调整所有参数，因此启用接口（或协议）`hw:0,0` 的比特完美模式是否真如我们所期望的那样，仍值得商榷。而在 FreeBSD 中，不仅有直观的 `sound(4)` 驱动中的 `dev.pcm.%d.bitperfect` 参数，更有其它配置选项和完全透明的操作，提供了更多调控自由度。

<img width="1100" height="745" alt="image" src="https://github.com/user-attachments/assets/5c83a823-c738-4204-99f7-e07488dd0e3b" />


## FreeBSD 与 Linux 的音频硬件支持

不过必须指出，FreeBSD 和 Linux 之间有一个非常重要的区别。在 Linux 中，哪怕是在比特完美模式（`hw:0,0`）下，几乎所有内置声卡或 USB DAC 也都能正常工作；但 FreeBSD 并非如此。启用参数 `dev.pcm.%d.bitperfect=1` 有时会引发问题。例如，设备支持的 PCM 格式描述符无法被正确识别（尤其是针对 XMOS 通信芯片），强制设置该参数，包括使用 `dev.pcm.%d.play.vchanformat=s16le:2.0`，都无法达到预期效果。我测试过的 [Presonus Audiobox iOne](https://m4c.pl/blog/presonus-ione-linux/) 系列设备就是如此。此时需要关闭比特完美模式，系统会自动启用对所有数据源以 48 kHz 的固定重采样。但在 Linux 下，这些设备运行正常。

此外，还应提及一个长期存在的问题，特别是针对使用 USB DAC 的硬件配置——播放过程中偶尔出现的干扰声和点击声，大约每十几分钟出现一次。虽然这没有固定规律——某些硬件规格完全正常，而某些则问题频出。此问题值得另写文章详细探讨，尽管尝试了不同的延迟参数、缓冲区调整、PCM 设备调试和各个组件间的干扰分析，至今仍未找到解决方案。

**幸运的是，在大多数情况下，FreeBSD 中仍能按预期配置音频设备。简单来说，大部分参数系统默认已作出合理选择。**

## FreeBSD 与音频子系统配置

接下来介绍 FreeBSD 及其音频子系统的参数配置。本文所述设置基于最新版本 FreeBSD 14.2，自 2024 年以降，经过数年停滞后，FreeBSD 的音频栈迎来了许多新特性。按 FreeBSD 基金会的说法，[相关工作仍在持续推进](https://www.freebsd.org/status/report-2024-01-2024-03/#_audio_stack_improvements)。

音频服务器和播放器由 [MPD](https://www.musicpd.org/)（音乐播放器守护进程，Linux 中命名为 `mpd`，FreeBSD 中为 `musicpd(1)`）承担。MPD 服务器的接口工具包括 `ncmpcpp(1)` 和 `mpc`（FreeBSD 中为 `musicpc(1)`）。

**听音系统规格：**

* **电脑**：HP t540 瘦客户机，AMD Ryzen，4GB DDR4 内存，16GB SSD
* **存储**：Seagate Expansion 2TB USB SSD
* **DAC**：Cambridge Audio DacMagic Plus（新电容线路：松下 FM，ELNA 音频专用；Tomanek 电源），Presonus AudioBox iOne
* **USB 线缆**：WireWorld ULTRAVIOLET 8
* **耳机放大器**：Forum608 IV 双单声道（A 类）
* **RCA 连接线**：Audioquest Goldengate 0.6 米
* **耳机**：Beyerdynamic 990 Pro 250 欧姆，990 Edition 600 欧姆

**最常听的专辑：**

* Iron Maiden – Fear Of The Dark（铁娘子乐队-恐惧黑暗）
* McIntosh Audiophile Test Reference 发烧友测试参考
* Alan Parsons Project – Stereotomy（亚伦派森实验乐团-Stereotomy）
* The WHO – Quadrophenia（谁人乐队-四重人格）
* Depeche Mode – Violator（赶时髦乐队-Violator）
* HD Audiophile Speaker Set-Up 高清发烧音箱设置（192-24）
* TOOL – Lateralus（工具乐队-Lateralus）

### FreeBSD 配置文件

FreeBSD 系统配置的主要且最重要的部分基于三个配置文件：`/boot/loader.conf`、`/etc/sysctl.conf`、`/etc/rc.conf`。以下是在比特完美音频和实时操作系统（RTOS）角色下系统运行相关的配置项。

#### /boot/loader.conf

```ini
sysctlinfo_load="YES" ①
mac_priority_load="YES" ②
#hint.pcm.5.eq=1 ③
```

① 添加模块 `sysctlinfo(4)`，该模块负责与各个内核 sysctl 状态和 MIB 树组件进行扩展的进程通信。此模块非必需，但部分应用程序（例如 `mixertui(8)`）会使用该接口。

② 模块 `mac_priority(4)` 用于建立权限调度规则，使得特权用户可以让工具 [`rtprio(1)`](https://man.freebsd.org/cgi/man.cgi?query=rtprio) 以实时优先级运行进程。

③该参数负责启用 `dev.pcm.5` 设备的图形均衡器（equalizer）（上述规格中指的是 USB DAC 设备）。在比特完美配置中我们省略此项，具体内容详见后文 **FreeBSD 中的图形均衡器** 一节。


#### /etc/sysctl.conf

```ini
kern.timecounter.alloweddeviation=0 ①
hw.usb.uaudio.buffer_ms=2 ②
hw.snd.latency=0 ③
dev.pcm.5.play.vchans=0 ④
dev.pcm.5.bitperfect=1 ⑤
hw.snd.default_unit=5 ⑥
#hw.snd.vpc_0db=80 ⑦
#dev.pcm.5.eq_preamp=-5 ⑧
```

①这是个非常重要的参数，**它表示提高用于同步进程唤醒的软件时钟的精度，从而最小化处理器因进入和退出空闲状态而执行相对高开销操作的次数**。

如果想更深入了解进程唤醒延迟及系统时钟频率对该延迟的影响，推荐阅读 [Paul Herman 的文章](https://www.dragonflybsd.org/presentations/nanosleep/)，文中除了测试结果，还提供了[基准测试源码](https://www.dragonflybsd.org/presentations/nanosleep/wakeup_latency.c)。

为了直观展示参数 `kern.timecounter.alloweddeviation` 对延迟的影响，下面是我在系统上的测试结果：

```sh
# ./wakeup_latency
0.007341
0.014410
0.037370
0.037336
0.037375
0.037345
0.037356
...
# sysctl kern.timecounter.alloweddeviation=0
kern.timecounter.alloweddeviation: 5 -> 0

# ./wakeup_latency
0.000061
0.000095
0.000077
0.000093
0.000096
0.000090
0.000062
...
```

② 驱动参数 `snd_uaudio(4)` 适用于 USB 音频设备（系统在检测到兼容设备时会加载该模块）。该参数定义了处理过程中以毫秒为单位报告的数据时间范围。时间越短，数据延迟越低，但建议根据实际情况进行参数调试和试验。

③ 数据传输延迟，除非软件（例如播放器）因某些原因将此值修改为更合适的数值。

④ 虚拟播放/录音通道数。我们只关注单一音源的播放，因此禁用虚拟通道。此外，启用比特完美模式也需要设置此选项为禁用。

⑤ 启用比特完美模式并禁用虚拟通道后，系统会绕过数字信号处理（DSP）、频率变换/重采样、均衡器等处理。纯净的 PCM 数据流将直接传输到音频终端设备。

⑥ 在上述硬件规格中，音频设备为 USB DAC，系统中显示为 `dev.pcm.5` —— 我们将其设置为默认设备。

⑦ 音频驱动的相对音量级别参数，默认值为 45。这在比特完美模式下影响微乎其微，但在系统图形均衡器（equalizer）配置中则较为重要；在配置和比特完美模式下影响可忽略不计。

⑧与上述参数类似，此为相对音量级别参数，但针对系统图形均衡器的操作（默认 +0.0dB），在比特完美模式下影响可忽略不计。关于参数 `dev.pcm.%d.eq_preamp` 和 `hw.snd.vpc_0db` 的详细含义，请参见 **FreeBSD 中的图形均衡器** 一节。

#### /etc/rc.conf

```ini
musicpd_enable="YES" ①
```

①在我们的配置中，使用 Music Player Daemon（MPD，音乐播放器守护进程）服务器作为播放器——在系统启动时以普通模式启动它。在其余配置中，我们以实时优先级启动 `musicpd(1)`。

## FreeBSD 命令：帮助检查驱动程序、内核参数（sysctl）及音频子系统状态

通过以下命令，可以列出已加载的内核模块，确认我们添加的组件是否存在：

```sh
# kldstat
```

执行结果应类似如下：

```sh
Id Refs Address                Size Name
 1   43 0xffffffff80200000  1f3c6c0 kernel
 2    1 0xffffffff8213d000     3368 sysctlbyname_improved.ko
 3    1 0xffffffff82142000     77d8 cryptodev.ko
 4    1 0xffffffff8214a000     5b18 sysctlinfo.ko
 5    1 0xffffffff82150000     3c48 mac_priority.ko
 6    1 0xffffffff82154000   5da658 zfs.ko
 7    1 0xffffffff83020000     3390 acpi_wmi.ko
 8    1 0xffffffff83024000     4250 ichsmb.ko
 9    1 0xffffffff83029000     2178 smbus.ko
10    1 0xffffffff8302c000     e5b0 snd_uaudio.ko
11    1 0xffffffff8303b000     2a68 mac_ntpd.ko
12    1 0xffffffff8303e000     3560 fdescfs.ko
13    1 0xffffffff83042000    1aec0 ext2fs.ko
```

以下命令将显示计算机上安装的音频设备：

```sh
# cat /dev/sndstat
```

系统检测到的音频设备示例如下：

```sh
Installed devices:
pcm0: <Realtek ALC233 (Analog 2.0+HP/2.0)> (play/rec)
pcm1: <Realtek ALC233 (Analog)> (play/rec)
pcm2: <Intel Braswell (HDMI/DP 8ch)> (play)
pcm3: <Intel Braswell (HDMI/DP 8ch)> (play)
pcm4: <Intel Braswell (HDMI/DP 8ch)> (play)
pcm5: <Cambridge Audio Cambridge Audio USB Audio 2.0> (play) default
No devices installed from userspace.
```

内核参数 `hw.snd.verbose` 默认值为 `0`。将其设置为更高的值，可通过 ioctl 接口获得更多详细信息：

```sh
# sysctl hw.snd.verbose=4
# cat /dev/sndstat
```

接下来的命令会显示音频设备的更多详细信息（列表中仅显示感兴趣的设备）：

```sh
...
pcm5:  on uaudio0 (1p:0v/0r:0v) default
        snddev flags=0x3e7<SIMPLEX,AUTOVCHAN,SOFTPCMVOL,BUSY,MPSAFE,REGISTERED,BITPERFECT,VPC>
        [dsp5.play.0]: spd 44100, fmt 0x00201000, flags 0x2000012c, 0x00000001, pid 1059 (musicpd)
                interrupts 45405, underruns 0, feed 45404, ready 8192
                [b:5648/2824/2|bs:8192/2048/4]
                channel flags=0x2000012c<RUNNING,TRIGGERED,SLEEPING,BUSY,BITPERFECT>
                {userland} -> feeder_root(0x00201000) -> {hardware}
...
```

特定 USB 设备的通信接口状态可通过命令 `usbconfig(8)` 调用（此处物理地址 `ugen0.3` 对应系统中的 `dev.pcm.5`）：

```sh
# usbconfig -d ugen0.3 dump_device_desc
```

以下是该 USB 设备的详细接口数据：

```sh
ugen0.3:  at usbus0, cfg=0 md=HOST spd=HIGH (480Mbps) pwr=ON (2mA)

  bLength = 0x0012
  bDescriptorType = 0x0001
  bcdUSB = 0x0200
  bDeviceClass = 0x00ef  <Miscellaneous device>
  bDeviceSubClass = 0x0002
  bDeviceProtocol = 0x0001
  bMaxPacketSize0 = 0x0040
  idVendor = 0x22e8
  idProduct = 0xdac2
  bcdDevice = 0x0326
  iManufacturer = 0x0001  <Cambridge Audio >
  iProduct = 0x0002  <Cambridge Audio USB Audio 2.0>
  iSerialNumber = 0x0003  <0000>
  bNumConfigurations = 0x0002
```

连接 `sound(4)` 驱动与音频设备 PCM 驱动的 sysctl 接口参数：

```sh
# sysctl hw.snd
```

结果应类似如下：

```sh
hw.snd.maxautovchans: 16
hw.snd.default_unit: 5
hw.snd.default_auto: 0
hw.snd.verbose: 0
hw.snd.vpc_mixer_bypass: 0
0hw.snd.feeder_rate_quality: 1
hw.snd.feeder_rate_round: 25
hw.snd.feeder_rate_max: 2016000
hw.snd.feeder_rate_min: 1
hw.snd.feeder_rate_polyphase_max: 183040
hw.snd.feeder_rate_presets: 100:8:0.85 100:36:0.92 100:164:0.97
hw.snd.feeder_eq_exact_rate: 0
hw.snd.feeder_eq_presets: PEQ:16000,0.2500,62,0.2500:-9,9,1.0:44100,48000,88200,96000,176400,192000
hw.snd.basename_clone: 1
hw.snd.compat_linux_mmap: 0
hw.snd.syncdelay: -1
hw.snd.usefrags: 0
hw.snd.vpc_reset: 0
hw.snd.vpc_0db: 45
hw.snd.vpc_autoreset: 1
hw.snd.timeout: 5
hw.snd.latency_profile: 1
hw.snd.latency: 0
hw.snd.report_soft_matrix: 1
hw.snd.report_soft_formats: 1
```

所选音频设备的状态及驱动数据：

```sh
# sysctl dev.pcm.5
```

以下输出确认了多项信息，其中包括比特完美模式已明确启用（该模式的正确参数为：`dev.pcm.%d.bitperfect=1`、`dev.pcm.%d.play.vchans=0`），且当前处理的数据频率为 44098 MHz：

```sh
dev.pcm.5.feedback_rate: 44098
dev.pcm.5.mode: 3
dev.pcm.5.bitperfect: 1
dev.pcm.5.buffersize: 0
dev.pcm.5.play.vchans: 0
dev.pcm.5.hwvol_mixer: vol
dev.pcm.5.hwvol_step: 5
dev.pcm.5.%iommu:
dev.pcm.5.%parent: uaudio0
dev.pcm.5.%pnpinfo:
dev.pcm.5.%location:
dev.pcm.5.%driver: pcm
dev.pcm.5.%desc: Cambridge Audio Cambridge Audio USB Audio 2.0
```

## 在 FreeBSD 中以实时模式运行 MPD 音乐播放器

在 **FreeBSD 与音频子系统配置** 一节中，我们已准备好系统以实时系统模式运行，即加载了模块 `mac_priority(4)`，并通过调整参数 `kern.timecounter.alloweddeviation` 将进程延迟降至最低。通过命令 `rtprio(1)`，可在系统实时调度器中启动服务器和 MPD。但在此之前，需先关闭系统启动时自动启动的服务 `musicpd(1)`：

```sh
# service musicpd stop
# rtprio 0 musicpd /usr/local/etc/musicpd.conf
```

在 FreeBSD 中，最高实时优先级为 0。优先级编号从 0 到 RTP\_PRIO\_MAX（通常为 31），数值越低优先级越高。

要验证 MPD 服务器是否以实时且正确的优先级运行，可使用命令 `ps` 查看进程列表，并筛选出 `musicpd(1)` 进程：

```sh
# ps -o rtprio -axl | grep [m]usicpd
```

`ps` 命令的输出应类似如下：

```sh
real:0 137 1146 1 0 -52 0 415080 214420 select I<s - 0:06.55 musicpd /usr/local/etc/musicpd.conf
```

另一种通过命令 `top(1)` 检查 `musicpd(1)` 是否正确以实时模式启动的方法：

```sh
# top -p `pgrep -d "," musicpd`
```

```sh
last pid:  1162;  load averages:  0.03,  0.07,  0.04              up 0+12:05:50  21:38:34
50 processes:  2 running, 46 sleeping, 2 waiting
CPU: 50.0% user,  0.0% nice, 50.0% system,  0.0% interrupt,  0.0% idle
Mem: 497M Active, 64M Inact, 998M Wired, 49M Buf, 2267M Free
ARC: 85M Total, 27M MFU, 55M MRU, 514K Header, 1935K Other
     66M Compressed, 153M Uncompressed, 2.32:1 Ratio
Swap: 2048M Total, 2048M Free

  PID USERNAME    THR PRI NICE   SIZE    RES STATE    C   TIME    WCPU COMMAND
 1146 mpd           8 -52   r0   605M   227M select   0   0:12   0.00% musicpd
```

## FreeBSD 中用于调试和排查 USB 音频设备的其他命令

```sh
# sysctl hw.usb.debug=1
# tail -f /var/log/messages
```

```sh
...
Jan 21 21:55:44 freebsd kernel: usbd_pipe_enter: enter
Jan 21 21:55:44 freebsd kernel: usbd_transfer_done: err=USB_ERR_NORMAL_COMPLETION
Jan 21 21:55:44 freebsd kernel: usbd_callback_wrapper_sub: xfer=0xfffffe007c950278 endpoint=0xfffff800038ad968 sts=0 alen=2816, slen=2816, afrm=64, nfrm=64
Jan 21 21:55:44 freebsd kernel: usbd_pipe_start: start
Jan 21 21:55:44 freebsd kernel: usbd_transfer_submit: xfer=0xfffffe007c950278, endpoint=0xfffff800038ad968, nframes=64, dir=write
Jan 21 21:55:44 freebsd kernel: usb_dump_endpoint: endpoint=0xfffff800038ad968 edesc=0xfffff800056d6491 isoc_next=11268 toggle_next=0 bEndpointAddress=0x01
Jan 21 21:55:44 freebsd kernel: usb_dump_queue: endpoint=0xfffff800038ad968 xfer:
Jan 21 21:55:44 freebsd kernel: usbd_pipe_enter: enter
Jan 21 21:55:44 freebsd kernel: usbd_transfer_done: err=USB_ERR_NORMAL_COMPLETION
Jan 21 21:55:44 freebsd kernel: usbd_callback_wrapper_sub: xfer=0xfffffe007c950148 endpoint=0xfffff800038ad968 sts=0 alen=2824, slen=2824, afrm=64, nfrm=64
Jan 21 21:55:44 freebsd kernel: usbd_pipe_start: start
Jan 21 21:55:44 freebsd kernel: usbd_transfer_submit: xfer=0xfffffe007c950148, endpoint=0xfffff800038ad968, nframes=64, dir=write
Jan 21 21:55:44 freebsd kernel: usb_dump_endpoint: endpoint=0xfffff800038ad968 edesc=0xfffff800056d6491 isoc_next=11332 toggle_next=0 bEndpointAddress=0x01
Jan 21 21:55:44 freebsd kernel: usb_dump_queue: endpoint=0xfffff800038ad968 xfer:
...
```

```sh
# sysctl hw.usb.uaudio.debug=1
# tail -f /var/log/messages
```

```sh
...
Jan 21 21:53:43 freebsd kernel: uaudio_chan_play_sync_callback: Value = 0x00058330
Jan 21 21:53:43 freebsd kernel: uaudio_chan_play_sync_callback: Comparing 44099 Hz :: 44100 Hz
Jan 21 21:53:44 freebsd kernel: uaudio_chan_play_sync_callback: Value = 0x00058330
Jan 21 21:53:44 freebsd kernel: uaudio_chan_play_sync_callback: Comparing 44099 Hz :: 44100 Hz
Jan 21 21:53:45 freebsd kernel: uaudio_chan_play_sync_callback: Value = 0x00058330
Jan 21 21:53:45 freebsd kernel: uaudio_chan_play_sync_callback: Comparing 44099 Hz :: 44100 Hz
Jan 21 21:53:46 freebsd kernel: uaudio_chan_play_sync_callback: Value = 0x00058330
Jan 21 21:53:46 freebsd kernel: uaudio_chan_play_sync_callback: Comparing 44099 Hz :: 44100 Hz
Jan 21 21:53:47 freebsd kernel: uaudio_chan_play_sync_callback: Value = 0x00058330
Jan 21 21:53:47 freebsd kernel: uaudio_chan_play_sync_callback: Comparing 44099 Hz :: 44100 Hz
Jan 21 21:53:48 freebsd kernel: uaudio_chan_play_sync_callback: Value = 0x00058330
Jan 21 21:53:48 freebsd kernel: uaudio_chan_play_sync_callback: Comparing 44099 Hz :: 44100 Hz
...



## FreeBSD 中的图形均衡器

FreeBSD 系统的声音驱动（`sound(4)`）中实现了一款简单的软件图形均衡器（equalizer），支持实时调节低音和高音。内核默认编译时，均衡器调节频率为 62 Hz 和 16 kHz，也可以通过编译内核时指定其他参数进行更改。要启用均衡器，需要在文件 `/boot/loader.conf` 中设置参数 `hint.pcm.%d.eq`（启用后需重启系统）：

```sh
#/boot/loader.conf
...
hint.pcm.5.eq=1
```

启用 `hint.pcm.%d.eq` 后，混音器应用中会出现额外的低音和高音调节控件。我们可以使用图形界面应用 `mixertui(1)` 或系统命令 `mixer(1)` 来设置这些参数，例如：

```sh
mixer -f /dev/mixer5 vol=0.75 pcm=0.75 bass=0.82 treble=0.76
```

系统会返回设置的数值及其状态：

```sh
vol.volume: 0.75:0.75 -> 0.75:0.75
pcm.volume: 0.75:0.75 -> 0.75:0.75
bass.volume: 0.82:0.82 -> 0.82:0.82
treble.volume: 0.76:0.76 -> 0.76:0.76
pcm5:mixer:  on uaudio0 (play)
    vol       = 0.75:0.75     pbk
    bass      = 0.82:0.82     pbk
    treble    = 0.76:0.76     pbk
    pcm       = 0.75:0.75     pbk
```

<img width="738" height="410" alt="image" src="https://github.com/user-attachments/assets/136a312d-a4be-486d-a70f-df5f748a8c2d" />


开启并使用图形均衡器参数显然会对音频信号产生干扰。因此，**当均衡器开启时，音频驱动会自动关闭比特完美模式**：

```sh
# sysctl dev.pcm.5 | grep bitperfect
```

结果：

```sh
dev.pcm.5.bitperfect: 0
```

>音频均衡有可能引入一定的失真或音质劣化风险，尤其是在过度使用时。为防止这种情况，FreeBSD 声音驱动提供了两个参数：`hw.snd.vpc_0db` 和 `dev.pcm.%d.eq_preamp`（官方文档尚未详细描述）。这两个参数分别设置声卡驱动和均衡器的相对“零”音量级别，通常能为均衡计算留出更大的余量以避免失真：
>
>* `hw.snd.vpc_0db` —— 默认启用，值为 `45`。增大该参数（即降低音量级别）会为经过均衡校正后的频率处理提供更大的动态范围。
>* `dev.pcm.%d.eq_preamp` —— 默认值为 `0`，范围为 -9 dB 至 +9 dB。例如，设置为 `-5` 表示先将整个音频流整体衰减 5 dB（相对于最大增益），后续的放大均基于该点（0 dB）进行。这样听感音量会降低，但如上所述，可以更好地补偿因均衡放大某些频率产生的失真。
>
>这两个参数建议交替使用，通常配置到文件 `/etc/sysctl.conf`，也可以在运行时直接设置。

示例：

```sh
# sysctl hw.snd.vpc_0db=80
```

```sh
hw.snd.vpc_0db: 45 -> 80
```

```sh
# sysctl dev.pcm.5.eq_preamp=-5
```

```sh
dev.pcm.5.eq_preamp: +0.0dB -> -5.0dB
```

## 高品质均衡器——FFmpeg 滤镜

在发烧友圈里，是否使用音频均衡器一直存在着不同的看法。一方面，纯粹主义者主张尽可能自然、不做任何修改地还原声音；另一方面，实用主义者建议使用图形均衡器，根据个人喜好、房间声学条件或音响设备的缺陷进行调节。

也可以折中，即适度使用均衡器，寻找自然音质与个性化调音之间的平衡点。合适的图形均衡器能在不引入明显失真的情况下提升听感质量。

FFmpeg 项目中的 [libavfilter](https://ffmpeg.org/ffmpeg-filters.html) 库提供了极佳的高级滤镜。我们可以在 MPD 播放器中轻松使用它们，FFmpeg 同时承担信号解码角色。FreeBSD ports 仓库中的 MPD 已内置了 libavfilter 库，而 Debian 12 需要从 `backports` 仓库更新 MPD 以支持该功能。

当然，使用均衡器时无法实现完全的比特完美模式，但在我们的配置中，比特完美信号传递到 FFmpeg 解码器，并通过 libavfilter 送达 DAC 设备或声卡。**这种图形均衡器在音质上远胜于 FreeBSD `sound(4)` 驱动实现的系统均衡器和 Linux 中的 `alsaequal` 插件。**

在 FreeBSD 和 Linux 上启用 FFmpeg 滤镜前，也应先关闭系统均衡器。

## 使用 FFmpeg 滤镜配置 MPD

利用 FFmpeg 及其 libavfilter 库作为 MPD 播放器的图形均衡器，我们主要关注以下滤镜：[bass](https://ffmpeg.org/ffmpeg-filters.html#bass_002c-lowshelf)、[treble](https://ffmpeg.org/ffmpeg-filters.html#treble_002c-highshelf) 和 [anequalizer](https://ffmpeg.org/ffmpeg-filters.html#anequalizer)。这样不仅能在 FreeBSD 实现高品质的软件均衡器，也能在 Linux 中同样使用。此外，FFmpeg 滤镜还适合基于 MPD 的流媒体系统。

启用滤镜时，需要在 MPD 配置文件 `musicpd.conf` 中添加一个新的 `filter {}` 区块，用于定义滤镜（或多个滤镜），并在 `audio_output {}` 区块中添加滤镜。同时确保 `mixer_type` 和 `replay_gain_handler` 参数设置正确，适合比特完美模式。系统配置方面保持之前 **配置 FreeBSD 与音频子系统** 中描述的方案。

### FFmpeg —— bass 与 treble 滤镜示例

示例组合了 `bass` 和 `treble` 滤镜：

```sh
filter {
  plugin "ffmpeg"
  name "equalizer"
  graph "bass=f=62:t=h:w=120:g=10:n=1:r=f64, treble=f=16000:t=h:width=8000:g=2:n=1:r=f64"
}

audio_output {
  type "oss"
  name "OSS"
  mixer_type "hardware"
  replay_gain_handler "none"
  filters "equalizer"
}
```

滤镜说明：

* `bass` —— 在 62 Hz 频率处以 120 Hz 带宽提升 10 个点，使用 64 位数据的浮点精度运算；
* `treble` —— 在 16 kHz 频率处以 8 kHz 带宽提升 3 个点，使用 64 位数据的浮点精度运算。

>这两个滤镜中非常重要的参数是 `n`，它负责对信号进行归一化。简单来说，`n` 会实时自动降低信号振幅（音量），从而扩展计算范围，避免失真和杂音的产生。

基于上述滤波器的频率响应可视化：

<img width="768" height="222" alt="image" src="https://github.com/user-attachments/assets/b900cbcc-342b-48dd-a894-94477074c4fc" />


### FFmpeg – 滤镜 anequalizer

FFmpeg 库中还有一个值得注意的滤镜，即 `anequalizer`。它是一个多通道参数均衡器，基于模拟的切比雪夫（Chebyshev）和巴特沃斯（Butterworth）滤波器（由于通带信号失真较小，建议用于音频应用）。`anequalizer` 滤镜的系数可以根据中心频率、峰值增益、带宽和带宽增益来计算。

上述通过 `anequalizer` 构建的用于 MPD 的 `bass` 和 `treble` 滤波器的特性定义如下：

```sh
filter {
    plugin "ffmpeg"
    name "equalizer"
    graph "anequalizer=c0 f=62 w=120 g=10 t=0|c1 f=62 w=120 g=10 t=0|c0 f=16000 w=8000 g=2 t=0|c1 f=16000 w=8000 g=2 t=0"
}
```

正如你所见，使用滤镜 `anequalizer`，我们几乎可以构建任意图形均衡器的特性。但有一个注意点——它缺少对处理后信号进行归一化的选项，无法消除滤波处理带来的失真。而这种失真在 `g` 参数增益值为 3 时就能听出来。音频驱动的参数，比如 `hw.snd.vpc_0db`，在这里不起作用，因为信号振幅仅在滤波之后才被驱动程序降低。唯一的解决办法是使用 MPD 中的软件音量混合器。设置为 40% 时，即使 `g=10`，也不会听到失真，虽然这并不是获得理想音质的最佳方式。

我们通过设置 MPD 的 `mixer_type` 参数来启用软件音量混合器，配置如下：

```ini
mixer_type "software"
```

### 设计 anequalizer 滤镜的特性

可以通过 [Intona fima 的 DSP 编辑器](https://intona.eu/lpi/index.html) 设计和可视化 `anequalizer` 滤镜特性波形及其参数。

下面这个滤镜定义的等效表达：

```sh
graph "anequalizer=c0 f=62 w=120 g=10 t=0|c1 f=62 w=120 g=10 t=0|c0 f=16000 w=8000 g=2 t=0|c1 f=16000 w=8000 g=2 t=0"
```

在 DSP 应用中对应为：

```sh
peq 62Hz q0.51 10dB
peq 16kHz q2 3dB
```

定义中给出的 `w` 参数所对应的 `q` 参数（[Q 因子](https://en.wikipedia.org/wiki/Q_factor)）由以下关系确定：

<img width="172" height="88" alt="~4WTCG370H5}1%QEL 750 U" src="https://github.com/user-attachments/assets/0d57c3cd-c885-4a08-a758-4c0365a6612d" />


我建议你设计一个符合自己喜好的图形均衡器。

## 总结

本文涵盖了如何配置 FreeBSD 系统及其音频子系统以获得最佳音质，讨论了进程架构的多个方面，包括 FreeBSD 系统的实时调度器、比特完美信号模式，以及如何启用系统均衡器并正确配置以提升音频处理质量。最后，我展示了如何在 MPD 播放器中使用 FFmpeg 滤镜实现高质量的图形均衡器（这部分内容也推荐给 Linux 用户）。音频轨道的最终阶段及其产生的音质，更多地取决于硬件规格：声卡、DAC、耳机、功放、音箱等。所以可以说，真正的音频冒险才刚刚重新开始 😉

---

作者简介：我的名字是 Marcin Szewczyk-Wilgan，我是一名全栈设计师。专业从事排版、出版设计和视觉传播。作为认证的 UX 设计师，我还基于 WordPress 和 Woocommerce 创建可用性高、排名靠前的网站和网店。此外，我管理着 Linux 和 FreeBSD 系统，提供主机托管、存储和安全服务。联系邮箱：contact@m4c.pl。
