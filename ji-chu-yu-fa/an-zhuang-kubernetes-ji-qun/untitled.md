# StepByStep

> OS: Ubuntu 16.04 LTS   
> 网络: 国内普通网络

### 创建虚拟机

* 双核，一定要双核（为什么一定要双核）
* 8GB  （低内存会发生什么）

以上规格的机器要有两台，一台用作Master，另一台用做Slave，pod会安排在slave机器上使用

### 在所有机器上：修改软件源并安装K8s套件

1.修改你的 /etc/apt/sources.list 来安装docker

{% code-tabs %}
{% code-tabs-item title="etc/apt/sources.list" %}
```bash
# 系统安装源 & 关于docker的安装源在这里
deb http://mirrors.aliyun.com/ubuntu/ xenial main restricted
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted
deb http://mirrors.aliyun.com/ubuntu/ xenial universe
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates universe
deb http://mirrors.aliyun.com/ubuntu/ xenial multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse
```
{% endcode-tabs-item %}
{% endcode-tabs %}

2. 然后在命令行敲出如下命令，来安装kubeadm以及相应kubernetes组件

{% code-tabs %}
{% code-tabs-item title="terminal" %}
```bash
$ cat <<EOF > /etc/apt/sources.list.d/kubernetes.list
> deb http://mirrors.ustc.edu.cn/kubernetes/apt kubernetes-xenial main
> EOF
```
{% endcode-tabs-item %}
{% endcode-tabs %}

3. 更新软件源，下面会报出一些关于GPG证书的错误，不用理会这些并不会影响我们安装K8s套件

{% code-tabs %}
{% code-tabs-item title="terminal" %}
```bash
$ apt-get update

Hit:1 http://mirrors.aliyun.com/ubuntu xenial InRelease
Hit:2 http://mirrors.aliyun.com/ubuntu xenial-updates InRelease                                                                     
Hit:3 http://mirrors.aliyun.com/ubuntu xenial-backports InRelease                                                                   
Get:4 http://mirrors.ustc.edu.cn/kubernetes/apt kubernetes-xenial InRelease [8,993 B]               
Ign:4 http://mirrors.ustc.edu.cn/kubernetes/apt kubernetes-xenial InRelease
Fetched 8,993 B in 0s (40.4 kB/s)
Reading package lists... Done
W: GPG error: http://mirrors.ustc.edu.cn/kubernetes/apt kubernetes-xenial InRelease: The following signatures couldn't be verified because the public key is not available: NO_PUBKEY 6A030B21BA07F4FB
W: The repository 'http://mirrors.ustc.edu.cn/kubernetes/apt kubernetes-xenial InRelease' is not signed.
N: Data from such a repository can't be authenticated and is therefore potentially dangerous to use.
N: See apt-secure(8) manpage for repository creation and user configuration details.
```
{% endcode-tabs-item %}
{% endcode-tabs %}

4. 安装docker以及K8s对应套件, 在进行下一步之前确保下面这些软件已经安装上了

{% code-tabs %}
{% code-tabs-item title="terminal" %}
```bash
apt-get install -y docker.io kubelet kubernetes-cni=0.6.0-00 kubeadm
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### 在所有机器上：准备镜像

下面就是最重要的一步，拉镜像，否则如果所需要的镜像缺失它会立刻去Google哪儿拉，这很麻烦，所以我们需要先准备好所有需要的镜像。 我们先来看看我们需要那些镜像以及他们对应的版本

```bash
$ kubeadm config images list

k8s.gcr.io/kube-apiserver:v1.13.1
k8s.gcr.io/kube-controller-manager:v1.13.1
k8s.gcr.io/kube-scheduler:v1.13.1
k8s.gcr.io/kube-proxy:v1.13.1
k8s.gcr.io/pause:3.1
k8s.gcr.io/etcd:3.2.24
k8s.gcr.io/coredns:1.2.6
```

所以可以看的出来，我们的kubernetes版本是v1.13.1还有一些其他所需要的内容，所以准备一个shell脚本，在images中，将其中的版本号换成你现在正在用的版本号，确保你拉的就是你所需要的版本

保存后，执行 `bash images_list.sh` 去从阿里云那里拉取镜像，for循环可以分成两个部分，第一部分是执行去阿里云那里拉取镜像，但是拉回来的都是阿里的名字，于是第二部分我们把它改成k8s.gcr的名字，确保kubeadm能根据镜像名称找到这个镜像

{% code-tabs %}
{% code-tabs-item title="images\_list.sh" %}
```bash
images=(
    kube-apiserver:v1.13.1
    kube-controller-manager:v1.13.1
    kube-scheduler:v1.13.1
    kube-proxy:v1.13.1
    pause:3.1
    etcd:3.2.24
    coredns:1.2.6
)

for imageName in ${images[@]} ; do
    docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName
    docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName k8s.gcr.io/$imageName
done
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### 初始化k8s的master节点

下面就会是最重要的一步，集群能不能起来最重要的就是这一步

```bash
$ kubeadm init --kubernetes-version=<your_version> 
```

一定要带上这个version来指定你要初始化什么版本，这相当于告诉kubeadm你已经都有现成的东西了，直接来用就好了，否则它会要先去Google哪儿先检查并拉取最新版本的kubenetes.

#### kubeadm init常见卡住的情况怎么处理

```bash
$ kubeadm init --kubernetes-version=v1.13.1
[init] Using Kubernetes version: v1.13.1
[preflight] Running pre-flight checks
	[WARNING Hostname]: hostname "dev1" could not be reached
	[WARNING Hostname]: hostname "dev1": lookup dev1 on 103.224.222.222:53: no such host
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
<----------------------然后就卡在这里了------------------------>
^C
```

那就说明它去Google哪儿拉镜像去了, 过了很长时间以后它会返回报错信息，信息不用看都知道内容如下：“ 我需要k8s.gcr.XXXX 但是我没办法拉取”   。也正是你的镜像没有准备好，它才会去Google哪儿拉镜像回来，根据它报错的信息我们来处理，举个例子, 你的报错信息可能会长这样

```bash
[ERROR ImagePull]: 

failed to pull image k8s.gcr.io/kube-apiserver:v1.13.0: 
output: Error response from daemon: Get https://k8s.gcr.io/v2/: 

net/http: request canceled while waiting for connection 
(Client.Timeout exceeded while awaiting headers)
error: exit status 1
```

看见了那个大大的Client.Timeout了吗？看了那个大大的k8s.gcr了吗？ 在这个例子中他说它需要一个叫的镜像，但是你没有，所以他去了Google哪儿。 所以怎么办呢，解决方法有没有写对

* 检查一下你的 images\_list.sh，看看版本号有没有写对，有没有把阿里的名字改过来
* 如果`images_list.sh`没问题，试一下 docker images 来看看你的镜像库里到底有没有需要的镜像

#### 正确的init结果

```bash
[init] Using Kubernetes version: v1.13.1
[preflight] Running pre-flight checks
	[WARNING Hostname]: hostname "dev1" could not be reached
	[WARNING Hostname]: hostname "dev1": lookup dev1 on 103.224.222.222:53: no such host
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[kubelet-start] Activating the kubelet service
     <...一大堆类似的信息...>
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes master has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of machines by running the following on each node
as root:

  kubeadm join 192.168.0.66:6443 --token r4fu1b.d5sb52nxxseqqs89 --discovery-token-ca-cert-hash sha256:88ed8f4807173291b1d34195014841d9f0ad07cff8734b46bd403a0011a2b90b
```

如果结果是以上的这样的，那么你的init基本上是成功了，在以上的报告中它告诉了你了两个重要的信息

#### 1. 你需要保存一下你的配置以及你的授权信息

你作为管理员可以去干涉K8s集群内部的事务，是因为你在初始化这个master的时候创建了一把密钥，你把这个密钥放到.kube下来证明你作为管理员的身份，执行  

```bash
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config

# 如果你不保存，每次想对Cluster有什么操作就会提示
The connection to the server localhost:8080 was refused - did you specify the right host or port?
```

#### 2. 告诉你如何让别的节点加入你这个Master

确保别的节点也有那一大堆镜像，如果加入过程中卡住了，那么就是这个节点缺少镜像

```bash
$ kubeadm join 192.168.0.66:6443 --token r4fu1b.d5sb52nxxseqqs89 --discovery-token-ca-cert-hash sha256:88ed8f4807173291b1d34195014841d9f0ad07cff8734b46bd403a0011a2b90b
```

### 

### 确认Pod已经加入集群了

通过以下命令来查看各个pod的任务都有哪些，以及他们现在的状态是怎样的，你的node如果需要与master沟通，就需要一个pod来处理沟通相关的任务

```bash
$ kubectl get pod -n kube-system -o wide

NAME                           READY   STATUS              RESTARTS   AGE   IP             NODE   NOMINATED NODE   READINESS GATES
coredns-86c58d9df4-pxk97       0/1     ContainerCreating   0          46m   <none>         dev2   <none>           <none>
coredns-86c58d9df4-tjrj6       0/1     ContainerCreating   0          46m   <none>         dev2   <none>           <none>
etcd-dev1                      1/1     Running             0          45m   192.168.0.66   dev1   <none>           <none>
kube-apiserver-dev1            1/1     Running             0          45m   192.168.0.66   dev1   <none>           <none>
kube-controller-manager-dev1   1/1     Running             0          45m   192.168.0.66   dev1   <none>           <none>
kube-proxy-7lk9f               1/1     Running             0          46m   192.168.0.66   dev1   <none>           <none>
<--------------->
kube-proxy-pt6l2               1/1     Running             0          13m   192.168.0.68   dev2   <none>           <none>
<--------------->
kube-scheduler-dev1            1/1     Running             0          45m   192.168.0.66   dev1   <none>           <none>
```

在上面的图中可以发现有一个叫做proxy的pod，是储存在我们的工作节点里面的，它READY的状态是1/1说明已经完全准备好了，状态也是Running, 这个节点目前没有任何问题，但是

#### Pod问题诊断

这个节点状态很可能就不是这样的，正常情况下应该很快这个节点就已经添加上来了，但是如果这个节点状态是err或者pending一类，而且很久了，那就证明这个节点有问题，关于这种情况，我们查询这个节点的最近的详细活动信息，就能看到到底是因为什么而出了错

```bash
kubectl describe pod -n kube-system <pod_name>
```



### 查看你节点的状态

你的pod没有问题，说明这个小东西已经准备好要开始工作了，但是你的节点呢，节点在加入集群以后需要去跟Master相互沟通，你需要一个节点之间的网络来做到与外界沟通。 

通过以下内容我们可以发现，虽然之前的pod是已经准备好开始工作了，但是因为网络环境本身并没有准备好，因此节点本身并不能开始工作，他们的状态全部显示为`NotReady`

Q:（pod沟通与节点沟通是一回事吗？节点之间如果需要另外的网络那么之前是怎么加入的？为什么是通过内网IP的性质来沟通的？ 如果不在同一个VPC里面是还能不能相互沟通了）

```bash
$ kubectl get nodes

NAME   STATUS     ROLES    AGE     VERSION
dev1   NotReady   master   2m34s   v1.13.1
dev2   NotReady   <none>   2m13s   v1.13.1
```

