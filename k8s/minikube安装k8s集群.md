# minikube 安装教程
## 准备工作
```sh
#关闭防火墙
systemctl stop firewalld
systemctl disable firewalld

vi /etc/fstab 
#注释掉swap那一行
sysctl -p
swapoff -a
```

## 安装docker
```sh
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
sudo yum makecache fast
sudo yum -y install docker-ce
sudo systemctl start docker
cat > /etc/docker/daemon.json <<EOF
{
  "registry-mirrors": ["https://uvioyo5q.mirror.aliyuncs.com"],
  "exec-opts": ["native.cgroupdriver=cgroupfs"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}
EOF
mkdir -p /etc/systemd/system/docker.service.d
systemctl daemon-reload
systemctl restart docker
systemctl enable docker
```

## 安装minikube
```sh
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && chmod +x minikube
sudo mkdir -p /usr/local/bin/
sudo install minikube /usr/local/bin/
```

## 拉取镜像并改名
```sh
docker pull mirrorgooglecontainers/kube-apiserver-amd64:v1.16.2
docker pull mirrorgooglecontainers/kube-controller-manager-amd64:v1.16.1
docker pull mirrorgooglecontainers/kube-scheduler-amd64:v1.16.2
docker pull mirrorgooglecontainers/kube-proxy-amd64:v1.16.2
docker pull mirrorgooglecontainers/pause-amd64:3.1
docker pull mirrorgooglecontainers/etcd:3.3.15-0
docker pull coredns/coredns:1.6.2

docker tag mirrorgooglecontainers/kube-apiserver-amd64:v1.16.2 k8s.gcr.io/kube-apiserver:v1.16.0
docker tag mirrorgooglecontainers/kube-controller-manager-amd64:v1.16.1 k8s.gcr.io/kube-controller-manager:v1.16.0
docker tag mirrorgooglecontainers/kube-scheduler-amd64:v1.16.2 k8s.gcr.io/kube-scheduler:v1.16.0
docker tag mirrorgooglecontainers/kube-proxy-amd64:v1.16.2 k8s.gcr.io/kube-proxy:v1.16.0
docker tag mirrorgooglecontainers/pause-amd64:3.1 k8s.gcr.io/pause:3.1
docker tag mirrorgooglecontainers/etcd:3.3.15-0 k8s.gcr.io/etcd:3.3.15-0
docker tag coredns/coredns:1.6.2 k8s.gcr.io/coredns:1.6.2
```

## 启动
```sh
minikube start --vm-driver=none
```
