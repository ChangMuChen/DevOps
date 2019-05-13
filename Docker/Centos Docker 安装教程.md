# Centos Docker 安装教程 #

>我认为官方的说明一件很详细了，我这里做个笔记，方便以后直接查阅。

>官方网站：https://docs.docker.com/install/linux/docker-ce/centos/

## 一键安装脚本 ##
```cmd
yum update -y
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
systemctl enable docker
systemctl start docker
systemctl status docker
 
```
>中间过程有点漫长请耐心等待

##  镜像加速器 ##

>我个人感觉还好啦，但是考虑到有的同学的网络环境，建议使用阿里云加速器

```cmd

sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://uvioyo5q.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker

```

>上面用的是我个人在阿里申请的加速器，大家可以自己去注册账号并申请下。

>另外我个人喜欢把系统升级到最新，并加上BBR，教程请浏览：https://blog.muyuchenge.com/2019/04/03/bbr-安装教程

>担心新同学找不到位置，贴个图给大家看下：
>![镜像加速器](https://blog.muyuchenge.com/wp-content/uploads/2019/04/TIM截图20190403145749.png)

