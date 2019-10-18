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
docker pull mirrorgooglecontainers/kube-addon-manager-amd64:v9.0.2
docker pull dieudonnecc/storage-provisioner:v1.8.1

docker tag mirrorgooglecontainers/kube-apiserver-amd64:v1.16.2 k8s.gcr.io/kube-apiserver:v1.16.0
docker tag mirrorgooglecontainers/kube-controller-manager-amd64:v1.16.1 k8s.gcr.io/kube-controller-manager:v1.16.0
docker tag mirrorgooglecontainers/kube-scheduler-amd64:v1.16.2 k8s.gcr.io/kube-scheduler:v1.16.0
docker tag mirrorgooglecontainers/kube-proxy-amd64:v1.16.2 k8s.gcr.io/kube-proxy:v1.16.0
docker tag mirrorgooglecontainers/pause-amd64:3.1 k8s.gcr.io/pause:3.1
docker tag mirrorgooglecontainers/etcd:3.3.15-0 k8s.gcr.io/etcd:3.3.15-0
docker tag coredns/coredns:1.6.2 k8s.gcr.io/coredns:1.6.2
docker tag mirrorgooglecontainers/kube-addon-manager-amd64:v9.0.2 k8s.gcr.io/kube-addon-manager:v9.0.2
docker tag dieudonnecc/storage-provisioner:v1.8.1 gcr.io/k8s-minikube/storage-provisioner:v1.8.1
```

## 启动
```sh
minikube start --vm-driver=none
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
## 查看节点
```sh
kubectl get nodes
```

## 安装kuboard
```sh
kubectl apply -f https://kuboard.cn/install-script/kuboard.yaml
#打开
http://{ip}:32567
#获取token
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep kuboard-user | awk '{print $1}')
```
