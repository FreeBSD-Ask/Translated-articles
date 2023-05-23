# 这就是我如何在 FreeBSD 上安装 Python，并准备好与在线教程一起使用。

## 利用 Python 3.8 和 Pipenv，用于管理虚拟环境和通过 pip安装软件包

```sudo pkg install -y python38```

```sudo -H python3.8 -m ensurepip```

```sudo -H pip3.8 install --upgrade pip setuptools```

```sudo -H pip3.8 install pipenv```

## 最后一个配置步骤是在 FreeBSD 上设置 UTF-8 。如果您还没有这样做，请查看下面链接的 FreeBSD Workshop 简介中的说明

[Intro to FreeBSD](https://github.com/possnfiffer/bsd-pw/blob/gh-pages/docs/Intro_to_FreeBSD_Workshop.md#iocage)

## 带有 pipenv 使用命令的 Python

```pipenv install b2```

（或者任何包名称在此处都可以代替b2）

pipenv 将为您创建虚拟环境 ，并在其中安装任何软件包。

或者，您可以使用

```pipenv shell```

创建一个虚拟环境并打开其中的 shell ，这样您就可以键入一些正常的内容，如

```pip install b2```

这将安装 BackBlaze B2 的 b2 客户端，我用它来做异地备份，但它不会把这些保存到 Pipfile中，所以我推荐使用 pipenv 安装的方式，以方便使用，而常规的 venv 模块则用于自动化部署的情况。

另外，pipenv 使用 Pipfile 而不是 requirements.tx t，所以你可以导入一个requirements.txt 文件，例如

```pipenv install -r requirements.txt```

## 常规的 Python venv 模块使用命令

```python3.8 -m venv venv```

```source venv/bin/activate.csh```

```pip install --upgrade pip setuptools```

```pip install b2```

```pip install -r requirements.txt```

```deactivate to exit virtualenv```

### 一旦您有安装的需要。。。

日常使用情况是这样的

```pipenv shell```

从包含 Pipfile 的项目目录中

```python```

您将看到您的>>Python repl，尽情享受吧！

### 这对一些有 setup.py 的项目也有效。

```
pipenv install -e project_dir/
pipenv shell
cd project_dir/
python setup.py develop
```
