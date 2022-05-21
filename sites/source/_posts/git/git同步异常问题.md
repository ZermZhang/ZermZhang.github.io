---
title: git同步异常问题
tags: ['github','同步']
date: 2022-05-21 15:31
categories: ['git']
---

> 常见的git同步问题的收集

<!-- more -->

# 1. LibreSSL SSL_read
在向github推送commit的时候，会包如下错误：
![报错信息](https://s2.loli.net/2022/05/21/nESf8NwvXkDbeGt.png)
上面的错误，可以通过如下的命令解决：
```shell
git config --global --unset http.proxy
git config --global --unset https.proxy
```
经过处理后，即可以进行再次同步：
![同步成功](https://s2.loli.net/2022/05/21/kUyrnTzZLvpBb6g.png)