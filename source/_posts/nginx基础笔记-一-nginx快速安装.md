---
title: nginx快速安装
date: 2017-11-09 16:34:13
tags: [技术分享,nginx]
---
#### 一、介绍
> 写此nginx从入门到实践系列笔记,为以教促学,让自己系统地学习掌握nginx的配置和搭建高可用架构。
系统环境:centos7.2

#### 二、nginx快速安装
>访问nginx官网:nginx.org 点击download后选择最底下 stable version(稳定版本)的链接找到这里:
![](http://upload-images.jianshu.io/upload_images/2660278-0174a81150a15f5c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
在系统新建nginx.repo并将此官网的yum源拷贝进去
```python
vim /etc/yum.repos.d/nginx.repo
```
>接着修改OS和OSRELEASE，我的环境为centos7.2系统故我修改如下
```python
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/mainline/centos/7/$basearch/
gpgcheck=0
enabled=1
```
>保存后测试
```python
yum list |grep nginx
```
>![](http://upload-images.jianshu.io/upload_images/2660278-38a62ef92a9decc1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
>出现该列表则为成功添加
接下来便可直接使用
```python
yum install nginx
```
>来安装nginx
安装完毕后测试:
```python
nginx -v
```
>打印出nginx版本则为安装成功

#### 三、启动nginx
>安装完毕后我们来启动nginx:
```python
sudo systemctl start nginx.service
```
>此时便可通过域名或者ip访问web页面来预览nginx的默认页面:
>![](http://upload-images.jianshu.io/upload_images/2660278-01ea50292c2fc8c2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
设置开机启动nginx:
```python
sudo systemctl enable nginx.service
```

到此便是安装完成
