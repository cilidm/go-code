# Page 1

Use "kubectl explain " for a detailed description of that resource (e.g. kubectl explain pods). See 'kubectl get -h' for help and examples.&#x20;

### 这里可以配置一下别名，因为kubectl get 用的太多

```bash
cat >> ~/.bashrc << EOF 
alias kg='kubectl get'
alias k='kubectl'
alias kd='kubectl describe pods'
alias ke='kubectl explain'
alias ka='kubectl apply'
EOF
```

这里是我配置的别名\
`source ~/.bashrc` 执行下就够了

### 常用的命令有

```bash
kg namespaces
kg node
kg pods(查找pod)
kg service(查找service)
kg deploy(查找deploy)
kg sts(查找statefulsets)
kg pv(查找persistentvolumes)
kg pvc(查找persistentvolumeclaims)
kg cm(查找configmaps)
kg ing (查找limitranges)
kd #pod-name(#pod-name 是你集群中的pod名称)
k logs -f pod/#pod-name
k edit #pod-name
kg pods #pod-name -o wide 查看pod在那台主机上
kg pods #pod-name -o yaml 查看pod创建的yaml文件
```

\------ 同样的 -o 参数也可以对应到其他组件上 service deployment等等上
