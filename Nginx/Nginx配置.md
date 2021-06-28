# Nginx配置

## 配置文件结构

```shell
vim nginx.conf

#全局参数(配置影响nginx全局的指令。一般有运行nginx服务器的用户组，nginx进程pid存放路径，日志存放路径，配置文件引入，允许生成worker process数等。)
user nginx nginx	#定义nginx运行的用户和组
pid /run/nginx.pid;	#pid文件位置
error_log /var/log/nginx/error.log notice;	#全局日志输出位置,notice是日志级别
worker_processes auto;	#开启的工作进程数量,和cpu核心数量相等
worker_rlimit_nofile 65535;	#一个nginx 进程打开的最多文件描述符数目，理论值应该是最多打开文件数（ulimit -n）与进程数相除，但是nginx分配请求并不是那么均匀，所以最好与ulimit -n值一致

#events块(配置影响nginx服务器或与用户的网络连接。有每个进程的最大连接数，选取哪种事件驱动模型处理连接请求，是否允许同时接受多个网路连接，开启多个网络连接序列化等。)
events {
	use epoll;
	worker_connections	51200;	#每个work进程可以建立的最大的连接数,并发限定总数是 worker_processes 和 worker_connections 的乘积;在设置了反向代理的情况下，max_clients = worker_processes * worker_connections / 2  因为作为反向代理服务器，每个并发会建立与客户端的连接和与后端服务的连接，会占用两个连接。
}

# http 服务相关设置
http {
	(可以嵌套多个server，配置代理，缓存，日志定义等绝大多数功能和第三方模块的配置。)
	#日志格式
	log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    access_log  /var/log/nginx/access.log  main;
    error_log /var/log/nginx/error.log info;
    
    #limit限制
    #设置用于保存各种 key（比如当前连接数）的共享内存的参数。 5m 就是 5兆字节，这个值应该被设置的足够大以存储（32K*5） 32byte 状态或者（16K*5） 64byte 状态。
    limit_conn_zone $binary_remote_addr zone=addr:5m;
    #我们设置的值是 100，也就是说我们允许每一个 IP 地址最多同时打开有 100 个连接。
    limit_conn addr 50;
    #限速模块，前3M下载时不限速
    limit_rate_after 3m;
    #限速模块
    limit_rate 512k;
    
    #基础设置
    #禁用ssi
    ssi off;
    #禁用autoindex 模块
    autoindex off;
    #媒体类型,标准
    include       mime.types;
    #默认文件类型，默认为text/plain
    default_type  application/octet-stream;
    #隐藏版本号
    server_tokens off;
    #编码格式
    charset UTF-8;
    
    #信息传输
    #开启高效文件传输模式，sendfile指令指定nginx是否调用sendfile函数来输出文件，对于普通应用设为 on，如果用来进行下载等应用磁盘IO重负载应用，可设置为off，以平衡磁盘与网络I/O处理速度，降低系统的负载。如果图片显示不正常把这个改成off。
    sendfile        on;
    #必须在sendfile开启模式才有效，告诉nginx在一个数据包里发送所有头文件，而不一个接一个的发送。
    tcp_nopush     on;
    #必须在sendfile开启模式才有效告诉nginx不要缓存数据，而是一段一段的发送--当需要及时发送数据时，就应该给应用设置这个属性，这样发送一小块数据信息时就不能立即得到返回值。
    tcp_nodelay on;
    
    #超时设置
    #客户端连接保持会话超时时间，超过这个时间，服务器断开这个链接，对于后端是php，可以低一些，因为php解析快，java的话要长一些，java解析慢
    keepalive_timeout  30;
    #设置请求头的超时时间。我们也可以把这个设置低些，如果超过这个时间没有发送任何数据，nginx将返回request time out的错误
    client_header_timeout 10;
    #设置请求体的超时时间。我们也可以把这个设置低些，超过这个时间没有发送任何数据，和上面一样的错误提示
    client_body_timeout 10;
    #响应客户端超时时间，服务端给客户端发送数据，如果客户端迟迟不接受没超过以下时间将断开连接
    send_timeout 10;
    #告诉nginx关闭不响应的客户端连接。这将会释放那个客户端所占有的内存空间。
    reset_timedout_connection on;
	
	#server_name控制
	#对虚拟主机服务器名字(server_name www.xx.com这种)在内存中做hash，如果名字太长，就需要将如下值变大为64
	#保存服务器名字的hash表是由指令server_names_hash_max_size 和server_names_hash_bucket_size所控制的。参数hash bucket size总是等于hash表的大小，并且是一路处理器缓存大小的倍数。在减少了在内存中的存取次数后，使在处理器中加速查找hash表键值成为可能。如果hash bucket size等于一路处理器缓存的大小，那么在查找键的时候，最坏的情况下在内存中查找的次数为2。第一次是确定存储单元的地址，第二次是在存储单元中查找键 值。因此，如果Nginx给出需要增大hash max size 或 hash bucket size的提示，那么首要的是增大前一个参数的大小
    server_names_hash_bucket_size 64;
    #存储服务器名字的值大小，默认512kb，如果一个server对应多个域名，就要加大此值
    server_names_hash_max_size 256;
    
    #提交缓存
    #nginx 会将整个请求头都放在一个 buffer 里面，如果用户的请求头太大，这个 buffer 装不下，那 nginx 就会重新分配一个新的更大的 buffer来装请求头，这个大 buffer 可以通过 large_client_header_buffers 来设置，比如配置 4 8k，就是表示有四个 8k 大小的buffer 可以用。
    #客户端请求头部的缓冲区大小。这个可以根据你的系统分页大小来设置，一般一个请求的头部大小不会超过1k，不过由于一般系统分页都要大于1k，所以这里设置为分页大小。分页大小可以用命令getconf PAGESIZE取得。
    client_header_buffer_size 32k;
    #此指令规定了用于读取大型客户端请求头的缓冲区的最大数量和大小。 这些缓冲区仅在缺省缓冲区不足时按需分配。 当处理请求或连接转换到保持活动状态时，释放缓冲区。如下例子：
    large_client_header_buffers 4 64k;
    #NGINX上传文件最大限制。 如果请求大于指定的大小，则NGINX发回HTTP 413（Request Entity too large）错误。如果在上传大文件，可以将此值设置大一些
    client_max_body_size 20m;
    
    #这个将为打开文件指定缓存，默认是没有启用的，max指定缓存数量，建议和打开文件数一致，inactive 是指经过多长时间文件没被请求后删除缓存。
    open_file_cache max=100000 inactive=20s;
    #这个是指多长时间检查一次缓存的有效信息。
    open_file_cache_valid 30s;
    #open_file_cache指令中的inactive 参数时间内文件的最少使用次数，如果超过这个数字，文件描述符一直是在缓存中打开的，如上例，如果有一个文件在inactive 时间内一次没被使用，它将被移除。
    open_file_cache_min_uses 2;
    #指定了当搜索一个文件时是否缓存错误信息，也包括再次给配置中添加文件。我们也包括了服务器模块，这些是在不同文件中定义的。如果你的服务器模块不在这些位置，你就得修改这一行来指定正确的位置
    open_file_cache_errors off;
    
    #压缩
    #开启页面压缩
    gzip  on;
    #gzip压缩是要申请临时内存空间的，假设前提是压缩后大小是小于等于压缩前的。例如，如果原始文件大小为10K，那么它超过了8K，所以分配的内存是8 * 2 = 16K;再例如，原始文件大小为18K，很明显16K也是不够的，那么按照 8 * 2 * 2 = 32K的大小申请内存。如果没有设置，默认值是申请跟原始数据相同大小的内存空间去存储gzip压缩结果。
    gzip_buffers 16 8k;
    #进行压缩的原始文件的最小大小值，也就是说如果原始文件小于1K，那么就不会进行压缩了
    gzip_min_length 1024K;
    # 默认值: gzip_http_version 1.1(就是说对HTTP/1.1协议的请求才会进行gzip压缩)
# 识别http的协议版本。由于早期的一些浏览器或者http客户端，可能不支持gzip自解压，用户就会看到乱码，所以做一些判断还是有必要的。 
# 注：99.99%的浏览器基本上都支持gzip解压了，所以可以不用设这个值,保持系统默认即可。
# 假设我们使用的是默认值1.1，如果我们使用了proxy_pass进行反向代理，那么nginx和后端的upstream server之间是用HTTP/1.0协议通信的，如果我们使用nginx通过反向代理做Cache Server，而且前端的nginx没有开启gzip，同时，我们后端的nginx上没有设置gzip_http_version为1.0，那么Cache的url将不会进行gzip压缩
    gzip_http_version 1.1;
    # 默认值：1(建议选择为4)
    # gzip压缩比/压缩级别，压缩级别 1-9，级别越高压缩率越大，当然压缩时间也就越长（传输快但比较消耗cpu）。
    gzip_comp_level 6;
    #需要进行gzip压缩的Content-Type的Header的类型。建议js、text、css、xml、json都要进行压缩；图片就没必要了，gif、jpge文件已经压缩得很好了，就算再压，效果也不好，而且还耗费cpu。
    gzip_types text/HTML text/plain application/x-javascript text/css application/xml;
    # 禁用IE6的gzip压缩，又是因为杯具的IE6。当然，IE6目前依然广泛的存在，所以这里你也可以设置为“MSIE [1-5].”
    # IE6的某些版本对gzip的压缩支持很不好，会造成页面的假死，今天产品的同学就测试出了这个问题后来调试后，发现是对img进行gzip后造成IE6的假死，把对img的gzip压缩去掉后就正常了为了确保其它的IE6版本不出问题，所以建议加上gzip_disable的设置
    gzip_disable "msie6";

    # 默认值：off
    # Nginx作为反向代理的时候启用，开启或者关闭后端服务器返回的结果，匹配的前提是后端服务器必须要返回包含"Via"的 header头。
    #off - 关闭所有的代理结果数据的压缩
    #expired - 启用压缩，如果header头中包含 "Expires" 头信息
    #no-cache - 启用压缩，如果header头中包含 "Cache-Control:no-cache" 头信息
    #no-store - 启用压缩，如果header头中包含 "Cache-Control:no-store" 头信息
    #private - 启用压缩，如果header头中包含 "Cache-Control:private" 头信息
    #no_last_modified - 启用压缩,如果header头中不包含 "Last-Modified" 头信息
    #no_etag - 启用压缩 ,如果header头中不包含 "ETag" 头信息
    #auth - 启用压缩 , 如果header头中包含 "Authorization" 头信息
    #any - 无条件启用压缩
    gzip_proxied any;
    #尽量发送压缩过的静态文件
    gzip_static on;
    
    # fastcgi调优（配合PHP引擎动态服务）
    fastcgi_connect_timeout    300;	#指定连接到后端fastCGI的超时时间
    fastcgi_send_timeout    300;	#向FastCGI传送请求的超时时间，这个值是指已经完成两次握手后向FastCGI传送请求的超时时间
    fastcgi_read_timeout    300;	#指定接收FastcGI应答的超时时间，这个值是指己经完成两次握手后接收FastCGI应答的超时时间
    fastcgi_buffer_size    32k;	#指定读取FastCGI应答第一部分需要用多大的缓冲区，这个值表示将使用1个64KB的缓冲区读取应答的第一部分（应答头），可以置为fastcgi_buffers选项指定的缓冲区大小。
    fastcgi_buffers    8 32k;	#指定本地需要用多少和多大的缓冲区来缓冲FastCGI的应答请求。如果一个PHP脚本所产生的页面大小为256KB,为其分配4个64KB的缓冲区来缓存；如果页面大小大于256KB，那么大于256KB的部分会缓存到fastcgi_temp指定的路径中，但是这并不是好方法，因为内存中的数据处理速度要快于硬盘。一般这个值应该为站点中PHP脚本所产生的页面大小的中间值，如果站点大部分脚本所产生的页面大小为256KB，那么可以把这个值设置为"16 16k"、"16 16k" 
    
	upstream static_pools {
		server    10.1.96.3:80 weight=1 max_fails=2 fail_timeout=30s;
        server    10.1.96.4:80 weight=1 max_fails=2 fail_timeout=30s;
	}

	include	domains/*.conf;
}
```

```shell
vim domains/*.conf
server {
	(配置虚拟主机的相关参数)
	listen       80;#监听端口
	server_name  www.zjswdlt.cn;#域名
	root         /data/webapps/htdocs;
	access.log   /var/logs/webapps_access.log main
	#配置代理
	location / {
	    #代理的后端服务器
		proxy_pass http://static_pools
		#关闭proxy重定向
		proxy_redirect off;
        # 要使用 nginx 代理后台获取真实的 IP 需在 nginx.conf 配置中加入配置信息
        proxy_set_header Host	$http_host;	# 包含客户端真实的域名和端口号；
        proxy_set_header X-Real-IP	$remote_addr;	 # 表示客户端真实的IP（remote_addr 是一个 Nginx 的内置变量，它获取到的是 Nginx 层前端的用户 IP 地址，这个地址是一个 4 层的 IP 地址）；
        （相对于下面的方式而言更加准确，因为 remote_addr 是直接获取第一层代理的用户 IP 地址，如果直接把这个地址传递给 X-Real，这样就会更加准确。但是它有什么劣势呢？如果是多级代理的话，用户如果不是直接请求到最终的代理层，而是在中间通过了 n 层带来转发过来的话，此时 remote_addr 可能获取的不是用户的信息，而是 Nginx 最近一层代理过来的 IP 地址，此时同样没有获取到真实的用户 IP 地址信息。）
        proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;	#在多层代理时会包含真实客户端及中间每个代理服务器的IP（加了一个转发到后端的标准 head 信息，把用户的 IP 信息通过 X-Forwarded-For  方式传递过去）
		proxy_set_header X-Forwarded-Proto	$scheme;	# 表示客户端真实的协议（http还是https）；
		proxy_set_header X-NginX-Proxy true;
		
		proxy_connect_timeout 90; #nginx 跟后端服务器连接超时时间
		proxy_send_timeout 90; #nginx向后端发送请求报文的超时时间
		proxy_read_timeout 90; #nginx读取后端响应内容的超时时间
		
		proxy_buffering on;	#缓冲开关，nignx会把后端返回的内容先放到缓冲区当中，然后再返回给客户端（边收边传，不是全部接收完再传给客户端）
		proxy_buffer_size 128k;	#缓冲区大小
		proxy_buffers 4 64k;	#缓冲区数量及每个buffer被分配的内存大小
		proxy_busy_buffers_size 256k;	#忙碌的缓冲区大小控制同时传递给客户端的buffer数量
		proxy_max_temp_file_size 256k;
	}
	
	#动静分离
	location ~ .*\.(gif|jpg|jpeg|svg|bmp|png|ico|txt|js|css|html)$ {
        root    tomcat_static;
        expires 1d;
        #防盗链（即防止别人盗用网站的资源）
        valid_referers none blocked     chenhh.xyz *.chenhh.xyz;
        if ($invalid_referer) {
            return 404;
        }
    }
    
    #配置nginx监测状态页面
    location /status {
        access_log  off;        #关闭访问日志
		stub_status on;         #开启状态页监测
		allow 192.168.0.0/16;	#只允许192.168.0.0/16网段访问
		deny all;				#其余网段禁止访问
	}
    
	location ~ .*\.(gif|jpg|jpeg|bmp|png|ico|txt|js|css)$ {
		if () {
			rewrite ...
		}
	}
	定义日志;
}
```

## 日志中变量的含义

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
$document_root ： 当前请求的文档根目录或别名。 
$host ： 请求主机头字段，否则为服务器名称。优先级：HTTP请求行的主机名>"HOST"请求头字段>符合请求的服务器名 
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

location [=|~|~*|^~] /uri/ {...}

```shell
= /uri 表示精确匹配，优先级也是最高的 
^~ /uri 表示 uri 以某个常规字符串开头,理解为匹配uri路径即可 
~ pattern 表示区分大小写的正则匹配 
~* pattern 表示不区分大小写的正则匹配 
!~ pattern 表示区分大小写不匹配的正则 
!~* pattern 表示不区分大小写不匹配的正则
/uri 不带任何修饰符，也表示前缀匹配，但是在正则匹配之后。
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

替换后的url根据四种flag进行处理：

|           |                                                              |
| --------- | ------------------------------------------------------------ |
| last      | 停止所有 rewrite 相关指令，浏览器地址栏URL地址不变，然后使用新的 URI 进行 location 匹配。 |
| break     | 停止所有 rewrite 相关指令， 和 last 不同的是，last 接着继续使用新的 URI 匹配location。而 break 则是直接使用当前的 URI 进行请求处理，能避免重复rewrite，last 一般在 server，break 一般在 location。 |
| redirect  | URI 中不包含协议如 https://，但依然希望它返回 30x，让浏览器二次请求然后获取到结果就需要 redirect。 |
| permanent | 和 redirect 类似，但是直接返回 301 永久重定向。              |

示例：

```shell
root html/;
        location /first {
            rewrite /first(.*) /second$1 last;
            return 200 'first!';
        }
        location /second {
            rewrite /second(.*) /third$1 break;
            return 200 'second!';
        }
        location /third {
            return 200 'third!';
        }
访问/first/1.txt的结果为：html/third/1.txt的内容
访问/second/1.txt的结果为：html/third/1.txt的内容
访问/third/1.txt的结果为：third的内容
```

示例：

```shell
# http://www.test.com/test/abc/1.html ⇒ http://www.test.com/ccc/bbb/2.html
location /test {
    rewrite .* /ccc/bbb/2.html permanent;
}
# http://www.test.com/2015/ccc/bbb/2.html ==> http://www.test.com/2014/ccc/bbb/2.html
location /2015 {
    rewrite ^/2015/(.*)$ /2014/$1 permanent;
}
# http://www.test.com/2015/ccc/bbb/2.html  ==> http://jd.com/index.php
location /2015 {
    if ($host ~* test.com) {
        rewrite .* https://jd.com/index.php permanent;
    }
}
# http://www.test.com/kkk/1.html ==> http://jd.com/kkk/1.html
location / {
    root html;
    index index.html index.htm;
    if ($host ~* test.com) {
        rewrite .* http://jd.com/$request_uri permanent;
    }
}
# www.test.com/www ===> www.test.com/www/
# ^/(.*)([^/])$表示以/符号开始并紧跟着任何字符，同时不是以/为结束的字符串
if (-d $request_filename) {
    rewrite ^/(.*)([^/])$ http://$host/$1$2/ permanent;
}
# http://www.test.com/login/robin.html     ==>  http://www.test.com/reg/login.php?user=robin
locaiton /login {
    rewrite ^/login/(.*).html /reg/login.php?user=$1 permanent;
}
```



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

if (condition) { ... }

可以用if和nginx的变量来匹配一些东西，让匹配的ip或者是访问的页面做某些限制或跳转。

不支持 && 或 || 也不支持嵌套。如果需要利用 && 可以通过设置变量的方式。

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

## nginx状态码和日志

http返回状态码(Status-Code)

```text
200     成功
301     永久重定向(redirect) 
302     临时重定向(redirect) 
304     浏览器缓存
403     请求不到首页，权限被拒绝
404     资源找不到
500     服务器内部错误，程序代码错误
502     找不到后端的资源
504     请求超时
```

浏览器（客户端）访问网站携带的参数，以及服务端返回的参数

```shell
//1.概况
Request URL: http://www.52wiki.cn/static/bootstrap/css/bootstrap.min.css       # 请求的URL地址
Request Method: GET                             # 请求的方法(获取)
Status Code: 200 OK (from memory cache)         # 返回的状态
Remote Address: 61.184.215.226:80               # 请求的地址
//2.客户端请求的头部信息
Accept: butes                                   # 请求的类型
Accept-Encoding: gzip, deflate                  # 是否进行压缩
Accept-Language: zh-CN,zh;q=0.9                 # 请求的语言
Cache-Control: max-age=0                        # 缓存
Connection: keep-alive                          # TCP长连接
Host: www.52wiki.cn                             # 请求的域名
If-Modified-Since: Tue, 04 Dec 2018 09:58:20 GMT# 修改的时间
If-None-Match: "a49-56b5ce607fe00"              # 标记
Upgrade-Insecure-Requests:1                     # 在http和https之间起的一个过渡作用
User-Agent: Mozilla/5.0                         # 请求浏览器的工具
"=== 请求一个空行 ==="
"=== 请求内容主体 ==="
//3.服务端响应的头部信息
HTTP/1.1 304 Not Modified                       # 返回服务器的http协议，状态码
Date: Fri, 14 Sep 2018 09:14:28 GMT             # 返回服务器的时间
Server: Apache/2.4.6 (CentOS) PHP/5.4.16        # 返回服务器使用的软件（Apache php）
Connection: Keep-Alive                          # TCP长连接
Keep-Alive: timeout=5, max=100                  # 长连接的超时时间
ETag: "a49-56b5ce607fe00"                       # 验证客户端标记
"=== 返回一个空行 ==="
"=== 返回内容主体 ==="
```

nginx日志统计

```shell
1.根据访问IP统计UV     awk '{print $1}'  access.log|sort | uniq -c |wc -l
2.统计访问URL统计PV    awk '{print $7}' access.log|wc -l     
3.查询访问最频繁的URL  awk '{print $7}' access.log|sort | uniq -c |sort -n -k 1 -r|more    
4.查询访问最频繁的IP   awk '{print $1}' access.log|sort | uniq -c |sort -n -k 1 -r|more
```

