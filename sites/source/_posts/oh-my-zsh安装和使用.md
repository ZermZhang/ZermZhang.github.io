---
title: oh-my-zsh安装和使用
tags: ['zsh','linux']
date: 2022-04-21
categories: ['linux']
---

## oh-my-zsh安装
oh-my-zsh是是针对zsh命令行环境的配置包装框架，通过它，我们可以省去针对zsh繁琐的配置过程，几乎达到开箱即用的地步。
<!--more-->
oh-my-zsh可以通过脚本进行安装：
```shell
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```
安装之后，是zsh的默认界面，相对比较简陋，看起来和bash并没有什么区别，但是我们可以通过丰富的插件配置完成zsh的一个蜕变。下面介绍两款常用的oh-my-zsh配置。

## 插件、主题选择和配置
oh-my-zsh的插件安装方式非常简单，只需要将对应插件的工程clone到oh-my-zsh的plugins文件夹下，并通过配置文件进行简单配置即可使用。
#### zsh-syntax-highlighting
```shell
git clone \
    https://github.com/zsh-users/zsh-syntax-highlighting.git \
    ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```
#### zsh-autosuggestions
```shell
git clone \
    git://github.com/zsh-users/zsh-autosuggestions \
    ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```

插件clone完成后，重启zsh或者`source ~/.zshrc`即可。

## oh-my-zsh的更新
oh-my-zsh更新检测命令`omz updateß`