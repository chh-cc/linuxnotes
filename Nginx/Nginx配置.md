# Nginx配置

## 配置文件结构

```shell
vim nginx.conf
#全局参数
...
events {
	...
}
# http 服务相关设置
http {
	...
	一些优化参数;
	upstream static_pools {
		...
	}

	include	domains/*.conf;
}
```

```shell
vim domains/*.conf
server {
	...
	location / {
		...
	}
	location ~ .*\.(gif|jpg|jpeg|bmp|png|ico|txt|js|css)$ {
		if () {
			rewrite ...
		}
	}
	定义日志;
}
```

## 配置文件详解

### CPU亲和性

亲和力就是为多核 CPU 需做到让 Nginx 服务充分的配合使用，从而提高性能。

让每一个 Nginx 的 worker 线程都能够固定到具体的 CPU 核心上。

```shell
worker_cpu_affinity = auto
或
worker_processes	8;	#通常设置成和 cpu 核数相等（top然后按1或more /proc/cpuinfo可查看核数，生产环境一般不超过8个，进程太多并不好） 
worker_cpu_affinity	00000001 00000010 00000100 00001000 00010000 00100000 01000000 10000000;	#将8个进程分配到8个cpu
```

### 模型优化

Linux 系统下一切皆文件，比如我们打开一个设备，它便会产生一个文件描述符。在产生一个进程时，这个进程便需要一个进程描述符，这个进程描述符也是一个文件。所以在 Nginx 处理请求的时候，**每一个请求都会产生处理请求的描述符**。

epoll 本身是以异步非阻塞模型来处理请求流中的事件流。

```shell
events {
use epoll;
}
```

### 零拷贝

静态文件并不需要流转到用户态中，直接通过内核态效率更高，所以这时我们就需要在 Nginx 中开启 sendfile on，这样静态文件就可以在内核态中完成转发，而不用再去绕道用户态，提高了效率。

```shell
http {
sendfile        on;	#开启文件高效传输
}
```

### 文件压缩

```shell
http {
gzip on #负责打开后端的压缩功能；
gzip_buffer 16 8k  #表示设置 Nginx 在处理文件压缩时的内存空间(16个单位为8k的内存空间)；
gzip_comp_level 6 #表示 Nginx 在处理压缩时的压缩等级，通常等级越高它的压缩比就越大，但并不是说压缩比越大越好，还是需要根据实际情况来选择合适的压缩比，压缩比太大影响性能，压缩比太小起不到应有的效果，一般来说推荐你设置成 6 就比较合适；（1~9）
gzip_http_version 1.1 #表示只对 HTTP 1.1 版本的协议进行压缩；
gzip_min_length 256 #表示只有大于最小的 256 字节长度时才进行压缩，如果小于该长度就不进行压缩；
gzip_proxied any #代表 Nginx 作为反向代理时依据后端服务器时返回信息设置一些 gzip 压缩策略；
gzip_vary on #表示是否发送 Vary：Accept_Encoding 响应头字段，实现通知接收方服务端作了 gzip 压缩；
application/vnd.ms-fontobject image/x-icon;  #gip 压缩类型；
gzip_disable “msie6”;  #关闭 IE6 的压缩。
}
```

### 缓存优化

```shell
#浏览器缓存
location / {
expires 30d;
}
#https缓存
ssl_session_cache shared:SSL:10m;
ssl_session_timeout 10m; #SSL session过期时间
#打开文件缓存
open_file_cache max=1000 inactive=20s; #inactive 是指经过多长时间文件没被请求后删除缓存。
open_file_cache_valid 30s; #这个是指多长时间检查一次缓存的有效信息
open_file_cache_min_uses 2; #参数时间内文件的最少使用次数，如果超过这个数字，文件描述符一直是在缓存中打开的
open_file_cache_error on;
max 表示最大能够缓存的文件个数，inactive 表示最少的用户使用次数。我们结合看一下，这个表示在 20 秒内最小需要使用两次。如果没有使用的话，就会把元数据删掉，也这就是一个淘汰元数据的策略。
```

### 代理配置

```shell
location / {
        proxy_pass http://bbs;	#代理的后端服务器URL
        proxy_redirect off;
        # 要使用 nginx 代理后台获取真实的 IP 需在 nginx.conf 配置中加入配置信息
        proxy_set_header Host $http_host;	# 包含客户端真实的域名和端口号；
        proxy_set_header X-Real-IP $remote_addr;	 # 表示客户端真实的IP；	
        proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;	#在多层代理时会包含真实客户端及中间每个代理服务器的IP
		proxy_set_header X-Forwarded-Proto $scheme;	# 表示客户端真实的协议（http还是https）；
		proxy_set_header X-NginX-Proxy true;
		
		proxy_connect_timeout 30; #nginx 跟后端服务器连接超时时间
		proxy_send_timeout 60; #后端服务器数据回传时间
		proxy_read_timeout 60; #连接成功后，后端服务器响应时间
		
		proxy_buffering on;	#缓冲开关，nignx会把后端返回的内容先放到缓冲区当中，然后再返回给客户端（边收边传，不是全部接收完再传给客户端）
		proxy_buffer_size 32k;	#缓冲区大小
		proxy_buffers 4 128k;	#缓冲区数量
		proxy_busy_buffers_size 256k;	#忙碌的缓冲区大小控制同时传递给客户端的buffer数量
		proxy_max_temp_file_size 256k;
}
```

### 防盗链

```shell
location ~ .*\.(gif|jpg|jpeg|svg|bmp|png|ico|txt|js|css|html)$ {
        root    tomcat_static;
        expires 1d;
        #防盗链
        valid_referers none blocked     chenhh.xyz *.chenhh.xyz;
        if ($invalid_referer) {
            return 404;
        }
}
```



```shell
vim nginx.conf
#全局参数
user	www www;	#worker进程的管理用户和用户组
error_log  logs/error.log;    # 指定错误日志 
worker_processes	8;
worker_cpu_affinity	00000001 00000010 00000100 00001000 00010000 00100000 01000000 10000000;
worker_rlimit_nofile	10240;	#每个进程的最大文件打开数

events {
use epoll;
worker_connections	4096;	#单个 worker process 进程可以同时接收多少访问请求
multi_accept	on;	#尽可能多的接受请求. 
}

# http 服务相关设置
http { 
include	mime.types;
default_type	application/octet-stream;

charset utf-8;
server_names_hash_bucket_size    128;	#如果定义了大量名字，或者定义了非常长的名字，那就需要调整服务器名字的hash表大小

sendfile        on;
tcp_nopush     on;	#防止网络阻塞
tcp_nodelay    on;	#防止网络阻塞,包含了keepalive参数才有效
keepalive_timeout  60;	#客户端连接保持会话的超时时间。超过这个时间，服务器会关闭该连接
#client_header_timeout 15;	#客户端请求头读取超时时间．如超过这个时间，客户端还没有发送任何数据，Nginx将返回“Request timeout(408)＂错误，默认值是60。
#client_body_timeout 15;	#客户端请求主体读取超时时间。如超过这个时间，客户端还没有发送任何数据，Nginx将返回“Request timeout(408)错误，默认值是60
#send_timeout 15;	#指定响应客户端的超时时间。这个超时仅限于两个连接活动之间的时间，如果超过这个时间，客户端没有任何活动，Nginx将会关闭连接。

client_header_buffer_size    4k;
large_client_header_buffer_size    4 32k;

client_max_body_size        300m;	#允许客户端请求的最大单文件字节数（上传文件大小）,根据业务调整
client_body_buffer_size     512k;	#缓冲区代理缓冲用户端请求的最大字节数

#代理配置
proxy_connect_timeout	10;	#nginx跟后端服务器连接超时时间(代理连接超时，单位秒)。不要设置的太小，否则会报504错误。
proxy_send_timeout	10;	#后端服务器数据回传时间(代理发送超时，单位秒)
proxy_read_timeout	10;	#连接成功后，后端服务器响应时间(代理接收超时) 

proxy_buffer_size	16k;	#设置代理（nginx）保存用户头信息的缓冲区大小
proxy_buffers	4 64k;	#缓存区的数目和大小
proxy_busy_buffers_size	128k; #高负荷下缓冲大小（proxy_buffers*2）
proxy_temp_file_write_size    128k;

# fastcgi调优（配合PHP引擎动态服务）
fastcgi_connect_timeout    300;	#指定连接到后端fastCGI的超时时间
fastcgi_send_timeout    300;	#向FastCGI传送请求的超时时间，这个值是指已经完成两次握手后向FastCGI传送请求的超时时间
fastcgi_read_timeout    300;	#指定接收FastcGI应答的超时时间，这个值是指己经完成两次握手后接收FastCGI应答的超时时间
fastcgi_buffer_size    32k;	#指定读取FastCGI应答第一部分需要用多大的缓冲区，这个值表示将使用1个64KB的缓冲区读取应答的第一部分（应答头），可以置为fastcgi_buffers选项指定的缓冲区大小。
fastcgi_buffers    8 32k;	#指定本地需要用多少和多大的缓冲区来缓冲FastCGI的应答请求。如果一个PHP脚本所产生的页面大小为256KB,为其分配4个64KB的缓冲区来缓存；如果页面大小大于256KB，那么大于256KB的部分会缓存到fastcgi_temp指定的路径中，但是这并不是好方法，因为内存中的数据处理速度要快于硬盘。一般这个值应该为站点中PHP脚本所产生的页面大小的中间值，如果站点大部分脚本所产生的页面大小为256KB，那么可以把这个值设置为"16 16k"、"16 16k" 

gzip	on;
gzip_min_length	1k;
gzip_buffers	4 16k;
gzip_http_version	1.1;
gzip_comp_level	2;
gzip_types	text/plain application/javascript text/css application/xml;
gzip_vary	on;

server_tag    off;
servert_info    off;
server_tokens    off;

log_format	main  '$remote_addr - $remote_user [$time_local] "$request" '
                  '$status $body_bytes_sent "$http_referer" '
                  '"$http_user_agent" "$http_x_forwarded_for"';
access_log  /var/log/nginx/access.log  main;    #设置访问日志的位置和格式 
upstream bbs {
server    10.1.96.3:80 weight=1 max_fails=2 fail_timeout=30s;
server    10.1.96.4:80 weight=1 max_fails=2 fail_timeout=30s;
}
include	/domains/*.conf	#加载一个配置文件，作为虚拟主机
#限流
}

```

```shell
vim domains/chenhh.xyz_80.conf
# 虚拟服务器的相关设置
server {
    listen      104.207.128.168:80;
    server_name blog.chenhh.xyz chenhh.xyz;
    error_page500 502 503 504 = /50x.html;
    if ($host != 'blog.chenhh.xyz') {
        rewrite ^/(.*)$ http://blog.chenhh.xyz/$1 permanent;
    }
    rewrite ....
    ....
    
    location / {
        #代理配置
        proxy_pass http://bbs;	#代理的后端服务器URL
        proxy_redirect off;
        # 要使用 nginx 代理后台获取真实的 IP 需在 nginx.conf 配置中加入配置信息
        proxy_set_header Host $http_host;	# 包含客户端真实的域名和端口号；
        proxy_set_header X-Real-IP $remote_addr;	 # 表示客户端真实的IP；	
        proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;	#在多层代理时会包含真实客户端及中间每个代理服务器的IP
		proxy_set_header X-Forwarded-Proto $scheme;	# 表示客户端真实的协议（http还是https）；
		proxy_set_header X-NginX-Proxy true;
		
		proxy_connect_timeout 30;
		proxy_send_timeout 60;
		proxy_read_timeout 60;
		proxy_buffering on;	#缓冲开关，nignx会把后端返回的内容先放到缓冲区当中，然后再返回给客户端（边收边传，不是全部接收完再传给客户端）
		proxy_buffer_size 32k;	#缓冲区大小
		proxy_buffers 4 128k;	#缓冲区数量
		proxy_busy_buffers_size 256k;	#忙碌的缓冲区大小控制同时传递给客户端的buffer数量
		proxy_max_temp_file_size 256k;
    }
    #动静分离
    location ~ .*\.(gif|jpg|jpeg|svg|bmp|png|ico|txt|js|css|html)$ {
        root    tomcat_static;
        expires 1d;
        #防盗链
        valid_referers none blocked     chenhh.xyz *.chenhh.xyz;
        if ($invalid_referer) {
            return 404;
        }
    }
    #自定义404页面
    location = /50x.html {
        root    /usr/local/nginx/html;
    }

    access_log  /data/logs/tomcat/access.log    main;
    error_log   /data/logs/tomcat/error.log     crit;
}
```

## 日志中每个字段的含义

```shell
$http_x_real_ip 记录客户端IP地址,通常使用
$remote_addr, 这里结合负载均衡的配置获取真实的客户端IP 
$http_host 请求的域名 
$time_local 通用日志格式下的本地时间。 
$request 记录请求的URL和HTTP协议版本,及请求方式。 格式为"METHOD URL VERSION" 
$request_length 请求的长度（包括请求行，请求头和请求正文）。 
$status 记录请求状态，HTTP状态码。 
$body_bytes_sent 发送给客户端的响应体大小，不包括响应头。 $http_referer 记录从哪个页面链接访问过来的 
$http_user_agent 记录客户端浏览器相关信息 
$remote_addr 远端服务器，到底是哪台代理服务器来的请求。
```

## nginx自带变量

```shell
$args ： 这个变量等于请求行中的参数，同$query_string 
$content_length ： 请求头中的Content-length字段。 
$content_type ： 请求头中的Content-Type字段。 
$document_root ： 当前请求在root指令中指定的值。 
$host ： 请求主机头字段，否则为服务器名称。 
$http_user_agent ： 客户端agent信息 
$http_cookie ： 客户端cookie信息 
$limit_rate ： 这个变量可以限制连接速率。 
$request_method ： 客户端请求的动作，通常为GET或POST。 
$remote_addr ： 客户端的IP地址。 
$remote_port ： 客户端的端口。 
$remote_user ： 已经经过Auth Basic Module验证的用户名。 $request_filename ： 当前请求的文件路径，由root或alias指令与URI请求生成。 
$scheme ： HTTP方法（如http，https）。 
$server_protocol ： 请求使用的协议，通常是HTTP/1.0或HTTP/1.1。 $server_addr ： 服务器地址，在完成一次系统调用后可以确定这个值。 $server_name ： 服务器名称。 
$server_port ： 请求到达服务器的端口号。 
$request_uri ： 包含请求参数的原始URI，不包含主机名，如：”/foo/bar.php?arg=baz”。 
$uri ： 不带请求参数的当前URI，$uri不包含主机名，如”/foo/bar.html”。 $document_uri ： 与$uri相同。 
... ... 
更多的变量，我们可以通过产线官网: http://nginx.org/en/docs/http/ngx_http_core_module.html#variables
```

## location

```shell
= 表示精确匹配，优先级也是最高的 
^~ 表示 url 以某个常规字符串开头,理解为匹配url路径即可 
~ 表示区分大小写的正则匹配 
~* 表示不区分大小写的正则匹配 
!~ 表示区分大小写不匹配的正则 
!~* 表示不区分大小写不匹配的正则 
/ 通用匹配，任何请求都会匹配到 
```

优先级：

- = 
- ^~    
- ~|~*|!~|!~*   
- /

查找顺序：

```text
1：带有“=“的精确匹配优先 
2：没有修饰符的精确匹配 
3：正则表达式按照他们在配置文件中定义的顺序 
4：带有“^~”修饰符的，开头匹配 
5：带有“~” 或“~*” 修饰符的，如果正则表达式与URI匹配 
6：没有修饰符的，如果指定字符串与URI开头匹配
```

location 配置示例

没有修饰符:

```shell
location /abc {
root /usr/share/nginx/html
index qfedu.html
}
那么，如下是对的：
http://qfedu.com/abc
http://qfedu.com/abc?p1
http://qfedu.com/abc/
```

~* 表示：指定的正则表达式不区分大小写：

```shell
location ~* ^/abc$ {
root /usr/share/nginx/htm
index qfedu.html
}
那么，如下是对的：
http://qfedu.com/abc
http://qfedu..com/ABC
http://qfedu..com/abc?p1=11&p2=22
```



## rewrite规则

Rewrite 对称 URL Rewrite，即URL重写，就是把传入Web的请求重定向到其他URL的过程。

- URL Rewrite 最常见的应用是 URL 伪静态化，是将动态页面显示为静态页面方式的一种技 术。比如 http://www.123.com/news/index.php?id=123 使用URLRewrite 转换后可以显示 为 http://www.123.com/news/123.html 对于追求完美主义的网站设计师，就算是网页的地址也希望看起来尽量简洁明快。理论上，搜 索引擎更喜欢静态页面形式的网页，搜索引擎对静态页面的评分一般要高于动态页面。所以， UrlRewrite可以让我们网站的网页更容易被搜索引擎所收录。 
- 从安全角度上讲，如果在URL中暴露太多的参数，无疑会造成一定量的信息泄漏，可能会被一 些黑客利用，对你的系统造成一定的破坏，所以静态化的URL地址可以给我们带来更高的安全 性。 
- 实现网站地址跳转，例如用户访问 360buy.com，将其跳转到jd.com。例如当用户访问 qfedu.com 的 80端口时，将其跳转到443端口。

语法：

```shell
rewrite regex replacement [flag];
#这里的使用 regex 匹配 URI，并将匹配到的 URI 替换成新的 URI（replacement）。如果有多个rewrite，执行顺序是从上到下依次执行，匹配到一个后匹配并不会终止，会继续匹配下去，直到返回最后一个匹配为止。如果想中途终止，则需要设置 flag 参数。
#当然上面说的都是重写 URI，如果 replacement 中包含了任何协议相关，如：http:// 和 https://，则请求就直接返回 302 重定向终止了。
#当然，浏览器在接收到 30x 的状态码后，会再度根据这个返回去请求 rewrite 之后的地址，最终得到所要想要的结果。如果不是 30x 的状态码，则属于 nginx 内部跳转，浏览器不需要再度发起请求。
```

四种flag：

|           |                                                              |
| --------- | ------------------------------------------------------------ |
| last      | 停止所有 rewrite 相关指令，然后使用新的 URI 进行 location 匹配。 |
| break     | 停止所有 rewrite 相关指令， 和 last 不同的是，last 接着继续使用新的 URI 匹配location。而 break 则是直接使用当前的 URI 进行请求处理，能避免重复rewrite，last 一般在 server，break 一般在 location。 |
| redirect  | URI 中不包含协议如 https://，但依然希望它返回 30x，让浏览器二次请求然后获取到结果就需要 redirect。 |
| permanent | 和 redirect 类似，但是直接返回 301 永久重定向。              |

正则表达：

| 字符 | 描述                                                         |
| :--- | :----------------------------------------------------------- |
| .    | 匹配除换行符以外的任意字符                                   |
| ?    | 匹配前面的字符零次或一次                                     |
| +    | 匹配前面的字符一次或多次                                     |
| *    | 匹配前面的字符0次或多次                                      |
| \d   | 匹配一个数字字符。等价于[0-9]                                |
| \    | 将后面接着的字符标记为一个特殊字符或一个原义字符或一个向后引用。如“\n”匹配一个换行符，而“$”则匹配“$” |
| ^    | 匹配字符串的开始                                             |
| $    | 匹配字符串的结尾                                             |
| {n}  | 匹配前面的字符n次                                            |
| {n,} | 匹配前面的字符n次或更多次                                    |
| [c]  | 匹配单个字符c                                                |
| [az] | 匹配a-z小写字母的任意一个                                    |

小括号()之间匹配的内容，可以在后面通过$1来引用，$2表示的是前面第二个()里的内容

## if

|         |                              |
| ------- | ---------------------------- |
| =或!=   | 直接比较变量和内容           |
| ~       | 区分大小写正则表达式匹配     |
| ~*      | 不区分大小写的正则表达式匹配 |
| !~      | 区分大小写的正则表达式不匹配 |
| -f和!-f | 用来判断文件是否存在         |
| -d和!-d | 用来判断目录是否存在         |
| -e和!-e | 用来判断文件或目录是否存在   |
| -x和!-x | 用来判断文件是否可执行       |

if全局变量

