# 如何将基于 Electron 的应用程序移植到 FreeBSD

- 原文链接：[How to port an Electron-based application to FreeBSD](https://blog.c6h12o6.org/post/freebsd-electron-app/)
- 作者：Hiroki Tagato
- 2020 年 3 月 3 日
- 作者复刻的 GitHub 地址：<https://github.com/tagattie/FreeBSD-Electron>

[Electron](https://www.electronjs.org/) 是一款流行的框架，可用于使用 HTML、CSS 和 JavaScript 等 Web 技术开发桌面应用程序。基于 Electron 构建的应用程序有很多（无论开源还是闭源），可以在以下网站找到一些例子：  

- [Electron Apps](https://www.electronjs.org/apps)  
- [Awesome Electron](https://github.com/sindresorhus/awesome-electron)  

在 [Wanted Ports](https://wiki.freebsd.org/WantedPorts) 列表中待了一段时间后，Electron 于 2019 年 5 月正式进入 FreeBSD 的 Ports，目前支持两个主要版本（4 和 6）。自那以后，两款基于 Electron 的应用程序——[Visual Studio Code](https://www.freshports.org/editors/vscode/) 和 [Atom](https://www.freshports.org/editors/atom/)（恰好都是代码编辑器）——已进入 Ports。  

如果你查看这些 ports，会发现它们包含冗长的 Makefile 和大量的补丁文件。因此，你可能会认为移植基于 Electron 的应用程序是件繁琐的工作。然而，事实并非如此。这两个编辑器属于特例，移植基于 Electron 的应用程序通常比你想象的要简单。  

借助一些[辅助 Makefile](https://github.com/tagattie/FreeBSD-Electron/tree/master/Mk/Uses)（仍在开发中），你可以通过编写不到 100 行的 Makefile 和一些配套文件来创建一个 port。  

在这篇文章中，我将介绍如何使用这些 `.mk` 文件来移植基于 Electron 的应用程序。我选择了一个简单但仍然实用的应用程序 [YouTube Music Desktop](https://github.com/ytmdesktop/ytmdesktop) 作为示例。

## 准备工作

在开始移植之前，我们需要进行一些准备工作。首先，我们需要获取关于该应用程序的信息，例如：  

- 该应用程序所依赖的 Electron 版本，  
- 构建该应用程序所需的 Node 版本，  
- 构建该应用程序所需的 Node 包管理器。  

应用程序的源代码归档文件中的 `README` 文档/ `package.json` 文件通常会提供这些信息。让我们下载该归档文件并查看内容。  

```sh
fetch https://github.com/ytmdesktop/ytmdesktop/archive/v1.8.2.tar.gz
tar -xzf v1.8.2.tar.gz
less ytmdesktop-1.8.2/package.json
```  

- **Electron 版本** —— Electron 通常被指定为应用程序项目的开发依赖项。在 `package.json` 文件的 `devDependencies` 部分，我们找到如下行：`"electron": "^7.1.11",`，因此所需的 Electron 版本为 **`7`**。  
- **Node 版本** —— 该归档文件中没有找到关于 Node 版本的具体描述。因此，我们假设可以使用所有受支持的 Node 版本。在本例中，我们选择 **`12`** 版本。（**注意**：这只是因为我个人喜欢使用最新的 LTS（长期支持）版本，其他受支持的版本应该也可以正常运行。）  
- **包管理器** —— 应用程序项目通常使用一个文件来锁定所有依赖项（Node 模块）的版本，以确保项目的可复现性。如果存在 `package-lock.json` 文件，则表示使用的是“NPM”包管理器；而 `yarn.lock` 文件表示使用的是“Yarn”。在本例中，该归档文件包含了这两个文件，这可能意味着可以使用任意一款包管理器。这里我们选择 NPM 作为包管理器（纯粹是个人决定 :-）。  

### 总结  

我们将使用以下版本：  

- Electron: **7**  
- Node: **12**  
- 包管理器：**NPM**

还需要进行一步准备工作。如上所述，port 将使用那些仍在开发中的 `.mk` 文件提供的功能。因此，我们需要获取这些文件并将它们复制到 ports 目录中。  

```sh
git clone https://github.com/tagattie/FreeBSD-Electron.git
cp FreeBSD-Electron/Mk/Uses/*.mk ${PORTSDIR}/Mk/Uses
```  

现在，我们已准备好开始创建 port。  

## 移植  

>**注意**
>
>可在[我复刻版 Ports 仓库](https://github.com/tagattie/freebsd-ports/tree/master/multimedia/ytmdesktop)中找到完整的 port。如果你想快速查看它的样子，可以前往该链接。  

### 初始化 port  

首先，初始化 port，例如执行以下命令。（**注意**：这里使用了 `ports-mgmt/porttools`。）  

```sh
cd ${PORTSDIR}
port create multimedia/ytmdesktop
```  

### 放置 `package.json` 和锁定文件  

将应用程序源代码归档文件中的 `package.json` 和 `package-lock.json` 复制到 port 的 `files/packagejsons` 目录中。  

```sh
cd multimedia/ytmdesktop
mkdir -p files/packagejsons
cp /path/to/archive/ytmdesktop-1.8.2/package*.json files/packagejsons
```  

### 为什么需要这样做？  

我认为移植基于 Electron 的应用程序最麻烦的部分是准备分发文件。官方 port 需要能够使用 Poudriere 进行构建，这意味着必须预先下载所有分发文件（包括应用程序依赖的所有 Node 模块），并使用校验和文件 (`distinfo`) 验证其完整性。  

使用 NPM 的项目会在 `package.json` 和 `package-lock.json` 中指定所有模块依赖项。辅助 `.mk` 文件利用这些文件来“预获取”应用程序所需的所有 Node 模块，并将它们打包成一个归档文件，以便进行校验和验证。  

因此，你无需手动处理 Node 模块的分发文件。Ports 基础设施会在执行 `make makesum` 时自动处理这些 Node 模块的命名、获取和校验。

### 编写 Makefile  

现在，让我们来编写 Makefile。我只会介绍与 Electron 相关的部分。完整的 Makefile 请参考[我的复刻版仓库](https://github.com/tagattie/freebsd-ports/blob/master/multimedia/ytmdesktop/Makefile)。  

我们需要指定三个重要的变量：`USES`、`USE_NODE` 和 `USE_ELECTRON`。首先来看 `USES` 和 `USE_NODE`。通过定义这两个变量，指定的 Electron 版本、Node 版本以及包管理器会被自动添加到必要的依赖项中。  

```makefile
USES=           electron:7 node:12,build
USE_NODE=       npm
```  

接下来的 `USE_ELECTRON` 变量涉及提供的核心功能，因此我会详细解释这些特性。  

```makefile
USE_ELECTRON=   prefetch extract prebuild build:builder
PREFETCH_TIMESTAMP=     1582793516
```  

- `prefetch` —— 如果在 `${DISTDIR}` 目录中找不到分发文件，则在 fetch 阶段会使用预存的 `package.json` 和 `package-lock.json` 下载应用程序所依赖的所有 Node 模块。下载的 Node 模块会被打包成一个自动命名的 tar 文件，作为 `DISTFILES` 之一。  
  - `PREFETCH_TIMESTAMP` —— 如果使用 `prefetch` 功能，则必须定义此变量。该变量是时间戳，会赋值给 tar 归档中的所有目录、文件和链接，以确保归档文件的可复现性。你可以使用命令 `date '+%s'` 获取一个合适的值。  
- `extract` —— 在 extract 阶段，将预获取的 Node 模块安装到 port 的工作源码目录中。  
- `prebuild` —— 在 build 阶段，会针对指定版本的 Node 重新编译本机 Node 模块，使其能够被 Node 执行，以便构建应用程序。此外，该功能还会在应用程序打包之前，针对指定版本的 Electron 重新编译本机 Node 模块。  

简单来说，这三个特性将 `npm install` 过程拆分为三个阶段，使其适用于 port 构建流程。  

最后一个特性与应用程序的打包有关，它会生成可分发的应用程序形式。根据 [官方文档](https://www.electronjs.org/docs/tutorial/application-distribution)，常见的打包工具包括：  

- [electron-forge](https://www.electronforge.io/)  
- [electron-builder](https://www.electron.build/)  
- [electron-packager](https://npm.im/electron-packager)  

当前，该特性支持 `electron-builder` 和 `electron-packager`（分别对应 `build:builder` 和 `build:packager`）。  

打包工具通常被指定为开发依赖项。查看 `package.json` 文件的 `devDependencies` 部分，我们可以找到如下行：  

```json
"electron-builder": "^21.2.0"
```  

这表明该应用程序依赖于 `electron-builder`。

我们已经快要完成了，但还有最后一件事要做。安装阶段的准备工作是最繁琐的部分。你需要从零开始编写安装目标，主要涉及以下几个步骤：  

- 为应用程序创建一个包装脚本；  
- 为应用程序创建 `.desktop` 入口文件；  
- 生成应用程序图标（除非源代码归档中已经包含了图标）；  
- 在 Makefile 中编写 `do-install` 目标。  

包装脚本的模板可能如下所示：  

```sh
#! /bin/sh

export NODE_ENV=production
export ELECTRON_IS_DEV=0

electron%%ELECTRON_VER_MAJOR%% %%DATADIR%%/resources/app.asar $@
```  

在 `do-install` 目标中，手动安装创建的包装脚本、`.desktop` 文件和图标。此外，不要忘记将打包工具生成的应用程序资源目录复制到 `${DATADIR}`。  

```makefile
do-install:
        # 将包装脚本、.desktop 入口文件和图标安装到适当位置
        # （此处省略部分代码）
        # 安装应用程序数据目录到 ${DATADIR}
        ${MKDIR} ${STAGEDIR}${DATADIR}
        cd ${WRKSRC}/dist/linux-unpacked && \
                ${COPYTREE_SHARE} resources ${STAGEDIR}${DATADIR}
```  

## 构建  

最后，我们已经准备好构建这个 port 了。  

```sh
make makesum # 生成 distinfo
make build
```  

此外，我们仍然需要 `pkg-descr` 和 `pkg-plist` 文件来打包该应用程序。不过，由于这些工作与 Electron 无关，这部分就留给你自行完成了。
