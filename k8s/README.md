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
  "insecure-registries":["http://172.16.50.96:6543"],
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

## 系统环境设置

### 关闭防火墙
>所有节点
```sh
systemctl stop firewalld
systemctl disable firewalld
```

### host修改
>所有节点运行

```shell
cat << EOM > /etc/hosts
192.168.40.128 node1
192.168.40.129 node2
192.168.40.130 node3
192.168.40.131 admin
EOM
```

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
kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=192.168.40.131 --image-repository registry.aliyuncs.com/google_containers
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

#创建key并显示加入命令
kubeadm token create --print-join-command

#排除master污点
kubectl taint nodes --all node-role.kubernetes.io/master-

#撤销部署
kubeadm reset

#开启代理
kubectl proxy --address='0.0.0.0'  --accept-hosts='^*$'

#私有仓库
kubectl -n fortest create secret docker-registry devsecret --docker-server=172.16.50.96:6543 --docker-username=docker-dev --docker-password=NP123456  --docker-email=changdaohang@anpe.cn
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
kubectl create clusterrolebinding etcd-certs \
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

### NFS实现卷存储


### 备忘
```shell
kubeadm config images list  #查看需要的镜像
docker pull mirrorgooglecontainers/kube-apiserver-amd64:v1.16.0
docker pull mirrorgooglecontainers/kube-controller-manager-amd64:v1.16.0
docker pull mirrorgooglecontainers/kube-scheduler-amd64:v1.16.0
docker pull mirrorgooglecontainers/kube-proxy-amd64:v1.16.0
docker pull mirrorgooglecontainers/pause-amd64:3.1
docker pull mirrorgooglecontainers/etcd:3.3.15-0
docker pull coredns/coredns:1.6.2

docker tag mirrorgooglecontainers/kube-apiserver-amd64:v1.16.0 k8s.gcr.io/kube-apiserver:v1.16.0
docker tag mirrorgooglecontainers/kube-controller-manager-amd64:v1.16.0 k8s.gcr.io/kube-controller-manager:v1.16.0
docker tag mirrorgooglecontainers/kube-scheduler-amd64:v1.16.0 k8s.gcr.io/kube-scheduler:v1.16.0
docker tag mirrorgooglecontainers/kube-proxy-amd64:v1.16.0 k8s.gcr.io/kube-proxy:v1.16.0
docker tag mirrorgooglecontainers/pause-amd64:3.1 k8s.gcr.io/pause:3.1
docker tag mirrorgooglecontainers/etcd:3.3.15-0 k8s.gcr.io/etcd:3.3.15-0
docker tag coredns/coredns:1.6.2 k8s.gcr.io/coredns:1.6.2

docker pull vbouchaud/nfs-client-provisioner-arm64
docker tag vbouchaud/nfs-client-provisioner-arm64 quay.io/external_storage/nfs-client-provisioner:v3.1.0-k8s1.11

docker pull vbouchaud/nfs-client-provisioner
docker tag vbouchaud/nfs-client-provisioner quay.io/external_storage/nfs-client-provisioner:v3.1.0-k8s1.11
```

