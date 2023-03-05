
# 通过源码在 Ubuntu20.04 上安装 python

## Step1: 配置构建环境

1. 更新 `apt`
```bash

sudo apt update

```

2. 安装所需依赖及一些工具:
```bash

sudo apt install build-essential zlib1g-dev libncurses5-dev libgdbm-dev libnss3-dev libssl-dev libreadline-dev libffi-dev libsqlite3-dev wget libbz2-dev pkg-config

```
## Step2: 下载和解压源文件

1. 复制需要安装的 python 发行包地址：
打开 *[Python Source Releases](https://www.python.org/downloads/source/)* 找到你需要使用的 Python 版本, 然后复制下载链接地址。
以 Python3.10.10 为例，你复制到的链接地址应该是：https://www.python.org/ftp/python/3.10.10/Python-3.10.10.tgz

2. 使用`wget`下载发行包：
> 注意：在执行下面的命令之前你可以先切换工作目录到你希望保存发行包的地方（工作目录可以使用`pwd`命令查看）

```bash

wget https://www.python.org/ftp/python/3.10.10/Python-3.10.10.tgz

```
正常执行完上述指令后你应该可以在当前工作目录找到一个名为`Python-3.10.10.tgz`的文件。

3. 解压发行包：
在存有`Python3.10.10.tgz`的目录执行如下命令解压发行包：
```bash

tar xzf Python3.10.10.tgz

```
成功执行指令后当前工作目录会出现一个与发行包同名的文件夹（Python3.10.10），进入该文件夹：
```bash

cd Python3.10.10

```

## Step3: 配置和构建 Python
1. 检查依赖：
进入解压后的文件夹后你应该会看到一个名为`configure`的可执行文件，执行如下指令：
```bash

./configure --enable-optimizations

```

2. 开始构建 Python
执行如下指令开始构建 Python
```bash

make -j $(nproc)

```
如果你的机器是多核的话`-j`参数可以利用多个核心完成构建，为你节省大量时间。`nproc`可以获取你当前运行机器的核心数。

3. 安装 Python
在成功完成上述操作后就可以安装 Python 了：
```bash

sudo make altinstall

```
使用`altinstall`而不是使用`install`的目的是为了保留你位于`/usr/bin/python`的 Python
如果一切顺利的话你将在`/usr/local/bin`目录下看到安装好的 Python。
你试着运行一下它，如果能正常运行，那就恭喜你！
