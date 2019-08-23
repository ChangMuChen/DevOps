# K8S安装与部署教程
## 节点规划
- master节点
- node1节点
- node2节点
- node3节点
## 安装运行时环境
>三选一，推荐docker
### docker安装与设置
>所有节点运行
```sh
# Install Docker CE
## Set up the repository
### Install required packages.
yum install yum-utils device-mapper-persistent-data lvm2 -y

### Add Docker repository.
yum-config-manager \
  --add-repo \
  https://download.docker.com/linux/centos/docker-ce.repo

## Install Docker CE.
yum update -y && yum install docker-ce-18.06.2.ce -y

## Create /etc/docker directory.
mkdir /etc/docker

# Setup daemon.
cat > /etc/docker/daemon.json <<EOF
{
  "registry-mirrors": ["https://uvioyo5q.mirror.aliyuncs.com"],
  "exec-opts": ["native.cgroupdriver=systemd"],
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

# Restart Docker
systemctl daemon-reload
systemctl restart docker
systemctl enable docker
```
### CRI-O
>所有节点

#### 先决条件
```sh
modprobe overlay
modprobe br_netfilter

# Setup required sysctl params, these persist across reboots.
cat > /etc/sysctl.d/99-kubernetes-cri.conf <<EOF
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

sysctl --system
```
#### 安装与启动
```sh
# Install prerequisites
yum-config-manager --add-repo=https://cbs.centos.org/repos/paas7-crio-311-candidate/x86_64/os/

# Install CRI-O
yum install -y runc
yum install --nogpgcheck cri-o -y
systemctl start crio
```
### containerd
>所有节点
#### 先决条件
```sh
cat > /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF
modprobe overlay
modprobe br_netfilter

# Setup required sysctl params, these persist across reboots.
cat > /etc/sysctl.d/99-kubernetes-cri.conf <<EOF
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

sysctl --system
```

#### 安装与启动
```sh
# Install containerd
## Set up the repository
### Install required packages
yum install yum-utils device-mapper-persistent-data lvm2 -y

### Add docker repository
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

## Install containerd
yum update -y && yum install containerd.io -y

# Configure containerd
mkdir -p /etc/containerd
containerd config default > /etc/containerd/config.toml

# Restart containerd
systemctl restart containerd
systemctl enable containerd
```
#### 设置cgroup driver
```sh
# 设置config.toml
vim /etc/containerd/config.toml
#修改内容如下
plugins.cri.systemd_cgroup = true
```

## 系统环境设置

### 关闭防火墙
>所有节点
```sh
systemctl stop firewalld
systemctl disable firewalld
```

### host修改
>所有节点运行
cat << EOM > /etc/hosts
192.168.40.128 node1
192.168.40.129 node2
192.168.40.130 node3
192.168.40.131 admin
EOM

### 关闭swap
>所有节点运行
```sh
vi /etc/fstab 
#注释掉swap那一行
sysctl -p
swapoff -a
```

## 安装kubelet kubeadm kubectl
>所有节点
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

# Set SELinux in permissive mode (effectively disabling it)
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system

yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
systemctl enable --now kubelet
systemctl restart --now kubelet

```

<!-- ## 修改配置
>所有节点
```sh
mkdir -p /etc/cni/net.d/
cat << EOF > /etc/cni/net.d/10-flannel.conf
{"name":"cbr0","type":"flannel","delegate":{"isDefaultGateway":"true"}}
EOF
``` -->

## 初始化master
>master节点运行
```sh
kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=192.168.40.131
```

## 命令可行
将/etc/kubernetes/admin.conf拷贝到其他节点相同位置

>所有节点
```sh
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## 加入集群
```sh
#以下命令会在初始化master完成后出现
kubeadm join 192.168.40.131:6443 --token fcvcgl.b9d5qewsvalomg3t \
    --discovery-token-ca-cert-hash sha256:920cc8c1222ebfff37b7b0a49779b8f79f8040a6c75c3dc7c8b1aec00875d511
```
## 安装网络模块
>所有节点
```sh
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```


## 其他
```sh
#查看日志
journalctl -f -u kubelet

#获取节点
kubectl get nodes

#删除节点
kubectl delete node node1


#撤销部署
kubeadm reset
```


## 扩展1：安装看板
>master节点
### 安装
```sh
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta1/aio/deploy/recommended.yaml
```

### 更改接口访问方式
```sh
kubectl get pods -n kube-system
kubectl get svc -n kube-system
#以打补丁方式修改dasboard的访问方式
kubectl patch svc kubernetes-dashboard -p '{"spec":{"type":"NodePort"}}' -n kube-system  
kubectl get svc -n kube-system
#访问https://host:port
```

### 产生Token
```sh
kubectl create serviceaccount cluster-admin-dashboard-sa
kubectl create clusterrolebinding cluster-admin-dashboard-sa \
  --clusterrole=cluster-admin \
  --serviceaccount=default:cluster-admin-dashboard-sa

#查看用户
kubectl get secret | grep cluster-admin-dashboard-sa
#获取用户Token
kubectl describe secrets/cluster-admin-dashboard-sa-token-6thzn
```

### 使其支持谷歌浏览器
```sh
mkdir key && cd key
#生成证书
openssl genrsa -out dashboard.key 2048 
#填写所在主机IP
openssl req -new -out dashboard.csr -key dashboard.key -subj '/CN=192.168.246.200'
openssl x509 -req -in dashboard.csr -signkey dashboard.key -out dashboard.crt 
#删除原有的证书secret
kubectl delete secret kubernetes-dashboard-certs -n kube-system
#创建新的证书secret
kubectl create secret generic kubernetes-dashboard-certs --from-file=dashboard.key --from-file=dashboard.crt -n kube-system
#查看pod
kubectl get pod -n kube-system
#重启pod
kubectl delete pod <pod name> -n kube-system
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta1/aio/deploy/recommended.yaml
```
