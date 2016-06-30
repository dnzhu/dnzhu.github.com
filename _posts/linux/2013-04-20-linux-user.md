---
layout: post
title: "linux 账户管理"
description: ""
category: linux
tags: []
---

> 在linux系统中，对用户和组的管理是通过id来实现的。用户的ID为UID,组ID为GID。root账户的UID=0,1-499之间的id被系统所保留，我们创建的普通账户，ID从500开始。

> linux的组有基本组和附加组之分，一个账户只能加入一个基本组，但可以同时加入多个附加组，创建账户时，系统默认会创建 同名的基本组，并设置账户加入这个基本基本组。


{% include JB/setup %}