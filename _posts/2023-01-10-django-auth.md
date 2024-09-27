---
layout: post
title: "Django 授权、rest-framework 验证"
date: 2023-01-10 15:36:00 +0800
categories: [ Django,python,memo ]
---

#### backends

django 自带的backends,目的是用于登录授权.

它所希望的效果是,通过扩展开发backends,完成其他系统的对接.

然后通过修改配置文件(settings.py)完成终极目标,而不需要以侵入的方式,去修改原本的接口中关于登录部分的代码.

#### authentication

restframework 的 authentication,应该处于校验.

即,通过访问接口时携带的信息,**找到User**.

比如 Cookie,Session,亦或者jwt等.

它不被设计来做登录,仅该用于查找用户的过程.

#### permission_classes

这里的权限,是前面authentication寻找用户的后处理.

此时用户可能已经找到了User,但可能也没找到User(匿名用户).

这里的配置就用于判断:此时的已登录用户或者匿名用户,是否有权限继续访问.

所以这里的配置,比较粗,包括: 不需要登录,需要登录,管理员用户,管理员或者只允许读等(django 自身的permission).

#### 回顾

开发时很多功能都能各自为营完成,但仍需要一个更优雅的,可复用更强的代码组织形式.
