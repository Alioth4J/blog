---
author:
  name: "Alioth4J"
date: 2024-04-20
linktitle: 3步删除旧的 Linux 内核版本
title: 3步删除旧的 Linux 内核版本
type:
- post
- posts
weight: 10
series:
- Linux
aliases:
- /blog/delete-old-linux-kernel/
---

## 背景
经过更新，旧的 Linux 内核版本仍然存在，在 UEFI 启动时仍然有选项。现在需要将其删除。
## 步骤
### 1. 打开终端(这居然也算一步~)
### 2. 运行 `rpm -qa | grep kernel` 命令查看当前安装的内核版本
在笔者的电脑上，发现有5个旧的内核版本，5个新的内核版本
### 3. 运行 `sudo dnf remove kernel-[old_kernel_version]` 命令删除旧的内核版本
将上述 `[old_kernel_version]` 替换为具体的旧内核版本号，只需要选择5个中的1个即可，会自动关联其它的  
接着根据提示输入 `y`  ，等待删除成功
## 成功了
UEFI 启动时会发现旧的内核版本已经移除
