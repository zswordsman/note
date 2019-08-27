
# Nginx + fastcgi + php 的原理与关系

* CGI：Common Gateway Interface 公共网关接口，web服务器和脚本语言通信的一个标准、接口、协议【协议】
* FastCGI：CGI协议的升级版【协议】
* PHP-CGI: 实现了CGI接口协议的PHP脚本解析器【程序】
* PHP-FPM: 管理和调度php-cgi进程，进而实现了FastCGI接口协议的程序【程序】

问题：CGI对每个请求会parse一遍对应脚本的配置文件（如php.ini），
加载配置和扩展，初始化执行环境，性能非常差，所有有了下面的流程：
![](http://pwh5m3xnl.bkt.clouddn.com/mweb/15669180381085.jpg)
那么实现Fastcgi协议的程序，如PHP-FPM是怎么做的呢？首先，Fastcgi会先启一个master进程，解析配置文件，初始化执行环境，然后再启动多个worker进程，这个worker就是php-cgi。当请求过来时，master会传递给一个worker，然后立即可以接受下一个请求。这样就避免了重复的劳动，效率自然是高。而且当worker不够用时，master可以根据配置预先启动几个worker等着，比如20worker，当然空闲worker太多时，也会停掉一些，这样就提高了性能，也节约了资源。这就是fastcgi的对进程的管理。

-------

结合 用户对动态PHP网页访问过程来理解
第一步：用户将http请求发送给nginx服务器
第二步：nginx会根据用户访问的URI和后缀对请求进行判断
1.例如用户访问的index.php，nginx则会根据配置文件中的location进行匹配，例如：

```linux
root@json:/data/web# cat /etc/nginx/conf.d/blog.conf 
server {
    root /data/web/blog/;
    index index.html index.htm;
    server_name www.fwait.com;
    location / {
        try_files $uri $uri/ /index.html;
    }
    location /blog/ {
        #alias /usr/share/doc/;
        auth_basic "authorized users only";
        auth_basic_user_file /etc/nginx/passwd.conf;
        #autoindex on;
        allow 192.168.1.103;
        deny all;
    }
    location ~ \.php$ {
        include /etc/nginx/fastcgi_params;
        fastcgi_intercept_errors on;
        fastcgi_pass 127.0.0.1:9000;
    }

}
```
用户访问的是index.php，则会匹配到location ～ \\.php$，这个的含义是对用户通过URI访问的资源进行区分大小的匹配，并且访问的资源是以.php结尾的。
nginx根据用户请求的资源匹配到具体的location后，会执行location对应的动作，location中动作的含义是：
include /etc/nginx/fastcgi_params; #表示nginx会调用fastcgi这个接口
fastcgi_intercept_errors on; #表示开启fastcgi的中断和错误信息记录
fastcgi_pass 127.0.0.1:9000; # 表示nginx通过fastcgi_pass将用户请求的资源发给127.0.0.1:9000进行解析，这里的nginx和php脚本解析服务器是在同一台机器上，所以127.0.0.1:9000表示的就是本地的php脚本解析服务器。
根据nginx服务器的配置，可以看出，用户访问的是动态的php资源，nginx会调用php相关脚本解析程序对用户访问的资源进行解析。

第三步：通过第二步可以看出，用户请求的是动态内容，nginx会将请求交给fastcgi客户端，通过fastcgi_pass将用户的请求发送给php-fpm
如果用户访问的是静态资源呢，那就简单了，nginx直接将用户请求的静态资源返回给用户。

第四步：fastcgi_pass将动态资源交给php-fpm后，php-fpm会将资源转给php脚本解析服务器的wrapper

第五步：wrapper收到php-fpm转过来的请求后，wrapper会生成一个新的线程调用php动态程序解析服务器
如果用户请求的是需要读取例如MySQL数据库等，将会触发读库操作;
如果用户请求的是如图片/附件等，PHP会触发一次查询后端存储服务器如通过NFS进行存储的存储集群;

第六步：php会将查询到的结果返回给nginx

第七步：nginx构造一个响应报文将结果返回给用户
这只是nginx的其中一种，用户请求的和返回用户请求结果是异步进行，即为用户请求的资源在nginx中做了一次中转，nginx可以同步，即为解析出来的资源，服务器直接将资源返回给用户，不用在nginx中做一次中转。
用一张图表示  nginx fastcgi  wrapper php之间的关系:一图胜千言 
![](http://pwh5m3xnl.bkt.clouddn.com/mweb/15669171869084.jpg)


