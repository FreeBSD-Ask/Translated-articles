# zfs 速查手册

- 原文链接：[OpenZFS Cheat Sheet](https://freebsdfoundation.org/blog/openzfs-cheat-sheet/)
- 作者：Benedict Reuschling (benedict@reuschling.org)
- 原发布时间：2025 年 3 月

![](https://github.com/user-attachments/assets/5e63f6dd-1e88-40d8-af30-dbfc9f83c56d)


### 存储池配置  

**单个磁盘**  
无冗余，挂载为 `/mypool`，无需在 `/etc/fstab` 中添加条目。  
```sh
zpool create mypool disk1
```

**条带（RAID-0）**  
无冗余，任意一块磁盘损坏都会导致数据全部丢失。性能高，可用容量为所有磁盘总和。  
```sh
zpool create mystripe disk1 disk2
```

**镜像（RAID-1）**  
可承受一块磁盘损坏，VDEV 在 `zpool status` 输出中显示为 `mirror-0`，可并行读取所有磁盘，写入速度较慢，总容量低于 RAID-0。  
```sh
zpool create mymirror mirror disk1 disk2
```

**单奇偶校验（RAID-Z1）**  
至少需要 3 块磁盘，可承受 1 块磁盘损坏，奇偶校验信息分布在所有磁盘上，读写速度较快，性能相当，容量约为 66%。在 `zpool status` 输出中显示为 `raidz1-0` VDEV。  
```sh
zpool create paritypool raidz disk1 disk2 disk3
```

**双奇偶校验（RAID-Z2）**  
每个 VDEV 可承受 2 块磁盘损坏，比 RAID-Z1 慢，至少需要 4 块磁盘，可用容量约为 50%。  
```sh
zpool create myraidz2 raidz2 disk1 disk2 disk3 disk4
```

**RAID10**  
至少需要 4 块磁盘，每个 VDEV 可承受 1 块磁盘损坏，读速快，写速为单个磁盘的一半，容量为 50%。是冗余性、容量和性能的良好折中方案。  
```sh
zpool create myr10 mirror disk1 disk2 mirror disk3 disk4
```

**三奇偶校验（RAID-Z3）**  
比 RAID-Z2 稍慢，每个 VDEV 可承受 3 块磁盘损坏，至少需要 5 块磁盘，可用容量约为 40%。  
```sh
zpool create myz3 raidz3 disk1 disk2 disk3 disk4 disk5
```

### 显示存储池状态  

**显示存储池状态**【包括磁盘配置、错误信息、适用的更新、上次 scrub 时间（若有）】
```sh
zpool status
```

**显示存储池容量、已用空间和可用空间**  
```sh
zpool list
```

**显示存储池 I/O 统计信息**  
```sh
zpool iostat
```
选项：
- `-v` 显示各个设备  
- `-w` 显示 I/O 延迟  
- `-r` 显示请求大小直方图  
- `-l` 显示等待时间统计  
- `-q` 显示队列统计  

**显示存储池的管理命令历史**  
```sh
zpool history
```
选项：
- `-l` 详细格式  
- `-i` 仅显示事务组等事件  

**获取存储池属性及其值，并可使用 `zpool set` 修改默认值**  
```sh
zpool get
```

### 特殊 VDEV  

#### **二级 ARC（L2ARC）**  
当数据不再适合存储在 ZFS 主内存缓存（ARC：自适应替换缓存）时，L2ARC 充当快速读取缓存。读取请求将由 L2ARC 处理，需使用高速存储设备（如闪存）才能获得显著效果。使用 `cache` 关键字将设备添加到存储池。  
```sh
zpool add mypool cache /dev/nda0
```

#### **ZFS 意图日志（ZFS intent log, ZIL）**  
将同步写入转换为异步写入（不影响读取），使应用程序能更快确认数据已写入，类似于数据库事务日志。当写入完成并存储到底层存储介质后，ZIL 将被清除。ZIL 需要快速存储设备，但无需大容量。使用关键字 `log` 将设备添加到存储池。  
```sh
zpool add mypool log /dev/nda1
```

#### **备用盘（Spare）**  
备用盘在替换故障磁盘之前不会参与 I/O 操作，可以由手动操作或外部故障管理软件触发替换。使用关键字 `spare` 添加备用盘。  
```sh
zpool add mypool spare /dev/nda2
```

---

### **存储池扩展**  

#### **将单磁盘存储池转换为 RAID1**  
为单磁盘存储池设备 `/dev/nda0` 添加镜像磁盘 `/dev/nda1`：  
```sh
zpool attach mypool mirror /dev/nda0 /dev/nda1
```

#### **替换故障磁盘**  
将存储池中的故障磁盘 `/dev/nda2` 替换为设备 `/dev/nda3`：  
```sh
zpool replace mypool /dev/nda2 /dev/nda3
```

---

### **存储池导入/导出**  

#### **导出存储池**  
完成所有 I/O 操作后，从文件系统层次结构中卸载存储池。可在其他系统上导入以恢复存储池状态。  
```sh
zpool export mypool
```

#### **导入存储池**  
将存储池导入当前系统。  

- **扫描 ZFS 存储池标识**  
  ```sh
  zpool import
  ```
- **通过 ID 或名称导入存储池**  
  ```sh
  zpool import mypool
  ```
- **重命名存储池**  
  ```sh
  zpool import oldname newname
  ```
- **在不同路径下挂载已导入的存储池（防止覆盖现有存储池）**  
  ```sh
  zpool import -R /media mypool
  ```
- **以只读模式导入存储池**  
  ```sh
  zpool import -o readonly mypool
  ```
- **尝试恢复最近销毁的存储池**  
  ```sh
  zpool import -D mypool
  ```

---

### **ZFS 数据集**  

数据集位于存储池之上，并占用其空间。访问方式如下：`mypool/data`。每个存储池创建时，都会默认生成一个与其同名的顶级数据集。例如，执行以下命令：  
```sh
zpool create test disk1
```
将创建一个名为 `test` 的存储池和数据集，顶级数据集将挂载为 `/test`，并可包含子数据集。  

#### **创建数据集**  
创建一个名为 `ds` 的新数据集，大部分属性继承自父数据集：  
```sh
zfs create mypool/ds
```
创建多个数据集：  
```sh
zfs create -p mypool/home/fred
```

---

### **显示数据集信息**  

**列出已用空间、可用空间和挂载点**  
```sh
zfs list
```

**按 ZFS 路径显示数据集**  
```sh
zfs list mypool/ds
```

**列出数据集及其所有子数据集**  
```sh
zfs list -r mypool/ds
```

**限制显示数据集的层级深度**  
```sh
zfs list -d 1 mypool/home
```

**仅显示数据集名称**  
```sh
zfs list -o name mypool/home
```

**移除输出头部**  
```sh
zfs list -Ho name mypool/home
```

**修改显示顺序**  
```sh
zfs list -o used,avail,refer,name
```

**按列排序**  
```sh
zfs list -rs refer mypool/home
```

**倒序排列**  
```sh
zfs list -rS refer mypool/home
```

### **显示存储池可用空间**  
列出每个数据集及其子数据集的可用空间和已用空间，包括快照和任何预留空间：  
```sh
zfs list -o space
```
**推荐**：使用该命令代替 `df -h`。  

---

### **重命名数据集**  
重命名数据集或在存储池层次结构内移动数据集，类似于 Unix `mv` 命令：  
```sh
zfs rename mypool/home/fred mypool/home/eva
```

---

### **销毁数据集**  
**模拟销毁**（不会执行实际删除）：  
```sh
zfs destroy -nv mypool/old
```
**输出**：  
```
would destroy mypool/old
```

**执行销毁并显示结果**：  
```sh
zfs destroy -v mypool/old
```
**输出**：  
```
will destroy mypool/old
```

**递归销毁数据集及其子数据集**：  
```sh
zfs destroy -r mypool/testdata
```

---

### **属性管理**  

数据集通过继承父数据集的大部分属性来提高灵活性。子数据集可根据需要覆盖这些属性。仅可更改 `SOURCE` 列中的默认属性，存储池的属性规则相同。  

#### **显示属性**  
多种方式可以查看存储池和数据集的属性：  

- **查看存储池的所有属性**  
  ```sh
  zpool get all mypool
  ```
- **查看数据集的所有属性**  
  ```sh
  zfs get all mypool/dataset
  ```
- **查看单个属性（容量）**  
  ```sh
  zpool get capacity mypool
  ```
- **查看多个属性（容量与健康状态）**  
  ```sh
  zpool get capacity,health mypool
  ```

#### **修改属性**  
- **禁用访问时间更新**（可提高性能）：  
  ```sh
  zfs set atime=off mypool
  ```
- **更改挂载点**：  
  ```sh
  zfs set mountpoint=/media mypool/ds
  ```

#### **自定义属性**  
可为数据集定义自定义属性（`key=value`），数据集会继承这些属性，但可更改其值。  

- **创建自定义属性**：  
  ```sh
  zfs set warranty:expires=2048/04/20 mypool
  ```
- **列出自定义属性**：  
  ```sh
  zfs get warranty:expires mypool
  ```
- **重置属性值**（继承父级值）：  
  ```sh
  zfs inherit warranty:expires mypool
  ```
- **删除自定义属性**（递归删除）：  
  ```sh
  zfs inherit -r warranty:expires mypool
  ```

---

### **数据完整性检查（Scrub）**  

ZFS 在数据存储时会计算并存储校验和，覆盖整个数据集层次结构。I/O 错误、驱动故障、内存损坏、损坏的电缆等因素可能导致校验和不匹配。  

如果存储池具有足够的冗余（如镜像或 RAID-Z），ZFS 可以检测到这些错误并自动修复（**自我修复**）。  

建议**每月运行一次 Scrub**，重新计算校验和。如果发现数据损坏，ZFS 会从冗余 VDEV 获取正确的数据并修正错误。  

**启动 Scrub**：  
```sh
zpool scrub mypool
```

**查看 Scrub 进度和错误信息**：  
```sh
zpool status
```

**ZFS 无需 `fsck`**，因为 ZFS 自带数据完整性检查机制。

### **卷（Volumes）**  

ZFS 卷使用连续的存储池空间，并可通过 iSCSI 在网络上导出。当使用非 ZFS 文件系统格式化卷时，该文件系统仍可自动利用所有 ZFS 的底层功能。  

---

### **创建卷**  

**创建一个 10GB 的 ZFS 卷**：  
```sh
zfs create -V 10G mypool/vol1
```

---

### **创建稀疏卷（Sparse Volume）**  

稀疏卷是**超分配**的卷，不会立即占用所有预留空间，而是随着数据的增长逐步填充。  

**创建一个 1PB（1 PB）稀疏卷**：  
```sh
zfs create -V 1P -s mypool/sparse1PBvol
```

### **配额（Quota）**  

每个数据集默认可以使用整个存储池的空间。配额用于限制数据集的最大存储量。ZFS **严格** 执行配额限制，任何超出配额的写入都会被阻止。  

---

### **定义配额**  

ZFS **默认无配额**。可使用 `zfs set` 命令为特定数据集设置配额：  

```sh
zfs set quota=10G mypool/dataset
```
该配额适用于该数据集及其所有子数据集（包括未来创建的子数据集）。这些数据集**共享** 配额总量。  

如果仅想限制**父数据集本身**，不影响子数据集，则使用 `refquota`：  

```sh
zfs set refquota=10G mypool/dataset
```

#### **用户和组配额**  
- **限制特定用户的存储空间**：
  ```sh
  zfs set userquota@fred=10G mypool/home/fred
  ```
- **限制某个用户组的存储空间**：
  ```sh
  zfs set groupquota@projectX=100G mypool/projectX
  ```

---

### **显示配额信息**  

- **查询数据集的配额**：
  ```sh
  zfs get quota mypool/dataset
  ```
- **查询 `refquota`**：
  ```sh
  zfs get refquota mypool/home/fred
  ```
- **查询用户配额**：
  ```sh
  zfs userspace mypool/home/fred
  ```
- **查询组配额**：
  ```sh
  zfs groupspace mypool/projectX
  ```

---

### **移除配额**  

将数据集的配额取消（`refquota` 也同样适用）：  

```sh
zfs set quota=none mypool/home/fred
```

---

### **预留空间（Reservation）**  

预留（Reservation）确保存储池中一定量的空间始终可用，不受其他数据集占用。这减少了存储池的可用空间，但可以防止单个数据集占满整个池，有助于容量规划。  

#### **定义预留空间**  

使用 `reservation` 属性设置数据集的预留空间：  
```sh
zfs set reservation=100G mypool/home
```
预留空间**会被子数据集继承**。如果只想对该数据集本身生效，而不影响子数据集，则使用 `refreservation`：  
```sh
zfs set refreservation=10G mypool/home/eve
```

#### **显示预留空间**  

- **查看数据集的预留空间**：  
  ```sh
  zfs get reservation mypool/home/eve
  ```
- **查看 `USEDREFRESERV` 列中的预留空间**：  
  ```sh
  zfs list -o space
  ```

#### **移除预留空间**  
将 `reservation` 或 `refreservation` 设为 `none`：  
```sh
zfs set reservation=none mypool/home/eve
```

---

### **快照（Snapshots）**  

快照提供了一种**快速保存数据集只读状态**的方法，可用于恢复数据集到特定时间点的状态。无需回滚整个快照，也可以单独恢复其中的文件。  

**ZFS 通过 `.zfs` 目录提供只读访问**，可从快照中读取文件。  

---

### **创建快照**  

- **创建快照**：  
  ```sh
  zfs snapshot mypool/ds@mysnapshot
  ```
- **简写命令**：  
  ```sh
  zfs snap mypool/ds@mysnapshot
  ```
- **递归创建快照（包含所有子数据集）**：  
  ```sh
  zfs snap -r mypool/ds@mysnapshot
  ```

---

### **显示快照**  

- **列出所有快照**：  
  ```sh
  zfs list -t snap mypool/ds
  ```
- **列出数据集及其所有子数据集的快照**：  
  ```sh
  zfs list -rt snap mypool/ds
  ```

---

### **显示自上次快照以来的写入数据量**  

- `written` 属性显示自上次快照以来写入的数据量。  
- 配合 `used` 和 `referenced` 属性可以直观查看存储池空间使用情况：  

```sh
zfs list -rt all -o name,used,refer,written mypool
```

---

### **比较快照**  

使用 `zfs diff` 比较当前数据集与某个快照之间的更改：  

```sh
zfs diff mypool/ds@backup
```

**输出说明**：

| 符号 | 说明 |
|:------:|:------|
| `+` | 新增文件 |
| `-` | 删除文件 |
| `M` | 修改文件 |
| `R` | 重命名文件（应用于目录时表示元数据更改） |

对比 **两个快照** 之间的更改：  

```sh
zfs diff mypool/ds@backup1 mypool/ds@backup2
```

### **快照回滚（Snapshot Rollback）**  

回滚操作会**丢弃当前数据集的状态**，恢复到指定快照的状态。所有在该快照之后创建的数据**都会被删除**。  

```sh
zfs rollback mypool/ds@backup2
```

如果要回滚到更早的快照，**必须先使用选项 `-r` 删除所有中间快照**：  

```sh
zfs rollback -r mypool/ds@backup1
```

---

### **挂载快照（Snapshot Mounting）**  

可将快照**以只读方式**挂载到文件系统：  

```sh
mount -t zfs mypool/ds@backup /mnt/backup
```

---

### **删除快照（Snapshot Deletion）**  

#### **模拟删除（Dry Run）**  

建议先执行**模拟删除**（选项 `-n`）查看即将删除的内容，配合 `-v` 显示详细信息：  

```sh
zfs destroy -vn mypool/ds@backup
```

如果确认无误，去掉 `-n` 选项执行删除：  

```sh
zfs destroy -v mypool/ds@backup
```

#### **递归删除快照**  

删除指定快照及其所有子快照：  

```sh
zfs destroy -rv mypool/ds@backup
```

#### **删除快照范围（Delete Range）**  

假设快照列表如下：
```sh
mypool@a  
mypool@b  
mypool@c  
mypool@d  
mypool@e  
```

- **删除快照 `@b` 至 `@d`（含 `@d`）**：
  ```sh
  zfs destroy -v mypool@b%d
  ```
- **删除 `@b` 及其后所有快照**：
  ```sh
  zfs destroy -v mypool@b%
  ```
- **删除 `@b` 之前的所有快照**：
  ```sh
  zfs destroy -v mypool%@b
  ```

---

### **快照保护（ZFS Holds）**  

ZFS 能为快照创建**保护标记**（tag），被标记的快照不能删除。可以对一个快照添加**多个**保护标记，只有 **所有标记都被移除** 后，快照才可以删除。  

#### **创建快照保护（Hold）**  

```sh
zfs hold keepme mypool/home@important
```

#### **递归列出快照的所有 Holds**  

```sh
zfs holds -r mypool/home@important
```

#### **解除快照保护（Release Hold）**  

```sh
zfs release keepme mypool/home@important
```

---

### **克隆（Clones）**  

克隆是**可写的快照副本**，创建的克隆数据集**包含原快照的所有数据**。  

#### **创建克隆**  

```sh
zfs clone mypool/ds@backup mypool/myclone
```

克隆数据集的 `origin` 属性指向其**来源快照**：  

```sh
zfs get origin mypool/myclone
```

#### **解除克隆对原快照的依赖（Promote Clone）**  

ZFS **不允许删除** 有依赖克隆的快照。要删除原快照，必须**先提升克隆为独立数据集**（`promote`）：  

```sh
zfs promote mypool/myclone
```

此时：
- `origin` 属性消失
- 该快照成为克隆的新基础
- 原快照可安全删除  

#### **删除克隆（Clone Removal）**  

可像删除普通数据集一样删除克隆：  

```sh
zfs destroy mypool/myclone
```

### **加密（Encryption）**  

ZFS 支持数据集加密，每个数据集可以有独立的密钥。一些元数据仍然保持未加密状态，以便执行 **scrub** 等操作。  

#### **创建加密数据集（Create Encrypted Dataset）**  

设置数据集的加密密码：  

```sh
zfs create -o encryption=on -o keyformat=passphrase -o keylocation=prompt mypool/secret
```

#### **查看加密状态（Load Status）**  

获取密钥状态：  

```sh
zfs get keystatus mypool/secret
```

#### **加载密钥（Load Key）**  

解锁数据集：  

```sh
zfs load-key mypool/secret
```

---

### **权限委派（Delegation）**  

ZFS 允许将 **ZFS 命令的权限** 委派给非 root 用户。  

#### **授权用户（Delegate Permission to User）**  

授予用户 `joe` 对 `atime` 属性的修改权限：  

```sh
zfs allow -u joe atime mypool/dataset
```

#### **查看现有权限（Display Delegations）**  

```sh
zfs allow mypool/dataset
```

#### **撤销用户权限（Remove Delegation）**  

移除 `joe` 的 `compression` 权限：  

```sh
zfs unallow -u joe compression mypool/dataset
```

#### **授权给用户组（Delegate Permission to Group）**  

授予 `mygroup` 组 `atime` 属性权限：  

```sh
zfs allow -g mygroup atime mypool/dataset
```

#### **允许用户委派权限（Delegate Permission to Delegate）**  

授予 `jill` 允许其他用户管理数据集权限的权限：  

```sh
zfs allow -u jill allow mypool/dataset
```

#### **创建权限集合（Create a Set of Permissions）**  

创建权限集合 `@myset`，包含 `mount`、`snapshot`、`rollback` 和 `destroy` 权限：  

```sh
zfs allow -s @myset mount,snapshot,rollback,destroy mypool/dataset
```

#### **应用权限集合（Use Permission Set for Delegations）**  

将 `@myset` 赋予用户 `jill`：  

```sh
zfs allow -u jill @myset mypool/dataset
```

---

### **快照发送与接收（Sending and Receiving Snapshots）**  

ZFS 能以字节流的方式**本地和通过网络传输快照**，可用于备份。  

#### **将快照写入文件（Write Snapshot to a File）**  

```sh
zfs send mypool/ds@backup > dsbackup
```

要恢复快照，可使用 `zfs receive`，并通过 `-v` 选项显示详细信息：  

```sh
zfs send -v mypool/ds@backup > target
```

#### **从快照创建数据集（Create Dataset from Snapshot）**  

```sh
zfs recv mypool/backup < dsbackup
```

显示传输详细信息：  

```sh
zfs recv -v mypool/backup < dsbackup
```

#### **不使用中间文件进行复制（Replication Without Intermediary File）**  

直接通过管道传输快照数据：  

```sh
zfs send mypool/ds@backup | zfs recv mypool/new
```

#### **通过 SSH 复制（Replication via SSH）**  

使用 SSH 传输快照到另一台主机的 ZFS 池：  

```sh
zfs send poolA/ds@backup | ssh host zfs recv poolB/new
```

---

### **权限管理（Send and Receive Permissions）**  

#### **发送权限（Send Permissions）**  

允许 `sender` 用户发送数据集：  

```sh
zfs allow -u sender send,snapshot mypool/source
```

#### **接收权限（Receive Permissions）**  

接收方 `receiver` 需要额外的权限：  

```sh
zfs allow -u receiver compression,mountpoint,mount,create,receive mypool/destination
```
