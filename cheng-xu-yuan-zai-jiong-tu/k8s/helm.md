# helm

### 安装

{% embed url="https://github.com/helm/helm" %}

### 添加阿里仓库

```bash
 helm repo add apphub https://apphub.aliyuncs.com/
 
 helm repo update
```

### 搜索

```bash
helm search repo stable/mysql

helm install  mydb  stable/mysql

```

### 创建项目

```text
helm create mygin 

# 渲染
helm install my  mygin --dry-run --debug 

# 渲染+安装
helm install my  mygin -n myweb
(helm install [NAME] [CHART] [flags])

# 测试
helm install my mygin --dry-run --debug 
```



