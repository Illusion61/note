# HTTP原理

## HTTP响应状态码

#### 含义

HTTP状态码是服务器用于向客户端提供有关请求状态的信息

#### 常见状态码

- 1xx信息性状态码:

```http
HTTP/1.1 100 Continue：请求已成功接收，客户端应继续发送请求的其余部分。
```

- 2xx成功状态码:

```http
HTTP/1.1 200 OK :请求成功，服务器成功返回请求的数据。
```

```http
HTTP/1.1 204 No Content：请求成功，但服务器没有返回任何内容
```

- 3xx重定向状态码:

```http
HTTP/1.1 301 Moved Permanently：原来的网页不提供服务,进行重定向到新的位置
```

```http
HTTP/1.1 304 Not Modified：客户端可以使用缓存的版本，无需服务器返回资源。
```

- **4xx客户端错误**:

```http
HTTP/1.1 400 Bad Request：服务器无法理解客户端的请求。(请求格式错误)
```
```http
HTTP/1.1 401 Unauthorized：请求要求身份验证。
```
```http
HTTP/1.1 404 Not Found：请求的资源在服务器上未找到。
```

- **5xx服务器错误**:

```http
HTTP/1.1 500 Internal Server Error：服务器遇到了一个未预期的错误。
```

```http
HTTP/1.1 503 Service Unavailable：服务器当前无法处理请求，可能是由于过载或维护。
```

## 请求方式

- GET:用来从服务器获取信息
- POST:在请求体中传递参数
- HEADER:只获取请求头,用来对服务器作健康检测

```bash
curl -I [url] #只获取请求头信息
```

## 缓存策略

- ctrl+F5:强制刷新无视缓存
- 缓存过期,才会向服务器发送请求验证资源
- 验证资源的过程,服务器比较客户端请求头中的两个值:
  - **If-Modified-Since**:该字段包括了浏览器之前返回的**Last-Modified**字段的值,如果这个**值没有发生改变**说明数据**没有被修改过**
  - **If-Not-Match**:该字段包括了浏览器之前返回的**E-tag**字段的值,如果这个值与服务器中的E-tag**匹配**说明数据**没有被修改过**
- 两种情况
  - 数据被修改:200OK,响应体中重新传输数据
  - 数据没有被修改:304NotModified,响应体中没有数据
- Cache-Control

- Expires

## HTTP版本

#### HTTP0.9

- **只支持GET**请求方式,不支持其他的请求方式
- **没有请求头/响应头**,只有返回的HTTP信息
- 服务器响应之后,**立即关闭TCP连接**
  - 三次握手->HTML->四次挥手->三次握手->PNG1->四次挥手->三次握手->PNG2->四次挥手

#### HTTP1.0

- 支持GET,POST,DELETE,HEADER等**多种方式**
- **新增了请求头/响应头**的概念,可以返回一些元信息比如协议版本号,内容类型等
- **扩充**了传输文件的**格式**,例如图片,视频,音频,二进制等都可以传输
- 服务器响应之后,**立即关闭TCP连接**

#### HTTP1.1

- 服务器响应默认**保持TCP连接**(长连接)
  - 三次握手->HTML->PNG1->PNG2->四次挥手
  - 四次挥手在**所有信息都传输完成的时候**进行,而不是在用户关闭网页的时候
- 支持**管道化**,流水线式发送请求再接收请求,而不是等到上一个请求收到之后再发送下一个请求
  - 管道化:请求->请求->请求--->接收->接收->接收
  - 非管道化:请求--->接收->请求--->接收->请求--->接收
  - HTTP1.1的管道化:限制发送和接收的顺序要一致,如果先发送的请求响应没有收到,后面的响应就会暂时被缓存**而不进行渲染(队头阻塞)**
- 缓存资源:在请求资源的时候先查看本地是否有缓存的资源
  - 如果**本地有之前缓存**的资源且**没有过期**,**不会发送请求**
  - 如果本地缓存过期,浏览器会向服务器中发送验证请求包含If-Modified-Since和If-Not-Match比较是否重传数据
  - 刷新网页,也会重新发送验证请求
- 断点传输:上传/下载资源的时候如果资源过大,就会分段传输;这样传输中断之后也可以继续传输剩下的部分
- 单个TCP连接上**只能同时发送一个请求和响应**,而浏览器一般会**建立多个TCP连接**(对于单个域名最多6-8个)来**并行传输数据**
- 请求头比较冗长,尤其是Cookie每个请求都要发送一次长度又很长

#### HTTP2:目标是在用户和网站之间只建立一个连接

- 二进制分帧:
  - 将请求头,请求体,响应头,响应体划分为不同的数据帧进行传输;其中头信息不会再细分,数据部分会继续细分成不同的帧
  - **并行传输为多路复用提供基础**
  - 和断点续传的区别是:
    - **断点续传**关注于在传输中断之后**继续传输**(数据包之间没有编号逐个传输)
    - 而**二进制分帧**是数据帧有**编号**,同时传输而不是逐个传输**,通过并行传输增大速度**
- **多路复用**:将所有的请求和响应都在同一个TCP连接中进行
  - 减少建立/销毁TCP(TLS)的开销
  - 新建的TCP会有慢启动(开始速度很慢之后才会慢慢提升速度),保持一个TCP避免了这种问题
  - 多路复用分级:
    - 数据**帧**:最小的单位,属于一个流
    - 数据**流**:单个请求传输流会分成不同的帧
    - 连接:一个TCP连接中有很多个不同的数据流
- 现行的浏览器一般强制要求HTTP2协议使用TLS加密(https),为了保证数据的加密安全性
- 队头阻塞问题依然存在
- 建立TCP/TLS连接需要延迟
- 多路复用导致服务器压力上升,并且每个连接的平均资源被稀释容易Timeout

#### HTTP3:底层使用UDP,将可靠性保证交给应用层解决

- 

## MIME协议

- MIME协议规定了文件的类型,确定了浏览器打开文件的格式
  - text/*:文本文件
  - video/*:作为视频打开
  - image/*:作为图片打开
  - audio/*:作为音频打开
  - 其他文件类型:直接下载
- 常见的MIME类型

```nginx
types {
		text/html                             html htm shtml;
		text/css                              css;
		text/xml                              xml;
    
		image/gif                             gif;
		image/jpeg                            jpeg jpg;
		image/png                             png;
		image/webp                            webp;
		image/x-icon                          ico;
		image/x-ms-bmp                        bmp;
    
		application/javascript                js;
		application/json                      json;
		application/pdf                       pdf;
    
		video/mpeg                            mpeg mpg;
		video/mp4                             mp4;
		video/webm                            webm;
    
		audio/ogg                             ogg;
		audio/mpeg                            mp3;
}
```

# HTTP请求

```http
POST /api/user/code?phone=13581955198 HTTP/1.1
Host: m.sciform.one
User-Agent: PostmanRuntime/7.33.0
Content-Length: 62
Accept: */*
Accept-Encoding: gzip, deflate, br
Content-Type: application/json
Postman-Token: eb0b73c6-ad29-4972-bd7c-b58433cfb4d6

{
    "name": "Harry",
    "age": 18,
    "score": "117"
}
```

## 第一部分:请求起始行

```http
POST /api/user/code?phone=13581955198&name=harry HTTP/1.1
```

- 请求方法:GET/POST/...
- 请求路径uri(包含请求头**参数**):uri有长度限制,因此没法传递太多的参数信息
- 请求的具体协议

## 第二部分:请求头(参数)

参考https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/

- 请求头部是由浏览器或者客户端**自动生成**的
- 请求参数只是客户端**主观希望**的,但服务器不一定会按照请求参数的要求返回

```http
Host: sciform.one
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:109.0) Gecko/20100101 Firefox/118.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
DNT: 1
Connection: keep-alive
Referer: http://sciform.one/
```

- Host:服务器
- User-Agent:用户客户端类型
- Accept:接收什么类型的内容(无图模式就不接收image/*的类型)
- Accept-Language:用户接收语言的优先级
- Accept-Encoding:客户端可以接受的压缩方法,如果服务端进行压缩需要在content-encoding中提及
- DNT(Do-Not-Track):追踪,DNT为0代表愿意网站追踪信息,为0代表不愿意;但是DNT没有法律强制力很多网站会忽略
- Connection:表示客户端希望每次HTTP请求是否新建TCP连接(但具体行为由服务器控制)
  - keep-alive:客户端希望保持TCP连接,这样传输很多文件的时候就不用重新建立TCP连接
  - close:客户端希望关闭TCP连接,每次请求文件单独建立TCP连接
- Cookie:一段加密的字符串用来表示用户身份
- Referer:发送这个请求的网页url,用来做防盗链或者日志统计

## 第三部分:请求空行

用来划分请求参数和请求体

## 第四部分:请求主体

- 一般没有请求主体
- post默认参数会放在请求体request-body中

# HTTP响应

```http
HTTP/1.1 200 OK
Date: Mon, 16 Oct 2023 05:49:40 GMT
Server: Apache/2.4.52 (Ubuntu)
Last-Modified: Wed, 29 Mar 2023 11:56:14 GMT
ETag: "f8a9-5f808a9b67530"
Accept-Ranges: bytes
Content-Length: 63657
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive
Content-Type: image/png

.PNG
.
...
IHDR...^............u.. .IDATx^.}..\U....}7..7. =!	!.F. U.. J.....S...SD.( 
"...H/....B.!	..g....3.s~....@.F.?...L{..}..{.........X...(.@...............5P..b
......
```

## 第一部分:响应起始行

```http
HTTP/1.1 200 OK
```

- 协议:使用的协议
- 状态码
- 状态码的解释

## 第二部分:响应头(参数)

```http
HTTP/1.1 200 OK
Connection: close
Content-Length: 10671
Accept-Ranges: bytes
Content-Type: text/html; charset=utf-8
Date: Mon, 16 Oct 2023 07:22:43 GMT
Etag: "6514e987-29af"
Last-Modified: Thu, 28 Sep 2023 02:48:39 GMT
Server: nginx/1.18.0 (Ubuntu)
```

- Connection:表示服务器决定是保持TCP连接还是关闭TCP连接
- Content-Length:内容长度,帮助客户端提升性能
- Content-Type:相应内容的类型,遵循MIME协议
- Date:服务器生成响应的日期和时间
- Etag:服务器使用哈希算法生成的资源唯一标识符,用来判断资源是否被修改
- Last-Modified:上一次修改请求的内容是在什么时候
- Server:服务器的版本

## 第三部分:响应空行

响应空行用来区分响应头和响应主体

## 第四部分:响应体

- 响应的内容主体

- 对于某些特定的响应体格式,浏览器会自动选择Content-Type,比如mp4就会是video/mp4
- 响应体比较长,就会分成很多个包进行发送

# 常见的软件配置

## apache2安装配置

#### 安装/禁用模块

- a2enmod...
- a2dismod...

#### 配置(子)网站

- 配置HTTP

```xml
<VirtualHost *:80>
    ServerName m.sciform.one
    DocumentRoot /var/www/mobile/hmdp
    <Directory /var/www/mobile/hmdp>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
        # ProxyPass / http://localhost:8080/
    	# ProxyPassReverse / http://localhost:8080/
</VirtualHost>
```

- 配置HTTPS

```xml
<IfModule mod_ssl.c>
	<VirtualHost _default_:443>
		ServerName m.sciform.one
		DocumentRoot /var/www/mobile/hmdp
		ErrorLog ${APACHE_LOG_DIR}/error.log
		CustomLog ${APACHE_LOG_DIR}/access.log combined
		# 添加以下几行配置，用于启用SSL
		SSLEngine on
		SSLCertificateFile      /etc/letsencrypt/live/sciform.one/fullchain.pem
		SSLCertificateKeyFile /etc/letsencrypt/live/sciform.one/privkey.pem

		<Directory /var/www/mobile/hmdp>
			Options Indexes FollowSymLinks
			AllowOverride All
			Require all granted
		</Directory>
		# ProxyPass / http://localhost:8080/
		# ProxyPassReverse / http://localhost:8080/
	</VirtualHost>
</IfModule>
```

## nginx安装配置

- 默认网页目录:/usr/share/nginx/html

#### 配置文件

- nginx中location会找**第一个匹配的location**:location位置越精确,应该越靠上

- location:

  ```conf
  root：指定该location下的资源根目录(会自动添加path)
  alias: 指定该location下的资源根目录(不会自动添加path)
  proxy_pass: 将请求代理到定的后端服务器。
  rewrite: 重写请求的url
  return: 返回指定的状态码和信息
  ```

```nginx
/etc/nginx/nginx.conf
worker_processes 1; #启动1个nginx进程
events {
	worker_connections 1024; #每个进程可以处理1024个连接
}
http { #http模块
	include /etc/nginx/mime.types; #支持哪些多媒体格式
	default_type application/octet-stream; #设置默认格式
	sendfile on;
	keepalive_timeout 65;
	charset utf-8; #设置字符集
    include /etc/nginx/conf.d/*.conf; #导入所有的网站配置文件
}
```

```nginx
/etc/nginx/conf.d/localhost.conf
server{ #一个网站
    listen 0.0.0.0:80 default_server; #监听端口80,default_server代表域名不匹配的时候默认的服务器
    server_name localhost; #网站的域名
    location /blog { #!!location位置越精确,应该越靠上!!
        #root html; #站点根目录/usr/share/nginx/html
        alias /var/www/blog;
        index index.html index.htm; #默认首页,从左往右找
    }
    location / {
        #root html; #站点根目录/usr/share/nginx/html
        root /var/www/html;
        index index.html index.htm; #默认首页,从左往右找
    }
}
```

```nginx
/etc/nginx/mime.types
types {
		text/html                             html htm shtml;
		text/css                              css;
		text/xml                              xml;
		image/gif                             gif;
		image/jpeg                            jpeg jpg;
		image/png                             png;
		image/webp                            webp;
		image/x-icon                          ico;
		image/x-ms-bmp                        bmp;
		application/javascript                js;
		application/json                      json;
		application/pdf                       pdf;
		video/mpeg                            mpeg mpg;
		video/mp4                             mp4;
		video/webm                            webm;
		audio/ogg                             ogg;
		audio/mpeg                            mp3;
}
```

#### web服务器开启日志

- 网络安全法规定日志至少不低于6个月

- 日志路径:

```shell
---> /var/log/nginx:
--->access.log:访问日志
--->error.log:错误日志
```

- 自定义日志格式:
```nginx
log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"'; #定义main的格式


access_log /var/log/nginx/https_access.log main;#日志文件路径+使用什么格式的日志
```

- 关闭访问特定文件/目录下的日志

```nginx
location = /favicon\.ico {
    access_log off;
}
location ~ .*\.(gif|jpg|jpeg|bmp|png|ico|txt|mp3|mp4|swf)$ {
    expires 365d;
}
```



#### 配置https证书

```nginx
/etc/nginx/conf.d/manga.sciform.one.conf
server{ #一个网站
    listen 0.0.0.0:443 ssl;
    server_name manga.sciform.one; #网站的域名
    default_type 'text/html';
    charset utf-8;
    access_log /var/log/nginx/https_access.log main;
    
    #ssl证书文件路径
    ssl_certificate     /var/www/credentials/manga.sciform.one/fullchain.pem;
    ssl_certificate_key  /var/www/credentials/manga.sciform.one/privkey.pem;
    # ssl验证相关配置
    ssl_session_timeout  5m;    #缓存有效期(分钟),有效期内进行会话不用重新建立TLS连接
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;    #加密算法
    ssl_protocols TLSv1.2 TLSv1.3;    #安全链接可选的加密协议
    ssl_prefer_server_ciphers on;   #使用服务器端的首选算法,而不是客户端请求的算法
    
    location / {
        root /var/www/manga;
        index index.html index.htm; #默认首页,从左往右找
    }
}
```

```nginx
/etc/nginx/conf.d/unsecure_manga.sciform.one.conf
#HTTP重定向
server {
    	access_log off;#关闭日志
        listen 0.0.0.0:80;
        server_name manga.sciform.one;
    	#location / {
        return 301 https://manga.sciform.one$request_uri;
    	#}
}
```

#### 配置使用HTTP2

- 必须使用ssl/tls加密,否则会502badGateway

```nginx
server {
    listen 0.0.0.0:443 ssl http2;
    server_name blog.sciform.one;
    #ssl证书文件路径
    ssl_certificate     /var/www/credentials/manga.sciform.one/fullchain.pem;
    ssl_certificate_key  /var/www/credentials/manga.sciform.one/privkey.pem;
    # ssl验证相关配置
    ssl_session_timeout  5m;    #缓存有效期(分钟),有效期内进行会话不用重新建立TLS连接
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;    #加密算法
    ssl_protocols TLSv1.2 TLSv1.3;    #安全链接可选的加密协议
    ssl_prefer_server_ciphers on;   #使用服务器端的首选算法,而不是客户端请求的算法
    location / {
        root /var/www/blog;
    }
}
```

#### 开启gzip压缩

- 节省流量,降低服务器带宽占用
- 提升网站访问速度
- 提升CPU性能的消耗
- **gzip基本只对文字类的内容有用,对于图片和视频几乎没用**

```nginx
location / {
    gzip on; #开启
    gzip_min_length 1k; #超过1kb的文件才会进行压缩
    gzip_buffers 4 32m; #gzip缓冲区共有4个每个32mB,如果文件大小超过缓冲区,就无法按照gzip压缩
    gzip_http_version 2; #http版本
    gzip_comp_level 9; #压缩等级1-9,9最高
    gzip_types */*; #压缩类型设置
    gzip_vary on; #响应头中添加Content-Encoding:gzip
    gzip_disable "MSIE [1-7]\."; #遇到IE浏览器1-7取消gzip压缩(不支持)
    root /var/www/manga;
}
```

#### 开启自动目录

```nginx
server {
    listen 80 default_server;
    server_name localhost;
    autoindex on; #开启自动目录
    autoindex_exact_size off; #显示k,m而不是精确数值b
    location / {
        root /var/www/html;
        index index.html index.htm;
    }
}
```

#### 访问控制

- 传输层控制:防火墙控制访问
- 应用层控制:nginx白名单黑名单,可以针对网站禁止用户
  - deny和allow先配置的先生效
  - deny:限制哪些ip访问
  - allow:允许哪些ip无视deny访问

```nginx
server{
    ...;
    #黑名单模式:先拒绝,再允许所有
    deny 107.173.181.210;
    allow 0.0.0.0;
    
    #或者白名单模式:先允许,再拒绝所有
    allow 107.173.181.210;
    allow 192.168.101.0/24;
    deny 0.0.0.0;
    ...;
}
```

- 403-Forbidden的三种情况
  - 没有index.html
  - 应用层上被nginx给deny了
  - **nginx用户对于目录没有rx权限或者对文件没有r权限(systemctl)里面可以指定路径**

##

# 分布式网络请求

## 负载均衡(nginx实现)

#### 负载均衡的好处

- 可以减轻**单个服务器的流量负担**(API-Gateway的带宽需求依然很高)
- 可以**分配CPU/内存**负担
- 支持快速的**可扩展性**

#### nginx实现反向代理

```nginx
location / {
    proxy_pass http://sciform.one; #反向代理
    proxy_set_header X-Real-IP $remote_addr;#使用客户端IP地址而不是代理服务器IP地址
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
}
```

```nginx
#反向代理支持https参考网址 https://docs.nginx.com/nginx/admin-guide/security-controls/securing-http-traffic-upstream/
location / {
    
}
```

- 轮询模式:

  - 普通轮询方式:轮流分配负载

  ```nginx
  
  ```

  

  - 权重轮询方式:根据服务器的权重进行分发

- 最少连接

- ip_hash



```mongo
db.createUser( { user: "nodebb", pwd: "6d2ySW2py", roles: [ "root" ] } )
db.createUser( { user: "nodebb", pwd: "6d2ySW2py", roles: [ "readWrite" ] } )
mongodb://nodebb:6d2ySW2py@127.0.0.1:27017
```

