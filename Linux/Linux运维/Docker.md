## 学习目标

* Docker概述
* Docker安装
* Docker命令
* Docker镜像
* 容器数据卷
* Docker File
* Docker网络
* IDEA整合Docker
* Docker Compose
* Docker Swarm
* CI\CD Jenkins



## Docker概述

项目能不能带上环境安装打包！

环境配置是十分麻烦的，为了不重复配置环境，所以出现了Docker。

传统：开发人员开发jar包，运维人员部署生产环境

现在：开发打包（带上环境）部署上线，一套流程做完！



* 隔离：Docker核心思想！打包装箱！每个箱子之间相互隔离的！



比较Docker和虚拟机技术的不同：

* 传统虚拟机，虚拟出一套硬件，运行一个完整的操作系统，然后在操作系统上安装和运行软件
* 容器内的应用之间运行在宿主机上，容器没有自己的内核，也没有虚拟我们的硬件，所以轻便
* 每个容器间是互相隔离，每个容器内都有一个属于自己的文件系统，互不影响。



**优点：**

* 应用更快速的交付和部署
* 更便捷的升级和扩容
* 更简单的系统运维
* 更高效的计算资源利用



## Docker安装



### 名词概念

* **镜像（image）：**

  docker镜像就好比是一个模板，可以通过这个模板来创建容器服务，镜像>run>容器。

* **容器（container）：**

  Docker利用容器技术，独立运行一个或者一组应用，通过镜像来创建。

  启动，停止，删除，基本命令！

  目前就可以把这个容器理解为就是一个简易的Linux系统

* **仓库（repository）：**

  仓库就是存放镜像的地方！

  仓库分为共有仓库和私有仓库！

  Docker Hub（默认是国外的）

  阿里云服务器（配置镜像加速）！



### 安装Docker

> 环境准备：centos 7

```bash
[root@iz2ze17cdsoswya7zqq2lxz ~]# uname -r
3.10.0-693.2.2.el7.x86_64
```



开发文档：

```bash
# 1.卸载旧版本Docker
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
                  
# 2.需要的安装包
$ sudo yum install -y yum-utils

# 3.设置镜像地址（阿里云）
$ sudo yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
    
# 非必要步骤，更新yum软件包索引
$ yum makecache fast

# 4.安装最新docker docker-ce 社区
$ sudo yum install docker-ce docker-ce-cli containerd.io

# 5.启动docker
$ sudo systemctl start docker

# 6.运行HelloWord
$ sudo docker run hello-world
```



**阿里云镜像加速**

在阿里云服务中找到容器镜像服务，开通按步骤配置。



### 回顾HelloWord执行流程



* 开始
* Docker会在本机寻找镜像
  * 找到并允许此镜像
  * 没有找到去DockerHub上下载
    * 在DockerHub上找到下载
    * 找不到返回错误



### 底层原理

**Docker是怎么工作得？**

Docker是一个cs结构得系统，Docker得守护进程运行在主机上。通过Socket客户端访问！

DockerServer接收到DockerClient得指令，对Docker容器执行命令！



![1589798518166](J:\桌面\文章\文章图片\1589798518166.png)



**Docker为什么比VM快？**

1. Docker有着比虚拟机更少得抽象层。

2. docker利用得宿主机得内核，vm需要GuestOS。

   新建一个docker容器，不需要像虚拟机一样重新加载一个操作系统内核。





### 容器命令

有了镜像才可以创建容器！！！

**新建容器并启动**

```shell
docker run [可选项] image

# 可选项
--name="name"	#容器命名，区分容器
-d				#后台方式运行
-it				#使用交互方式运行，进入容器查看内容
-p				#指定容器的端口 -p 8080:8080
	-p 主机端口:容器端口 （常用）
	-p ip:主机端口:容器端口
	-p 容器端口
	容器端口
-p				#随机指定端口

# 例子
docker run -it centos /bin/bash	#启动并进入容器
exit	#退出容器
```



**查看容器**

```shell
docker ps	#查看正在运行的容器
docker ps -a #查看全部（包括曾经）运行的容器
docker ps -a -n=1 #就查看最近的一个
docker ps -q # 只显示容器编号
```



**退出容器**

```shell
exit	#退出并停止容器
Ctrl +P +Q #退出不停止容器
```



**删除容器**

```shell
docker rm 容器id					#删除指定容器
docker rm -f $(docker ps -aq)	 #删除全部，运行中的不可以删除，强制删除需要加-f
```



**启动和停止容器**

```shell
docker start 容器id		# 启动容器
docker restart 容器id		# 重容器
docker stop 容器id		# 停止当前正在运行的容器
docker kill 容器id		# 停止当前正在运行的容器
```



### 其它命令

**后台启动容器**

```bash
# 命令 docker run -d 镜像名
docker run -d centos

# 问题docker ps，发现centos停止了

#常见的坑：docker容器使用后台运行，就必须有一个前台进程，docker发现没有应用，就会自动停止
```



**查看日志**

```bash
docker logs -f -t --tail 数字 容器id

# 例子
docker run -d centos /bin/sh -c "while true;do echo hello;sleep 1;done"
docker logs -tf -tail 10 dce7b86171bf

# -tf	#时间戳显示日志
# --tail number	#显示条数
```



**查看容器中的进程信息**

```bash
docker top 容器id
```



**查看镜像元数据**

```bash
docker inspect 容器id
```



**进入当前正在运行的容器**

```bash
docker exec -it 容器id /bin/bash  #方式一，开启新的终端
docker attach 容器id /bin/bash #方式二，进入正在执行的终端
```



**从容器内拷贝文件到主机上**

```bash
docker cp 容器id:容器内路径 目的路径
```



## 可视化

* portainer（学习使用）

  ```bash
  docker run -d -p 8088:9000 --restart=always -v /var/run/docker.sock:/var/run/docker.sock --privileged=true portainer/portainer
  ```

* Rancher（CI/CD再用）



**什么是portainer？**

Docker图形化界面管理工具！提供一个后台面板供我们操作！

```bash
docker run -d -p 8088:9000 --restart=always -v /var/run/docker.sock:/var/run/docker.sock --privileged=true portainer/portainer
```

测试外网，访问8088端口！！！



平时不会用，测试完即可。。。





## Docker镜像

### 镜像是什么

镜像是一种轻量级，可执行的独立软件包，用来打包软件运行环境和基于运行环境开发的软件，它包含运行某个软件的所有内容，包括代码，运行库，环境变量和配置文件。

所有的应用，直接打包成docker镜像，就可以直接跑起来！

如何得到镜像：

* 从远程仓库下载
* 朋友拷贝
* 自己制作一个镜像DockerFile





### Docker镜像加载原理



> UnionFS（联合文件系统）

下载的时候可以看出一层层的效果！！！

UnionFS (联合文件系统) : Union文件系统( UnionFS )是一种分层、轻量级并且高性能的文件系统，它支持对文件系统的修改作为一次提交来一层层的叠加，同时可以将不同目录挂载到同一个虚拟文件系统下(unite several directories into a single virtual filesystem)。Union 文件系统是Docker镜像的基础。镜像可以通过分层来进行继承，基于基础镜像(没有父镜像) ， 可以制作各种具体的应用镜像。

特性: 一次同时加载多个文件系统，但从外面看起来，只能看到一个文件系统，联合加载会把各层文件系统叠加起来，这样最终的文件系统会包含所有底层的文件和目录。



> Docker镜像加载原理

docker的镜像实际上由一层一层的文件系统组成,这种层级的文件系统UnionFS.

bootfs(boot file system)主要包含bootloader和kernel, bootloader主要是引导加载kernel, Linux刚启动时会加载bootfs文件系统,在Docker镜像的最底层是bootfs.这一层与我们典型的Linux/Unix系统是-样的,包含boot加载器和内核。当boot加载完成之后整个内核就都在内存中了,此时内存的使用权已由bootfs转交给内核,此时系统也会卸载bootfs.

rootfs (root file system) , 在bootfs之上。包含的就是典型Linux系统中的/dev, /proc, /bin, /etc等标准目录和文件。rootfs就是各种不同的操作系统发行版,比如Ubuntu , Centos等等。



为什么我们平时安装的Centos都好几个G，而Docker里的才几百M？

对于一个精简的OS，rootfs可以很小，只需要包含最基本的命令，工具和程序库就可以了，因为底层直接用Host的kernel，自己只需要提供rootfs就可以了。由此可见对于不同发行版本的Linux，bootfs基本是一致的，rootfs会有差别，因此不同发行版本可以公用bootfs。





### 分层原理

> 分层的镜像

下载镜像时，我们可以发现，是一层一层的在下载的！！！



**理解：**

所有的Docker镜像都起始于一个基础镜像层，当进行修改或增加新的内容时，就会在当前镜像层之上，创建新的镜像层。



举一个简单的例子，假如基于centos创建一个新的镜像，这就是镜像的第一层；如果在该镜像中添加Python包，就会在基础镜像之上创建第二个镜像层；如果继续添加一个安全补丁，就会创建第三个镜像层。



镜像层是所有镜像的组合，比如说一个镜像有6个文件，前五个可能来至上一层又或者同一层的某个文件被新文件替换。。。



你只需要理解，一个镜像的文件，可能是多个镜像部分文件的组合，这样可以充分利用空间！



> 特点

Docker镜像默认都是只读的，当容器启动时，一个新的可写成被加载到镜像的顶部！

这一层就是我们通常说的容器层，容器层之下都叫镜像层！

![1589901814432](J:\桌面\文章\文章图片\1589901814432.png)



### commit镜像

```bash
docker commit #提交容器成为一个新的副本

docker commit -m="提交描述" -a="作者" 容器id 目标镜像名:[TAG]

# 提交之后查看
docker images

```





## 容器数据卷

### 什么是容器数据卷

如果数据都在容器中，那么我们容器一删除，数据就会丢失！

==需求：数据可以持久化！==

容器之间可以有一个数据共享的技术！Docker容器中产生的数据，同步到本地！

这就是卷技术！目录的挂载，将我们容器内的目录，挂载到Linux上面！



**总结：容器的持久化和同步操作（双向绑定）！容器间也是可以数据共享的！**



### 使用数据卷

> 方式一：直接使用命令挂载  -v

```bash
docker run -it -v 主机目录:容器内的目录
```

即使容器没有启动，也可以修改文件内容，容器启动后自动更新。



**实战：安装MySQL**

思考：MySQL的数据持久化问题！

```bash
# 1.拉取镜像
docker pull mysql:5.7

# 2.启动镜像进行数据挂载
docker run -d -p 3344:3306 -v /home/mysql/comf:/etc/mysql/conf.d -v /home/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 --name mysql01 mysql:5.7

# 3.开启端口防火墙并远程连接测试

# 4.即使把容器删了，数据也不会丢失
```



### 具名和匿名挂载

```bash
# 匿名挂载
docker run -d -P --name nginx01 -v /etc/nginx nginx #不指定主机目录

# 查看所有volume
docker volume ls

# 具名挂载
docker run -d -P --name nginx01 -v test:/etc/nginx nginx #指定名称而不是目录

# 查看具体目录
docker volume inspect test
```

所有docker容器内的卷，没有指定目录的情况下都是在==/var/lib/docker/volumes/xxx/_data==

我们通过具名挂载可以方便的找到我们的一个卷，大多数情况下使用具名挂载



```bash
# 如何确定是具名挂载还是匿名挂载，还是指定路径挂载！
-v 容器内路径			#匿名挂载
-v 卷名:容器内路径		  #具名挂载
-v /宿主机路径:容器内路径 #指定路径挂载
```



扩展：

```bash
# 通过 -v 容器内路径:ro rw 改变读写权限
ro readonly #只读
rw readwrite #可读可写

# 一旦设置了容器权限，容器对我们挂载出来的内容就有限定了！
docker run -d -P --name nginx01 -v test:/etc/nginx:ro nginx
docker run -d -P --name nginx01 -v test:/etc/nginx:rw nginx

# ro 只要看的ro就说明这个路径只能通过宿主机来操作，容器内部是无法操作的！
```



### 初始DockerFile

Dockerfile就是用来构建docker镜像的构建文件！命令脚本！先体验一下！

> 方式二

通过这个脚本可以生成镜像，镜像是一层一层的，脚本就是一个一个命令，每个命令一层！

```bash
# 创建一个dockerfile文件

FROM centos

VOLUME ["volume01","volume02"]  #匿名挂载

CMD echo "----end----"
CMD /bin/bash
```



```bash
# 构建镜像
docker build -f dockerfile文件 -t 镜像名:版本 .
```



### 容器数据卷

多个mysql同步数据！

一个容器挂载另一个容器，实现数据同步！

被挂载的容器称为父容器！！！

**多个容器直接实现数据共享**

```bash
docker run -it --name docker02 --volumes-from docker01 镜像id #docker01挂载docker02
```

==需要注意的是，父容器必须挂载数据卷无论是以-v还是dockerfile的形式，都必须有一个挂载目录==

此时，通过挂载父容器的所有容器共享此目录！！

还有一个重点，数据卷只要还有容器在使用，就不会失去它的作用！

也就是说即使父容器挂了，也不会影响其它容器，其它容器也可以作为新容器的父容器！！！



```bash
# 例子：多个mysql同步数据
docker run -d -p 3344:3306 -v /etc/mysql/conf.d -v /var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 --name mysql01 mysql:5.7

docker run -d -p 3344:3306 -e MYSQL_ROOT_PASSWORD=123456 --name mysql02 --volumes-from docker01 mysql:5.7
```



经常用在容器之间配置信息的传递！



## DockerFile

### DockerFile介绍

dockerfile是用来构建docker镜像文件！命令参数的脚本。

**构建步骤：**

1. 编写一个dockerfile文件
2. docker build 构建成为一个镜像
3. docker run 运行镜像
4. docker push 发布镜像（Docker Hub，阿里云仓库！）



很多官方的镜像都是基础包，很多功能没有，我们通常会自己搭建自己的镜像！



### DockerFile 构建过程

**基础知识：**

1. 每个保留关键字（指令）都必须是大写字母
2. 执行从上到下顺序执行
3. #表示注释
4. 每个命令都会创建提交一个新的镜像层，并提交！



dockerfile是面向开发的，我们发布项目，做镜像，就需要编写dockerfile文件，这个文件十分简单！

docker镜像逐渐成为企业交互的标注！！！

步骤：

DockerFile：构建文件，定义了一切的步骤，源代码

DockerImages：通过DockerFile构建生成的镜像，最终发布和运行的产品！

Docker容器：容器就是镜像运行起来提供服务！



### DockerFile指令

* FROM：基础镜像（centos，ubantu等）
* MAINTAINER：镜像的作者（姓名+邮箱）
* RUN：镜像构建的时候需要运行的命令
* ADD：步骤，压缩包（tomcat压缩包），会自动解压
* WORKDIR：镜像的工作目录
* VOLUME：挂载的目录
* EXPOSE：暴露端口（类似-p做的事）
* CMD：指定这个容器启动的时候运行的命令，只有最后一个会生效，可被替代
* ENTRYPOINT：指定这个容器启动的时候运行的命令，==可以在运行时追加命令==
* ONBUILD：当构建了一个被继承DockerFile 这个时候就会运行ONBUILD 的指令。触发指令
* COPY：类似ADD，将我们文件拷贝到镜像中。
* ENV：构建的时候设置环境变量



```bash
# 实战测试
# DockerHub 中99%镜像是从FROM scratch 这个基础镜像过来的，然后配置需要的软件和配置来进行构建
# 1.编写dockerfile文件
vim mydockerfile-centos

FROM centos
MAINTAINER lhf<280001404@qq.com>

ENY MYPATH /usr/local
WORKDIR $MYPATH

RUN yum -y install vim
RUN yum -y install net-tools

EXPOSE 80

CMD echo $MYPATH
CMD echo "----end----"
CMD /bin/bash

# 2.构建镜像
docker build -f mydockerfile-centos -t mycentos:1.0 .

# 3.测试运行
docker run -it mycentos:0.1
ifconfig   #测试安装的命令
```

**列出镜像的历史**

docker history 镜像id



**ENTRYPOINT**

用ENTRYPOINT执行的命令，可以在运行的时候镜像id后面加-选项，之间追加进去，CMD则不行。



### 实战：Tomcat镜像

1. 准备tomcat和jdk的tar包

2. 编写dockerfile文件

   ```bash
   # 官方命名Dockerfile
   
   FROM centos
   MAINTAINER lhf<280001404@qq.com>
   
   COPY readme.txt /usr/local/readme.txt
   
   ADD jdk-8ull-linux-x64.tar.gz /usr/local
   ADD apche-tomcat-9.0.22.tar.gz /usr/local
   
   RUN yum -y install vim
   
   ENV MYPATH /usr/local
   WORKDIR $MYPATH
   
   ENV JAVA_HOME /usr/local/jdk1.8.0_11
   ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
   ENV CATALINA_HOME /usr/local/apche-tomcat-9.0.22
   ENV CATALINA_BASH /usr/local/apche-tomcat-9.0.22
   ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/bin:$CATALINA_BASH_BASH/bin
   
   EXPOSE 8080
   
   CMD /usr/local/apache-tomcat-9.0.22/bin/startup.sh && tail -F /url/local/apache-tomcat-9.0.22/bin/logs/catalina.out
   ```

3. 构建镜像

   ```bash
   docker build -t diytomcat .
   ```

4. 启动镜像

   ```bash
   docker run -d -p 8088:8080 -v /root/dockerfiletest/tomcat/test:/usr/local/apache-tomcat-9.0.35/webapps/test diytomcat
   ```

5. 测试镜像

   访问外网的8088端口测试。

   在主机的test目录创建index.html，并测试访问！！！



### 发布镜像

> 发布到DockerHub

1. 在DockerHub上注册账号

2. 在服务器上登录

   ```bash
   docker login -u 用户名 -p 密码
   docker push 镜像名:版本号
   ```



> 阿里云镜像服务

1. 登录阿里云
2. 找到容器镜像服务
3. 创建命名空间
4. 创建镜像仓库（本地仓库）
5. 查看基本信息，按操作指南完成



## 小结

![1590073688772](J:\桌面\文章\文章图片\1590073688772.png)



## Docker网络

### 理解Docker0

清空所有环境！！！



在服务器上通过ip addr可以发现，服务器为我们的docker生成了一个docker0的网卡！作为网关！

我们每启动一个docker容器，docker就会给docker容器分配一个ip，通过桥接与主机互通！

主机会多出来一个网卡与容器内部桥接。

```bash
# 我们发现容器带来的网卡，都是一对一对的
# veth-pair 就是一对虚拟设备接口，成对出现，一端连着协议，一端彼此相连
# 正因为有这个特性，veth-pair 充当一个桥梁，连接各种虚拟网络设备
# OpenStac，Docker容器之间的连接，OVS的链接，都是使用veth-pair技术
```

**同一台服务器下的容器间如何进行网络通信**

测试的时候我们发现，我们在服务器上是可以ping同服务器上的每个容器的！

而通过veth-pair技术，容器间也是可以ping通的！！！

容器通过veth-pair连接docker0，通过docker0与其它容器通信！

![1590113820384](J:\桌面\文章\文章图片\1590113820384.png)

Docker中的所有的网络接口都是虚拟的。

### --link（不推荐使用）

**思考一个场景，我们编写了一个微服务，database url=ip，项目不重启，数据库ip换掉了，我们希望可以处理这个问题，可以通过名字来进行访问容器？**

```bash
# 用link命令连接两个容器（就是配置hosts主机民映射）
docker run -d -P --name 容器2 --link 容器1 镜像名
# 测试
docker exec -it 容器2 ping 容器1	#成功ping通
# 无法反向ping通的
```



### 自定义网络

> 查看所有的docker网络

```bash
docker network ls
```

**网络模式**

bridge：桥接模式（相当于通过一个中间桥连接）（默认）

none：不配置网络

host：和宿主机共享网络

container：容器内网络连通（用的少，局限性很大）



> 创建网络

```bash
# 我们直接启动的命令（下面二者相同）
docker run -d -P --name 容器名称  镜像名
docker run -d -P --name 容器名称 --net bridge 镜像名

# 自定义网络
docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 mynet
# 查看
docker network ls
# 查看自定义网络
docker network inspect mynet
# 使用自定义网络运行容器
docker run -d -P --name 容器名称 --net mynet 镜像名
```

**推荐使用自定义网络，可以自动完成hosts主机名解析的配置，而无需使用--link**

**好处：**不同的集群使用不同的网络，保证集群是安全和健康的。



### 网络连通

之前的知识告诉了我们，再同一个网段下的容器可以相互ping通。

```bash
docker network connect 网络名 容器名 #将其他网络段的容器，连接进自定义网络中
# 自定义网络会为这个容器再分配一个ip
# 一个容器，两个ip
```

