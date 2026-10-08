> 本身是一个容器运行载体或称之为管理引擎。

## docker基本组成

1. 镜像（image）：只读模板，用来创建docker容器，一个镜像可以创建多个容器。镜像类似于Java的类模板。
2. 容器（container）：镜像创建的运行实例，类似虚拟化的运行环境，可以独立运行一个或一组的应用。 

面向对象角度：容器是镜像运行时的实体，容器为镜像提供了标准的、隔离的运行环境。容器可以被启动、开始、停止、删除，每个容器都是隔离的、安全的平台。

    - 镜像容器角度：容器是一个简易版的Linux系统和运行在其中的应用程序。
3. 仓库（repository）：集中存放镜像文件的地方。官方仓库：docker hub，国内一般使用阿里云

## docker镜像
> 镜像是一个轻量级、可执行的独立软件包，它包含运行某个软件所需的所有的内容，我们把应用程序和配置依赖打包好形成一个可交付的运行环境（包括代码、运行所需库、环境变量和配置文件等），这个打包好的运行环境就是image镜像文件。
>
> 通过这个镜像文件才能生成Docker容器实例。
>

## 分层的镜像
### UnionFS（联合文件系统）
Union文件系统是一种分层、轻量级并且高性能的文件系统，它支持对文件系统的修改作为一次提交来一层层地叠加，同时可以将不同目录挂载到同一个虚拟文件系统下。Union文件系统是docker镜像的基础。镜像可以通过分层来进行继承，基于基础镜像（即无父镜像），可以制作各种具体的应用镜像。

特性： 一次同时加载多个文件系统，但从外部来看，只能看到一个文件系统，联合加载会把各层问及那系统叠加起来，这样最终的文件系统会包含所有底层的文件和目录。

### Docker镜像加载原理
docker的镜像实际上是由多层文件系统组成的，这就是UnionFS。

BootFS（boot file system 引导文件系统）主要包含boot loader和kernel，bootloader主要是引导加载kernel，Linux刚启动时会记载BootFS，在Docker镜像的最底层是BootFS。这一层与经典的Linux/Unix系统是一样的，包含boot加载器和内核。当boot加载完成之后整个内核就都在内存中了， 此时内存的使用权已由BootFS转交给内核，此时系统也会卸载BootFS。

### 为什么镜像分层？
好处：共享资源，方便复制迁移。

## 重点理解
> Docker镜像层都是只读的，容器层是可写的。
>
>  
>
> 当容器启动时，一个新的可写层被加载到镜像的顶部。这一层通常被称为“容器层”，“容器层”之下的都叫“镜像层”。
>

## docker常用命令
### 帮助启动类命令


1. 启动docker：systemctl start docker
2. 停止docker：systemctl stop docker
3. 重启docker：systemctl restart docker
4. 查看docker状态：systemctl status docker
5. 开机启动：systemctl enable docker
6. 查看docker概要信息：docker info
7. 查看docker总体帮助文档：docker --help
8. 查看docker命令帮助文档：docker 具体命令 --help

### 镜像命令
1. docker images [ ]：列出本地主机上的镜像 
    - -a：列出本地所有的镜像（含历史映像层）
    - -q：只显示镜像ID（IMAGE ID）
2. docker search [ ] 镜像名称：搜索镜像 
    - --limit N：显示点赞数排名前N的镜像，默认N=5
3. docker pull 镜像名称[ ]：下载镜像 
    - :latest：最新版，这是默认值，可以不填
    - :TAG：指定版本
4. docker system df：查看镜像/容器/数据卷所占的空间
5. docker rmi [ ] 镜像ID/名称：删除镜像 
    - -f：强制执行
6. docker rmi [ ] 镜像ID/名称 镜像ID/名称：删除多个镜像
7. docker rmi [ ] ${docker images -qa}：删除全部镜像

### docker虚悬镜像
> 仓库名、标签都是的镜像，俗称虚悬镜像（dangling image）
> 这东西没啥用，遇到删除即可

### 容器命令
#### 前台交互式启动容器
**docker run -it [ ] 镜像名称 /bin/bash**

+ --name="容器名称"：为容器指定一个名称；
+ -i：以交互模式运行容器，通常于 -t 同时使用；
+ -t：为容器分配一个伪输入终端，通常于 -i 同时使用；
+ -P：随机端口映射；
+ -p：指定端口映射；

#### 后台静默式启动容器
**docker run -d 镜像名称**

> 容器后台运行，就必须有一个前台进程，否则容器会自动退出运行。
>

+ -d：后台运行容器并返回容器ID，即启动守护式容器（后台运行）；

#### 查看正在运行的容器
docker ps [ ]

+ -a：列出所有的运行过的容器（正在运行 + 历史运行）。
+ -l：显示最近创建的容器。
+ -n：显示最近创建的N个容器。
+ -q：静默模式，只显示容器编号。

#### 退出容器
1. exit：退出容器，容器停止运行。
2. ctrl + p + q：退出容器，容器不停止运行。

#### 启动已停止运行容器
docker start 容器ID或容器名称

#### 重启容器
docker restart 容器ID容器名称

#### 停止容器
普通停止：docker stop 容器ID或容器名称

强制停止：docker kill 容器ID或容器名称

#### 删除容器
删除已停止容器：docker rm 容器ID

强制删除容器：docker rm -f 容器ID

一次删除多个容器：docker rm -f ${docker ps -a -q} 或 docker ps -a -q | xargs docker rm

#### 查看容器日志
docker logs 容器ID

#### 查看容器内运行的进程
docker top 容器ID

#### 查看容器内部细节
docker inspect 容器ID

#### 重新进入运行中容器并以命令行交互
1. docker exec -it 容器ID /bin/bash
2. docker attach 容器ID

**推荐使用 exec**

两者的区别：  
attach直接进入容器启动命令的终端，不会启动新进程，使用exit退出，会导致容器的停止；  
exec是在容器中重新打开新终端，并且启动新进程，使用exit退出，不会导致容器的停止

#### 容器文件clone到主机
docker cp 容器ID:容器内路径 目的主机路径

#### 容器的导入、导出
+  导入：cat 文件名.tar | docker import - 镜像用户/镜像名称:镜像版本号 
    - 使用docker images 可以查看到我们导出的镜像文件。
+  导出：docker export 容器ID > 文件名.tar 

### docker commit命令
docker commit 提交容器副本使之成为一个新容器。

docker commit -m="描述信息" -a="作者" 容器ID 待创建的镜像名:[标签名]

## 发布本地镜像
### 发布到阿里云仓库
**阿里云开发者平台**创建**镜像仓库**：控制台 ==> 容器镜像服务 ==> 个人实例 ==> 命名空间（没有就创建） ==> 镜像仓库（没有就创建）



创建完镜像仓库后，阿里云会自动生成一些命令脚本，其中就有**将镜像推送到Registry**命令，直接使用即可。

### docker私有库
1. 下载镜像 docker registry
2. 运行私有库，相当于本地有个私有的Docker hub 
    - docker run -d -p 5000:5000 -v /zzyy/myregistry/:/tmp/registry --privileged=true registry
    - -v 代表添加数据卷，后面有数据卷的解释
3. curl验证私有库上有哪些镜像 
    - curl -XGET [http://IP](http://IP):port/v2/_catalog
4. 按照私有库规范修改待推送镜像的tag 
    - docker tag 镜像名  IP:port/镜像名称:标签名
5. 设置docker允许http方式推送镜像 
    - vim /etc/docker/daemon.json
    - 添加 **"insecure-registry": ["IP:port"]**
    - 切记：多个配置项之间使用 **“逗号 ,”** 隔开。
6. push镜像到私有库 
    - docker push 符合私有库规范的镜像名称
7. pull拉取私有库镜像并运行 
    - docker pull IP:PORT/镜像名称:标签名

### docker容器数据卷
> 就是目录或者文件，存在于一个或多个容器中，由docker挂载到容器，但不属于联合文件系统，因此能够绕过Union File System提供一些用于持续存储或共享数据的特性；
>
>  
>
> 卷的设计目的就是为了数据的持久化，完全独立于容器的生存周期，因此Docker不会在容器删除时删除其挂载的数据卷。
>

#### 容器数据卷的功能
-v /宿主机绝对路径目录:/容器内目录 镜像名称

功能特点：

1. 数据卷可在容器之间共享或重用数据；
2. 数据卷中的更改可直接实时生效；
3. 数据卷中的更改不会包含在镜像的更新中；
4. 数据卷的生命周期一直持续到没有容器使用它为止。

**运行时添加数据卷命令**：docker run -it -privileged=true -v /宿主机绝对路径目录:/容器内目录 镜像名称

**查看数据卷是否挂在成功**：docker inspect 容器ID	这个命令会显示容器的很多信息，其中就有**Mounts**信息，Mounts信息会显示挂载信息

### 数据卷的ro和rw读写规则
**:rw** 容器实例可读可写，默认规则，即 **-v /宿主机绝对路径目录:/容器内目录:rw 镜像名称**

**:ro** 容器实例只能读取，ro = read only，即**-v /宿主机绝对路径目录:/容器内目录:ro 镜像名称**

#### 数据卷的继承和共享
继承：docker run -it --privileged=true --volumes-from 父类数据卷名称 镜像名称
