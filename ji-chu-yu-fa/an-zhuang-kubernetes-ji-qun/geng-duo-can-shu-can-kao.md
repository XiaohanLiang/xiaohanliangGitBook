# 更多参数参考

### kubeadm init

 `--pod-network-cidr=10.244.0.0/16` 这个是一个非常重要的指令，接下来所有pod的地址全都会出现在这个区段内，此外为了安装容器网络我们必须加上这个指令\(如果你想安装的网络是Calico这里你必须写`192.168.0.0/16`\)

 `--apiserver-advertise-address=<your_private_address>` 在kubeadm init结束过后会留下一串包含有主机地址的join指令，工作节点通过这个指令加入Master节点，所以在这里你可以指定一下主机的私有地址，当然也可以留白使得kube自行判断主机地址

 `--apiserver-cert-extra-sans=10.0.2.15,192.168.56.100` 如果你的虚拟机绑定了有多张网卡，那么这条指令你就必须加上



