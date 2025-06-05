---
title: Leveraging V2Ray on the Outer Layer and VPN on the Inner Layer
description: Follow the white rabbit
date: 2025-05-18
slug: v2ray&vpn
image: 
categories:
    - Network
---

In this blog post, we’ll explore a dual-layer network connection method: using V2Ray on the outside for traffic obfuscation, while employing a VPN on the inside to secure the data.  

## Why Not Just Use One of Them?
### V2Ray Only
It’s important to note that V2Ray functions as a proxy, meaning that by default it doesn’t modify certain HTTP headers, such as `X-Forwarded-For` and `Cf-Connecting-Ip`. This can result in transparent proxy, where the underlying IP addresses are exposed.  

### VPN Only
Sometimes it's OK. However, it may be detected and blocked. For instance, many network filters and deep packet inspection techniques are capable of identifying OpenVPN’s distinctive handshake and traffic patterns.  

## Understanding the Dual-Layer Architecture
### Outer Layer: V2Ray
It supports multiple protocols (such as VMess or VLESS) and advanced obfuscation techniques. When used as the outer layer, V2Ray serves to scramble and mask your network traffic, making it harder for someone to detect your actual data or even the fact that you’re using a proxy.  

### Inner Layer: VPN
Running a VPN within the V2Ray tunnel adds an extra level of encryption and secure access, particularly for those who want an additional layer of privacy.  

## Operating Guide
### Prerequisites
Setup V2Ray and VPN.  

### Order of Bootstrap
You are supposed to launch the V2Ray client first, and only then initiate the VPN connection. This is because V2Ray, being used as the outer layer, needs to establish a secure channel before the VPN attempts to create its tunnel within it.  

### About firewalld
Only need to open v2ray's port, preventing others to detect actively.  
