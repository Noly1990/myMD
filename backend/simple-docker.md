# Simple Docker

> Docker入门及安装

## Docker的理论基础

> 简介：Docker 是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化。容器是完全使用沙箱机制，相互之间不会有任何接口。

Docker以下几点要点：
- Docker镜像(Images)

    俗称Docker的镜像，这个可难懂了。你暂时可以认为这个就像我们要给电脑装系统用的系统CD盘，里面有操作系统的程序，并且还有一些CD盘在系统的基础上安装了必要的软件，做成的一张 “只读” 的CD。
    
- Docker容器(Container)

    俗称Docker的容器，这个是最关键的东西了。Docker Container是真正跑项目程序、消耗机器资源、提供服务的地方，Docker Container通过Docker Images启动，在Docker Images的基础上运行你需要的代码。 你可以认为Docker Container提供了系统硬件环境，然后使用了Docker Images这些制作好的系统盘，再加上你的项目代码，跑起来就可以提供服务了。 
    
- Docker客户端(Client)

    Docker提供给用户的客户端。Docker Client提供给用户一个终端，用户输入Docker提供的命令来管理本地或者远程的服务器。
    
- Docker主机(Host)
 
    一个物理或者虚拟的机器用于执行 Docker 守护进程和容器。也就是运行Docker的主体，相当于VMware虚拟出的Centos系统

- Docker仓库(Registry)

    这个可认为是Docker Images的仓库，就像git的仓库一样，用来管理Docker镜像的，提供了Docker镜像的上传、下载和浏览等功能，并且提供安全的账号管理可以管理只有自己可见的私人image。就像git的仓库一样，docker也提供了官方的Registry，叫做Dock Hub(http://hub.Docker.com)
    
- Docker Machine
    
    Docker Machine是一个简化Docker安装的命令行工具，通过一个简单的命令行即可在相应的平台上安装Docker，比如VirtualBox、 Digital Ocean、Microsoft Azure。

## Docker的安装

在Vmware里新建一个CentOS 7.X 以供学习使用，基本的VMware的安装自行搜索

在CentOS中直接使用yum源安装Docker，官方建议安装docker ce（社区版） ，但似乎目前1.13.1的安装使用起来也没有什么问题。

#### 安装

```
yum install docker


//运行docker服务
service docker start


docker version
//检查版本，正确的如下
Client:
 Version:         1.13.1
 API version:     1.26
 Package version: <unknown>
 Go version:      go1.8.3
 Git commit:      774336d/1.13.1
 Built:           Wed Mar  7 17:06:16 2018
 OS/Arch:         linux/amd64
Server:
 Version:         1.13.1
 API version:     1.26 (minimum version 1.12)
 Package version: <unknown>
 Go version:      go1.8.3
 Git commit:      774336d/1.13.1
 Built:           Wed Mar  7 17:06:16 2018
 OS/Arch:         linux/amd64
 Experimental:    false
```

#### 切换国内镜像

```
//建议把docker的Registry设置成国内的镜像加快下载速度
vi /etc/docker/daemon.json
{ 
"registry-mirrors": ["https://registry.docker-cn.com"] 
}

//更改配置文件后重启docker服务
service docker restart
```

以上就算在测试机上安装完docker，为了便于测试，我们再把系统的防火墙关闭

```
//停止firewall
systemctl stop firewalld.service 

//禁止firewall开机启动
systemctl disable firewalld.service 
```
后续如果有相关的安全配置，推荐使用iptables进行管理端口限制

---

## Docker的简单流程

先不论快捷的docker建立方式，来描述下常规情况下一个docker容器的建立过程

```
graph LR
Dockerfile-->Image;
Image-->Container;
```

### Dockerfile

Dockerfile相当于是build image的一个指南，通过一些描述，制作适合目的条件的image镜像=>[前往简要说明](https://blog.csdn.net/marco_90/article/details/53434937)

### Image

Image相当于系统带上相关必备软件的集合，由Dockerfile描述而来

常用image指令
```
docker images||docker image //查看当前image
docker image build  //构建image
docker image pull   //从源库下载image
docker image rm     //删除image
```

### Container

Container是最终运行服务的容器，是Docker最终的载体

常用的container指令
```
docker container ls || docker container ls -all  //查看当前||素有容器
docker container run    //运行一个新容器
docker container start      //开始一个已有容器
docker container restart    //重启一个已有容器
docker container rm     //删除一个容器
docker container stop   //停止一个正在运行容器
docker container inspect    //查看容器的详情
docker container exec   //一般用于进入容器
docker container kill   //杀死一个正在运行的容器
```

### 常用docker指令
```
docker cp 从本机复制文件到特定容器
```

#### 一个简单示例

编写完Dockerfile以后，使用docker build指令制作image，下面我们来看一个简单的node.js的koa2的web应用

1. 新建一个目录docker-demo，再新建Dockerfile，然后添加.dockerignore;

```
//docker-demo/Dockerfile

FROM node:8.11
COPY . /app
WORKDIR /app
RUN npm install --registry=https://registry.npm.taobao.org
EXPOSE 3000

```
```
//docker-demo/.dockerignore

.git
node_modules
npm-debug.log

```
- FROM: 以8.11的node版本为基础
- COPY：将.（当前目录下的文件）复制到容器中得/app目录
- WORKDIR: 之后运行命令的工作目录为/app
- RUN: 运行一个指令，这里为npm安装依赖
- EXPOSE: 服务暴露的端口为3000

.dockerignore是打包image文件的忽略配置，与.gitignore类似

2. 在当前目录下的终端，运行npm init -y新建个package.json，新建个app.js;

```
//docker-demo/package.json

{
  "name": "koa2server",
  "version": "1.0.0",
  "description": "",
  "main": "app.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
    "koa2": "^2.0.0-alpha.7"
  }
}
```

```
//docker-demo/app.js

const Koa = require('koa2')
const app = new Koa()

app.use( async ( ctx ) => {
  ctx.body = 'hello koa2'
})

app.listen(3000)
console.log('[demo] start-quick is starting at port 3000')

```
3. 使用image build制作image文件
```
docker image build -t koa2-demo .
```
-t 用来指定image的名字，最后有个 . 代表从当前目录创建image

4. 创建完成以后就能查看了

```
docker image ls
```
如果没有错误，应该会如下所示的反馈
```
REPOSITORY              TAG                 IMAGE ID            CREATED             SIZE
koa2-demo                latest              8824ba699e80        2 hours ago         678 MB
```

5. 创建完image之后是，通过image创建运行容器

```
docker container run -p 8000:3000 -it koa-demo /bin/bash
```
- -p : 端口映射,用本机的8000端口映射容器的3000端口
- -it : 容器的shell映射当前shell，可以操作容器的shell
- /bin/bash : 容器启动后，第一个执行/bin/bash，让用户可以连接shell操作

6. 如果运行正常，会返回容器的shell界面，然后查看下目录文件 ls ，然后运行相应的程序node app.js
```
root@66d80f4aaf1e:/app# node app.js
```
7. 如果一切正常，访问本机的localhost:8000就能收到hello koa2的字样，表示服务启动成功


## Docker的运用实例

下面通过三个例子，来对docker的实际运用做介绍，尤其适合在自己的开发机上，进行docker部署各种组建，这样对于自己的电脑也不会运行太多的服务，影响性能，方便管理
- 通过docker部署node.js
- 通过docker安装mysql
- 通过docker安装nginx
- 通过docker安装redis

1. 通过docker部署node.js项目

在上一节中就已经展示了一个node项目如何部署到docker中的，这里再通过另外一种方式，完成
目前的node.js的LTS版本是8.X.X，当前最新的是8.11.1,所以我们只要在docker pull一个8.11版本的node image,先search一下
```
docker search node:8.11
docker pull node:8.11
```

2. 通过docker安装mysql

mysql的安装，以mysql_5.7为例,下载image安装前搜一搜
```
docker search mysql
//搜索结果
docker pull mysql:5.7
5.7: Pulling from library/mysql
f2aa67a397c4: Pull complete
1accf44cb7e0: Pull complete
2d830ea9fa68: Pull complete
740584693b89: Pull complete
4d620357ec48: Pull complete
ac3b7158d73d: Pull complete
a48d784ee503: Pull complete
bf1194add2f3: Pull complete
0e5c74178a02: Pull complete
e9201d309436: Pull complete
bf1ac4524e8e: Pull complete
Digest: sha256:f030e84582d939d313fe2ef469b5c65ffd0f7dff3b4b98e6ec9ae2dccd83dcdf
Status: Downloaded newer image for mysql:5.7
```

在本机上新建一个dockermysql文件夹，存储mysql的相关信息，用户后续对容器文件目录的映射，在文件夹内新建conf,logs,data文件夹，存储对应的信息
在dockermysql文件下运行

```
docker run -p 3307:3306 --name mysql5.7 -v $PWD/conf/my.cnf:/etc/mysql/my.cnf -v $PWD/logs:/logs -v $PWD/data:/mysql_data -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7
```
因为我本机存在运行在3306端口上的mysql，所以在-p把容器的mysql映射到本机3307端口，如果没有自定义配置，就省略conf的-v

3. 通过docker安装nginx（似乎并没有什么必要）



4. 通过docker安装redis
