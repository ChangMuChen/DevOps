# RabbitMQ 安装教程
>Docker 环境，安装教程：https://blog.muyuchenge.com/2019/04/03/centos-docker-安装教程/

## 获取镜像
```
docker pull rabbitmq
```

## 启动容器
```
 docker run -itd --hostname np-rabbit --name np-rabbitmq-01 -e RABBITMQ_DEFAULT_USER=admin -e  \
RABBITMQ_DEFAULT_PASS=NP123456 -p 5671:5671 -p 15672:15672 rabbitmq
```
## 启动插件
### 进入容器
```
docker exec -it np-rabbitmq-01 /bin/bash
```
### 启动插件
```
rabbitmq-plugins enable --offline rabbitmq_mqtt rabbitmq_federation_management rabbitmq_stomp
```
### 退出容器
```
exit
```
### 重启容器
```
docker restart np-rabbitmq-01
```
## 进入web管理界面
```
http://ip:15672
```
>因为重启的原因，可能要等几秒中才能加载web