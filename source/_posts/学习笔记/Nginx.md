---
title: Nginx
date: 2018-07-19
categories:
    - 学习
tags:
    - nginx
---
### 配置文件结构
```
...              #全局块
events {         #events块
   ...
}

http      #http块
{
    ...   #http全局块
    server        #server块
    { 
        ...       #server全局块
        location [PATTERN]   #location块
        {
            ...
        }
        location [PATTERN] 
        {
            ...
        }
    }
    server
    {
      ...
    }
    ...     #http全局块
}
```
* (全局设置)           设置的指令将影响其他所有设置
    * Events 
    * HTTP
        * upstream              负载均衡服务器设置
        * Server (主机设置)     指定主机和端口
            * location          用于匹配网页位置
            * location
    * Server
    
```
server继承全局块 | location继承server | upstream既不会继承其他设置也不会被继承
```

#### Nginx 的全局配置
```
user nobody nobody;
worker_processes 2;
error_log logs/error.log notice;
pid logs/nginx.pid;
worker_rlimit_nofile 65535;
 
events{
    use epoll;
    worker_connections 65536;
}
```
* **user** 是个主模块指令，指定Nginx Worker进程运行用户以及用户组，默认由nobody账号运行
* **worker_processes** 是个主模块指令，指定了Nginx要开启的进程数。每个Nginx进程平均消耗10M-12M内存。建议指定和CPU的数量一致即可
* **error_log** 是个主模块指令，用来定义全局错误日志文件。日志输出级别有debug（最详细）、info、notice、warm、error、crit（最少）可供选择
* **pid** 是个主模块指令，用来指定进程pid的存储文件位置
* **worker_rlimit_nofile** 用于绑定worker进程和CPU,linux内核2.4以上

events事件指令是设定Nginx的工作模式以及连接数上限：
* **user** 是个事件模块指令，用来指定Nginx的工作模式。Nginx支持的工作模式有select、poll、kqueue、epoll、rtsig、和/dev/poll。
其中select和poll都是标准的工作模式、kqueue和epoll是高效的工作模式、不同的是epoll用在Linux平台上，而kqueue用在BSD系统中。对于Linux
系统，epoll工作模式首先
* **worker_connections**也是事件模块指令，用于定义Nginx每个进程的最大默认连接数，默认是1024。最大客户端连接数由worker_processes和
worker_connections决定，即max_client=worker_processes\*worker_connections。在作为反向代理时，max_client=worker_processes\*worker_connections/4。
进程的最大连接数受Linux系统进程的最大打开文件数限制，在执行操作系统命令**ulimit -n 65536**后worker_connections的设置才能生效

#### HTTP服务器配置
```
http{
    include      conf/mime.types;
    default_type  application/octet-stream; 
    log_format main '$remote_addr - $remote_user [$time_local] '
     '"$request" $status $bytes_sent '
     '"$http_referer" "$http_user_agent" '
     '"$gzip_ratio"';
     log_format download '$remote_addr - $remote_user [$time_local] '
     '"$request" $status $bytes_sent '
     '"$http_referer" "$http_user_agent" '
     '"$http_range" "$sent_http_content_range"';
    client_max_body_size  20m; 
    client_header_buffer_size    32K;
    large_client_header_buffers  4 32k;
    Sendfile  on;
    tcp_nopush     on;
    tcp_nodelay    on;
    keepalive_timeout 60;
    client_header_timeout  10;
    client_body_timeout    10;
    send_timeout          10;
```

* **include** 是实现对配置文件所包含的文件的设定，可以减少主配置文件的复杂度。
* **default_type** 属于HTTP核心模块指令，这里设定默认类型为二进制流，也就是当文件类型未定义时使用这种方式，
例如在没有设置PHP环境是，Nginx是不予解析的，此时用浏览器访问PHP文件就会出现下载窗口
* **log_format** 是Nginx的HttpLog模块指令，用于指定Nginx日志的输出格式。main为此日志输出格式的名称，可以在下面access_log指令中引用
* **client_max_body_size** 用来设置允许客户端请求的最大的单个文件字节数
* **client_header_buffer_size** 用来指定客户端请求的header buffer大小。对于大多数请求，1k的缓冲区大小已经足够了，如果自定义
了消息头或者有更大的cookie，可以增加缓冲区大小，这里设置为32K
* **large_client_header_buffers**用来指定客户端请求中较大的消息头的缓存最大数量和大小， 4 为个数 32k为大小 最大缓冲缓存量4*32k
* **sendfile** 参数用于开启高效文件传输模式，将tcp_nopush和tcp_nodelay两个指令设置为on用于防止网络阻塞
* **keepalive_timeout** 设置客户端连接保持活动的超时时间，超过这个时间服务器将会关闭连接
* **client_header_timeout** 设置客户端请求头读取超时时间，如果超过这个时间,客户端还没有发送任何数据，Nginx将返回“Request time out（408）”错误
* **client_body_timeout** 设置客户端请求主体读取超时时间，如果超过这个时间,客户端还没有发送任何数据，Nginx将返回“Request time out（408）”错误，默认值60
* **send_timeout** 指定响应客户端的超时时间，这个超时仅限于两个连续活动之间的时间，如果超过这个时间，客户端还没有任何活动，Nginx将会关闭连接

#### HttpGzip模块配置
HttpGzip模块支持在线实时压缩传输数据流，通过/opt/nginx/sbin/nginx -V命令可以查看安装Nginx时的编译选项。
```
[root@vps ~]# /opt/nginx/sbin/nginx  -V
nginx version: nginx/1.0.14
built by gcc 4.4.6 20110731 (Red Hat 4.4.6-3) (GCC)
configure arguments: --with-http_stub_status_module --with-http_gzip_static_module --prefix=/opt/nginx
```
下面是HttpGzip模块在Nginx配置中的相关属性设置
```
gzip on;
gzip_min_length 1k;
gzip_buffers 4 16k;
gzip_http_version 1.1;
gzip_comp_level 2;
gzip_types text/plain application/x-javascript text/css application/xml
gzip_vary on;
```
* **gzip**用于设置开启或者关闭gzip模块
* **gzip_min_length** 表示允许压缩的页面最小字节数，页面字节数据从header头中的Content-Length中获取，默认值0，建议设置成大于1K
,小于1K可能会越压缩越大
* **gzip_buffers** 表示申请4个单位为16K的内存，作为压缩结果流缓存，默认值是申请与原始值大小相同的的内存空间来存储gzip压缩结果
* **gzip_http_version** 用于设置识别HTTP协议版本，默认是1.1，目前大部分浏览器已支持Gzip解压，使用默认即可
* **gzip_comp_level** 用来指定Gzip压缩比，1压缩比最小，处理速度快，9压缩比最大，传输速度快，但是处理最慢，也比较消耗cpu资源
* **gzip_types** 用来指定压缩的类型，无论是否指定，"text/html"类型总是被压缩的
* **gzip_vary** 选项可以让前端的缓存服务器经过Gzip压缩的页面，例如用Squid缓存经过Nginx压缩的数据

#### 负载均衡配置
```
upstream xiebiao123.com{
    ip_hash;
    server 192.168.8.11:80;
    server 192.168.8.12:80 down;
    server 192.168.8.13:8009 max_fails=3 fail_timeout=20s;
    server 192.168.8.146:8080;
}
```
upstream是Nginx的HTTP Upstream模块，这个模块通过一个简单的调度算法来实现客户端IP到后端服务器的负载均衡。
在上面的设定中，通过upstream指令指定了一个负载均衡器的名称xiebiao123.com。这个名称可以任意指定，在后面需要的地方直接调用即可
Nginx的负载均衡模块目前支持4种调度算法:
* 轮询（默认）：每个请求按时间顺序逐一分配到不同的后端服务，如果后端某台服务器宕机，故障系统被自动剔除。
* weigh ：指定轮询权值，weight值越大，分配到的访问几率越高
* ip_hash ： 每个请求按访问IP的hash结果分配，这样来自同一个IP的访客固定访问一个后台服务
* fair ： 根据后端服务的响应时间来分配请求，响应时间短的优先分配，Nginx本身不支持fair,需要使用必须下载Nginx的upstream_fair模块
* url_hash ：按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器，可以进一步提升后端服务的效率，Nginx本身不支持url_hash
,可以安装Nginx的hash软件包

在HTTP Upstream模块中，可以通过server指令指定后端服务器的IP地址和端口，同时还可以设定每个后端服务器在负载均衡调度中的状态。常用的状态有：
* down：表示当前的server暂时不参与负载均衡
* backup：预留的备份机器。当其他所有的非backup机器出现故障或者忙的时候，才会请求backup机器，因此这台机器的压力最轻；
* max_fails：允许请求失败的次数，默认为1。当超过最大次数时，返回proxy_next_upstream 模块定义的错误；
* fail_timeout：在经历了max_fails次失败后，暂停服务的时间。max_fails可以和fail_timeout一起使用。

> 注意，当负载调度算法为ip_hash时，后端服务器在负载均衡调度中的状态不能是weight(权重)和backup。

#### server虚拟主机配置
```
server{
    listen         80;
    server_name    192.168.12.188  www.ixdba.net;
    index index.html index.htm index.jsp;
    root  /web/wwwroot/www.ixdba.net
    charset gb2312;
    access_log logs/nginx.log main;
```
* listen 用于指定虚拟机的服务端口
* server_name 用来指定IP地址或者域名，多个域名之间用空格分开
* index 用于设定访问的默认首页地址
* root 用于指定虚拟机的网页根目录，这个目录可以是相对路径也可以是绝对路径
* charset 用于网页的默认编码格式
* access_log 用来指定此虚拟机的访问日志存放路径，最后的main用于指定日志的格式（上文中的main）
> 建议将虚拟机配置写进另外一个文件，然后通过include指令包含进来，这样更方便管理和维护

#### location URL匹配配置
location 支持正则表达式匹配，也支持条件判断匹配，用户可以通过location指令实现Nginx对动、静态网页进行过滤
处理。使用location URL匹配配置还可以实现反向代理，用于实现PHP动态解析或者负载均衡策略。
```
location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$  {
    root    /web/wwwroot/www.ixdba.net;
    expires 30d;
}
```
以上这段设置是通过location指令来对网页URL进行分析处理，所有扩展名以.gif、.jpg、.jpeg、.png、.bmp、.swf结尾的静态文件都交给nginx处理，
而expires用来指定静态文件的过期时间，这里是30天

```
location ~ .*.jsp$ { 
    index index.jsp;       
    proxy_pass http://localhost:8080;
}
```
在最后这段设置中，location是对此虚拟主机下动态网页的过滤处理，也就是将所有以.jsp为后缀的文件都交给本机的8080端口处理。

#### StubStatus模块配置
stubStatus模块能够获取Nginx自上次启动以来的工作状态，此模块非核心模块，需要在Nginx编译安装时手工指定才能使用此功能，
以下指令启用获取Nginx工作状态的功能
```
location /NginxStatus {
    stub_status      on;
    access_log       logs/NginxStatus.log;
    auth_basic       "NginxStatus";
    auth_basic_user_file    ../htpasswd;
}
```
* stub_status 设置为no,表示启用StubStatus的工作状态统计功能
* access_log 用来指定StubStatus模块的访问日志文件
* auth_basic Nginx的一种认证机制
* auth_basic_user_file 用来指定认证的密码文件

### 配置详解
https://www.cnblogs.com/knowledgesea/p/5175711.html

### 负载均衡 & 反向代理
https://www.cnblogs.com/knowledgesea/p/5199046.html


<!-- more -->