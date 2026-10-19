# Docker 使用指南

## 1. Docker 基础概念

Docker 是一个开源的容器化平台，用于构建、分发和运行容器化应用程序。

Docker 可以将应用程序及其运行依赖打包成标准化的镜像，通过容器技术实现应用程序的隔离运行和快速部署。

与传统虚拟机不同，Docker 容器通常不需要运行完整的客户机操作系统，而是共享宿主机的内核，因此具有启动速度快、资源开销相对较低等特点。

Docker 的主要作用：

- **环境一致性**：统一开发、测试和生产环境。
- **应用隔离**：不同应用可以在相互隔离的容器中运行。
- **快速部署**：通过镜像快速创建和部署应用。
- **可移植性**：在兼容的平台和运行环境中部署相同的镜像。
- **资源利用**：相比完整虚拟机，容器通常具有较低的资源开销。

### 1.1 Docker 基本组成

Docker 主要由镜像、容器和镜像仓库三个核心部分组成。

| 组成 | 英文 | 作用 |
|---|---|---|
| 镜像 | Image | 创建容器的只读模板 |
| 容器 | Container | 镜像创建的运行实例 |
| 镜像仓库 | Repository | 存储和管理相关容器镜像 |
| 镜像注册服务 | Registry | 提供镜像存储、检索和分发服务 |

#### 镜像（Image）

镜像是用于创建容器的只读模板，包含应用程序运行所需的文件、运行时环境、依赖库及相关配置。

一个镜像可以创建多个容器。

从面向对象的角度，可以将镜像类比为 Java 中的类模板。

#### 容器（Container）

容器是镜像创建的运行实例，用于提供相对独立的应用运行环境。

容器可以被创建、启动、停止、重启和删除。

从面向对象的角度，可以将容器类比为类创建的实例对象。

容器通常具有以下特点：

- 具有独立的文件系统视图。
- 可以拥有独立的网络和进程命名空间。
- 可以限制 CPU、内存等资源的使用。
- 共享宿主机内核，而不是运行完整的独立操作系统。

需要注意，容器提供的是操作系统级隔离，并不等同于完整虚拟机的隔离边界。

#### 镜像仓库（Repository / Registry）

镜像仓库用于存储和管理镜像。

常见镜像注册服务：

- **Docker Hub**：Docker 官方提供的公共镜像注册服务。
- **阿里云容器镜像服务 ACR**：提供镜像托管和分发服务。
- **Harbor**：开源的企业级容器镜像仓库管理平台。
- **GitHub Container Registry（GHCR）**：GitHub 提供的容器镜像注册服务。
- **Docker Registry**：可以自行部署的镜像注册服务。

其中：

- Registry 是提供镜像存储和分发能力的服务。
- Repository 是 Registry 中用于组织相关镜像的逻辑仓库。
- Tag 是用于区分镜像版本或变体的标签。

### 1.2 Docker 工作原理

Docker 采用客户端与服务端架构。

主要组件：

- **Docker Client**：客户端，通过 `docker` 命令向 Docker 后台服务发送请求。
- **Docker Daemon**：后台服务，负责管理镜像、容器、网络和数据卷。
- **Container Runtime**：负责容器的创建和运行。
- **Registry**：负责存储和分发镜像。

基本工作流程：

```text
Docker Client
     │
     │ docker 命令
     ▼
Docker Daemon
     │
     ├── 管理镜像（Image）
     │
     ├── 管理容器（Container）
     │
     ├── 管理网络（Network）
     │
     └── 管理数据卷（Volume）
              │
              ▼
       Registry / Docker Hub
```

例如：

```bash
docker run nginx
```

Docker 通常会执行以下操作：

1. 检查本地是否存在所需镜像。
2. 如果镜像不存在，则从镜像仓库拉取。
3. 根据镜像创建容器。
4. 为容器配置文件系统、网络等运行环境。
5. 启动容器主进程。

### 1.3 Docker 与虚拟机的区别

| 对比项 | Docker 容器 | 虚拟机 |
|---|---|---|
| 虚拟化方式 | 操作系统级虚拟化 | 硬件虚拟化 |
| 操作系统内核 | 通常共享宿主机内核 | 通常运行独立的客户机内核 |
| 启动速度 | 通常较快 | 通常较慢 |
| 资源开销 | 相对较低 | 相对较高 |
| 隔离方式 | Namespace、Cgroups 等 | Hypervisor、虚拟硬件等 |
| 主要用途 | 应用部署、微服务、开发环境 | 完整操作系统隔离、多系统运行 |

注意：

在 macOS 和 Windows 上运行 Linux 容器时，Docker Desktop 通常会通过轻量级 Linux 虚拟机提供 Linux 内核。

因此，容器共享的是 Docker 所使用的 Linux 内核，并不一定是物理宿主机本身的内核。

---

## 2. Docker 镜像原理

### 2.1 Docker 镜像

Docker 镜像是一种标准化的软件交付形式，通常包含：

- 应用程序代码。
- 运行时环境。
- 系统工具和依赖库。
- 配置文件。
- 默认启动命令。
- 相关元数据。

镜像本身不会直接运行，而是用于创建容器。

基本关系：

```text
Dockerfile
    │
    │ docker build
    ▼
Docker Image
    │
    │ docker run
    ▼
Docker Container
```

同一个镜像可以创建多个相互独立的容器。

### 2.2 镜像分层机制

Docker 镜像采用分层存储机制。

镜像由多个只读层组成，每一层通常记录一组文件系统变更。

例如：

```text
应用程序镜像
┌──────────────────────┐
│ 应用程序代码层       │
├──────────────────────┤
│ 应用依赖层           │
├──────────────────────┤
│ 运行时环境层         │
├──────────────────────┤
│ 基础系统文件层       │
└──────────────────────┘
```

镜像层具有以下特点：

1. **只读性**：镜像层创建后不可直接修改。
2. **共享性**：不同镜像可以共享相同的底层内容。
3. **复用性**：构建镜像时可以复用符合条件的缓存层。
4. **增量性**：镜像层记录文件系统变化，而不是每层都保存一份完整文件系统。

#### UnionFS 与 OverlayFS

UnionFS（联合文件系统）是一类能够将多个文件系统层组合为统一文件系统视图的技术。

Docker 的分层镜像机制与联合文件系统的思想密切相关。

在 Linux 环境中，Docker 常使用 OverlayFS 等技术实现镜像层与容器可写层的组合。

其核心思想是：

```text
容器可写层
    +
镜像只读层 N
    +
镜像只读层 N-1
    +
...
    +
基础镜像层
    ↓
统一的容器文件系统视图
```

不同存储驱动和镜像存储实现的底层机制可能有所区别。

### 2.3 Docker 镜像加载原理

Docker 容器通常共享宿主机或 Docker Linux 虚拟机提供的内核。

镜像主要提供用户态文件系统和应用程序运行所需的内容，而不是为每个容器打包一套独立的 Linux 内核。

镜像中的基础文件系统通常称为 RootFS。

RootFS 可以包含：

- `/bin`
- `/etc`
- `/lib`
- `/usr`
- `/var`

具体目录取决于镜像所使用的基础系统和应用需求。

容器运行时，Docker 将镜像层与容器可写层组合起来，为应用提供统一的文件系统视图。

### 2.4 为什么镜像采用分层结构？

主要优势：

1. **共享资源**：多个镜像可以共享相同的镜像层。
2. **减少存储开销**：避免重复存储相同内容。
3. **提高构建效率**：利用构建缓存，减少重复操作。
4. **提高传输效率**：拉取或推送镜像时可以复用已有层。
5. **方便镜像扩展**：可以基于已有镜像构建新的应用镜像。

### 2.5 容器可写层

需要重点理解：

> Docker 镜像层是只读的，运行中的容器通常具有独立的可写层。

创建容器时，Docker 会在镜像层之上增加容器可写层。

```text
容器
┌──────────────────────┐
│ 容器可写层（RW）     │
├──────────────────────┤
│ 镜像层 3（RO）       │
├──────────────────────┤
│ 镜像层 2（RO）       │
├──────────────────────┤
│ 镜像层 1（RO）       │
└──────────────────────┘
```

其中：

- `RO`：Read Only，只读。
- `RW`：Read Write，可读写。

当容器修改镜像中的文件时，存储系统通常通过写时复制（Copy-on-Write）等机制处理文件变更。

需要注意：

- 容器可写层中的数据通常随容器删除而丢失。
- 镜像层不会因为容器中的文件修改而改变。
- 需要长期保存的数据应使用 Volume 或 Bind Mount。

---

## 3. Docker 环境与服务管理

### 3.1 Docker 服务管理

以下命令主要适用于采用 systemd 管理 Docker Engine 的 Linux 系统。

```bash
# 启动 Docker 服务
sudo systemctl start docker

# 停止 Docker 服务
sudo systemctl stop docker

# 重启 Docker 服务
sudo systemctl restart docker

# 查看 Docker 服务状态
sudo systemctl status docker

# 设置开机自动启动
sudo systemctl enable docker

# 取消开机自动启动
sudo systemctl disable docker
```

注意：

macOS 和 Windows 上的 Docker Desktop 通常不通过 `systemctl` 管理，应使用 Docker Desktop 应用或其支持的管理方式。

### 3.2 Docker 基本信息

```bash
# 查看 Docker 版本
docker --version

# 查看客户端和服务端版本
docker version

# 查看 Docker 详细信息
docker info

# 查看 Docker 命令帮助
docker --help

# 查看指定命令帮助
docker run --help

# 查看 Docker 磁盘空间使用情况
docker system df

# 查看详细磁盘使用情况
docker system df -v
```

---

## 4. Docker 镜像管理

### 4.1 查看本地镜像

```bash
# 查看本地镜像
docker images

# 等价写法
docker image ls

# 查看包括中间镜像在内的镜像列表
docker images -a

# 仅显示镜像 ID
docker images -q

# 查看虚悬镜像
docker images --filter dangling=true

# 查看指定镜像的详细信息
docker image inspect nginx:alpine

# 查看镜像构建历史
docker history nginx:alpine
```

### 4.2 搜索远程镜像

```bash
# 在 Docker Hub 搜索 nginx 镜像
docker search nginx

# 限制显示结果数量
docker search nginx --limit 5
```

说明：

`--limit` 用于限制结果数量，并不保证按照点赞数进行排序。

### 4.3 拉取镜像

```bash
# 拉取 nginx 镜像
docker pull nginx

# 拉取指定标签
docker pull nginx:alpine

# 拉取 Ubuntu 指定版本
docker pull ubuntu:24.04
```

镜像名称通常采用以下格式：

```text
[Registry地址/]仓库名[:标签]
```

例如：

```text
nginx:alpine
ubuntu:24.04
用户名/项目名:1.0
registry.example.com/project/app:1.0
```

注意：

- 不指定标签时，通常默认使用 `latest`。
- `latest` 只是一个普通标签，不代表镜像一定是最新构建或最新稳定版本。
- 生产环境建议使用明确的版本标签，必要时通过 Digest 固定镜像内容。

### 4.4 删除镜像

```bash
# 删除指定镜像
docker rmi nginx:alpine

# 根据镜像 ID 删除
docker rmi 镜像ID

# 删除多个镜像
docker rmi 镜像1 镜像2

# 强制删除指定镜像引用
docker rmi -f 镜像ID

# 清理虚悬镜像
docker image prune

# 清理所有没有被容器引用的镜像
docker image prune -a
```

注意：

- 正被容器使用的镜像通常不能直接删除。
- `docker image prune -a` 的清理范围比普通 `prune` 更大。
- `docker rmi -f` 可能强制移除镜像引用，应谨慎操作。

### 4.5 虚悬镜像（Dangling Image）

虚悬镜像通常指没有仓库名和标签引用的镜像。

例如：

```text
REPOSITORY    TAG       IMAGE ID
<none>        <none>    abc123456789
```

虚悬镜像可能在镜像重新构建、标签更新等操作后出现。

可以执行：

```bash
# 查看虚悬镜像
docker images --filter dangling=true

# 删除虚悬镜像
docker image prune
```

虚悬镜像不一定是无效数据，也可能仍然包含有用内容，应根据实际情况决定是否清理。

### 4.6 镜像导出与导入

使用 `save` 和 `load` 可以将镜像保存为归档文件，并在另一台设备上导入。

```bash
# 将镜像保存为 tar 文件
docker save -o nginx.tar nginx:alpine

# 从 tar 文件加载镜像
docker load -i nginx.tar
```

主要特点：

- 可以保存镜像层及相关元数据。
- 适合镜像备份、离线传输和环境迁移。
- 与容器文件系统的 `export/import` 不同。

---

## 5. Docker 容器管理

### 5.1 创建并运行容器

基本语法：

```bash
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

常用参数：

| 参数 | 作用 |
|---|---|
| `--name` | 指定容器名称 |
| `-i` | 保持标准输入打开 |
| `-t` | 分配伪终端 |
| `-it` | 交互式运行容器 |
| `-d` | 后台运行容器 |
| `--rm` | 容器退出后自动删除 |
| `-p` | 指定端口映射 |
| `-P` | 随机发布镜像声明的端口 |
| `-v` | 挂载数据卷或宿主机路径 |
| `--mount` | 使用结构化参数配置挂载 |
| `-e` | 设置环境变量 |
| `--env-file` | 从文件加载环境变量 |
| `--network` | 指定容器网络 |
| `--restart` | 设置容器重启策略 |
| `--privileged` | 赋予容器扩展的系统权限，存在较高安全风险 |

### 5.2 前台交互式运行

```bash
# 创建 Ubuntu 容器并进入 Bash
docker run -it ubuntu:24.04 bash

# 创建 Alpine 容器并进入 Shell
docker run -it alpine sh

# 退出后自动删除容器
docker run --rm -it alpine sh
```

参数说明：

- `-i`：保持标准输入打开。
- `-t`：分配伪终端。
- `--rm`：容器退出后自动删除容器。

注意：

并不是所有镜像都包含 `/bin/bash`。

例如 Alpine 通常使用 `sh`。

### 5.3 后台运行容器

```bash
# 后台运行 nginx
docker run -d nginx:alpine

# 指定容器名称
docker run -d --name web nginx:alpine

# 指定端口映射
docker run -d --name web -p 8080:80 nginx:alpine
```

端口映射格式：

```text
-p 宿主机端口:容器端口
```

例如：

```bash
docker run -d -p 8080:80 nginx:alpine
```

表示：

```text
宿主机 8080 端口
       │
       ▼
容器 80 端口
```

访问：

```text
http://localhost:8080
```

如果只希望本机访问，可以绑定到回环地址：

```bash
docker run -d --name web \
  -p 127.0.0.1:8080:80 \
  nginx:alpine
```

需要注意：

`-d` 表示容器在后台运行，但并不意味着容器一定会持续运行。

**容器的生命周期主要由其主进程决定。主进程退出后，容器通常也会停止。**

### 5.4 设置容器自动重启

```bash
# 容器异常退出时自动重启
docker run -d --restart on-failure nginx:alpine

# 除非手动停止，否则自动重启
docker run -d --restart unless-stopped nginx:alpine

# 设置容器始终自动重启
docker run -d --restart always nginx:alpine
```

常见策略：

| 策略 | 含义 |
|---|---|
| no | 默认，不自动重启 |
| on-failure | 主进程异常退出时重启 |
| always | 尽可能保持容器运行 |
| unless-stopped | 自动重启，但尊重手动停止状态 |

### 5.5 查看容器

```bash
# 查看正在运行的容器
docker ps

# 查看所有容器
docker ps -a

# 仅显示容器 ID
docker ps -q

# 查看所有容器 ID
docker ps -aq

# 查看最近创建的容器
docker ps -l

# 查看最近创建的 5 个容器
docker ps -n 5

# 查看容器详细信息
docker inspect 容器名或ID

# 查看容器资源使用情况
docker stats
```

### 5.6 启动、停止与重启容器

```bash
# 启动已停止的容器
docker start 容器名或ID

# 停止容器
docker stop 容器名或ID

# 重启容器
docker restart 容器名或ID

# 强制停止容器
docker kill 容器名或ID
```

区别：

- `stop`：通常先发送正常终止信号，等待一段时间后再强制终止。
- `kill`：默认发送 `SIGKILL` 信号，强制结束容器主进程。

### 5.7 删除容器

```bash
# 删除已停止的容器
docker rm 容器名或ID

# 强制删除正在运行的容器
docker rm -f 容器名或ID

# 删除所有已停止的容器
docker container prune

# 删除所有容器（危险操作）
docker rm -f $(docker ps -aq)
```

注意：

最后一条命令会删除所有容器，包括正在运行的容器。

删除容器通常不会自动删除命名数据卷，但容器可写层中的数据会丢失。

### 5.8 查看容器日志

```bash
# 查看容器日志
docker logs 容器名或ID

# 实时查看日志
docker logs -f 容器名或ID

# 查看最近 100 行日志
docker logs --tail 100 容器名或ID

# 查看最近 10 分钟的日志
docker logs --since 10m 容器名或ID

# 显示日志时间戳
docker logs -t 容器名或ID
```

日志能否通过 `docker logs` 查看，还取决于容器使用的日志驱动和应用的日志输出方式。

### 5.9 查看容器内部进程

```bash
# 查看容器内部运行的进程
docker top 容器名或ID

# 查看容器资源使用情况
docker stats 容器名或ID

# 查看容器详细配置
docker inspect 容器名或ID
```

### 5.10 进入运行中的容器

主要有两种方式：

```bash
# 在容器中创建新的交互式 Bash 进程
docker exec -it 容器名或ID bash

# 使用 Shell
docker exec -it 容器名或ID sh

# 附加到容器主进程的标准输入输出
docker attach 容器名或ID
```

两者的区别：

| 命令 | 特点 |
|---|---|
| docker exec | 在容器中启动新的进程 |
| docker attach | 连接到容器已有主进程的标准输入输出 |

通常推荐使用：

```bash
docker exec -it 容器名或ID sh
```

原因：

- 不会直接附加到容器主进程。
- 退出 `exec` 启动的 Shell 通常不会停止容器。
- 适合进入容器进行调试。

注意：

`attach` 的退出行为取决于容器主进程及信号处理方式。

### 5.11 退出容器

交互式容器：

```bash
exit
```

也可以使用：

```text
Ctrl + D
```

如果当前 Shell 是容器主进程，退出后容器通常会停止。

对于通过 `attach` 附加的终端，可以使用默认分离快捷键：

```text
Ctrl + P
Ctrl + Q
```

在支持该快捷键的交互式终端中，可以从容器分离而不主动停止容器。

### 5.12 容器与宿主机之间复制文件

```bash
# 将容器中的文件复制到宿主机
docker cp 容器名:/容器内路径 宿主机路径

# 将宿主机文件复制到容器中
docker cp 宿主机路径 容器名:/容器内路径
```

示例：

```bash
# 复制容器中的配置文件
docker cp web:/etc/nginx/nginx.conf ./nginx.conf

# 将本地配置复制到容器
docker cp ./nginx.conf web:/etc/nginx/nginx.conf
```

注意：

通过 `docker cp` 修改的容器文件通常不会影响原始镜像。

如果文件没有存储在持久化挂载中，删除容器后相关修改通常会丢失。

### 5.13 容器导出与导入

```bash
# 将容器文件系统导出为 tar
docker export 容器名或ID -o container.tar

# 从 tar 文件导入为新的镜像
docker import container.tar myimage:1.0
```

也可以使用管道：

```bash
docker export 容器ID > container.tar

cat container.tar | docker import - myimage:1.0
```

注意：

`export/import` 主要处理容器文件系统。

与 `save/load` 不同，它通常不会完整保留原始镜像的构建历史和运行配置。

### 5.14 Docker Commit

`docker commit` 可以根据容器当前可写文件系统的状态创建新的镜像。

```bash
docker commit \
  -m "描述信息" \
  -a "作者" \
  容器ID \
  镜像名称:标签
```

示例：

```bash
docker commit \
  -m "添加自定义配置" \
  -a "developer" \
  web \
  my-nginx:1.0
```

注意：

- `docker commit` 不会将挂载数据卷中的数据打包进镜像。
- 适合调试、实验或临时保存容器状态。
- 正式项目通常推荐通过 Dockerfile 构建镜像，以保证构建过程可复现。

---

## 6. Docker 数据持久化

### 6.1 数据卷（Volume）

Docker 数据卷用于持久化容器数据。

容器本身具有独立的可写层，但删除容器时，可写层中的数据通常也会被删除。

数据卷能够将需要持久化的数据存储在容器可写层之外。

主要作用：

1. 持久化应用数据。
2. 在多个容器之间共享数据。
3. 将数据生命周期与容器生命周期分离。
4. 避免重要数据直接依赖容器可写层。

### 6.2 Docker 挂载类型

Docker 常用的挂载方式包括：

| 类型 | 说明 | 常见用途 |
|---|---|---|
| Volume | 由 Docker 管理的数据卷 | 数据库、应用持久化数据 |
| Bind Mount | 将宿主机文件或目录挂载到容器 | 开发代码、配置文件 |
| tmpfs | 将数据存储在内存中 | 临时数据、敏感临时文件 |

#### Volume

由 Docker 管理数据的实际存储位置。

```bash
docker volume create app-data
```

挂载：

```bash
docker run -d \
  --name app \
  -v app-data:/app/data \
  nginx:alpine
```

#### Bind Mount

直接使用宿主机上的文件或目录。

```bash
docker run -d \
  --name web \
  -v /宿主机目录:/容器目录 \
  nginx:alpine
```

#### tmpfs

将临时数据保存在内存中，适用于 Linux 容器。

```bash
docker run --rm \
  --tmpfs /tmp \
  alpine sh -c "echo hello > /tmp/test.txt"
```

### 6.3 创建与管理数据卷

```bash
# 创建数据卷
docker volume create mydata

# 查看所有数据卷
docker volume ls

# 查看数据卷详细信息
docker volume inspect mydata

# 删除指定数据卷
docker volume rm mydata

# 清理未使用的数据卷
docker volume prune
```

注意：

删除数据卷可能造成永久性数据丢失。

Docker 通常不会因为停止或删除一个容器就自动删除其命名数据卷，但显式删除数据卷或执行相关清理命令时，数据仍可能丢失。

### 6.4 使用数据卷

```bash
# 创建数据卷
docker volume create app-data

# 启动容器并挂载数据卷
docker run -d \
  --name app \
  -v app-data:/app/data \
  nginx:alpine
```

挂载格式：

```text
-v 数据卷名称:容器内路径
```

也可以使用 `--mount`：

```bash
docker run -d \
  --name app \
  --mount type=volume,source=app-data,target=/app/data \
  nginx:alpine
```

### 6.5 使用宿主机目录挂载

```bash
docker run -d \
  --name web \
  -p 127.0.0.1:8080:80 \
  -v "$(pwd)/html:/usr/share/nginx/html:ro" \
  nginx:alpine
```

其中：

- `$(pwd)/html`：宿主机当前目录下的 `html` 目录。
- `/usr/share/nginx/html`：容器中的目标目录。
- `ro`：容器只能通过该挂载读取数据。

该命令适用于 Linux、macOS 等支持相应 Shell 语法的环境。

使用前应确保宿主机上的 `html` 目录存在，并包含需要提供的静态文件。

### 6.6 数据卷读写权限

挂载时可以指定读写权限。

#### 可读写（rw）

```bash
docker run -d \
  --name app \
  -v app-data:/app/data:rw \
  nginx:alpine
```

`rw` 为默认行为。

#### 只读（ro）

```bash
docker run -d \
  --name app \
  -v app-data:/app/data:ro \
  nginx:alpine
```

`ro` 表示容器不能通过该挂载写入数据。

### 6.7 查看挂载信息

```bash
docker inspect 容器名或ID
```

输出中的 `Mounts` 字段包含挂载信息，例如：

```json
{
  "Type": "volume",
  "Name": "app-data",
  "Source": "...",
  "Destination": "/app/data",
  "RW": true
}
```

其中：

- `Type`：挂载类型。
- `Name`：数据卷名称。
- `Source`：实际来源路径。
- `Destination`：容器内部的挂载路径。
- `RW`：是否具有写权限。

### 6.8 多容器共享数据卷

多个容器可以挂载同一个数据卷。

```bash
# 创建共享数据卷
docker volume create shared-data

# 容器 A 挂载数据卷
docker run -d \
  --name container-a \
  -v shared-data:/data \
  nginx:alpine

# 容器 B 挂载相同的数据卷
docker run -d \
  --name container-b \
  -v shared-data:/data \
  nginx:alpine
```

两个容器可以通过 `/data` 访问同一个数据卷。

需要注意：

多个容器同时写入相同数据时，应用程序需要自行处理并发访问和数据一致性问题。

### 6.9 通过已有容器共享数据卷

Docker 也支持通过 `--volumes-from` 复用已有容器的挂载配置。

```bash
docker run -d \
  --name container-b \
  --volumes-from container-a \
  nginx:alpine
```

该命令会复用 `container-a` 的数据卷挂载。

需要注意：

- `--volumes-from` 并不表示两个容器存在面向对象意义上的继承关系。
- 新项目通常更推荐显式声明命名数据卷。
- 普通数据卷挂载一般不需要使用 `--privileged=true`。

---

## 7. Docker 容器网络

Docker 提供容器网络功能，用于实现容器之间以及容器与外部网络之间的通信。

### 7.1 常用网络命令

```bash
# 查看 Docker 网络
docker network ls

# 创建自定义网络
docker network create app-network

# 查看网络详细信息
docker network inspect app-network

# 将容器连接到指定网络
docker network connect app-network 容器名

# 断开容器与网络的连接
docker network disconnect app-network 容器名

# 删除网络
docker network rm app-network
```

### 7.2 使用自定义网络

```bash
# 创建网络
docker network create app-network

# 创建容器并加入网络
docker run -d \
  --name web \
  --network app-network \
  nginx:alpine
```

在用户自定义的 Bridge 网络中，容器通常可以通过容器名称进行 DNS 解析。

例如：

```text
Container A
    │
    │ http://web:80
    ▼
Container web
```

对于需要容器间通信的应用，推荐使用自定义网络或 Docker Compose 管理网络。

---

## 8. Dockerfile 镜像构建

### 8.1 Dockerfile

Dockerfile 是描述 Docker 镜像构建过程的文本文件。

开发者可以通过 Dockerfile 定义：

- 基础镜像。
- 工作目录。
- 文件复制操作。
- 依赖安装操作。
- 环境变量。
- 对外声明的端口。
- 容器默认启动命令。

基本流程：

```text
项目源代码
     │
     ├── Dockerfile
     │
     │ docker build
     ▼
Docker Image
     │
     │ docker run
     ▼
Docker Container
```

### 8.2 Dockerfile 常用指令

| 指令 | 作用 |
|---|---|
| FROM | 指定基础镜像 |
| RUN | 在构建过程中执行命令 |
| COPY | 将构建上下文中的文件复制到镜像 |
| ADD | 复制文件，并提供额外的归档处理等能力 |
| WORKDIR | 设置工作目录 |
| ENV | 设置环境变量 |
| ARG | 定义构建参数 |
| EXPOSE | 声明容器预期使用的端口 |
| VOLUME | 声明挂载点 |
| USER | 指定默认运行用户 |
| CMD | 指定容器默认启动命令 |
| ENTRYPOINT | 配置容器的主要执行程序 |

### 8.3 Dockerfile 示例

以构建一个 Nginx 静态网站镜像为例。

项目结构：

```text
project/
├── Dockerfile
└── dist/
    └── index.html
```

Dockerfile：

```dockerfile
# 使用 Nginx 基础镜像
FROM nginx:alpine

# 复制静态资源
COPY ./dist/ /usr/share/nginx/html/

# 声明容器端口
EXPOSE 80
```

构建镜像：

```bash
docker build -t my-web:1.0 .
```

运行容器：

```bash
docker run -d \
  --name my-web \
  -p 127.0.0.1:8080:80 \
  my-web:1.0
```

访问：

```text
http://localhost:8080
```

注意：

`EXPOSE` 只是声明镜像预期使用的端口，并不会自动将该端口发布到宿主机。

需要通过 `-p` 等方式配置实际端口映射。

### 8.4 构建镜像

```bash
# 根据当前目录的 Dockerfile 构建镜像
docker build -t myapp:1.0 .

# 指定 Dockerfile
docker build -f Dockerfile.dev -t myapp:dev .

# 不使用构建缓存
docker build --no-cache -t myapp:1.0 .

# 查看镜像
docker images
```

### 8.5 .dockerignore

`.dockerignore` 用于排除不需要发送到 Docker 构建上下文中的文件。

常见配置：

```dockerignore
# Git
.git/

# 编辑器
.vscode/
.idea/

# 依赖目录
node_modules/

# 日志
*.log

# 本地环境文件
.env
.env.local
.env.*.local

# 临时目录
tmp/
temp/
```

注意：

- `.dockerignore` 与 `.gitignore` 的用途不同。
- `.dockerignore` 主要用于控制 Docker 构建上下文。
- 不应忽略构建镜像时实际需要的文件。
- 不应通过 Dockerfile 将密码、密钥等敏感信息固化到镜像中。

---

## 9. Docker 镜像发布

### 9.1 发布到 Docker Hub

首先登录：

```bash
docker login
```

为本地镜像添加标签：

```bash
docker tag myapp:1.0 用户名/myapp:1.0
```

推送镜像：

```bash
docker push 用户名/myapp:1.0
```

拉取镜像：

```bash
docker pull 用户名/myapp:1.0
```

退出登录：

```bash
docker logout
```

### 9.2 发布到阿里云容器镜像服务

阿里云容器镜像服务 ACR 提供镜像仓库管理与镜像分发能力。

基本流程：

1. 登录阿里云控制台。
2. 进入容器镜像服务 ACR。
3. 选择对应实例。
4. 创建命名空间。
5. 创建镜像仓库。
6. 查看仓库提供的登录、标记和推送命令。

通常需要执行：

```bash
# 登录镜像仓库
docker login Registry地址

# 为本地镜像添加标签
docker tag 本地镜像:版本 Registry地址/命名空间/镜像名:版本

# 推送镜像
docker push Registry地址/命名空间/镜像名:版本

# 拉取镜像
docker pull Registry地址/命名空间/镜像名:版本
```

具体 Registry 地址、认证方式及推送命令应以阿里云控制台提供的信息为准。

### 9.3 搭建本地私有 Registry

Docker Distribution 可以用于搭建私有镜像仓库。

#### 1. 拉取 Registry 镜像

```bash
docker pull registry:3
```

#### 2. 创建数据卷

```bash
docker volume create registry-data
```

#### 3. 启动 Registry

```bash
docker run -d \
  --name registry \
  --restart always \
  -p 127.0.0.1:5000:5000 \
  -v registry-data:/var/lib/registry \
  registry:3
```

其中：

- `5000`：Registry 默认监听端口。
- `registry-data`：用于持久化镜像仓库数据。
- `/var/lib/registry`：Registry 的默认文件存储目录。

该示例仅绑定到本机回环地址，适合本地开发和测试。

#### 4. 查看 Registry 中的镜像仓库

```bash
curl http://127.0.0.1:5000/v2/_catalog
```

#### 5. 添加镜像标签

```bash
docker tag nginx:alpine localhost:5000/nginx:demo
```

#### 6. 推送镜像

```bash
docker push localhost:5000/nginx:demo
```

#### 7. 拉取镜像

```bash
docker pull localhost:5000/nginx:demo
```

#### 8. 运行镜像

```bash
docker run -d \
  --name local-nginx \
  -p 127.0.0.1:8081:80 \
  localhost:5000/nginx:demo
```

### 9.4 使用 HTTP 私有 Registry

对于非本机地址的 HTTP Registry，Docker 可能拒绝通过不安全连接推送或拉取镜像。

Linux Docker Engine 可以通过以下配置允许指定的不安全 Registry：

文件：

```text
/etc/docker/daemon.json
```

配置示例：

```json
{
  "insecure-registries": [
    "192.168.1.100:5000"
  ]
}
```

修改后需要重启 Docker 服务：

```bash
sudo systemctl restart docker
```

注意：

- 正确配置项为 `insecure-registries`。
- JSON 配置项之间需要使用逗号分隔。
- Docker Desktop 应通过其支持的 Docker Engine 设置方式进行配置。
- 不安全 Registry 配置仅适合受控的测试环境。
- 生产环境应优先使用 HTTPS/TLS，并配置适当的身份验证和访问控制。

---

## 10. Docker Compose

### 10.1 Docker Compose 简介

Docker Compose 用于定义和管理多容器应用。

通过 YAML 配置文件，可以统一定义：

- 多个容器服务。
- 镜像或构建方式。
- 端口映射。
- 环境变量。
- 数据卷。
- 网络。
- 服务间依赖关系。

相比逐条执行 `docker run`，Docker Compose 更适合管理具有多个组件的应用。

### 10.2 Compose 配置示例

以 Nginx 静态网站为例。

项目结构：

```text
project/
├── Dockerfile
├── compose.yaml
└── dist/
    └── index.html
```

`compose.yaml`：

```yaml
services:
  web:
    build: .
    ports:
      - "127.0.0.1:8080:80"
    restart: unless-stopped
```

Docker Compose 默认可以识别项目目录中的 `compose.yaml`。

这里的 `build: .` 表示使用当前目录中的 Dockerfile 构建镜像。

### 10.3 Docker Compose 常用命令

```bash
# 启动服务
docker compose up

# 后台启动服务
docker compose up -d

# 构建并启动服务
docker compose up -d --build

# 查看服务状态
docker compose ps

# 查看服务日志
docker compose logs

# 实时查看日志
docker compose logs -f

# 停止服务
docker compose stop

# 启动已停止的服务
docker compose start

# 重启服务
docker compose restart

# 停止并删除 Compose 管理的容器和网络
docker compose down

# 停止并删除容器、网络及相关数据卷
docker compose down -v
```

注意：

- `docker compose down` 默认不会删除命名数据卷。
- `docker compose down -v` 会删除 Compose 管理的命名数据卷及相关匿名数据卷，可能导致数据丢失。
- Compose V2 推荐使用 `docker compose`，而不是旧的独立命令 `docker-compose`。

---

## 11. Docker 数据清理

Docker 长期使用后，可能积累大量镜像、容器、数据卷和构建缓存。

### 11.1 查看磁盘占用

```bash
# 查看 Docker 磁盘占用
docker system df

# 查看详细磁盘占用
docker system df -v
```

### 11.2 清理镜像

```bash
# 清理虚悬镜像
docker image prune

# 清理未被容器引用的镜像
docker image prune -a
```

### 11.3 清理容器

```bash
# 清理已停止的容器
docker container prune
```

### 11.4 清理网络

```bash
# 清理未使用的网络
docker network prune
```

### 11.5 清理数据卷

```bash
# 清理符合条件的未使用数据卷
docker volume prune
```

### 11.6 清理构建缓存

```bash
# 清理构建缓存
docker builder prune

# 清理更多未使用的构建缓存
docker builder prune -a
```

### 11.7 综合清理

```bash
# 清理未使用的容器、网络、虚悬镜像及构建缓存
docker system prune

# 扩大未使用镜像的清理范围
docker system prune -a

# 包含匿名数据卷清理
docker system prune -a --volumes
```

注意：

- `docker system prune -a` 可能删除后续仍需使用的本地镜像。
- `--volumes` 会增加数据删除范围。
- 清理命令并不意味着一定可以恢复所有磁盘空间。
- 执行清理前应确认相关镜像、容器及数据卷是否仍然需要。

---

## 12. Docker 常见问题

### 12.1 Docker 服务未启动

常见报错：

```text
Cannot connect to the Docker daemon
```

可能原因：

- Docker Engine 未启动。
- Docker Desktop 未运行。
- 当前用户没有访问 Docker Socket 的权限。
- 当前 Docker Context 配置不正确。

检查：

```bash
docker info
```

Linux 系统可以查看服务状态：

```bash
sudo systemctl status docker
```

### 12.2 容器启动后立即退出

常见原因：

- 容器主进程执行完成。
- 主进程异常退出。
- 启动命令配置错误。
- 应用程序缺少依赖或必要配置。

排查：

```bash
# 查看所有容器
docker ps -a

# 查看容器日志
docker logs 容器名或ID

# 查看容器详细信息
docker inspect 容器名或ID
```

注意：

后台运行参数 `-d` 不会阻止主进程退出。

### 12.3 容器无法访问

可能原因：

- 容器未运行。
- 端口映射错误。
- 应用程序没有监听预期端口。
- 应用程序只监听容器内部的回环地址。
- 宿主机防火墙或网络配置阻止访问。

检查：

```bash
# 查看容器状态及端口映射
docker ps

# 查看容器日志
docker logs 容器名或ID

# 查看容器端口映射
docker port 容器名或ID
```

### 12.4 容器删除后数据丢失

如果数据只保存在容器可写层中，删除容器后通常无法继续访问这些数据。

需要持久化的数据应使用：

- Docker Volume。
- Bind Mount。
- 外部数据库或对象存储服务。

对于数据库等重要服务，应同时制定备份和恢复方案。

### 12.5 镜像拉取失败

常见报错：

```text
Error response from daemon
```

或者：

```text
connection timed out
```

可能原因：

- 网络无法访问镜像仓库。
- DNS 解析异常。
- 镜像名称或标签错误。
- 私有仓库需要身份验证。
- 镜像仓库存在访问限制。

可以检查：

```bash
# 查看 Docker 详细信息
docker info

# 测试镜像拉取
docker pull hello-world

# 查看镜像仓库登录状态
docker login
```

如果需要配置镜像加速、代理或私有仓库，应根据 Docker Engine、Docker Desktop 以及当前网络环境分别配置。

### 12.6 容器内部没有 Bash

某些轻量级镜像没有安装 Bash。

例如 Alpine 镜像通常使用：

```bash
docker exec -it 容器名 sh
```

如果镜像中没有 Shell，则需要使用镜像提供的其他调试方式。

### 12.7 Docker 命令需要 sudo

在部分 Linux 环境中，普通用户可能没有访问 Docker Daemon 的权限。

可以根据系统安装方式配置相应权限。

需要注意：

Docker 的 `docker` 用户组通常具有较高的系统权限，不应将不受信任的用户随意加入该组。

---

## 13. Docker 常用命令速查

| 分类 | 命令 | 作用 |
|---|---|---|
| 基础 | `docker version` | 查看 Docker 版本 |
| 基础 | `docker info` | 查看 Docker 详细信息 |
| 基础 | `docker system df` | 查看磁盘占用 |
| 镜像 | `docker images` | 查看本地镜像 |
| 镜像 | `docker search nginx` | 搜索镜像 |
| 镜像 | `docker pull nginx` | 拉取镜像 |
| 镜像 | `docker rmi 镜像名` | 删除镜像 |
| 镜像 | `docker image prune` | 清理虚悬镜像 |
| 镜像 | `docker build -t app:1.0 .` | 构建镜像 |
| 镜像 | `docker tag` | 创建镜像标签 |
| 镜像 | `docker push` | 推送镜像 |
| 镜像 | `docker save` | 导出镜像归档 |
| 镜像 | `docker load` | 导入镜像归档 |
| 容器 | `docker run` | 创建并运行容器 |
| 容器 | `docker ps` | 查看运行中的容器 |
| 容器 | `docker ps -a` | 查看所有容器 |
| 容器 | `docker start` | 启动容器 |
| 容器 | `docker stop` | 停止容器 |
| 容器 | `docker restart` | 重启容器 |
| 容器 | `docker rm` | 删除容器 |
| 容器 | `docker logs` | 查看容器日志 |
| 容器 | `docker exec -it` | 在容器中执行交互式命令 |
| 容器 | `docker inspect` | 查看容器详细信息 |
| 容器 | `docker cp` | 复制容器文件 |
| 容器 | `docker stats` | 查看资源使用情况 |
| 数据卷 | `docker volume ls` | 查看数据卷 |
| 数据卷 | `docker volume create` | 创建数据卷 |
| 数据卷 | `docker volume rm` | 删除数据卷 |
| 网络 | `docker network ls` | 查看网络 |
| 网络 | `docker network create` | 创建网络 |
| Compose | `docker compose up -d` | 后台启动服务 |
| Compose | `docker compose down` | 停止并删除 Compose 容器及网络 |
| Compose | `docker compose logs -f` | 实时查看服务日志 |
| 清理 | `docker system prune` | 清理未使用的 Docker 资源 |

---

## 14. Docker 使用注意事项

1. **容器不等于虚拟机**：容器通常共享宿主机或 Docker Linux 虚拟机的内核。

2. **镜像与容器不同**：镜像是创建容器的模板，容器是基于镜像创建的实例。

3. **镜像层是只读的**：容器通常通过独立的可写层保存运行期间的文件变化。

4. **容器删除不等于数据卷删除**：命名数据卷通常独立于容器生命周期，但显式删除或清理数据卷仍然会造成数据丢失。

5. **Dockerfile 优于手动 Commit**：正式项目通常通过 Dockerfile 构建镜像，以提高构建过程的可复现性。

6. **避免滥用 privileged 模式**：`--privileged` 会显著扩大容器权限，普通应用和数据卷挂载通常不需要使用。

7. **谨慎使用 latest 标签**：生产环境应使用明确的版本标签，必要时固定镜像 Digest。

8. **避免直接暴露敏感服务端口**：数据库、管理后台和本地测试服务应根据实际需求限制端口绑定地址和访问权限。

9. **不要将敏感信息写入镜像**：密码、访问令牌和私钥应使用适当的密钥管理机制，而不是直接写入 Dockerfile 或镜像层。

10. **注意镜像架构兼容性**：在 ARM64 和 AMD64 等不同架构间部署时，需要确认镜像是否支持目标平台。

11. **合理使用 Docker Compose**：单个简单容器可以使用 `docker run`，多个相互依赖的服务通常更适合使用 Docker Compose。

12. **谨慎执行清理命令**：特别是涉及 `--volumes`、`prune -a` 等参数的操作，应先确认数据是否已经备份。

---

## 15. 参考资料

- [Docker 官方文档](https://docs.docker.com/)
- [Docker CLI 命令参考](https://docs.docker.com/reference/cli/docker/)
- [Docker Engine](https://docs.docker.com/engine/)
- [Dockerfile 参考](https://docs.docker.com/reference/dockerfile/)
- [Docker Compose](https://docs.docker.com/compose/)
- [Docker 数据卷](https://docs.docker.com/engine/storage/volumes/)
- [Docker Hub](https://hub.docker.com/)
- [Docker Distribution Registry](https://distribution.github.io/distribution/)