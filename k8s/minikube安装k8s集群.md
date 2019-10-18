# minikube 安装教程

## 安装minikube
```sh
mkdir minikube
cd minikube
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 \
>   && chmod +x minikube
sudo mkdir -p /usr/local/bin/
sudo install minikube /usr/local/bin/
```

## 使用minikube部署集群

### 使用国内镜像启动
```sh
minikube start --vm-driver=none --registry-mirror=https://registry.docker-cn.com
```

