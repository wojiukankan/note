启动docker run -d  --privileged=true test/centos:v1 /usr/sbin/init


进入docker exec -it centos7 /bin/bash


    3.5保存新镜像

docker save -o nginx.tar nginx2:latest
    3.6删除新旧镜像

docker rmi fff815b9c91f b175e7467d66
    3.7load新镜像

docker load -i ./nginx.tar