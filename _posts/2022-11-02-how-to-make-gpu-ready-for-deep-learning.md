---
layout: post
title: 便捷配置 Tensorflow GPU 环境
date: 2022-11-02 20:32:00 +0800
categories: [ python,tensorflow, amazing ]
---

- 安装

```shell
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu124
pip install tensorflow
```
- 使用

```python
try:
    import torch
except _:
    pass
import tensorflow as tf
```

- 相关阅读

- [动态库和静态库](https://cn.bing.com/search?q=%E5%8A%A8%E6%80%81%E5%BA%93+%E9%9D%99%E6%80%81%E5%BA%93&qs=n&form=QBRE&sp=-1&pq=%E5%8A%A8%E6%80%81%E5%BA%93+j&sc=10-5&sk=&cvid=E18652CF2D6A4CFEB9991C78556CA726&ghsh=0&ghacc=0&ghpl=)
- [cuda安装](https://cn.bing.com/search?q=cuda%E5%AE%89%E8%A3%85&cvid=d6b2f51b1e1e4021822b8a83a1ae00f4&aqs=edge..69i57j0l8.262j0j1&FORM=ANAB01&PC=U531)
- [cuda、cudnn 区别](https://cn.bing.com/search?q=cuda+cudnn+%E5%8C%BA%E5%88%AB&qs=n&form=QBRE&sp=-1&pq=cuda+cudnn+qu%27bie&sc=0-17&sk=&cvid=A3B63060D7FB4AB88FB24C1FCDB96CBD&ghsh=0&ghacc=0&ghpl=)
