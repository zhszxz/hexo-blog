---
title: Docker
tags: Docker
categories:
  - Java
  - Docker
date: 2025-07-01 10:16:52
---

# Docker 学习笔记

# 一、Docker 基础概念

## 1\.1 什么是 Docker

Docker 是一款开源的**容器化部署工具**，基于 Go 语言开发，能够将应用程序及其依赖、运行环境、配置文件统一打包为镜像，实现**一次构建、随处运行**。

其核心解决了传统部署的痛点：环境不一致、依赖冲突、部署繁琐、资源占用高、迁移困难等问题，广泛应用于开发、测试、生产全流程。

## 1\.2 核心优势

- **轻量高效**：容器共享宿主机内核，无需完整操作系统，资源占用远低于虚拟机，启动速度秒级响应

- **环境一致**：打包完整运行环境，彻底解决“本地能跑、服务器报错”的环境差异问题

- **便携可移植**：镜像可跨系统、跨服务器迁移，部署流程标准化、自动化

- **隔离性强**：容器之间资源、进程、网络相互隔离，互不干扰

- **弹性扩展**：配合 Compose、K8s 可快速实现服务扩容、缩容与集群部署

## 1\.3 核心四大组件

- **Docker 引擎（Docker Engine）**：核心服务，包含守护进程、API、命令行工具，是 Docker 运行的基础

- **镜像（Image）**：只读的应用模板，包含应用、依赖、环境、配置，是创建容器的模板，不可修改

- **容器（Container）**：镜像运行后的实例，可读写、可启停、可删除，是独立的运行环境

- **仓库（Registry）**：存放镜像的远程仓库，用于镜像的上传、下载、共享，官方仓库为 Docker Hub

## 1\.4 容器 vs 虚拟机（核心区别）

| 对比维度   | Docker 容器                        | 虚拟机                         |
| ---------- | ---------------------------------- | ------------------------------ |
| 系统层级   | 共享宿主机内核，仅隔离应用层       | 独立完整操作系统，包含独立内核 |
| 资源占用   | 极低，MB 级资源占用                | 极高，GB 级资源占用            |
| 启动速度   | 秒级启动                           | 分钟级启动                     |
| 隔离性     | 进程、网络、文件系统隔离，内核共享 | 完全硬件级隔离，安全性更高     |
| 部署复杂度 | 简单、轻量化、自动化程度高         | 繁琐、配置复杂、占用资源多     |

# 二、Docker 常用命令

所有 Docker 命令均基于 `docker` 核心指令，分为环境信息、镜像操作、容器操作三大类，以下为高频实用命令。

## 2\.1 环境信息命令

- `docker \-\-version`：查看 Docker 版本信息

- `docker info`：查看 Docker 系统详细信息、容器/镜像数量、仓库配置

- `docker login`：登录 Docker 仓库（默认 Docker Hub）

- `docker logout`：退出仓库登录

## 2\.2 镜像操作命令（核心）

- `docker images`：查看本地所有镜像（`\-a` 查看全部镜像，含虚悬镜像）

- `docker search 镜像名`：从仓库搜索镜像（例：`docker search nginx`）

- `docker pull 镜像名:版本`：下载镜像（不加版本默认 latest 最新版）

- `docker rmi 镜像ID/镜像名`：删除本地镜像（需先删除对应容器）

- `docker build \-t 镜像名:版本 \.`：根据 Dockerfile 构建镜像（末尾 \. 为当前目录）

- `docker tag 原镜像:版本 新镜像:版本`：给镜像打标签，用于仓库推送

- `docker save \-o 文件名\.tar 镜像名:版本`：导出镜像为本地压缩包

- `docker load \-i 文件名\.tar`：导入本地镜像压缩包

## 2\.3 容器操作命令（高频）

- `docker ps`：查看**正在运行**的容器（`\-a` 查看所有容器，含已停止）

- `docker run \[参数\] 镜像名:版本`：创建并启动容器（最核心命令）

- `docker start 容器ID/容器名`：启动已停止的容器

- `docker stop 容器ID/容器名`：优雅停止容器（等待进程结束）

- `docker kill 容器ID/容器名`：强制杀死容器进程

- `docker restart 容器ID/容器名`：重启容器

- `docker rm 容器ID/容器名`：删除容器（`\-f` 强制删除运行中容器）

- `docker exec \-it 容器ID 命令`：进入运行中容器终端（例：`docker exec \-it nginx /bin/bash`）

- `docker logs 容器ID`：查看容器日志（`\-f` 实时查看日志）

- `docker cp 本地路径 容器ID:容器路径`：本地文件拷贝到容器

- `docker cp 容器ID:容器路径 本地路径`：容器文件拷贝到本地

## 2\.4 docker run 常用核心参数

- `\-d`：后台守护进程运行容器

- `\-it`：交互式终端运行，进入容器内部

- `\-\-name 容器名`：自定义容器名称

- `\-p 宿主机端口:容器端口`：端口映射（外部访问容器核心）

- `\-v 宿主机目录:容器目录`：数据卷挂载，持久化数据

- `\-\-restart=always`：容器退出后自动重启，开机自启

- `\-\-network=网络名`：指定容器所属网络

# 三、Docker 存储

Docker 容器默认数据存储在容器内部，容器删除后数据会丢失，因此需要**数据持久化**方案，核心分为三种存储方式。

## 3\.1 存储分类

### 3\.1\.1 绑定挂载（Bind Mount）

直接将**宿主机指定目录/文件**挂载到容器内目录，双向同步数据，适用于开发调试场景。

**特点**：

- 目录完全依赖宿主机路径，灵活性高

- 宿主机修改文件，容器实时生效，反之亦然

- 移植性差，更换服务器需重新配置路径

**使用示例**：

```Plain Text
docker run -d -v /home/test:/usr/share/nginx/html --name nginx nginx
```

### 3\.1\.2 数据卷（Volume）—— 官方推荐

Docker 统一管理的专属存储目录，存放在 Docker 工作目录中，独立于宿主机文件系统，是生产环境首选持久化方案。

**特点**：

- 完全由 Docker 管理，无需手动维护宿主机路径

- 独立于容器，容器删除后数据卷保留，可复用

- 支持多容器共享数据卷，移植性、安全性更强

**常用数据卷命令**：

- `docker volume ls`：查看所有数据卷

- `docker volume create 卷名`：创建自定义数据卷

- `docker volume inspect 卷名`：查看数据卷详细存储路径

- `docker volume rm 卷名`：删除指定数据卷

- `docker volume prune`：清理无用悬空数据卷

**使用示例**：

```Plain Text
docker run -d -v nginx-data:/usr/share/nginx/html --name nginx nginx
```

### 3\.1\.3 临时文件系统（tmpfs）

数据存储在宿主机内存中，仅临时使用，容器停止后数据立即清空，适用于临时缓存、敏感临时数据场景。

## 3\.2 三种存储方式对比

| 存储方式       | 持久化 | 移植性 | 适用场景                             |
| -------------- | ------ | ------ | ------------------------------------ |
| 绑定挂载       | 支持   | 差     | 本地开发、代码热更新调试             |
| 数据卷         | 支持   | 优     | 生产环境、数据持久化、多容器共享数据 |
| tmpfs 临时存储 | 不支持 | 一般   | 临时缓存、敏感临时数据存储           |

# 四、Dockerfile

Dockerfile 是**镜像构建脚本**，由一系列指令组成，用于自动化、标准化构建自定义镜像，替代手动打包镜像的繁琐操作，是容器化项目的核心文件。

## 4\.1 Dockerfile 核心特性

- 指令自上而下顺序执行，每条指令生成一层镜像层

- 支持镜像层缓存，重复构建时跳过未修改指令，提升构建速度

- 所有指令均为只读，构建完成后生成固定镜像

## 4\.2 常用核心指令（最全）

- **FROM**：基础镜像，必须是文件第一条指令，指定当前镜像基于哪个镜像构建
  示例：`FROM centos:7`、`FROM java:8`

- **MAINTAINER**：镜像作者信息（姓名、邮箱），可选指令
  示例：`MAINTAINER test test@163\.com`

- **WORKDIR**：设置容器工作目录，进入容器默认跳转该目录，目录不存在自动创建
  示例：`WORKDIR /usr/local/project`

- **COPY**：将本地文件/目录拷贝到容器内，仅支持本地资源
  示例：`COPY \./app\.jar /usr/local/app\.jar`

- **ADD**：高级拷贝，支持本地文件、远程URL文件、自动解压压缩包
  示例：`ADD \./test\.tar\.gz /usr/local/`

- **RUN**：构建镜像时执行的命令，用于安装依赖、配置环境
  示例：`RUN yum install \-y nginx`

- **EXPOSE**：声明容器对外暴露的端口，仅为标识，不主动开放端口
  示例：`EXPOSE 8080`

- **ENV**：设置容器环境变量，全局生效
  示例：`ENV SPRING\_PROFILES\_ACTIVE prod`

- **CMD**：容器启动后默认执行的命令，仅最后一条生效，可被 run 命令覆盖
  示例：`CMD \[\&\#34;java\&\#34;,\&\#34;\-jar\&\#34;,\&\#34;app\.jar\&\#34;\]`

- **ENTRYPOINT**：容器入口命令，优先级高于 CMD，不会被 run 覆盖，用于固定启动指令

- **VOLUME**：声明容器数据卷，用于数据持久化

## 4\.3 CMD 与 ENTRYPOINT 核心区别

- **CMD**：容器启动默认命令，执行 `docker run` 时可直接替换，多条仅最后一条生效

- **ENTRYPOINT**：固定容器启动入口，run 指令无法覆盖，可配合 CMD 实现参数传递

## 4\.4 完整 Dockerfile 示例（Java 项目）

```Plain Text
# 基础镜像：jdk8
FROM java:8
# 作者信息
MAINTAINER docker-test 123@qq.com
# 工作目录
WORKDIR /usr/local/app
# 拷贝项目jar包到容器
COPY ./demo.jar app.jar
# 暴露端口
EXPOSE 8080
# 容器启动命令
CMD ["java","-jar","app.jar"]
```

## 4\.5 镜像构建命令

```Plain Text
# 构建镜像 -t 指定镜像名和版本 . 代表当前目录读取Dockerfile
docker build -t demo-app:1.0 .
```

# 五、Docker 仓库

Docker 仓库是存储、分发 Docker 镜像的服务，用于镜像的共享、版本管理，分为**公共仓库**和**私有仓库**。

## 5\.1 仓库分类

### 5\.1\.1 公共仓库（Docker Hub）

Docker 官方公共仓库，免费提供大量开源镜像（Nginx、MySQL、Java、Redis 等），是最常用的镜像来源。

**核心操作**：

- 搜索镜像：`docker search 镜像名`

- 下载镜像：`docker pull 镜像名:版本`

- 上传镜像：需先登录账号、给镜像打标签，再推送

### 5\.1\.2 私有仓库

企业内部搭建的私有镜像仓库，用于存储项目自定义镜像，保证数据安全、内网快速部署，常见方案：Docker Registry、Harbor。

## 5\.2 镜像上传完整流程（Docker Hub）

1. 登录仓库：`docker login`（输入账号密码）

2. 给本地镜像打标签（规范：用户名/镜像名:版本）：`docker tag demo\-app:1\.0 用户名/demo\-app:1\.0`

3. 推送镜像到仓库：`docker push 用户名/demo\-app:1\.0`

4. 退出登录：`docker logout`

## 5\.3 私有仓库 Registry 搭建（简易版）

```Plain Text
# 启动私有仓库容器
docker run -d -p 5000:5000 --name registry registry
# 测试私有仓库镜像推送
docker tag 本地镜像:版本 127.0.0.1:5000/镜像名:版本
docker push 127.0.0.1:5000/镜像名:版本
```

# 六、Docker 网络

Docker 网络用于实现**容器与容器、容器与宿主机、容器与外网**的通信，Docker 内置多种网络模式，适配不同业务场景。

## 6\.1 查看网络基础命令

- `docker network ls`：查看所有 Docker 网络

- `docker network inspect 网络名`：查看网络详细信息、接入容器

- `docker network create 网络名`：创建自定义网络

- `docker network rm 网络名`：删除自定义网络

## 6\.2 四大核心网络模式

### 6\.2\.1 bridge 桥接模式（默认模式）

Docker 默认网络模式，所有容器默认接入 bridge 网桥。

**特点**：

- 同一网桥下容器可通过 IP 相互通信

- 容器独立 IP，与宿主机网段隔离

- 支持端口映射，外网可访问容器服务

- 不同网桥下容器默认无法互通

### 6\.2\.2 host 主机模式

容器直接复用宿主机网络，无独立 IP，端口、网络完全与宿主机共享。

**特点**：

- 网络性能最高，无网络隔离开销

- 无需端口映射，宿主机端口直接对应容器端口

- 隔离性差，容器占用宿主机端口，易冲突

**使用方式**：`docker run \-\-network=host 镜像名`

### 6\.2\.3 none 无网络模式

关闭容器所有网络功能，容器无网卡、无 IP，完全隔离网络，适用于**纯本地离线任务**，安全性极高。

### 6\.2\.4 自定义 bridge 网络（生产推荐）

用户手动创建的桥接网络，相较于默认 bridge 网络，新增**域名解析功能**。

**核心优势**：同一自定义网络下，容器可直接通过**容器名**相互访问，无需记住 IP，适配微服务通信场景。

**使用示例**：

```Plain Text
# 创建自定义网络
docker network create my-net
# 启动容器并加入自定义网络
docker run -d --name nginx --network=my-net nginx
docker run -d --name mysql --network=my-net mysql
```

## 6\.3 网络通信总结

- 同默认 bridge 网络：容器 IP 互通，容器名不通

- 同自定义 bridge 网络：IP、容器名均可互通（核心优势）

- host 网络：容器与宿主机完全互通

- none 网络：完全断网隔离

# 七、Docker Compose

Docker Compose 是 Docker **多容器编排工具**，用于解决多服务项目（如前端\+后端\+数据库\+缓存）批量部署、启停、管理的问题，实现**一键启动、一键停止**整套服务。

## 7\.1 核心作用

- 统一管理多个关联容器，无需逐个启动、配置容器

- 通过 YAML 文件标准化定义所有服务的端口、挂载、网络、依赖

- 支持服务依赖排序启动，解决启动顺序冲突（先数据库、后后端服务）

## 7\.2 核心概念

- **服务（service）**：单个容器对应的应用服务（如 mysql、nginx、java 服务）

- **项目（project）**：由多个服务组成的整套项目，默认以当前目录名为项目名

## 7\.3 docker\-compose\.yml 核心语法

默认配置文件名必须为 `docker\-compose\.yml`，YAML 语法严格缩进（2个空格）。

- `version`：Compose 版本声明

- `services`：定义所有服务（核心模块）

- `image`：服务依赖的镜像

- `build`：指定 Dockerfile 路径，自动构建镜像

- `ports`：端口映射（宿主机:容器）

- `volumes`：数据卷挂载

- `networks`：指定服务所属网络

- `depends\_on`：服务依赖，指定启动顺序

- `environment`：环境变量配置

- `restart`：重启策略

## 7\.4 完整 Compose 示例（Java\+MySQL\+Nginx）

```Plain Text
version: '3.8'
# 定义服务
services:
  # mysql服务
  mysql:
    image: mysql:5.7
    container_name: mysql
    ports:
      - "3306:3306"
    volumes:
      - mysql-data:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: 123456
    restart: always
    networks:
      - app-net

  # java后端服务
  app:
    build: ./
    container_name: app-service
    ports:
      - "8080:8080"
    depends_on:
      - mysql
    restart: always
    networks:
      - app-net

  # nginx前端服务
  nginx:
    image: nginx:latest
    container_name: nginx-web
    ports:
      - "80:80"
    volumes:
      - ./nginx/html:/usr/share/nginx/html
    restart: always
    networks:
      - app-net

# 定义数据卷
volumes:
  mysql-data:

# 定义自定义网络
networks:
  app-net:
    driver: bridge
```

## 7\.5 Compose 常用核心命令

所有命令需在 `docker\-compose\.yml` 所在目录执行

- `docker\-compose up \-d`：后台启动所有服务（首次自动创建容器、网络、数据卷）

- `docker\-compose down`：停止并删除所有服务容器、网络（保留数据卷）

- `docker\-compose ps`：查看当前项目所有服务状态

- `docker\-compose logs \-f`：实时查看所有服务日志

- `docker\-compose restart`：重启所有服务

- `docker\-compose build`：重新构建自定义镜像

- `docker\-compose start/stop`：启动/停止服务（不删除容器）

## 7\.6 核心注意事项

- `depends\_on` 仅控制**启动顺序**，不等待依赖服务完全就绪（如MySQL启动未初始化完成），需项目自身做重试兼容

- 修改 yml 配置文件后，需重新执行 `docker\-compose up \-d` 生效

- 生产环境优先使用自定义网络，保证服务间稳定通信

# 八、Docker 生产最佳实践 \&amp; 优化技巧

## 8\.1 镜像优化原则（减小镜像体积、提升安全性）

镜像体积越小，部署速度越快、漏洞风险越低，生产环境必须遵循轻量化构建原则。

- **选用极简基础镜像**：优先使用 `alpine`、`slim` 轻量化镜像，替代完整版系统镜像，体积可缩减80%以上，例如 `nginx:alpine`、`java:8\-alpine`。

- **合并 RUN 指令**：多条安装、配置命令合并为一个 RUN，减少镜像分层，避免冗余层堆积。

- **清理构建缓存**：安装依赖后立即清理 yum/apt 缓存、临时文件，防止缓存打包进镜像。

- **禁止打包敏感文件**：通过 `\.dockerignore` 忽略配置密钥、日志、本地缓存、Git文件等无关内容。

- **分层构建（多阶段构建）**：分离构建环境和运行环境，仅打包运行所需文件，彻底剥离编译工具、依赖缓存。

## 8\.2 \.dockerignore 文件配置（必备）

作用：构建镜像时忽略指定文件/目录，减小镜像体积、规避敏感信息泄露，放置在 Dockerfile 同级目录。

**通用完整配置**：

```Plain Text
# 忽略Git文件
.git
.gitignore

# 忽略日志、临时文件
*.log
tmp/
temp/

# 忽略依赖缓存
node_modules/
maven/repository/

# 忽略本地配置、密钥
.env
*.properties
*.yml.local

# 忽略IDE配置
.idea/
.vscode/
*.iml

# 忽略系统文件
.DS_Store
```

## 8\.3 多阶段构建实战（Java项目极简示例）

解决传统构建镜像体积过大问题，第一阶段编译打包，第二阶段仅保留运行环境，镜像体积大幅压缩。

```Plain Text
# 第一阶段：构建阶段（maven镜像用于编译打包）
FROM maven:3.8-openjdk-8 AS build
WORKDIR /build
COPY ./pom.xml .
COPY ./src ./src
# 编译打包，跳过测试
RUN mvn clean package -Dmaven.test.skip=true

# 第二阶段：运行阶段（仅保留jre运行环境）
FROM java:8-alpine
WORKDIR /app
# 从构建阶段拷贝打包好的jar包
COPY --from=build /build/target/*.jar app.jar
EXPOSE 8080
CMD ["java","-jar","app.jar"]
```

## 8\.4 容器生产运行规范

- **禁止使用 root 运行容器**：自定义普通用户启动服务，降低容器提权风险，提升安全性。

- **限制容器资源**：启动容器配置 CPU、内存限制，防止单容器抢占宿主机全部资源。
  示例：`docker run \-\-memory=512m \-\-cpus=0\.5 镜像名`

- **统一重启策略**：生产服务统一配置 `\-\-restart=always`，保证服务异常退出后自动恢复。

- **禁止裸端口运行**：所有服务通过端口映射对外暴露，不使用 host 网络（特殊业务除外），保障网络隔离性。

- **数据全部持久化**：数据库、缓存、业务数据必须挂载数据卷，严禁数据存于容器内部。

# 九、Docker 常见报错与解决方案（实战避坑）

## 9\.1 镜像/容器常见问题

- **问题1：镜像删除失败，提示容器占用**
  **原因**：该镜像存在已停止/运行中的容器
  **解决**：先删除对应容器 `docker rm \-f 容器ID`，再删除镜像 `docker rmi 镜像ID`

- **问题2：虚悬镜像（\&lt;none\&gt;:\&lt;none\&gt;）占用空间**
  **解决**：一键清理无用镜像、网络、数据卷 `docker system prune \-a`

- **问题3：docker run 端口占用报错**
  **原因**：宿主机端口已被其他程序/容器占用
  **解决**：更换映射端口，或停止占用端口的容器/服务

## 9\.2 Dockerfile 构建报错

- **问题1：构建提示 context 过大**
  **原因**：构建目录包含大量无关文件，未配置 \.dockerignore
  **解决**：添加 \.dockerignore 文件，忽略无用目录和文件

- **问题2：CMD/ENTRYPOINT 启动容器立即退出**
  **原因**：容器前台进程执行完毕，无常驻进程
  **解决**：确保启动命令为前台常驻进程，避免执行完立即结束

## 9\.3 网络通信报错

- **问题1：同容器无法通过容器名通信**
  **原因**：使用默认 bridge 网络，无域名解析功能
  **解决**：将容器加入**自定义 bridge 网络**

- **问题2：容器无法访问外网**
  **原因**：DNS配置异常、网桥故障
  **解决**：重启Docker服务 `systemctl restart docker`，或手动配置容器DNS

