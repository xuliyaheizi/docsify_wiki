# Docker

## 一、概念

一款产品的开发往往需要多套环境，比如开发环境、测试环境以及最终的上线环境；环境的配置是时分麻烦的，每一个机器都要部署环境（Redis、ES、Hadoop集群…），尤其是各种集群的部署特别浪费时间，常常会遇到 ”在我电脑上可以运行，在你电脑上不能运行“、”版本更新导致服务不用“、”不能跨平台“ 等等大量问题，这对运维人员来说就十分棘手。在以前，开发人员发开完成就发布一个jar或者war包，其他的都交给运维人员来做；而现在，开发即运维，打包部署上线一套流程走完：开发人员会将项目及其附带的环境一起打包jar+(Redis Jdk ES MySQL)成一整套发布，称为镜像，这样就不再需要再配置环境，直接执行一整套即可，省去环境配置的麻烦且保证了一致性；
Docker 的思想**来源于集装箱**，打包装箱，每个箱子互相隔离
Docker 通过**隔离机制**，可以将服务器运行到极致

### 1.2 Docker的应用场景

- Web 应用的自动化打包和发布。
- 自动化测试和持续集成、发布。
- 在服务型环境中部署和调整数据库或其他的后台应用。
- 从头编译或者扩展现有的 OpenShift 或 Cloud Foundry 平台来搭建自己的 PaaS 环境。

### 1.3 对比虚拟机技术

 **虚拟机技术**

缺点：

- 资源占用十分大
- 冗余步骤多
- 启动慢

 容器化技术：不是模拟的一个完整的操作系统

**Docker和虚拟机的不同：**

- 传统虚拟机技术，虚拟出一套硬件，运行一个完整的操作系统，在这个系统上安装和运行软件	
- 容器内的应用直接运行在宿主机的内核上，容器没有自己的内核，也没有虚拟硬件，轻便快速
- 每个容器互相隔离，每个容器都有一个自己的文件系统，互不影响

### 1.4 Docker的优点

- 更高效的利用系统资源
- 由于容器不需要进行**硬件虚拟化**以及**运行完整操作系统**等额外开销，Docker 对系统资源的利用率更高。无论是应用执行速度、内存损耗或者文件存储速度，都要比传统虚拟机技术更高效。因此，相比虚拟机技术，一个相同配置的主机，往往可以运行更多数量的应用。

**更快速的启动时间**

传统的虚拟机技术启动应用服务往往需要数分钟，而 Docker 容器应用，由于直接运行于宿主内核，无需启动完整的操作系统，因此可以做到秒级、甚至毫秒级的启动时间。大大的节约了开发、测试、部署的时间。

**一致的运行环境**

开发过程中一个常见的问题是环境一致性问题。由于开发环境、测试环境、生产环境不一致，导致有些 bug 并未在开发过程中被发现。而 Docker 的镜像提供了除内核外完整的运行时环境，确保了应用运行环境一致性，从而不会再出现「这段代码在我机器上没问题啊」 这类问题。

**持续交付和部署**

- 对开发和运维（DevOps）人员来说，最希望的就是一次创建或配置，可以在任意地方正常运行。
- 使用 Docker 可以通过定制应用镜像来**实现持续集成、持续交付、部署**。开发人员可以通过 Dockerfile 来进行镜像构建，并结合持续集成（Continuous Integration）系统进行集成测试，而运维人员则可以直接在生产环境中快速部署该镜像，甚至结合持续部署（Continuous Delivery/Deployment）系统进行自动部署。
- 而且使用** Dockerfile 使镜像构建透明化**，不仅仅开发团队可以理解应用运行环境，也方便运维团队理解应用运行所需条件，帮助更好的在生产环境中部署该镜像。

**更轻松的迁移**

由于 Docker 确保了执行环境的一致性，使得应用的迁移更加容易。Docker 可以在很多平台上运行，无论是物理机、虚拟机、公有云、私有云，甚至是笔记本，其运行结果是一致的。因此用户可以很轻易的将在一个平台上运行的应用，迁移到另一个平台上，而不用担心运行环境的变化导致应用无法正常运行的情况。

**更轻松的维护和扩展**

Docker 使用的分层存储以及镜像的技术，使得应用重复部分的复用更为容易，也使得应用的维护更新更加简单，基于基础镜像进一步扩展镜像也变得非常简单。此外，Docker 团队同各个开源项目团队一起维护了一大批高质量的 官方镜像，既可以直接在生产环境使用，又可以作为基础进一步定制，大大的降低了应用服务的镜像制作成本。

### 1.5 引入Docker后：DevOps（开发，运维）

**应用更快速的交付和部署**

- 传统：一堆帮助文档，安装程序
- Docker：打包镜像发布测试，一键运行

**更便捷的升级和扩缩容**

- 使用Docker之后，部署应用就和搭积木一样
- 项目打包成一个镜像，比如服务器A出现性能瓶颈，水平拓展，在服务器B上一键运行镜像

**更简单的系统运维**

- 容器化之后，开发、测试环境高度一致

**更高效的计算资源利用**

- Docker是内核级别的虚拟化，可以在一个物理机上运行很多容器实例，服务器性能可以被压榨到极致

## 二、Docker的安装配置

### 2.1 Docker的基本组成

#### 镜像

镜像就是**一个可读的模块**，可以通过这个模块创建容器服务，**一个镜像可以创建多个容器**（最终服务运行或者项目运行就是在容器中）

#### 容器

- Docker利用容器技术，独立运行的一个或一组应用。**容器是用镜像创建的运行实例**
- 它可以被启用，开始，停止，删除。**每个容器都是相互隔离的，保证安全的平台**
- 可以把容器看作是一个简易版的Linux系统（包括root用户权限，进程空间，用户空间和网络空间等）和运行在其中的应用程序
- 容器的定义和镜像几乎一模一样，也是一堆层的统一视角，唯一区别在与容器的最上面那一层是可读可写的

#### 仓库

- 仓库是集中存放镜像文件的场所
- 仓库和仓库注册服务器(Registry)是有区别的，仓库注册服务器上往往存放着多个仓库，每个仓库中又包含了多个镜像，每个有不同的标签
- 仓库分为公开仓库和私有仓库两种形式
- 最大的开发仓库是国外的Docker Hub，存放了数量庞大的镜像供用户下载
- 国内的公开仓库包括阿里云，网易云都有容器服务器（需要配置镜像加速）

### 2.2 安装

#### 2.2.1 卸载旧版本

```
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```
#### 2.2.2 安装Docker软件包

```
sudo yum install -y yum-utils
```
#### 2.2.3 设置镜像仓库地址

```
# 默认是国外的
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
    
# 换成阿里云镜像地址
sudo yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```
#### 2.2.4 安装最新版Docker Engine和容器

安装前建议先将将服务器上的软件包信息现在本地缓存，以提高安装软件的速度
```
sudo yum makecache fast

# docker-ce社区版(docker-ee企业版)
sudo yum install docker-ce docker-ce-cli containerd.io
```
#### 2.2.5 启动Docker

```
sudo systemctl start docker
```
#### 2.2.6 运行Hello World映像测试

```
sudo docker run hello-world
```
### 2.3 卸载

#### 2.3.1 卸载Docker依赖

```
# 1、卸载Docker依赖: Docker Engine,CLI和Containerd软件包
sudo yum remove docker-ce docker-ce-cli containerd.io
```
#### 2.3.2 删除Docker资源

```
# 2、删除Docker资源: 所有镜像,容器和卷(主机上的镜像,容器,卷或自定义配置文件不会自动删除)
sudo rm -rf /var/lib/docker
```
### 2.4 阿里云镜像加速

```
sudo mkdir -p /etc/docker

sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://73z5h6yb.mirror.aliyuncs.com"]
}
EOF

sudo systemctl daemon-reload

sudo systemctl restart docker

```
## 三、Docker的常用命令

**镜像管理**

```
docker images：列出本地所有镜像
docker search <IMAGE_ID/NAME>：查找image
docker pull  <IMAGE_ID> ： 下载image
docker push <IMAGE_ID>：上传image
docker rmi <IMAGE_ID>：删除image
```
**容器管理**

```
docker run -i -t <IMAGE_ID> /bin/bash：
-i：标准输入给容器    
-t：分配一个虚拟终端    /bin/bash：执行bash脚本
-d：以守护进程方式运行（后台）
-P：默认匹配docker容器的5000端口号到宿主机的49153 to 65535端口
	是容器内部端口随机映射到主机的端口。
-p <HOT_PORT>:<CONTAINER_PORT>：指定端口号
  是容器内部端口绑定到指定的主机端口。
- -name： 指定容器的名称
- -rm：退出时删除容器
--restart=always

docker stop <CONTAINER_ID>： 停止container
docker start   <CONTAINER_ID> ： 重新启动container
docker ps - Lists containers.
-l：显示最后启动的容器
-a：同时显示停止的容器，默认只显示启动状态

docker attach <CONTAINER_ID> 连接到启动的容器
docker logs <CONTAINER_ID>  : 输出容器日志
-f：实时输出
docker cp <CONTAINER_ID>:path hostpath：复制容器内的文件到宿主机目录上
docker rm <CONTAINER_ID>：删除container
docker rm `docker ps -a -q`：删除所有容器
docker kill `docker ps -q`
docker rmi `docker images -q -a`
docker wait <CONTAINER_ID>：阻塞对容器的其他调用方法，直到容器停止后退出

docker top <CONTAINER_ID>：查看容器中运行的进程
docker diff <CONTAINER_ID>：查看容器中的变化
docker inspect <CONTAINER_ID>：查看容器详细信息（输出为Json）
-f：查找特定信息，如 docker inspect  - f  '{{ .NetworkSettings.IPAddress }}'
      docker commit -m "comment" -a "author" <CONTAINER_ID>  ouruser/imagename:tag

      docker extc -it <CONTAINER> <COMMAND>：在容器里执行命令，并输出结果
```
**网络管理**

```
docker run -P：随机分配端口号
docker run -p 5000:5000：绑定特定端口号（主机的所有网络接口的5000端口均绑定容器的5000端口）
docker run -p 127.0.0.1:5000:5000：绑定主机的特定接口的端口号
docker run  - d  - p  127.0 . 0.1 : 5000 : 5000 / udp training / webapp python app . py：绑定udp端口号
docker port <CONTAINER_ID> 5000：查看容器的5000端口对应本地机器的IP和端口号
使用Docker Linking连接容器：
Docker为源容器和接收容器创建一个安全的通道，容器之间不需要暴露端口，接收的容器可以访问源容器的数据
docker run -d -P --name <CONTAINER_NAME> --link <CONTAINER_NAME_TO_LINK>:<ALIAS>  
```
**数据管理**

```
Data Volumes：volume是在一个或多个容器里指定的特殊目录
数据卷可以在容器间共享和重复使用
可以直接修改容器卷的数据
容器卷里的数据不会被包含到镜像中
容器卷保持到没有容器再使用它
可以在容器启动的时候添加-v参数指定容器卷，也可以在Dockerfile里用VOLUMN命令添加
docker run -d -P --name web -v /webapp training/webapp python app.py
也可以将容器卷挂载到宿主机目录或宿主机的文件上，<容器目录或文件>的内容会被替换为<宿主机目录或文件>的内容，默认容器对这个目录有可读写权限
docker run -d -P --name web -v <宿主机目录>:<容器目录> training/webapp python app.py
可以通过指定ro，将权限改为只读
docker run -d -P --name web -v <宿主机目录>:<容器目录>:ro training/webapp python app.py
在一个容器创建容器卷后，其他容器便可以通过--volumes-from共享这个容器卷数据，如下：
docker run -d -v /dbdata --name db1 training/postgres echo Data-only container for postgres
首先启动了一个容器，并为这个容器增加一个数据卷/dbdata，然后启动另一个容器，共享这个数据卷
docker run -d --volumes-from db1 --name db2 training/postgres
此时db2使用了db1的容器卷，当容器db1被删除时，容器卷也不会被删除，只有所有容器不再使用此容器卷时，才会被删除
docker rm -v：删除容器卷
除了共享数据外，容器卷另一个作用是用来备份、恢复和迁移数据
docker run --volumes-from db1 -v /home/backup:/backup ubuntu tar cvf /backup/backup.tar /dbdata
启动一个容器数据卷使用db1容器的数据卷，同时新建立一个数据卷指向宿主机目录/home/backup，将/dbdata目录的数据压缩为/backup/backup.tar
docker run -v /dbdata --name dbdata2 ubuntu /bin/bash
docker run --volumes-from dbdata2 -v /home/backup:/backup busybox tar xvf /backup/backup.tar
启动一个容器，同时把backup.tar的内容解压到容器的backup
```
## 四、Docker run的流程和底层原理

### 底层原理

- Docker是一个 Client-Server结构的系统，Docker的守护进程运行在主机上，通过Socker从客户端访问
- DockerServer接收到DockerClient的指令，就会执行这个命令

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/1647136139380-afe2793d-c4c6-404c-94d4-9d75c1f8342b.png" alt="image.png" style="zoom: 50%;" />

#### Docker为什么比VM快

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/1647136129476-a8b0ea74-2efa-4a8f-8ed9-2ab2024f1953.png" alt="image.png" style="zoom: 50%;" />

1. Docker有着比虚拟机更少的抽象层，由于Docker不需要Hypervisor实现**硬件资源虚拟化**，运行在Docker容器上的程序直接使用的都是实际物理机的硬件资源，因此在Cpu、内存利用率上Docker将会在效率上有明显优势。
1. Docker利用的是**宿主机的内核**，而不需要Guest OS，因此，当新建一个容器时，**Docker不需要和虚拟机一样重新加载一个操作系统**，避免了**引导、加载操作系统内核**这个比较费时费资源的过程，当新建一个虚拟机时，虚拟机软件需要加载Guest OS，这个新建过程是分钟级别的，而Docker由于直接利用宿主机的操作系统则省略了这个过程，因此新建一个Docker容器只需要几秒钟。
|  | **Docker容器** | **虚拟机（VM）** |
| --- | --- | --- |
| 操作系统 | 与宿主机共享OS | 宿主机OS上运行宿主机OS |
| 存储大小 | 镜像小，便于存储与传输 | 镜像庞大（vmdk等） |
| 运行性能 | 几乎无额外性能损失 | 操作系统额外的cpu、内存消耗 |
| 移植性 | 轻便、灵活、适用于Linux | 笨重、与虚拟化技术耦合度高 |
| 硬件亲和性 | 面向软件开发者 | 面向硬件运维者 |

## 五、Docker命令实战

### 1.部署Nginx

```
# 1. 搜索镜像
docker search nginx

# 2. 下载镜像
docker pull nginx

# 3. 查看镜像是否下载成功
docker images

# 4. 启动容器(-d:后台运行 --name:容器名 -p:暴露端口 宿主机端口3344:容器端口80)
docker run -d --name nginx01 -p 3344:80 nginx

# 5. 查看容器是否启动
docker ps

# 6. 本机测试
curl localhost:3344
```
### 2.部署Tomcat

```
# 官方使用
# 之前我们都是后台启动,停止容器后,仍可以通过docker ps -a命令查到容器的使用记录
# --rm表示用完就删除容器及历史记录,通过命令查不到;
docker run -it --rm tomcat:9.0

# tomcat启动运行
[root@zsr home]# docker run -d -p 3355:8080 --name tomcat01 tomcat
Unable to find image 'tomcat:latest' locally
latest: Pulling from library/tomcat
Digest: sha256:94cc18203335e400dbafcd0633f33c53663b1c1012a13bcad58cced9cd9d1305
Status: Downloaded newer image for tomcat:latest
ffc2e1098b06fc0c0eb259fd93801e97087cbc33d0034a534208a44c491ab7a3
# 进入容器,发现webapps为空
[root@zsr home]# docker exec -it tomcat01 /bin/bash
root@ffc2e1098b06:/usr/local/tomcat# ls
BUILDING.txt	 LICENSE  README.md	 RUNNING.txt  conf  logs	    temp     webapps.dist
CONTRIBUTING.md  NOTICE   RELEASE-NOTES  bin	      lib   native-jni-lib  webapps  work
root@ffc2e1098b06:/usr/local/tomcat# cd webapps
root@ffc2e1098b06:/usr/local/tomcat/webapps# ls
```
### 3.部署es+kibana

```
# es暴露的端口很多
# es十分耗内存
# es的数据一般需要放置在安全目录！挂载

# 启动elasticsearch容器(启动后会很卡顿,因为很占内存)
docker run -d --name elasticsearch -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:7.6.2

# 监控容器资源消耗(该命令也十分耗资源,服务器卡死)
docker stats [容器ID]
```
可以看到内存占用率很高；这时候赶紧关闭容器，增加内存的限制，再重新启动一个容器
```
# 重新启动elasticsearch容器,增加内存限制(-e ES_JAVA_OPTS="-Xms64m -Xmx512m" 最小64m 最大512m),防止服务器卡死
docker run -d --name elasticsearch -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms64m -Xmx512m" elasticsearch:7.6.2

# 查看运行的容器
[root@zsr ~]# docker ps
CONTAINER ID   IMAGE                 COMMAND                  CREATED          STATUS          PORTS                                            NAMES
b6e7bc8063a5   elasticsearch:7.6.2   "/usr/local/bin/dock…"   39 seconds ago   Up 37 seconds   0.0.0.0:9200->9200/tcp, 0.0.0.0:9300->9300/tcp   elasticsearch01

# 监控容器资源消耗
[root@zsr ~]# docker stats b6e7bc8063a5
```
## 六、Portainer可视化面板

### 什么是 portainer ?

Docker 图形化界面管理工具，提供一个后台面板供我们操作

portainer（先用这个，不是最佳选项；学习CI/CD时再用 Rancher）

### 下载

```
docker run -d -p 8088:9000 \
--restart=always -v /var/run/docker.sock:/var/run/docker.sock --privileged=true portainer/portainer
```
### 访问测试

我们用docker ps命令查看一下正在运行的容器，可以看到 portainer 正在运行
```
[root@zsr ~]# docker ps
CONTAINER ID   IMAGE                 COMMAND        CREATED          STATUS          PORTS                    NAMES
b72d6033ae2e   portainer/portainer   "/portainer"   25 seconds ago   Up 23 seconds   0.0.0.0:8088->9000/tcp   vibrant_dhawan
```
## 七、Docker镜像讲解

### 镜像是什么

**镜像** 是一种**轻量级，可执行**的软件包，用来打包软件运行环境和基于运行软件开发的软件，它包含运行某个软件所需的所有内容，包括代码、运行时、库、环境变量和配置文件。
所有的应用，直接打包成docker镜像，就可以直接跑起来

### Docker镜像加载原理

> **UnionFS(联合文件系统)：**一种分层、轻量级且高性能的文件系统，它支持对文件系统的修改作为一次提交来一层层的叠加，同时可以将不通目录挂载到同一个虚拟文件系统下（unite serveral directories into a single virtual filesystem）。Union 文件系统是 Docker 镜像的基础，镜像可以通过分层来继承，基于基础镜像（没有父镜像），可以制作各种具体的应用镜像。
> **特性：**一次同时加载多个文件系统，但从外面看起来，只能看到一个文件系统，联合加载会把各层文件系统叠加起来，这样最终的文件系统会包含所有底层的文件和目录。

Docker 的镜像实际上由一层一层的文件系统组成，这种层级的文件系统就是 UnionFS 。

bootfs(boot file system)主要包含bootloader和kernel，bootloader主要是引导加载kerel，Linux刚启动时会加载 bootfs文件系统，在Docker镜像的最底层时bootfs。这一层与我们典型的Linux/Unix内核是一样的，包含boot加载器和内核。当boot加载完成之后整个内核就都在内存中了，此时内存的使用权已由bootfs转交给内核，此时系统也会卸载bootfs

rootfs(root file system)在bootfs之上，包含的就是典型的 Linux系统中的 /dev、/proc、/bin、/etc 等标准文件。rootfs 就是各种不同的操作系统发行版。
<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/1647136088883-171c6da4-ecd8-44c0-9c31-08f71ac80a91.png" alt="image.png" style="zoom:50%;" />
平时安装的虚拟机的CentOS都是好几个G，为什么Docker中才200MB？
<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/1647136076268-f42418a8-2127-4c4b-a2de-d305a027676f.png" alt="image.png" style="zoom:50%;" />
对于一个精简的 OS，rootfs 可以很小，只需要包含最基本的命令、工具和程序库就可以了，因为底层使用的是主机的Kernel，自己只需要提供rootfs就可以了。由此可见，对于不同linux发行版，bootfs是基本一致的，rootfs会有差别，因此不同的发行版可以公用bootfs

### 分层理解

当我们下载一个镜像的时候，可以看到，是一层一层的在下载！
<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/1647136047327-7df5fa94-ace7-48ad-b4f2-7e14f4b7eee3.png" alt="image.png" style="zoom:50%;" />
资源共享：如果有多个镜像从相同的Base镜像构建而来，那么宿主机只需要在磁盘上保留一份base镜像，同时内存中也只需要加载一份base镜像，这样就可以为所有的容器服务，且每一层的镜像都可以被共享。

### 理解

- 所有的 Docker镜像都起始于一个基础镜像层，当进行修改或增加新的内容时，就会在当前镜像层之上创建新的镜像层
- 举一个简单的例子，假如基于 Ubuntu Linux 16.04 创建一个新的镜像，这就是新镜像的第一层；如果要在该镜像中添加python包，就会在基础镜像层之上创建了新的一个镜像层；如果继续添加一个安全补丁，就会创建第三个镜像层

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/1647136229204-4ce26854-8d72-424c-aef7-103fdacaaac0.png" alt="image.png" style="zoom: 67%;" />

在添加额外的镜像层的同时，镜像始终保持是当前所有镜像的组合。下图中举了一个简单的例子，每个镜像层包含 3 个文件，而镜像包含了来自两个镜像层的 6 个文件

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/1647136265918-df07b12e-023a-434b-b48e-064557c7a1a2.png" alt="image.png" style="zoom:67%;" />

上图中的镜像层跟之前图中的略有区别，主要目的是便于展示文件。
下图中展示了一个稍微复杂的三层镜像，在外部看来整个镜像只有 6 个文件，这是因为最上层中的文件7 是文件 5 的一个更新版本

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/1647136277237-803db563-654a-4d58-ba83-139b091cae1b.png" alt="image.png" style="zoom:67%;" />

这种情况下，上层镜像层中的文件覆盖了底层镜像层中的文件。这样就使得文件的更新版本作为一个新镜像层添加到镜像当中。
Docker 通过存储引擎（新版本采用快照机制）的方式来实现镜像层堆栈，并保证多镜像层对外展示为统一的文件系统。
Linux 上可用的存储引擎有 AUFS、Overlay2、Device Mapper、Btrfs 以及 ZFS。顾名思义，每种存储引擎都基于 Linux 中对应的文件系统或者块设备技术，并且每种存储引擎都有其独有的性能特点。
Docker 在 Windows 上仅支持 windowsfilter 一种存储引擎，该引擎基于 NTFS 文件系统之上实现了分层和 CoW[1]。
下图展示了与系统显示相同的三层镜像。所有镜像层堆叠并合并，对外提供统一的视图。

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/1647136291693-1b3720b0-b639-4fe2-88d7-7fde261db968.png" alt="image.png" style="zoom:67%;" />

#### 特点

Docker镜像都是只读的，当容器启动时，一个新的可写层被加载到镜像的顶部！
这一层就是我们通常说的容器层，容器之下的都叫镜像层！
<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/1647136305781-a81bf143-ee9c-484d-83c6-2d0f7ada2528.png" alt="image.png" style="zoom:67%;" />

### Commit镜像

> 当我们通过镜像启动一个容器时，分为了两层：容器层和镜像层；镜像层是可读的，容器层可写，我们的所有操作都是基于容器层，当我们对容器层修改完后，就可以再将修改后的容器层和不变的镜像曾一起打包成一个新的镜像，也就是本节要讲的Commit镜像

```
# 提交容器成为一个新的副本
docker commit
docker commit -m="提交的描述信息" -a="作者" 容器id 目标镜像名:[TAG]

```
## 八、SpringBoot微服务打包成Docker镜像

首先将Docker里面清理一下

- 移除所有的容器：docker rm -f $(docker ps -aq)
- 移除所有的镜像：docker rmi -f $(docker images -aq)

### 1.创建一个SpringBoot项目

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/1647137829095-a3729c17-6095-4335-b944-c4af3ec838e0.png" alt="image.png" style="zoom:67%;" />

### 2.打包应用

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/1647137856411-062082f7-6de2-4d58-9e2d-9a4bf9cdde7d.png" alt="image.png" style="zoom:67%;" />

### 3.编写dockerfile

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/1647137893141-0818b7df-7c8f-4c86-9f05-6d478608a182.png" alt="image.png" style="zoom:67%;" />

### Dockerfile指令详解

| 指令 | 作用 | 示例 |
| --- | --- | --- |
| FROM | 指定该镜像的基础镜像 | FROM java:8 |
| MAINTAINAER | 指定作者信息 | MAINTAINAER zhulin |
| RUN | 构建镜像时执行的脚本 | RUN ["executable", "param1", "param2"] |
| ENTRYPOINT | 容器启动后执行的命令 | ENTRYPOINT _[_"java","-jar","/weather.jar"_]_ |
| CMD | 容器启动后执行的命令 | CMD ["executable", "param1", "param2"]
CMD ["param1","param2"] -- 此种格式是提供给 ENTRYPOINT 的默认参数； |
| EXPOSE | Docker服务器容器暴露的端口号，供互联系统使用 | EXPOSE 8080 |
| ENV | 指定环境变量 | ENV spring.profiles.active prod |
| ADD/COPY | 添加文件(夹)到容器 | ADD/COPY . /tmp/app |
| VOLUME | 创建一个可以从本地主机或其他容器挂载的挂载点，一般用来存放数据库和需要保持的数据等 | VOLUME ["/logs"] |
| USER | 运行容器时的用户名或UID | USER root |

### SpringBoot Dockerfile范文

```

# 基础镜像
FROM maven:3.5.0-jdk-8-alpine
 
# 作者
MAINTAINER liweichao
 
# 添加当前目录到镜像中
ADD . /tmp/app/
 
# 构建应用，串写指令，压缩镜像大小。
RUN cd /tmp/app && mvn clean package -X &&\
    # 拷贝编译结果到指定目录
    mv /tmp/app/target/*.jar /var/lib/app.jar &&\
    #清理编译痕迹
    rm -rf /tmp/app
 
# 设置编码格式
ENV LANG="zh_CN.UTF-8"
 
#设置环境变量
ENV spring.profiles.active="prod"
 
# 日志挂载
VOLUME ["/logs"]
 
# 暴露端口
EXPOSE 8080 9090 50983
 
# 启动执行
ENTRYPOINT java -server -Dfile.encoding=UTF-8 -Xmx1025m -Xss256k -Xdebug -Xnoagent -Djava.compiler=NONE -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=50983 -Djava.security.egd=file:/dev/./urandom -jar /var/lib/app.jar
```
### 4. 构建镜像

在jar包和dockerfile文件下构建镜像
**docker build -t testhellojar .**
### 5.发布运行

运行：docker run -d -p 服务器端口 : 项目端口号  --name 容器名 镜像名

## 使用idea集成远程docker部署项目

### 1.修改docker配置文件

```
vim /usr/lib/systemd/system/docker.service

在ExecStart=/usr/bin/dockerd-current 后面加上

-H tcp://0.0.0.0:2375 -H unix://var/run/docker.sock

重新加载配置文件，重启docker
systemctl daemon-reload

systemctl start docker
```
### 2.打包插件配置（dockerfile-maven)

```xml
<build>
  <plugins>
    <plugin>
      <groupId>com.spotify</groupId>
      <artifactId>dockerfile-maven-plugin</artifactId>
      <version>1.4.13</version>
      <executions>
        <execution>
          <id>default</id>
          <goals>
            <goal>build</goal>
            <goal>push</goal>
          </goals>
        </execution>
      </executions>
      <configuration>
        <repository>javastack/${project.name}</repository>
        <tag>${project.version}</tag>
        <buildArgs>
          <JAR_FILE>${project.build.finalName}.jar</JAR_FILE>
        </buildArgs>
        <dockerfile>src/main/docker/Dockerfile</dockerfile>
      </configuration>
    </plugin>
  </plugins>
</build>

```
### 3.idea内Dockerfile文件配置

```
# 添加 Java 8 镜像来源
FROM java:8

# 添加参数
ARG JAR_FILE

# 添加 Spring Boot 包
ADD target/${JAR_FILE} app.jar

# 执行启动命令
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]

```
## 九、Docker网络桥接  bridge

### Bridge存在的目的

隔离各个容器，使得每个容器的端口号都是隔离的。如果不隔离开来，那么容器将和宿主机，容器和容器间都会发生端口占用的情况。
docker的桥接网络使用虚拟网桥，bridge网络用于同一主机上的docker容器相互通信，连接到同一个网桥的docker容器可以相互通信，当我们启动docke时，会自动创建一个默认bridge网络，除非我们进行另外的配置，新创建的容器都会自动连接到这个网络，我们也可以自定义自己的bridge网络，docker文档建议使用自定义bridge网络
连接到同一bridge网络的容器可以相互访问彼此任意一个端口，如果不发布端口，外界将无法访问这些容器，在创建容器时，通过-p或是--publish指令发布端口

### 自定义bridge网络与默认bridge网络对比：

- 默认桥接网络中的容器只能通过IP地址访问其他容器（除非使用遗留的-link指令连接两个容器），而自定义桥接网络提供DNS解析，可以通过容器的名字或是别名访问其他容器
- 容器可以自由的进入或是退出自定义桥接网络，如果想要退出默认桥接网络，需要先停止容器的运行，然后重新创建该容器，并指定需要连接的其他网络
- 如果更改了默认桥接网络的网络配置，需要重新启动docker，并且由于默认桥接网络只有一个，因此所有容器的网络配置都是一样的，而用户自定义网络可以在创建时指定网络配置（例如默认网关、MTU等），不需要重启docker，灵活性更高
- 在默认桥接网络中，可以通过--link参数连接两个容器来共享环境变量，用户自定义网络中无法使用这种方式，但是docker提供了更好的方式：                 
   1. 多个容器可以使用docker volume（这是docker存储数据的一种方式，以后会介绍）挂载到同一个文件，在文件中指明环境变量，从而实现所容器的环境变量共享                                                                  
   1. 多个容器可以使用同一个docker-compose（与docker service有关）文件启动 ，可以在该文件中定义共享环境变量                                                   
   1. 可以使用swarm services，并且通过  secrets 和 configs  （这两个还没看）实现环境变量共享
### 使用自定义网络

#### 1.创建一个自定义网络

> $ docker network create 网络ID

可以指定子网、IP地址范围、网关等网络配置，详情请看[docker network create](https://docs.docker.com/engine/reference/commandline/network_create/#specify-advanced-options)

#### 2.移除自定义网络

> $ docker network rm 网络ID

移除自定义网络前先移除该网络上的所有容器
#### 3.查看某个网络的详情

> $docker network inspect 网络ID

#### 4.两个容器通过bridge网络互连

> $docker network connect 网络ID 容器名

#### 5.离开用户自定义网络

> $docker network disconnect 网络ID 容器名

