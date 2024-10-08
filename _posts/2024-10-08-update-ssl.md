---
layout: post
title: 基于阿里云免费SSL证书,周期更新ingress ssl
date: 2024-09-27 10:07:00 +0800
categories: [ k8s, memo ]
---

以前,阿里云可以免费申请为期一年的个人测试 SSL 证书.

但近期, 免费的个人测试版SSL证书有效期仅为3个月了.

## 进入数字证书管理

![数字证书管理](/assets/img/2024-10-08/ssl_manager.png)

## 进入SSL 证书

![SSL 证书](/assets/img/2024-10-08/into_ssl.png)

## 创建免费证书

![创建证书](/assets/img/2024-10-08/create_free_ssl.png)
![输入参数](/assets/img/2024-10-08/input_param.png)

**注意**:阿里云在创建免费证书环节,需要预先购买证书（此处已经购买免费的个人测试证书），否则会提示无证书，需要点击购买。

## 配置验证

![DNS 配置](/assets/img/2024-10-08/dns_config.png)

创建证书的参数，仅仅是用于发起审批。其中有个审批环节是需要对方确认域名是你的。
所以需要配置了临时子域名供对方实测.通过则进入人工环节——人工审批环节，平均用时12分钟左右,但其标注1-2天内都正常。

**注意**:临时子域名在发起申请后,在验证环节详情里面有,需要在看到这个后, 再去域名控制系统配置.在整个环节结束后,该子域名可以删除.

## 下载证书

![ssl download](/assets/img/2024-10-08/ssl_download.png)

解压得到后缀为 `.pem` 和 `.key` 的两个文件

## K8S 创建secret

```shell
kubectl create secret tls xxx.xxx.com.cn --cert=xxx.pem --key=xxx.key -n app-deploy
```

其中:

  - `kubectl`是k8s 管理工具,需要提前在使用的电脑上配置好
  - `xxx.xxx.com.cn` 是创建的secret的名字
  - `cert` 和 `key` 分别指向下载的证书文件
  - `-n` 特指`namespace`,此处需要是`app-deploy`

需要注意的是, ssl 在做更新时, 新的secret 名字需要与旧的secret 相同.

因为[**nginx ingress 使用新的SSL名字,不会生效**](/2024/09/27/update-ingress-tls-secret.html)


## [证书在线检查](https://myssl.com/)

![Check ssl](/assets/img/2024-10-08/check_ssl.png)


