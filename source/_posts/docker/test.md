---
title: docker的基本使用
date: 2022-06-29 10:08:14
categories:
- docker
tags: 
- docker
---
# docker 容器
## 一 docker安装

```
#1、yum源 更新到最新版本
yum update
#2、安装需要的软件包，yum-util提供yum-config-manager功能，另外两个是devicemapper驱动依赖的
yum install -y yum-utils device-mapper-persistent-data 1vm2
#3、设置yum源
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
#4、安装docker,出现输入的界面都按y
yum install -y docker-ce
#5、查看docker版本，验证是否验证成功
docker -v
```
## 二 配置docker镜像
默认情况下docker会从 [docker hub](https://hub.docker.com) 拉取镜像 会特别慢
一般采用配置镜像加速器 （阿里云的镜像加速器）

```
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://9vpg9z3m.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

## 三 docker服务相关命令
### 3.1 启动docker服务
``` 
    systemctl start  docker
```

### 3.2 查看docker服务状态
```
systemctl status  docker
```


### 3.3 停止docker服务
```
systemctl stop  docker
```

### 3.4 开机自动启动docker服务
```
systemctl enable  
```

### 3.5 重启docker服务
```
systemctl restart  docker
```

## 四 镜像相关命令
### 4.1 查看本地所有镜像
```
docker images
```

### 4.2 查看本地所有镜像的id
```
docker images -q
```

### 4.3 从网络中寻找镜像
```
docker search reids
```

### 4.4 拉取镜像
```
docker pull redis:版本号  版本号可以在docker hub上查找
``` 

### 4.5 删除镜像
```
dcoker rmi 镜像id  删除指定id的镜像
docker rmi `docker images -q` 删除所有本地镜像
```

## 五 容器命令

### 5.1 创建容器
```
docker run 参数
    -i 保持容器运行 
    -t 为容器分配一个伪终端
    -d 在后台运行
    -it 交互式容器  使用exit 退出容器时会关闭容器 
    -id 守护式容器 使用docker exec 进入容器 并且退出不会关闭容器
    -- name 为创建的容器起一个名称
```

``` 
docker run -it --name=c1 tomcat /bin/bash
 ```

### 5.2 进入容器 可以通过name 或者id 进入容器
```
docker exec -it name /bin/bash
 ```

### 5.3 查看正在运行的容器
``` 
docker ps 
docker ps -a 查看所有容器
```

### 5.4 启动容器
``` 
docker start name
 ```

### 5.5 关闭容器
```
 docker stop name
 ```

### 5.5 删除容器
``` 
docker rm name
docker rm `docker ps -aq` 删除所有容器 
```

### 5.6 查看容器信息
```
docker inspect name
```


## 六 数据卷
### 6.1 数据卷的概念
- 数据卷是宿主机中的一个目录或文件
- 当容器目录和数据卷目录绑定后，对方的修改会立即同步
- 一个数据卷可以被多个容器同时挂载
- 一个容器也可以被挂载多个数据卷
 

### 6.2 配置数据卷
```
docker run ... -v 宿主机目录:容器中目录
    1.目录必须是绝对路径
    2.如果目录不存在，会自动创建
    3.可以挂载多个数据卷
    
docker run -id --name=c1 -v /root/data:/root/data_container  tomcat /bin/bash 
```


## 七 应用部署
### 7.1 部署mysql
1.拉取mysql镜像
``docker pull mysql``
2.创建容器 设置端口映射  和 目录映射
```
mkdir /root/mysql
docker run -id -p 3306:3306 --name mysql -v /root/mysql/conf:/etc/mysql/conf.d -v /root/mysql/logs:/logs -v /root/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 mysql
```
3. 如果使用sqlyoung连接docker中的mysql需要修改加密规则
```
1. 进入docker容器
docker exec -it mysql /bin/bash
2. 进入容器中的mysql
mysql -uroot -p123456
3. 修改加密规则
alter user 'root'@'%' identified with mysql_native_password by '123456';
flush privileges;

```

### 7.2 部署tomcat
1. 拉取tomcat镜像
`` docker pull tomcat``
2.创建容器 设置端口映射  和 目录映射
```
mkdir /root/tomcat
docker run -id  --restart=always --name=tomcat -p 8080:8080 -v /root/tomcat:/usr/local/tomcat/webapps tomcat
```

### 7.3 部署ngnix
1. 拉取ngnix镜像
`` docker pull nginx``
2.创建容器 设置端口映射  和 目录映射
```
mkdir /root/nginx 
cd /root/nginx
mkdir conf
cd /root/nginx/conf
vim nginx.conf
```
3. 在/root/tomcat中创建ROOT文件作为项目的根目录


### 7.4 部署redis
1. 拉取redis镜像
`` docker pull redis``
2.创建容器 设置端口映射
``docker run --restart=always  -id --name=redis -p 6379:6379 redis``

### 7.5  部署rabbitmq
1. 拉取rabbitmq 镜像 (management 带有web控制台)
    `` docker pull rabbitmq:management`` 
2. 创建容器 设置端口映射
```
#默认情况下就是guest
docker run -id  --restart=always --name rabbitmq  -p 5672:5672 -p 15672:15672 -v /root/rabbitmq:/var/lib/rabbitmq  rabbitmq:management


5672：应用访问端口；15672：控制台Web端口号


# 可以指定登录的用户名和密码
docker run -id -p 15672:15672  -p  5672:5672  -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=123456 --name rabbitmq --hostname=rabbitmqhostone  rabbitmq 
```

### 7.6 部署nginx
- **拉取nginx镜像**
    ```docker
    docker pull nginx
    ```

- **创建nginx容器**
  ```docker
  docker run  --restart=always --name nginx -p 80:80 -d nginx
  ```
 
- **从nginx容器中映射核心文件** 
> 1.创建挂载目录
  ```shell
    mkdir -p /root/nginx/{conf,log,html}
  ```
  
> 2.拷贝nginx容器对应的文件默认配置
  ```shell
    docker cp nginx:/etc/nginx/nginx.conf /root/nginx/conf/nginx.conf
    docker cp nginx:/etc/nginx/conf.d /root/nginx/conf/conf.d
    docker cp nginx:/usr/share/nginx/html /root/nginx/
  ```

>3. 停止并删除nginx容器
 ```docker
   docker stop nginx 
   docker rm nginx
 ```
- **重新启动nginx镜像重新新容器**
 ```shell
    docker run -d --name nginx -p 80:80 \
    -v /root/nginx/conf/nginx.conf:/etc/nginx/nginx.conf \
    -v /root/nginx/conf/conf.d:/etc/nginx/conf.d \
    -v /root/nginx/log:/var/log/nginx \
    -v /root/nginx/html:/usr/share/nginx/html \
    --privileged=true nginx

 ```
 ## 7.7 部署nacos
 - 拉取镜像
 > docker pull nacos/nacos-server
 - 创建容器
 > docker run -d -e prefer_host_mode=127.0.0.1 -e MODE=standalone -v /nacos/logs:/opt/software/nacos/logs -p 8848:8848 --name nacos --restart=always nacos/nacos-server

## 八 dockerfile
### 8.1 将容器转为镜像
```
docker commit 容器id 镜像名称:版本号   # 将容器转成镜像 在转为镜像的时候是不会将挂在目录下的文件转为镜像的

docker save -o 压缩文件名称 镜像名称:版本号 # 将镜像变成压缩文件

docker load -i 压缩文件名称  # 将压缩文件在转为镜像

```

### 8.2 dockerfile 
```
使用dockerfile 自定义镜像 启动springboot项目
1.创建一个文件  vim springboot_dockerfile 输入下面内容

FROM java:8
MAINTAINER xyh<2773604834@qq.com>
ADD spring-boot-hello-0.0.1-SNAPSHOT.jar app.jar
CMD java -jar app.jar    
    
2. 将dockerfile 文件编译成一个镜像文件
docker build -f ./springboot_dockerfile  -t app .

3. 使用编译好的镜像文件 创建容器
docker run -id -p 9000:9000  app
    
```

## 九 docker compose 
### 9.1 docker compose 服务编排
Docker Compose是一个编排多容器分布式部署的工具，提供命令集管理容器化应用的完整开发周期，包括服务构建,启动和停止。
- 利用Dockerfile定义运行环境镜像
- 使用docker-compose,ym定义组成应用的各服务
- 运行docker-compose up启动应用