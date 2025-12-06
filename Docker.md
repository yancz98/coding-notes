#  一、Docker Engine

[ Docker 官网](https://www.docker.com/)

## 1.1、Overview

Docker 是一个用于开发、运输和运行应用程序的开放平台。Docker 使您能够将应用程序与基础架构分开，以便您可以快速交付软件。使用 Docker，您可以像管理应用程序一样管理基础架构。通过利用 Docker 的方法来快速传输、测试和部署代码，您可以显着减少编写代码和在生产环境中运行代码之间的延迟。

### 1、Docker 平台

Docker 提供了在称为容器的松散隔离环境中打包和运行应用程序的能力。隔离和安全性允许您在给定主机上同时运行多个容器。容器是轻量级的，包含运行应用程序所需的一切，因此您无需依赖主机上当前安装的内容。您可以在工作时轻松共享容器，并确保与您共享的每个人都获得以相同方式工作的相同容器。

Docker 提供工具和平台来管理容器的生命周期：

- 使用容器开发您的应用程序及其支持组件。
- 容器成为分发和测试应用程序的单元。
- 准备就绪后，将应用程序作为容器或编排服务部署到生产环境中。无论您的生产环境是本地数据中心、云提供商还是两者的混合体，这都是一样的。

### 2、Docker 可以做什么？

- 快速、一致地交付您的应用程序
- 响应式部署和扩展
- 在相同的硬件上运行更多的工作负载

### 3、Docker 架构

![Docker 架构图](Images/Docker_Architecture_old.svg)

（1）Docker Client（客户端）

Docker 客户端用于与 Docker daemon 交互，客户端将命令发送到守护进程执行。客户端和守护进程使用 REST API，通过 UNIX 套接字或网络接口通信。客户端可以与多个守护进程通信，可以连接到本机或远程守护程序。

另一个 Docker 客户端是 Docker Compose，它允许使用由一组容器组成的应用程序。

（2）Docker Daemon（守护程序）

Docker 守护程序负责构建、运行和分发 Docker 容器的繁重工作。 Docker 守护程序侦听 Docker API 请求并管理 Docker 对象（Images、Containers、Networks、Volumes）。守护进程还可以与其他守护进程通信以管理 Docker 服务。

（3）Docker Registry（镜像仓库）

> 公共镜像仓库：[Docker Hub](https://hub.docker.com/) 

集中的存储、分发镜像的服务。

当使用 `docker pull` or `docker run` 命令时，将从配置的镜像仓库中提取所需的镜像。当使用 `docker push` 命令时，镜像像会被推送到配置的仓库中。 

（4）Docker 对象

① 镜像：

相当于是一个 `root` 文件系统，镜像名 = <仓库名>:<标签>。

镜像是一个只读模板，其中包含创建 Docker 容器的说明。使用简单的语法创建一个 Dockerfile，定义创建和运行镜像所需的步骤。Dockerfile 中的每条指令都会在镜像中创建一个层。当更改 Dockerfile 并重建镜像时，仅重建哪些已更改的层。与其它虚拟化技术先比，这是使镜像如此轻量、小巧和快速的部分原因。

运行容器时，它使用隔离的文件系统。此自定义文件系统由容器映像提供。由于镜像包含容器的文件系统，它必须包含运行应用程序所需的一切（所有依赖项、配置、脚本、二进制文件等）。镜像还包含容器的其他配置，例如环境变量、运行的默认命令、和其他元数据。 

② 容器：

镜像是静态的定义， 容器是镜像的可运行实例。可以使用 Docker API 或 CLI 创建、启动、停止、删除、暂停容器。还可以将容器连接到一个或多个网络，将存储附加到它，甚至可以根据其当前状态创建新镜像。 

简而言之，容器是你机器上的沙盒进程，与主机上的所有其他进程隔离。这种隔离利用了 [kernel namespaces and cgroups](https://medium.com/@saschagrunert/demystifying-containers-part-i-kernel-space-2c53d6979504)，这些功能在 Linux 中已经存在了很长时间。Docker 致力于使这些功能简单易用。

### 4、底层技术

Docker 是用 Go 语言编写，利用 Linux 内核的几个特性（内核命名空间和 cgroups）来提供其功能。Docker 使用一种称为容器 namespaces 的技术来提供隔离的工作空间。当运行容器时，Docker 会为该容器创建一组命名空间。这些命名空间提供了一层隔离，容器在各自的命名空间中运行，并且它的访问权限仅限于该命名空间。

Docker 引擎充当 client-server 应用程序，具有：

- dockerd：持续运行的守护进程服务器。

- APIs：指定程序可以用来与 Docker daemon 进行对话和指示的接口。
- docker：命令行界面（CLI）客户端。

### 5、容器与传统虚拟机对比

容器技术解决的是程序的部署运行方面的问题。

|        |  启动  |                     硬盘使用                      |   性能   |     系统支持量     |
| -----: | :----: | :-----------------------------------------------: | :------: | :----------------: |
|   容器 |  秒级  |                     一般为 MB                     | 接近原生 | 单机支持上千个容器 |
| 虚拟机 | 分钟级 | 镜像过大（存储了OS运行所需的所有文件），一般为 GB |   弱于   |     一般几十个     |

-   镜像过大的问题：对于同一系列的操作系统的不同版本、分支来说，其动态链接库、内核、常用工具程序等，都是相同的。如果一台物理机上部署了多个同一操作系统的虚拟机，那么每个虚拟机镜像中就必然有大量相同的文件。使用 COW 技术消除冗余文件。

>   COW: Copy-on-write

写时复制（COW）是一种计算机领域的优化策略。其核心思想是，如果有多个调用者（callers）同时请求相同资源（如内存或磁盘上的数据存储），他们会共同获取相同的指针指向相同的资源，直到某个调用者试图修改资源的内容时，系统才会真正复制一份专用副本（private copy）给该调用者，而其他调用者所见到的最初的资源仍然保持不变。

-   启动过慢的问题：也是因为把整个操作系统的启动流程都包含进去了。主要通过 namespace 和 cGroup 技术实现。

## 1.2、Install

> 查询 Linux 内核版本：
>
> 内核版本低于 3.10，或者缺少内核模块，则 Docker 无法正常运行。

```
# 查看内核 release
$ uname -r
3.10.0-1160.90.1.el7.x86_64

# 查询 Linux 系统版本
$ cat /etc/redhat-release
CentOS Linux release 7.9.2009 (Core)

# 查看 Linux 内核版本
cat /proc/version
Linux version 3.10.0-1160.90.1.el7.x86_64 (mockbuild@kbuilder.bsys.centos.org) (gcc version 4.8.5 20150623 (Red Hat 4.8.5-44) (GCC) ) #1 SMP Thu May 4 15:21:22 UTC 2023
```

### 1、使用 yum 安装

> [CentOS 下安装](https://docs.docker.com/engine/install/centos/)

```shell
# 安装依赖包
$ sudo yum install -y yum-utils 

# 配置 yum 软件源
$ sudo yum-config-manager \
    --add-repo \
    https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

# 官方源（国外）
# $ sudo yum-config-manager \
#     --add-repo \
#     https://download.docker.com/linux/centos/docker-ce.repo

# 安装 Docker Engine 和 Docker CLI
$ sudo yum install docker-ce docker-ce-cli containerd.io
```

### 2、管理 Docker

```shell
# 开机自启 启动/禁用
$ systemctl enable docker
$ systemctl disable docker

# 启动 Docker
$ systemctl start docker

# 停止 Docker
$ systemctl stop docker

# 重启 Docker
$ systemctl restart docker

# 查看 Docker 概要信息
$ docker info

# 查看 Docker 帮助文档
$ docker --help

# 手动启动守护进程
$ dockerd
```

### 3、镜像加速器

```shell
# 查看是否在 docker.service 文件中配置过镜像地址
$ systemctl cat docker | grep '\-\-registry\-mirror'

# 添加配置文件：/etc/docker/daemon.json
# 注释要去掉
{
  "registry-mirrors": [
    # 网易云加速器
    "https://hub-mirror.c.163.com",
    # 百度云加速器
    "https://mirror.baidubce.com" 
  ]
}

# 重启 Docker 服务
$ sudo systemctl daemon-reload
$ sudo systemctl restart docker

# 如果从结果中看到了如下内容，说明配置成功
$ docker info
...
Registry Mirrors:
 https://hub-mirror.c.163.com/
...
```

### 4、安装成功

```shell
$ docker info
# Docker Client info
Client: Docker Engine - Community
 Version:    24.0.2
 Context:    default
 Debug Mode: false
 Plugins:
  buildx: Docker Buildx (Docker Inc.)
    Version:  v0.10.5
    Path:     /usr/libexec/docker/cli-plugins/docker-buildx
  compose: Docker Compose (Docker Inc.)
    Version:  v2.18.1
    Path:     /usr/libexec/docker/cli-plugins/docker-compose

# Docker Server info
Server:
 Containers: 0
  Running: 0
  Paused: 0
  Stopped: 0
 Images: 0
 Server Version: 24.0.2
 Storage Driver: overlay2
  Backing Filesystem: extfs
  Supports d_type: true
  Using metacopy: false
  Native Overlay Diff: true
  userxattr: false
 Logging Driver: json-file
 Cgroup Driver: cgroupfs
 Cgroup Version: 1
 Plugins:
  Volume: local
  Network: bridge host ipvlan macvlan null overlay
  Log: awslogs fluentd gcplogs gelf journald json-file local logentries splunk syslog
 Swarm: inactive
 Runtimes: io.containerd.runc.v2 runc
 Default Runtime: runc
 Init Binary: docker-init
 containerd version: 3dce8eb055cbb6872793272b4f20ed16117344f8
 runc version: v1.1.7-0-g860f061
 init version: de40ad0
 Security Options:
  seccomp
   Profile: builtin
 Kernel Version: 3.10.0-1160.90.1.el7.x86_64
 Operating System: CentOS Linux 7 (Core)
 OSType: linux
 Architecture: x86_64
 CPUs: 1
 Total Memory: 1.794GiB
 Name: hecs-287380
 ID: f04fe2ba-6ca3-452c-86ab-72af75ddd0bb
 Docker Root Dir: /var/lib/docker
 Debug Mode: false
 Experimental: false
 Insecure Registries:
  127.0.0.0/8
 Live Restore Enabled: false
```

### 5、Docker 根目录

> Docker 可执行文件

```shell
> ll /usr/bin/docker*

-rwxr-xr-x 1 root root  37004160 3月  20 23:17 /usr/bin/docker*
-rwxr-xr-x 1 root root 105951040 3月  20 23:17 /usr/bin/dockerd*
-rwxr-xr-x 1 root root     14836 3月  20 23:17 /usr/bin/dockerd-rootless-setuptool.sh*
-rwxr-xr-x 1 root root      7249 3月  20 23:17 /usr/bin/dockerd-rootless.sh*
-rwxr-xr-x 1 root root   2193298 3月  20 23:17 /usr/bin/docker-proxy*
```



> 与 Docker 相关的本地资源默认存放在 `/var/lib/docker/` 目录下：

```
/var/lib/docker/
├─ buildkit		
├─ containers	容器
├─ image		镜像
├─ network		网络
├─ overlay2		存储驱动程序：overlay2 的层
├─ ...          其它存储驱动程序的层
├─ plugins		插件
├─ runtimes		运行时
├─ swarm		
├─ tmp			临时目录
└─ volumes		数据卷
```

### 6、卸载 Docker

```powershell
# 卸载 Docker Engine、CLI、Containerd 和 Docker Compose 软件包：
$ sudo yum remove docker-ce docker-ce-cli containerd.io docker-compose-plugin

# 主机上的映像、容器、卷或自定义配置文件不会自动删除。要删除所有映像、容器和卷：
$ sudo rm -rf /var/lib/docker
$ sudo rm -rf /var/lib/containerd
```



## 1.3、Storage

默认情况下，在容器内创建的所有文件都存储在可写容器层上。这意味着：

- 当该容器不再存在时，数据不会持久存在，并且如果另一个进程需要数据，则很难从容器中取出数据。
- 容器的可写层与运行容器的主机紧密耦合。您不能轻易地将数据移动到其他地方。
- 写入容器的可写层需要存储驱动程序来管理文件系统。存储驱动程序使用 Linux 内核提供联合文件系统。与使用 直接写入主机文件系统的 *数据卷* 相比，这种额外的抽象降低了性能。

Docker 有两个选项供容器将文件存储在主机上，以便即使在容器停止后文件仍然存在：volume 和 bind mount。

Docker 还支持在主机内存中存储文件的容器：tmpfs mount。

> volume、bind mount、tmpfs mount 之间的差异是数据在 Docker 主机上的位置。

![Types of mounts and where they live on the Docker host](Images/Docker_types-of-mounts.png)

- Volumes：存储在由 Docker 管理的主机文件系统的一部分中 `/var/lib/docker/volumes`，非 Docker 进程不能改文件系统的这一部分。volume 是在 Docker 中持久保存数据的最佳方式。
- bind mounts：可以存储在主机系统的任何位置。它们甚至可能是重要的系统文件或目录。Docker 主机或 Docker 容器上的非 Docker 进程可以随时修改它们。
- tmpfs mounts：仅存储在主机系统的内存中，永远不会写入主机系统的文件系统。

### 1、卷：volume

> volume 的优点：

- 卷比绑定挂载更容易备份或迁移。
- 您可以使用 Docker CLI 命令或 Docker API 管理卷。
- 卷适用于 Linux 和 Windows 容器。
- 卷可以更安全地在多个容器之间共享。
- 卷驱动程序允许您将卷存储在远程主机或云提供商上，以加密卷的内容或添加其他功能。
- 新卷的内容可以由容器预先填充。
- Docker Desktop 上的卷比 Mac 和 Windows 主机上的绑定挂载具有更高的性能。

- 当不保证 Docker 主机具有给定的目录结构或文件结构时，优先使用卷。

（1）管理卷

```shell
# 创建一个卷
# 不指定名字时，会生成随机且唯一的名字
$ docker volume create <volume_name>

# 列出所有卷
$ docker volume ls
DRIVER    VOLUME NAME

# 检查卷
$ docker volume inspect <volume_name...>
[
    {
        "CreatedAt": "2022-08-18T00:20:53+08:00",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/test/_data",
        "Name": "test",
        "Options": {},
        "Scope": "local"
    }
]

# 删除卷
$ docker volume rm <volume_name...>

# 删除未使用的数据卷
$ docker volume prune
```

（2）`-v` 或 `--mount` 标志

```shell
$ docker run -d --name <容器名> \
    # 方式1：
    -v <volume_name>:<容器中的挂载路径>:[rw|ro] \
    # 方式2：
    --mount type=volume,source=<volume_name>,target=<容器中挂载的路径>,[rw, readonly|ro] \
    <镜像名>

    # 注：如遇权限不足
    --privileged=true  # 授予此容器扩展权限

    # 使用匿名卷
    #  若不指定挂载卷，则使用匿名卷（随机 hash 值名）
    #  使用 `--rm` 选项容器退出时，自动移除容器和卷
    -v /var/lib/mysql

# ======================
#  -v, --volume 格式说明
# ======================

# 对于命名卷
#  -v <volume_name>:<容器中的挂载路径>:[rw, ro]
#  由三个字段组成，由冒号字符分隔。字段的顺序必须正确

# 对于匿名卷
#  -v /var/lib/mysql
#  第一个字段可以省略

# ========================================
#  --mount 格式 --mount <key>=<value>, ...
# ========================================
<key>                   <value>
type                    volume（默认）|bind|tmpfs
source|src              对于命名卷，这是卷名称。对于匿名卷，省略此字段。
destination|dst|target  挂载在容器中的文件或目录路径（绝对路径）。
-                       readonly|ro 绑定挂载以只读方式挂载到容器中。
bind-propagation        卷使用 rprivate 绑定传播，绑定传播对于卷是不可配置的。
```

（3）启动一个带有卷的容器

```shell
# --mount 绑定
$ docker run -d \
    --name mysql_3306 \
    -p 3306:3306 \
    -e MYSQL_ROOT_PASSWORD=root \
    --mount source=mysql,target=/var/lib/mysql \
    mysql:5.7
    
# -v 绑定
$ docker run -d \
    --name mysql_3307 \
    -p 3307:3306 \
    -e MYSQL_ROOT_PASSWORD=root \
    -v mysql:/var/lib/mysql \
    mysql:5.7

# 从指定容器加载卷（共享容器数据卷）
# 启动 MySQL 时，若卷被占用会启动失败
$ docker run -d \
    --name mysql-3308 \
    -p 3308:3306 \
    -e MYSQL_ROOT_PASSWORD=root \
    --volumes-from mysql-3306 \
    mysql:5.7

# 查看容器的挂载信息
$ docker inspect -f '{{json .Mounts}}' mysql_3306 | jq
[
  {
    "Type": "volume",
    "Name": "database",
    "Source": "/var/lib/docker/volumes/database/_data",
    "Destination": "/var/lib/mysql",
    "Driver": "local",
    "Mode": "z",
    "RW": true,
    "Propagation": ""
  }
]
```

> Tips：

- 如果将一个 **empty volume** 装载到容器中存在文件或目录的目录中，这些文件或目录将传播（复制）到该空卷中，这是预填充另一个容器需要的数据的好方法。

- 如果您启动一个容器并指定一个尚不存在的卷，则会为您创建一个空卷。
- 如果将绑定挂载或非空卷挂载到容器中存在某些文件或目录的目录中，这些文件或目录将会被覆盖。隐藏的文件不会被删除或更改，但在绑定挂载或卷挂载时不可访问。

### 2、绑定挂载：bind mount

自 Docker 早期以来，绑定挂载就已经存在。与卷相比，绑定装载的功能有限。绑定挂载非常高效，但它们依赖于主机的文件系统具有可用的特定目录结构。您不能使用 Docker CLI 命令直接管理绑定装载。

> 绑定挂载的良好用例

- 从主机共享配置文件到容器。如：Docker 默认通过 `/etc/resolv.conf` 从主机挂载到每个容器来为容器提供 DNS 解析方式。
- 在 Docker 主机和容器上的开发环境之间共享源代码或构建工件。
- 当 Docker 主机的文件或目录结构保证与容器所需的绑定挂载一致时。

（1）`-v` 或 `--mount` 标志

```shell
$ docker run -d --name <容器名> \
    # 方式1：
    -v <主机路径>:<容器中的挂载路径>:[rw, ro, z|Z] \
    # 方式2：
    --mount type=bind,source=<宿主机路径>,target=<容器中挂载的路径>,[rw, readonly|ro] \
    <镜像名>

    # 注：如遇权限不足
    --privileged=true  # 授予此容器扩展权限
    
# ========================================
#  --mount 格式 --mount <key>=<value>, ...
# ========================================
<key>                   <value>
type                    volume（默认）|bind|tmpfs
source|src              主机上的绝对路径
destination|dst|target  挂载在容器中的文件或目录路径（绝对路径）。
-                       readonly|ro 绑定挂载以只读方式挂载到容器中。
bind-propagation        绑定传播属性：rprivate|private|rshared|shared|rslave|slave。

注：--mount 不支持 z|Z 选项修改 selinux 标签。
```

注：

- 使用 `-v|--volume` 绑定挂载时，Docker 主机路径不存在时，会自动创建端点，它总是创建为一个目录。

- 使用 `--mount` 绑定挂载时，Docker 主机路径不存在时，不会自动创建，而是会产生错误。

> **绑定传播属性：**

| 传播设置   | 描述                                                         |
| ---------- | ------------------------------------------------------------ |
| `rprivate` | 默认值。与 private 相同，这意味着原始挂载点或副本挂载点中的任何挂载点都不会向任一方向传播。 |
| `private`  | 挂载是私有的。其中的子挂载不暴露给副本挂载，副本挂载的子挂载不暴露给原始挂载。 |
| `rshared`  | 与 shared 相同，但传播也延伸到嵌套在任何原始或副本挂载点内的挂载点。 |
| `shared`   | 原始挂载的子挂载暴露给副本挂载，并且副本挂载的子挂载也传播到原始挂载。 |
| `rslave`   | 与 slave 相同，但传播也延伸到嵌套在任何原始或副本挂载点内的挂载点。 |
| `slave`    | 类似于 shared 挂载，但仅限于一个方向。如果原始挂载暴露了子挂载，则副本挂载可以看到它。但是，如果副本挂载暴露了子挂载，则原始挂载无法看到它。 |

（2）启动一个绑定挂载的容器

```shell
# -v 主机目录不存在时，自动创建
$ docker run -d \
    --name mysql_3309 \
    -p 3309:3306 \
    -e MYSQL_ROOT_PASSWORD=root \
    -v /data/mysql-09:/var/lib/mysql \
    mysql:5.7
    
# --mount 主机目录不存在时，出错
$ docker run -d \
    --name mysql_3310 \
    -p 3310:3306 \
    -e MYSQL_ROOT_PASSWORD=root \
    --mount type=bind,src=/data/mysql-10,dst=/var/lib/mysql \
    mysql:5.7
    
# 查看绑定信息
$ docker inspect -f '{{json .Mounts}}' mysql_3309 | jq
[
  {
    "Type": "bind",
    "Source": "/data/mysql",
    "Destination": "/var/lib/mysql",
    "Mode": "",
    "RW": true,
    "Propagation": "rprivate"
  }
]
```

（3）配置 selinux 标签

如果使用 selinux，则可以添加 `z|Z` 选项来修改挂载到容器中的主机文件或目录的 selinux 标签。这会影响主机本身上的文件或目录，并可能产生 Docker 范围之外的后果。

- `z`：表示绑定挂载内容在多个容器之间共享。
- `Z`：表示绑定装载内容是私有的且不共享的。

使用这些选项时要格外小心。绑定挂载一个系统目录，如 `/home` 或 `/usr` 使用 `Z` 选项会使您的主机无法操作，您可能需要手动重新标记主机文件。

### 3、tmpfs 挂载：tmpfs mount

当你创建一个带有`tmpfs`挂载的容器时，容器可以在容器的可写层之外创建文件。

与卷和绑定挂载相反，`tmpfs`挂载是临时的，只保留在主机内存中。当容器停止时，`tmpfs`mount 被移除，并且写入那里的文件将不会被持久化。

> tmpfs 挂载的良好用例：

- 当您不希望数据保留在主机上或容器内时，最适合使用 tmpfs 挂载。这可能是出于安全原因，或者当您的应用程序需要写入大量非持久状态数据时保护容器的性能。

> tmpfs 挂载的局限性：

- 与卷和绑定挂载不同，您不能在容器之间共享 `tmpfs` 挂载。
- 此功能仅在您在 Linux 上运行 Docker 时可用。
- 在 tmpfs 上设置权限可能会导致它们在容器重启后重置。

（1）选择 `--tmpfs` 或 `--mount`

```shell
$ docker run -d --name <容器名> \
    # 方式1：
    --tmpfs <容器中的挂载路径> \
    # 方式2：
    --mount type=tmpfs,destination=<容器中挂载的路径> \
    <镜像名>

# ========================================
#  --mount 格式 --mount <key>=<value>, ...
# ========================================
<key>                   <value>
type                    volume（默认）|bind|tmpfs
destination|dst|target  挂载在容器中的文件或目录路径（绝对路径）。
tmpfs-size              tmpfs 绑定的大小（以字节为单位）。默认无限制。
tmpfs-mode              tmpfs 的八进制文件模式。默认为 1777 或 world-writable。
```

注：`--tmpfs` 不允许您指定任何可配置选项，并且只能与独立容器一起使用。



### 4、存储驱动程序

要有效地使用存储驱动程序，重要的是要了解 Docker 如何构建和存储镜像，以及容器如何使用这些镜像。您可以使用此信息做出明智的选择，以了解从应用程序中持久保存数据的最佳方式，并避免在此过程中出现性能问题。

（1）存储驱动与 Docker 卷

Docker 使用存储驱动程序来存储图像层，并将数据存储在容器的可写层中。容器的可写层在容器被删除后不会持久化，而是适合存储运行时产生的临时数据。存储驱动程序针对空间效率进行了优化，但（取决于存储驱动程序）写入速度低于本机文件系统性能，尤其是对于使用写时复制文件系统的存储驱动程序。写入密集型应用程序（例如数据库存储）会受到性能开销的影响，特别是如果只读层中存在预先存在的数据。

将 Docker 卷用于写入密集型数据、必须在容器生命周期之外保留的数据以及必须在容器之间共享的数据。

（2）镜像和图层

Docker 镜像由一系列层构建而成。每层代表镜像 Dockerfile 中的一条指令。除了最后一层之外的每一层都是只读的。

```dockerfile
# syntax=docker/dockerfile:1
FROM ubuntu:18.04
LABEL org.opencontainers.image.authors="org@example.com"
COPY . /app
RUN make /app
RUN rm -r $HOME/.cache
CMD python /app/app.py
```

- `FROM`：首先从 `ubuntu:18.04` 镜像创建一个图层。
- `LABEL` ：仅修改图像的元数据，不会生成新层。
- `COPY`：从 Docker 客户端的当前目录添加一些文件。
- 第一个 `RUN`：使用 `make` 命令构建您的应用程序，并将结果写入新层。
- 第二个 `RUN`：删除缓存目录，并将结果写入新层。
- `CMD`：指定在容器内运行什么命令，它只修改图像的元数据，不会产生图像层。

注意：

- 每一层只是与前一层的一组差异。
- 添加和删除文件都会产生一个新层。

当您创建一个新容器时，您会在底层之上添加一个新的可写层。这一层通常被称为“容器层”。对正在运行的容器所做的所有更改，都将写入这个薄的可写容器层。

存储驱动程序处理有关这些层彼此交互方式的细节。可以使用不同的存储驱动程序，在不同的情况下各有优缺点。

（3）容器和图层

容器和镜像像之间的主要区别在于顶部的可写层。所有添加新数据或修改现有数据的容器写入都存储在这个可写层中。当容器被删除时，可写层也被删除。底层图像保持不变。

【隔离】因为每个容器都有自己的可写容器层，并且所有更改都存储在这个容器层中，所以多个容器可以共享对同一底层图像的访问，同时拥有自己的数据状态。

Docker 使用存储驱动程序来管理镜像层和可写容器层的内容。每个存储驱动程序以不同方式处理实现，但所有驱动程序都使用可堆叠镜像像层和 copy-on-write (CoW) 策略。

（4）磁盘上的容器大小

```shell
# 查看正在运行的容器的大概大小
$ docker ps -s
CONTAINER ID  IMAGE      COMMAND  CREATED  STATUS  PORTS  NAMES       SIZE
c24aec90864f  mysql:5.7  ...      ...      ...     ...    mysql_3306  4B (virtual 569MB)
2192bee9c415  mysql:5.7   ...      ...      ...     ...   mysql_3307  84B (virtual 569MB)
```

- `size`：容器的可写层的数据量（在磁盘上）。
- `virtual size`：容器使用的只读镜像数据加上容器的可写层所用的数据量 size。

| CONTAINER ID | IMAGE     | COMMAND | CREATED | STATUS | PORTS | NAMES      | SIZE                |
| ------------ | --------- | ------- | ------- | ------ | ----- | ---------- | ------------------- |
| c24aec90864f | mysql:5.7 | ...     | ...     | ...    | ...   | mysql_3306 | 4B (virtual 569MB)  |
| 2192bee9c415 | mysql:5.7 | ...     | ...     | ...    | ...   | mysql_3307 | 84B (virtual 569MB) |

多个容器可能共享部分或全部只读镜像数据。从同一个镜像启动的两个容器共享 100% 的只读数据，而两个具有不同镜像且具有共同层的容器共享这些共同层。因此，您不能只计算虚拟大小的总和。这可能会高估总磁盘使用量。

如上两个容器从完全相同的图像开始，则这些容器在磁盘上的总大小 = SUM（ 每个容器的 `size`）+ 一个镜像的大小（`virtual size` - `size`）。

（5）copy-on-write（CoW）策略

copy-on-write 是一种共享和复制文件以实现最大效率的策略。如果一个文件或目录存在于镜像中的较低层，而另一层（包括可写层）需要对其进行读取访问，则它只使用现有文件。第一次另一层需要修改文件时（在构建图像或运行容器时），文件被复制到该层并进行修改。这最大限度地减少了 I/O 和每个后续层的大小。

① 共享促进较小的图像

② 复制使容器高效

### 5、Docker 存储驱动程序

理想情况下，很少有数据写入容器的可写层，并且您使用 Docker 卷写入数据。但是，某些工作负载要求您能够写入容器的可写层。这就是存储驱动程序的用武之地。

```shell
# 检查您当前的存储驱动程序 overlay2
$ docker info

...
Server:
 Containers: 7
  Running: 2
  Paused: 0
  Stopped: 5
 Images: 1
 Server Version: 24.0.2
 Storage Driver: overlay2
  Backing Filesystem: extfs
  ...
```



## 1.4、Network

容器网络是指容器相互连接和通信的能力，或者与非 Docker 工作负载的能力。

容器只能看到具有 IP 地址、网关、路由表、DNS 服务和其他网络详细信息的网络接口。

### 1、公布端口

默认情况下，当您使用 `docker create` 或 `docker run` 创建或运行容器时，容器不会向外界公开其任何端口。使用 `-p, --publish` 标志使端口可用于 Docker 外部的服务。这会在主机中创建一个防火墙规则，将容器端口映射到 Docker 主机上的端口到外部世界。

```shell
# 格式： -p <主机端口>:<容器端口>

# 将容器中的 TCP 端口 80 映射到 Docker 主机上的 8080 端口。
-p 8080:80
-p 8080:80/tcp

# 将容器中的 UDP 端口 80 映射到 Docker 主机上的 8080 端口。
-p 8080:80/udp

# 只有 Docker 主机可以访问已发布的容器端口
-p 127.0.0.1:8080:80
```

如果你想让其他容器可以访问一个容器，则没有必要发布容器的端口。容器间通信是通过将容器连接到同一网络（通常是桥接网络）来实现的。

### 2、网络驱动程序

Docker 的网络子系统是可插拔的，使用驱动程序。默认存在多个驱动程序，并提供核心网络功能：

| 网络模式  | 说明                                                         | 总结                                                         |
| --------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| bridge    | 桥接网络（默认）：当您的应用程序在需要与同一主机上的其他容器通信的容器中运行时，通常会使用桥接网络。为每个容器分配 IP，并将容器连接到 docker0。 | 适用于在同一Docker 守护进程主机上运行的容器。                |
| host      | 主机网络：去掉容器和 Docker 宿主机之间的网络隔离，直接使用宿主机的网络。 | --network host                                               |
| overlay   | 覆盖网络：将多个 Docker 守护进程连接在一起，并使 Swarm 服务和容器能够跨节点通信。这种策略消除了进行操作系统级路由的需要。 | 当您需要在不同 Docker 主机上运行的容器进行通信时，或者当多个应用程序使用 Swarm 服务协同工作时的最佳选择。 |
| ipvlan    | IPvlan 网络使用户可以完全控制 IPv4 和 IPv6 寻址。            | IPvlan 类似于 Macvlan，但不会为容器分配唯一的 MAC 地址。当可以分配给网络接口或端口的 MAC 地址数量受到限制时，请考虑使用 IPvlan。 |
| macvlan   | Macvlan 网络允许您为容器分配 MAC 地址，使其在您的网络上显示为物理设备。Docker 守护进程通过 MAC 地址将流量路由到容器。 | 当您从 VM 设置迁移或需要您的容器看起来像网络上的物理主机时，Macvlan 网络是最好的，每个容器都有一个唯一的 MAC 地址。 |
| none      | 将一个容器与宿主机和其他容器完全隔离。none 不适用于 Swarm 服务。 | --network none                                               |
| 网络插件  | 您可以通过 Docker 安装和使用第三方网络插件。                 | 第三方网络插件允许您将 Docker 与专用网络堆栈集成。           |
| container | 新创建的容器不会创建自己的网卡和配置自己的IP，而是和一个指定的容器共享 IP，端口范围等。 | --network container:NAME                                     |

（1）bridge

当启动 Docker 时，会自动创建一个默认的桥接网络（也称为桥接），除非另有指定，否则新启动的容器会连接到默认桥接。你还可以创建用户定义的自定义桥接网络。用户定义的桥接网络优先于默认的桥接网络。

- 整个宿主机的桥接模式都是 docker0，每个接口叫 veth，在本地主机和容器内分别创建一个虚拟接口，并让他们彼此联通（这一对接口叫 veth pair）；

- 每个容器实例内部也有一块网卡，每个接口叫 eth0；

> 使用了默认的桥接网络

```shell
# 列出所有网络
$ docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
cf83c34ff097   bridge    bridge    local
650958dd588e   host      host      local
6a0e3bf486e0   none      null      local

# 检查之前启动的 MySQL 用的什么网络
#  "NetworkID": "cf83c34ff097..."
$ docker inspect -f '{{json .NetworkSettings.Networks}}' mysql_3307 | jq
{
  "bridge": {
    "IPAMConfig": null,
    "Links": null,
    "Aliases": null,
    "NetworkID": "cf83c34ff097a27d3a7c22068f692975d8bfff641c5d7b836e9ee31c9c066532",
    "EndpointID": "ea8d5843cf9571be982b150ab834329bd1cc582c60568bf99a23653b96d05cda",
    "Gateway": "172.17.0.1",
    "IPAddress": "172.17.0.2",
    "IPPrefixLen": 16,
    "IPv6Gateway": "",
    "GlobalIPv6Address": "",
    "GlobalIPv6PrefixLen": 0,
    "MacAddress": "02:42:ac:11:00:02",
    "DriverOpts": null
  }
}
```

用户定义的桥接和默认桥接之间的区别

- 用户定义的桥接提供容器之间的自动 DNS 解析。

- 用户定义的桥接提供更好的隔离。

- 容器可以动态地附加到用户定义的网络或从中分离。

  在容器的生命周期内，您可以动态地将其连接到用户定义的网络或断开连接。要从默认桥接网络中删除容器，您需要停止容器并使用不同的网络选项重新创建它。 

- 每个用户定义的网络都会创建一个可配置的桥接。

- 默认桥接网络上的链接容器共享环境变量。 

连接到同一个用户定义的桥接网络的容器有效地相互暴露所有端口。对于不同网络上的容器或非 Docker 主机可以访问的端口，必须使用 `-p, --publish` 发布端口 。

选项

下表描述了在使用 bridge 驱动程序创建自定义网络时可以传递给 `--option` 的驱动程序特定选项。

其中一些选项也可用作 dockerd CLI 的标志，您可以在启动 Docker daemon  时使用它们来配置默认的 docker0 桥接。

| 选项                                           | 默认          | 描述                                | dockerd -Flag |
| :--------------------------------------------- | :------------ | :---------------------------------- | ------------- |
| com.docker.network.bridge.name                 |               | 创建 Linux 桥接时要使用的接口名称。 | -             |
| com.docker.network.bridge.enable_ip_masquerade | true          | 启用 IP 伪装。                      | --ip-masq     |
| com.docker.network.bridge.enable_icc           | true          | 启用或禁用容器间连接。              | --icc         |
| com.docker.network.bridge.host_binding_ipv4    |               | 绑定容器端口时的默认 IP。           | --ip          |
| com.docker.network.driver.mtu                  | 0（没有限制） | 设置容器网络最大传输单元 (MTU)。    | --mtu         |
| com.docker.network.container_iface_prefix      | eth           | 为容器接口设置自定义前缀。          | -             |

Docker 守护进程支持一个 `--bridge` 标志，您可以使用它来定义自己的 `docker0` 桥接。如果要在同一主机上运行多个守护程序实例，请使用此选项。

使用自定义桥接

```shell
# 创建自定义桥接（-d 默认为：bridge）
$ docker network create -d bridge mynet 

# 列出所有网络
$ docker network ls

# 将容器连接到自定义桥接
# 连接到同一网络的所有容器都可以相互访问容器上的所有端口

# 1 创建或启动容器时，指定网络
$ docker run -d \
    --name mysql_mynet \
    -p 3306:3306 \
    -e MYSQL_ROOT_PASSWORD=root \
    -v mysql:/var/lib/mysql \
    --network mynet \
    mysql:5.7

# 2 将正在运行的容器连接到指定网络
$ docker network connect mynet mysql_3307

# 检查容器使用的网络
$ docker inspect -f '{{json .NetworkSetting.Networks }}' mysql_3307 | jq
{
  "mynet": {
    "IPAMConfig": {},
    "Links": null,
    "Aliases": [
      "2192bee9c415"
    ],
    "NetworkID": "eac4a1f132b68a8a84f646f0e333eea8a4d23eba769d6a1e292b161be2a5b3f0",
    "EndpointID": "3987ad6e8a57a44e326ebc3da86e3544ebcbb9c24e9583b93be01896df89f43e",
    "Gateway": "172.18.0.1",
    "IPAddress": "172.18.0.3",
    "IPPrefixLen": 16,
    "IPv6Gateway": "",
    "GlobalIPv6Address": "",
    "GlobalIPv6PrefixLen": 0,
    "MacAddress": "02:42:ac:12:00:03",
    "DriverOpts": {}
  },
  "bridge": {
    "IPAMConfig": null,
    "Links": null,
    "Aliases": null,
    "NetworkID": "cf83c34ff097a27d3a7c22068f692975d8bfff641c5d7b836e9ee31c9c066532",
    "EndpointID": "ea8d5843cf9571be982b150ab834329bd1cc582c60568bf99a23653b96d05cda",
    "Gateway": "172.17.0.1",
    "IPAddress": "172.17.0.2",
    "IPPrefixLen": 16,
    "IPv6Gateway": "",
    "GlobalIPv6Address": "",
    "GlobalIPv6PrefixLen": 0,
    "MacAddress": "02:42:ac:11:00:02",
    "DriverOpts": null
  }
}

# 断开容器与网络的连接
$ docker network disconnect mynet mysql_mynet

# 删除自定义桥接网络
# 如果有容器连接到当前网络，则会出错
$ docker network rm mynet
```

使用默认桥接网络

默认网桥被认为是 Docker 的遗留细节，不建议用于生产。配置它是一种手动操作，并且存在技术缺陷。

如果创建或运行容器时，您没有使用 `--network` 标志指定网络，并且指定了网络驱动程序，则默认情况下您的容器将连接到默认 `bridge` 网络。连接到默认 `bridge` 网络的容器可以通信，但只能通过 IP 地址进行通信，除非它们使用遗留的 `--link` 标志进行链接。 

> 应用

```shell
# 
```



> 配置默认 bridge 网络

```shell
$ vim /etc/docker/daemon.json

{
  "bip": "192.168.1.1/24",
  "fixed-cidr": "192.168.1.0/25",
  "fixed-cidr-v6": "2001:db8::/64",
  "mtu": 1500,
  "default-gateway": "192.168.1.254",
  "default-gateway-v6": "2001:db8:abcd::89",
  "dns": ["10.20.1.2","10.20.1.3"]
}

# 重启 Docker 生效
```

桥接网络的连接限制

由于 Linux 内核设置的限制，桥接网络变得不稳定，当 1000 个或更多容器连接到单个网络时，容器间通信可能会中断。 

（2）[overlay](https://docs.docker.com/network/drivers/overlay/)

（3）host

如果您对容器使用主机网络模式，则该容器的网络堆栈不会与 Docker 主机隔离（容器共享主机的网络命名空间），并且容器不会分配自己的 IP 地址。例如，如果您运行绑定到端口 80 的容器并使用主机网络，则容器的应用程序可在主机 IP 地址的端口 80 上使用。 

鉴于容器在使用主机模式网络时没有自己的 IP 地址 ，端口映射不会生效，并且忽略 `-p, --publish` 和 `-P, --publish-all` 选项，而是产生警告： 

```
WARNING: Published ports are discarded when using host network mode
```

主机模式网络可用于优化性能，并且在容器需要处理大量端口的情况下，因为它不需要网络地址转换 (NAT)，并且没有为每个端口创建 “userland-proxy”。 

主机网络驱动程序仅适用于 Linux 主机。

> 使用主机网络

```
docker run -d --network host redis

# host 模式下 -p 配置无效
```



（4）[IPvlan](https://docs.docker.com/network/drivers/ipvlan/)

（5）[Macvlan](https://docs.docker.com/network/drivers/macvlan/)

（6）none

如果要完全隔离容器的网络堆栈，可以在启动容器时使用 `--network-none` 标志。在容器中，仅创建环回设备。

以下示例显示了使用none网络驱动程序在alpine容器中显示ip链接的输出。

```shell
# 使用 none 网络驱动程序在 alpine 容器中显示 ip addr
$ docker run --rm \
    --network none \
    alpine:latest \
    ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
```



（）container

> 当父容器停掉后，子容器的网络不可用。



### 3、管理网络

```
docker network COMMAND

Commands：
    connect     将容器连接到网络
    create      创建网络
    disconnect  断开容器与网络的连接
    inspect     显示一个或多个网络上的详细信息
    ls          列出网络
    prune       删除所有未使用的网络
    rm          删除一个或多个网络

# 1
docker network connect [OPTIONS] NETWORK CONTAINER
Options:
      --alias strings           Add network-scoped alias for the container
      --driver-opt strings      driver options for the network
      --ip string               IPv4 address (e.g., "172.30.100.104")
      --ip6 string              IPv6 address (e.g., "2001:db8::33")
      --link list               Add link to another container
      --link-local-ip strings   Add a link-local address for the container
```

## 1.5 Container

Docker在隔离的容器中运行进程。容器是在主机上运行的进程。主机可以是本地的或远程的。当您执行 `docker run` 时，运行的容器进程是隔离的，因为它有自己的文件系统，自己的网络，以及自己独立于主机的隔离进程树。

### 1、运行容器

```
docker run [OPTIONS] IMAGE[:TAG|@DIGEST] [COMMAND] [ARG...]

OPTIONS:
	--name <container_name>	指定容器名称
	-c, --cpu-shares		CPU份额（相对权重）
	-d, --detach			以守护进程运行
	-e, --env list			设置环境变量
	-h, --hostname			容器主机名
	-it						以交互式终端模式运行
	-m, --memory bytes		内存限制
	-p, --publish			将容器端口映射到主机，[host_port]:[container_port]
	-P, --publish-all		将 EXPOSE 暴露的端口映射到随机端口
	--rm					容器退出时自动移除容器（同时移除匿名数据卷）
	--restart=always
	-v, --volume			绑定数据卷
	--volumes-from			从指定容器加载卷
	--mount					将文件系统挂载到容器
	-w, --workdir 			指定容器内的工作目录
    --privileged=true       授予此容器扩展权限（特权）
	-u                      -u 0 用root用户登录容器，而不是镜像的默认用户
    
COMMAND:
	env		查看镜像支持的环境变量
```

> 以下命令运行一个 `ubuntu` 容器，以交互方式附加到您的本地命令行会话，然后运行 `/bin/bash`

```
$ docker run -i -t ubuntu /bin/bash
```

当您运行此命令时，会发生以下情况（假设您使用的是默认注册表配置）：

- 如果您在本地没有`ubuntu`映像，Docker 会从您配置的注册表中提取它，就像您`docker pull ubuntu`手动运行一样。
- Docker 会创建一个新容器，就像您`docker container create` 手动运行命令一样。

- Docker 为容器分配一个读写文件系统，作为它的最后一层。这允许正在运行的容器在其本地文件系统中创建或修改文件和目录。

- Docker 创建了一个网络接口来将容器连接到默认网络，因为您没有指定任何网络选项。这包括为容器分配 IP 地址。默认情况下，容器可以使用主机的网络连接连接到外部网络。

- Docker 启动容器并执行`/bin/bash`. 因为容器以交互方式运行并附加到您的终端（由于`-i`and`-t` 标志），所以您可以在输出记录到终端时使用键盘提供输入。

- 当您键入`exit`终止`/bin/bash`命令时，容器会停止但不会被删除。您可以重新启动或删除它。

### 2、列出运行容器

```
docker ps [OPTIONS]
docker container ls

OPTIONS：
	-a 列出正在运行+历史运行（已停止）的容器
	-l 显示最近创建的容器
	-n 显示最近n个创建的容器
	-q 静默模式，只显示容器编号
```

### 3、终止容器

```
# 启动已停止运行的容器
docker start <容器ID/容器名>

# 重启容器
docker restart <容器ID/容器名>

# 停止容器
docker stop <容器ID/容器名>

# 强制停止容器
docker kill <容器ID/容器名>

# 删除已停止的容器
docker rm [OPTIONS] <容器ID/容器名 ...>
# 批量删除已停止的容器
docker container prune

OPTIONS：
	-f 强制删除，删除未停止的容器
	-v 删除容器时，同时删除数据卷
	
# 例：批量删除容器
docker rm -f $(docker ps -aq)
```

### 4、查看容器细节

```
# 获取容器的日志
docker logs [OPTIONS] <容器ID/容器名>

OPTIONS:
	-f 跟踪日志输出
	-n 显示条数，默认 all
	
# 显示容器的运行进程
docker top <容器ID/容器名>

# 返回 Docker 对象的底层信息
docker inspect [OPTIONS] <容器ID/容器名 ...>

# 查看具体项 -f,--format
docker inspect -f '{{json .State.Pid }}' <容器ID/容器名 ...> | jq

注：jq 可以对 json 文本进行格式化输出。
curl -O http://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
rpm -ivh epel-release-latest-7.noarch.rpm
yum install -y jq
```

### 5、进入容器

> 进入 -d 启动的守护状态容器。

```
# exit 会导致容器停止
docker attach [OPTIONS] <容器ID/容器名>

# exit 不会导致容器停止（推荐）
docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
```

### 6、导入导出

```
# 拷贝容器文件到主机
docker cp 容器ID:容器内路径 目的主机路径

# 导出
docker export 容器ID > 文件名.tar

# 导入
cat 文件名.tar | docker import - 镜像用户/镜像名:镜像版本号
docker import http://example.com/exampleimage.tgz example/imagerepo
```

## 1.6 Image

```sehll
Usage:  docker image COMMAND

Manage images

Commands:
  build       Build an image from a Dockerfile
  history     Show the history of an image
  import      Import the contents from a tarball to create a filesystem image
  inspect     Display detailed information on one or more images
  load        Load an image from a tar archive or STDIN
  ls          List images
  prune       Remove unused images
  pull        Download an image from a registry
  push        Upload an image to a registry
  rm          Remove one or more images
  save        Save one or more images to a tar archive (streamed to STDOUT by default)
  tag         Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE
```



### 1、搜索镜像

```
docker search <image_name>

	--limit 默认25
```

| NAME     | DESCRIPTION | STARS    | OFFICIAL | AUTOMATED      |
| -------- | ----------- | -------- | -------- | -------------- |
| 镜像名称 | 镜像说明    | 点赞数量 | 官方认证 | 是否是自动构建 |

### 2、下载镜像

```
# 标签：默认 latest
docker pull [选项] [Docker Registry 地址[:端口号]/]仓库名[:标签]
```

### 3、列出镜像

```
# 列出已下载到本地主机上的镜像
docker images <镜像名>
docker image ls <镜像名>

options：
	-a 列出本地所有镜像（中间层镜像）
	-q 只显示镜像ID
	
# 查看镜像、容器、数据卷所占空间
docker system df
```

| REPOSITORY | TAG          | IMAGE ID | CREATED  | SIZE     |
| ---------- | ------------ | -------- | -------- | -------- |
| 仓库名     | 标签（版本） | 镜像 ID  | 创建时间 | 镜像大小 |

### 4、镜像 Tag

```
# 为镜像创建新标签
docker image tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]
```

### 5、虚悬镜像

> 虚悬镜像：既没有仓库名，也没有标签的镜像。

```
# 显示虚悬镜像
docker image ls -f dangling=true

# 删除虚悬镜像
docker image prune
```

### 6、删除镜像

```
# 可以用镜像ID、镜像名删除镜像
docker image rm [选项] <镜像名 ...>

	-f 强制删除
	
简写：
docker rmi [选项] <镜像名 ...>
```

### 7、commit 定制镜像

> 不要使用 `docker commit` 定制镜像，定制镜像应该使用 `Dockerfile` 来完成。
>
> 镜像的分层：镜像是多层存储，每一层是在前一层的基础上进行的修改；而容器同样也是多层存储，是在以镜像为基础层，在其基础上加一层作为容器运行时的存储层。

```
# 将容器的存储层保存下来成为镜像
docker commit [选项] <容器ID或容器名> [<仓库名>[:<标签>]]

OPTIONS:
	-a, --author  用户
	-m, --message 信息
	
# 查看容器的具体改动（存储层）
docker diff <容器ID或容器名>	

# 查看镜像的历史记录
docker history [镜像名]
```

### 8、镜像分层

```
# 查看镜像创建时各层的命令
docker image history <镜像名>

--no-trunc 不截断输出
```

### 9、导出导入镜像

```shell
# 导出
docker image save [OPTIONS] IMAGE [IMAGE...]

Options:
  -o, --output string   Write to a file, instead of STDOUT

# 例1
docker image save -o base.tar base-api

# 例2
docker image save base-api > base.tar


# 导入
docker image load [OPTIONS]

Options:
  -i, --input string   Read from tar archive file, instead of STDIN
  -q, --quiet          Suppress the load output

# 例
docker image load -i base.tar
```







# 二、Docker Build

Docker Build 是 Docker Engine 最常用的功能之一。每当你创建镜像时，你都在使用 Docker Build。构建是软件开发生命周期的关键部分，允许您打包和捆绑代码并将其发送到任何地方。

> Dockfile 是用来构建 Docker 镜像的文本文件，是有一条条构建镜像所需的指令和参数构成的脚本。

## 1、Dockerfile 指令

##### FROM

> 继承基础镜像，当前镜像基于哪个镜像开始构建。

尽可能使用当前的官方图像作为图像的基础。我们推荐使用 `Alpine`，因为它受到严格控制且体积小（目前低于 6 MB），同时仍然是一个完整的 Linux 发行版。 

##### COPY & ADD

> COPY 将文件/目录从构建上下文目录中复制到新的一层的镜像内。

```
# 推荐
COPY [--chown=<user>:<group>] <src...> <dest>

OPTIONS:
	--chown=<user>:<group>	改变容器内文件的所属用户及所属组
	--from=<name>			从 from 指定的构建阶段中寻找源文件 <src>
	
注：
    - <src>  可以是多个，甚至可以是通配符。
    - <dest> 可以是镜像内的绝对路径，也可以是相对于工作目录的相对路径（WORKDIR 指定）。
    - <dest> 不存在时会自动创建。
    - 使用 COPY 指令，源文件的各种元数据（如读、写、执行权限、文件变更时间等）都会保留。
```

> ADD 指令和 COPY 的格式和性质基本一致。
>
> [Dockerfile 最佳实践文档](https://www.kancloud.cn/docker_practice/docker_practice/972883)：推荐使用 COPY 复制文件，仅在需要自动解压缩的场合使用 ADD。

```
# 仅在需要自动解压时使用
ADD [--chown=<user>:<group>] <源路径...> <目标路径>

OPTIONS:
	--chown=<user>:<group> 改变容器内文件的所属用户及所属组

注：
	- 当 <源路径> 是一个 URL 时，Docker 引擎会尝试下载这个文件放到 <目标路径> 去，下载的文件权限为 600，可以另外添加 RUN 层修改文件权限。
	- 当 <源路径> 是一个 tar 压缩文件时，压缩格式为 gzip、bzip2、xz 的情况下，ADD 指令将会自动解压缩后放到 <目标路径> 去。
	- ADD 指令会令镜像构建缓存失效，从而可能会令镜像构建变得比较缓慢。
```

##### RUN

> RUN 是在 `docker build` 时运行。
>
> RUN 指令将在当前镜像上加新的一层，并执行任何命令和提交结果，生成的提交镜像将用于 Dockfile 中的后续步骤。（每一个 `RUN` 都是启动一个容器、执行命令、然后提交存储层文件变更。）
>
> 分层 RUN 指令和生成提交符合 Docker 核心概念，提交成本低，并且可以通过 docker history 中的任意步骤创建容器，像 git 代码控制一样。

```
# shell 格式
# 默认以 /bin/sh -c <command> 格式运行
# 等价于 RUN ["/bin/sh", "-c", "command"]
RUN <command>

# exec 格式（推荐）
# exec 形式会被解析为JSON格式，必须用双引号
RUN ["可执行文件", "参数1", "参数2", ...]
```

注：只有 RUN、COPY、ADD 指令会改变层，其他指令会创建临时中间图像，并且不会增加构建的大小。

##### CMD

> CMD 是在 `docker run` 时运行，可以有多个 CMD 指令，但只有最后一个生效，且会被 `docker run` 之后的参数覆盖。
>
> CMD 为容器指定默认要运行的程序及启动命令，程序运行结束，容器也就结束。

```
# shell 格式
# 等价于 CMD ["/bin/sh", "-c", "command"]
CMD <命令>

# exec 格式（推荐）
CMD ["可执行文件", "参数1", "参数2", ...]

# 参数列表格式
# 在指定 ENTRYPOINT 指令后，用 CMD 指定具体参数
CMD ["参数1", "参数2", ...]
```

##### ENTRYPOINT

> 类似于 CMD 指令，都是在指定容器启动程序及参数。但其不会被 `docker run` 的命令行参数指定的指令所覆盖，而且这些命令行参数会被当作参数送给 ENTRYPOINT 指令指定的程序。 
>
> 在 docker run 时，也可以用 --entrypoint 选项来覆盖。
>
> 只能出现一次。

```
<ENTRYPOINT> "<CMD>"
ADD、COPY、ENV、EXPOSE、FROM、LABEL、USER、WORKDIR、VOLUME、STOPSIGNAL、ONBUILD、RUN
```

##### ENV

> 设置环境变量，ADD、COPY、ENV、EXPOSE、FROM、LABEL、USER、WORKDIR、VOLUME、STOPSIGNAL、ONBUILD、RUN 这些指令都支持环境变量的展开。
>
> 每 ENV 行创建一个新的中间层，就像 RUN 命令一样。 

```
ENV <key> <value>
ENV <key1>=<value1> <key2>=<value2>...

# 这样删除环境变量无效
RUN echo $ADMIN_USER > ./mark
RUN unset ADMIN_USER

# 在单个层中设置、使用和取消设置变量
RUN export ADMIN_USER="mark" \
    && echo $ADMIN_USER > ./mark \
    && unset ADMIN_USER
```

##### ARG

> 和 ENV 效果一样，都是设置环境变量。
>
> 其默认值可以在 `docker build --build-arg 参数名=值` 时覆盖。
>
> 注：若 ARG 在 FROM 之前指定，则只能用于 FROM 中，要想在 FROM 之后使用，须再次指定。

```
ARG <参数名>[=<默认值>]
```

##### VOLUME

> 定义匿名卷
>
> 运行时，使用 `docker run -d -v mydata:/data xxx` 覆盖这个挂载设置。

```
VOLUME ["<路径1>", "<路径2>"...]
```

##### EXPOSE

> 声明运行时容器提供的服务端口（仅声明暂未开启）。
>
> 声明好处：
>
> - 便于理解这个镜像服务的守护端口，以方便映射；
> - 再运行时使用 `docker run -P` 时，会自动随机映射 EXPOSE 的端口。

```
EXPOSE <端口1> [<端口2>...]
```

##### WORKDIR

> 用来指定工作目录（或者称为当前目录），以后各层的当前目录就被改为指定的目录，若目录不存在 WORKDIR 会帮你自动创建。

```
WORKDIR <工作目录路径>
```

##### USER

> 改变之后层的执行 `RUN`， `CMD` 以及 `ENTRYPOINT` 这类命令的身份。
>
> 注意，`USER` 只是帮助你切换到指定用户而已，这个用户必须是事先建立好的，否则无法切换。

```
USER <用户名>[:<用户组>]
```

##### HEALTHCHECK

> `HEALTHCHECK` 只可以出现一次，如果写了多个，只有最后一个生效。 

```
HEALTHCHECK [选项] CMD <命令>：设置检查容器健康状况的命令
HEALTHCHECK NONE：如果基础镜像有健康检查指令，使用这行可以屏蔽掉其健康检查指令

选项：
    --interval=<间隔>：两次健康检查的间隔，默认为 30 秒；
    --timeout=<时长>：健康检查命令运行超时时间，如果超过这个时间，本次健康检查就被视为失败，默认 30 秒；
    --retries=<次数>：当连续失败指定次数后，则将容器状态视为 unhealthy，默认 3 次。
```

##### ONBUILD

> `ONBUILD` 是一个特殊的指令，它后面跟的是其它指令，比如 `RUN`, `COPY` 等，而这些指令，在当前镜像构建时并不会被执行。只有当以当前镜像为基础镜像，去构建下一级镜像的时候才会被执行。

```
ONBUILD <其它指令>
```

##### LABEL

> 用来给镜像以键值对的形式添加一些元数据。

```
LABEL <key>=<value> <key>=<value> <key>=<value> ...
```

##### SHELL

> SHELL 指令可以指定 RUN ENTRYPOINT CMD 指令的 shell，Linux 中默认为 `["/bin/sh", "-c"]`。

```
SHELL ["executable", "parameters"]
```

## 2、从 Dockerfile 构建镜像（docker build）

> Docker 能够通过 stdin 管道传输 Dockerfile 使用本地或远程构建上下文来构建镜像。

```
docker build [OPTIONS] PATH | URL | -

OPTIONS:
	-t,--tag	镜像名称
	-f			Dockerfile 的名称（默认 'PATH/Dockerfile'）
	--no-cache	不使用缓存
	
PARAMETERS:
	PATH	使用本地（PATH）构建上下文中的 Dockerfile 构建镜像
	URL		使用远程（URL）构建上下文中的 Dockerfile 构建镜像
	-		通过 stdin 管道传输 Dockerfile 构建镜像
			用 `-` 作为文件名来指示 Docker 从 stdin 读取 Dockerfile
```

> Docker 的工作原理（了解构建上下文）

- Docker Client 将构建命令后面指定的 PATH (`.`) 目录下的所有文件打包成一个 tar 包，发送给 Docker Daemon；

- Docker Daemon 收到客户端发送的 tar 包后解压（得到：构建上下文），根据 Dockerfile 里面的指令进行镜像的分层构建；

- 如果不指定 Dockerfile，客户端默认在指定上下文路径下寻找 Dockerfile 文件；Dockerfile 可以不在构建上下文路径下，此时需要通过 -f 参数明确指定。

#### （1）使用 PATH 构建上下文中的 Dockerfile 构建镜像

```
# 将当前目录作为构建上下文，并指定镜像名称
# 默认用 ./Dockerfile 构建镜像
docker build -t app:v1 .
```

#### （2）通过 stdin 中的 Dockerfile 构建映像，而不发送构建上下文（仅包含 Dockerfile）

> 通过 stdin 管道传输 Dockerfile 对于在不将 Dockerfle 写入磁盘的情况下执行一次性构建非常有用，或者在生成 Dockerfil 之后不应继续存在的情况下也非常有用。

```
docker build -t app:v2 -<<EOF
FROM alpine
RUN echo "hello world"
EOF

# 如果使用此语法构建，使用 COPY 或 ADD 的 Dockerfile 将失败。
docker build -t test -<<EOF
FROM alpine
COPY main.go /
RUN echo "hello world"
EOF

Sending build context to Docker daemon  2.048kB
Step 1/3 : FROM alpine
 ---> 9c6f07244728
Step 2/3 : COPY main.go /
COPY failed: file not found in build context or excluded by .dockerignore: stat main.go: file does not exist
```

#### （3）使用 stdin 中的 Dockerfile 从本地构建上下文构建镜像

```
docker build -t app:v3 -f- . <<EOF
FROM golang:1.17-alpine
COPY main.go /
RUN go run /main.go
EOF

Sending build context to Docker daemon  5.167kB
Step 1/3 : FROM golang:1.17-alpine
 ---> 270c4f58750f
Step 2/3 : COPY main.go /
 ---> 3330649f8230
Step 3/3 : RUN go run /main.go
 ---> Running in 15793f4c0842
Hello YCZ!
Removing intermediate container 15793f4c0842
 ---> 2fdeff45a2e8
Successfully built 2fdeff45a2e8
Successfully tagged app:v3
```

#### （4）使用 stdin 中的 Dockerfile 从远程构建上下文构建镜像

```
docker build -t app:v4 -f- https://github.com/docker-library/hello-world.git <<EOF
FROM busybox
COPY hello.c ./
EOF

# Docker 会在本机执行 git clone，并将这些文件作为构建上下文发送给守护进程
# 因此需要再主机上安装 git
```

#### （5）使用 .dockerignore 排除

- 使用 Dockfile 创建镜像时要添加 `.dockerignore` 文件或使用干净的工作目录。
- `.dockerignore` 需要在构建上下文的根目录中。

## 3、案例

```
# 目录结构

/app
├─ Dockerfile
├─ main.go
└─ ...
```

#### （1）创建 Dockerfile

> Dockerfile

```dockerfile
# syntax=docker/dockerfile:1 
# 可选，指示 Docker 构建器在解析 Dockerfile 时使用什么语法

# FROM 继承
# 告诉 Docker 我们的镜像中包含 golang:1.17-alpine 镜像的所有功能
FROM golang:1.17-alpine

# 工作目录
WORKDIR /app

# 将当前目录复制到镜像中的 /app
COPY . .

# 执行 go build
RUN go build -o server main.go

# CMD 可执行程序必须为绝对路径
CMD ["/app/server"]
```

> main.go

```go
package main

import (
    "fmt"
)

func main() {
    fmt.Println("Hello YCZ!")
}
```

#### （2）构建镜像并运行

```
# 构建
> cd /app
> docker build -t hello-ycz:v1 .

# 查看镜像（发现生成的镜像过于庞大：316MB）
> docker images
REPOSITORY    TAG           IMAGE ID       CREATED          SIZE
hello-ycz     v1            98fa0a4e5ab9   33 minutes ago   316MB（太大）

# 运行镜像
docker run --name=hello-ycz-v1 hello-ycz:v1
```

#### （3）多阶段构建（镜像小）

> MultistageDockerfile

```dockerfile
## Build
# 第一构建阶段：仅用于 build 出可执行文件
# AS <NAME> 命名构建阶段
FROM golang:1.18-alpine AS build

WORKDIR /app

COPY . .

RUN go build -o server main.go

## Deploy
# 第二构建阶段：这里的改变将影响生成的镜像
# 轻量的 alpine：适用于静态二进制文件的精益部署
FROM alpine

WORKDIR /app

# 从 build 阶段复制 server
# 该文件仅存在于前一个 Docker 阶段，因此需要用 --from=build 来复制它
COPY --from=build /app/server /app


# 由于 alpine 镜像使用的是 musl libc 而不是 gnu libc
# /lib64/ 目录是不存在的，但它们是兼容的，可以创建 lib64 软链接
# 动态链接库位置错误导致：
#  - exec /app/server: no such file or directory
#  - sh: /app/server: not found
#使用 golang:1.18-alpine 不需要做兼容处理
#RUN mkdir /lib64 && ln -s /lib/libc.musl-x86_64.so.1 /lib64/ld-linux-x86-64.so.2

# 定义暴露的端口
EXPOSE 9999

CMD ["/app/server", "--config", "/app/app.yaml"]
```

> 构建 & 运行

```
# 构建
> docker build -t hello-ycz -f MultistageDockerfile .

# 查看镜像（此时镜像：7.32MB）
> docker images
REPOSITORY    TAG           IMAGE ID       CREATED          SIZE
hello-ycz     latest        5f7d56bf57ad   54 seconds ago   7.32MB（小）
hello-ycz     v1            98fa0a4e5ab9   40 minutes ago   316MB

# 运行镜像
> docker run --name=hello-ycz hello-ycz
```

## 4、BuildKit 构建镜像

```
# 启用 BuildKit 构建
DOCKER_BUILDKIT=1 docker build .

# 写入配置文件 `/etc/docker/daemon.json`
{
	"features": { "buildkit": true } 
}
```





# 三、Docker Compose

> `Compose` 是 Docker 官方的开源项目，负责实现对 Docker 容器集群的快速编排。 
>
> `Compose` 是一个用于定义和运行多个 Docker 容器的工具。
>
> 通过一个单独的 `docker-compose.yml` 模板文件（YAML 格式）来定义一组相关联的应用容器为一个项目（project）。
>
> - 服务 (`service`)：一个应用容器，实际上可以运行多个相同镜像的实例。
> - 项目 (`project`)：由一组关联的应用容器组成的一个完整业务单元。

## 1、安装

> 如果您为 Windows、Mac 或 Linux 安装了 Docker Desktop，那么您已经拥有 Docker Compose！

```
$ docker compose version
```

> 将 Docker Compose 作为单独的包安装

```
## 1 安装为可执行程序（docker-compose）
# 下载
$ curl -L https://github.com/docker/compose/releases/download/v2.7.0/docker-compose-linux-x86_64 > /usr/local/bin/docker-compose

# 可执行权限
$ chmod +x /usr/local/bin/docker-compose

# 卸载
$ sudo rm /usr/local/bin/docker-compose

# 安装完成
$ docker-compose version

## 2 安装为 Docker 插件（docker compose）
$ cp /usr/local/bin/docker-compose /usr/local/lib/docker/cli-plugins

# 安装完成
$ docker-compose version
```

## 2、docker-compose 模板

> 注意：不能使用 tab 缩进，只能用空格。

```yml
# 模板版本
version: "3.7"

# 项目名称（默认：directory name）
name: "app"

# 定义服务（容器）
services:
    # 第一个服务
    web:
        # 容器名
        container_name: web
        
        # 镜像名（TAG 未设置或为空时，使用 latest）
        image: "web:${TAG:-latest}"  
        
        # 网络配置（数组）
        networks:                    
            - web-network
            
        # 端口映射（数组）
        ports:                       
            - 8080:80
            
        # 数据挂载（数组）
        # 同时配置挂载、绑定到同一目录，后面的配置会覆盖前面的，后面的生效
        volumes:                     
            - web-data:/app    # 挂载数据卷
        #   - /data/web:/app   # 文件系统挂载
        
        # 命令行
        command: "bash"
        
        # 在容器中设置环境变量
        environment:                 
            - TAG=v1   # docker run -e VARIABLE=VALUE ...
            - DEBUG    # docker run -e VARIABLE ... 将环境变量从 shell 直接传递给容器
        
        # 将环境变量从外部文件传递到容器（默认从：./.env）
        env_file:                    
            - .env     # docker run --env-file=FILE ...

    # 第二个服务
    
    # ...
 
 
# 定义数据卷
# docker-compose 文件中绑定未定义的数据卷时，不会自动创建，需手动创建
volumes:
    web-data:
        # 以下配置可以省略，主要用于更改 volume 的位置
        driver: local
        driver_opts:
            type: 'none'
            o: 'bind'
            device: '/data/docker/ES/volumes'
    
# 定义网络
networks:
    web-network:
```

> 环境变量：

- `${TAG:-latest}`：如果环境变量在环境中未设置或为空，则使用默认值。

- `${TAG-latest}`：如果环境变量在环境中未设置，则使用默认值。

- `${TAG:?latest}`：如果环境变量在环境中未设置或为空，则退出，并显示 err 信息。

- `${TAG?latest}`：如果环境变量在环境中未设置，则退出，并显示 err 信息。

- 环境变量的优先级：？？？

  - .env 文件 > docker-compose 文件

  docker-compose 文件 > 命令行参数 > .env 文件 > Dockerfile > 未定义变量

> 环境文件：

- Compose 期望 `.env` 文件中的每一行都具有 `KEY=VALUE` 格式 。
- 引号没有特殊处理。这意味着它们是 VALUE 的一部分。所以字符串不用加 `""`。

## 3、命令详解

> 如无特殊说明，命令的对象是项目，项目中所有的服务都会受命令影响。

```
# 基本格式
docker compose [OPTIONS] COMMAND

Define and run multi-container applications with Docker

Options:
      --ansi string                Control when to print ANSI control characters ("never"|"always"|"auto") (default "auto")
      --compatibility              Run compose in backward compatibility mode
      --dry-run                    Execute command in dry run mode
      --env-file stringArray       指定环境配置文件
  -f, --file stringArray           指定 Compose 配置文件 
                                   (default: docker-compose.yml)
      --parallel int               Control max parallelism, -1 for unlimited (default -1)
      --profile stringArray        Specify a profile to enable
      --progress string            Set type of progress output (auto, tty, plain, quiet) (default "auto")
      --project-directory string   指定工作目录
                                   (default: 第一个指定 Compose 配置文件的路径)
  -p, --project-name string        项目名称

Commands:
  attach      Attach local standard input, output, and error streams to a service's running container
  build       Build or rebuild services
  config      Parse, resolve and render compose file in canonical format
  cp          Copy files/folders between a service container and the local filesystem
  create      Creates containers for a service
  down        Stop and remove containers, networks
  events      Receive real time events from containers
  exec        Execute a command in a running container
  images      List images used by the created containers
  kill        Force stop service containers
  logs        View output from containers
  ls          List running compose projects
  pause       Pause services
  port        Print the public port for a port binding
  ps          List containers
  pull        Pull service images
  push        Push service images
  restart     Restart service containers
  rm          Removes stopped service containers
  run         Run a one-off command on a service
  scale       Scale services
  start       Start services
  stats       Display a live stream of container(s) resource usage statistics
  stop        Stop services
  top         Display the running processes
  unpause     Unpause services
  up          Create and start containers
  version     Show the Docker Compose version information
  wait        Block until the first service container stops
  watch       Watch build context for service and rebuild/refresh containers when files are updated


# 生成或重建服务
docker compose build [SERVICE...]

# 将 compose 文件转换为平台的规范格式并打印
docker compose convert
```

```

```

（2）



## 4、案例

### （1）Dockerfile

> 用 Dockerfile 定义应用程序的环境，以便可以在任何地方复制它。

```
# Dockerfile

...
```

### （2）docker-compose.yml

> 在 `docker-compose.yml` 中定义组成应用程序的服务，以便它们可以在隔离环境中一起运行。

```dockerfile
    # 第二个服务
    database:
        container_name: mysql-3306
        image: mysql:5.7
        ports:
            - 3306:3306
        volumes:
            - /data/mysql:/var/lib/mysql
        environment:
            - MYSQL_ROOT_PASSWORD=root

    # 第三个服务
    cache:
        container_name: redis
        image: redis
        ports: 
            - 6379:6379
        volumes: 
            - /data/redis:/data

```

### （3）docker compose up

> `docker compose up`  启动并运行您的整个应用程序。

```
docker-compose up -d
```

注意：

- `docker run` 时，会自动创建不存在的数据卷，但在 composer 中不会自动创建，需要在顶层 volumes 部分定义数据卷，然后才能在服务中挂载数据卷。

### （4）docker compose 资源限制

> 定义 docker-compose.yml 文件

```yaml
# docker compose --compatibility up -d

version: "3.7"

name: "mongodb"

services:
  mongo_27017:
    container_name: mongo_27017
    image: mongo_7.0.5
    restart: always
    ports:
      - "27017:27017"
    volumes:
      - /data/docker/mongodb/db:/data/db
    deploy:
      resources:
        # mongo_27017 服务的资源限制为 50% 的 CPU 和 32G 的内存
        limits:
          cpus: "0.50"
          memory: 32G
        # 同时，这个服务最少需要 25% 的 CPU 和 1G 内存
        reservations:
          cpus: "0.25"
          memory: 1G

  mongo_27022:
    container_name: mongo_27022
    image: mongo_7.0.5
    restart: always
    ports:
      - "27022:27017"
    volumes:
      - /data/docker/mongodb/db_64_79:/data/db
    deploy:
      resources:
        limits:
          memory: 32G

  mongo_27023:
    container_name: mongo_27023
    image: mongo_7.0.5
    restart: always
    ports:
      - "27023:27017"
    volumes:
      - /data/docker/mongodb/db_80_95:/data/db
    deploy:
      resources:
        limits:
          memory: 32G

  mongo_27024:
    container_name: mongo_27024
    image: mongo_7.0.5
    restart: always
    ports:
      - "27024:27017"
    volumes:
      - /data/docker/mongodb/db_96_111:/data/db
    deploy:
      resources:
        limits:
          memory: 32G

  mongo_27025:
    container_name: mongo_27025
    image: mongo_7.0.5
    restart: always
    ports:
      - "27025:27017"
    volumes:
      - /data/docker/mongodb/db_112_127:/data/db
    deploy:
      resources:
        limits:
          memory: 32G

```

> 启动服务

```sh
# 需要 --compatibility 以兼容模式运行，否则限制不生效
$ docker compose --compatibility up -d
[+] Running 6/6
 ✔ Network mongodb_default  Created
 ✔ Container mongo_27023    Started
 ✔ Container mongo_27025    Started
 ✔ Container mongo_27017    Started
 ✔ Container mongo_27024    Started
 ✔ Container mongo_27022    Started      
```

> 验证

```sh
# docker 版本 version 26.1.3（低版本没有此功能）
$ docker compose stats
CONTAINER ID   NAME          CPU %     MEM USAGE / LIMIT   MEM %     NET I/O           BLOCK I/O         PIDS
782a410c0f42   mongo_27017   0.02%     440.2MiB / 32GiB    1.34%     66.3kB / 1.67MB   153MB / 4.12MB    35
1c94a819b194   mongo_27023   0.01%     203MiB / 32GiB      0.62%     73.8kB / 99.4kB   54.1MB / 5.02MB   32
fa2854f4c535   mongo_27022   0.03%     818.4MiB / 32GiB    2.50%     96kB / 1.46MB     437MB / 4.37MB    32
16b934bb4097   mongo_27024   0.02%     203.3MiB / 32GiB    0.62%     75.7kB / 102kB    71MB / 4.35MB     32
0104bcdf3ed0   mongo_27025   1.55%     829.1MiB / 32GiB    2.53%     105kB / 1.72MB    455MB / 4.37MB    32
```



# 四、Docker Hub







# 五、其它

## 1、Portainer Docker 可视化工具

```shell
docker run --name portainer -d \
  --restart=always \
  -p 9000:9000 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /data/docker-run/portainer/data:/data \
  portainer/portainer
```



## 2、CIG 容器重量级监控系统

CAdvisor：容器资源监控工具，包括 内存、CPU、网络IO、磁盘IO等。

InfluxDB：Go编写的一个开源分布式时序、事件和指标数据库。

Granfana：数据监控分析可视化平台。









