# Pod的生命周期

## The life of a Pod 

Pod被设计成为一种生命周期短暂的单位。其基本的生命周期基本可以概括为 创建，分配一个唯一的ID，安排到工作节点上去，然后在这个节点上一直呆到最后被销毁。 如果这个工作机器本身出现故障，并且在规定时间内也没能恢复，接下来我们就着手安排去删掉这个Pod。

一旦Pod被创建以后，它是不能被迁移到别的工作机器上去的。 我们只会在别的机器上去创建一个一模一样的Pod, 连Pod名字都能保证一样，但是IP会是新的IP。根据你的设置，内存片段可以跟Pod拥有同样的生命周期，也就是Pod被重建以后内存也会是完全崭新的内存。 根据之前所说，同样也可以做到Pod被删除以后内存保留下来。

## 销毁一个Pod的过程是怎样的

考虑到Pod本质上是系统里的一个进程，进程总会要结束的，不能一直运行着。那么相比较于直接发送一个`KILL`信号，也不给一点Clean Up时间， 我们还是希望能清理干净以后再清理这个pod。所以你作为管理者你就需要知道如何去正确的销毁一个pod，知道什么时候这个pod已经被销毁了，你还要知道怎么去确认这个pod已经被销毁了。

当用户发送了一个销毁请求的时候，在强制销毁之前，系统会记录下一个既定好的缓冲时间。首先我们会向pod内每一个容器发送一个`TERM`信号。等缓冲时间一过，我们就朝所有的主进程发送KILL信号， 此时这个pod就从`API Server`中被删掉了。如果在缓冲期内kubelet发生重启，那么重启结束会重新尝试删除。

1. 用户发送销毁相关的命令，默认的缓冲时间是30秒
2. 系统通过`API-Server`收到相关销毁指令，系统记录下此事件以及缓冲时间。超过了缓冲时间，这个Pod就会被认为“已经销毁” ， 并在`etcd`\(系统信息\)中移除这个pod的相关信息
3. 在客户端指令中再次查看这个pod，它的状态变成了`Terminating`
4. \(与3同步发生\) API-Server因为已经记录下这次操作，因此当Kubelet看到了这种标记的时候，Kubelet开始着手清理自己管理着的pod
   1. Pod内所有容器收到一个`TERM`信号
   2. 如果Pod内存在有一个`preStop`工件，并且`preStop`在缓冲期结束仍旧在运行，API-Server中会记录下`preStop`的标志，并且将缓冲期延长两秒
5. \(与3同步发生\) 将这个Pod从可提供服务的列表中删除，`replication controller` 记录下这个pod不再是一个运行中的pod。`Load Balancer` 不再给这个pod分配任务
6. 缓冲期结束，向pod内所有进程发送`SIGKILL` 信号
7. Kubetlet意识到销毁过程已经结束，通过API-Server将缓冲时间重新设置为0，象征销毁过程终止，同时将这个Pod从`etcd`\(系统信息\)中永久移除，此时从客户端我们将不能再看到这个pod

#### 终止相关的指令

* `kubelet delelte --grace-period=<seconds>` 手动设置缓冲时间
  * `seconds = 0`强制删除命令，API-Server一旦收到强制删除命令根本就不会等Kubelet确认，因为seconds=0本身就是由kubelet来设置的，所以你设置seconds=0相当于模拟出了Kubelet通过API-Server将缓冲时间设置为0这一步，直接来到etcd删除掉相关信息这一步骤。
  * 同时Kubelet则向自己的pod直接发送`SIGKILL`信号



## 查看Pod当前的状态

Pod状态毫无疑问是一个非常重要的信息，是我们诊断错误并判断下一步策略最重要的信息来源。 PodStatus就能够详细描述Pod的状态，以下列出两个主要的field： `Phase` + `Condition` 

#### Phase

| 阶段 | 备注 |
| :--- | :--- |
| `Pending` | K8s集群已经接受了这个pod，正在准备创建，但是由于容器镜像目前尚未准备好，例如正在尝试联网下载容器镜像 |
| `Running` | Pod已经被分配至节点，pod内所有容器均已被创建。已经有容器已经开始正常工作了 |
| `Succeeded` | Pod内所有容器均已正常终止进程，容器没有重启的打算 |
| `Failed` | Pod内所有容器均已终止进程，但是有的容器终止的过程中出现了一些问题，也就是说，可能是exit-code非零，或者是由系统强制终止进程 |
| `Unknown` | 出现某些问题。 现在无法获取Pod状态，大部分时候是由于网络问题无法连接 |

#### Conditions

* `lastProbeTime` - 最后一次查询pod状态是发生在什么时候
* `lastTransitionTime` - 最后一次Pod-Phase状态发生变更是什么时候
* `message` - 日志，详细记录关于pod-phase状态变更的细节
* `reason` - 用一个词简要描述为什么Pod-Phase发生变更
* `status` - True/False/Unknown
* **`type`** - 
  * `Pod-Scheduled` - 目前这个pod已经被安排至一个节点
  * `Ready` - 目前这个pod已经准备就绪可以开始工作，应该被加到负载均衡器上
  * `Initialized` - 容器已经成功初始化
  * `Unschedulable` - 目标机器缺少资源，无法将节点安排至目标机器上工作
  * `ContainersReady` - 所有容器已经准备好开始工作

#### Remaining Fields - [详细描述点击这里](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.13/#podstatus-v1-core)

| Fields | Description |
| :--- | :--- |
| containerStatuses | 一个数组，用于描述pod中每个容器对应的状态 |
| hostIP | 工作机的IP地址，如果Pod尚未参与分配，值空 |
| podIP | Pod被分配到的IP地址，如果pod没分配到地址，值空 |

## 容器健康状态检测

工作机上的Kubelet每隔一段时间就会针对容器执行健康状态检测。 每个容器自身都会携带一个Handler，Kubelet通过调用这个Handler来执行健康状态检测，Handler包含有以下三种：

1. ExecAction - 这种Handler在容器内部执行一些既定的命令，指令本身是一种带有返回值的函数，只哦要这些返回值全都是0，那么诊断结果就是健康
2. TCPSocketAction - 这种Handler针对容器IP + 端口展开TCP连接，如果端口开放，诊断结果为健康
3. HTTPGetAction - 这种Handler针对容器IP+ 端口展开HTTP发送请求，HTTP请求返回值大于200小于400，诊断结果就是健康的

每一种诊断都拥有三种结果 成功\(条件满足\) / 失败\(条件不满足\) / 未知\(都没执行完，诊断结果拿不到，什么操作都不做\)  。 除此之外，针对一个正在运行中的容器，Kubelet还可以执行以下诊断：

1. livenessProbe - 这个诊断用于判断容器是不是还在运行中。 如果诊断结果表示容器已经不在运行了，Kubelet会销毁这个容器，然后执行重新生成流程。如果容器不去执行livenessProbe，默认状态是它一直在正在运行中
2. readinessProbe - 这个诊断用于判断容器是否已经准备好开始提供服务。如果诊断结果表示容器

