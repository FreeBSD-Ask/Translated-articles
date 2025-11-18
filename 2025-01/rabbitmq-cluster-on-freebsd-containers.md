# 在 FreeBSD 容器上运行 RabbitMQ 集群

- [RabbitMQ Cluster on FreeBSD Containers](https://vermaden.wordpress.com/2019/06/05/rabbitmq-cluster-on-freebsd-containers/)
- 作者：𝚟𝚎𝚛𝚖𝚊𝚍𝚎𝚗
- 2019/06

我真的很喜欢那种小而简单、专注做好一件事并且做得非常好的专用解决方案——也许是因为我太喜欢 UNIX 了。这种理念的一个好例子是 Minio 对象存储，它实现了 S3 协议，支持分布式集群、纠删码，并内置了 Web 界面，还有许多其他特性——我曾在《Distributed Object Storage with Minio on FreeBSD》一文中介绍过。

RabbitMQ 是另一个这样的例子——它大概是目前最流行的 AMQP 协议实现之一——同样带有一个小巧、精致的 Web 界面。但它和 Minio 的区别在于“力量”。Minio 的 Web 界面非常基础、面向用户，大多数管理和配置任务都需要通过 CLI 完成。Minio 的 Web 界面主要提供创建/删除 buckets、上传/下载文件等功能。而 RabbitMQ 的 Web 界面则非常强大，一旦启用，你几乎不需要命令行了，所有事情都可以通过 Web 界面完成。

![rabbitmq-logo.png](https://vermaden.wordpress.com/wp-content/uploads/2019/06/rabbitmq-logo-1.png?w=960)

和 ActiveMQ、Apache Kafka 等其它消息系统相比，根据 Google Trends 的趋势来看，RabbitMQ 非常流行。

![rabbitmq-trends.jpg](https://vermaden.wordpress.com/wp-content/uploads/2019/06/rabbitmq-trends.jpg?w=960)

今天我想向你展示在 FreeBSD 容器上构建一个带有镜像队列的高度冗余 RabbitMQ 集群的消息系统。

在 FreeBSD 提供的所有虚拟化方式中（VirtualBox / Bhyve / QEMU / Jails / Docker），我选择了最轻量的 FreeBSD Containers —— Jails :🙂:

图例依旧保持不变。

在 **host** 系统上以 **root** 用户执行的命令：

```
host # command
```

在 **host** 系统上以 **普通用户** 执行的命令：

```
host % command
```

在 **rabbitX** Jail 中执行的命令：

```
rabbitX # command
```

# Jail 配置

首先我们将创建用于本次部署的基础 Jails。host 系统与这些 *Jails Containers* 都使用 FreeBSD 11.2-RELEASE 系统。


```sh
host # mkdir -p /jail/BASE
host # fetch -o /jail/BASE/11.2-RELEASE.base.txz http://ftp.freebsd.org/pub/FreeBSD/releases/amd64/12.1-RELEASE/base.txz
host # for I in 1 2; do echo ${I}; mkdir -p /jail/rabbit${I}; tar --unlink -xpJf /jail/BASE/11.2-RELEASE.base.txz -C /jail/rabbit${I}; done
1
2
host #
```

我们现在已经有了 2 个空的、干净的 Jails。

接下来要把这些 Jails 的配置加入 **`/etc/jail.conf`** 文件中。

因为我使用的是笔记本作为 Jail 的宿主机，所以 Jails 将会绑定无线网卡 **`wlan0`**，并使用 **`192.168.43.10X`** 这些地址。同时我也添加了 **`10.0.0.10X`** 这组地址，主要是为了撰写本文时操作更方便。

```sh
host # for I in 1 2
do
  cat >> /etc/jail.conf << __EOF
rabbit${I} {
  host.hostname = rabbit${I}.local;
  ip4.addr += 192.168.43.10${I};
  ip4.addr += 10.0.0.10${I};
  interface = wlan0;
  path = /jail/rabbit${I};
  exec.start = "/bin/sh /etc/rc";
  exec.stop = "/bin/sh /etc/rc.shutdown";
  exec.clean;
  mount.devfs;
  allow.raw_sockets;
}

__EOF
done
host #
```

这就是配置完成后的 **/etc/jail.conf** 文件的样子。

```sh
host # cat /etc/jail.conf
rabbit1 {
  host.hostname = rabbit1.local;
  ip4.addr += 192.168.43.101;
  ip4.addr += 10.0.0.101;
  interface = wlan0;
  path = /jail/rabbit1;
  exec.start = "/bin/sh /etc/rc";
  exec.stop = "/bin/sh /etc/rc.shutdown";
  exec.clean;
  mount.devfs;
  allow.raw_sockets;
}

rabbit2 {
  host.hostname = rabbit2.local;
  ip4.addr += 192.168.43.102;
  ip4.addr += 10.0.0.102;
  interface = wlan0;
  path = /jail/rabbit2;
  exec.start = "/bin/sh /etc/rc";
  exec.stop = "/bin/sh /etc/rc.shutdown";
  exec.clean;
  mount.devfs;
  allow.raw_sockets;
}
```

现在我们可以启动这些 Jail 了。

```sh
host # for I in 1 2; do service jail onestart rabbit${I}; done
Starting jails: rabbit1.
Starting jails: rabbit2.
```

Jail 已正常运行。


```sh
# jls
   JID  IP Address      Hostname                      Path
     1  192.168.43.101  rabbit1.local                 /jail/rabbit1
     2  192.168.43.102  rabbit2.local                 /jail/rabbit2
```

现在是时候给 Jail 添加 DNS 服务器，以便它们能够访问互联网。


```sh
host # for I in 1 2; do cat /jail/rabbit${I}/etc/resolv.conf; done
nameserver 1.1.1.1
nameserver 1.1.1.1
```

现在我们将把软件包源从 **'quarterly'** 切换到 **'latest'**。

```sh
host # for I in 1 2; do sed -i '' s/quarterly/latest/g /jail/rabbit${I}/etc/pkg/FreeBSD.conf; done

host # for I in 1 2; do grep latest /jail/rabbit${I}/etc/pkg/FreeBSD.conf; done
  url: "pkg+http://pkg.FreeBSD.org/${ABI}/latest",
  url: "pkg+http://pkg.FreeBSD.org/${ABI}/latest",
```

# RabbitMQ 安装

现在我们可以安装包 RabbitMQ 了。

```
host # for I in 1 2; do jexec rabbit${I} env ASSUME_ALWAYS_YES=yes pkg install -y rabbitmq; echo; done
Bootstrapping pkg from pkg+http://pkg.FreeBSD.org/FreeBSD:11:amd64/latest, please wait...
Verifying signature with trusted certificate pkg.freebsd.org.2013102301... done
[rabbit1.local] Installing pkg-1.10.5_5...
[rabbit1.local] Extracting pkg-1.10.5_5: 100%
Updating FreeBSD repository catalogue...
pkg: Repository FreeBSD load error: access repo file(/var/db/pkg/repo-FreeBSD.sqlite) failed: No such file or directory
[rabbit1.local] Fetching meta.txz: 100%    944 B   0.9kB/s    00:01    
[rabbit1.local] Fetching packagesite.txz: 100%    6 MiB 745.4kB/s    00:09    
Processing entries: 100%
FreeBSD repository update completed. 32114 packages processed.
All repositories are up to date.
Updating database digests format: 100%
The following 2 package(s) will be affected (of 0 checked):

New packages to be INSTALLED:
        rabbitmq: 3.7.15
        erlang-runtime19: 21.3.8.2

Number of packages to be installed: 2

The process will require 104 MiB more space.
41 MiB to be downloaded.
[rabbit1.local] [1/2] Fetching rabbitmq-3.7.15.txz: 100%    9 MiB 762.2kB/s    00:12    
[rabbit1.local] [2/2] Fetching erlang-runtime19-21.3.8.2.txz: 100%   33 MiB 978.8kB/s    00:35    
Checking integrity... done (0 conflicting)
[rabbit1.local] [1/2] Installing erlang-runtime19-21.3.8.2...
[rabbit1.local] [1/2] Extracting erlang-runtime19-21.3.8.2: 100%
[rabbit1.local] [2/2] Installing rabbitmq-3.7.15...
===> Creating groups.
Creating group 'rabbitmq' with gid '135'.
===> Creating users
Creating user 'rabbitmq' with uid '135'.
[rabbit1.local] [2/2] Extracting rabbitmq-3.7.15: 100%
Message from erlang-runtime19-21.3.8.2:

===========================================================================

To use this runtime port for development or testing, just prepend
its binary path ("/usr/local/lib/erlang19/bin") to your PATH variable.

===========================================================================

(...)

// 对于另一个 rabbit2 Jail，同样执行相同的操作。 //
```

让我们验证包 RabbitMQ 是否安装成功。

```sh
host # for I in 1 2; do jexec rabbit${I} which rabbitmqctl; done
/usr/local/sbin/rabbitmqctl
/usr/local/sbin/rabbitmqctl
```

# RabbitMQ 设置

接下来我们将在 Jails 中配置 **/etc/hosts** 文件。

```sh
host # for I in 1 2; do cat >> /jail/rabbit${I}/etc/hosts << __EOF
192.168.43.101 rabbit1
192.168.43.102 rabbit2

__EOF
done
```

……以及快速验证。

```sh
host # cat /jail/rabbit?/etc/hosts | grep 192.168.43 | sort -n | uniq -c
2 192.168.43.101 rabbit1
2 192.168.43.102 rabbit2
```

由于我们已经安装了包 RabbitMQ，现在需要启用并启动它。

```sh
host # jexec rabbit1 /usr/local/etc/rc.d/rabbitmq rcvar
# rabbitmq
#
rabbitmq_enable="NO"
#   (default: "")
```

如我们所见，需要在每个 Jail 的 **/etc/rc.conf** 文件中设置 **rabbitmq_enable=YES**。

```sh
host # for I in 1 2; do jexec rabbit${I} sysrc rabbitmq_enable=YES; done
rabbitmq_enable:  -> YES
rabbitmq_enable:  -> YES
```

现在我们可以在 Jail 中启动 RabbitMQ 了。

```sh
host # for I in 1 2; do jexec rabbit${I} service rabbitmq start; done
Starting rabbitmq.
Starting rabbitmq.
```

现在我们有四个 RabbitMQ 实例已经启动并运行。

默认启用的插件列表：无。

## RabbitMQ 插件

```sh
rabbit1 # rabbitmq-plugins list
 Configured: E = explicitly enabled; e = implicitly enabled
 | Status: * = running on rabbit@rabbit1
 |/
[  ] rabbitmq_amqp1_0                  3.7.15
[  ] rabbitmq_auth_backend_cache       3.7.15
[  ] rabbitmq_auth_backend_http        3.7.15
[  ] rabbitmq_auth_backend_ldap        3.7.15
[  ] rabbitmq_auth_mechanism_ssl       3.7.15
[  ] rabbitmq_consistent_hash_exchange 3.7.15
[  ] rabbitmq_event_exchange           3.7.15
[  ] rabbitmq_federation               3.7.15
[  ] rabbitmq_federation_management    3.7.15
[  ] rabbitmq_jms_topic_exchange       3.7.15
[  ] rabbitmq_management               3.7.15
[  ] rabbitmq_management_agent         3.7.15
[  ] rabbitmq_mqtt                     3.7.15
[  ] rabbitmq_peer_discovery_aws       3.7.15
[  ] rabbitmq_peer_discovery_common    3.7.15
[  ] rabbitmq_peer_discovery_consul    3.7.15
[  ] rabbitmq_peer_discovery_etcd      3.7.15
[  ] rabbitmq_peer_discovery_k8s       3.7.15
[  ] rabbitmq_random_exchange          3.7.15
[  ] rabbitmq_recent_history_exchange  3.7.15
[  ] rabbitmq_sharding                 3.7.15
[  ] rabbitmq_shovel                   3.7.15
[  ] rabbitmq_shovel_management        3.7.15
[  ] rabbitmq_stomp                    3.7.15
[  ] rabbitmq_top                      3.7.15
[  ] rabbitmq_tracing                  3.7.15
[  ] rabbitmq_trust_store              3.7.15
[  ] rabbitmq_web_dispatch             3.7.15
[  ] rabbitmq_web_mqtt                 3.7.15
[  ] rabbitmq_web_mqtt_examples        3.7.15
[  ] rabbitmq_web_stomp                3.7.15
[  ] rabbitmq_web_stomp_examples       3.7.15
```

现在是时候启用 Web 界面插件了。

```sh
host # for I in 1 2; do jexec rabbit${I} rabbitmq-plugins enable rabbitmq_management; done
The following plugins have been configured:
  rabbitmq_management
  rabbitmq_management_agent
  rabbitmq_web_dispatch
Applying plugin configuration to rabbit@rabbit1...
The following plugins have been enabled:
  rabbitmq_management
  rabbitmq_management_agent
  rabbitmq_web_dispatch

started 3 plugins.

(...)

// 对于另一个 rabbit2 Jail，同样启用 Web 界面插件。 //
```


现在我们已经在每个 RabbitMQ FreeBSD Jail 中启用了 Web 界面插件。

大写的 ‘**E**’ 表示这是我们主动启用的插件，而小写的 ‘**e**’ 表示该插件仅作为其他我们请求启用的插件的依赖而被启用。

```sh
rabbit1 # rabbitmq-plugins list
 Configured: E = explicitly enabled; e = implicitly enabled
 | Status: * = running on rabbit@rabbit1
 |/
[  ] rabbitmq_amqp1_0                  3.7.15
[  ] rabbitmq_auth_backend_cache       3.7.15
[  ] rabbitmq_auth_backend_http        3.7.15
[  ] rabbitmq_auth_backend_ldap        3.7.15
[  ] rabbitmq_auth_mechanism_ssl       3.7.15
[  ] rabbitmq_consistent_hash_exchange 3.7.15
[  ] rabbitmq_event_exchange           3.7.15
[  ] rabbitmq_federation               3.7.15
[  ] rabbitmq_federation_management    3.7.15
[  ] rabbitmq_jms_topic_exchange       3.7.15
[E*] rabbitmq_management               3.7.15
[e*] rabbitmq_management_agent         3.7.15
[  ] rabbitmq_mqtt                     3.7.15
[  ] rabbitmq_peer_discovery_aws       3.7.15
[  ] rabbitmq_peer_discovery_common    3.7.15
[  ] rabbitmq_peer_discovery_consul    3.7.15
[  ] rabbitmq_peer_discovery_etcd      3.7.15
[  ] rabbitmq_peer_discovery_k8s       3.7.15
[  ] rabbitmq_random_exchange          3.7.15
[  ] rabbitmq_recent_history_exchange  3.7.15
[  ] rabbitmq_sharding                 3.7.15
[  ] rabbitmq_shovel                   3.7.15
[  ] rabbitmq_shovel_management        3.7.15
[  ] rabbitmq_stomp                    3.7.15
[  ] rabbitmq_top                      3.7.15
[  ] rabbitmq_tracing                  3.7.15
[  ] rabbitmq_trust_store              3.7.15
[e*] rabbitmq_web_dispatch             3.7.15
[  ] rabbitmq_web_mqtt                 3.7.15
[  ] rabbitmq_web_mqtt_examples        3.7.15
[  ] rabbitmq_web_stomp                3.7.15
[  ] rabbitmq_web_stomp_examples       3.7.15
```

现在——为了创建集群——我们需要这些 RabbitMQ 实例共享相同的 ERLANG cookie。在 FreeBSD 系统上，ERLANG cookie 位于 **/var/db/rabbitmq/.erlang.cookie**。

```sh
rabbot1 # cat /var/db/rabbitmq/.erlang.cookie; echo
NOEVQNXJDNLAJOSVWNIW
rabbot1 #
```

我们需要先停止 RabbitMQ，以便更改 ERLANG cookie。

```sh
host # for I in 1 2; do jexec rabbit${I} service rabbitmq stop; done
Stopping rabbitmq.
Waiting for PIDS: 88684.
Stopping rabbitmq.
Waiting for PIDS: 20976.
```

接下来在每个 FreeBSD Jail 上设置相同的 ERLANG cookie。

```sh
host # for I in 1 2; do cat > /jail/rabbit${I}/var/db/rabbitmq/.erlang.cookie << __EOF
RABBITMQFREEBSDJAILS
__EOF
done
```

……现在我们需要再次启动它们。

```sh
host # for I in 1 2; do jexec rabbit${I} service rabbitmq start; done
Starting rabbitmq.
Starting rabbitmq.
```

快速验证一下。

```sh
host # for I in 1 2; do jexec rabbit${I} cat /var/db/rabbitmq/.erlang.cookie; done
RABBITMQFREEBSDJAILS
RABBITMQFREEBSDJAILS
```

## RabbitMQ 管理用户

现在我们将在 RabbitMQ 实例中创建一个名为 **admin** 的管理用户。

```sh
host # for I in 1 2; do jexec rabbit${I} rabbitmqctl add_user admin ADMINPASSWORD; done
Adding user "admin" ...
Adding user "admin" ...

host # for I in 1 2; do jexec rabbit${I} rabbitmqctl set_user_tags admin administrator; done
Setting tags for user "admin" to [administrator] ...
Setting tags for user "admin" to [administrator] ...

host # for I in 1 2; do jexec rabbit${I} rabbitmqctl set_permissions -p / admin ".*" ".*" ".*" ; done
Setting permissions for user "admin" in vhost "/" ...
Setting permissions for user "admin" in vhost "/" ...
```

我们现在应该可以登录 **[http://192.168.43.101:15672/](http://192.168.43.101:15672/)**（或者 **[http://10.0.0.101:15672/](http://10.0.0.101:15672/)**）的 RabbitMQ 管理页面了。

![01-rabbitmq-login.png](https://vermaden.wordpress.com/wp-content/uploads/2019/06/01-rabbitmq-login.png?w=960)

登录后，一个实用的 RabbitMQ 仪表板将显示给你。

![02-rabbitmq-dashboard.png](https://vermaden.wordpress.com/wp-content/uploads/2019/06/02-rabbitmq-dashboard.png?w=960)

## RabbitMQ 集群设置

接下来我们将创建 RabbitMQ 集群。

```sh
rabbit1 # rabbitmqctl cluster_status
Cluster status of node rabbit@rabbit1 ...
[{nodes,[{disc,[rabbit@rabbit1]}]},
 {running_nodes,[rabbit@rabbit1]},
 {cluster_name,},
 {partitions,[]},
 {alarms,[{rabbit@rabbit1,[]}]}]

rabbit2 # hostname
rabbit2.local

rabbit2 # rabbitmqctl join_cluster rabbit@rabbit1
Error: this command requires the 'rabbit' app to be stopped on the target node. Stop it with 'rabbitmqctl stop_app'.
Arguments given:
        join_cluster rabbit@rabbit1

Usage

rabbitmqctl [--node ] [--longnames] [--quiet] join_cluster [--disc|--ram]
```

首先，我们需要停止 RabbitMQ 的“应用程序”，以便加入集群。

```sh
rabbit2 # rabbitmqctl stop_app
Stopping rabbit application on node rabbit@rabbit2 ...

rabbit2 # rabbitmqctl join_cluster rabbit@rabbit1
Clustering node rabbit@rabbit2 with rabbit@rabbit1

rabbit2 # rabbitmqctl start_app
Starting node rabbit@rabbit2 ...
 completed with 5 plugins.

rabbit2 # rabbitmqctl cluster_status
Cluster status of node rabbit@rabbit2 ...
[{nodes,[{disc,[rabbit@rabbit1,rabbit@rabbit2]}]},
 {running_nodes,[rabbit@rabbit1,rabbit@rabbit2]},
 {cluster_name,},
 {partitions,[]},
 {alarms,[{rabbit@rabbit1,[]},{rabbit@rabbit2,[]}]}]

rabbit1 # rabbitmqctl cluster_status
Cluster status of node rabbit@rabbit1 ...
[{nodes,[{disc,[rabbit@rabbit1,rabbit@rabbit2]}]},
 {running_nodes,[rabbit@rabbit2,rabbit@rabbit1]},
 {cluster_name,},
 {partitions,[]},
 {alarms,[{rabbit@rabbit2,[]},{rabbit@rabbit1,[]}]}]
```

现在我们已经形成了一个两节点的 RabbitMQ 集群。接下来我们将其重命名为 **cluster**。

```sh
rabbit1 # rabbitmqctl set_cluster_name rabbit@cluster
Setting cluster name to rabbit@cluster ...

rabbit1 # rabbitmqctl cluster_status
Cluster status of node rabbit@rabbit1 ...
[{nodes,[{disc,[rabbit@rabbit1,rabbit@rabbit2]}]},
 {running_nodes,[rabbit@rabbit2,rabbit@rabbit1]},
 {cluster_name,},
 {partitions,[]},
 {alarms,[{rabbit@rabbit2,[]},{rabbit@rabbit1,[]}]}]
```

下面是在 Web 界面中查看我们集群的样子。

![08-rabbitmq-cluster.png](https://vermaden.wordpress.com/wp-content/uploads/2019/06/08-rabbitmq-cluster-1.png?w=960)

## RabbitMQ 高可用策略

要在 RabbitMQ 中实现 [高可用（镜像）队列](https://www.rabbitmq.com/ha.html)，需要创建一个 *Policy*（策略）。我们将声明一个名为 **ha** 的 *Policy*，它匹配名称以 **ha-** 前缀开头的队列，从而将这些队列配置为在集群中的两个节点上镜像。

创建该 *Policy* 的命令如下：

```
rabbit1 # rabbitmqctl set_policy ha "^ha-\.*" '{"ha-mode":"all","ha-sync-mode":"automatic"}'
Setting policy "ha-mirror" for pattern "^ha-\." to "{"ha-mode":"all","ha-sync-mode":"automatic"}" with priority "0" for vhost "/" ...
```

……或者，你也可以使用 Web 界面来创建该策略。

无论使用哪种方法，最终都会得到所需的 **ha** *Policy*，如下所示。

![03-rabbitmq-policy.png](https://vermaden.wordpress.com/wp-content/uploads/2019/06/03-rabbitmq-policy-1.png?w=960)

# 发送消息到队列

现在我们已经有了一个两节点的 RabbitMQ 集群，并且为名称以 **ha-** 前缀开头的队列启用了高可用功能。接下来我们将测试 RabbitMQ 设置，使用 **send.go** 脚本创建并发送消息到队列——正如你可能猜到的，这个脚本是用 Go 语言编写的。我们需要在 **host** 系统上安装 Go 语言。

## 安装 Go 语言

```sh
host # pkg install go
Updating FreeBSD repository catalogue...
FreeBSD repository is up to date.
All repositories are up to date.
The following 1 package(s) will be affected (of 0 checked):

New packages to be INSTALLED:
        go: 1.12.5,1

Number of packages to be installed: 1

The process will require 262 MiB more space.
75 MiB to be downloaded.

Proceed with this action? [y/N]: y
(...)

host % go version
go version go1.12.5 freebsd/amd64
```

这是 **send.go** 脚本——我们将使用它向 **ha-default** 队列发送 10 条消息。它基于 [RabbitMQ Hello World](https://www.rabbitmq.com/tutorials/tutorial-one-go.html) 教程。

```go
host % cat send.go
package main

import (
  "log"
  "amqp"
)

func FAIL_ON_ERROR(err error, msg string) {
  if err != nil {
    log.Fatalf("%s: %s", msg, err)
  }
}

func main() {
  conn, err := amqp.Dial("amqp://admin:ADMINPASSWORD@10.0.0.101:5672/")
  FAIL_ON_ERROR(err, "ER: failed to connect to RabbitMQ")
  defer conn.Close()

  ch, err := conn.Channel()
  FAIL_ON_ERROR(err, "ER: failed to open channel")
  defer ch.Close()

  q, err := ch.QueueDeclare(
    "ha-default", // 队列名称
    false,        // 是否持久化
    false,        // 未使用时是否删除
    false,        // 是否排他
    false,        // 是否不等待
    nil,          // 参数
```
  )
  FAIL_ON_ERROR(err, "ER: failed to declare queue")

  body := "Hello World!"

  for i := 1; i <= 10; i++ {
    err = ch.Publish(
      "",     // exchange 交换
      q.Name, // routing key 路由键
      false,  // mandatory 强制
      false,  // immediate 立即
      amqp.Publishing{
        ContentType: "text/plain",
        Body:        []byte(body),
      })
    log.Printf("IN: sent message '%s' (%d)", body, i)
    FAIL_ON_ERROR(err, "ER: failed to publish message")
  }

}
```

接下来我们将运行它。

```sh
host % go run send.go
send.go:5:3: cannot find package "amqp" in any of:
        /usr/local/go/src/amqp (from $GOROOT)
        /home/vermaden/.gopkg/src/amqp (from $GOPATH)
```

我们缺少 Go 语言的 **amqp** 包。

需要从 [https://github.com/streadway/amqp](https://github.com/streadway/amqp) 页面下载。我们将通过下载整个 ZIP 包的方式获取它。

```sh
host % mkdir -p ~/.gopkg/src
host % cd !$
host % pwd
/home/vermaden/.gopkg/src
host % fetch https://github.com/streadway/amqp/archive/master.zip
host % unzip master.zip 
Archive:  /home/vermaden/.gopkg/src/master.zip
   creating: amqp-master/
 extracting: amqp-master/.gitignore
 extracting: amqp-master/.travis.yml
 (...)
 extracting: amqp-master/uri.go
 extracting: amqp-master/uri_test.go
 extracting: amqp-master/write.go
host % rm master.zip
host % mv amqp-master amqp
host % cd amqp
host % pwd
/home/vermaden/.gopkg/src/amqp
host % exa
_examples          confirms.go         delivery_test.go        LICENSE            spec091.go
spec               confirms_test.go    doc.go                  pre-commit         tls_test.go
allocator.go       connection.go       example_client_test.go  read.go            types.go
allocator_test.go  connection_test.go  examples_test.go        read_test.go       uri.go
auth.go            consumers.go        fuzz.go                 README.md          uri_test.go
certs.sh           consumers_test.go   gen.sh                  reconnect_test.go  write.go
channel.go         CONTRIBUTING.md     go.mod                  return.go          
client_test.go     delivery.go         integration_test.go     shared_test.go     
```

我们还需要确保 **PATH** 和 **GOPATH** 配置正确。为此，需要将它们写入你的交互式 shell 配置文件中。

```sh
# GO SHELL SETUP
mkdir -p ~/.gopkg
export GOPATH=~/.gopkg
export PATH="${PATH}:~/.gopkg"
```

现在我们可以继续向队列发送消息了。

```sh
host % go run send.go
2019/06/05 13:53:59 IN: sent message 'Hello World!' (1)
2019/06/05 13:53:59 IN: sent message 'Hello World!' (2)
2019/06/05 13:53:59 IN: sent message 'Hello World!' (3)
2019/06/05 13:53:59 IN: sent message 'Hello World!' (4)
2019/06/05 13:53:59 IN: sent message 'Hello World!' (5)
2019/06/05 13:53:59 IN: sent message 'Hello World!' (6)
2019/06/05 13:53:59 IN: sent message 'Hello World!' (7)
2019/06/05 13:53:59 IN: sent message 'Hello World!' (8)
2019/06/05 13:53:59 IN: sent message 'Hello World!' (9)
2019/06/05 13:53:59 IN: sent message 'Hello World!' (10)
%
```

**ha-default** 队列已创建并发送了 10 条消息。

![04-rabbitmq-queue](https://vermaden.wordpress.com/wp-content/uploads/2019/06/04-rabbitmq-queue-1.png?w=960)

现在我们需要从队列中“接收”这些消息，这时 **receive.go** 脚本派上用场。它同样基于 [RabbitMQ Hello World](https://www.rabbitmq.com/tutorials/tutorial-one-go.html) 教程。

```go
host % cat receive.go
package main

import (
  "log"
  "amqp"
)

func FAIL_ON_ERROR(err error, msg string) {
  if err != nil {
    log.Fatalf("%s: %s", msg, err)
  }
}

func main() {
  conn, err := amqp.Dial("amqp://admin:ADMINPASSWORD@10.0.0.102:5672/")
  FAIL_ON_ERROR(err, "ER: failed to connect to RabbitMQ")
  defer conn.Close()

  ch, err := conn.Channel()
  FAIL_ON_ERROR(err, "ER: failed to open channel")
  defer ch.Close()

  q, err := ch.QueueDeclare(
    "ha-default", // 队列名称
    false,        // 是否持久化
    false,        // 未使用时是否删除
    false,        // 是否排他
    false,        // 是否不等待
    nil,          // 参数
  )
  FAIL_ON_ERROR(err, "ER: failed to declare queue")

  msgs, err := ch.Consume(
    q.Name, // 队列
    "",     // 消费者
    true,   // 自动确认
    false,  // 是否排他
    false,  // no-local
    false,  // 是否不等待
    nil,    // 参数
  )
  FAIL_ON_ERROR(err, "ER: failed to register consumer")

  forever := make(chan bool)

  go func() {
    for d := range msgs {
      log.Printf("IN: received message: %s", d.Body)
    }
  }()

  log.Printf("IN: waiting for messages")
  log.Printf("IN: to exit press CTRL+C")
  <-forever
}
```


这是运行后的输出。该程序会一直运行，直到你使用 **CTRL-C** 组合键手动结束它。

```sh
host % go run receive.go
2019/06/05 13:54:34 IN: waiting for messages
2019/06/05 13:54:34 IN: to exit press CTRL+C
2019/06/05 13:54:34 IN: received message: Hello World!
2019/06/05 13:54:34 IN: received message: Hello World!
2019/06/05 13:54:34 IN: received message: Hello World!
2019/06/05 13:54:34 IN: received message: Hello World!
2019/06/05 13:54:34 IN: received message: Hello World!
2019/06/05 13:54:34 IN: received message: Hello World!
2019/06/05 13:54:34 IN: received message: Hello World!
2019/06/05 13:54:34 IN: received message: Hello World!
2019/06/05 13:54:34 IN: received message: Hello World!
2019/06/05 13:54:34 IN: received message: Hello World!
^C
%
```

如果你仔细查看源码，你可能已经注意到，我是在 **rabbit1** 节点（**10.0.0.101**）“发送”消息，而在 **rabbit2** 节点（**10.0.0.102**）“接收”这些消息的。

## 简单基准测试

接下来我们将进行一个简单的基准测试：保持 **receive.go** 脚本运行，同时修改 **send.go** 脚本的 **for** 循环，发送 **100000** 条消息。

```sh
host % go run receive.go
2019/06/05 13:52:34 IN: waiting for messages
2019/06/05 13:52:34 IN: to exit press CTRL+C
```

……现在开始发送消息。


```sh
host % go run send.go
2019/06/05 13:53:59 IN: sent message 'Hello World!' (1)
2019/06/05 13:53:59 IN: sent message 'Hello World!' (2)
2019/06/05 13:53:59 IN: sent message 'Hello World!' (3)
(...)
2019/06/05 13:56:26 IN: sent message 'Hello World!' (99998)
2019/06/05 13:56:26 IN: sent message 'Hello World!' (99999)
2019/06/05 13:56:26 IN: sent message 'Hello World!' (100000)
%
```

这个简单基准测试的结果如下。

![05-rabbitmq-benchmark.png](https://vermaden.wordpress.com/wp-content/uploads/2019/06/05-rabbitmq-benchmark-1.png?w=960)

在两个 FreeBSD Jails 内，这个 RabbitMQ 集群实例大约可以处理每秒 4000-5000 条消息。

# 高可用性测试

现在我们将测试 RabbitMQ 集群的高可用性。

目前 **ha-default** 队列在 **rabbit1** 节点上。接下来我们将停止 **rabbit1** Jail，观察 RabbitMQ Web 界面的反应。

```sh
host # jls
   JID  IP Address      Hostname                      Path
     1  192.168.43.101  rabbit1.local                 /jail/rabbit1
     2  192.168.43.102  rabbit2.local                 /jail/rabbit2

host # killall -9 -j 1

host # umount /jail/rabbit1/dev
```

我们的 **ha-default** 队列在几秒钟内切换到了 **rabbit2** 节点 —— 高可用功能按预期工作。

![06-rabbitmq-ha-node-fail.png](https://vermaden.wordpress.com/wp-content/uploads/2019/06/06-rabbitmq-ha-node-fail-1.png?w=960)

接下来启动 **rabbit1** Jail，以恢复冗余。

```sh
host # service jail onestart rabbit1
Starting jails: rabbit1.
host #
```
![07-rabbitmq-ha-node-back.png](https://vermaden.wordpress.com/wp-content/uploads/2019/06/07-rabbitmq-ha-node-back-1.png?w=960)

**ha-default** 队列恢复了冗余，显示 **+1** 标记，但仍然位于 **rabbit2** 节点。

……最后一点小庆祝——这是我博客的第 50 篇文章（不包含 **Valuable News** 系列） :🙂:

## 更新 1 – 本月 RabbitMQ 动态

文章 [RabbitMQ Cluster on FreeBSD Containers](https://vermaden.wordpress.com/2019/06/05/rabbitmq-cluster-on-freebsd-containers/) 被收录在 [This Month in RabbitMQ – July 2019](https://www.rabbitmq.com/blog/2019/07/09/this-month-in-rabbitmq-july-2019/) 中。

感谢提及！

## 更新 2 – 降低 RabbitMQ CPU 使用率

如 **Felix Ehlers** 在 [Twitter](https://twitter.com/FelixEhlers/status/1157769492870717440) 所报告，设置变量 **RABBITMQ_SERVER_ADDITIONAL_ERL_ARGS="+sbwt none"** ，可以降低 RabbitMQ 的 CPU 使用率。
