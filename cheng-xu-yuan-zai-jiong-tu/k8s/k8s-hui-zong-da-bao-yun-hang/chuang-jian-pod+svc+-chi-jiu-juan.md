---
description: https://blog.csdn.net/weixin_38748858/article/details/102514721
---

# 创建pod+svc+持久卷

### Deployment

```yaml
apiVersion: apps/v1
kind: Deployment  # 类型是部署
metadata:
  name: mysql-deployment  # 对象的名字
#   namespace: myk8s-ns     # namespace 不指定默认使用default
spec:
  selector:
    matchLabels:
      app: mysql #用来绑定label是“mysql”的Pod
  strategy:
    type: Recreate
  template:   # 开始定义Pod
    metadata:
      labels:
        app: mysql  #Pod的Label，用来标识Pod
    spec:
      containers: # 开始定义Pod里面的容器
        - image: mysql:5.7
          name: mysql-con
          imagePullPolicy: Never
          env:   #  定义环境变量
            - name: MYSQL_ROOT_PASSWORD  #  环境变量名
              value: root  #  环境变量值
            - name: MYSQL_USER
              value: dbuser
            - name: MYSQL_PASSWORD
              value: dbuser
          args: ["--default-authentication-plugin=mysql_native_password"]
          ports:
            - containerPort: 3306 # mysql端口
              name: mysql

```

### Svc

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql-service
  labels:
    app: mysql
spec:
  type: NodePort
  selector:
      app: mysql
  ports:
  - protocol : TCP
    nodePort: 30306
    port: 3306
    targetPort: 3306

```

### Local持久卷

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  storageClassName:  standard #持久卷存储类型，它需要与持久卷申请的类型相匹配
  local:
    path: /home/vagrant/database/mysql #宿主机的目录
  nodeAffinity:
    required:
      nodeSelectorTerms:
        - matchExpressions:
            - key: kubernetes.io/hostname
              operator: In
              values:
                - minikube # Node的名字
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
  labels:
    app: mysql
spec:
  accessModes:
    - ReadWriteOnce
  # storageClassName:  # 这里的存储类型注释掉了
  resources:
    requests:
      storage: 1Gi #1 GB

```

如果不知道Node名字，可用如下命令查看：

```
vagrant@ubuntu-xenial:/$ kubectl get node
NAME       STATUS   ROLES    AGE    VERSION
minikube   Ready    master   6d3h   v1.15.2
```
