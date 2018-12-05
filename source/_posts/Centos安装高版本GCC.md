---
title: Centos安装高版本GCC
date: 2018-12-05 15:06:36
tags:
    - Linux环境
---

# 安装版本GCC

```
yum install centos-release-scl -y
yum install devtoolset-6-gcc devtoolset-6-gcc-c++
```

# 设置环境
```
vi ~/.bashrc 

在最后面加入
source /opt/rh/devtoolset-6/enable
```