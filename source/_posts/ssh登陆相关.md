---
title: ssh登陆相关
date: 2023-03-01 13:50:51
tags:
  - linux
catagories:
  - 计算机
---

# 密钥登陆

linux 下可以使用 ssh 远程登陆集群, 为了登陆方便, 我们可以使用密钥, 生成密钥的命令为:

```bash
ssh-keygen
```

其中需要输入文件名和 key 的类型。  
使用`ssh-copy-id`命令可以将`filename.pub`公钥分发给远程主机:

```bash
ssh-copy-id hapo@webserver1
```

以上命令会将默认公钥分发过去, 也可以使用

```bash
ssh-copy-id -i ~/.ssh/filename.pub hapo@webserver1
```

指定需要分发的公钥。  
此时就能不需要密码登陆远程服务器, 若密钥名字不是默认的名字(`~/.ssh/id_rsa`等), 则可以用以下命令指定:

```bash
ssh -i ~/.ssh/filename hapo@websever1
```

另外可以使用以下命令查看密钥和公钥对应的指纹:

```bash
ssh-keygen -l -f ~/.ssh/filename
ssh-keygen -l -f ~/.ssh/filename.pub
```

<!--more-->

# ssh 配置文件

可以在`~/.ssh/config`中写如下命令简化`ssh`登陆时所需参数:

```bash
Host hapo
    HostName ip
    Port 22
    User hapo
    IdentityFile ~/.ssh/id_rsa
    ServerAliveInterval 60 # 每60秒发送一次空请求
    ServerAliveCountMax 10 # 断开时重新连接的次数
```

# 使用`ssh-agent`和`ssh-add`

`ssh-agent`可以记录密钥的指纹, 并且自动查找和发送到服务器端, 因此不需要在指定所使用的密钥。
启动`ssh-agent`的命令为

```bash
eval `ssh-agent`
```

接下里就可以使用`ssh-add`添加密钥指纹

```bash
ssh-add #添加默认的密钥指纹
ssh-add ~/.ssh/id_rsa_1 #指定密钥的指纹
```

可以用以下命令查看添加过的密钥指纹

```bash
ssh-add -L
```

可以用以下命令修改添加过的密钥指纹

```bash
ssh-add -D # 删除ssh-agent中的所有密钥指纹
ssh-add -d key_file # 删除指定密钥指纹
```

另外, 杀掉现在正在运行的`ssh-agent`的命令为

```bash
ssh-agent -k
```

# `oathtool`

`oathtool`可以用于生成二次验证, 其使用命令行是

```bash
oathtool -b --totp <identity>
```

`<identity>`为用于生成二次验证的身份码。

# `sshpass`

`sshpass`可以用于在命令行输入密码, 命令行如下:

```bash
sshpass -p <password> <user>@<hostname>
```

通过结合`oathtool`, 可以实现免二次验证:

```bash
#！/bin/bash
totp=`oathtool -b --totp <identity>`
sshpass -p "<password> $totp" <user>@<hostname>
```

# `expect`

`expect`命令可以用于与终端进行交互, `expect`使用的是 tcl 语言, 这里不准备说明语法，只说明对应的一些用法
