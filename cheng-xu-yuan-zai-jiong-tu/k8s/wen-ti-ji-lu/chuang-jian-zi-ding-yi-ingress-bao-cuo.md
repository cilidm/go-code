---
description: >-
  创建自定义ingress报错：Internal error occurred: failed calling webhook
  “validate.nginx.ingress.kubernetes.io
---

# 创建自定义ingress报错

## 问题描述：

当我使用ingress-demo.yaml文件创建自定义的ingress时  
使用命令创建：`kubectl apply -f ingress-demo.yaml`  
报如下错误：

```bash
Internal error occurred: failed calling webhook "validate.nginx.ingress.kubernetes.io": Post 
https://ingress-nginx-controller-admission.kube-system.svc:443/networking/v1beta1/ingresses?
timeout=10s: dial tcp 10.0.0.5:8443: connect: connection refused
```

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-demo
  namespace: test
spec:
  rules:
  - host: test.test.test
    http:
      paths:
      - path: /
        backend:
          serviceName: test-service
          servicePort: 80

```

原因分析： 我刚开始使用yaml的方式创建nginx-ingress，之后删除了它创建的命名空间以及 clusterrole and clusterrolebinding ，但是没有删除ValidatingWebhookConfiguration ingress-nginx-admission，这个ingress-nginx-admission是在yaml文件中安装的。当我再次使用helm安装nginx-ingress之后，创建自定义的ingress就会报这个错误。

解决方案： 最后参考下面的文章解决此问题

使用下面的命令查看 webhook 

```bash
kubectl get validatingwebhookconfigurations
ingress-nginx-admission
```

删除ingress-nginx-admission

```bash
kubectl delete -A ValidatingWebhookConfiguration ingress-nginx-admission
```



