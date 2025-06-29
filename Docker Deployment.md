### 前言

本节将简单介绍如何在 [Docker](./Docker.md) 中部署特定的容器。

### 版本环境

- VMware Workstation 16 Pro（16.2.3 build-19376536）
- CentOS 7.9（CentOS-7-x86_64-DVD-2009）
- Docker CE 26.0.0
- MySQL 5.7.0 | MySQL 8.3.0
- Navicat 16.3.8

### Docker

默认情况下 docker 不会跟随系统一起启动。

如果需要 docker 服务自启，则需手动将 docker 服务添加到系统的自启动服务列表中：

```shell
systemctl enable docker.service
```

如果希望取消自启动，则使用以下命令：

```shell
systemctl disable docker.service
```

另外是关于 docker 端口映射的问题，CentOS 7.9 下不需要为 docker 服务配置服务放行。即外部主机总是可以通过宿主机的任意端口实现容器的访问，而不需要使用命令将特定端口添加到放行列表中。

但对于其他的 Linux 系统来说，是否需要为 docker 应用放行取决于防火墙策略，请自行注意。

### MySQL

MySQL 服务一般部署 Linux 系统中，但部署过程较为繁杂，且一不小心可能就会出现错误。通过借助 docker 容器技术，可以在几分钟内完成 MySQL 服务的部署，且难度很小。

#### 1. 获取镜像

软件的镜像就如同系统镜像一般，只要将其展开即可使用。

MySQL 服务的镜像被命名为 mysql，使用 `docker pull` 命令可获取最新版本的 MySQL 镜像：

```shell
docker pull mysql
```

如果希望获取特定版本的 MySQL 镜像，可以使用 tag 标签：

```bash
# 获取 MySQL 5.7 版本镜像
docker pull mysql:5.7

# 获取 MySQL 最新版本镜像
docker pull mysql:latest
```

附上查看当前 docker 内所有镜像 images 的命令：

```shell
docker images 
```

#### 2. 启动容器

**可以将 docker 想象成于虚拟机，软件镜像 image 则相当于系统镜像，容器 container 即为可运行在 docker 中的虚拟机系统。虚拟机系统所对应的开机、关机状态，即对应着软件镜像的运行、停止状态。**

这意味着仅仅获取软件镜像是不够的，必须让这些镜像成为可以运行的容器才行，而容器必须经过创建。

使用 `docker create` 命令创建容器（类比虚拟机操作，这相当于使用系统镜像安装虚拟机系统）：

```bash
docker create --name=my_database --restart=always --privileged=true -e MYSQL_ROOT_PASSWORD=0217 -e TZ=Asia/Shanghai -v ~/mysql/conf.d:/etc/mysql/conf.d -p 3306:3306 mysql:latest
```

参数说明：

- `--name`：容器名称，不能与其他容器重名；
- `--restart=always`：表示在 docker 重启后，该容器也会自动启动；
- `--privileged=true`：默认情况下 root 只是普通用户，加上该参数后 root 将拥有真正的 root 权限；
- `-e`：用于设定容器的环境变量，这里用于设定 MySQL 中 root 用户的密码与容器时区；
- `-v`：挂载本机目录到容器目录中，本机目录如果不存在则会自动创建，容器目录必须为绝对路径；
- `-p`：本机端口与容器端口的映射，本机端口不可重复映射（容器具有不同 IP 因此容器端口可以重复）。

该命令将创建容器并返回容器的 `ID` 值，类似于：

```
a03785914297cda90f6826ddf72dc1d48907480ee94899b95b211dbae4363410
```

创建完毕后，使用 `docker start` 命令加容器 `ID` 即可启动容器（类比虚拟机操作，这相当于启动虚拟机系统）：

```bash
docker start a03785
```

或者使用容器 `name`：

```bash
docker start my_database
```

实际开发中，较常使用的是 `docker run` 命令，该命令用于创建并启动容器：

```shell
docker run -d --name=my_database --restart=always --privileged=true -e MYSQL_ROOT_PASSWORD=0217 -e TZ=Asia/Shanghai -v ~/mysql/conf.d:/etc/mysql/conf.d -p 3306:3306 mysql:latest
```

参数说明：

- `-d`：表示允许容器后台运行。

**重点注意，推荐将 `-d` 参数置于 `docker run` 命令之后，将指定的容器镜像 `mysql:tag` 作为命令结尾，其余配置参数则置于它们之间，这样可以避免 `docker run` 命令出现不必要的执行错误。**

如果希望运行不同版本的 MySQL 例如 MySQL 5.7 和 MySQL 8.0 容器，指定镜像的 tag 即可：

```shell
# 宿主机端口为 3306，容器名称 my_database_a，数据库版本 5.7
docker create --name=my_database --restart=always --privileged=true -e MYSQL_ROOT_PASSWORD=0217 -e TZ=Asia/Shanghai -v ~/mysql/conf.d:/etc/mysql/conf.d -p 3306:3306 mysql:5.7

# 宿主机端口为 3307，容器名称 my_database_b，数据库版本 latest
docker create --name=my_database --restart=always --privileged=true -e MYSQL_ROOT_PASSWORD=0217 -e TZ=Asia/Shanghai -v ~/mysql/conf.d:/etc/mysql/conf.d -p 3306:3306 mysql:8.0
```

如需多次搭建实验性的 MySQL 环境，可以使用 `.sh` 文件来帮助快速重置容器：

```shell
#!/bin/bash

# 检查参数数量是否正确
if [ "$#" -ne 1 ]; then
    echo "Usage: $0 <container_name_or_id>"
    exit 1
fi

container="$1"

# 停止并删除容器
docker stop "$container"
docker rm "$container"
docker create --name=my_database --restart=always --privileged=true -e MYSQL_ROOT_PASSWORD=0217 -e TZ=Asia/Shanghai -v ~/mysql/conf.d:/etc/mysql/conf.d -p 3306:3306 mysql:latest
```

为 `.sh` 文件赋予执行权限：

```bash
chmod +x reset_mysql.sh
```

重置 MySQL 容器只需要执行 `.sh` 文件时提供容器 `ID` 即可：

```bash
./reset_mysql.sh 97db19f2b27
```

这样 Linux 将按顺序执行 `.sh` 文件内的命令，自动完成包括停止容器、删除容器和创建并启动新容器等操作。

附上查看当前 docker 内所有容器 container 的命令：

```bash
# 查看所有容器
docker ps -a

# 查看当前正在运行中的容器
docker ps
```

#### 3. 目录挂载

目录挂载（Mounting）是指将一个文件系统（通常是存储在磁盘或其他设备上的文件系统）连接到计算机的文件系统目录树中的过程。这个过程使得文件系统中的文件和目录可以在计算机中被访问和操作。

在 Linux 和类 Unix 系统中，目录挂载是通过 `mount` 命令实现的。具体来说，挂载操作会将一个文件系统（如硬盘分区、USB 存储设备、网络存储等）的内容连接到文件系统树中的一个指定位置（挂载点）上。一旦挂载完成，文件系统中的内容就可以通过挂载点来访问。

例如将 USB 存储设备挂载到 Windows 系统， USB 存储设备需连接至计算机，Windows 系统会自动将 USB 存储设备上的文件系统的内容挂载到 Windows 系统中的一个虚拟磁盘上，并为该磁盘分配一个盘符，例如 H 盘。

用户即可通过访问Windows 系统上的 H 盘，来间接访问 USB 存储设备上的文件内容。

Linux 系统下，假如接入计算机的 USB 存储设备的设备路径为 `/dev/sdb1`，一般需要手动挂载该设备：

```bash
mount /dev/sdb1 /mnt
```

其中 `/mnt` 是一个常用的挂载点，通常被用于临时挂载额外的设备或文件系统。这样通过访问 `/mnt` 目录，即可间接访问 USB 存储设备上的文件内容。

实际上，创建容器时的目录挂载也是同样的道理。可以认为宿主机上的目录是“USB 存储设备”。容器启动时，会自动将宿主机上的“USB 存储设备”挂载至容器内的挂载点，以实现内容访问，且挂载两端的文件系统是实时同步的。

MySQL 在启动时会加载多个配置文件，并按照一定的顺序进行处理，一般 MySQL 会优先加载 `/etc/my.cnf` 默认的配置文件，之后再加载 `/etc/mysql/conf.d` 目录下的所有 `.cnf` 文件。

这意味着，如果 `/etc/mysql/conf.d` 目录内拥有其他的 `.cnf` 配置文件，那么 MySQL 也会加载并使用它们。上述命令中 `-v` 参数会将本机的 `~/mysql/conf.d` 目录挂载到容器内的  `/etc/mysql/conf.d` 目录下。

那么本机只需要在容器启动前，将自定义的配置文件，例如 `my.cnf` 置于本机的 `~/mysql/conf.d` 目录内，容器启动之后，这些配置文件会自动映射到容器内的 `/etc/mysql/conf.d` 目录中并交由 MySQL 加载。

这其实就是 `-v` 目录挂载参数的主要作用之一。docker 的目录挂载可以将容器的配置或数据目录等映射至本机目录，映射一般是实时同步的，这样方便用户对容器内的数据进行直接的操作，例如修改配置或备份数据等。

但需要注意，实时同步不意味着实时应用，例如配置文件只有在 MySQL 启动时才会读取。MySQL 处于运行状态时，即便修改配置文件，配置也不会实时生效，必须重启 MySQL 容器后新的配置才会生效。

#### 4. 容器编排

容器编排（docker compose）用于定义和运行多容器 docker 应用程序。

CentOS 系统下完成 docker-ce 的安装后，一般 docker-compose-plugin 程序也会同时被安装：

```
~]# yum list installed | grep docker-compose
docker-compose-plugin.x86_64    2.25.0-1.el7                   @docker-ce-stable
```

如果系统中存在 docker-compose-plugin 程序，则意味着容器编排可用。

容器编排依赖 `.yml` 编排文件，一种约定俗成的命名是 `docker-compose.yml` 文件，但这并不意味着不能以其他方式命名。

所谓编排文件，可以简单理解为将创建容器的命令转换成 yaml 格式后的文件，例如：

```bash
docker create --name=my_database --restart=always --privileged=true -e MYSQL_ROOT_PASSWORD=0217 -e TZ=Asia/Shanghai -v ~/mysql/conf.d:/etc/mysql/conf.d -p 3306:3306 mysql:latest
```

其对应的 `compose.yaml` 文件为：

```yaml
services:
  mysql:
    image: mysql:latest
    container_name: my_database
    restart: always
    privileged: true
    environment:
      MYSQL_ROOT_PASSWORD: '0217'
      TZ: Asia/Shanghai
    volumes:
      - ~/mysql/conf.d:/etc/mysql/conf.d
    ports:
      - 3306:3306
```

编排文件创建完毕后，在编排文件所在目录下执行命令：

```bash
docker compose up -d
```

那么 docker 就会使用 `compose.yaml` 文件内的参数来创建指定的容器。

容器编排实际是大规模部署和管理容器化应用程序时的必要性的产物，其作用为：

1. **自动化部署和管理**：容器编排平台（如 Kubernetes、Docker Swarm 等）能够自动化应用程序的部署、扩展、升级和维护。这意味着可以更快速、可靠地交付和管理应用程序，而不需要手动操作每个容器实例；
2. **弹性伸缩**：容器编排允许根据负载需求动态伸缩应用程序实例的数量。这意味着可以根据流量量或其他指标增加或减少容器实例，以确保您的应用程序始终具有适当的资源；
3. **服务发现和负载均衡**：容器编排平台提供了内置的服务发现和负载均衡机制，使容器能够相互发现并自动路由请求到可用的实例上。这样可以轻松地管理多个容器之间的通信和流量分发；
4. **容错性和高可用性**：容器编排可以配置容器的健康检查和自动恢复机制，以确保容器实例的健康状态。如果某个容器失败，编排系统可以自动替换它，从而提高应用程序的容错性和高可用性；
5. **资源管理和优化**：容器编排允许有效地管理和优化容器使用的资源，如 CPU、内存和存储。可以为每个容器定义资源限制和请求，以确保资源的公平分配和最大化利用。

从容器编排的作用上看，可以知道其实它并不简单。实际上，编排文件也不仅仅是将创建容器命令直接转换为 yaml 文件那么肤浅，例如容器编排还允许为容器自定义 IP 范围：

```yaml
services:
  mysql:
    image: mysql:latest
    container_name: my_database
    restart: always
    privileged: true
    environment:
      MYSQL_ROOT_PASSWORD: '0217'
      TZ: Asia/Shanghai
    volumes:
      - ~/mysql/conf.d:/etc/mysql/conf.d
    ports:
      - 3306:3306
    networks:
      - my_network

networks:
  my_network:
    driver: bridge
    ipam:
      driver: default
      config:
        - subnet: 192.168.11.0/24
          gateway: 192.168.11.254
```

这样通过 `compose.yaml` 文件创建的 `my_database` 容器所使用的 IP 地址将会是 `192.168.11.1`。如果编排文件中存在其他的容器，这些容器同样会使用 `192.168.11.0/24` 段内的 IP 作为容器的 IP 地址。

```
~]# docker inspect --format='{{.NetworkSettings.Networks.root_my_network.IPAddress}}' my_database
192.168.11.1

~]# docker inspect --format='{{.NetworkSettings.Networks.root_my_network.Gateway}}' my_database
192.168.11.254
```

碍于篇幅有限，这里只要求对容器编排有基本的了解即可。

#### 5. 进入容器

容器创建并启动后，通过以下命令可以进入容器的 bash 终端：

```shell
docker exec -it my_database bash
```

#### 6. 登录服务

进入容器后使用以下命令登录 MySQL 数据库：

```bash
mysql -u root -p
```

```
~]# mysql -u root -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.3.0 MySQL Community Server - GPL

Copyright (c) 2000, 2024, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```

#### 7. 中文乱码

一般未经配置的 MySQL 都会存在中文乱码的问题。以下数据库命令可以查看数据库编码的配置情况：

```sql
SHOW VARIABLES LIKE 'character%';
```

MySQL 8.3.0 未配置编码前的查询结果为：

```
mysql> SHOW VARIABLES LIKE 'character%';
+--------------------------+--------------------------------+
| Variable_name            | Value                          |
+--------------------------+--------------------------------+
| character_set_client     | latin1                         |
| character_set_connection | latin1                         |
| character_set_database   | utf8mb4                        |
| character_set_filesystem | binary                         |
| character_set_results    | latin1                         |
| character_set_server     | utf8mb4                        |
| character_set_system     | utf8mb3                        |
| character_sets_dir       | /usr/share/mysql-8.3/charsets/ |
+--------------------------+--------------------------------+
8 rows in set (0.01 sec)
```

建议使用配置文件的方式，来解决中文乱码的问题。

MySQL 8.0 版本以上，可新建以下 `my.cnf` 配置文件：

```properties
[client]
default-character-set=utf8mb4

[mysqld]
character-set-server=utf8mb4
```

MySQL 5.7 版本以下，可新建以下 `my.cnf` 配置文件：

```properties
[client]
default-character-set=utf8

[mysqld]
character-set-server=utf8
```

由于创建 `my_database` 容器的时候，已经进行过目录挂载了，因此可以直接在本机的 `~/mysql/conf.d` 目录下创建 `my.cnf` 配置文件。重启容器后配置文件会自动同步至容器内的 `/etc/mysql/conf.d` 目录中并被加载。

MySQL 8.3.0 已配置编码后的查询结果为：

```
mysql> SHOW VARIABLES LIKE 'character%';
+--------------------------+--------------------------------+
| Variable_name            | Value                          |
+--------------------------+--------------------------------+
| character_set_client     | utf8mb4                        |
| character_set_connection | utf8mb4                        |
| character_set_database   | utf8mb4                        |
| character_set_filesystem | binary                         |
| character_set_results    | utf8mb4                        |
| character_set_server     | utf8mb4                        |
| character_set_system     | utf8mb3                        |
| character_sets_dir       | /usr/share/mysql-8.3/charsets/ |
+--------------------------+--------------------------------+
8 rows in set (0.01 sec)
```

对于 8.0 和 5.7 版本来说，在编码受系统支持的前提下，可以共用同一个 my.cnf 配置文件。

谨记 MySQL 在启动时会加载多个配置文件，并按照一定的顺序进行处理。其中 `/etc/mysql/conf.d` 目录下的配置总是会被 MySQL 顺序加载，以对数据库进行自定义的初始化操作。

#### 8. 编码详情

配置文件中，`[client]` 内的  `default-character-set` 用于配置以下参数的值：

- `character_set_client`、`character_set_connection`、`character_set_results`

这些参数都属于 SESSION 级别的参数，即客户端可以自行控制这些参数的配置。如果在 `my.cnf` 配置中更改了这些参数配置，那么进入 MySQL 容器并登录 MySQL 后，当前会话就会使用这些参数配置。

另外 `[mysqld]` 内的 `character-set-server` 用于配置以下参数的值：

- `character_set_database`、`character_set_server`

这些参数则属于服务端配置，任何客户端都需要遵循服务端的配置。

其次，参数 `character_set_system` 反映了操作系统的默认字符集，而参数 `character_set_filesystem` 则反映在磁盘上存储和读取数据文件时的字符编码，它们都不属于配置文件的一部分。

举个例子，假如有以下 `my.cnf` 配置：

```properties
[client]
default-character-set=latin2

[mysqld]
character-set-server=latin1
```

重启 MySQL 容器再次进入容器并登录数据库后，查询编码集会得到：

```
mysql> SHOW VARIABLES LIKE 'character%';
+--------------------------+--------------------------------+
| Variable_name            | Value                          |
+--------------------------+--------------------------------+
| character_set_client     | latin2                         |
| character_set_connection | latin2                         |
| character_set_database   | latin1                         |
| character_set_filesystem | binary                         |
| character_set_results    | latin2                         |
| character_set_server     | latin1                         |
| character_set_system     | utf8mb3                        |
| character_sets_dir       | /usr/share/mysql-8.3/charsets/ |
+--------------------------+--------------------------------+
8 rows in set (0.00 sec)
```

但如果是通过 Windows 系统上的 Navicat 数据库工具连接 MySQL 数据库，那编码集会变为：

```
mysql> SHOW VARIABLES LIKE 'character%';
+--------------------------+--------------------------------+
| Variable_name            | Value                          |
+--------------------------+--------------------------------+
| character_set_client     | utf8mb4                        |
| character_set_connection | utf8mb4                        |
| character_set_database   | latin1                         |
| character_set_filesystem | binary                         |
| character_set_results    | utf8mb4                        |
| character_set_server     | latin1                         |
| character_set_system     | utf8mb3                        |
| character_sets_dir       | /usr/share/mysql-8.3/charsets/ |
+--------------------------+--------------------------------+
8 rows in set (0.00 sec)
```

因为 Navicat 在连接数据库时，可以决定会话所使用的字符集（编码集）：

<div align="center"><img src="images/Docker%20Deployment.images/Snipaste_2024-04-12_19-04-45.png" alt="Snipaste_2024-04-12_19-04-45" style="width:65%;" /></div>

Navicat 16.3.8 中“自动”表示“客户端字符集”使用的字符集为 `utf8mb4`。

参数 `character_set_database` 和 `character_set_server` 等由 MySQL 服务端控制，其中前者是数据库创建时默认使用的字符集，后者则是存储数据时数据库所使用的字符集，这里两个参数是无法通过客户端改变的。

不过实际存在一种临时更改的方式：

```mysql
SET character_set_database=utf8mb4;
SET character_set_server=utf8mb4;
```

通过这种方式修改的参数值，会在数据库重连后被重置。

关于 `character_set_client`、`character_set_connection`、`character_set_results`、 `character_set_database` 和 `character_set_server` 等参数的实际含义可以不需要特别了解，因为避免出现编码出现问题最好的解决方式，是在服务端的主机上添加以下配置：

```
[client]
default-character-set=utf8mb4

[mysqld]
character-set-server=utf8mb4
```

这样至少能够确保 `character_set_database` 和 `character_set_server` 两个参数能够兼容中文字符。其次就只需要注意客户端使用的字符集即可，一般确保其为 utf8mb4 编码即可。

**最后来梳理一下数据库字符集（编码）和数据表字符集之间的关系，其中：**

- **参数 `character_set_database` 决定了创建数据库时默认使用的字符集；**
- **数据库使用的字符集决定了在该数据库中创建数据表时默认使用的字符集。**

不过无论是创建数据库还是创建数据表，它们都支持在创建时添加自定义字符集：

```mysql
CREATE DATABASE test_charset CHARSET=latin2;
```

```mysql
CREATE TABLE test_charset (
	id INT NOT NULL
) CHARSET=latin2;
```

以上语句中，即便参数 `character_set_database` 指定使用 utf8mb4 编码，但该数据库 test_charset 仍旧会使用创建数据库时指定的 latin2 编码；同理，数据表 test_cahrset 也会使用创建数据表时指定的 latin2 编码。

#### 9. 排序字符

完成字符集配置之后，通常会改变默认的排序字符集：

```
mysql> SHOW VARIABLES LIKE 'collation%';
+----------------------+--------------------+
| Variable_name        | Value              |
+----------------------+--------------------+
| collation_connection | utf8mb4_0900_ai_ci |
| collation_database   | utf8mb4_0900_ai_ci |
| collation_server     | utf8mb4_0900_ai_ci |
+----------------------+--------------------+
3 rows in set (0.01 sec)
```

其中：

- `character_set_connection` 对应 `collation_connection`；
- `character_set_database` 对应 `collation_database`；
- `character_set_server` 对应 `collation_server`。

这里只需要记住常用的排序字符集有 `utf8mb4_unicode_ci`、`utf8mb4_general_ci` 和 `utf8mb4_0900_ai_ci` 即可，其中 `utf8mb4_0900_ai_ci` 是 MySQL 8.0 之后默认使用的排序字符集。

本篇使用的是 MySQL 8.3.0 版本，在字符集配置为 `utf8mb4` 时，默认使用 `utf8mb4_0900_ai_ci` 排序字符集。
