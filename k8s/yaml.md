# Yaml

## K8S-yaml的使用及命令

```bash
# YAML配置文件管理对象
# 对象管理：
# 创建deployment资源
kubectl create -f nginx-deployment.yaml
# 查看deployment
kubectl get deploy
# 查看ReplicaSet
kubectl get rs
# 查看pods所有标签
kubectl get pods --show-labels
# 根据标签查看pods
kubectl get pods -l app=nginx
# 滚动更新镜像
kubectl set image deployment/nginx-deployment nginx=nginx:1.11
# 或者
kubectl edit deployment/nginx-deployment
# 或者
kubectl apply -f nginx-deployment.yaml
# 实时观察发布状态：
kubectl rollout status deployment/nginx-deployment
# 查看deployment历史修订版本
kubectl rollout history deployment/nginx-deployment
kubectl rollout history deployment/nginx-deployment --revision=3
# 回滚到以前版本
kubectl rollout undo deployment/nginx-deployment
kubectl rollout undo deployment/nginx-deployment --to-revision=3
# 扩容deployment的Pod副本数量
kubectl scale deployment nginx-deployment --replicas=10
# 设置启动扩容/缩容
kubectl autoscale deployment nginx-deployment --min=10 --max=15 --cpu-percent=80
```

## POD

实例1：三种策略

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-test
  labels:
     os: centos
spec:
  containers:
  - name: hello
    image: centos:7
    env:
    - name: Test
      value: "123456"
    command: ["bash","-c","while true;do date;sleep 1;done"]
  restartPolicy: OnFailure
```

```
支持三种策略：
Always：当容器终止退出后，总是重启容器，默认策略。
OnFailure：当容器异常退出（退出状态码非0）时，才重启容器。
Never：当容器终止退出，从不重启容器。
```

实例2：数据持久化和共享

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-test1
  labels:
     test: centos
spec:
  containers:
  # 第一个容器
  - name: hello-write
    image: centos:7
    command: ["bash","-c","for i in {1..1000};do echo $i >> /data/hello;sleep 1;done"]
  # 第二个容器
  - name: hello-read
    image: centos:7
    command: ["bash","-c","for i in {1..1000};do cat $i >> /data/hello;sleep 1;done"]
    volumeMounts:
      - name: data
        mountPath: /data
  # 数据卷
  volumes:
  - name: data
    hostPath:
      path: /data
```

实例3：pod的端口映射

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.10
    ports:
    - name: http
      containerPort: 80
      hostIP: 0.0.0.0
      hostPort: 80
      protocol: TCP
   - name: https
     containerPort: 443
     hostIP: 0.0.0.0
     hostPort: 443
     protocol: TCP
```

实例4

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.10
    ports:
    - containerPort: 80
    livenessProbe:
      httpGet:
        path: /index.html
        port: 80
```

详细解析

```yaml
apiVersion: v1 #指定api版本，此值必须在kubectl apiversion中  
kind: Pod #指定创建资源的角色/类型  
metadata: #资源的元数据/属性  
  name: web04-pod #资源的名字，在同一个namespace中必须唯一  
  labels: #设定资源的标签，详情请见http://blog.csdn.net/liyingke112/article/details/77482384
    k8s-app: apache  
    version: v1  
    kubernetes.io/cluster-service: "true"  
  annotations:            #自定义注解列表  
    - name: String        #自定义注解名字  
spec: #specification of the resource content 指定该资源的内容  
  restartPolicy: Always #表明该容器一直运行，默认k8s的策略，在此容器退出后，会立即创建一个相同的容器  
  nodeSelector:     #节点选择，先给主机打标签kubectl label nodes kube-node1 zone=node1  
    zone: node1  
  containers:  
  - name: web04-pod #容器的名字  
    image: web:apache #容器使用的镜像地址  
    imagePullPolicy: Never #三个选择Always、Never、IfNotPresent，每次启动时检查和更新（从registery）images的策略，
                           # Always，每次都检查
                           # Never，每次都不检查（不管本地是否有）
                           # IfNotPresent，如果本地有就不检查，如果没有就拉取
    command: ['sh'] #启动容器的运行命令，将覆盖容器中的Entrypoint,对应Dockefile中的ENTRYPOINT  
    args: ["$(str)"] #启动容器的命令参数，对应Dockerfile中CMD参数  
    env: #指定容器中的环境变量  
    - name: str #变量的名字  
      value: "/etc/run.sh" #变量的值  
    resources: #资源管理，请求请见http://blog.csdn.net/liyingke112/article/details/77452630
      requests: #容器运行时，最低资源需求，也就是说最少需要多少资源容器才能正常运行  
        cpu: 0.1 #CPU资源（核数），两种方式，浮点数或者是整数+m，0.1=100m，最少值为0.001核（1m）
        memory: 32Mi #内存使用量  
      limits: #资源限制  
        cpu: 0.5  
        memory: 32Mi  
    ports:  
    - containerPort: 80 #容器开发对外的端口
      name: httpd  #名称
      protocol: TCP  
    livenessProbe: #pod内容器健康检查的设置，详情请见http://blog.csdn.net/liyingke112/article/details/77531584
      httpGet: #通过httpget检查健康，返回200-399之间，则认为容器正常  
        path: / #URI地址  
        port: 80  
        #host: 127.0.0.1 #主机地址  
        scheme: HTTP  
      initialDelaySeconds: 180 #表明第一次检测在容器启动后多长时间后开始  
      timeoutSeconds: 5 #检测的超时时间  
      periodSeconds: 15  #检查间隔时间  
      #也可以用这种方法  
      #exec: 执行命令的方法进行监测，如果其退出码不为0，则认为容器正常  
      #  command:  
      #    - cat  
      #    - /tmp/health  
      #也可以用这种方法  
      #tcpSocket: //通过tcpSocket检查健康   
      #  port: number   
    lifecycle: #生命周期管理  
      postStart: #容器运行之前运行的任务  
        exec:  
          command:  
            - 'sh'  
            - 'yum upgrade -y'  
      preStop:#容器关闭之前运行的任务  
        exec:  
          command: ['service httpd stop']  
    volumeMounts:  #详情请见http://blog.csdn.net/liyingke112/article/details/76577520
    - name: volume #挂载设备的名字，与volumes[*].name 需要对应    
      mountPath: /data #挂载到容器的某个路径下  
      readOnly: True  
  volumes: #定义一组挂载设备  
  - name: volume #定义一个挂载设备的名字  
    #meptyDir: {}  
    hostPath:  
      path: /opt #挂载设备类型为hostPath，路径为宿主机下的/opt,这里设备类型支持很多种 

# 原文链接：https://blog.csdn.net/liyingke112/article/details/76155428
```

### Pod管理-创建/查询/更新/删除 <a href="#blogtitle0" id="blogtitle0"></a>

```bash
# 基本管理：
# 创建pod资源
kubectl create -f pod.yaml
# 查看pods
kubectl get pods pod-test
# 查看pod描述
kubectl describe pod pod-test
# 替换资源
kubectl replace -f pod.yaml -force
# 删除资源
kubectl delete pod pod-test
# 健康管理
# 提供Probe机制，有以下两种类型：livenessProbe如果检查失败，将杀死容器，然后根据Pod的重启策略来决定是否重启。readinessProbe
# 如果检查失败，Kubernetes会把Pod从服务代理的分发后端剔除。
# Probe支持以下三种检查方法：httpGet发送HTTP请求，返回200-400范围状态码为成功。exec执行Shell命令返回状态码是0为成功。tcpSocket发起TCP Socket建立成功。
```

## Deployment

简单例子

```yaml
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.10
        ports:
        - containerPort: 80
```

详细解析

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata: <Object>
spec: <Object>
  minReadySeconds: <integer> #设置pod准备就绪的最小秒数
  paused: <boolean> #表示部署已暂停并且deploy控制器不会处理该部署
  progressDeadlineSeconds: <integer>
  strategy: <Object> #将现有pod替换为新pod的部署策略
    rollingUpdate: <Object> #滚动更新配置参数，仅当类型为RollingUpdate
      maxSurge: <string> #滚动更新过程产生的最大pod数量，可以是个数，也可以是百分比
      maxUnavailable: <string> #
    type: <string> #部署类型，Recreate，RollingUpdate
  replicas: <integer> #pods的副本数量
  selector: <Object> #pod标签选择器，匹配pod标签，默认使用pods的标签
    matchLabels: <map[string]string> 
      key1: value1
      key2: value2
    matchExpressions: <[]Object>
      operator: <string> -required- #设定标签键与一组值的关系，In, NotIn, Exists and DoesNotExist
      key: <string> -required-
      values: <[]string>   
  revisionHistoryLimit: <integer> #设置保留的历史版本个数，默认是10
  rollbackTo: <Object> 
    revision: <integer> #设置回滚的版本，设置为0则回滚到上一个版本
  template: <Object> -required-
    metadata:
    spec:
      containers: <[]Object> #容器配置
      - name: <string> -required- #容器名、DNS_LABEL
        image: <string> #镜像
        imagePullPolicy: <string> #镜像拉取策略，Always、Never、IfNotPresent
        ports: <[]Object>
        - name: #定义端口名
          containerPort: #容器暴露的端口
          protocol: TCP #或UDP
        volumeMounts: <[]Object>
        - name: <string> -required- #设置卷名称
          mountPath: <string> -required- #设置需要挂载容器内的路径
          readOnly: <boolean> #设置是否只读
        livenessProbe: <Object> #就绪探测
          exec: 
            command: <[]string>
          httpGet:
            port: <string> -required-
            path: <string>
            host: <string>
            httpHeaders: <[]Object>
              name: <string> -required-
              value: <string> -required-
            scheme: <string> 
          initialDelaySeconds: <integer> #设置多少秒后开始探测
          failureThreshold: <integer> #设置连续探测多少次失败后，标记为失败，默认三次
          successThreshold: <integer> #设置失败后探测的最小连续成功次数，默认为1
          timeoutSeconds: <integer> #设置探测超时的秒数，默认1s
          periodSeconds: <integer> #设置执行探测的频率（以秒为单位），默认1s
          tcpSocket: <Object> #TCPSocket指定涉及TCP端口的操作
            port: <string> -required- #容器暴露的端口
            host: <string> #默认pod的IP
        readinessProbe: <Object> #同livenessProbe
        resources: <Object> #资源配置
          requests: <map[string]string> #最小资源配置
            memory: "1024Mi"
            cpu: "500m" #500m代表0.5CPU
          limits: <map[string]string> #最大资源配置
            memory:
            cpu:         
      volumes: <[]Object> #数据卷配置
      - name: <string> -required- #设置卷名称,与volumeMounts名称对应
        hostPath: <Object> #设置挂载宿主机路径
          path: <string> -required- 
          type: <string> #类型：DirectoryOrCreate、Directory、FileOrCreate、File、Socket、CharDevice、BlockDevice
      - name: nfs
        nfs: <Object> #设置NFS服务器
          server: <string> -required- #设置NFS服务器地址
          path: <string> -required- #设置NFS服务器路径
          readOnly: <boolean> #设置是否只读
      - name: configmap
        configMap: 
          name: <string> #configmap名称
          defaultMode: <integer> #权限设置0~0777，默认0664
          optional: <boolean> #指定是否必须定义configmap或其keys
          items: <[]Object>
          - key: <string> -required-
            path: <string> -required-
            mode: <integer>
      restartPolicy: <string> #重启策略，Always、OnFailure、Never
      nodeName: <string>
      nodeSelector: <map[string]string>
      imagePullSecrets: <[]Object>
      hostname: <string>
      hostPID: <boolean>
status: <Object>
# 原文链接：https://blog.csdn.net/weixin_44006354/article/details/100634729
```

## Service

```
服务类型：
ClusterIP
分配一个内部集群IP地址，只能在集群内部访问（同Namespace内的Pod），默认ServiceType。
 NodePort
分配一个内部集群IP地址，并在每个节点上启用一个端口来暴露服务，可以在集群外部访问。
访问地址：<NodeIP>:<NodePort>
LoadBalancer
分配一个内部集群IP地址，并在每个节点上启用一个端口来暴露服务。
除此之外，Kubernetes会请求底层云平台上的负载均衡器，将每个Node（[NodeIP]:[NodePort]）作为后端添加进去。
 ExternalName
通过CNAME将Service与externalName的值映射。要求kube-dns的版本为v1.7+。
```

实例1

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    run: nginx
  name: nginx
  namespace: default
spec:
  ports:
  - port: 88
    targetPort: 80
  selector:
    app: nginx
```

实例2：指定ip

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: nginx
  ports:
  - name: http
    protocol: TCP
    port: 888
    targetPort: 80
#  可以指定ip
  clusterIP: "10.10.10.123"
#  - name: https
#    protocol: TCP
#    port: 443
#    targetPort: 9377
```

实例3：发布服务

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  labels:
    app: nginx
spec:
  selector:
    app: nginx
  ports:
  - name: http
    port: 8080
    targetPort: 80
    nodePort: 30001
  type: NodePort
```

**链接：**

[**https://www.cnblogs.com/fuyuteng**](https://www.cnblogs.com/fuyuteng/p/9460534.html)

&#x20;

&#x20;**kubernetes核心组件kube-proxy：**

[https://www.cnblogs.com/fuyuteng/p/11598768.html](https://www.cnblogs.com/fuyuteng/p/11598768.html)
