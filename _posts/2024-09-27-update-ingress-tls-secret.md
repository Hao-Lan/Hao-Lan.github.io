---
layout: post
title: k8s ingress tls secret update
date: 2024-09-27 10:07:00 +0800
categories: [ k8s, amazing ]
---


## When a reload is required

The next list describes the scenarios when a reload is required:

- New Ingress Resource Created.
- TLS section is added to existing Ingress.
- Change in Ingress annotations that impacts more than just upstream configuration. For instance annotation does not require a reload.load-balance
- A path is added/removed from an Ingress.
- An Ingress, Service, Secret is removed.
- Some missing referenced object from the Ingress is available, like a Service or Secret.
- A Secret is updated.

## 相关阅读

- [How it works](https://kubernetes.github.io/ingress-nginx/how-it-works/)
- [k8s中ingress-nginx-controller什么时候会重新加载以及如何避免频繁加载配置的策略](https://www.cnblogs.com/chuanzhang053/p/16169759.html#:~:text=1%E3%80%81%E6%A6%82%E8%BF%B0%20%E6%88%91%E4%BB%AC%E9%83%BD%E7%9F%A5%E9%81%93)
- [如何在线更新k8s集群ingress即将要到期的SSL证书](https://blog.csdn.net/qq_23191379/article/details/109290435#:~:text=%E5%87%86%E5%A4%87%E5%A5%BD%E6%96%B0%E7%94%B3%E8%AF%B7%E7%9A%84sg.)