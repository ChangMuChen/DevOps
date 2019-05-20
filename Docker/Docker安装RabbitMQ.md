# RabbitMQ 安装教程
>Docker 环境，安装教程：https://github.com/ChangMuChen/DevOps/blob/master/Docker/Centos%20%E5%AE%89%E8%A3%85Docker.md

## 获取镜像
```
docker pull rabbitmq
```

## 启动容器
```
docker run -itd --hostname np-rabbit --name np-rabbitmq-01 -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=NP123456 -p 5671:5671 -p 5672:5672 -p 4369:4369 -p 25672:25672 -p 15672:15672 rabbitmq
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
