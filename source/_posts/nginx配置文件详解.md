title: nginx配置文件详解
date: 2015-4-7 14:17:59
tags: Nginx Linux
---
中文文档:http://wiki.nginx.org/NginxChs

```
#用户 用户组
user www www;
#工作进程，根据硬件调整，有人说几核cpu，就配几个，我觉得可以多一点
worker_processes 5；
#错误日志
error_log logs/error.log;
#pid文件位置
pid logs/nginx.pid;
worker_rlimit_nofile 8192;
events {
		#工作进程的最大连接数量，根据硬件调整，和前面工作进程配合起来用，尽量大，但是别把cpu跑到100%就行
		worker_connections 4096;
		}
http {
	include conf/mime.types;
	#反向代理配置，可以打开proxy.conf看看
	include /etc/nginx/proxy.conf;
	#fastcgi配置，可以打开fastcgi.conf看看
	include /etc/nginx/fastcgi.conf;
	default_type application/octet-stream;
	#日志的格式
	log_format main '$remote_addr – $remote_user [$time_local] $status '
	'"$request” $body_bytes_sent “$http_referer"'
	'"$http_user_agent” “$http_x_forwarded_for"';
	#访问日志
	access_log logs/access.log main;
	sendfile on;
	tcp_nopush on;
	#根据实际情况调整，如果server很多，就调大一点
	server_names_hash_bucket_size 128; # this seems to be required for some vhosts
	#这个例子是fastcgi的例子，如果用fastcgi就要仔细看
	server { 
		# php/fastcgi
		listen 80;
		#域名，可以有多个
		server_name domain1.com www.domain1.com;
		#访问日志，和上面的级别不一样，应该是下级的覆盖上级的
		access_log logs/domain1.access.log main;
		root html;
		location / {
			index index.html index.htm index.php;
		}
		#所有php后缀的，都通过fastcgi发送到1025端口上
		#上面include的fastcgi.conf在此应该是有作用，如果你不include，那么就把fastcgi.conf的配置项放在这个下面。
		location ~ .php$ {
				fastcgi_pass 127.0.0.1:1025;
		}
}
#这个是反向代理的例子
server { # simple reverse-proxy
	listen 80;
	server_name domain2.com www.domain2.com;
	access_log logs/domain2.access.log main;
	#静态文件，nginx自己处理
	location ~ ^/(images|javascript|js|css|flash|media|static)/ {
			root /var/www/virtual/big.server.com/htdocs;
			#过期30天，静态文件不怎么更新，过期可以设大一点，如果频繁更新，则可以设置得小一点。
			expires 30d;
			}
	#把请求转发给后台web服务器，反向代理和fastcgi的区别是，反向代理后面是web服务器，fastcgi后台是fasstcgi监听进程，当然，协议也不一样。
	location / {
			proxy_pass http://127.0.0.1:8080;
			}
		}
#upstream的负载均衡，weight是权重，可以根据机器配置定义权重。据说nginx可以根据后台响应时间调整。后台需要多个web服务器。

upstream big_server_com {
		server 127.0.0.3:8000 weight=5;
		server 127.0.0.3:8001 weight=5;
		server 192.168.0.1:8000;
		server 192.168.0.1:8001;
		}
server {
		listen 80;
		server_name big.server.com;
		access_log logs/big.server.access.log main;
		location / {
				proxy_pass http://big_server_com;
				}
		}
}

```
—————————————–
Nginx 安置后只有一个法式文件，自己并不供给各类办理法式，它是利用参数和体系旌旗灯号机制对 Nginx 历程自己举行节制的。 Nginx 的参数包罗有如下几个：
-c ：利用指定的设置装备摆设文件而不是 conf 目次下的 nginx.conf 。
-t：测试设置装备摆设文件是否准确，在运行时必要从头加载设置装备摆设的时辰，此号令很是主要，用来检测所点窜的设置装备摆设文件是否有语法错误。
-v：表现 nginx 版本号。
-V：表现 nginx 的版本号以及编译情况信息以及编译时的参数。
比方我们要测试某个设置装备摆设文件是否誊写准确，我们可以利用以下号令
sbin/nginx – t – c conf/nginx2.conf

经由过程旌旗灯号对 Nginx 举行节制
Nginx 撑持下表中的旌旗灯号：
  



旌旗灯号名
  



感化形貌
  



TERM, INT
  



快速封闭法式，中断当前正在处置的恳求
  



QUIT
  



处置完当前恳求后，封闭法式
  



HUP
  



从头加载设置装备摆设，并开启新的事情历程，封闭就的历程，此操纵不会间断恳求
  



USR1
  



从头打开日记文件，用于切换日记，比方天天天生一个新的日记文件
  



USR2
  



光滑进级可实行法式
  



WINCH
  



自在封闭事情历程
有两种体例来经由过程这些旌旗灯号去节制 Nginx，第一是经由过程 logs 目次下的 nginx.pid 检察当前运行的 Nginx 的历程ID，经由过程 kill – XXX来节制 Nginx，此中 XXX 便是上表中列出的旌旗灯号名。若是您的体系中只有一个 Nginx 历程，那您也可以经由过程 killall 号令来完成，比方运行 killall – s HUP nginx 来让 Nginx 从头加载设置装备摆设。
设置装备摆设 Nginx

先来看一个现实的设置装备摆设文件：
```
user  nobody;# 事情历程的属主
worker_processes  4;# 事情历程数，一样平常与 CPU 核数等同
#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;
#pid        logs/nginx.pid;
events {
    use epoll;#Linux 下机能最好的 event 模式
    worker_connections  2048;# 每个事情历程许可最大的同时毗连数
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    #log_format  main  '$remote_addr - $remote_user [$time_local] $request '
    #                  '"$status" $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';
    #access_log  off;
    access_log  logs/access.log;# 日记文件名
    sendfile        on;
    #tcp_nopush     on;
    tcp_nodelay     on;
    keepalive_timeout  65;
    include   gzip.conf;
    # 集群中的全部背景办事器的设置装备摆设信息
    upstream tomcats {
		  server 192.168.0.11:8080 weight=10;
		  server 192.168.0.11:8081 weight=10;
		  server 192.168.0.12:8080 weight=10;
		  server 192.168.0.12:8081 weight=10;
		  server 192.168.0.13:8080 weight=10;
		  server 192.168.0.13:8081 weight=10;
	    }
    server {
        listen       80;#HTTP 的端口
        server_name  localhost;
        charset utf-8;
        #access_log  logs/host.access.log  main;
  location ~ ^/NginxStatus/ {
     stub_status on; #Nginx 状况监控设置装备摆设
     access_log off;
  }
  location ~ ^/(WEB-INF)/ {
     deny all;
  }
  location ~ \.(htm|html|asp|php|gif|jpg|jpeg|png|bmp|ico|rar|css|js|
  zip|java|jar|txt|flv|swf|mid|doc|ppt|xls|pdf|txt|mp3|wma)$ {
             root /opt/webapp;
     expires 24h;
        }
        location / {
     proxy_pass http://tomcats;# 反向代办署理
     include proxy.conf;
        }
        error_page 404 /html/404.html;
        # redirect server error pages to the static page /50x.html
        #
  error_page 502 503 /html/502.html;
        error_page 500 504 /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
```

Nginx 监控
上 面是一个现实网站的设置装备摆设实例，此中灰色笔墨为设置装备摆设申明。上述设置装备摆设中，起首我们界说了一个 location ~ ^/NginxStatus/，如许经由过程 http://localhost/NginxStatus/ 就可以监控到 Nginx 的运行信息，表现的内容如下：

```
Active connections: 70
server accepts handled requests
14553819 14553819 19239266
Reading: 0 Writing: 3 Waiting: 67
```

NginxStatus 表现的内容意思如下：
· active connections – 当前 Nginx 正处置的勾当毗连数。
· server accepts handled requests — 统共处置了 14553819 个毗连 , 乐成建立 14553819 次握手 ( 证实中心没有失败的 ),统共处置了 19239266 个恳求 ( 均匀每次握手处置了 1.3 个数据恳求 )。
· reading — nginx 读取到客户真个 Header 信息数。
· writing — nginx 返回给客户真个 Header 信息数。
· waiting — 开启 keep-alive 的环境下，这个值即是 active – (reading + writing)，意思便是 Nginx 已经处置完正在等待下一次恳求指令的驻留毗连。
静态文件处置
经由过程正则表达式，我们可让 Nginx 辨认出各类静态文件，比方 images 路径下的全部恳求可以写为：

```
location ~ ^/images/ {
    root /opt/webapp/images;
}
```

而下面的设置装备摆设则界说了几种文件范例的恳求处置体例。

```
location ~ \.(htm|html|gif|jpg|jpeg|png|bmp|ico|css|js|txt)$ {
    root /opt/webapp;
    expires 24h;
}
```

对付比方图片、静态 HTML 文件、js 剧本文件和 css 样式文件等，我们但愿 Nginx 直接处置并返回给欣赏器，如许可以大大的加速网页欣赏时的速率。是以对付这类文件我们必要经由过程 root 指令来指定文件的存放路径，同时由于这类文件并不常点窜，经由过程 expires 指令来节制其在欣赏器的缓存，以削减不需要的恳求。 expires 指令可以节制 HTTP 应答中的“ Expires ”和“ Cache-Control ”的头标（起到节制页面缓存的感化）。您可以利用比方以下的格局来誊写 Expires：

```
expires 1 January, 1970, 00:00:01 GMT;
expires 60s;
expires 30m;
expires 24h;
expires 1d;
expires max;
expires off;
```

动态页面恳求处置
Nginx 自己并不撑持此刻风行的 JSP、ASP、PHP、PERL 等动态页面，可是它可以经由过程反向代办署理将恳求发送到后真个办事器，比方 Tomcat、Apache、IIS 等来完成动态页面的恳求处置。前面的设置装备摆设示例中，我们起首界说了由Nginx 直接处置的一些静态文件恳求后，其他全部的恳求经由过程 proxy_pass 指令传送给后真个办事器（在上述例子中是Tomcat）。最简略的 proxy_pass 用法如下：

```
location / {
    proxy_pass        http://localhost:8080;
    proxy_set_header  X-Real-IP  $remote_addr;
}
```

这里我们没有利用到集群，而是将恳求直接送到运行在 8080 端口的 Tomcat 办事上来完成近似 JSP 和 Servlet 的恳求处置。
当页面的拜候量很是大的时辰，每每必要多个应用办事器来配合负担动态页面的实行操纵，这时我们就必要利用集群的架构。 Nginx 经由过程 upstream 指令来界说一个办事器的集群，最前面阿谁完备的例子中我们界说了一个名为 tomcats 的集群，这个集群中包罗了三台办事器共 6 个 Tomcat 办事。而 proxy_pass 指令的写法酿成了：

```
location / {
    proxy_pass        http://tomcats;
    proxy_set_header  X-Real-IP  $remote_addr;
}
```

在 Nginx 的集群设置装备摆设中，Nginx 利用最简略的均匀分派法则给集群中的每个节点分派恳求。一旦某个节点失效时，大概从头起效时，Nginx 城市很是实时的处置状况的转变，以包管不会影响到用户的拜候。
总结
尽 管整个法式包只有五百多 K，但麻雀虽小、五脏俱全。 Nginx 官方供给的各类功效模块包罗万象，连系这些模块可以完备各类百般的设置装备摆设要求，比方：压缩、防盗链、集群、FastCGI、流媒体办事器、 Memcached 撑持、URL 重写等等，更关头的是 Nginx 拥有 Apache 和其他 HTTP 办事器无法对比的高机能。您乃至可以在不转变原有网站的架构上，经由过程在前端引入 Nginx 来晋升网站的拜候速率。
本文只是简略先容了 Nginx 的安置以及常见的根基的设置装备摆设和利用，更多关于 Nginx 的信息请阅读文章背面的参考资本。在这里要很是感激我的伴侣——陈磊（chanix@msn.com），他一向在做 Nginx 的中文WIKI（http://wiki.codemongers.com/NginxChs），同时也是他先容给我这么好的一款软件。
若是您的网站是运行在 Linux 下，若是您并没有利用一些很是庞大的并且确定 Nginx 无法完成的功效，那您应该尝尝 Nginx


