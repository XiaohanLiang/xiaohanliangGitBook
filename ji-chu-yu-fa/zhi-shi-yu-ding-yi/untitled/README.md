# 从Master组件开始

> 在教程中我们尽量使用了最快，最简单的方式让所有人都能在最快的时间里生成一个Master以及一个工作节点。回忆之前有一个步骤叫做 kubeadm config images list  。我们看看它返回了什么，来判断我们需要准备那些镜像。

### Kubeadm config images list 到底是想说明什么

什么是Master ? 首先要明白，Master是一个抽象的概念，我们用一个机器代表了一个Master，但这并不是说一个Master就一定是一台机器，两个Master就是两台机器。

Master由许多组件所构成，这么多个组件垒在一起逻辑上共同形成了一个叫Master的东西。然后这个东西拥有许多功能，管理着整个集群，我们就叫这一小堆东西Master。同样我们把观念再拓宽一点，只要一堆东西能够提供我们所有需要的功能，我们就可以叫它Master啊。我不管你这堆东西是怎么实现的，单点也好或者怎么一个策略都行，只要你能提供我要的功能，你就是Master。

由什么构成了Master?  也就是这一小堆东西到底是由什么构成？ 这就是这条命令的意义所在，我们通过这条命令，在当前情况下我们的Master应当由那些组件所构成

```bash
$ kubeadm config images list
> k8s.gcr.io/kube-apiserver:v1.13.1
> k8s.gcr.io/kube-controller-manager:v1.13.1
> k8s.gcr.io/kube-scheduler:v1.13.1
> k8s.gcr.io/kube-proxy:v1.13.1
> k8s.gcr.io/pause:3.1
> k8s.gcr.io/etcd:3.2.24
> k8s.gcr.io/coredns:1.2.6
```



### 由此我们做出总结...

* 关于Kubernetes 集群的管理工具包含有： Kubectl / Kubeadm .... 这些都是我们通过apt-get下载的
* Kubernetes集群中, Master节点的组件有：ApiServer / Scheduler ... 这些都是我们pull下来的镜像

