---
layout: post
title: unikernel
tags: paper research unikernel
---

# Unikernel

## 概述

Unikernel类似外核，都是不区分内核态和用户态，通过library system来与硬件交互。而Unikernel与Library不同的是，它主要跑在云平台上，借助Hypervisor的硬件虚拟化能力来完成底层操作。

## Unikernel 特点

- Unikernel抹去了系统的复杂性，使得系统为应用定制，去除了许多因为通用化而加入的不必要的代码。
- Unikernel去除了内核态与用户态切换的过程。由于其内存是但地址空间，不区分内核和应用，所以其启动速度极快。
- Unikernel由于其OS是定制化的，所以不存在通用的服务，所以同样的攻击手段不一定可以适用于其他的服务。
- Unikernel也是一种不可变的基础设施，编译之后不可修改，安装一次，不做修改，用过即扔。

## 现有的Unikernel项目

- ClickOS
- LightVM
- MirageOS
- HaLVM
- rumprun
- IncludeOS
- OSv
- gVisor