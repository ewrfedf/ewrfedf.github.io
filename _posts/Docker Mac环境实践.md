# Docker 实践 Mac 环境



### [2. CMD flow](#2)

  [2.1 创建一个新的 Docker 虚拟机。](#2.1)

```sh
$ docker-machine create --driver virtualbox default		
```

[2.2 列出所有可用的 docker machine](#2.2)

```sh
$ docker-machine ls
```

[2.3 获取 default 虚拟机的环境变量](#2.3)

```sh
$ docker-machine env default
```

[2.4 连接到 default 虚拟机](#2.4)

```sh
$ eval "$(docker-machine env default)"
```

[2.5 安装集装箱程序 jenkins](#2.5) 

```sh
#-d 启动后后台运行服务、-P 自动映射 port、-p 指定port、-u="root" 指定用户
$ docker run -d -i -P --name ci jenkins 
```

[2.6 查看 IP & 端口](#2.6)

```sh
$ docker port ci   # 使用 create --name 参数获取
$ docker-machine ls # 可以看到 IP 信息
```



[2.7 停止 Container ](#2.7)

```sh
$ docker stop ci  
```

[2.8 删除 Container](#2.8)

```sh
$ docker rm ci   # 不会删除 Image rmi 会删除 Image
```

[2.9 一个容器连接到另一个容器](#2.9)

```sh
$ docker run -i -t --name sonar -d -link mmysql:db   tpires/sonar-server
sonar 
# 容器连接到mmysql容器，并将mmysql容器重命名为db。这样，sonar容器就可以使用db的相关的环境变量了。
```

### [3.配置 Docker 镜像服务](#3)

  进入 VM 安装路径 编辑 config.json文件 /Users/ ** /.docker/machine/machines/default/config.json

将第 46 行

 `  "RegistryMirror": [],`

修改为：

`"RegistryMirror": ["https://your.mirror.com"],`





### [4. 定制 Image](#4)

参考

- [使用 Dockerfile 构建镜像](http://www.widuu.com/docker/userguide/dockerimages.html)


- [Installing more tools](https://hub.docker.com/_/jenkins/)


- [安装gradle](http://www.jianshu.com/p/949a5a2ddceb)



附录

* #### docker 命令

  | 命令                             | 含义                        |
  | ------------------------------ | ------------------------- |
  | docker create                  | 会创建一个容器但是不会立刻启动           |
  | docker run                     | 会创建并且启动某个容器               |
  | docker run --rm                | 在容器运行完毕之后删除该容器            |
  | docker run -t -i               | 打开某个容器之后能够与其进行交互          |
  | docker stop                    | 关闭某个容器                    |
  | docker start                   | 启动某个容器                    |
  | docker restart                 | 重新启动某个容器                  |
  | docker rm                      | 删除某个容器                    |
  | docker rm -v                   | 移除所有与该容器相关的Volume         |
  | docker kill                    | 发送SIGKILL信号量到某个容器         |
  | docker attach                  | 附着到某个正在运行的容器              |
  | docker wait                    | 阻塞直到某个容器关闭                |
  | docker images                  | 展示所有的镜像                   |
  | docker import                  | 从原始码中创建镜像                 |
  | docker build                   | 从某个Dockfile中创建镜像          |
  | docker commit                  | 从某个Container中创建镜像         |
  | docker rmi                     | 移除某个镜像                    |
  | docker load                    | 以STDIN的方式从某个tar包中加载镜像     |
  | docker save                    | 以STDOUT的方式将镜像存入到某个tar包中   |
  | docker ps                      | 列举出所有正在运行的容器              |
  | docker ps -a                   | 展示出所有正在运行的和已经停止的容器        |
  | docker logs                    | 从某个容器中获取log日志             |
  | docker inspect                 | 检测关于某个容器的详细信息             |
  | docker events                  | 从某个容器中获取所有的事件             |
  | docker port                    | 获取某个容器的全部的开放端口            |
  | docker top                     | 展示某个容器中运行的全部的进程           |
  | docker stats                   | 展示某个容器中的资源的使用情况的统计信息      |
  | docker diff                    | 展示容器中文件的变化情况              |
  | docker history                 | 展示镜像的全部历史信息               |
  | docker tag                     | 为某个容器设置标签                 |
  | docker cp                      | 在容器与本地文件系统之间进行文件复制        |
  | ocker export                   | 将某个容器中的文件系统的内容输出到某个tar文件中 |
  | docker exec                    | 在容器中运行某个命令                |
  | docker exec -it foo /bin/bash. | 在某个名字为foo的容器中运行交互命令       |



* #### docker-machine 命令：

  | 命令               | 含义               |
  | ---------------- | ---------------- |
  | help             | 查看帮助信息           |
  | active           | 查看活动的Docker主机    |
  | config           | 输出连接的配置信息        |
  | create           | 创建一个Docker主机     |
  | env              | 显示连接到某个主机需要的环境变量 |
  | inspect          | 输出主机更新信息         |
  | ip               | 获取Docker主机地址     |
  | kill             | 停止某个Docker主机     |
  | ls               | 列出所有管理的Docker主机  |
  | regenerate-certs | 为某个主机重新成功TLS认证信息 |
  | restart          | 重启Docker主机       |
  | rm               | 删除Docker主机       |
  | scp              | 在Docker主机之间复制文件  |
  | ssh              | SSH到主机上执行命令      |
  | start            | 启动一个主机           |
  | status           | 查看一个主机状态         |
  | stop             | 停止一个主机           |
  | upgrade          | 更新主机Docker版本为最新  |
  | url              | 获取主机的URL         |



















