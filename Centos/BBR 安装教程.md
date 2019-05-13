# BBR 安装教程 #
>声明：非常感谢 **秋水逸冰** 提供的技术共享，网址：https://teddysun.com/489.html

>备注：大神的官网在海外，不FQ的话可能无法登陆。

## Step 1. 一键安装BBR

```cmd
yum update -y
wget --no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh && chmod +x bbr.sh && ./bbr.sh

```
>如果没有wget，安装即可：`yum install wget -y`

>等待完成即可，中间可能会提示你回车，回车即可。

> 使用此脚本会把系统内核更新为最新的，所以我习惯把其他的东东也都更新下，请看step2

## Step 2. 更新其他插件 ##
```cmd
yum remove kernel-headers -y
yum -y remove kernel kernel-tools
yum remove kernel-tools-libs.x86_64 kernel-tools.x86_64 -y
yum --disablerepo=\* --enablerepo=elrepo-kernel install -y kernel-ml-tools.x86_64
yum --enablerepo=elrepo-kernel -y install kernel-ml-headers -y
yum update -y
 
```

>BBR对TCP的处理确实是最好的，无可厚非

