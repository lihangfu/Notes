## 1、基本概念

### 1.1 Nginx简介

> Nginx是一个高性能的HTTP和反向代理服务器，特点是占有内存少，并发能力强。

> Nginx专为性能优化而开发，性能是其最重要的考量，实现上非常注重效率，能经受高负载的考验，有报告表明能支持高达50000个并发连接数。



### 1.2 反向代理



#### 1.2.1 正向代理

* 在客户端（浏览器）配置代理服务器，通过代理服务器访问互联网。

#### 1.2.2 反向代理

* 反向代理，其实客户端对代理是无感知的，因为客户端不需要配置就可以访问，我们只需要将请求发送到反向代理服务器，由反向代理服务器去选择目标服务器获取数据后，在返回给客户端，此时反向代理服务器和目标服务器对外就是一个服务器，暴露的事代理服务器地址，隐藏了真实服务器IP地址。

### 1.3 负载均衡

单个服务器解决不了，我们增加服务器的数量，然后将请求分发到各个服务器上，将原先请求集中到单个服务器上的情况改为将请求分发到多个服务器上，将负载分发到不同的服务器，也就是我们所说的负载均衡。



### 1.4 动静分离

为了加快网站的解析速度，可以把动态页面和静态页面由不同的服务器来解析，加快解析速度。降低原来单个服务器的压力。



## 2、基础操作

### 2.1 常用命令

```bash
# 1、查看版本号
./nginx -v

# 2、启动nginx
./nginx
./nginx -c /usr/local/nginx/conf/nginx.conf	# 以指定的配置文件启动

# 3、关闭nginx
./nginx -s quit		# 正常退出
./nginx -s stop		# 强制退出

# 4、重新加载nginx,平滑加载配置文件
./nginx -s reload

# 5、检查配置文件修改是否正确
./nginx -t -c /etc/nginx/nginx.conf
./nginx -t
```



### 2.2 配置文件

#### 2.2.1 配置文件的组成

```bash
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



* **全局块**

  配置影响nginx全局的指令。一般有运行nginx服务器的用户组，nginx进程pid存放路径，日志存放路径，配置文件引入，允许生成worker process数等。 

*  **events块** 

  配置影响nginx服务器或与用户的网络连接。有每个进程的最大连接数，选取哪种事件驱动模型处理连接请求，是否允许同时接受多个网路连接，开启多个网络连接序列化等。

*  **http块** 

  可以嵌套多个server，配置代理，缓存，日志定义等绝大多数功能和第三方模块的配置。如文件引入，mime-type定义，日志自定义，是否使用sendfile传输文件，连接超时时间，单连接请求数等。 

  *  **server块** 

     配置虚拟主机的相关参数，一个http中可以有多个server。

  *  **location块** 

     配置请求的路由，以及各种页面的处理情况。 



#### 2.2.2 配置文件详解

```bash
########### 每个指令必须有分号结束。#################
#user administrator administrators;  #配置用户或者组，默认为nobody nobody。
#worker_processes 2;  #允许生成的进程数，默认为1
#pid /nginx/pid/nginx.pid;   #指定nginx进程运行文件存放地址
error_log log/error.log debug;  #制定日志路径，级别。这个设置可以放入全局块，http块，server块，级别以此为：debug|info|notice|warn|error|crit|alert|emerg
events {
    accept_mutex on;   #设置网路连接序列化，防止惊群现象发生，默认为on
    multi_accept on;  #设置一个进程是否同时接受多个网络连接，默认为off
    #use epoll;      #事件驱动模型，select|poll|kqueue|epoll|resig|/dev/poll|eventport
    worker_connections  1024;    #最大连接数，默认为512
}
http {
    include       mime.types;   #文件扩展名与文件类型映射表
    default_type  application/octet-stream; #默认文件类型，默认为text/plain
    #access_log off; #取消服务日志    
    log_format myFormat '$remote_addr–$remote_user [$time_local] $request $status $body_bytes_sent $http_referer $http_user_agent $http_x_forwarded_for'; #自定义格式
    access_log log/access.log myFormat;  #combined为日志格式的默认值
    sendfile on;   #允许sendfile方式传输文件，默认为off，可以在http块，server块，location块。
    sendfile_max_chunk 100k;  #每个进程每次调用传输数量不能大于设定的值，默认为0，即不设上限。
    keepalive_timeout 65;  #连接超时时间，默认为75s，可以在http，server，location块。

    upstream mysvr {   
      server 127.0.0.1:7878;
      server 192.168.10.121:3333 backup;  #热备
    }
    error_page 404 https://www.baidu.com; #错误页
    server {
        keepalive_requests 120; #单连接请求上限次数。
        listen       4545;   #监听端口
        server_name  127.0.0.1;   #监听地址       
        location  ~*^.+$ {       #请求的url过滤，正则匹配，~为区分大小写，~*为不区分大小写。
           #root path;  #根目录
           #index vv.txt;  #设置默认页
           proxy_pass  http://mysvr;  #请求转向mysvr 定义的服务器列表
           deny 127.0.0.1;  #拒绝的ip
           allow 172.18.5.54; #允许的ip           
        } 
    }
}
```



**上面是nginx的基本配置，需要注意的有以下几点：**

**1、几个常见配置项：**

- 1.$remote_addr 与 $http_x_forwarded_for 用以记录客户端的ip地址；
- 2.$remote_user ：用来记录客户端用户名称；
- 3.$time_local ： 用来记录访问时间与时区；
- 4.$request ： 用来记录请求的url与http协议；
- 5.$status ： 用来记录请求状态；成功是200；
- 6.$body_bytes_s ent ：记录发送给客户端文件主体内容大小；
- 7.$http_referer ：用来记录从那个页面链接访问过来的；
- 8.$http_user_agent ：记录客户端浏览器的相关信息；

**2、惊群现象：一个网路连接到来，多个睡眠的进程被同时叫醒，但只有一个进程能获得链接，这样会影响系统性能。**

**3、每个指令必须有分号结束。**



## 3、反向代理

```
server {
    	listen		80;					// 监听的端口
    	server_name www.123.com;		// 指定主机名

    	location / {					// 匹配的路径
    		root html;
    		proxy_pass http://10.180.60.134:8080;// 反向代理到哪
    		index	index.html index.html;
    	}
}

或者
server {
    	listen		80;					// 监听的端口
    	server_name www.123.com;		// 指定主机名

    	location ~/edu/ {					// 匹配的路径
    		root html;
    		proxy_pass http://10.180.60.134:8080;// 反向代理到哪
    		index	index.html index.html;
    	}
    	location ~/vod/ {					// 匹配的路径
    		root html;
    		proxy_pass http://10.180.60.134:8080;// 反向代理到哪
    		index	index.html index.html;
    	}
}
```



**匹配示例**

```
location [ = | ~ | ~*| ^~] uri {					// 匹配的路径

}
```

* =：用于不含正则表达式的 uri 前，要求请求字符串与 uri 严格匹配，如果匹配成功，就停止继续向下搜索并立即处理该请求。
* ~：用于表示 uri 包含正则表达式，并且区分大小写。
* ~*：用于表示 uri 包含正则表达式，并且不区分大小写。
* ^~：用于不含正则表达式的 uri 前，要求 Nginx 服务器找到标识 uri 和请 4求字符串匹配度最高的 location 后，立即使用此 location 处理请求，而不再使用 location 块中的正则 uri 和请求字符串做匹配。
* 注意：如果 uri 包含正则表达式，则必须要有 ~ 或者 ~* 表示。



## 4、负载均衡

```
upstream mysvr {   						// 定义上流服务
		server 10.180.60.134:8080;
		server 10.180.60.134:8081;
}
server {
    	listen		80;
    	server_name www.123.com;
    	location ~/text/ {
    		root html;
    		proxy_pass http://mysvr;	// 转发到上流服务
    		index	index.html index.html;
    	}
}
```



**负载均衡策略：**

* 轮询（默认，无需任何配置）：请求按请求时间顺序逐一分配到不同的后端服务，如果后的服务 down 掉，可以自动剔除。
* weight：权重，默认为1，权重越高被分配的客户端就越多。

```
// 指定沦陷几率，weight 和访问比率成正比，用户后端服务性能不均的情况。
upstream mysvr {   						// 定义上流服务
		server 10.180.60.134:8080 weight=10;
		server 10.180.60.134:8081 weight=10;
}
```

* ip_hash：每个请求按访问 ip 的 hash 结果分配，这样每个客户端固定访问一个后端服务，可以解决 session 的问题。

```
upstream mysvr {   						// 定义上流服务
		ip_hash
		server 10.180.60.134:8080;
		server 10.180.60.134:8081;
}
```

* fair（第三方）：按后端服务器的响应时间来分配请求，响应时间短的优先分配。

```
upstream mysvr {   
		server 10.180.60.134:8080;
		server 10.180.60.134:8081;  
		fair;						# 有的版本不支持
}
```



## 5、动静分离

>Nginx 动静分离鉴定来说就是把动态跟静态请求分开，不能理解成只是单纯的把动态页面和静态页面物理分开，严格意义上说应该是动态请求跟静态请求分开，可以理解成使用 Nginx 处理静态页面，Tomcat 处理动态页面。动静分离从目前实现角度来讲大致分为两种。

1. 一种是纯粹把静态文件独立成单独的域名，放在独立的服务器上，也就是目前主流推崇的方案；
2. 另外一种就是动态跟静态文件混合在一起发布，通过 nginx 来分开。



通过 location 指定不同后缀名实现不同的请求和转发。通过 expires 参数设置，可以使浏览器缓存过期时间，减少与服务器之间的请求和流量。

具体 expires定义：是给一个资源设定一个过期时间，也就是说无需去服务端验证，直接通过浏览器自身确认是否过期即可，所以不会产生额外的流量。此种方法非常适合不经常变动的资源。（如果经常更新的文件，不建议使用 expires 来缓存），如果设置为 3d，表示在这三天之内访问这个 URL，发生一个请求，比对服务器该文件最后更新时间没有变化，则不会从服务器抓取，返回状态码304，如果有修改，则直接从服务器重新下载，返回状态码 200。



**访问静态配置**

```
server {
	listen	80;
	server_name	192.168.17.129;
	
	location /www/ {					# 匹配路径
		root	/data/;					# 静态资源路径
		index	index.html	index.htm;	
	}
	
	location /image/ {
		root	/data/;
		autoindex	on;					# 访问文件时，列出文件夹内容
	}
}
```

