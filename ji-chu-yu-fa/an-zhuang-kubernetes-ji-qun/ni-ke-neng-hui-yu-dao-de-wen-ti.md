# 你可能会遇到的问题

> 除去我们之前提到的最明显的，因为镜像不存在而产生的init失败  
> 根据你创建机器时环境的细微差别，你仍然可能会遇到很多其他问题  
> 不要焦虑，我们都帮你准备好了，遇到我真是算你走运

### X509错误 - Certificate

如果你的虚拟机包含有多张网卡，你需要加上 --apiserver-cert-extra-sans参数来涵盖多张网卡所对应的不同的私有的地址，这个错误出现的原因是K8s之间相互交流依托于非常多的CA证书来完成。这些证书的生成于你的私有IP之上，用来证明你的身份。

但是你拥有很多私有IP你又不说，可能`kubectl`发起于一个地址但是`Node join` 却去了另外一个地址

### Failed to run Kubelet & cgroupfs

```text
error: failed to run Kubelet: failed to create kubelet: misconfiguration: kubelet cgroup driver: "cgroupfs" is different from docker cgroup driver: "systemd"​
```

你目前使用的docker cgroup driver是systemd，但是Kubelet使用的cgroup driver是cgroupfs。按照它的意思来说这两个应该是保持一致的，显然你这里并不一致，ok现在我们就把他们搞得一致

**修改Kubelet的启动文件中的cgroup-driver 大**并

```text
$ vim /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

```

