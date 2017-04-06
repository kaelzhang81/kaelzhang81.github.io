---
layout:     post
title:      "在mac上运行docker"
subtitle:   "run docker on mac osx"
date:       2016-03-08 22:00:00
author:     "Kaelzhang"
header-img: "img/header/mac docker.png"
catalog:    true
tags:
    - docker
    - mac
---

>Build, Ship and Run Any App, Anywhere！

## 安装Docker
1.到https://www.docker.com/products/docker-toolbox下载docker toolbox
2.安装toolbox

Click the installer link to download.

Install Docker Toolbox by double-clicking the package or by right-clicking and choosing “Open” from the pop-up menu.

The installer launches the “Install Docker Toolbox” dialog.

## 运行Docker
在Spotlight中启动Docker Quickstart Terminal，执行Docker

## 访问私有registry
1.启动
在bash上执行：
>docker-machine ssh

进入虚拟机，并编辑/var/lib/boot2docker/profile文件，增加EXTRA_ARGS:
>EXTRA_ARGS="--insecure-registry 10.86.56.126:5000"

xxxxxx为ip或registry名字docker-registry.com.intra，如果是registry名记得在hosts内加上名字映射。





##### 参考资料：

1.https://github.com/docker