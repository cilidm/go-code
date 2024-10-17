# kubectl 安装

## 安装方法

### OSX

```bash
brew install kubectl
# 或者
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/darwin/amd64/kubectl
```

### Linux

```bash
wget https://storage.googleapis.com/kubernetes-release/release/v1.22.3/bin/linux/amd64/kubectl
chmod +x kubectl
```

### Windows

```bash
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/windows/amd64/kubectl.exe
# 或者
choco install kubernetes-cli
```
