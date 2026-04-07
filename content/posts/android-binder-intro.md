---
title: "Android Binder 机制入门"
date: 2026-04-07
tags: ["Android", "Binder", "IPC"]
description: "从零理解 Android 进程间通信的核心机制 Binder"
showToc: true
---

## 什么是 Binder？

Binder 是 Android 系统中最核心的进程间通信（IPC）机制，几乎所有系统服务调用都经过 Binder。

## 为什么不用传统 Linux IPC？

传统 Linux 提供了管道、Socket、共享内存等 IPC 方式，但 Binder 在 Android 场景下有明显优势：

- **安全性**：每次通信自动携带调用方 UID/PID，内核级别验证
- **性能**：只需一次数据拷贝（传统 Socket 需要两次）
- **面向对象**：支持远程对象引用，接口设计更自然

## 核心组件

| 组件 | 作用 |
|------|------|
| Binder 驱动 | 内核模块，负责进程间数据传递 |
| ServiceManager | 服务注册与查询中心 |
| IBinder | 远程调用接口 |
| AIDL | 自动生成 Stub/Proxy 代码的接口描述语言 |

## 后续

下一篇将深入 Binder 驱动的内存映射机制。
