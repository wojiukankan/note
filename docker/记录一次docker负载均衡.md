#### 环境 windows10
- 启用虚拟化和hyper-v
- 到[docker官网](https://hub.docker.com/?overlay=onboarding)可以下载windows用的客户端

#### 以centos镜像搭建一个nginx+php环境
- `docker pull centos` 下载centos镜像
- `docker run -it centos bash`启动并进入centos镜像
- 参考[centos7安装nginx](linux/centos7安装nginx.md)、[centos7安装php](linux/centos7安装php.md)进行环境搭建
- 搭建好之后执行`docker commit -a "提交镜像作者" -m "描述" 容器id 容器名:版本`，例如`docker commit -a "wojiukankan" -m "描述" e6a01fd1a674 centos:dev1`
- 此时就把搭建好的环境保留下来了，下次可以直接通过启动新镜像来运行(注意自启的服务会有异常使用`docker run -d --privileged=true wojiukankan/centos:dev1 /usr/sbin/init`启动)
- 但是此时的dev1镜像时跟原来的centos有关联的，删除dev1前无法删除centos，可以用`docker save -o centos.tar centos:dev1`先把镜像保存为tar压缩包，然后用`docker load -i ./centos.tar`加载
> 发现保存出来的镜像非常大，1G多
#### 想办法精简镜像
- 用Dockerfile进行构建
- 基础镜像用centos
```Dockerfile
    FROM centos
    RUN sh命令(以前文章写的centps安装nginx/php)
    ...
```
- FROM 指定基础镜像，RUN执行sh命令，因为dockerfile每一行代码都会添加一层，导致体积变大，所以尽量奖多个sh命令用&&连接执行，并清理掉中途下载、生成的中间文件
- 最后发现镜像也很大
- 研究了一个官方的镜像，发现有个叫alpine的linux系统，可以以他为基础构建镜像
- 同样的步骤，把centos的命令换城alpine的，对应的依赖包也是
- alpine：apk add安装 apk update更新索引 apk del删除索引
- 最终构建出来大小到了600M，还是有点大
#### 用php官方的Dockerfile进行修改
- 参考官方的Dockerfile文件修改一下php下载地址，密钥验证即可完成构建
- 构建出来的php镜像只有80M
#### 用docker-compose启动镜像