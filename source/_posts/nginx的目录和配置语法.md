---
title: nginx的目录和配置语法
date: 2017-11-10 13:30:34
tags: [技术分享,nginx]
---
#### 一、nginx的各种安装目录
| 路径        | 类型   |  作用  |
| --------   | -----:  | :----:  |
| /etc/logrotate.d/nginx     |  配置文件 |   nginx日志轮转用于logrotate服务的日志切割    |
| /etc/nginx/nginx.conf<br/>/etc/nginx/conf.d/default.conf         |   配置文件   |   nginx主配置文件   |
| /etc/nginx/fastcgi_params<br/>/etc/nginx/uwsgi_params<br/>/etc/nginx/scgi_params        |  配置文件  |  cgi配置相关<br/>fastcgi配置  |
|/etc/nginx/koi-utf<br/>/etc/nginx/win-utf<br/>/etc/nginx/koi-win|  配置文件  |  编码转换映射转化文件  |
|  /etc/nginx/mime.types   |  配置文件  |  设置http协议的Content-Type与扩展名对应关系  <br/>处理一些识别不了的扩展名的时候需要用到  |
|  /usr/lib/systemd/system/nginx.service<br/>/usr/lib/systemd/system/nginx-debug.service<br/>/etc/sysconfig/nginx<br/>/etc/sysconfig/nginx-debug  |  配置文件  |   用于配置出系统守护进程管理器管理方式 |
|  /usr/sbin/nginx<br/>/usr/sbin/nginx-debug |   命令 |   nginx服务终端命令 |
|   /usr/share/doc/nginx-1.12.2/COPYRIGHT<br/>/usr/share/man/man8/nginx.8.gz  |  文件、目录  |  nginx的手册和帮助文件  |
|  /var/cache/nginx/  |  目录  |  nginx的缓存目录  |
|  /var/log/nginx/  |  目录  |  nginx的日志目录  |
#### 二、nginx的编译配置参数
命令`nginx -V`输出的就是nginx编译时候用到的参数

|  编译选项  |  作用  |
| --------   | :----:  |
|  --prefix=/etc/nginx<br/>--sbin-path=/usr/sbin/nginx<br/>--modules-path=/usr/lib64/nginx/modules<br/>--conf-path=/etc/nginx/nginx.conf<br/>--error-log-path=/var/log/nginx/error.log<br/>--http-log-path=/var/log/nginx/access.log<br/>--pid-path=/var/run/nginx/pid<br/>--lock-path=/var/run/nginx.lock  |  安装目的目录或路径  |
|  --httpd-client-body-temp-path=/var/cache/nginx/client_temp<br/>--httpd-proxy-temp-path=/var/cache/nginx/proxy_temp<br/>--http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp<br/>--http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp<br/>--http-scgi-temp-path=/var/chache/nginx/scgi_temp  |  执行对应模块时，Nginx所保留的临时性文件  |
|  --user=nginx<br/>--group=nginx  |  设定Nginx进程启动的用户和组用户  |
|  --with-cc-opt=parameters  |  设置额外的参数将被添加到CFLAGS变量  |
|  --with-ld-opt=parameters  |  设置附加的参数，连接系统库  |

#### 三、nginx.conf基础配置语法
>nginx.conf基础配置语法
nginx.conf 作为主配置文件
include /etc/nginx/conf.d/*.conf 读到这会把该目录的.conf也读进来
>* **全局性的和服务级别的**
`user`设置使用用户  
`worker_processes`进行增大并发连接数的处理(最好设置跟cpu保持一致)
`error_log`nginx的错误日志
`pid`nginx服务启动时候pid 
>* event对事件的模块
`worker_connections`一个进程允许处理的最大连接数
`use`定义使用的内核模型

```
http{
  server{
  listen 80;  //端口
  server_name localhost; //域名
      location /  //子目录和当前目录
      {
                  root   /usr/share/nginx/html;      //指定目录
                  index   index.html     index.htm   //默认访问页面
      }
      error_page    500 502 503 504 /50.html
      location = /50x.html{
        root /usr/share/nginx/html;
    }
  }
  server{
  ... ...
  }
}
```
>修改配置文件后需要重启nginx<br/>
重启nginx(进程停掉,重新启动一次)
`systemctl restart nginx.service`
不关闭服务柔和地重启(重新读取一次配置文件)
`systemctl reload nginx.service`
