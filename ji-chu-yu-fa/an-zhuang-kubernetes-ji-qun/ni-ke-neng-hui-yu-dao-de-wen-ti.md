# 你可能会遇到的问题

> 除去我们之前提到的最明显的，因为镜像不存在而产生的init失败  
> 根据你创建机器时环境的细微差别，你仍然可能会遇到很多其他问题  
> 不要焦虑，我们都帮你准备好了，遇到我真是算你走运



### X509错误 - Certificate

1. 网上别人这么说：如果你的虚拟机包含有多张网卡，你需要加上 --apiserver-cert-extra-sans参数来涵盖多张网卡所对应的不同的私有的地址，这个错误出现的原因是K8s之间相互交流依托于非常多的CA证书来完成。这些证书的生成于你的私有IP之上，用来证明你的身份。  但是你拥有很多私有IP你又不说，可能`kubectl`发起于一个地址但是`Node join` 却去了另外一个地址
2. **我自己并重新init了一次，但是却没有执行cp -i /etc/kubernetes/admin.conf $HOME/.kube/config 这时候你用的是上一次的证书，这时候你虽然IP是对的上的，但是次数匹配不上，因此产生了X509权限错误**： 如果我kubeadm reset并重新init了一次，但是却没有执行cp -i /etc/kubernetes/admin.conf $HOME/.kube/config 这时候你用的是上一次的证书，这时候你虽然IP是对的上的，但是次数匹配不上，因此产生了X509权限错误



### Failed to run Kubelet & cgroupfs  

```text
error: failed to run Kubelet: failed to create kubelet: misconfiguration: kubelet cgroup driver: "cgroupfs" is different from docker cgroup driver: "systemd"​
```

你目前使用的docker cgroup driver是systemd，但是Kubelet使用的cgroup driver是cgroupfs。按照它的意思来说这两个应该是保持一致的，显然你这里并不一致，ok现在我们就把他们搞得一致

**1. 先看看Docker使用的什么启动文件** ： Docker用的是croupfs

```bash
$ docker info | grep Cgroup
> Cgroup Driver: cgroupfs
```

**2. 修改Kubelet的启动文件：**并添加如下参数使得driver与docker保持一致

```text
$ vim /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

> Environment="KUBELET_CGROUP_ARGS=--cgroup-driver=cgroupfs"
# 或 #
> Environment="KUBELET_CGROUP_ARGS=--cgroup-driver=cgroupfs --pod-infra-container-image=registry.cn-hangzhou.aliyuncs.com/google_containers/pause-amd64:3.1"
```

**3. 重启Kubelet**

```bash
$ systemctl daemon-reload
$ systemctl enable kubelet
$ systemctl restart kubelet
```



### ERROR: FileContent--proc-sys-net-bridge-bridge-nf-call-iptables

```text
[init] Using Kubernetes version: v1.9.8
[init] Using Authorization modes: [Node RBAC]
[preflight] Running pre-flight checks.
	[WARNING Hostname]: hostname "localhost.master" could not be reached
	[WARNING Hostname]: hostname "localhost.master" lookup localhost.master on 8.8.8.8:53: no such host
	[WARNING FileExisting-crictl]: crictl not found in system path
[preflight] Some fatal errors occurred:
	[ERROR FileContent--proc-sys-net-bridge-bridge-nf-call-iptables]: /proc/sys/net/bridge/bridge-nf-call-iptables contents are not set to 1
[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
```

这种问题出现是因为防火墙或者`iptable`对`bridge`数据进行处理导致的，我们现在要禁止防火墙的工作

```text
$ cat <<EOF >  /etc/sysctl.d/k8s.conf
> net.bridge.bridge-nf-call-ip6tables = 1
> net.bridge.bridge-nf-call-iptables = 1
> EOF

$ sysctl -p /etc/sysctl.d/k8s.conf
```



###  ERROR Swap: Please disable swap

首先明白一点，如果你的swap=on 你是绝对没办法init你的kubeadm的，按照下面的方法去禁用swap，并测试一下：输入`top` 命令，若 KiB Swap一行中 total 显示 0 则关闭成功

```text
$ vim /etc/fstab  注释掉引用swap的行
$ sudo swapoff -a
```

再说说为什么，首先我们限制了一个Pod最大可能存储内容的数量，由于内存Swap机制，如果一个Pod容量到达上限，它可能会被swap到一个新的内存区域里，从而继续申请内存。但是如果swap出的空间不足，这两个Pod可能会被直接杀掉，这两个决定都不利于我们管理Pod，因此禁止使用Swap最好

