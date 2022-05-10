---
title: ssh登陆信息配置
tags: ['ssh','linux']
date: 2022-05-10
categories: ['linux']
---
当你需要经常登陆到远程服务器上进行操作的时候，SSH链接是一个跳不过去的话题。

<!--more-->

当你需要链接的服务器相对较少的时候，你可以直接记忆下对应的ip和相关信息，但是随着你需要关注的服务器越来越多，你会发现ip、用户名、密码等信息的记忆对你来说越来越不可承受。
当然，OpenSSH早就注意到了这个问题，它允许用户设置配置文件，并通过配置文件里的相关信息进行登陆。

# 1. 文件说明
配置文件的路径通常位于`~/.ssh/config`。当然，默认情况下，`config`文件可能不存在，这个时候你可以手动创建对应的文件，并不会影响后续的使用。
```
touch ~/.ssh/config && chmod 700 ~/.ssh/config
```

# 2. 配置格式
SSH配置文件通常采用如下的配置格式:
```
Host name1
    Port 22
    User user-name
    HostName ip
    IdentityFile ~/.ssh/id_rsa
```
SSH配置文件中的每一个部分称做一个节。
每个节都是以`Host`开头，并且需要包含每个SSH链接所必需的相关选项。
如上面的配置信息，是一个常见的例子：
* Host name1: 当前Host配置的别名，需要的时候可以直接通过当前的别名进行ssh登陆。
* Port 22: ssh建立链接的时候使用的端口，默认是22
* User user-name: 建立ssh链接的时候需要用到的用户名
* HostName ip: ssh链接的目的服务器的ip
* IdentityFile ~/.ssh/id_rsa: 需要通过私钥登陆的时候的文件

当你需要使用`name1`的链接的时候，只需要在命令行中输入`ssh name1`即可以创建对应的ssh链接。

## 2.1 Host模式
Host指令可以包含一个模型或者是通过空格进行分隔的模式列表。每个模式可以包含零个活多个非空字符或者说明符之一：
1. `*` - 匹配任意（零个或多个）个字符
`Host *`可以匹配所有的Host
2. `?` - 匹配一个字符
`Host mu?`可以匹配以`mu`开头的任意3个字符的Host
3. `!` - 否定匹配
`Host * !mu`可以匹配除了`mu`以外的任意Host

SSH按节对配置文件进行读取，如果有多个host-name匹配，则优先使用第一个匹配到的节中的选项。

## 2.2 SSH配置共享
为了减少SSH配置文件中大量的冗余配置，可以对SSH配置中的内容进行共享，如下例子：
```
Host zermzhang
    User zermzhang

Host market
    User market

Host *zhang
    User zhang
    HostName 1.1.1.2
    IdentityFile ~/.ssh/id_rsa_2
    
Host *
    User root
    HostName 1.1.1.1
    Port 22
    IdentityFile ~/.ssh/id_rsa
```
如上，这是一个可以互相共享ssh配置信息的SSH配置文件的声明。

如果在命令行输入`ssh zermzhang`, 在逐节检测SSH配置文件的过程中，会按照顺序依次匹配到`Host zermzhang, Host *zhang, Host *`. 匹配到对应的Host之后，会按照匹配到的顺序依次获取对应的ssh配置。
1. 在`Host zermzhang`中获取到`User`
2. 在`Host *zhang`中获取到`User, HostName, IdentityFile`三个配置。但是因为已经在`Host zermzhang`中获取到了`User`，所以在该节中只会采用`HostName, IdentityFile`两个配置。
3. 同样，在`Host *`中只会采用`Port`这一个配置。

因此，输入`ssh zermzhang`之后可以获取到的完整配置是：
```
User zermzhang
HostName 1.1.1.2
IdentityFile ~/.ssh/id_rsa_2
Port 22
```
后续，OpenSSH客户端会根据上述信息创建对应的ssh链接