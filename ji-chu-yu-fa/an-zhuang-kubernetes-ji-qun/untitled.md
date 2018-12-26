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

所以可以看的出来，我们的kubernetes版本是v1.13.1还有一些其他所需要的内容，所以准备一个shell脚本，在images中，将其中的版本号换成你现在正在用的版本号，确保你拉的就是正确的

{% code-tabs %}
{% code-tabs-item title="image\_list.sh" %}
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

