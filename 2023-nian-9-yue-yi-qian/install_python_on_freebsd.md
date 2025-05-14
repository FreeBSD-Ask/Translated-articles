# 如何在 FreeBSD 上安装 Python

 - 原文链接：<https://github.com/possnfiffer/bsd-pw/blob/gh-pages/docs/Install_Python_on_Freebsd.md>
 - 作者：Roller Angel
 - 译者：飞鱼
 - 校对整理：ykla【】部分为 ykla 注解。

## 使用 Python 3.8 和 Pipenv 管理虚拟环境并通过 pip 安装软件包

```
sudo pkg install -y python38
sudo -H python3.8 -m ensurepip
sudo -H pip3.8 install --upgrade pip setuptools
sudo -H pip3.8 install pipenv
```
【`-H` 参数将环境变数中的 HOME（家目录）指定为要变更身份的使用者家目录，在上述中即 `/root`。】

## 最后一个配置步骤是在 FreeBSD 上设置 UTF-8。如果你还没有这样做，请查看下面链接的 FreeBSD Workshop 简介中的说明

[FreeBSD 简介](https://github.com/possnfiffer/bsd-pw/blob/gh-pages/docs/Intro_to_FreeBSD_Workshop.md#iocage)

## 使用带有 pipenv 命令的 Python

```
pipenv install b2
```

（或者任何包名称在此处都可以代替 b2）

pipenv 将为你创建虚拟环境，并可在其中安装软件包。

或者，你可以使用

```
pipenv shell
```

创建一个虚拟环境并打开其中的 shell，这样你就可以键入一些正常的内容，如：

```
pip install b2
```

这将安装 BackBlaze B2 的 b2 客户端，我用它来做异地备份，但它不会把这些保存到 Pipfile 中，所以我推荐使用 pipenv 安装的方式，以方便使用，而常规的 venv 模块则用于自动化部署的情况。

另外，pipenv 使用 Pipfile 而非 `requirements.txt`，所以你可以导入 `requirements.txt` 文件，例如

```
pipenv install -r requirements.txt
```

## 常规的 Python venv 模块使用命令

```
python3.8 -m venv venv
source venv/bin/activate.csh
pip install --upgrade pip setuptools
pip install b2
pip install -r requirements.txt
deactivate to exit virtualenv
```

### 如果你有安装的需求

日常使用情况是这样的

```
pipenv shell
```

从包含 Pipfile 的项目目录中运行：

```
python
```

你将看到你的 >>Python，尽情享受吧！

### 这对一些有 setup.py 的项目也有效

```
pipenv install -e project_dir/
pipenv shell
cd project_dir/
python setup.py develop
```
