# Docker + Jenkins 安装部署教程 #

>Docker 的安装教程：https://blog.muyuchenge.com/2019/04/03/centos-docker-安装教程

## 无脑运行一波 ##

```cmd
docker pull jenkins/jenkins
mkdir /var/jenkins_home
chmod a+rwx /home/user/
chmod a+rw /var/run/docker.sock 
docker run -itd --restart=always --name=jenkins \
-p 8888:8080 -p 50000:50000 \
-v /var/jenkins_home:/var/jenkins_home  jenkins/jenkins
 
```
## 必要操作 ##
>因为里面使用了google的网站导致安装插件出现问题，有的还出现Jenkins 离线的问题，所以要进行配置文件的修改

**Step 1:**
```cmd
vim /var/lib/jenkins_home/updates/default.json
```
将文件开始的“www.google.com”改为“www.baidu.com”

**Step 2:**
```cmd
vim /var/lib/jenkins_home/hudson.model.UpdateCenter.xml
```
将“https://updates.jenkins.io/update-center.json”改为“http://updates.jenkins.io/update-center.json”

**Step 3:**
```cmd
docker restart jenkins
```
## 后续操作 ##

浏览器打开 "http://ip:8888" 选择安装推荐的插件，并等待安装完毕。 安装完毕后进入第一个管理员设置界面，我一般是直接使用admin进行下一步。

如果打开首页空白，先进入“http://ip:8888/configureSecurity”，将授权策略里的“匿名用户具有可读权限”复选框取消，然后保存。进入http://ip:8888/restart 进行重启。再打开就正常了，然后开始你的表演。

>如果需要更改时区（默认的是UTC），在Jenkins页面进入“系统管理”-“脚本命令行”执行：
>`System.setProperty('org.apache.commons.jelly.tags.fmt.timeZone', 'Asia/Shanghai')`