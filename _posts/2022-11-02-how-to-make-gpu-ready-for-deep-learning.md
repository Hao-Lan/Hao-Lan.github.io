---
layout: post
title:  "如何为深度学习准备GPU 环境?"
date:   2022-11-02 20:32:00 +0800
---

理解,在有N卡的前提下,如何使`tensorflow` 或者 `pytorch` 利用GPU 做加速.

## 相关概念

- N卡

N 卡特指英伟达厂家的显卡.

- 显卡驱动

操作系统使用显卡（硬件）的底层程序（驱动）.

- CUDA

英伟达推出的、基于英伟达GPU 的、通用并行计算平台和编程模型.

更偏架构、概念,位于显卡驱动上层.

- CUDA Toolkit

CUDA 的具体呈现方式.

可以涵括编译CUDA 程序、动态库调用CUDA接口等功能的工具.

多数场合所提到的,CUDA 的安装,CUDA 特指 CUDA toolkit.

并且安装时受显卡驱动的版本约束.

- cudnn

cuDNN是CUDA 内,用于深度神经网络的GPU加速库.

## 操作

#### tensorflow

较新版本的tensorflow,安装时不用区分cpu 和 gpu 版本.

只是在使用时,如果配置好了GPU环境,会默认选择GPU 做并行.

所以只需要在电脑上,按照如下流程配置好CUDA环境,tensorflow 可自动进入GPU 状态:

- 安装CUDA
- 安装cudnn
- 配置系统环境变量

#### pytorch

如果要使用GPU加速pytorch,则需要在安装时,指定GPU 版本.这一点同老版本的tensorflow 类似.

详见[pytorch 官网](https://pytorch.org/get-started/locally/)

但他的方便之处在于,不需要显式安装 CUDA 和 cudnn.对于初学者很友好.

#### 如何更方便的配置 tensorflow GPU 环境

由于:

- pytorch 自带CUDA 的动态库(相当于部分CUDA toolkit)
- pytorch 和 tensorflow 运行时,仅需CUDA 动态库

所以也可以选择同时安装 GPU 版本的 pytorch,和 tensorflow.

在使用tensorflow 做开发时,仅需在项目文件最外层,做如下操作,即可利用GPU 加速:

```
try:
    import torch
except _:
    pass
import tensorflow as tf
```

**Why?** 输出环境变量看看明白了

## 相关检索

- [动态库和静态库](https://cn.bing.com/search?q=%E5%8A%A8%E6%80%81%E5%BA%93+%E9%9D%99%E6%80%81%E5%BA%93&qs=n&form=QBRE&sp=-1&pq=%E5%8A%A8%E6%80%81%E5%BA%93+j&sc=10-5&sk=&cvid=E18652CF2D6A4CFEB9991C78556CA726&ghsh=0&ghacc=0&ghpl=)
- [cuda安装](https://cn.bing.com/search?q=cuda%E5%AE%89%E8%A3%85&cvid=d6b2f51b1e1e4021822b8a83a1ae00f4&aqs=edge..69i57j0l8.262j0j1&FORM=ANAB01&PC=U531)
- [cuda、cudnn 区别](https://cn.bing.com/search?q=cuda+cudnn+%E5%8C%BA%E5%88%AB&qs=n&form=QBRE&sp=-1&pq=cuda+cudnn+qu%27bie&sc=0-17&sk=&cvid=A3B63060D7FB4AB88FB24C1FCDB96CBD&ghsh=0&ghacc=0&ghpl=)
