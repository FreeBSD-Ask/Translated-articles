# 利用 FreeBSD Capsicum 框架实现程序沙箱化

- 原文：[Sandbox Your Program Using FreeBSD's Capsicum](https://cdaemon.com/posts/capsicum)
- 作者： Jake Freeland  
- 发布日期：2023 年 9 月 1 日

当今时代，数据泄露的平均成本高达四百万美元以上，令人惊讶的是，只有 51% 的公司计划增加安全投资^[1](https://cdaemon.com/posts/capsicum#cite_note-1)^。这一数字很可能归因于实施健全安全解决方案所需的高昂成本。

许多安全框架都非常复杂。Capsicum 的独特之处在于其简洁性。开发者可以相对容易地将 Capsicum 集成到程序中，从而保护应用并降低高昂的实施成本。

本文展示了 Capsicum 的功能，并提供了将该框架集成到新旧程序中的完整指南。

## Capsicum

Capsicum 是款轻量级的安全框架，提供了用于限制程序能力的原语。更具体地说，Capsicum 能让开发者将其程序隔离到安全沙箱中。该框架基于最小特权原则设计，程序仅能访问其运行所需的资源。

## 能力沙箱

在支持 Capsicum 的系统上，程序可以使用 `cap_enter(2)` 进入沙箱：

```c
#include <sys/capsicum.h>
#include <stdio.h>

int
main(void)
{
    /* 进入 Capsicum 的能力模式。 */
    cap_enter();
    printf("能力模式下的 Hello world\n");
    return (0);
}
```

> **注意**
>
> 为简洁起见，本文档中的代码片段省略了错误处理。实际使用时，应在适当位置加入错误处理。

不需要链接额外的库；对 Capsicum 的支持已内置于标准 C 库（libc）。

Capsicum 能让开发者通过能力模式将程序沙箱化。在程序调用 `cap_enter(2)` 进入能力模式后，程序将无法自行获取新的资源。例如，使用 `open(2)` 打开文件会触发能力违规，造成函数调用失败，并将 `errno` 设置为 `ECAPMODE`（**译者注：特定错误码**）：这在能力模式下不允许。

一种对程序进行 Capsicum 化的方法是在进入能力模式之前打开好所有资源。程序事先拥有的资源可以在沙箱中继续使用。

若程序需要访问某子目录域中数量未知的资源，则可以使用 `openat(2)`、`mkdirat(2)`、`bindat(2)` 等 `*at()` 系统调用。这些函数需要额外的描述符参数，用作相对引用点来打开新资源。

```c
int dirfd;

dirfd = open("/home/jfree", O_RDONLY | O_DIRECTORY);
cap_enter();

/* 打开 "/home/jfree/foo"。 */
if (openat(dirfd, "foo", O_RDONLY) < 0)
    printf("这不会发生\n");
```

在这个例子中，`dirfd` 在进入能力模式之前获取。若程序进入能力模式，`dirfd` 就能作为相对参考点访问 `home/jfree/foo`。无法访问相对引用所提供子目录域之外的资源。

```c
int dirfd;

dirfd = open("/home/jfree", O_RDONLY | O_DIRECTORY);
cap_enter();

/* 打开 "/home/beastie"。 */
if (openat(dirfd, "../beastie", O_RDONLY) < 0)
    printf("这将会发生\n");
```

该 `openat(2)` 调用会失败，因为 `../beastie` 路径指向的目录不在 `dirfd` 提供的目录层级之内。

进程间通信同样受到限制。进入能力模式的进程不能向其他进程发送信号，命名共享内存对象被禁止。某些系统接口完全不可用，如 `reboot(2)` 和 `kldload(2)`。不过，这些限制都有相应的解决方法。


### 沙箱化的分区

预先打开资源对那些资源需求可预测的程序非常有效，但有些程序需要按需访问资源。在这种情况下，开发者可选择仅对程序的特定部分进行沙箱化。

如果整个程序无法在沙箱中运行，那么可以将其隔离分区（compartmentalization）。隔离分区的做法是把一个程序拆分成多个分区，每个分区承担自身的基本任务。通过隔离分区架构，开发者可以将受信任的代码保留在沙箱之外，而将不安全或危险的代码隔离到沙箱中的分区。如果危险代码中发现了安全漏洞，它将被隔离。

Capsicum 为程序的特定部分提供了直观的沙箱接口。程序可以在任意时刻派生一个新的子进程，并在能力模式下执行危险代码。诸如管道和套接字这样的进程间通信原语可用于数据交换，而不会触发能力违规。

```c
pid_t pid;
int pipefd[2], result;

pipe(pipefd);
/*
 * 创建一个子进程并将其隔离在能力沙箱中，
 * 在其中执行危险代码。
 */
pid = fork();
if (pid == 0) {
    close(pipefd[0]);
    cap_enter();
    result = dangerous_function();
    write(pipefd[1], &result, sizeof(result));
    exit(0);
}
close(pipefd[1]);
/* 从沙箱子进程获取结果。 */
result = read(pipefd[0], &result, sizeof(result));
printf("Result: %d\n", result);
/* 在父进程中继续正常执行。 */
```

可以在沙箱之外运行父进程，而子进程在隔离环境中执行危险代码。如果程序已经实现了隔离分区，开发者可以从沙箱化每个分区开始。大多数分区可能需要针对能力模式进行一些重构，但开发者可以自行选择哪些部分需要沙箱化。若正确实施，相较于对整个程序进行沙箱化，这种方式工作量更少，但安全性有大幅提升。

### 使用 libcasper(3) 请求资源

有些程序在设计时并未考虑隔离分区。开发者可以重新架构这些软件，但这通常需要大量时间和资源。幸运的是，库 `libcasper(3)` 为那些隔离分区无效的复杂程序提供了帮助。开发者可以利用 `libcasper(3)` 提供的接口在能力沙箱中获取新资源。

#### 使用 Casper 服务

在程序进入能力模式之前，可以与一个 casper 服务建立通信通道。Casper 服务是运行在能力沙箱之外的进程，与调用进程并行运行。建立的通信通道可用于向 casper 服务请求新资源。

> **注意**
>
> 必须在进入能力模式之前就打开 `libcasper(3)` 通道，否则 casper 进程会继承父进程的沙箱。

```c
cap_channel_t *cap_casper, *cap_net;
struct addrinfo *res;
int s;

/* 获取访问 libcasper(3) 服务的能力。 */
cap_casper = cap_init();

/*
 * 使用 cap_casper 能力与 "system.net" casper 服务
 * 建立通信通道。
 */
cap_net = cap_service_open(cap_casper, "system.net");

/*
 * 不再需要打开更多 casper 服务。
 * 关闭 cap_casper 能力。
 */
cap_close(cap_casper);

/*
 * 使用 cap_net(3) 库提供的 getaddrinfo() 变体。
 */
cap_getaddrinfo(cap_net, "freebsd.org", "80", NULL, &res);

s = socket(res->ai_family, res->ai_socktype, res->ai_protocol);

/*
 * 使用 cap_net(3) 库提供的 connect() 变体。
 */
cap_connect(cap_net, s, res->ai_addr, res->ai_addrlen);
```

库 `cap_net(3)` 利用 `libcasper(3)` 提供了支持能力模式的 libc 网络函数，否则这些函数会因 `ECAPMODE` 而失败。通过 `libcasper(3)` 接口的函数通常以 `cap_` 前缀命名，以表明它们能在能力模式下成功执行。类似于 `cap_net(3)` 的其他 casper 服务库也存在，并提供以 `cap_` 为前缀的 libc 函数。可在 `libcasper(3)` 手册页中找到完整列表。

如果程序支持在没有 `libcasper(3)` 的情况下构建，开发者依然可以使用 `cap_` 函数，而无需用 `#ifdef WITH_CASPER` 包裹。大多数 casper 服务将其 `cap_` 函数定义为宏，当 `WITH_CASPER` 未定义时会替换为非 `cap_` 形式。

#### 创建 Casper 服务

Casper 服务库很好地隐藏了接口 `libcasper(3)`，使程序开发者无需直接与其交互。但有时程序需要访问现有 casper 服务库未提供的资源，这种情况下开发者可以自行创建 `libcasper(3)` 服务。

所有 `libcasper(3)` 服务都基于 `CREATE_SERVICE(3)` 宏：

```c
CREATE_SERVICE(name, limit_func, command_func, flags);
```

每个参数都很重要，其中 `command_func` 函数指针尤为值得注意，因为它是服务的主例程。当通过 `cap_service_open(3)` 打开服务时，服务会等待命令。待接收到命令，就会传递给 `command_func` 的 `cmd` 参数。

```c
/*
 * cap_net(3) casper 服务库使用的命令函数。
 */
static int
net_command(const char *cmd, const nvlist_t *limits, nvlist_t *nvlin,
    nvlist_t *nvlout)
{
    if (strcmp(cmd, "bind") == 0)
        return (net_bind(limits, nvlin, nvlout));
    else if (strcmp(cmd, "connect") == 0)
        return (net_connect(limits, nvlin, nvlout));
    else if (strcmp(cmd, "gethostbyname") == 0)
        return (net_gethostbyname(limits, nvlin, nvlout));
    else if (strcmp(cmd, "gethostbyaddr") == 0)
        return (net_gethostbyaddr(limits, nvlin, nvlout));
    else if (strcmp(cmd, "getnameinfo") == 0)
        return (net_getnameinfo(limits, nvlin, nvlout));
    else if (strcmp(cmd, "getaddrinfo") == 0)
        return (net_getaddrinfo(limits, nvlin, nvlout));

    return (EINVAL);
}

CREATE_SERVICE("system.net", net_limit, net_command, 0);
```

大多数 `command_func` 会将传入的 `cmd` 字符串与一组字面量比较，若匹配则调用对应函数。Casper 服务运行在能力沙箱之外，因此在 `command_func` 内调用的所有函数都会以环境权限（ambient authority）执行，这意味着它们可以随意获取新资源。

可以使用函数在程序和 casper 服务之间 `cap_xfer_nvlist(3)` 交换资源。命令字符串和相关资源必须先封装进 `nvlist(9)`，再传输给服务。Nvlist 可存储数字、字符串、二进制、描述符以及其他 nvlists。每个资源都必须配有一个标识名，用于检索。

```c
/*
 * 向与 @chan 绑定的 casper 服务发送 "bind" 命令。
 */
static int
cap_bind(cap_channel_t *chan, int sockfd, const struct sockaddr *addr,
    socklen_t addrlen)
{
    nvlist_t *nvl = nvlist_create(0);
    int error;

    nvlist_add_string(nvl, "cmd", "bind");
    nvlist_add_descriptor(nvl, "sockfd", sockfd);
    nvlist_add_binary(nvl, "addr", addr, addrlen);

    nvl = cap_xfer_nvlist(chan, nvl);
    if (nvl == NULL)
        return (-1);

    error = nvlist_get_number(nvl, "error");
    if (error != 0) {
        nvlist_destroy(nvl);
        errno = error;
        return (-1);
    }

    error = dup2(sockfd, nvlist_get_descriptor(nvl, "sockfd"));
    nvlist_destroy(nvl);

    return (error == -1 ? -1 : 0);
}
```

`cap_xfer_nvlist(3)` 函数会将 `nvl` nvlist 发送到与 `chan` 绑定的 casper 服务。一旦 nvlist 到达 casper 服务，命令字符串会被提取并传入服务的 `command_func`。

回顾 `net_command()` 中的片段：

```c
if (strcmp(cmd, "bind") == 0)
        return (net_bind(limits, nvlin, nvlout));
```

`cap_bind()` 发送了 `"bind"` 命令字符串，因此该 `strcmp()` 条件成立并调用 `net_bind()`。

```c
/*
 * net_bind() 函数的简化版。
 * 负责从 @nvlin 提取参数，调用 bind(2)，
 * 并将返回值加入 @nvlout。
 */
static int
net_bind(const nvlist_t *limits __unused, nvlist_t *nvlin, nvlist_t *nvlout)
{
    int sockfd;
    const void *addr;
    size_t len;

    addr = nvlist_get_binary(nvlin, "addr", &len);
    sockfd = nvlist_take_descriptor(nvlin, "sockfd");
    if (bind(sockfd, saddr, len) < 0) {
        int serrno = errno;
        close(sockfd);
        return (serrno);
    }
    nvlist_move_descriptor(nvlout, "sockfd", sockfd);

    return (0);
}
```

当 `bind(2)` 成功时，套接字必须被传回能力沙箱。`nvlist_move_descriptor(9)` 函数会将套接字描述符移入 `nvlout` nvlist，并接管该描述符的所有权。

回顾 `cap_bind()` 中的片段：

```c
nvl = cap_xfer_nvlist(chan, nvl);
    if (nvl == NULL)
        err(1, "Failed transfer bind() nvlist");
```

`cap_xfer_nvlist(9)` 的返回值是 `nvlout` nvlist，它持有已绑定的套接字描述符。可以通过 `nvlist_get_descriptor(nvl, "sockfd")` 从返回的 nvlist 中获取该描述符。

#### 限制 Casper 服务

Casper 服务提供的接口通常过于宽泛。在使用 `cap_net(3)` 时，你可能只需要服务提供的部分函数。大多数服务库会定义自己的限制机制，使开发者可以禁用程序不需要的函数。

```c
/*
 * 使用 cap_net(3) 的限制机制，仅允许解析 freebsd.org
 * 在 80 端口上的地址。
 *
 * 假设 cap_net(3) 服务已经被打开并在 @cap_net 上监听。
 */
cap_net_limit_t *limit;
int familylimit;

/* 仅允许名称解析 (cap_getaddrinfo(3))。 */
limit = cap_net_limit_init(cap_net, CAPNET_NAME2ADDR);

/* 将名称解析限制为 "freebsd.org" 的 80 端口。 */
cap_net_limit_name2addr(limit, "freebsd.org", "80");

/* 将名称解析限制为 IPv4 地址。 */
familylimit = AF_INET;
cap_net_limit_name2addr_family(limit, &familylimit, 1);

/* 将限制应用到 cap_net。 */
cap_net_limit(limit);
```

每个服务库都提供不同的限制机制。由于使用模式过多，建议开发者查阅各服务库的手册以获取详细信息和示例。


#### 为 Casper 服务创建限制

开发者可以在 `CREATE_SERVICE(3)` 中指定 `limit_func` 函数指针以限制服务接口。当程序调用 `cap_limit_set(3)` 时，提供的 `limits` 会被重定向到 casper 服务的 `limit_func` 中，并相应应用。该机制的灵活性允许服务对其接口进行精细限制。

大多数服务库会为 `cap_limit_set(3)` 提供包装函数，并使用自定义的 `_limit_t` 类型。该类型由服务库定义，用于跟踪服务特定的限制。

随着限制的细化，实现可能很快变得复杂。像 `cap_net(3)` 这样的服务提供了对接口的广泛控制，因此其限制函数需要处理所有边界情况。快速查看 `cap_net(3)` 的限制函数可以发现，为不同的“限制模式”实现了单独的验证函数，以确保限制被正确应用和执行。

```c
/*
 * cap_net(3) 中 net_limit() 函数的片段。
 * 为清晰起见，部分代码被省略。
 * 上下文见 "lib/libcasper/services/cap_net/cap_net.c"。
 */
while ((name = nvlist_next(newlimits, NULL, &cookie)) != NULL) {
        /* ... */
        if (strcmp(name, LIMIT_NV_BIND) == 0) {
                hasbind = true;
                if (!verify_bind_newlimts(oldlimits,
                    cnvlist_get_nvlist(cookie))) {
                        return (ENOTCAPABLE);
                }
        } else if (strcmp(name, LIMIT_NV_CONNECT) == 0) {
                hasconnect = true;
                if (!verify_connect_newlimits(oldlimits,
                    cnvlist_get_nvlist(cookie))) {
                        return (ENOTCAPABLE);
                }
        } else if (strcmp(name, LIMIT_NV_ADDR2NAME) == 0) {
                hasaddr2name = true;
                if (!verify_addr2name_newlimits(oldlimits,
                    cnvlist_get_nvlist(cookie))) {
                        return (ENOTCAPABLE);
                }
        } else if (strcmp(name, LIMIT_NV_NAME2ADDR) == 0) {
                hasname2addr = true;
                if (!verify_name2addr_newlimits(oldlimits,
                    cnvlist_get_nvlist(cookie))) {
                        return (ENOTCAPABLE);
                }
        }
}
```

限制函数依赖于其所限制的服务，因此没有统一的编写模式。想要为 casper 服务创建限制的开发者应参考 FreeBSD 源码中 `lib/libcasper/services` 的系统服务库示例。



#### 回顾：Casper 服务的组成部分

casper 服务由以下四个主要部分组成：

* 以 `cap_` 为前缀的函数，通过 `cap_xfer_nvlist(3)` 向 casper 服务发送命令；
* 在沙箱外执行命令相关代码并返回新资源的 `command_func`；
* 限制服务用途的 `limit_func`；
* 将服务拼接在一起的 `CREATE_SERVICE(3)` 宏。

虽然技术上可以创建返回任意命名资源的 casper 服务，但这会破坏将程序隔离到沙箱的目的。设计良好的 casper 服务接口应当有限且受约束，避免被利用。

### 检测违规

当程序进入能力模式时，它是否遵循沙箱规则并不总是显而易见。尝试打开受限资源的函数会触发能力违规，并返回 `errno = ECAPMODE`。即便有适当的错误检查，排查违规也可能耗时。幸运的是，`ktrace(2)` 内核跟踪工具能帮助发现违规。

通常，程序必须进入能力模式才会报告违规，但 `ktrace(2)` 可以在程序 **未进入** 能力模式时记录违规。这意味着开发者可以在不修改程序的情况下运行违规跟踪，查看违规发生的位置。由于程序未真正进入能力模式，它依然会获取资源并正常执行。

在程序开头添加以下两行即可启用违规跟踪：

```c
open("ktrace.out", O_RDONLY | O_CREAT | O_TRUNC);
ktrace("ktrace.out", KTROP_SET, KTRFAC_CAPFAIL, getpid());
```

这段代码创建了 `ktrace(2)` 的输出文件，并指定 `KTRFAC_CAPFAIL` 跟踪点以记录能力失败。

> 注意：`ktrace(2)` 手册页对系统调用的使用有详细说明。启用其他跟踪点（如 `KTRFAC_NAMEI` 记录文件名查找）有助于定位文件系统违规的来源。

下面的 `cap_violate` 例程试图触发 `ktrace(2)` 可捕获的所有违规。无需理解具体做什么，只需知道它能触发违规即可。

```c
open("ktrace.out", O_RDONLY | O_CREAT | O_TRUNC);
ktrace("ktrace.out", KTROP_SET, KTRFAC_CAPFAIL, getpid());

cap_rights_init(&rights, CAP_READ);
caph_rights_limit(STDERR_FILENO, &rights);
write(STDERR_FILENO, &val, sizeof(val));

cap_rights_set(&rights, CAP_WRITE);
caph_rights_limit(STDERR_FILENO, &rights);

kinf.kf_structsize = sizeof(struct kinfo_file);
fcntl(STDIN_FILENO, F_KINFO, &kinf);

socket(AF_INET, SOCK_RAW, IPPROTO_ICMP);

addr.sin_family = AF_INET;
addr.sin_port = htons(5000);
addr.sin_addr.s_addr = INADDR_ANY;
bind(socket(AF_INET, SOCK_DGRAM, IPPROTO_UDP),
    (const struct sockaddr *)&addr, sizeof(addr));
sendto(fd, NULL, 0, 0, (const struct sockaddr *)&addr, sizeof(addr));

kill(getppid(), SIGCONT);

openat(AT_FDCWD, "/", O_RDONLY);

CPU_SET(0, &cpuset_mask);
cpuset_setaffinity(CPU_LEVEL_WHICH, CPU_WHICH_PID, getppid(),
    sizeof(cpuset_mask), &cpuset_mask);
```

若进程被跟踪，跟踪数据会一直记录，直到进程退出或清除跟踪点。此时可以使用 `kdump(2)` 将 `ktrace(2)` 的转储转换为可读格式。

```shell
# ./cap_violate
# kdump
  1915 cap_violate CAP   operation requires CAP_WRITE, descriptor holds CAP_READ
  1915 cap_violate CAP   attempt to increase capabilities from CAP_READ to CAP_READ,CAP_WRITE
  1915 cap_violate CAP   system call not allowed: fcntl, cmd: F_KINFO
  1915 cap_violate CAP   socket: protocol not allowed: IPPROTO_ICMP
  1915 cap_violate CAP   system call not allowed: bind
  1915 cap_violate CAP   sendto: restricted address lookup: struct sockaddr { AF_INET, 0.0.0.0:5000 }
  1915 cap_violate CAP   kill: signal delivery not allowed: SIGCONT
  1915 cap_violate CAP   openat: restricted VFS lookup: AT_FDCWD
  1915 cap_violate CAP   cpuset_setaffinity: restricted cpuset operation
```

`cap_violate` 程序中的每次违规都会在 `kdump(1)` 输出中生成一条 CAP 记录。开发者可利用这些输出定位并替换触发违规的代码。

大多数实际程序应尽量避免违规，而不是像 `cap_violate` 那样触发它们。下面的 `kdump(1)` 输出展示了在启用 `KTRFAC_CAPFAIL` 与 `KTRFAC_NAMEI` 跟踪点后，使用 `unzip(1)` 解压归档文件（未进行 Capsicum 化）时的结果。

```shell
# unzip foo.zip
# kdump
  1926 unzip    NAMI  "foo.zip"
  1926 unzip    CAP   openat: restricted VFS lookup: AT_FDCWD
  1926 unzip    CAP   system call not allowed: open
  1926 unzip    NAMI  "/etc/localtime"
  1926 unzip    NAMI  "bar"
  1926 unzip    CAP   fstatat: restricted VFS lookup: AT_FDCWD
  1926 unzip    CAP   system call not allowed: mkdir
  1926 unzip    NAMI  "bar"
  1926 unzip    NAMI  "bar"
  1926 unzip    CAP   fstatat: restricted VFS lookup: AT_FDCWD
  1926 unzip    NAMI  "bar/bar.txt"
  1926 unzip    CAP   fstatat: restricted VFS lookup: AT_FDCWD
  1926 unzip    NAMI  "bar/bar.txt"
  1926 unzip    CAP   openat: restricted VFS lookup: AT_FDCWD
  1926 unzip    NAMI  "baz"
  1926 unzip    CAP   fstatat: restricted VFS lookup: AT_FDCWD
  1926 unzip    CAP   system call not allowed: mkdir
  1926 unzip    NAMI  "baz"
  1926 unzip    NAMI  "baz"
  1926 unzip    CAP   fstatat: restricted VFS lookup: AT_FDCWD
  1926 unzip    NAMI  "baz/baz.txt"
  1926 unzip    CAP   fstatat: restricted VFS lookup: AT_FDCWD
  1926 unzip    NAMI  "baz/baz.txt"
  1926 unzip    CAP   openat: restricted VFS lookup: AT_FDCWD
```

这类输出更接近开发者在自己程序中会看到的情况。`unzip(2)` 正在重建 zip 文件中的目录结构。所有 `open(2)`、`fstat(2)` 和 `mkdir(2)` 调用都被 libc 隐式转换为带 `AT_FDCWD` 的 `*at()` 变体。由于 `AT_FDCWD` 在能力模式下不可用，因此触发违规。这类问题可通过在进入能力模式前打开一个当前目录描述符（等价于 `AT_FDCWD`），并将其传递给 `openat(2)`、`fstatat(2)` 和 `mkdirat(2)` 来避免。

> **注意**
>
> 在能力模式下违规总会返回错误，但在跟踪过程中不会被当作错误，因此程序行为可能不同。

违规跟踪是开发者工具箱中的一项实用工具。只需几秒钟即可用 `ktrace(2)` 运行程序，其结果几乎总能作为使用 Capsicum 沙箱化程序的良好起点。

## 能力 (Capabilities)

尽管名称相似，能力模式 (capability mode) 和能力 (capabilities) 是不同的内核原语。

* 能力模式是 Capsicum 实现的安全沙箱。
* 能力是带有权限的文件描述符。

如果用户想要对一个能力描述符执行 `read(2)`，则该描述符必须具有 `CAP_READ` 权限。如果用户想要对一个套接字能力描述符执行 `bind(2)`，则该描述符必须具有 `CAP_BIND` 权限。几乎每个描述符操作都有细粒度的权限；完整列表见 `rights(4)` 手册页。

当使用 `open(2)`、`socket(2)` 等创建文件描述符时，它会被赋予完整的能力集。可以使用 `cap_rights_limit(2)` 函数限制描述符的能力。

```c
/*
 * 将文件描述符限制为只读。
 */
cap_rights_t rights;
int fd;
char buf[1] = 'x';

fd = open("/home/jfree/foo", O_RDWR);

cap_rights_init(&rights, CAP_READ);
cap_rights_limit(fd, &rights);

if (read(fd, buf, sizeof(buf)) < 0)
    printf("这不会发生，因为我们有 CAP_READ\n");

if (write(fd, buf, sizeof(buf)) < 0)
    printf("这会发生，因为我们缺少 CAP_WRITE\n");
```

能力的设计原则是最小化权限。一旦描述符的权限被限制，它就不应执行超出权限的操作。描述符的权限始终可以被限制，但不能被扩展。

```c
/*
 * 试图扩展描述符的权限。
 */
cap_rights_t rights;
int fd;

fd = open("/home/jfree/foo", O_RDWR);

cap_rights_init(&rights, CAP_READ);
cap_rights_limit(fd, &rights);

cap_rights_set(&rights, CAP_WRITE);
if (cap_rights_limit(fd, &rights) < 0)
    printf("应用权限失败；权限永远不能被扩展\n");
```

在对程序进行沙箱化过于严格的情况下，开发者可以选择限制能力描述符。能力提供了对允许操作的精细控制，但使用这种保护的程序容易受到人为疏忽的影响。如果开发者忘记限制某个能力，就可能引入恶意代码滥用的风险。

想要最大化安全性的开发者可以在能力模式下结合使用能力。能力提供了能力模式本身不具备的细粒度功能。例如，只允许描述符进行 `read(2)` 操作，这是能力可以做到的，但能力模式不能。


## 使用 Capsicum 提升安全性

[能力模式](https://cdaemon.com/posts/capsicum#the-capability-sandbox) 和 [能力](https://cdaemon.com/posts/capsicum#capabilities) 的设计初衷都是让程序更安全。能力模式通过将程序与系统其余部分隔离提供了确定的安全性。能力则提供了一种更灵活但不如前者严格的方式来限制程序权限。只要正确集成其中任意一种原语，开发者就能确信，他们的程序比引入 Capsicum 前更加安全。

## 参考资料

1. ["Cost of a Data Breach Report 2023"](https://www.ibm.com/reports/data-breach). IBM Corporation. 2023-07-24. Retrieved 2023-08-31.

## 相关资料

* [https://www.cl.cam.ac.uk/research/security/capsicum/](https://www.cl.cam.ac.uk/research/security/capsicum/)
* [https://www.usenix.org/legacy/events/sec10/tech/full\_papers/Watson.pdf](https://www.usenix.org/legacy/events/sec10/tech/full_papers/Watson.pdf)
* [https://wiki.freebsd.org/Capsicum](https://wiki.freebsd.org/Capsicum)
