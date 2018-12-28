---
description: 你可能对于pod已经有了一个零碎的认知，现在开始完全认识pod
---

# 详细介绍Pods

### 一个更加详细的定义

Pod， 豌豆荚。Pod就是外面那层皮，容器就是里面的豆子。一个pod里面会有一个或者多个容器，独享或者共享存储网络，容器模板。 一个pod里所有的内容都必须座落于同一个机器里，接受统一管辖。诸多容器之间，甚至连[上下文都是一致的](https://xiaohanliang.gitbook.io/notes/~/edit/drafts/-LUj75rPiQtVrenRPlz8/ji-chu-yu-fa/pods/xiang-xi-jie-shao-pods/cun-chu-wang-luo-wo-dong-shen-me-shi-shang-xia-wen)。  
  
Pod的存在描述了一种画面，画面里一个App运行在一台机器上，或者多个紧密关联的App运行在同一台机器上，你如果把多个容器布置在同一个Pod里，他们就好像运行在同一台机器上那样，共享着同样的存储，同一个IP那样

#### 

