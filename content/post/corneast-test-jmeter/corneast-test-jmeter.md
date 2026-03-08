---
title: Performance Test Result of Corneast
description: 
date: 2026-03-08
slug: corneast-test-jmeter.md
image: 
categories:
    - Java
---

## Test Environment
OS: Fedora Linux 42 x86_64  
Kernel: Linux 6.18.12-100.fc42.x86_64  
CPU: 12th Gen Intel(R) Core(TM) i7-12700H (20) @ 4.70 GHz  
Memory: 15.30 GiB  

## Test Logic
See [the source of corneast-test-jmeter](https://github.com/Alioth4J/corneast/tree/main/corneast-test-jmeter).  

## Statistics
```json
{
  "Total" : {
    "transaction" : "Total",
    "sampleCount" : 1000000,
    "errorCount" : 0,
    "errorPct" : 0.0,
    "meanResTime" : 18.627447000000156,
    "medianResTime" : 5.0,
    "minResTime" : 0.0,
    "maxResTime" : 79.0,
    "pct1ResTime" : 7.0,
    "pct2ResTime" : 8.0,
    "pct3ResTime" : 23.0,
    "throughput" : 35789.69972441931,
    "receivedKBytesPerSec" : 141.90056726674064,
    "sentKBytesPerSec" : 0.0
  },
  "Java Request" : {
    "transaction" : "Java Request",
    "sampleCount" : 1000000,
    "errorCount" : 0,
    "errorPct" : 0.0,
    "meanResTime" : 18.627447000000156,
    "medianResTime" : 5.0,
    "minResTime" : 0.0,
    "maxResTime" : 79.0,
    "pct1ResTime" : 7.0,
    "pct2ResTime" : 8.0,
    "pct3ResTime" : 23.0,
    "throughput" : 35789.69972441931,
    "receivedKBytesPerSec" : 141.90056726674064,
    "sentKBytesPerSec" : 0.0
  }
}
```

