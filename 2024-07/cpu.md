# 如何在 FreeBSD 中指定 CPU 类型

- [ＦｒｅｅＢＳＤでのＣＰＵＴＹＰＥの指定方法](https://qiita.com/nanorkyo/items/e0417d8c1388750cb72c)
- 作者：重村法克
- 发布于 2021 年 03 月 11 日，最后更新于 2021 年 03 月 18 日

## 引言

在研究如何指定 `CPUTYPE` 的方法时[1]，我突然想起自己从没看过相关的日文文档……于是下定决心，打算留下这份研究笔记。

这个指定方式基本上是不断追加的，因此我打算针对每个操作系统版本，调查 `bsd.cpu.mk`[1] 的差异。为了便于参考，这里也列出我已经研究过的版本。另外，区分大小写。

如果你正在使用最新的 CPU，而表中没有列出该型号，请指定最接近的一代 `CPUTYPE`。因为大多数 CPU 都继承了旧架构的功能[2]，所以应该不会生成报错信息。

那么，`CPUTYPE` 是什么呢？它是在 `/etc/make.conf` 中使用的变量，用于指定你当前所使用的 CPU，这样在编译时就可以生成针对该 CPU 优化的代码。对于使用软件包系统管理的人来说，这个设置基本没有关系[3]。

## 注意事项

* 这里列出的内容，并不保证可以针对旧的 CPU 架构进行编译。
* 特别是在编译器从 GCC 4 迁移到 LLVM 的过程中，可能出现原本还能用的 `CPUTYPE` 无法使用的情况。
* 本文从 10.0-RELEASE 开始进行调查，更旧的版本将不予调查。
* 其他架构方面怎么办呢……因为存在各种复杂关系和前后顺序问题[4]，所以不太容易说明。
* 比如其他架构有些已经移除（如 IA64、SPARC64），也有新增（如 AARCH64、RISC-V）等各种变化。

## `CPUTYPE`（amd64 环境及 i386 环境）

※已调查版本：13.0、12.2、12.1、12.0、11.4、11.3、11.2、11.1、11.0、10.4、10.3、10.2、10.1、10.0

* 尽量将较新的项目排在前面
* 想要区分 amd64/i386 通用的 CPU 与仅支持 i386 的 CPU
* 像“`Opteron` 支持 SSE3 的时候是哪代？”、“`Blue Lightning` 啊 `Cyrix` 啊 `NexGen` 啊，这些满满的回忆都去哪了！”……等等。

| 导入版本        | CPUTYPE（别名）                 | CPU 代号（架构名称）           | 备注                                  |
| ----------- | --------------------------- | ---------------------- | ----------------------------------- |
| 13.0        | `tigerlake`                 | Intel Tiger Lake       |                                     |
| 13.0        | `cooperlake`                | Intel Cooper Lake      |                                     |
| 13.0        | `cascadelake`               | Intel Cascade Lake     |                                     |
| 11.3 / 12.1 | `icelake-server`            | Intel Ice Lake-SP      |                                     |
| 11.3 / 12.1 | `icelake-client`            | Intel Ice Lake         |                                     |
| 11.2 / 12.1 | `cannonlake`                | Intel Cannon Lake      |                                     |
| 11.3 / 12.1 | `skylake-avx512`（`skx`）     | Intel Skylake          |                                     |
| 11.2        | `skylake-avx512`            | Intel Skylake          |                                     |
| 11.0        | `skylake`                   | Intel Skylake          |                                     |
| 11.0        | `broadwell`                 | Intel Broadwell        |                                     |
| 11.0        | `haswell`（`core-avx2`）      | Intel Haswell          |                                     |
| 11.0        | `ivybridge`（`core-avx-i`）   | Intel IvyBridge        |                                     |
| 11.0        | `sandybridge`（`corei7-avx`） | Intel SandyBridge      |                                     |
| 10.0        | `core-avx-i`                | Intel IvyBridge        |                                     |
| 10.0        | `corei7-avx`                | Intel SandyBridge      |                                     |
| 13.0        | `znver2`                    | AMD Zen2               |                                     |
| 11.2        | `znver1`                    | AMD Zen                |                                     |
| 11.0        | `bdver4`                    | AMD Excavator          |                                     |
| 10.1        | `bdver3`                    | AMD Steamroller        |                                     |
| 10.0        | `bdver2`                    | AMD Piledriver         |                                     |
| 10.0        | `bdver1`                    | AMD Bulldozer          |                                     |
| 13.0        | `btver2`                    | AMD Jaguar             |                                     |
| 10.0        | `btver1`                    | AMD Bobcat             |                                     |
| 11.3 / 12.1 | `tremont`                   | Intel Tremont          |                                     |
| 11.3 / 12.1 | `goldmont-plus`             | Intel Goldmont+        |                                     |
| 11.2        | `goldmont`                  | Intel Goldmont         |                                     |
| 11.0        | `silvermont`（`slm`）         | Intel Silvermont       |                                     |
| 10.1        | `slm`                       | Intel Silvermont       |                                     |
| 11.0        | `bonnell`（`atom`）           | Intel Bonnell          |                                     |
| 10.0        | `atom`                      | Intel Bonnell          |                                     |
| 11.2 / 12.1 | `knm`                       | Intel Knights Mill     |                                     |
| 11.0        | `knl`                       | Intel Knights Landing  |                                     |
| 10.0        | `penryn`                    | Intel Penryn           |                                     |
| 10.0        | `core2`                     | Intel Core2            |                                     |
| 10.0        | `westmere`                  | Intel Westmere         |                                     |
| 10.0        | `nehalem`（`corei7`）         | Intel Nehalem          |                                     |
| 10.0        | `yonah`                     | Intel Yonah            |                                     |
| 10.0        | `prescott`（`core`）          | Intel Prescott         |                                     |
| 10.0        | `nocona`（`prescott`）        | Intel Nocona           | 在 amd64 环境中，`prescott` 被视为 `nocona` |
| 10.0        | `pentium4`（`p4`）            | Intel Pentium 4        |                                     |
| 10.0        | `pentium4m`（`p4m`）          | Intel Pentium 4M       |                                     |
| 10.0        | `pentium3`（`p3`）            | Intel Pentium 3        |                                     |
| 10.0        | `pentium3m`（`p3m`）          | Intel Pentium 3M       |                                     |
| 10.0        | `pentium-m`（`p-m`）          | Intel Pentium M        |                                     |
| 10.0        | `pentium2`（`p2`）            | Intel Pentium 2        |                                     |
| 10.0        | `pentiumpro`（`i686`）        | Intel Pentium Pro      | FreeBSD 13/i386 的最低要求               |
| 10.0        | `pentium-mmx`（`i586/mmx`）   | Intel Pentium MMX      |                                     |
| 10.0        | `pentium`（`i586`）           | Intel Pentium          |                                     |
| 10.0        | `i486`                      | Intel i80486           |                                     |
| 10.0        | `i386`                      | Intel i80386           |                                     |
| 10.0        | `opteron-sse3`              | AMD Opteron            |                                     |
| 10.0        | `athlon64-sse3`             | AMD Athlon64           |                                     |
| 10.0        | `k8-sse3`                   | AMD K8                 |                                     |
| 10.0        | `opteron`                   | AMD Opteron            |                                     |
| 10.0        | `amdfam10`（`barcelona`）     | AMD K10                |                                     |
| 10.0        | `athlon64`                  | AMD Athlon64           |                                     |
| 10.0        | `athlon-fx`                 | AMD Athlon FX          |                                     |
| 10.0        | `athlon-mp`                 | AMD Athlon MP          |                                     |
| 10.0        | `athlon-xp`                 | AMD Athlon XP          |                                     |
| 10.0        | `athlon-4`                  | AMD Athlon 4           |                                     |
| 10.0        | `x86-64`                    | AMD Opteron            | FreeBSD/amd64 的最低要求                 |
| 10.0        | `k8`                        | AMD K8                 |                                     |
| 11.0        | `athlon`（`k7`）              | AMD Athlon             |                                     |
| 10.0        | `athlon`                    | AMD Athlon             |                                     |
| 10.0        | `athlon-tbird`              | AMD Athlon Thunderbird |                                     |
| 10.0        | `k7`                        | AMD K7                 |                                     |
| 10.0        | `k6`                        | AMD K6                 |                                     |
| 10.0        | `k6-2`                      | AMD K6-2               |                                     |
| 10.0        | `k6-3`                      | AMD K6-3               |                                     |
| 10.0        | `geode`                     | NSC Geode              |                                     |
| 10.0        | `k5`                        | AMD K5                 | 与 `pentium` 处理类似（不确定是否为别名）          |
| 10.0        | `crusoe`                    | Transmeta Crusoe       |                                     |
| 10.0        | `c7`                        | VIA C7                 | 与 `c3-m` 处理类似                       |
| 10.0        | `c3-2`                      | VIA C3-2               |                                     |
| 10.0        | `c3`                        | VIA C3                 |                                     |
| 10.0        | `winchip2`                  | Centaur WinChip 2      |                                     |
| 10.0        | `winchip2-c6`               | Centaur WinChip C6     |                                     |

1. `/usr/share/mk/bsd.cpu.mk`
2. 有些架构会偶尔回归到原点。
3. 因为它们会根据包构建系统中指定的 CPUTYPE 进行编译。
4. 向上兼容和向下兼容的关系。
