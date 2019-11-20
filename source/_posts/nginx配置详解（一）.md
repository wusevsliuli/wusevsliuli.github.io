---
title: nginx配置详解（一）
date: 2019-06-28 14:33:37
categories:
 - 服务
tags:
 - nginx
---
Nginx是一个功能丰富的Web服务器。它支持反向代理服务，内置了负载均衡和Web缓存功能。
<escape><!-- more --></escape>

#### Nginx配置结构
nginx配置如图所示

![nginx配置结构.png](nginx配置详解（一）/nginx配置结构.png)

其中server块和location块可以有多个  
nginx配置中可以通过如下写法定义变量
> set $变量名 变量值 

然后在配置中使用`$变量名`即可替代变量值  
后续配置说明中以 **$** 标记的都将表示变量，以下为使用的变量的含义，不再赘述
* $path: 表文件/文件夹路径  
* $num: 表数字  
* $user: 表用户名
* $group: 表用户组名

后续配置中 **|** 符号表示枚举

#### 1. nginx全局块
配置示例  
```bash
user $user $group;
worker_processes $num;
pid $path;
error_log $path $level;
include $path;
```
说明如下:
* user  
    Nginx将以此账户来运行  
    $user表示用户名，默认为nobody  
    $group表示用户组，默认为nobody 
* worker_processes  
    允许生成的nginx进程数，默认为1，一般不用修改，如果要修改提高性能官方的建议是修改成CPU的内核数，随意设置可能会导致问题，参见[nginx worker_processes 配置](https://www.cnblogs.com/aaron-agu/p/8003831.html)  
* pid  
    nginx在运行时会生成一个记录运行进程pid的文件  
    $path表示文件路径，文件名一般为nginx.pid  
    $level表示日志等级，取值有 `debug`|`info`|`notice`|`warn`|`error`|`crit`  
* error_log  
    nginx的产生的日志文件路径，也可以放在http块，server块中作为http或server的日志，后续不再说明  
    $path表示文件路径，默认一般为 nginx安装路径/logs/error.log   
* include  
    指定拓展的nginx配置文件（仅在主配置文件中），为简化nginx主配置文件，实现多个站点功能
    $path表示文件路径，拓展的nginx配置文件，拓展的配置文件只包含server块，如下
```bash
    server {
        ......
    }
```
#### 2. events块
配置示例
```bash
event {
    accept_mutex on|off;
    multi_accept on|off;
    use $model;
    worker_connections $num;
}
```
说明如下  
* accept_mutex    
    使用串行的方式来处理到达的新连接，防止[惊群现象](https://www.jianshu.com/p/1cdd61a6e3ea)发生，默认为on  
**PS**: 对nginx来说，worker_processes一般会设置成CPU个数，即便发生惊群现象影响也较小（相对于Apache几百个进程来说），主要表现为上下文切换增多或负载上升，如果**网站访问量较大**，为了系统吞吐量可以设为off  
* multi_accept  
    设置一个进程是否同时接受多个网络连接，默认为off  
* use  
nginx的事件驱动模型  
$model取值有 `select`|`poll`|`kqueue`|`epoll`|`resig`|`/dev/poll`|`eventport`  
**PS**: 一般不更改，nginx会自动选择一个最适合运行nginx操作系统的。  
**PS**: 如果选择的是kqueue，则将忽略multi_accept的配置
* worker_connections  
最大连接数,默认为512

#### 3. http块
##### 3.1 http全局块
配置示例
```bash
http {
    include $type;
    default_type $type;
    log_format $name $format;
    access_log on|off;
    access_log $path $name;
    sendfile on|off;
    sendfile_max_chunk $num;
    keepalive_timeout $num;
    error_page $num $url;
}
```
说明如下  
* include
文件扩展名与文件类型映射表
* default_type
默认文件类型，默认为text/plain
$type取值有
* log_format
自定义日志格式
$name为日志格式名，用于下面access_log定义日志格式
$format具体日志格式
* access_log
http日志
on|off 表示是否取消服务日志
$path表示日志文件路径
$format为log_format定义的日志格式，默认为combined
* sendfile
是否允许sendfile方式传输文件，默认为off。可以在http块，server块，location块
* sendfile_max_chunk
每个进程每次调用传输文件大小限制，默认为0，即不设上限
$num为文件大小, k为单位
* keepalive_timeout
连接超时时间，默认为75(秒)，可以在http，server，location块设置
* error_page
发生某种错误页跳转的页面，如404等
##### 3.2 upstream块
```bash
http {
    upstream $name {
        server $ip:$port;
        server $ip:$port backup;
    }
}
```
说明如下

##### 3.3 server块
###### 3.3.1 server全局块
```bash
http {
    server {
        keepalive_requests $num;
        listen $num;
        server_name $host;
    }
}
```
说明如下  
* keepalive_requests
单连接请求上限次数
* listen
监听端口
* server_name
监听地址
###### 3.3.2 location块
```bash
http {
    server {
        location $reg {
            root $path;
            index $filename;
            proxy_pass $host;
            deny $ip;
            allow $ip;
        }
    }
}
```
说明如下
* $reg
请求的url过滤，正则匹配，\~为区分大小写，\~\*为不区分大小写
* root
根目录
* index
默认页
* proxy_pass
请求转向$host服务器
* deny
拒绝的ip
* allow
允许的ip

