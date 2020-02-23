## CentOS7


**安装docker**
>yum install docker 

**安装以后最好修改镜像源，否则慢到怀疑人生**
>[阿里云官方加速]( https://cr.console.aliyun.com/#/accelerator )  
```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'

{
  "registry-mirrors": ["https://自已镜像源.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

**查看版本**  
>docker version

**查看更详细的信息(包括容器和镜像)**  
>docker info

**下载镜像 以安装tomcat为例** 
>可以先查询
docker search tomcat

拉取
>docker pull tomcat


**查看已有的镜像**  
>docker images


**创建并运行tomcat**
>docker run --pd 8888:8080 --name mytomcat tomcat
- -p 指定端口映射，格式为：主机 (宿主) 端口：容器端口
- -d 后台运行容器，并返回容器 ID；
- --name 为容器指定一个名称；
- 最后加上镜像的名字

**查看正在运行的容器**  
>docker ps
- -a 会将所有容器显示出来

**运行已经创建好的容器**  
>docker start 容器名

**停止运行中的容器**  
>docker stop 容器名

**重启容器**  
>docker restart 容器名

**进入容器目录**
>docker run -it tomcat /bin/bash
- -i 以交互模式运行容器，通常与 -t 同时使用；
- -t 为容器重新分配一个伪输入终端，通常与 -i 同时使用；

如果要进入已运行的容器
>docker exec -it 容器ID /bin/bash

**删除镜像**  
>docker rmi 镜像名

**删除容器**  
>docker rm 容器名


**查看容器日志**  
>docker logs 容器id


