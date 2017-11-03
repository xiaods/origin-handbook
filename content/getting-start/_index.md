+++
title = "Origin本地环境创建 for Mac"
description = ""
weight = 1
alwaysopen = true
+++

## 环境需要

1. 下载origin源码,并参考HACKING.md文档在本地搭建开发环境

```
$ hack/env ${COMMAND}
```
> 注意：hack/env会先启动一个centos环境并挂载源码，然后构建源码并生成可执行文件。

```
$ hack/env make release

$ hack/env hack/build-base-images.sh

$ hack/env hack/build-local-images.py
```


2. 启动cluster

```
$ oc cluster up --version=latest
```



