---
title: "在centos中安装chrome"
date: 2023-07-27T20:20:51+08:00
draft: false
tags: ['chrome', 'centos']
---


## 安装 chrome

1, 使用 `wget` 命令下载最新的 Chrome 64 位 rpm 软件包

``` bash
wget https://dl.google.com/linux/direct/google-chrome-stable_current_x86_64.rpm
```

2, 安装 rpm 包
``` bash
sudo dnf localinstall google-chrome-stable_current_x86_64.rpm
```

## 安装 microsoft edge

1, 下载并导入 microsoft 的 GPG 公钥

``` bash
sudo rpm --import https://packages.microsoft.com/keys/microsoft.asc
```

2, 添加程序安装源
``` bash
sudo dnf config-manager --add-repo https://packages.microsoft.com/yumrepos/edge
```

3, 修正安装源的文件名称
``` bash
sudo mv /etc/yum.repos.d/packages.microsoft.com_yumrepos_edge.repo /etc/yum.repos.d/microsoft-edge-stable.repo
```

4, 安装
``` bash
sudo dnf install microsoft-edge-stable
```

## 更新 chrome

``` bash
sudo dnf update google-chrome-stable
```

## 卸载 chrome

``` bash
sudo dnf remove google-chrome-stable
```

## 可能遇到的问题

1. 打开浏览器失败

由于我是在没有图形界面的系统上运行 chrome，系统缺少 X 服务器
需要在启动 chrome 时，添加参数 `--no-sandbox`

sandbox 是 chrome 的一种安全机制，会隔离不安全的代码。日常使用不建议关闭

2. 缺少中文字体，网页内中文显示为空格

安装语言包
``` bash
sudo yum -y groupinstall Fonts
```

## 参考链接

[running-headless-chrome-puppeteer-with-no-sandbox](https://stackoverflow.com/questions/50662388/running-headless-chrome-puppeteer-with-no-sandbox)
