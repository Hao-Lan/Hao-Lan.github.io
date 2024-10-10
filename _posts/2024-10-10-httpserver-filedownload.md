---
layout: post
title: 快速、临时提供文件下载服务器
date: 2024-09-27 10:07:00 +0800
categories: [ amazing ]
---

```shell
python -m http.server 8080
```

```shell
twistd -no web -p 8080 --path=.
```

前者不支持断点续传,后者提供,但需要额外安装`twisted`

## 相关阅读

- [What is a faster alternative to Python's http.server (or SimpleHTTPServer)?](https://stackoverflow.com/questions/12905426/what-is-a-faster-alternative-to-pythons-http-server-or-simplehttpserver)
