# StepByStep - 网络安装上

从上一篇开始，现在你已经拥有一个Master以及数个工作节点了，他们都初始化了，看起来很好，但是

1. 工作节点:       状态全是NotReady
2. Master-Pod:  CoreDNS并没有准备好, 它们的状态全都是ContainerCreating

出现这种情况，是因为这些节点之间的网络并没有被打通, 根本就是断网的状态

