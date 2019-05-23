## centos7安装nginx
> [转载于挨踢农民工](http://www.zhaoweihua.cn/article/30/nginx1.11-centos7.html)

- [官网下载安装包](http://nginx.org/en/download.html)，选择适合Linux的版本，这里选择最新的版本，下载到本地后上传到服务器或者centos下直接wget命令下载。
- 下载软件包`# wget https://nginx.org/download/nginx-1.16.0.tar.gz`
- 安装nginx
    - 先执行以下命令，安装nginx依赖库,如果缺少依赖库，可能会安装失败，具体可以参考文章后面的错误提示信息。
    ```
    # yum install gcc-c++
    # yum install pcre
    # yum install pcre-devel
    # yum install zlib 
    # yum install zlib-devel
    # yum install openssl
    # yum install openssl-devel
    ```
    - 解压安装包`# tar -zxvf nginx-1.16.0.tar.gz`
    - nginx被解压到了`~/nginx-1.16.0` 目录下（不要把压缩包解压到`/usr/local/nginx`目录下，因为nginx会默认安装`到/usr/local/nginx`目录下）
    - 切换到`~/nginx-1.16.0`目录`# cd nginx-1.16.0`
    - 执行# `# ./configure`,不附带任何参数编译，详细参数说明见文末
    - 该操作会检测当前系统环境，以确保能成功安装`nginx`，执行该操作后可能会出现以下几种提示：
    
        ```
        checking for OS
         + Linux 3.10.0-123.el7.x86_64 x86_64
        checking for C compiler ... not found
        ./configure: error: C compiler cc is not found
        ```
        - 如果出现以上错误提示信息，执行`# yum install gcc-c++`安装`gcc`，
        ```
        ./configure: error: the HTTP rewrite module requires the PCRE library.
        You can either disable the module by using --without-http_rewrite_module
        option, or install the PCRE library into the system, or build the PCRE library
        statically from the source with nginx by using --with-pcre=<path> option.
        ```
        - 如果出现上面提示，执行`# yum install pcre`,`# yum install pcre-devel`安装`PCRE`,
        ```
        ./configure: error: the HTTP gzip module requires the zlib library.
        You can either disable the module by using --without-http_gzip_module
        option, or install the zlib library into the system, or build the zlib library
        statically from the source with nginx by using --with-zlib=<path> option.
        ```
        - 如果出现以上提示，执行`# yum install zlib`,`# yum install zlib-devel`安装`zlib`,
        - 如果没有出现`./configure: error`提示，表示当前环境可以安装`nginx`
    - 执行`# make`和`# make install`编译`nginx`
    - 没有出错的话，表示`nginx`已经成功安装完成，默认安装位置为`/usr/local/nginx`。
    - 如果出现`cp: 'conf/koi-win' and '/usr/local/nginx/conf/koi-win' are the same file`，可能是你把安装包解压到了`/usr/local/nginx`目录，解决办法是将该目录重命名为其他名称后再执行`make,make install`.
- 配置`nginx`开机启动
    - 切换到`/lib/systemd/system/`目录，创建`nginx.service`文件`vim nginx.service`
    ```
    # cd /lib/systemd/system/
    # vim nginx.service
    ```
    文件内容如下：
    ```
    [Unit]
    Description=nginx 
    After=network.target 
       
    [Service] 
    Type=forking 
    ExecStart=/usr/local/nginx/sbin/nginx
    ExecReload=/usr/local/nginx/sbin/nginx reload
    ExecStop=/usr/local/nginx/sbin/nginx quit
    PrivateTmp=true 
       
    [Install] 
    WantedBy=multi-user.target
    ```
    退出并保存文件，执行`# systemctl enable nginx.service`使`nginx`开机启动
    - `# systemctl start nginx.service`      启动nginx
    - `# systemctl stop nginx.service`       结束nginx
    - `# systemctl restart nginx.service`    重启nginx
    - `# systemctl status nginx.service`     nginx状态
- 验证是否安装成功，输入`http://服务器IP/`如果能看到nginx的界面，就表示安装成功了
- 有可能需要开放80端口
    - `# firewall-cmd --zone=public --add-port=80/tcp --permanent`
    - `# firewall-cmd  --reload`
- `nginx`解析`php`,[转载](https://juejin.im/post/58db7d742f301e007e9a00a7)
    - 打开`nginx`配置文件
    - `# vim /usr/local/nginx/conf/nginx.conf`
    - 在最后加入`include /usr/local/nginx/conf/server/*;`，意思引入`/usr/local/nginx/conf/server/`下的所有文件，我们的虚拟主机配置就放到这里面
    - 切换到目录`# cd /usr/local/nginx/conf/server/`
    - 新建文件`# vim test_com.conf`,写入
        ```
        server {
            listen 80;                          #监听80端口
            server_name test.com;               #虚拟主机域名
            root /usr/local/nginx/html/test_com;#代码路径
    
            location / {
                    index index.php;            #跳转到test.com/index.php
                    autoindex on;
            }
    
            # 正则匹配到.php后缀的文件，代理到php-fpm
            location ~ \.php$ {
                    include /usr/local/nginx/conf/fastcgi.conf;  #加载nginx的fastcgi模块
                    fastcgi_intercept_errors on;
                    fastcgi_pass 127.0.0.1:9000;                 #nginx fastcgi进程监听的IP地址和端口
            }
        }
        ```
## 编译参数说明,转载自[官网](http://nginx.org/en/docs/configure.html)


参数名 | 作用 | 默认值
---|---|---
--prefix=path | nginx安装目录，下文的prefix均为此目录 | /usr/local/nginx
--sbin-path=path | nginx二进制文件路径 | prefix/sbin/nginx
--modules-path=path | nginx模块目录 | prefix/modules
--conf-path=path | 配置文件路径，可以使用`-c file`手动指定路径 | prefix/conf/nginx.conf
--error-log-path=path | 错误日志位置，可在配置文件中修改 | prefix/logs/error.log
--pid-path=path | nginx pid路径，可在配置文件中修改| prefix/logs/nginx.pid
--lock-path=path | 锁文件位置，可在配置中需改 | prefix/logs/nginx.lock
--user=name | 设置进程所有者，可修改 | nobody
--group=name | 设置用户组，可修改 |
--build=name | 设置nginx构建名称
--builddir=path | 设置构建目录
--with-select_module | 启用select模块|
--without-select_module | 禁用select模块|
--with-poll_module | 启用poll模块
--without-poll_module| 禁用poll模块
--with-threads|允许使用线程池
--with-file-aio|允许在FreeBSD和Linux上使用异步文件I/O(AIO)
--with-http_ssl_module|允许https，需要OpenSSL支持
--with-http_v2_module|允许支持HTTP2的模块
--with-http_realip_module|允许改变客户端请求头中客户端IP地址值
--with-http_addition_module|允许在响应内容中增加文本
--with-http_xslt_module </br>--with-http_xslt_module=dynamic|使用多个 XSLT stylesheets（样式表）将xml相应进行相应变换的过滤器模块,需要libxml2和libxslt库。
--with-http_image_filter_module </br> --with-http_image_filter_module=dynamic|允许转换JPEG，GIF，PNG和WebP格式的图像
--with-http_geoip_module <br> --with-http_geoip_module=dynamic | 使用预编译的MaxMind数据库解析客户端IP地址
--with-http_sub_module | 允许将一个指定的字符串替换为另一个字符串来修改响应
--with-http_dav_module | 启用对WebDav协议的支持
--with-http_flv_module | 为Flash Video（FLV）文件提供伪流服务器端支持
--with-http_mp4_module | 为MP4文件提供伪流服务器端支持
--with-http_gunzip_module | 使用“ Content-Encoding: gzip”解压缩响应
--with-http_gzip_static_module | 允许使用“.gz”文件扩展名发送预压缩文件
--with-http_auth_request_module | 允许通过子请求方式实现客户端授权
--with-http_random_index_module | 以斜杠字符`/`结尾的请求选择目录中的随机文件作为索引文件
--with-http_secure_link_module| 用于检查请求链接的真伪
--with-http_degradation_module | 启用ngx_http_degradation_module模块
--with-http_slice_module |将请求拆分为子请求，每个模块都返回一定范围的响应
--with-http_stub_status_module|提供对基本状态信息的访问
--without-http_charset_module|禁止指定的字符集添加到“Content-Type”响应头字段
--without-http_gzip_module|禁用压缩 HTTP服务器响应，需要zlib库
--without-http_ssi_module|禁用响应中处理SSI（服务器端包含）命令
--without-http_userid_module|禁用标示客户端设置合适的cookies
--without-http_access_module|禁用限制对某些客户端地址的访问。
--without-http_auth_basic_module|禁用通过使用“HTTP基本身份验证”协议验证用户名和密码来限制对资源的访问。
--without-http_mirror_module|禁用通过创建后台镜像子请求来实现原始请求的镜像
--without-http_autoindex_module|禁用处理以斜杠字符` /`结尾的请求时，生成目录列表
--without-http_geo_module| 禁用根据客户端IP地址生成变量。
--without-http_map_module|禁用根据其他变量值的值创建变量
--without-http_split_clients_module|禁止创建适合A/B测试的变量
--without-http_referer_module|不阻止对“Referer”头字段中具有无效值的请求访问站点
--without-http_rewrite_module|禁止伪静态，需要pcre库
--without-http_proxy_module|禁止代理
--without-http_fastcgi_module|禁用FastCGI
--without-http_uwsgi_module|禁用ngx_http_uwsgi_module
--without-http_scgi_module|禁用ngx_http_scgi_module模块
--without-http_grpc_module|禁用ngx_http_grpc_module模块。
--without-http_memcached_module|禁用memcached
--without-http_limit_conn_module|不限制每个密钥的连接数，例如，来自单个IP地址的连接数
--without-http_limit_req_module|不限制每个密钥的请求处理速率，例如，来自单个IP地址的请求的处理速率
--without-http_empty_gif_module|禁止透明gif。
--without-http_browser_module|禁止获取“User-Agent”
--without-http_upstream_hash_module|禁用构建实现散列负载平衡方法的模块 
--without-http_upstream_ip_hash_module|禁用构建实现ip_hash 负载平衡方法的模块
--without-http_upstream_least_conn_module|禁用构建实现least_conn 负载平衡方法的模块
--without-http_upstream_keepalive_module|禁用到上游服务器的连接缓存。
--without-http_upstream_zone_module|禁用将上游组的运行时状态存储在共享内存区域中。
--with-http_perl_module </br> --with-http_perl_module=dynamic|开启perl模块
--with-perl_modules_path=path|设置perl模块目录
--with-perl=path|设置perl二进制文件路径
--http-log-path=path|http日志路径|prefix/logs/access.log
--http-client-body-temp-path=path|保存客户端请求主体的临时文件的目录可修改|prefix/client_body_temp
--http-proxy-temp-path=path|存储临时文件和从代理服务器接收的数据的目录，可修改|prefix/proxy_temp
--http-fastcgi-temp-path=path|FastCGI服务器接收的数据的临时文件目录，可修改|prefix/fastcgi_temp
--http-uwsgi-temp-path=path|从uwsgi服务器接收的数据的临时文目录，可修改|prefix/uwsgi_temp
--http-scgi-temp-path=path|SCGI服务器接收的数据目录，可修改|prefix/scgi_temp
--without-http|禁用HTTP服务器
--without-http-cache|禁用HTTP缓存
--with-mail </br> --with-mail=dynamic|启用POP3/IMAP4/SMTP邮件代理服务器
--with-mail_ssl_module|将SSL/TLS协议支持添加到邮件代理服务器,需要OpenSSL库
--without-mail_pop3_module|禁用邮件代理服务器中的POP3协议
--without-mail_imap_module|禁用邮件代理服务器中的IMAP协议
--without-mail_smtp_module|禁用邮件代理服务器中的SMTP协议
--with-stream </br> --with-stream=dynamic|允许通过TCP/UDP代理和负载平衡
--with-stream_ssl_module|为上述添加SSL/TLS协议支持。需要OpenSSL库
--with-stream_realip_module|将客户端地址更改为PROXY协议头中发送的地址
--with-stream_geoip_module </br> --with-stream_geoip_module=dynamic|根据客户端IP地址和预编译的MaxMind数据库创建变量
--with-stream_ssl_preread_module|允许从ClientHello消息中提取信息而不终止SSL/TLS
--without-stream_limit_conn_module|不限制每个密钥的连接数，例如，来自单个IP地址的连接数。
--without-stream_access_module| 不限制对某些客户端地址的访问
--without-stream_geo_module|不能通过客户端IP地址的值获取变量
--without-stream_map_module|不能根据其他变量的值创建值
--without-stream_split_clients_module|禁止为A/B测试创建变量
--without-stream_return_module|禁止将一些指定值发送到客户端，然后关闭连接
--without-stream_upstream_hash_module|禁用实现散列负载平衡
--without-stream_upstream_least_conn_module|禁用least_conn负载平衡
--without-stream_upstream_zone_module|不将上游组的运行时状态存储在共享内存区域中
--with-google_perftools_module|使用Google Performance Tools分析nginx工作进程，该模块适用于nginx开发人员
--with-cpp_test_module|可以构建 ngx_cpp_test_module模块。
--add-module=path|外部模块路径
--add-dynamic-module=path|启用外部动态模块路径
--with-compat|实现动态模块兼容性
--with-cc=path|设置C编译器的路径
--with-cpp=path|设置C预处理器的路径
--with-cc-opt=parameters|这是额外的参数将被添加到CFLAGS变量
--with-ld-opt=parameters|设置附加的参数，链接系统库
--with-cpu-opt=cpu|期用按指定cpu生成
--without-pcre|禁用PCRE库的使用
--with-pcre|强制使用PCRE库
--with-pcre=path|设置PCRE库源的路径，ngx_http_rewrite_module 模块中的正则表达式支持需要该库
--with-pcre-opt=parameters|为PCRE设置其他构建选项。
--with-pcre-jit|使用“即时编译”支持（1.1.12，pcre_jit指令）构建PCRE库
--with-zlib=path|设置zlib库源的路径。ngx_http_gzip_module模块需要该库
--with-zlib-opt=parameters|为zlib设置其他构建选
--with-zlib-asm=cpu|使得能够使用指定的CPU中的一个优化的zlib汇编源程序： pentium，pentiumpro
--with-libatomic|强制libatomic_ops库使用
--with-libatomic=path|设置libatomic_ops库源的路径
--with-openssl=path|设置OpenSSL库源的路径
--with-openssl-opt=parameters|为OpenSSL设置其他构建选项
--with-debug|启用调试日志
