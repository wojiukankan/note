## 安装php
- 安装依赖 
    ```
    # yum install libxml2 libxml2-devel openssl openssl-devel bzip2 bzip2-devel libcurl libcurl-devel libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel gmp gmp-devel libmcrypt libmcrypt-devel readline readline-devel libxslt libxslt-devel
    ```
- [php官网](https://www.php.net/downloads.php)下载源码`# wget https://www.php.net/distributions/php-7.3.5.tar.gz`
- 解压`# tar -zxvf php-7.3.5.tar.gz`
- 进入文件夹`# cd php-7.3.5`
- 需要`GCC`环境，上面安装`nginx`时已经安装了
- `./configure`准备安装
- 添加以下参数：
    ```
    ./configure --prefix=/usr/local/php/ --with-mysqli --with-pdo-mysql --with-iconv-dir --with-freetype-dir --with-jpeg-dir --with-png-dir --with-zlib --with-libxml-dir --enable-bcmath --enable-soap --enable-zip --with-curl=/usr/local/bin/ --enable-fpm --with-fpm-user=www --with-fpm-group=www --enable-mbstring --enable-sockets --with-gd --with-openssl --with-mhash --disable-fileinfo --with-config-file-path=/usr/local/php/etc --enable-opcache
    ```
    参数解释：
    ```
    --with-config-file-path=/usr/local/php/etc 修改配置文件目录为/usr/local/php/etc
    --prefix=/usr/local/php 修改安装目录为/usr/local/php
    --with-mysqli           安装mysqli拓展
    --with-pdo-mysql        安装dpo拓展
    --with-iconv-dir        XMLRPC-EPI的iconv目录
    --with-freetype-dir     打开对freetype字体库的支持 
    --with-jpeg-dir         打开对jpeg图片的支持
    --with-png-dir          打开对png图片的支持
    --with-zlib             打开zlib库支持
    --with-libxml-dir       libxml2安装位置
    --enable-bcmath         打开图片大小调整,用到zabbix监控的时候用到了这个模块
    --enable-soap           期用soap支持
    --enable-zip            支持zip
    --with-curl             支持curl
    --enable-fpm            启用fpm
    --with-fpm-user=www     设置fpm的执行用户
    --with-fpm-group=www    设置fpm的执行用户组
    --enable-mbstring       启用mbstring
    --enable-sockets        启用sockets
    --with-gd               支持gd库
    --with-openssl          支持openssl
    --with-mhash            支持mhash
    --disable-fileinfo      关闭fileinfo,fileinfo在5.3以后就被默认安装的,小内存VPS上编译PHP会out of memory
    ```
- 遇到了报错`configure: error: libxml2 not found. Please check your libxml2 installation.`,先安装`libxml2`
    ```
    # yum install libxml2*
    ```
- 遇到报错`checking for cURL 7.15.5 or greater... configure: error: cURL version 7.15.5 or later is required to compile php with cURL support`,安装更新的`curl`
    - 下载`# wget https://curl.haxx.se/download/curl-7.64.1.tar.gz`
    - 解压`# tar -xzvf curl-7.64.1.tar.gz`
    - 安装
        ```
        # cd curl-7.64.1
        # ./configure
        # make
        # make install
        ```
    - 并把`./configure`中的`--with-curl`改为`--with-curl=/usr/local/bin/`(后面的路径未curl安装目录)
- 遇到报错`configure: error: jpeglib.h not found.`，需要安装`libjpeg`
    ```
    # yum install libjpeg*
    ```
- 遇到报错`configure: error: png.h not found.`，需要安装`libpng`
    ```
    # yum install libpng*
    ```
- 遇到报错`configure: error: freetype-config not found.`，需要安装`freetype`
    ```
    # yum install freetype*
    ```
- 遇到报错`configure: error: Please reinstall the libzip distribution`，需要安装`libzip`
    ```
    # yum install libzip*
    ```
- 遇到报错`checking for libzip... configure: error: system libzip must be upgraded to version >= 0.11`,发现之前`yum`安装的`libzip`版本不够，现在去[官网](https://libzip.org/download/)新版下载编译安装
    - 安装(1.5版本的`libzip`需要3.0.2以上版本的`cmake`)
    - 先安装`cmake`,[官网](https://cmake.org/download/)
        ```
        # wget https://github.com/Kitware/CMake/releases/download/v3.14.4/cmake-3.14.4.tar.gz
        # tar -xzvf cmake-3.14.4.tar.gz
        # cd cmake-3.14.4
        # ./bootstrap && gmake && gmake install
        # /usr/local/bin/cmake -version 可以看到版本
        ``` 
    - 安装`libzip`
    - 下载`# wget https://libzip.org/download/libzip-1.5.2.tar.gz`
    - 解压`# tar -xzvf libzip-1.5.2.tar.gz`
        ```
        # cd libzip-1.5.2
        # mkdir build && cd build && /usr/local/bin/cmake .. && make && make install
        ```
- 遇到报错`configure: error: off_t undefined; check your library configuration`
    ```
    添加搜索路径到配置文件
    # echo '/usr/local/lib64
    /usr/local/lib
    /usr/lib
    /usr/lib64'>>/etc/ld.so.conf
    更新配置
    # ldconfig -v
    ```
- 执行`# make && make install`安装php
- `php-ini`:源码包里面有配置文件:
    - `php.ini-development`测试开发环境
    - `php.ini-production`生产环境
- 复制一份到指定的目录下（根据自己的情况选用）:
    - `# cp php.ini-production /usr/local/php/etc/php.ini`
    - 编译参数写的配置文件目录在`/usr/local/php/etc/php.ini`,但实际上他每次都会去`/usr/local/lib/php.ini`找配置文件，不知道什么情况。
    - `# cp php.ini-production /usr/local/lib/php.ini`
- `php-fpm`配置
    - 复制一份新的`php-fpm`配置文件:
    - `# cd /usr/local/php/etc`
    - `# cp php-fpm.conf.default php-fpm.conf`
    - `# vim php-fpm.conf`
    - `# error_log=/usr/local/php/var/php-fpm.log`配置错误日志
    - `# pid=/usr/local/php/var/run/php-fpm.pid`配置pid文件
    - 保存退出
    - `# cd /usr/local/php/etc/php-fpm.d`
    - `# cp www.conf.default www.conf`
- 管理`php-fpm`配置:	
    - `# cd ~/php-7.3.5`
    - `# cp ./sapi/fpm/php-fpm.service /usr/lib/systemd/system/`下
- 配置开机启动`php-fpm`:
    ```
	systemctl enable php-fpm 开机启动启动php-fpm
	systemctl start php-fpm  启动php-fpm
    systemctl status php-fpm 查看状态
    systemctl stop php-fpm   停止php-fpm
    systemctl restart php-fpm   停止php-fpm
    ```
- 添加环境变量:
    - `# vim /etc/profile`
    - 在末尾追加:
    - `export PATH=$PATH:'/usr/local/php/bin/'`
    - 保存退出。
    - `# source /etc/profile`
    - 测试:`# php -v`
- 能显示出`php`版本就表示已经成功了。
- 如果需要区分`web`和`cli`环境，可以将`/usr/local/php/etc/php.ini` 复制一份，重命名为`php-cli.ini`
- `cp /usr/local/php/etc/php.ini/usr/local/php/etc/php-cli.ini`
- 遇到报错`ERROR: [pool www] cannot get uid for user 'www'`,创建`www`用户
- `# useradd www`
[转载](https://blog.csdn.net/mrtwenty/article/details/80458264 )


