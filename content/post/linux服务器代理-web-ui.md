+++
title = "Linux服务器代理 (Web UI)"
date = 2025-05-30
tags = ["Linux", "Clash"]
categories = ["Linux"]
draft = false
author = "Chris Ng"
+++

在服务器中，难免会遇到一些访问外网的需求。如果轻量使用的话，可以使用端口转发的方式，使用本地的代理，但是肯定会有些麻烦。因此，我经过一番搜索，找到了一个在服务器中能够轻松使用的代理，并且能够使用Web来切换节点，避免了远程连接没有ui的尴尬。


## 安装 Crash {#安装-crash}

网上已经有大牛写好了一个脚本，我们直接下载就行了，具体请参考 <https://github.com/juewuy/ShellCrash/tree/dev>

运行中根据指引进行安装即可，也可以自定义安装路径，本教程统一将clash安装在家目录中

```shell
cd ~
mkdir clash
```

接下来，在终端中

```shell
sudo -i #切换到root用户，如果需要密码，请输入密码
bash #如已处于bash环境可跳过
export url='https://fastly.jsdelivr.net/gh/juewuy/ShellCrash@master' && wget -q --no-check-certificate -O /tmp/install.sh $url/install.sh  && bash /tmp/install.sh && source /etc/profile &> /dev/null
```

如果失败，请更换别的源进行下载，具体参考上面给出的链接，写的非常详细！
接下来，按照指令进行安装：

{{< figure src="/ox-hugo/install_1.png" >}}

{{< figure src="/ox-hugo/install_2.png" >}}


## 配置 crash {#配置-crash}

目前应该为 `root` 用户，我们可以直接在当前用户下输入 `crash` 直接进行进入配置，或者是退出当前账户切换到不同用户后执行，但是注意需要使用 `sudo` ，否则会报错

```shell
# 管理员用户下
crash

# 普通用户下
# 管理员用户下，输入 exit 可切换为普通用户
sudo crash
```

进入脚本后，安装指令选择就好了，最后我们选择 **导入本地配置文件** ，需要记住存放本地配置文件的目录 `/tmp`

{{< figure src="/ox-hugo/1.png" >}}

{{< figure src="/ox-hugo/2.png" >}}

{{< figure src="/ox-hugo/3.png" >}}


## 订阅链接获取 {#订阅链接获取}


### 生成 `config.yaml` 文件 {#生成-config-dot-yaml-文件}

我们使用的订阅链接通常为机场的订阅链接，因此，我们需要借助机场的订阅链接来获取配置文件，获取订阅链接有三种方法

```shell
# 方法 1
sudo wget -O ./config.yaml [机场订阅链接]

# 方法 2
sudo curl -o ./config.yaml [机场订阅链接]

# 方法 3
sudo curl [机场订阅链接] >./config.yaml
```

理论上来说，输入以上其中一条路径后，会在当前路径生成一个 `config.yaml` 配置文件，我们需要将其拷贝到刚才我们记录的路径

```shell
sudo cp ./config.yaml /tmp
```

然后在命令行中重新启动 `crash` 即可

```shell
sudo crash
```

{{< figure src="/ox-hugo/4.png" >}}

加载配置后，按照指令输入 `1` 即可启动，然后按 `ctrl` 或者 `command` 然后鼠标点击链接即可查看节点的信息，通常来说，选择第三个即可。

{{< figure src="/ox-hugo/5.png" >}}

进入页面后，和Windows上clash的界面基本差不多。

{{< figure src="/ox-hugo/6.png" >}}
