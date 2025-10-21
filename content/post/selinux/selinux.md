---
title: Failures Caused by SELinux
description: What Went Wrong? 
date: 2025-10-21
slug: selinux
image: 
categories:
    - Linux
---

## Introduction
Sometimes you met a problem, you tried to debug, but found nothing was wrong. Then what went wrong? The failures may be caused by SELinux.  

## Failure Types
### Permission Denied
Service fails to start, the log only says "Permission denied".  

One common situation is a container startup failure.  

### Network Connection Denied
Unable to establish a connection.  

## Solution
### Check SELinux Status
```bash
getenforce
```

`Enforcing` means SELinux is working.  
`Permissive` means it only records.  

### Disable SELinux Temporarily
```bash
sudo setenforce 0
```

### Disable SELinux Permenantly
OS: Silverblue  

```bash
sudo rpm-ostree kargs --append enforcing=0
```
or  
```bash
sudo rpm-ostree kargs --editor
```
then add `enforcing=0`.  

