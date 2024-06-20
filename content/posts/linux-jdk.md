---
author:
  name: "Alioth4J"
date: 2024-04-13
linktitle: 在 Linux 上安装 JDK（以 JDK17 为例）
title: 在 Linux 上安装 JDK（以 JDK17 为例）
type:
- post
- posts
weight: 10
series:
- Linux
aliases:
- /blog/linux-jdk/
---

## 下载 JDK
- 访问 Oracle 官网：
https://www.oracle.com/java/technologies/downloads/#java17 链接: [link](https://www.oracle.com/java/technologies/downloads/#java17)
## 解压下载的 .tar.gz 文件
- 打开终端，
- 进入所在文件夹: `cd ~/Downloads`
- 解压文件: `tar -zxvf jdk-17_linux-x64_bin.tar.gz`
## 移动 JDK 目录
- 例如将解压后的 JDK 目录移动到`/usr/lib/jvm`: 
`sudo mv jdk-17.0.10 /usr/bin/jvm`
## 设置环境变量
- 使用 vi 编辑器进入 ~/.bashrc 文件（这是一个隐藏文件）:
`vi ~/.bashrc`
- 按`i`键进入编辑模式
- 在最后输入
```
export JAVA_HOME=/usr/lib/jvm/jdk-17.0.10
export PATH=$PATH:$JAVA_HOME/bin
```
- 按`esc`键退出编辑模式
- 输入`:wq`保存并退出
## 最后，验证安装
`java --version`
