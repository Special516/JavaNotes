## 使用 Maven 插件制作 Docker 镜像

### 1、Docker

#### 1.1 Docker 简介

Docker 是一个开源的应用容器引擎，开发者可以打包自己的应用到容器里面，然后迁移到其他机器的docker应用中，可以实现快速部署。如果出现的故障，可以通过镜像，快速恢复服务。

#### 1.2 安装 Docker

在 `Windows` 和 `Mac` 系统下可以去 [Docker官网](https://www.docker.com/products/docker-desktop) 下载 `Docker Desktop` 软件直接进行安装。

在 `Linux` 上安装 Docker：

> 安装 Docker 要求 CentOS 系统的内核版本高于 3.10

```shell
# 以下操作是在 root 用户下进行，非 root 用户可能出现权限不足的提示
# 备份原镜像源文件
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
# 设置 yum 的阿里镜像源
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
# 生成缓存
yum makecache

# 配置 docker-ce 的 yum 源
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

# 安装需要的依赖包
yum install -y yum-utils device-mapper-persistent-data lvm2

# 安装docker-ce
yum -y install docker-ce

# 设置 Docker 镜像加速，使用官方的镜像地址拉取镜像非常慢
vi /etc/docker/daemon.json
# 将下面的配置复制进 daemon.json 文件中
{"registry-mirrors": ["https://dockerhub.azk8s.cn"]}

# 启动 Docker
systemctl start docker

# 设置 Docker 开机启动
systemctl enable docker
```

#### 1.3 Docker 基本命令

|                     命令                      |          说明          |               例子                |
| :-------------------------------------------: | :--------------------: | :-------------------------------: |
|                 docker images                 |   查看docker已有镜像   |           docker images           |
|            docker search \<image>             |     查询指定的镜像     |       docker search tomcat        |
|       docker pull [OPTIONS] NAME[:TAG\|       |     拉取指定的镜像     |        docker pull tomcat         |
| docker run [OPTIONS] IMAGE [COMMAND] [ARG...] |    通过镜像创建容器    | docker run -p 8080:8080 -d tomcat |
|                 docker ps -a                  |    查看已启动的容器    |           docker ps -a            |
|                  docker stop                  |  停止一个运行中的容器  |        docker stop tomcat         |
|                 docker start                  | 启动一个已经停止的容器 |     docker start 3e6be8b7bd52     |
|                docker restart                 |      重新启动容器      |       docker restart tomcat       |
|                   docker rm                   |   删除已经停止的容器   |      docker rm 3e6be8b7bd52       |
|                  docker rmi                   |     删除指定的镜像     |         docker rmi tomcat         |

### 2、 Dockerfile

Dockerfile 是一个用来构建镜像的文本文件，文本内容包含了一条条构建镜像所需的指令和说明。

|  指令   |                             说明                             |
| :-----: | :----------------------------------------------------------: |
|  FROM   | 每个Dockerfile的开始必须添加，后面跟具体的镜像，表示基于哪个镜像进行构建 |
|  COPY   |  复制指令，从上下文目录中复制文件或者目录到容器里指定路径。  |
|   ADD   | ADD 指令和 COPY 的使用格式一致（同样需求下，官方推荐使用 COPY）功能也类似，当源文件为压缩格式的文件时（tar,gzip, bzip2）会自动解压 |
|   RUN   |                 用于执行后面跟着的命令行命令                 |
|   CMD   |   类似于 RUN 指令，用于运行程序，CMD在 `docker run` 时运行   |
| EXPOSE  |                       仅仅只是声明端口                       |
| WORKDIR | 指定工作目录。用 WORKDIR 指定的工作目录，会在构建镜像的每一层中都存在 |

```dockerfile
FROM tomcat:9-jdk8
WORKDIR /usr/local/tomcat/
ADD ./target/tazhzl.war webapps/
EXPOSE 8080
CMD ["bin/catalina.sh","run"]
```

### 3、docker-compose

#### 3.1 docker-compose简介

Docker Compose是 docker 提供的一个命令行工具，实现对Docker容器的快速编排，用来定义和运行由多个容器组成的应用。使用 compose，可以通过 YAML 文件声明式的定义应用程序的各个服务，并由单个命令完成应用的创建和启动。

#### 3.2 安装 docker-compose

在 `docker-compose` 的GitHub上下载 [docker-composr](https://github.com/docker/compose/releases) 选择对应的版本，将下载好的文件上传到服务器 `/usr/local/bin` 目录下并将文件改名为 `docker-compose` 再赋予可执行的权限 `chmod +x docker-compose` 。至此 `docker-compose` 安装完成。

#### 3.3 docker-compose命令

|        命令         |                说明                |
| :-----------------: | :--------------------------------: |
|  docker-compose up  | 根据docker-compose.yml文件启动容器 |
| docker-compose down | 停止docker-compose.yml文件中的容器 |

#### 3.4 docker-compose.yml

```yaml
version: '3'
services:
  registry-web:
    image: hyper/docker-registry-web
    container_name: registry_web
    restart: always
    ports:
      - 9090:8080
    volumes:
      - /opt/registry/registry-web/conf:/conf:ro
      - /opt/registry/registry-web/db:/data
    networks:
      - registry-net
    depends_on:
       - registry

  registry:
    image: registry
    container_name: registry
    restart: always
    ports:
      - 5000:5000
    volumes:
      - /opt/registry/registry:/var/lib/registry
    networks:
      - registry-net

networks:
  registry-net:
```

### 4、安装 Docker 私服 harbor（哈勃）

访问 [harbor](https://goharbor.io/) 官网下载最新的 harbor 版本。下载地址 [harbor](https://github.com/goharbor/harbor/releases) 

![](https://note.youdao.com/yws/public/resource/aaa8dcbdccaf115aa2e63f65f0eceb3d/xmlnote/60E3DD0C976849E48DDBA0DE32560B38/3049)

将下载好的文件上传至服务器进行解压，解压后会得到以下文件：

![](https://note.youdao.com/yws/public/resource/aaa8dcbdccaf115aa2e63f65f0eceb3d/xmlnote/21410D3F7D43423C97B68AD0FA8EABC8/3051)

修改 `harbor.yml` 文件：

```yaml
hostname: 172.16.59.130

http:
  # 配置 harbor 启动后的访问端口，默认 80
  port: 8090

# 此处配置的是 http 访问方式，需要将以下关于 https 的配置注释掉
#https:
#  # https port for harbor, default is 443
#  port: 443
#  # The path of cert and key files for nginx
#  certificate: /your/certificate/path
#  private_key: /your/private/key/path

harbor_admin_password: Harbor1234

# Harbor DB configuration
database:
  password: root123
  max_idle_conns: 50
  max_open_conns: 100

data_volume: /data

clair:
  updaters_interval: 12

jobservice:
  max_job_workers: 10

notification:
  # Maximum retry count for webhook job
  webhook_job_max_retry: 10

chart:
  absolute_url: disabled

log:
  level: info
  local:
    rotate_count: 50
    rotate_size: 200M
    location: /var/log/harbor

_version: 1.10.0

proxy:
  http_proxy:
  https_proxy:
  # no_proxy endpoints will appended to 127.0.0.1,localhost,.local,.internal,log,db,redis,nginx,core,portal,postgresql,jobservice,registry,registryctl,clair,chartmuseum,notary-server
  no_proxy:
  components:
    - core
    - jobservice
    - clair

```

配置完成后运行 `./install.sh` 命令即可开始安装 harbor。安装完成后会在该目录下生成一个 `docker-compose.yml` 文件，并且会自动运行，启动 harbor 运行所需要的 docker 容器。

使用 `docker ps` 命令可以查看容器的启动情况。

在浏览器中输入 `ip:端口` 即可访问 harbor 私服。

![](https://note.youdao.com/yws/public/resource/aaa8dcbdccaf115aa2e63f65f0eceb3d/xmlnote/82CB9745B60647EA883DDC527DA78816/3053)

私服安装完成，但此时还不能将镜像推送至私服。由于docker的镜像拉取、推送全部采用 `https` 传输协议，使用 `http` 协议会被认为是不安全的，从而禁止访问。

```shell
# 修改 docker.service
vim /lib/systemd/system/docker.service
# 在 ExecStart 后添加 -H tcp://0.0.0.0:2375 --insecure-registry=172.16.59.130:8090
ExecStart=/usr/bin/dockerd -H fd:// -H tcp://0.0.0.0:2375 --insecure-registry=172.16.59.130:8090

#-H tcp://0.0.0.0:2375 开启远程控制访问
#--insecure-registry=172.16.59.130:8090 配置私服地址，允许使用 http 协议访问
```

配置完成后重启docker服务

> 重启服务后可能会出现harbor服务无法访问的情况，可以进入harbor安装目录重新启动容器。
>
> docker-compose down				docker-compose up -d

### 5、maven 插件

使用 maven 命令直接打包应用为 docker 镜像需要在 `pom.xml` 文件中安装 `docker-maven-plugn` 

具体使用参阅官方文档 [io.fabric8/docker-maven-plugin](http://dmp.fabric8.io/#global-configuration) 

```xml
<plugin>
    <!--引入 docker-maven 插件-->
    <groupId>io.fabric8</groupId>
    <artifactId>docker-maven-plugin</artifactId>
    <version>0.33.0</version>
    <!--全局配置-->
    <configuration>
        <!--指定基础镜像拉取规则-->
        <imagePullPolicy>IfNotPresent</imagePullPolicy>
        <!--docker主机地址,用于完成docker各项功能!-->
        <dockerHost>tcp://172.16.59.130:2375</dockerHost>
        <!--registry地址,用于推送,拉取镜像-->
        <pushRegistry>172.16.59.130:8090/tazhzl</pushRegistry>
        <!--认证配置,用于私有registry认证-->
        <authConfig>
            <push>
                <username>admin</username>
                <password>Harbor1234</password>
            </push>
        </authConfig>
        <images>
            <image>
                <!--<name>命名空间/仓库名称:镜像版本号</name>-->
                <name>${project.build.finalName}:${project.version}</name>
                <!--别名:用于容器命名和在docker-compose.yml文件只能找到对应名字的配置-->
                <alias>${project.name}</alias>
                <build>
                    <!--指定Docker构建上下文-->
                    <contextDir>${project.basedir}</contextDir>
                    <!--使用dockerFile文件-->
                    <dockerFile>${project.basedir}/Dockerfile</dockerFile>
                    <assembly>
                        <!--指定war包文件在Docker上下文中的存储路径名称，默认 maven/-->
                        <name>target</name>
                        <descriptorRef>artifact</descriptorRef>
                    </assembly>
                </build>
            </image>
        </images>
    </configuration>
</plugin>
```

编写 `Dockerfile` 文件

```dockerfile
FROM tomcat:9-jdk8
MAINTAINER Freesoft
USER root
ENV baseDir /usr/local/tomcat/
WORKDIR $baseDir
COPY target/tazhzl.war webapps/
EXPOSE 8080
CMD ["bin/catalina.sh","run"]
```

完成后即可使用 `mvn clean package docker:build docker:push` 命令将应用打包成 docker 镜像并推送至 docker 私服。