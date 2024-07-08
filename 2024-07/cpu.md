# FreeBSD 中如何指定 CPU 类型

- <https://qiita.com/nanorkyo/items/e0417d8c1388750cb72c>


* [FreeBSD](https://qiita.com/tags/freebsd)
* [CPU](https://qiita.com/tags/cpu)
* [ 编译器](https://qiita.com/tags/%e3%82%b3%e3%83%b3%e3%83%91%e3%82%a4%e3%83%a9)
* [ 优化](https://qiita.com/tags/optimization)

发布于 2021 年 03 月 11 日，最后更新于 2021 年 03 月 18 日

# 开始

在查找如何指定 CPU 型号时^1^，突然想起自己好像没有看过日文文档…于是我下定决心进行调查备忘录。

这个规定基本上是追加的，所以我打算查看每个操作系统版本之间的差异 bsd.cpu.mk ^ 1 ^。我会先查找版本的差异并告诉你。请注意，区分大小写。

如果你使用的是最新的 CPU，并且未在此列表中，那么请指定最接近的 CPU 类型。大多数 CPU 都继承了之前架构的功能^ 2 ^，所以不应该会输出错误的代码。

现在让我们来谈谈 CPU 类型是什么，通过在 /etc/make.conf 中指定正在使用的 CPU，编译时将生成针对该 CPU 优化的代码，这就是所谓的 CPU 类型。对那些使用包管理系统的人来说，这是无关紧要的^ 3 ^。

## 注意事项

* 这里列出的内容不能保证适用于旧的 CPU 架构进行编译。
* 特别是在编译器从 GCC 4 切换到 LLVM 时，可能会导致之前可用的 CPU 类型不再被支持。
* 正在调查１０．０－Ｒ。不调查比这更旧的版本。
* 怎么处理其他的架构呢。有些复杂的和前后关系不明确，难以解释。
* 在其他架构中，一些消失了（ＩＡ６４，ＳＰＡＲＣ６４），一些增加了（ＡＡＲＣＨ６４，ＲＩＳＣＶ），各种各样。

# CPU 类型（amd64 环境和 i386 环境）

※调查版本：13.0、12.2、12.1、12.0、11.4、11.3、11.2、11.1、11.0、10.4、10.3、10.2、10.1、10.0

* 尽量确保最新的项目排在前面进行调查
* 希望将 amd64/i386 双用 CPU 和仅 i386 的 CPU 进行区分
* 当搭载 SSE3 的时期， Opteron 是什么来着？还有， Blue Lightning 和 Cyrix 之类的，还有 NexGen ，所有的回忆都飘到哪里去了！...

| 实现版本    | CPU 类型（别名）              | CPU 代码名（架构名）                  | 备注                                     |
| ---------------- | -------------------------------- | ------------------------------------ | -------------------------------------------- |
| 13.0           | `tigerlake`                               | Intel Tiger Lake                   |                                            |
| 13.0           | `cooperlake`                               | Intel Cooper Lake                  |                                            |
| 13.0           | `cascadelake`                               | Intel Cascade Lake                 |                                            |
| 11.3 / 12.1    | `icelake-server`                               | Intel Ice Lake-SP                  |                                            |
| 11.3 / 12.1    | `icelake-client`                               | Intel Ice Lake                     |                                            |
| 11.2 / 12.1    | `cannonlake`                               | Intel Cannon Lake                  |                                            |
| 11.3 / 12.1    | `skylake-avx512`（`skx`）                           | Intel Skylake]                     |                                            |
| 11.2           | `skylake-avx512`                               | Intel Skylake]                     |                                            |
| 11.0           | `skylake`                               | Intel Skylake                      |                                            |
| 11.0           | `broadwell`                               | Intel Broadwell                    |                                            |
| 11.0           | `haswell`（`core-avx2`）                           | Intel Haswell                      |                                            |
| 11.0           | `ivybridge`（`core-avx-i`）                           | Intel IvyBridge                    |                                            |
| 11.0           | `sandybridge`（`corei7-avx`）                           | Intel SandyBridge                  |                                            |
| 10.0           | `core-avx-i`                               | Intel IvyBridge                    |                                            |
| 10.0           | `corei7-avx`                               | Intel SandyBridge                  |                                            |
| 13.0           | `znver2`                               | AMD Zen2                           |                                            |
| 11.2           | `znver1`                               | AMD Zen                            |                                            |
| 11.0           | `bdver4`                               | AMD Excavator                      |                                            |
| 10.1           | `bdver3`                               | AMD Steamroller                    |                                            |
| 10.0           | `bdver2`                               | AMD Piledriver                     |                                            |
| 10.0           | `bdver1`                               | AMD Bulldozer                      |                                            |
| 13.0           | `btver2`                               | AMD Jaguar                         |                                            |
| 10.0           | `btver1`                               | AMD Bobcat                         |                                            |
| 11.3 / 12.1    | `tremont`                               | Intel Tremont                      |                                            |
| 11.3 / 12.1    | `goldmont-plus`                               | Intel Goldmont+                    |                                            |
| 11.2           | `goldmont`                               | Intel Goldmont                     |                                            |
| 11.0           | `silvermont`（`slm`）                           | Intel Silvermont                   |                                            |
| 10.1           | `slm`                               | Intel Silvermont                   |                                            |
| 11.0           | `bonnell`（`atom`）                           | Intel Bonnell                      |                                            |
| 10.0           | `atom`                               | Intel Bonnell                      |                                            |
| 11.2 / 12.1    | `knm`                               | Intel Knights Mill                 |                                            |
| 11.0           | `knl`                               | Intel Knights Landing              |                                            |
| 10.0           | `penryn`                               | Intel Penryn                       |                                            |
| 10.0           | `core2`                               | Intel Core2                        |                                            |
| 10.0           | `westmere`                               | Intel Westmere                     |                                            |
| 10.0           | `nehalem`（`corei7`）                           | Intel Nehalem                      |                                            |
| 10.0           | `yonah`                               | Intel Yonah                        |                                            |
| 10.0           | `prescott`（`core`）                           | Intel Prescot                      |                                            |
| 10.0           | `nocona`（`prescott`）                           | Intel Nocona                       | 在 amd64 环境中，`prescott` 被视为`nocona`。 |
| 10.0           | `pentium4`（`p4`）                           | Intel Pentium 4                    |                                            |
| 10.0           | `pentium4m`（`p4m`）                           | Intel Pentium 4M                   |                                            |
| 10.0           | `pentium3`（`p3`）                           | Intel Pentium 3                    |                                            |
| 10.0           | `pentium3m`（`p3m`）                           | Intel Pentium 3M                   |                                            |
| 10.0           | `pentium-m`（`p-m`）                           | Intel Pentium M                    |                                            |
| 10.0           | `pentium2`（`p2`）                           | Intel Pentium 2                    |                                            |
| 10.0           | `pentiumpro`（`i686`）                           | Intel Pentium Pro                  | FreeBSD13/i386 最低配置   |
| 10.0           | `pentium-mmx`（`i586/mmx`）                           | Intel Pentium MMX                  |                                            |
| 10.0           | `pentium`（`i586`）                           | Intel Pentium                      |                                            |
| 10.0           | `i486`                               | Intel i80486                       |                                            |
| 10.0           | `i386`                               | Intel i80386                       |                                            |
| 10.0           | `opteron-sse3`                               | AMD Opteron                        |                                            |
| 10.0           | `athlon64-sse3`                               | AMD Athlon64                       |                                            |
| 10.0           | `k8-sse3`                               | AMD K8                             |                                            |
| 10.0           | `opteron`                               | AMD Opteron                        |                                            |
| 10.0           | `amdfam10`（`barcelona`）                           | AMD K10                            |                                            |
| 10.0           | `athlon64`                               | AMD Athlon64                       |                                            |
| 10.0           | `athlon-fx`                               | AMD Athlon FX                      |                                            |
| 10.0           | `athlon-mp`                               | AMD Athlon MP                      |                                            |
| 10.0           | `athlon-xp`                               | AMD Athlon XP                      |                                            |
| 10.0           | `athlon-4`                               | AMD Athlon 4                       |                                            |
| 10.0           | `x86-64`                               | AMD Opteron                        | FreeBSD/amd64 最低配置     |
| 10.0           | `k8`                               | AMD K8                             |                                            |
| 11.0           | `athlon`（`k7`）                           | AMD Athlon                         |                                            |
| 10.0           | `athlon`                               | AMD Athlon                         |                                            |
| 10.0           | `athlon-tbird`                               | AMD Athlon Thunderbird             |                                            |
| 10.0           | `k7`                               | AMD K7                             |                                            |
| 10.0           | `k6`                               | AMD K6                             |                                            |
| 10.0           | `k6-2`                               | AMD K6-2                           |                                            |
| 10.0           | `k6-3`                               | AMD K6-3                           |                                            |
| 10.0           | `geode`                               | NSC Geode                          |                                            |
| 10.0           | `k5`                               | AMD K5                             |与 `pentium` 处理相同（不确定是不是别名）。    |
| 10.0           | `crusoe`                               | Transmeta Crusoe                   |                                            |
| 10.0           | `c7`                               | VIA C7                             |与 `c3-m` 相同。                       |
| 10.0           | `c3-2`                               | VIA C3-2                           |                                            |
| 10.0           | `c3`                               | VIA C3                             |                                            |
| 10.0           | `winchip2`                               | Centaur WinChip 2                  |                                            |
| 10.0           | `winchip2-c6`                               | Centaur WinChip C6                 |                                            |

---

1. `/usr/share/mk/bsd.cpu.mk`
2. 偶尔会回归到原点的架构存在。 ↩
3. 因为在包构建系统内指定了 CPU 类型而进行编译。 ↩
4. 向后和向前的兼容性。
