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
    - 执行# `# ./configure`
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
