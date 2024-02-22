# Docker30分钟入门

# 简介

一个用于构建、运行、传送应用程序的平台，将我们的应用打包成集装箱，小鲸鱼就会帮我们运送到任何需要的地方。通过Docker可以将配置文件、启动命令、应用程序、第三方软件包等打包到一起，以便在任何环境中都可以正确运行。

# 为什么使用Docker

假设我们要开发一个前端Vue、后端SpringBoot、数据库MySQL的项目。

如果没有Docker，我们可能需要先安装node.js，npm环境，java运行时环境，SpringBoot各种依赖，MySQL数据库，最后配置环境变量，再启动服务，网站才能正常运行起来。

项目更大的话可能还需要Redis缓存、Nginx负载均衡和各种微服务框架。

**还没完！**，如果要把项目部署到测试或者生产环境上，**还需要把这些步骤再重复一遍。**

而有了Docker，就可以把前端、后端、数据库打包成一个个的集装箱，只要在开发环境中运行成功，那么在其他环境中就一定可以成功。

# 和虚拟机的区别

`VMware VirtualBox WSL`，各种虚拟机都是提供完整的系统，是通过虚拟化程序实现的。

虚拟化：是将物理资源虚拟化为多个逻辑资源的技术，每个逻辑服务器都有自己的CPU 内存 硬盘 网络接口等等，他们之间完全隔离，可以独立运行。虚拟机一定程度上实现了资源整合，将一台服务器实现多台服务器的功能。

但每个虚拟机都需要占用大量资源，而且启动速度很慢。大多数情况下服务器只需要运行一个提供服务的应用程序就可以，不需要一个完整操作系统的所有功能，多少是**有点小题大做**了。会浪费大量的资源来运行系统内核、工具和图形界面等等。这些不需要的服务占用了大量资源，导致资源浪费和启动慢的问题。

容器：也是一种虚拟化技术，和虚拟机类似，也是一个独立的环境。可以运行应用程序。但是他不需要运行一个完整的操作系统，而是使用宿主机的操作系统，所以启动很快。同时需要的资源更少，可以在一台物理服务器上运行更多容器，充分利用资源减少浪费。

**注：Docker和容器是两个不同的概念**，Docker是容器的一种实现。

# 基本原理和概念

注：这里不会详细介绍，只介绍几个重要概念。

- **镜像images**：只读的模板，用来创建容器（Java中的类）
- **容器container**：容器是Docker的运行实例，提供了一个独立、可移植的环境，可以运行应用程序（Java类的实例化对象）
- 仓库：存放镜像的地方，最流行的是DockerHub

# 安装

Docker使用CS架构，C和S之间通过socket或者RESTful API进行通信，

Docker Daemon是S端的守护进程，负责管理Docker的各种资源，

C端负责向Daemon发送请求，接收请求后处理并返回给客户端。

所以各种客户端，都是通过客户端发送给Daemon的，Daemon处理并返回给客户端，然后就可以在终端看到结果了。

**注：微软[官方](https://learn.microsoft.com/zh-cn/windows/wsl/tutorials/wsl-containers)提供了基于WSL2来设置Docker Desktop for Windows的教程，这允许在没有开启hyper-V的电脑中使用Docker Desktop。**

通过WSL来使用Docker，安装完成后通过`sudo service docker start`启动Docker服务，通过`docker version`查看是否启动成功，成功的话可以看到`Server: Docker Engine - Community`的相关信息

通过命令`sudo systemctl enable docker`可以设置Docker开机启动（注：可能会没有作用，解决方法如下）

## 关于设置开机自启无效

以下内容来自Linux中国的知乎文章，[systemd 已可用于 WSL | Linux 中国](https://zhuanlan.zhihu.com/p/567924469)

先通过`ps -p 1`来查看一下，如果CMD属性为`systemd`，那自启动应该可以正常执行，否则需要先回到Windows中，试一下`wsl --version`，如果不能显示如下的信息，则需要执行一下`wsl --update`。

![image-20240222111938725](.\images\image-20240222111938725.png)

当`wsl --version`可以正常显示时，就可以下一步了。

回到WSL命令行中，`sudo vi /etc/wsl.conf`写一个`wsl.conf`文件，按`i`进入`insert`也就是编辑模式，输入

```
[boot]
systemd=true
```

按esc输入后输入`:wq`，重启WSL后输入`systemctl`并回车来尝试命令是否可用，显示出一大堆文件则说明成功，此时就可以再设置开机自启了。

![image-20240222112625714](.\images\image-20240222112625714.png)

# 容器化

将应用程序打包成容器，在容器中运行程序的过程。

1. 创建Dockerfile
2. 使用Dockerfile构建镜像
3. 使用镜像创建和运行容器

Dokefile是一个文本文件包含了指令，用来告诉Docker如何构建镜像，这个镜像中包括应用程序执行的所有命令，也就是各种依赖、配置环境和运行程序所需的所有内容。

# 实践环节

在命令行中创建一个文件夹，打开文件夹，新建一个程序并输出一段内容。

![image-20240221221154632](.\images\image-20240221221154632.png)

如果想要在没有Python的环境下运行`.py`文件，就需要安装python（跟废话似的）

这里创建一个`index.js`并输出一些内容

所以要在另一个环境中运行应用程序，需要哪些步骤？

1. 安装OS
2. 安装运行环境（python）
3. 复制程序、依赖包、配置文件
4. 执行启动命令运行程序

有了Docker后，将这些步骤写到Dockerfile中，剩下的工作由Docker自动完成。

```dockerfile
FROM node:14-alpine
COPY index.js /index.js
# CMD [ "node", "/index.js" ] 或写下方的形式
CMD node /index.js
```

在终端中使用`docker build -t 镜像名 .`，`.`表示当前目录，就是Dockerfile所在的目录。

执行成功后就生成了Docker镜像，使用`docker images`或`docker image ls`来查看所有的镜像

![image-20240221222850227](.\images\image-20240221222850227.png)

如何运行？`docker run hello-docker`

