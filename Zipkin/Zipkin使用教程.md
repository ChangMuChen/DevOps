# Zipkin按照教程 #
## 知识准备
**Zipkin：**[https://zipkin.apache.org/](https://zipkin.apache.org/)
**Sleuth：**[https://cloud.spring.io/spring-cloud-sleuth/spring-cloud-sleuth.html](https://cloud.spring.io/spring-cloud-sleuth/spring-cloud-sleuth.html)
## 搭建Zipkin服务
>这里采用Docker的方式进行搭建
### Step1 下载镜像
```shell
docker pull openzipkin/zipkin
```
### Step2 启动Zipkin相关容器
>这里使用`docker-compose`的方式进行启动
**创建目录**
```shell
sudo mkdir ~/zipkin
sudo cd ~/zipkin
sudo vi docker-compose.yml
```
**写入以下内容**
```yml
version: '3.7'
services:
  zipkin:
    image: openzipkin/zipkin
    container_name: zipkin
    environment:
      - STORAGE_TYPE=mysql
      - MYSQL_DB=zipkindb
      - MYSQL_USER=root
      - MYSQL_PASS=123456
      - MYSQL_HOST=192.168.1.119
      - MYSQL_TCP_PORT=3306
      - JAVA_OPTS=-Dzipkin.collector.rabbitmq.addresses=192.168.1.119:5672 -Dzipkin.collector.rabbitmq.username=admin -Dzipkin.collector.rabbitmq.password=NP123456 -Dzipkin.collector.rabbitmq.queue=zipkin
    ports:
      - 9411:9411

```
>这里用到了rabbitMQ以及Mysql，请自行搭建。

**运行容器**
```shell
docker-compose up -d
```

**打开页面**
[http://192.168.1.119:9411](http://192.168.1.119:9411)
![](https://i.imgur.com/NRBSWxf.jpg)
