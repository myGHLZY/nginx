# Nginx

## Nginx简介
1. Nginx（“engine x”）一个具有高性能的【HTTP】和【反向代理】的
【WEB服务器】，同时也是一个【POP3/SMTP/IMAP代理服务器】，是
由伊戈尔·赛索耶夫(俄罗斯人)使用C语言编写的，Nginx的第一个版本是
2004年10月4号发布的0.1.0版本。另外值得一提的是伊戈尔·赛索耶夫将
Nginx的源码进行了开源，这也为Nginx的发展提供了良好的保障。
2. 正向代理与反向代理  
   ***正向代理***
   ![](img/截屏2024-03-21%2022.29.06.png)
    - 正向代理是位于浏览器也就是客户端与服务器之间，客户端向代理发送一个请求并制定目标（原始服务器），然后代理向原始服务器转发请求并将获得的内容返回给客户端  
    - 像校园网访问、VPN、翻墙，都属于正向代理
    ![](img/截屏2024-03-21%2022.33.54.png)
    ![](img/截屏2024-03-21%2022.37.28.png)
   ***反向代理***
   ![](img/截屏2024-03-21%2022.29.52.png)
   - 反向代理则以代理服务器来接受Internet上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给Internet上请求的客户端。
   - ![](img/截屏2024-03-21%2022.37.37.png)

3. 优点
   - 速度更快、并发更高。Nginx之所以有这么高的并发处理能力和这么好的性能原因在于Nginx采用了多进程和I/O多路复用(epoll)的底层实现。
   - 配置简单，扩展性强
   - 高可靠性
   - 热部署
   - 成本低、BSD许可证
    ![](img/截屏2024-03-21%2022.39.43.png)
    - Linux就属于GPL

4. 关于安装
   这里采用手动式的安装
   ```
   # 关于nginx需要的一些工具和库(未尝试)
   yum install -y gcc pcre pcre-devel zlib zlib-devel openssl openssl-devel
   ```
   (1)进入官网查找需要下载版本的链接地址，然后使用wget命令进行下载
   ```
    wget http://nginx.org/download/nginx-1.25.0.tar.gz
   ```
   (2)建议大家将下载的资源进行包管理
   ```
    mkdir -p nginx/core
    mv nginx-1.16.1.tar.gz nginx/core
   ```
   (3)解压
   ```
    tar -xzf nginx-1.25.0.tar.gz
   ```
   (4)进入资源文件中，执行
   ```
    ./configure --prefix=/usr/local/nginx \
    --sbin-path=/usr/local/nginx/sbin/nginx \
    --modules-path=/usr/local/nginx/modules \
    --conf-path=/usr/local/nginx/conf/nginx.conf \
    --error-log-path=/usr/local/nginx/logs/error.log \
    --http-log-path=/usr/local/nginx/logs/access.log \
    --pid-path=/usr/local/nginx/logs/nginx.pid \
    --lock-path=/usr/local/nginx/logs/nginx.lock
   ```
   (5)编译与安装
   ```
    make
   ```
   ```
    make install
   ```
- 如果失败，请自行安装，检查防火墙等
![](img/截屏2024-03-21%2015.48.08.png)

5. nginx的启停命令
   ![](img/截屏2024-03-21%2022.53.46.png)
   上图就是nginx的工作模式。/usr/local/nginx/logs/nginx.pid ,所以可以通过查看该文件来获取nginx的master进程ID.
    - 信号方式暂时不叙述
    - nginx命令行

## Nginx核心配置文件结构
   - Nginx的核心配置文件默认是放在/usr/local/nginx/conf/nginx.conf。nginx.conf配置文件中默认有三大块：全局块、events块、http块http块中可以配置多个server块，每个server块又可以配置多个location块。
1. 全局块
   ![](img/截屏2024-03-21%2023.03.17.png)
   - user指令用于配置运行Nginx服务器的worker进程的用户和用户组。
  
        |语法 |user user [group]|  
        |---|---|  
        |默认值| nobody|  
        |位置| 全局块|  
    ```
        # 利用 useradd 添加一个用户
        useradd testUser
        ---
        # 在全局块中
        user testUser
        ---
        # 在/home/testUser 中创建/html/index.html,配置
        location / {
            root /home/testUser/html;
            index index.html index.htm;
        }
        ---
        # 发现当前用户只能访问自己目录下的静态资源
        综上所述，使用user指令可以指定启动运行工作进程的用户及用户组，这样对于系统的权限访问控制的更加精细，也更加安全。
    ```
    - master_process:用来指定是否开启工作进程。

        |语法| master_process on\off|  
        |---|---|  
        |默认值 |master_process on|  
        |位置| 全局块|  
    - worker_processes:用于配置Nginx生成工作进程的数量，这个是Nginx服务器实现并发处理服务的关键所在。理论上来说workder process的值越大，可以支持的并发处理量也越多，但事实上这个值的设定是需要受到来自服务器自身的限制，建议将该值和服务器CPU的内核数保存一致。

        |语法| worker_processes num/auto;|
        |---|---|
        |默认值| 1|
        |位置| 全局块|
    ```
        worker_processes 2;
    ```
    可见 worker进程变成两个
    ![](img/截屏2024-03-21%2023.10.26.png)

    - daemon：设定Nginx是否以守护进程的方式启动。默认为on
    - pid:用来配置Nginx当前master进程的进程号ID存储的文件路径
    - error_log:用来配置Nginx的错误日志存放路径
    - include:用来引入其他配置文件，使Nginx的配置更加灵活  
2. events块
   - accept_mutex:用来设置Nginx网络连接序列化
  
    |语法 accept_mutex on|off;|
    |---|---|
    |默认值| accept_mutex on; |
    |位置 |events|

    ![](img/截屏2024-03-21%2023.18.30.png)

    - multi_accept:用来设置是否允许同时接收多个网络连接
    - worker_connections：用来配置单个worker进程最大的连接数
    - use:用来设置Nginx服务器选择哪种事件驱动来处理网络消息。此处所选择事件处理模型是Nginx优化部分的一个重要内容，method的可选值有select/poll/epoll/kqueue等，之前在准备centos环境的时候，我们强调过要使用linux内核在2.6以上，就是为了能使用epoll函数来优化Nginx
    - 我们应该注意 multi_accept 和 accept_mutex
    ![](img/截屏2024-03-21%2023.33.03.png)
    ![](img/截屏2024-03-21%2023.33.39.png)
    可以看出accept_mutex侧重于请求到来之后唤醒worker, multi_accept侧重于是一次接收请求的个数，两者其实关系不大。此外，关于accept_mutex，如果开启，在请求量很大的情况下，反而会降低性能。

3. http块
   - 定义MIME-Type
  ```
    include mime.types;
    default_type application/octet-stream;
  ```
  - default_type:用来配置Nginx响应前端请求默认的MIME类型。可以位于http、server、location
  ```
    location /get_text {
        #这里也可以设置成text/plain
        default_type text/html;
        return 200 "This is nginx's text";
    }
    # 此时访问 /get_text 就会直接返回字符串
  ```
  - 自定义服务日志
  - sendfile:用来设置Nginx服务器是否使用sendfile()传输文件，该属性可以大大提高Nginx处理静态资源的性能。可以位于http、server、location。
  可以参考：https://blog.csdn.net/qq_36885515/article/details/123137996
  ![](img/截屏2024-03-22%2017.40.42.png)
  ![](img/截屏2024-03-22%2013.49.31.png)
  - keepalive_timeout:用来设置长连接的超时时间。
  ![](img/截屏2024-03-21%2023.45.16.png)
  - keepalive_requests:用来设置一个keep-alive连接使用的次数

## Nginx配置成系统服务
1. 在/usr/lib/systemd/system目录下添加nginx.service,内容如下:
```
[Unit]
Description=nginx web service
Documentation=http://nginx.org/en/docs/
After=network.target

[Service]
Type=forking
PIDFile=/usr/local/nginx/logs/nginx.pid
ExecStartPre=/usr/local/nginx/sbin/nginx -t -c /usr/local/nginx/conf/nginx.conf
ExecStart=/usr/local/nginx/sbin/nginx
ExecReload=/usr/local/nginx/sbin/nginx -s reload
ExecStop=/usr/local/nginx/sbin/nginx -s stop
PrivateTmp=true

[Install]
WantedBy=default.target
```
- 添加完成后，如有权限问题，设置权限
2. 使用系统命令来操作Nginx服务
```
启动: systemctl start nginx
停止: systemctl stop nginx
重启: systemctl restart nginx
重新加载配置文件: systemctl reload nginx
查看nginx状态: systemctl status nginx
开机启动: systemctl enable nginx
```

## Nginx命令配置到系统环境
1. 修改/etc/profile文件
```
vim /etc/profile
#在最后一行添加
export PATH=$PATH:/usr/local/nginx/sbin
#立即生效
source /etc/profile
```
2. 可在其他目录执行nginx命令
```
    nginx -V
```


## Nginx静态资源部署
### 1. Nginx静态资源的配置指令
1. listen:指令用来配置监听端口

|语法| listen address[:port] [default_server]...;listen port [default_server]...;|
|---|---|
|默认值 |listen *:80 or *:8000|
|位置 |server|

```
listen 127.0.0.1:8000; // listen localhost:8000 监听指定的IP和端口
listen 127.0.0.1; 监听指定IP的所有端口
listen 8000; 监听指定端口上的连接
listen *:8000; 监听指定端口上的连接
```
- default_server属性是标识符，用来将此虚拟主机设置成默认主机。所谓的默认主机指的是如果没有匹配到对应的address:port，则会默认执行的。如果不指定默认使用的是第一个server。
  
2. server_name：用来设置虚拟主机服务名称。

|语法 |server_name name ...;name可以提供多个中间用空格分隔|
|---|---|
|默认值| server_name "";|
|位置| server|
- 如 server_name:www.itheima.cn 在浏览器输入此地址，就会让当前server处理(前提是这个地址确是我们的)

3. location指令用来设置请求的URI
     
|语法| location [ = or ~ or ~* or ^~ or@ ] uri{...}|  
|---|---|
|默认值| —|
|位置| server,location|
- 不带符号，要求必须以指定模式开始
```
server {
    listen 80;
    server_name 127.0.0.1;
    location /abc{
        default_type text/plain;
        return 200 "access success";
    }
}
以下访问都是正确的
http://localhost/abc
http://localhost/abc?p1=TOM
http://localhost/abc/
http://localhost/abcdef
```
- = : 用于不包含正则表达式的uri前，必须与指定的模式精确匹配
- ~ ： 用于表示当前uri中包含了正则表达式，并且区分大小写 ~*: 用于表示当前uri中包含了正则表达式，并且不区分大小写
- ^~: 用于不包含正则表达式的uri前，功能和不加符号的一致，唯一不同的是，如果模式匹配，那么就停止搜索其他模式了。
   
4. 设置请求资源的目录root / alias

    |语法| root path|
    |---|---|
    |默认值| root html|
    |位置 |http、server、location|

    alias：用来更改location的URI

    |语法| alias path;|
    |---|---|
    |默认值| —|
    |位置 |location|
- root的处理结果是: root路径+location路径alias的处理结果是:使用alias路径替换location路径
alias是一个目录别名的定义，root则是最上层目录的含义。
如果location路径是以/结尾,则alias也必须是以/结尾，root没有要
求

5. index指令设置网站的默认首页
6. error_page指令error_page指令
   
   |语法 |error_page code ... [=[response]] uri;|
   |---|---|
    |默认值 |-|
    |位置| http、server、location......|
- 可以指定具体跳转的地址
    ```
        server {
            error_page 404 http://www.itcast.cn;
        }
    ```
- 可以指定重定向地址
  ```
        server{
            error_page 404 /50x.html;
            error_page 500 502 503 504 /50x.html;
            location =/50x.html{
                root html;
            }
        }
  ```
- 使用location的@符合完成错误信息展示
  ```
    server{
        error_page 404 @jump_to_error;
        location @jump_to_error {
            default_type text/plain;
            return 404 'Not Found Page...';
        }
    }
  ```
- 可选项=[response]的作用是用来将相应代码更改为另外一个
  ```
    server{
        error_page 404 =200 /50x.html;
            location =/50x.html{
            root html;
        }
    }
    这样的话，当返回404找不到对应的资源的时候，在浏览器上可以看到，
    最终返回的状态码是200，这块需要注意下，编写error_page后面的内
    容，404后面需要加空格，200前面不能加空格
  ```
### 2.静态资源优化配置语法
1. sendfile，用来开启高效的文件传输模式。(前面已经讨论过)
2. tcp_nopush：该指令必须在sendfile打开的状态下才会生效，主要是用来提升网络包的传输'效率'
   
   |语法 |tcp_nopush on or off;|
   |---|---|
    |默认值 |tcp_nopush off;|
    |位置| http、server、location|
3. tcp_nodelay：该指令必须在keep-alive连接开启的情况下才生效，来提高网络包传输的'实时性'
   
   |语法| tcp_nodelay on or off;|
   |---|---|
    |默认值 |tcp_nodelay on;|
    |位置 |http、server、locati|
- 经过刚才的分析，"tcp_nopush"和”tcp_nodelay“看起来是"互斥的"，那么为什么要将这两个值都打开，这个大家需要知道的是在linux2.5.9以后的版本中两者是可以兼容的，三个指令都开启的好处是，sendfile可以开启高效的文件传输模式，tcp_nopush开启可以确保在发送到客户端之前数据包已经充分“填满”， 这大大减少了网络开销，并加快了文件发送的速度。 然后，当它到达最后一个可能因为没有“填满”而暂停的数据包时，Nginx会忽略tcp_nopush参数， 然后，tcp_nodelay强制套接字发送数据。由此可知，TCP_NOPUSH可以与TCP_NODELAY一起设置，它比单独配置TCP_NODELAY具有更强的性能。所以我们可以使用如下配置来优化Nginx静态资源的处理

### GZIP
   - 接下来讨论的指令都来自ngx_http_gzip_module模块，该模块会在nginx安装的时候内置到nginx的安装环境中，也就是说我们可以直接使用这些指令。
1.  gzip指令：该指令用于开启或者关闭gzip功能
   
    |语法 |gzip on or off;|
    |---|---|
    |默认值| gzip off;|
    |位置| http、server、location...|
2. gzip_types指令：该指令可以根据响应页的MIME类型选择性地开启Gzip压缩功能
3.  gzip_comp_level指令：该指令用于设置Gzip压缩程度，级别从1-9,1
表示要是程度最低，要是效率最高，9刚好相反，压缩程度最高，但是效率最低最费时间。
4. gzip_vary指令：该指令用于设置使用Gzip进行压缩发送是否携
带“Vary:Accept-Encoding”头域的响应头部。主要是告诉接收方，所发送的数据经过了Gzip压缩处理
5. gzip_buffers指令：该指令用于处理请求压缩的缓冲区数量和大小。

    |语法| gzip_buffers number size;|
    |---|---|
    |默认值| gzip_buffers 32 4k|16 8k;|
    |位置| http、server、location|
- 其中number:指定Nginx服务器向系统申请缓存空间个数，size指的是每个缓存空间的大小。主要实现的是申请number个每个大小为size的内存空间。这个值的设定一般会和服务器的操作系统有关，所以建议此项不设置，使用默认值即可。

6. gzip_disable指令：针对不同种类客户端发起的请求，可以选择性地开启和关闭Gzip功能。
7.  gzip_http_version指令：针对不同的HTTP协议版本，可以选择性地开启和关闭Gzip功能。
8.  gzip_min_length指令：该指令针对传输数据的大小，可以选择性地开启和关闭Gzip功能
   - Gzip压缩功能对大数据的压缩效果明显，但是如果要压缩的数据比较小的化，可能出现越压缩数据量越大的情况，因此我们需要根据响应内容的大小来决定是否使用Gzip功能，响应页面的大小可以通过头信息中的Content-Length来获取。但是如何使用了Chunk编码动态压缩，该指令将被忽略。建议设置为1K或以上。
9. gzip_proxied指令：该指令设置是否对服务端返回的结果进行Gzip压缩。
- 实例
```
gzip on; #开启gzip功能
gzip_types *; #压缩源文件类型,根据具体的访问资源类型设定
gzip_comp_level 6; #gzip压缩级别
gzip_min_length 1024; #进行压缩响应页面的最小长度,contentlength
gzip_buffers 4 16K; #缓存空间大小
gzip_http_version 1.1; #指定压缩响应所需要的最低HTTP请求版本
gzip_vary on; #往头信息中添加压缩标识
gzip_disable "MSIE [1-6]\."; #对IE6以下的版本都不进行压缩
gzip_proxied off； #nginx作为反向代理压缩服务端返回数据的条件
```
- 我们可以将这些内容抽取到一个配置文件中nginx_gzip.conf，然后通过include指令把配置文件再次加载到nginx.conf配置文件中，方法使用。
```
include nginx_gzip.conf
```
  
10. Gzip和sendfile共存问题  
    - 前面在讲解sendfile的时候，提到过，开启sendfile以后，在读取磁盘上的静态资源文件的时候，可以减少拷贝的次数，可以不经过用户进程将静态文件通过网络设备发送出去，但是Gzip要想对资源压缩，是需要经过用户进程进行操作的。所以如何解决两个设置的共存问题。可以使用ngx_http_gzip_static_module模块的gzip_static指令来解决
    - gzip_static指令  
    gzip_static: 检查与访问资源同名的.gz文件时，response中以gzip相关的header返回.gz文件的内容。
    - 通过 ./configure --with-http_gzip_static_module安装此模块，未安装的需重新安装，参考黑马nginx(2021-04-01)P55

```
cd /usr/local/nginx/html
gzip jquery.js
```
### 浏览器缓存
1. 浏览器缓存流程
![](img/截屏2024-03-23%2019.56.10.png)
（1）用户首次通过浏览器发送请求到服务端获取数据，客户端是没有对
应的缓存，所以需要发送request请求来获取数据；  
（2）服务端接收到请求后，获取服务端的数据及服务端缓存的允许后，返回200的成功状态码并且在响应头上附上对应资源以及缓存信息；  
（3）当用户再次访问相同资源的时候，客户端会在浏览器的缓存目录中查找是否存在响应的缓存文件  
（4）如果没有找到对应的缓存文件，则走(2)步  
（5）如果有缓存文件，接下来对缓存文件是否过期进行判断，过期的判断标准是(Expires),  
（6）如果没有过期，则直接从本地缓存中返回数据进行展示  
（7）如果Expires过期，接下来需要判断缓存文件是否发生过变化  
（8）判断的标准有两个，一个是ETag(Entity Tag),一个是Last-Modified
（9）判断结果是未发生变化，则服务端返回304，直接从缓存文件中获取数据  
（10）如果判断是发生了变化，重新从服务端获取数据，并根据缓存协商(服务端所设置的是否需要进行缓存数据的设置)来进行数据缓存。  

2. 在nginx上的配置  
  - expires:该指令用来控制页面缓存的作用。可以通过该指令控制HTTP应答中的“Expires"和”Cache-Control" 
  
    |语法| expires [modified] time / expires epoch or max or off;|
    |---|---|
    |默认值| expires off;|
    |位置| http、server、location|

    - time:可以整数也可以是负数，指定过期时间，如果是负数，Cache-Control则为no-cache,如果为整数或0，则Cache-Control的值为max-age=time;
    - epoch: 指定Expires的值为'1 January,1970,00:00:01 GMT'(1970-01-0100:00:00)，Cache-Control的值no-cache
    - max:指定Expires的值为'31 December2037 23:59:59GMT' (2037-12-31 23:59:59) ，Cache-Control的值为10年
    - off:默认不缓存。
![](img/截屏2024-03-23%2020.03.13.png)

3. add_header指令是用来添加指定的响应头和响应值。

### Nginx的跨域问题解决
1. 同源策略
   - 浏览器的同源策略：是一种约定，是浏览器最核心也是最基本的安全功能，如果浏览器少了同源策略，则浏览器的正常功能可能都会受到影响。同源: 协议、域名(IP)、端口相同即为同源
2. 跨域问题
   - 有两台服务器分别为A,B,如果从服务器A的页面发送异步请求到服务器B获取数据，如果服务器A和服务器B不满足同源策略，则就会出现跨域问题。
   
3. 解决方案
   - 使用add_header指令，该指令可以用来添加一些头信息
   - 此处用来解决跨域问题，需要添加两个头信息Access-Control-Allow-Origin , Access-Control-Allow-Methods
   - Access-Control-Allow-Origin: 直译过来是允许跨域访问的源地址信息，可以配置多个(多个用逗号分隔)，也可以使用*代表所有源
   - Access-Control-Allow-Methods:直译过来是允许跨域访问的请求方式，值可以为 GET POST PUT DELETE...,可以全部设置，也可以根据需要设置，多个用逗号分隔
![](img/截屏2024-03-23%2020.08.33.png)

### 静态资源防盗链
- 资源盗链指的是此内容不在自己服务器上，而是通过技术手段，绕过别人的限制将别人的内容放到自己页面上最终展示给用户。以此来盗取大网站的空间和流量。简而言之就是用别人的东西成就自己的网站。
- 当浏览器向web服务器发送请求的时候，一般都会带上Referer,来告诉浏览器该网页是从哪个页面链接过来的.后台服务器可以根据获取到的这个Referer信息来判断是否为自己信任的网站地址，如果是则放行继续访问，如果不是则可以返回403(服务端拒绝访问)的状态信息。
- valid_referers:nginx会通就过查看referer自动和valid_referers后面的内容进行匹配，如果匹配到了就将$ invalid_referer变量置0，如果没有匹配到，则将$invalid_referer变量置为1，匹配的过程中不区分大小写。
- 可以设置的值有
  - none: 如果Header中的Referer为空，允许访问
  - blocked:在Header中的Referer不为空，但是该值被防火墙或代理进行伪装过，如不带"http://" 、"https://"等协议头的资源允许访问。
  - server_names:指定具体的域名或者IP
  - string: 可以支持正则表达式和*的字符串。如果是正则表达式，需要以~开头表示，例如
---
## Cookie、Session、Token等技术
### Cookie
   - Cookie 是一种在客户端存储数据的技术，它是由服务器发送给客户端的小型文本文件，存储在客户端的浏览器中,大小限制大致在 4KB 左右。在客户端发送请求时，浏览器会自动将相应的 Cookie 信息发送给服务器，服务器通过读取 Cookie 信息，就可以判断该请求来自哪个客户端。Cookie 可以用于存储用户的登录状态、购物车信息等。
![](img/截屏2024-03-23%2020.25.29.png)
1. 主要属性
   
    |属性名|描述|
    |---|---|
    |name|	cookie 的名称|
    |value|	cookie 的值|
    |comment|	cookie 的描述信息|
    |domain	|可以访问该 cookie 的域名|
    |expires|	cookie 的过期时间，具体某一时间|
    |maxAge|	cookie 的过期时间|
    |path	|cookie 的使用路径|
    |secure	|cookie 是否使用安全协议传输，比如 SSL 等|
    |version|	cookie 使用的版本号|
    |isHttpOnly|	指定该 Cookie 无法通过 JavaScript 脚本拿到，比如 Document.cookie 属性、XMLHttpRequest 对象和 Request API 都拿不到该属性。这样就防止了该 Cookie 被脚本读到，只有浏览器发出 HTTP 请求时，才会带上该 Cookie。|
2. Cookie的访问流程
   ![](img/截屏2024-03-23%2020.28.32.png)
3. 特点
   - cookie 存储在客户端
   - cookie 不可跨域，但是在如果设置了 domain，那么它们是可以在一级域名和二级域名之间共享的。

### session
- session 由服务端创建，当一个请求发送到服务端时，服务器会检索该请求里面有没有包含 sessionId 标识，如果包含了 sessionId，则代表服务端已经和客户端创建过 session，然后就通过这个 sessionId 去查找真正的 session，如果没找到，则为客户端创建一个新的 session，并生成一个新的 sessionId 与 session 对应，然后在响应的时候将 sessionId 给客户端，通常是存储在 cookie 中。如果在请求中找到了真正的 session，验证通过，正常处理该请求。

- 每一个客户端与服务端连接，服务端都会为该客户端创建一个 session，并将 session 的唯一标识 sessionId 通过设置 Set-Cookie 头的方式响应给客户端，客户端将 sessionId 存到 cookie 中。
  
1. 请求流程
![](img/截屏2024-03-23%2020.31.57.png)

### Token
- Token 是一种在客户端和服务端之间传递身份信息的方式。当用户登录成功后，服务端会生成一个 Token，将其发送给客户端。客户端在后续的请求中，需要将 Token 携带在请求头或请求参数中。服务端通过验证 Token 的合法性，就可以确定该请求来自哪个用户，并且可以根据用户的权限进行相应的操作。Token 可以有效地避免了 Cookie 的一些安全问题，比如 CSRF 攻击。

1. 组成
   - 标头（Header）：包含了算法和类型，用于指定如何对有效载荷进行编码和签名。常用的算法有HMAC、RSA、SHA等。
    - 有效载荷（Payload）：包含了一些信息，如用户ID、角色、权限等，用于验证身份和授权。有效载荷可以是加密的，也可以是明文的。
    - 签名（Signature）：是对标头和有效载荷进行签名后得到的值，用于验证token的完整性和真实性。签名通常使用私钥进行签名，并使用公钥进行验证。
    - 一个完整的token包含了标头、有效载荷和签名三个部分，它们一起构成了一个安全的令牌，用于进行身份验证和授权。
2. 请求流程
    ![](img/截屏2024-03-23%2020.38.59.png)


### 对比

cookie、session、token三者最终的目的都是一样：鉴权和认证，下面使用表格的方式直观的描述下三者的优缺点。

|方式|	特点|	优点|	缺点|
|---|---|---|---|
|cookie|	1.存储在客户端。2.请求自动携带 cookie。3.存储大小 4KB。	|1.兼容性好，因为是比较老的技术。2.很容易实现，因为 cookie 会自动携带和存储。	|1.需要单独解决跨域携带问题，比如多台服务器如何共享 cookie。2.会遭受 CSRF 攻击。3.存储在客户端，不够安全。|
|session|	1.存储在服务端。2.存储大小无限制。|	1.查询速度快，因为是个会话，相当于是在内存中操作。2.结合 cookie 后很容易实现鉴权。3.安全，因为存储在服务端。|	1.耗费服务器资源，因为每个客户端都会创建 session。2.占据存储空间，session 相当于存储了一个完整的用户信息。|
|token	|1.体积很小。2.自由操作存储在哪里。|	1.安全，因为 token 一般只有用户 id，就算被截取了也没什么用。2.无需消耗服务器内存资源，它相当于只存了用户 id，session 相当于存储了用户的所有信息。3.跨域处理较为方便，比如多台服务器之间可以共用一个 token。|	1.查询速度慢，因为 token 只存了用户 id，每次需要去查询数据库。|

- 需要注意 我们常说的JWT也是Token的一种
- 需要注意 Token的自动续期(网页的无感刷新问题)，一种解决办法是使用redis

- 讨论如何实现Cookie跨域问题

    - Access-Control-Allow-Credentials: true（将“允许跨域请求携带认证信息”的值设为true）
    - Access-Control-Allow-Origin: 请求域名（配置允许访问的域名）
    - 客户端也要设置 withCredentials 使其允许 Cookie 共享


## Rewrite功能
- Rewrite是Nginx服务器提供的一个重要基本功能，是Web服务器产品中
几乎必备的功能。主要的作用是用来实现URL的重写。
1. set指令 该指令用来设置一个新的变量。    
  
|语法| set $variable value;|
|---|---|
|默认值| —|
|位置| server、location、if|  

variable:变量的名称，该变量名称要用"$"作为变量的第一个字符，且不
能与Nginx服务器预设的全局变量同名。
value:变量的值，可以是字符串、其他变量或者变量的组合等。

Rewrite常用全局变量

|变量| 说明|
|---|---|
|$args|变量中存放了请求URL中的请求指令。比如http://192.168.200.133:8080?arg1=value1&args2=value2中的"arg1=value1&arg2=value2"，功能和$query_string一样|
|$http_user_agent|变量存储的是用户访问服务的代理信息(如果通过浏览器访问，记录的是浏览器的相关版本信息)|
|$document_uri|变量存储的是当前访问地址的URI。比如http://192.168.200.133/server?id=10&name=zhangsan中的"/server"，功能和$uri一样|
|$request_uri|变量中存储了当前请求的URI，并且携带请求参数，比如http://192.168.200.133/server?id=10&name=zhangsan中的"/server?id=10&name=zhangsan"|

<!-- 2. if指令    
***参考黑马的pdf***

##  -->




