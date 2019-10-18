# minikube 安装教程
## 准备工作
```sh
#关闭防火墙
systemctl stop firewalld
systemctl disable firewalld

#关闭swap
vi /etc/fstab 
#注释掉swap那一行
sysctl -p
swapoff -a
```
## 安装kubectl和kubeadm
```sh
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF

//国内用这个
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

# Set SELinux in permissive mode (effectively disabling it)
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

cat <<EOF >  /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system

yum install -y kubeadm kubectl --disableexcludes=kubernetes
```

## 安装minikube
```sh
mkdir minikube
cd minikube
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && chmod +x minikube
sudo mkdir -p /usr/local/bin/
sudo install minikube /usr/local/bin/
```

## 使用minikube部署集群
## 方式1
### 查看所需镜像
```sh
kubeadm config images list
```

### 拉取镜像并改名
```sh
docker pull mirrorgooglecontainers/kube-apiserver-amd64:v1.16.2
docker pull mirrorgooglecontainers/kube-controller-manager-amd64:v1.16.1
docker pull mirrorgooglecontainers/kube-scheduler-amd64:v1.16.2
docker pull mirrorgooglecontainers/kube-proxy-amd64:v1.16.2
docker pull mirrorgooglecontainers/pause-amd64:3.1
docker pull mirrorgooglecontainers/etcd:3.3.15-0
docker pull coredns/coredns:1.6.2

docker tag mirrorgooglecontainers/kube-apiserver-amd64:v1.16.2 k8s.gcr.io/kube-apiserver:v1.16.2
docker tag mirrorgooglecontainers/kube-controller-manager-amd64:v1.16.1 k8s.gcr.io/kube-controller-manager:v1.16.2
docker tag mirrorgooglecontainers/kube-scheduler-amd64:v1.16.2 k8s.gcr.io/kube-scheduler:v1.16.2
docker tag mirrorgooglecontainers/kube-proxy-amd64:v1.16.2 k8s.gcr.io/kube-proxy:v1.16.2
docker tag mirrorgooglecontainers/pause-amd64:3.1 k8s.gcr.io/pause:3.1
docker tag mirrorgooglecontainers/etcd:3.3.15-0 k8s.gcr.io/etcd:3.3.15-0
docker tag coredns/coredns:1.6.2 k8s.gcr.io/coredns:1.6.2
```

### 使用启动
```sh
minikube start --vm-driver=none --registry-mirror=https://registry.docker-cn.com
```
## 方式2
### 如果不用改镜像的方式，可以用阿里的镜像库这样启动
```sh
minikube start --vm-driver=none --registry-mirror=https://registry.docker-cn.com --image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers
```
