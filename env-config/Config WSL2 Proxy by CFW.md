
# 为 WSL2 配置 CFW 代理

## 前言：

更新 WSL 到 WSL2 之后发现 WSL2 和 Windows 宿主机不再是同一个本地网络了，这就直接导致直接设置`export http_proxy=127.0.0.1:7890`后无法通过代理访问外部链接的问题。在查询官方文档和阅读各方大佬的解决方案后总结出该方案。


## 环境说明：

- 宿主机系统：Windows 10 22H2
- WSL2系统：Ubuntu 20.04
- 代理软件：Clash For Windows v0.20.17
- 代理端口：56118（CFW随机生成的，大多数应该是：7890）

## 配置 CFW：

1. 首先，你需要在 Windows 系统上正常配置 CFW 并且验证其是否配置正确。

2. 然后在 `CFW > General` 中开启：`Allow LAN`、`TUN Mode`（会提示你安装 Service Mode）、`System Proxy` 这三个选项。

## 配置 Windows 防火墙：

1. 打开你的 Windows Defender 防火墙配置，找到一个名为：`clash-win64.exe`的规则，把专有和公共网络同时允许。如果找不到这个规则，那就需要手动添加，看下一步。

2. 手动添加 `clash-win64.exe` 规则：如果在第 1 步中你成功找到并设置了 `clash-win64.exe` 的规则，那就跳过这一步。
    1. 在你的资源管理器中找到一个叫 `clash-win64.exe` 的进程，打开文件所在位置，然后想办法复制这个文件所在的位置。
    2. 然后在 Windows Defender 防火墙中添加这个应用的规则。

## 配置 WSL2：

接下来的事情非常简单，如果你有 Linux shell脚本使用经验的话，我觉得你可以直接复制下面的代码去操作了。如果没有相关经验的话，那就执行下面的步骤吧。

1. 复制下面的脚本到一个你熟悉的文本编辑器，然后将第四行的 `port` 后面的数字换成你的 CFW 上显示的端口，大多数是 `7890` ，我的是 `56118`。接下来复制你改好的代码。

```bash
#!/bin/sh
hostip=$(cat /etc/resolv.conf | grep nameserver | awk '{ print $2 }')
wslip=$(hostname -I | awk '{print $1}')
port=56118

PROXY_HTTP="http://${hostip}:${port}"

set_proxy(){
  export http_proxy="${PROXY_HTTP}"
  export HTTP_PROXY="${PROXY_HTTP}"

  export https_proxy="${PROXY_HTTP}"
  export HTTPS_proxy="${PROXY_HTTP}"

  export ALL_PROXY="${PROXY_SOCKS5}"
  export all_proxy="${PROXY_SOCKS5}"

  echo "Proxy has been opened."
}

unset_proxy(){
  unset http_proxy
  unset HTTP_PROXY
  unset https_proxy
  unset HTTPS_PROXY
  unset ALL_PROXY
  unset all_proxy

  echo "Proxy has been closed."
}

test_setting(){
  echo "Host IP:" ${hostip}
  echo "WSL IP:" ${wslip}
  echo "Try to connect to Google..."
  resp=$(curl -I -s --connect-timeout 5 -m 5 -w "%{http_code}" -o /dev/null www.google.com)
  if [ ${resp} = 200 ]; then
    echo "Proxy setup succeeded!"
  else
    echo "Proxy setup failed!"
  fi
}

if [ "$1" = "set" ]
then
  set_proxy

elif [ "$1" = "unset" ]
then
  unset_proxy

elif [ "$1" = "test" ]
then
  test_setting
else
  echo "Unsupported arguments."
fi

```

2. 在你的用户目录下创建一个名为 `.proxy_config` 的目录

```bash

cd & mkdir .proxy_config

```

3. 进入刚才创建的 `.proxy_config` 目录，接下来创建一个名为 `proxy.sh` 的文件，并将你刚才复制好的代码写入文件。

4. 现在回到你的用户目录，开始编辑 `.bashrc` 文件，在该文件的最后两行添加以下代码，记得将 `fanburger` 这个用户名换成你自己的：

```bash
. /home/fanburger/.proxy_config/proxy.sh set
alias proxy="source /home/fanburger/.proxy_config/proxy.sh"

```
添加完成后执行:
```bash

source ./.bashrc

```

第一行代码用于当你登录 WSL 的时候自动开启代理。
第二行代码的用法如下，在终端执行以下命令即可进行相关操作：
- `proxy set` ：开启代理
- `proxy unset` ：关闭代理
- `proxy test` ：查看代理状态

## 参考文档：
1. [WSL配置Proxy代理引导](https://halc.top/p/6088c65c)
2. [WSL2配置代理](https://www.cnblogs.com/tuilk/p/16287472.html)
3. [Accessing network applications with WSL](https://learn.microsoft.com/en-us/windows/wsl/networking)