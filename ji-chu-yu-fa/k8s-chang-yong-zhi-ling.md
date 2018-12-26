# K8s常用指令

### Kubectl taint nodes

目前taint指令只在一个地方见到过，设置master节点同时也是工作节点的地方。 默认情况下Master节点是不参与工作的，但你也可以让它部署一点工作pod

```text
# 允许 Master 部署 Pod
kubectl taint nodes <master_node_name> node-role.kubernetes.io/master- --overwrite

# 禁止 Master 部署 Pod
kubectl taint nodes <master_node_name> node-role.kubernetes.io/master=:NoSchedule --overwrite
```



