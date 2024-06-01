# Nginx

## 一、nginx简介

### 1.1 nginx是什么

nginx(engine x)是一个高性能的http和**反向代理**web服务器，同时也提供了IMAP/POP3/SMTP服务，其特点是占有内存少，并发能力强

nginx高性能的服务器，处理高并发能力十分强大，能经受高负载的考验

nginx支持热部署，启动简单，可以做到7*24小时不间断运行，不需要重启

### 1.2 正向代理和反向代理

**正向代理**：是一个位于客户端和目标服务器之间的代理服务器，为了从原始服务器取得内容，客户端向代理服务器发送一个请求，并且指定目标服务器，之后代理向目标服务器转交并且将获得的内容返回给客户端；正向代理的情况下客户端必须要进行一些特别的设置才能使用

<img src="C:\Users\zhezhou\Desktop\工作文档\学习笔记\Nginx.assets\image-20230918101538596.png" alt="image-20230918101538596" style="zoom:80%;" />

**反向代理**：对于客户端来说吗，反向代理服务器就好像目标服务器，并且客户端不需要进行任何设置，客户端向反向代理发送请求，接着反向代理判断请求走向何处，并将请求转交给服务端，客户端只需要将反向代理服务器当成真正的服务器就好了

<img src="C:\Users\zhezhou\Desktop\工作文档\学习笔记\Nginx.assets\image-20230918101821712.png" alt="image-20230918101821712" style="zoom:80%;" />

> 正向代理中，客户端client和代理proxy处于同一个局域网中，对server透明
>
> 反向代理中，代理proxy和服务端server处于同一个局域网中，对client透明

### 1.3 nginx负载均衡

nginx有三种负载均衡的策略

1. 轮询法(默认)：每个请求按照时间顺寻逐一分配到不同的后端服务器，如果某个后端服务器宕机，则自动剔除
2. weight权重模式：加权轮询，指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况；这种方式比较灵活，当后端服务器性能存在差异的时候，通过配置权重，可以让服务器的性能得到充分发挥，有效利用资源；weight权重越高，被访问的概率越大
3. ip哈希：轮询的每一次请求大概率不会打在同一台服务器上，如果用户在某一台服务器登录了，但在下一次请求中轮询策略将请求发送到了另一台服务器，那么用户的登录状态就失效了，这显然是不合理的，ip hash能解决这个问题，如果客户端已经访问了某个服务器，当用户再次访问时，会将该请求通过hash算法，自动定位到该服务器，每个请求按照访问ip的hash结果分配

### 1.4 nginx动静分离

为了加快浏览器的解析速度，可以把动态页面和静态页面由不同的服务器来解析，加快解析速度

## 二、nginx的安装

windows下载地址：http://nginx.org/en/download.html

下载完成之后解压即可

Linux下载

为了安装nginx，首先需要安装c语言编译器：

```shell
yum -y install gcc gcc-c++ kernel-devel
```

然后安装nginx依赖的软件包：

```shell
yum install -y pcre pcre-devel zlib zlib-devel openssl openssl-devel
```

下载nginx的安装包

```shell
wget http://nginx.org/download/nginx-1.18.0.tar.gz
```

解压

```shell
tar -zxvf nginx-1.18.0.tar.gz -C /usr/local/nginx
```

如果有需要的话可以进行配置：

```shell
# 来到指定目录
cd /usr/local/src/nginx-1.18.0/
# 配置nginx
./configure \
--prefix=/opt/server/nginx \
--conf-path=/opt/server/nginx/conf/nginx.conf \
--pid-path=/opt/server/nginx/conf/nginx.pid \
--lock-path=/var/lock/nginx.lock \
--error-log-path=/opt/server/nginx/logs/error.log \
--http-log-path=/opt/server/nginx/logs/access.log \
--with-http_stub_status_module --with-http_ssl_module
# 或者采用默认配置
./configure
```

在安装目录下执行编译和安装

```shell
make

make install
```

安装完成之后在安装目录下会多出一个sbin目录，使用命令启动nginx

```shell
./nginx -c conf/nginx.conf

##常用命令：
./nginx  启动
./nginx -s stop  停止
./nginx -s quit  安全退出
./nginx -s reload  重新加载配置文件  如果我们修改了配置文件，就需要重新加载。
ps aux|grep nginx  查看nginx进程
./nginx -V   查看nginx详细配置
```

## 三、nginx配置文件

### 3.1 nginx配置文件结构

nginx配置文件由模块组成，这些模块由配置文件中指定的指令控制

指令分为两种：一种是简单指令，一种是块指令

+ **简单指令**：简单指令由名称和参数组成，由空格分开由分号结束
+ **块指令**：与简单指令相似，由一组{}包围，如果块指令可以在{}内包含其他指令，则称为上下文

简单的nginx配置文件：

```conf
# 全局区
worker_processes  1;  # 有1个工作的worker子进程
#可以自行修改,但太大会造成相互争夺CPU,一般来说：子进程数 = CPU数 * 核数

# 配置nginx的连接特性
events {
    worker_connections  1024; # 1个worker子进程能同时允许的最大连接数
}

# http服务器主要段
http {
    # 设定mime类型,类型由mime.types文件定义
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on; # 是否调用sendfile来输出文件

    keepalive_timeout  65; # 连接超时时间

    # 虚拟主机段
    server {
        listen       80; # 监听端口
        server_name  localhost; # 监听主机\域名\端口

    # 定义访问响应
        location / {
            root   html; # 根目录定位
            index  index.html index.htm; # 默认访问页面定位
        }

    # 定义错误提示页面
        location = /50x.html {
            root   html;
        }
}
```

### 3.2 全局块

配置文件的全局块是从文件开始到events块之间的内容，主要会设置一些影响 nginx 服务器整体运行的配置指令，主要包括配置运行 Nginx 服务器的用户（组）、允许生成的 worker process 数，进程 PID 存放路径、日志存放路径和类型以及配置文件的引入等

这是 Nginx 服务器并发处理服务的关键配置，worker_processes 值越大，可以支持的并发处理量也越多，但是会受到硬件、软件等设备的制约

- user [user] [group]  指定可以运行nginx服务的用户和用户组，只能在全局块配置 user指令在Windows上不生效，如果你制定具体用户和用户组会报警告

- worker_processes nginx进程数量worker_processes 比如设置为2 nginx将会开启一个master进程和2两个worker进程
- pid  logs/nginx.pid 存放pid文件

- error_log  logs/error.log  全局错误日志类型 debug info warn error 存放地址

```conf
##默认配置：
worker_processes 1
##附加配置：
##Nginx worker进程运行的用户及用户组，如果不指定group，则使用与user名称相同的组。
##语法：user user [group];
##Default:user nobody nobody;
user  root;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

##pid文件（master进程ID的pid文件存放路径）的路径
#pid        logs/nginx.pid;
```

### 3.3 events块

events 块涉及的指令主要影响 Nginx 服务器与用户的网络连接，常用的设置包括是否开启对多 work process 下的网络连接进行序列化，是否允许同时接收多个网络连接，选取哪种事件驱动模型来处理连接请求，每个 word process 可以同时支持的最大连接数等

- accept_mutex 默认开启-开启之后nginx 的多个worker将会以串行的方式来处理，只会有一个worker将会被唤起，其他的worker继续睡眠，如果不开启将会造成惊群效应多个worker全部唤起不过只有一个Worker能获取新连接，其它的Worker会重新进入休眠状态

- worker_connections 单个进程最大连接数（最大连接数=连接数+进程数）

```conf
##默认配置：
events {
	##每个 work process 支持的最大连接数为 1024
	worker_connections 1024  
}
##附加配置：
events {
    #参考事件模型，use [ kqueue | rtsig | epoll | /dev/poll | select | poll ]; epoll模型
    #是Linux 2.6以上版本内核中的高性能网络I/O模型，linux建议epoll，如果跑在FreeBSD上面，就用kqueue模型。
    #补充说明：
    #与apache相类，nginx针对不同的操作系统，有不同的事件模型
    #A）标准事件模型
    #Select、poll属于标准事件模型，如果当前系统不存在更有效的方法，nginx会选择select或poll
    #B）高效事件模型
    #Kqueue：使用于FreeBSD 4.1+, OpenBSD 2.9+, NetBSD 2.0 和 MacOS X.使用双处理器的MacOS X系统使用kqueue可能会造成内核崩溃。
    #Epoll：使用于Linux内核2.6版本及以后的系统。
    #/dev/poll：使用于Solaris 7 11/99+，HP/UX 11.22+ (eventport)，IRIX 6.5.15+ 和 Tru64 UNIX 5.1A+。
    #Eventport：使用于Solaris 10。 为了防止出现内核崩溃的问题， 有必要安装安全补丁。
    use epoll


    #单个进程最大连接数（最大连接数=连接数+进程数）
    #根据硬件调整，和前面工作进程配合起来用，尽量大，但是别把cup跑到100%就行。
    worker_connections  1024;

    #keepalive 超时时间
    keepalive_timeout 60;

    #客户端请求头部的缓冲区大小。这个可以根据你的系统分页大小来设置，一般一个请求头的大小不会超过1k，不过由于一般系统分页都要大于1k，所以这里设置为分页大小。
    #分页大小可以用命令getconf PAGESIZE 取得。
    #[root@web001 ~]# getconf PAGESIZE
    #但也有client_header_buffer_size超过4k的情况，但是client_header_buffer_size该值必须设置为“系统分页大小”的整倍数。
    client_header_buffer_size 4k;

    #这个将为打开文件指定缓存，默认是没有启用的，max指定缓存数量，建议和打开文件数一致，inactive是指经过多长时间文件没被请求后删除缓存。
    open_file_cache max=65535 inactive=60s;


    #这个是指多长时间检查一次缓存的有效信息。
    #语法:open_file_cache_valid time 默认值:open_file_cache_valid 60 使用字段:http, server, location 这个指令指定了何时需要检查open_file_cache中缓存项目的有效信息.
    open_file_cache_valid 80s;


    #open_file_cache指令中的inactive参数时间内文件的最少使用次数，如果超过这个数字，文件描述符一直是在缓存中打开的，如上例，如果有一个文件在inactive时间内一次没被使用，它将被移除。
    #语法:open_file_cache_min_uses number 默认值:open_file_cache_min_uses 1 使用字段:http, server, location  这个指令指定了在open_file_cache指令无效的参数中一定的时间范围内可以使用的最小文件数,如果使用更大的值,文件描述符在cache中总是打开状态.
    open_file_cache_min_uses 1;

    #语法:open_file_cache_errors on | off 默认值:open_file_cache_errors off 使用字段:http, server, location 这个指令指定是否在搜索一个文件是记录cache错误.
    open_file_cache_errors on;
}
```

### 3.4 http块

是 Nginx 服务器配置中最频繁的部分，代理、缓存和日志定义等绝大多数功能和第三方模块的配置都在这里。

需要注意的是：http 块也可以包括 **http** **全局块**、**server块**

1. http全局块：

   http 全局块配置的指令包括文件引入、MIME-TYPE 定义、日志自定义、连接超时时间、单链接请求数上限等。

   **upstream**（上游服务器设置，主要为反向代理、负载均衡相关配置，upstream 的指令用于设置一系列的后端服务器，设置反向代理及后端服务器的负载均衡)

2. server块

   这块和虚拟主机有密切关系，虚拟主机从用户角度看，和一台独立的硬件主机是完全一样的，该技术的产生是为了节省互联网服务器硬件成本

   每个 http 块可以包括多个 server 块，而每个 server 块就相当于一个虚拟主机，而每个 server 块也分为全局 server 块，以及可以同时包含多个 locaton 块

   + server全局块：最常见的配置是本虚拟机主机的监听配置和本虚拟主机的名称或 IP 配置
   + location块：一个server可以配置多个location块，这块的主要作用是基于 Nginx 服务器接收到的请求字符串（例如 server_name/uri-string），对虚拟主机名称（也可以是 IP 别名）之外的字符串（例如 前面的 /uri-string）进行匹配，对特定的请求进行处理。地址定向、数据缓存和应答控制等功能，还有许多第三方模块的配置也在这里进行

```conf
##http基本块结构：
# http服务器主要段
http {
    # 设定mime类型,类型由mime.types文件定义
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on; # 是否调用sendfile来输出文件

    keepalive_timeout  65; # 连接超时时间

    # 虚拟主机段
    server {
        listen       80; # 监听端口
        server_name  localhost; # 监听主机\域名\端口

    # 定义访问响应
        location / {
            root   html; # 根目录定位
            index  index.html index.htm; # 默认访问页面定位
        }

    # 定义错误提示页面
        location = /50x.html {
            root   html;
        }
    }
}
```

http全配置

```properties
#设定http服务器，利用它的反向代理功能提供负载均衡支持
http{
    #文件扩展名与文件类型映射表
    include mime.types;

    #默认文件类型
    default_type application/octet-stream;

    #默认编码
    charset utf-8;

    #服务器名字的hash表大小
    #保存服务器名字的hash表是由指令server_names_hash_max_size 和server_names_hash_bucket_size所控制的。参数hash bucket size总是等于hash表的大小，并且是一路处理器缓存大小的倍数。在减少了在内存中的存取次数后，使在处理器中加速查找hash表键值成为可能。如果hash bucket size等于一路处理器缓存的大小，那么在查找键的时候，最坏的情况下在内存中查找的次数为2。第一次是确定存储单元的地址，第二次是在存储单元中查找键 值。因此，如果Nginx给出需要增大hash max size 或 hash bucket size的提示，那么首要的是增大前一个参数的大小.
    server_names_hash_bucket_size 128;

    #客户端请求头部的缓冲区大小。这个可以根据你的系统分页大小来设置，一般一个请求的头部大小不会超过1k，不过由于一般系统分页都要大于1k，所以这里设置为分页大小。分页大小可以用命令getconf PAGESIZE取得。
    client_header_buffer_size 32k;

    #客户请求头缓冲大小。nginx默认会用client_header_buffer_size这个buffer来读取header值，如果header过大，它会使用large_client_header_buffers来读取。
    large_client_header_buffers 4 64k;

    #设定通过nginx上传文件的大小
    client_max_body_size 8m;

    #开启高效文件传输模式，sendfile指令指定nginx是否调用sendfile函数来输出文件，对于普通应用设为 on，如果用来进行下载等应用磁盘IO重负载应用，可设置为off，以平衡磁盘与网络I/O处理速度，降低系统的负载。注意：如果图片显示不正常把这个改成off。
    #sendfile指令指定 nginx 是否调用sendfile 函数（zero copy 方式）来输出文件，对于普通应用，必须设为on。如果用来进行下载等应用磁盘IO重负载应用，可设置为off，以平衡磁盘与网络IO处理速度，降低系统uptime。
    sendfile on;

    #开启目录列表访问，合适下载服务器，默认关闭。
    autoindex on;

    #此选项允许或禁止使用socke的TCP_CORK的选项，此选项仅在使用sendfile的时候使用
    tcp_nopush on;

    tcp_nodelay on;

    #长连接超时时间，单位是秒
    keepalive_timeout 120;

    #FastCGI相关参数是为了改善网站的性能：减少资源占用，提高访问速度。下面参数看字面意思都能理解。
    fastcgi_connect_timeout 300;
    fastcgi_send_timeout 300;
    fastcgi_read_timeout 300;
    fastcgi_buffer_size 64k;
    fastcgi_buffers 4 64k;
    fastcgi_busy_buffers_size 128k;
    fastcgi_temp_file_write_size 128k;

    #gzip模块设置
    gzip on; #开启gzip压缩输出
    gzip_min_length 1k;    #最小压缩文件大小
    gzip_buffers 4 16k;    #压缩缓冲区
    gzip_http_version 1.0; #压缩版本（默认1.1，前端如果是squid2.5请使用1.0）
    gzip_comp_level 2;     #压缩等级
    gzip_types text/plain application/x-javascript text/css application/xml;    #压缩类型，默认就已经包含textml，所以下面就不用再写了，写上去也不会有问题，但是会有一个warn。
    gzip_vary on;

    #开启限制IP连接数的时候需要使用
    #limit_zone crawler $binary_remote_addr 10m;


    #负载均衡配置
    upstream piao.jd.com {

        #upstream的负载均衡，weight是权重，可以根据机器配置定义权重。weigth参数表示权值，权值越高被分配到的几率越大。
        server 192.168.80.121:80 weight=3;
        server 192.168.80.122:80 weight=2;
        server 192.168.80.123:80 weight=3;

        #nginx的upstream目前支持4种方式的分配
        #1、轮询（默认）
        #每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除。
        #2、weight
        #指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况。
        #例如：
        #upstream bakend {
        #    server 192.168.0.14 weight=10;
        #    server 192.168.0.15 weight=10;
        #}
        #2、ip_hash
        #每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session的问题。
        #例如：
        #upstream bakend {
        #    ip_hash;
        #    server 192.168.0.14:88;
        #    server 192.168.0.15:80;
        #}
        #3、fair（第三方）
        #按后端服务器的响应时间来分配请求，响应时间短的优先分配。
        #upstream backend {
        #    server server1;
        #    server server2;
        #    fair;
        #}
        #4、url_hash（第三方）
        #按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器，后端服务器为缓存时比较有效。
        #例：在upstream中加入hash语句，server语句中不能写入weight等其他的参数，hash_method是使用的hash算法
        #upstream backend {
        #    server squid1:3128;
        #    server squid2:3128;
        #    hash $request_uri;
        #    hash_method crc32;
        #}

        #tips:
        #upstream bakend{#定义负载均衡设备的Ip及设备状态}{
        #    ip_hash;
        #    server 127.0.0.1:9090 down;
        #    server 127.0.0.1:8080 weight=2;
        #    server 127.0.0.1:6060;
        #    server 127.0.0.1:7070 backup;
        #}
        #在需要使用负载均衡的server中增加 proxy_pass http://bakend/;

        #每个设备的状态设置为:
        #1.down表示单前的server暂时不参与负载
        #2.weight为weight越大，负载的权重就越大。
        #3.max_fails：允许请求失败的次数默认为1.当超过最大次数时，返回proxy_next_upstream模块定义的错误
        #4.fail_timeout:max_fails次失败后，暂停的时间。
        #5.backup： 其它所有的非backup机器down或者忙的时候，请求backup机器。所以这台机器压力会最轻。

        #nginx支持同时设置多组的负载均衡，用来给不用的server来使用。
        #client_body_in_file_only设置为On 可以讲client post过来的数据记录到文件中用来做debug
        #client_body_temp_path设置记录文件的目录 可以设置最多3层目录
        #location对URL进行匹配.可以进行重定向或者进行新的代理 负载均衡
    }


    #虚拟主机的配置
    server {
        #监听端口
        listen 80;

        #域名可以有多个，用空格隔开
        server_name www.jd.com jd.com;
        #默认入口文件名称
        index index.html index.htm index.php;
        root /data/www/jd;

        #对******进行负载均衡
        location ~ .*.(php|php5)?$
        {
            fastcgi_pass 127.0.0.1:9000;
            fastcgi_index index.php;
            include fastcgi.conf;
        }

        #图片缓存时间设置
        location ~ .*.(gif|jpg|jpeg|png|bmp|swf)$
        {
            expires 10d;
        }

        #JS和CSS缓存时间设置
        location ~ .*.(js|css)?$
        {
            expires 1h;
        }

        #日志格式设定
        #$remote_addr与$http_x_forwarded_for用以记录客户端的ip地址；
        #$remote_user：用来记录客户端用户名称；
        #$time_local： 用来记录访问时间与时区；
        #$request： 用来记录请求的url与http协议；
        #$status： 用来记录请求状态；成功是200，
        #$body_bytes_sent ：记录发送给客户端文件主体内容大小；
        #$http_referer：用来记录从那个页面链接访问过来的；
        #$http_user_agent：记录客户浏览器的相关信息；
        #通常web服务器放在反向代理的后面，这样就不能获取到客户的IP地址了，通过$remote_add拿到的IP地址是反向代理服务器的iP地址。反向代理服务器在转发请求的http头信息中，可以增加x_forwarded_for信息，用以记录原有客户端的IP地址和原来客户端的请求的服务器地址。
        log_format access '$remote_addr - $remote_user [$time_local] "$request" '
        '$status $body_bytes_sent "$http_referer" '
        '"$http_user_agent" $http_x_forwarded_for';

        #定义本虚拟主机的访问日志
        access_log  /usr/local/nginx/logs/host.access.log  main;
        access_log  /usr/local/nginx/logs/host.access.404.log  log404;

        #对 "/connect-controller" 启用反向代理
        location /connect-controller {
            proxy_pass http://127.0.0.1:88; #请注意此处端口号不能与虚拟主机监听的端口号一样（也就是server监听的端口）
            proxy_redirect off;
            proxy_set_header X-Real-IP $remote_addr;

            #后端的Web服务器可以通过X-Forwarded-For获取用户真实IP
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

            #以下是一些反向代理的配置，可选。
            proxy_set_header Host $host;

            #允许客户端请求的最大单文件字节数
            client_max_body_size 10m;

            #缓冲区代理缓冲用户端请求的最大字节数，
            #如果把它设置为比较大的数值，例如256k，那么，无论使用firefox还是IE浏览器，来提交任意小于256k的图片，都很正常。如果注释该指令，使用默认的client_body_buffer_size设置，也就是操作系统页面大小的两倍，8k或者16k，问题就出现了。
            #无论使用firefox4.0还是IE8.0，提交一个比较大，200k左右的图片，都返回500 Internal Server Error错误
            client_body_buffer_size 128k;

            #表示使nginx阻止HTTP应答代码为400或者更高的应答。
            proxy_intercept_errors on;

            #后端服务器连接的超时时间_发起握手等候响应超时时间
            #nginx跟后端服务器连接超时时间(代理连接超时)
            proxy_connect_timeout 90;

            #后端服务器数据回传时间(代理发送超时)
            #后端服务器数据回传时间_就是在规定时间之内后端服务器必须传完所有的数据
            proxy_send_timeout 90;

            #连接成功后，后端服务器响应时间(代理接收超时)
            #连接成功后_等候后端服务器响应时间_其实已经进入后端的排队之中等候处理（也可以说是后端服务器处理请求的时间）
            proxy_read_timeout 90;

            #设置代理服务器（nginx）保存用户头信息的缓冲区大小
            #设置从被代理服务器读取的第一部分应答的缓冲区大小，通常情况下这部分应答中包含一个小的应答头，默认情况下这个值的大小为指令proxy_buffers中指定的一个缓冲区的大小，不过可以将其设置为更小
            proxy_buffer_size 4k;

            #proxy_buffers缓冲区，网页平均在32k以下的设置
            #设置用于读取应答（来自被代理服务器）的缓冲区数目和大小，默认情况也为分页大小，根据操作系统的不同可能是4k或者8k
            proxy_buffers 4 32k;

            #高负荷下缓冲大小（proxy_buffers*2）
            proxy_busy_buffers_size 64k;

            #设置在写入proxy_temp_path时数据的大小，预防一个工作进程在传递文件时阻塞太长
            #设定缓存文件夹大小，大于这个值，将从upstream服务器传
            proxy_temp_file_write_size 64k;
        }

        #本地动静分离反向代理配置
        #所有jsp的页面均交由tomcat或resin处理
        location ~ .(jsp|jspx|do)?$ {
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_pass http://127.0.0.1:8080;
        }
    }
}
```

### 3.5 配置总结

Nginx 配置文件主要分成四部分：main（全局设置）、server（主机设置）、upstream（上游服务器设置，主要为反向代理、负载均衡相关配置）和 location（URL匹配特定位置后的设置）。

main 部分设置的指令影响其他所有部分的设置；

server 部分的指令主要用于制定虚拟主机域名、IP 和端口号；

upstream 的指令用于设置一系列的后端服务器，设置反向代理及后端服务器的负载均衡；

location 部分用于匹配网页位置（比如，根目录“/”，“/images”，等等）