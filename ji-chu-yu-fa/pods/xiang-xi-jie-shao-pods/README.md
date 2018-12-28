---
description: 你可能对于pod已经有了一个零碎的认知，现在开始完全认识pod
---

# 详细介绍Pods

## Introduction

Pod， 豌豆荚。Pod就是外面那层皮，容器就是里面的豆子。一个pod里面会有一个或者多个容器，独享或者共享存储网络，容器模板。 一个pod里所有的内容都必须座落于同一个机器里，接受统一管辖。除此之外，你还可以对于Pod内诸多容器，做出[Context上的规定](https://xiaohanliang.gitbook.io/notes/~/edit/drafts/-LUoBV2QazpFJ1fo8FdO/ji-chu-yu-fa/pods/xiang-xi-jie-shao-pods/cun-chu-wang-luo-wo-dong-shen-me-shi-shang-xia-wen)  
  
Pod的存在描述了一种画面，画面里一个App运行在一台机器上，或者多个紧密关联的App运行在同一台机器上，你如果把多个容器布置在同一个Pod里，他们就好像运行在同一台机器上那样，共享着同样的存储，同一个IP那样

Pod中容器之间拥有相同的IP地址与不同的端口，他们之间的相互交流可以通过localhost, 如果你想你甚至可以通过进程间通信的方式来做，就比如Semaphore与共享内存的方式完成进程间通信。 但是如果这两个容器不在同一个Pod里面，他们就不能通过进程间通信的方式交流，只能通过



