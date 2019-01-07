# 1 Docker架构和底层技术简介
![Docker Platform](https://upload-images.jianshu.io/upload_images/4685968-8a50f52c8c208765.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![Docker Engine](https://upload-images.jianshu.io/upload_images/4685968-94cbf0ab167019a5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/4685968-de2befc2c04d25f8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/4685968-f5582bfe970d0c13.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![Docker Architecture](https://upload-images.jianshu.io/upload_images/4685968-468459a0cd76a9f0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![底层技术支持](https://upload-images.jianshu.io/upload_images/4685968-2483bf3e509b4f14.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
# 2 Docker Image概述
![](https://upload-images.jianshu.io/upload_images/4685968-70ef1300c4bdfbcb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![dockerimage结构](https://upload-images.jianshu.io/upload_images/4685968-5e3ffdc7ede27900.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
从基本的看起，一个典型的 Linux 文件系统由 bootfs 和 rootfs 两部分组成，
- bootfs(boot file system) 主要包含 bootloader 和 kernel，bootloader 主要用于引导加载 kernel，当 kernel 被加载到内存中后 bootfs 会被 umount 掉
- rootfs (root file system) 包含的就是典型 Linux 系统中的/dev，/proc，/bin，/etc 等标准目录和文件
![docker image 中最基础的两层结构](https://upload-images.jianshu.io/upload_images/4685968-fe22863d36925635.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
不同的 linux 发行版（如 ubuntu 和 CentOS ) 在 rootfs 这一层会有所区别，体现发行版本的差异性
传统的 Linux 加载 bootfs 时会先将 rootfs 设为 read-only，然后在系统自检之后将 rootfs 从 read-only 改为 read-write，然后就可在 rootfs 上进行读写操作了
但 Docker 在 bootfs 自检完毕之后并不会把 rootfs 的 read-only 改为 read-write，而是利用 union mount（UnionFS 的一种挂载机制）将 image 中的其他的 layer 加载到之前的 read-only 的 rootfs 层之上，每一层 layer 都是 rootfs 的结构，并且是read-only 的。所以，我们是无法修改一个已有镜像里面的 layer 的！只有当我们创建一个容器，也就是将 Docker 镜像进行实例化，系统会分配一层空的 read-write 的 rootfs ，用于保存我们做的修改。一层 layer 所保存的修改是增量式的，就像 git 一样
![](https://upload-images.jianshu.io/upload_images/4685968-46da122fa66ef4fc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
## 2.2 image的获取
### image的获取-1
![](https://upload-images.jianshu.io/upload_images/4685968-6bb5b24cad291667.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
### image的获取-2
![image的获取-2](https://upload-images.jianshu.io/upload_images/4685968-20c81225745462c3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![sudo docker pull ubuntu:16.04](https://upload-images.jianshu.io/upload_images/4685968-1d89f39390415913.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/4685968-7f6b1690bce02627.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![官方镜像仓库](https://upload-images.jianshu.io/upload_images/4685968-c7a0f3cf52174d23.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
# 3 DIY Base Image
![无需再用 sudo 权限](https://upload-images.jianshu.io/upload_images/4685968-24518ba6f7cb3f97.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/4685968-81717c25f8bf2f20.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/4685968-1b7bde9a7c7822e0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/4685968-3e4122555b3c50af.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/4685968-3680db12feae63a7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![运行结果](https://upload-images.jianshu.io/upload_images/4685968-8aeed6a6d3115375.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![vim Dockerfile](https://upload-images.jianshu.io/upload_images/4685968-d3d6e05936a5066f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![docker build -t root/hello-world .](https://upload-images.jianshu.io/upload_images/4685968-02b6d659365997ec.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/4685968-542b9b43e06f7fdb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![docker run root/hello-world](https://upload-images.jianshu.io/upload_images/4685968-563e8925bbde05e5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
# 4 初识Container
![什么是 Container](https://upload-images.jianshu.io/upload_images/4685968-62a26b73ac9a7279.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/4685968-43077ea9a69b27f2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
docker rm 默认就是移除 container
docker images = docker image ls
![](https://upload-images.jianshu.io/upload_images/4685968-5141e2b3adb62b25.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- docker rmi = docker image rm
![](https://upload-images.jianshu.io/upload_images/4685968-bad602072d2e7470.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![docker ps -a](https://upload-images.jianshu.io/upload_images/4685968-4cfa4be425ec20eb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![只显示 id](https://upload-images.jianshu.io/upload_images/4685968-c51873bbae28658e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![删除全部](https://upload-images.jianshu.io/upload_images/4685968-ef425e27274a4b7b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
# 5 构建自己的Docker镜像
- docker container commit
从容器创建一个新的镜像
```
-a :提交的镜像作者；

-c :使用Dockerfile指令来创建镜像；

-m :提交时的说明文字；

-p :在commit时，将容器暂停。
```
![](https://upload-images.jianshu.io/upload_images/4685968-1132c54fcc2051f9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- docker image build
使用 Dockerfile 创建镜像
```
使用当前目录的 Dockerfile 创建镜像，标签为 runoob/ubuntu:v1。

docker build -t runoob/ubuntu:v1 . 
使用URL github.com/creack/docker-firefox 的 Dockerfile 创建镜像。

docker build github.com/creack/docker-firefox
也可以通过 -f Dockerfile 文件的位置：

$ docker build -f /path/to/a/Dockerfile .
在 Docker 守护进程执行 Dockerfile 中的指令前，首先会对 Dockerfile 进行语法检查，有语法错误时会返回：

$ docker build -t test/myapp .
Sending build context to Docker daemon 2.048 kB
Error response from daemon: Unknown instruction: RUNCMD
```
![](https://upload-images.jianshu.io/upload_images/4685968-22c686d535b6da64.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- docker run 
创建一个新的容器并运行一个命令
```
-a stdin: 指定标准输入输出内容类型，可选 STDIN/STDOUT/STDERR 三项；

-d: 后台运行容器，并返回容器ID；

-i: 以交互模式运行容器，通常与 -t 同时使用；

-p: 端口映射，格式为：主机(宿主)端口:容器端口

-t: 为容器重新分配一个伪输入终端，通常与 -i 同时使用；

--name="nginx-lb": 为容器指定一个名称；

--dns 8.8.8.8: 指定容器使用的DNS服务器，默认和宿主一致；

--dns-search example.com: 指定容器DNS搜索域名，默认和宿主一致；

-h "mars": 指定容器的hostname；

-e username="ritchie": 设置环境变量；

--env-file=[]: 从指定文件读入环境变量；

--cpuset="0-2" or --cpuset="0,1,2": 绑定容器到指定CPU运行；

-m :设置容器使用内存最大值；

--net="bridge": 指定容器的网络连接类型，支持 bridge/host/none/container: 四种类型；

--link=[]: 添加链接到另一个容器；

--expose=[]: 开放一个端口或一组端口；
```

docker run -it ubuntu
![](https://upload-images.jianshu.io/upload_images/4685968-c607a53211cb5c8a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

新创建的container
![](https://upload-images.jianshu.io/upload_images/4685968-6faa797109b122b9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![docker commit wizardly_feynman root/ubuntu-vim](https://upload-images.jianshu.io/upload_images/4685968-5ac081dde9692622.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/4685968-c6c154f9ce89581e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![删除刚才创建的 docker](https://upload-images.jianshu.io/upload_images/4685968-a9ede667b30d5869.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
以上新建方式不推荐,下面看 
## 5.2 dockerfile 创建方法
![](https://upload-images.jianshu.io/upload_images/4685968-2a0527b19d94ece5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/4685968-6ea1ff0840293da7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 6 Dockerfile语法梳理及最佳实践
## 6.1 FROM
![](https://upload-images.jianshu.io/upload_images/4685968-351563c28498fae3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/4685968-63e54b4e97510154.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 6.2 LABEL
就如代码中的注释
![](https://upload-images.jianshu.io/upload_images/4685968-71c14853d754b1df.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/4685968-4096084d19b9dd6f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
 
6.3 RUN
![](https://upload-images.jianshu.io/upload_images/4685968-a7fc82e0c33b03b7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/4685968-ecc90a7b741e4412.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 6.4 WORKDIR
![](https://upload-images.jianshu.io/upload_images/4685968-6441dd56a3956296.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/4685968-4d29341e8b3e9323.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## ADD and COPY
![](https://upload-images.jianshu.io/upload_images/4685968-71d2a8195bfe6ab3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/4685968-98875a7091f3738a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## ENV
![](https://upload-images.jianshu.io/upload_images/4685968-36cdabfbabb60071.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/4685968-4fb9e6d5eacf5a20.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## VOLUME and EXPOSE
![](https://upload-images.jianshu.io/upload_images/4685968-453c6e29e2cb3b23.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## CMD and ENTRYPOINT
![](https://upload-images.jianshu.io/upload_images/4685968-07127360d1b00697.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 7
![](https://upload-images.jianshu.io/upload_images/4685968-366e8c17aca38450.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## Shell 和 Exec 格式
![](https://upload-images.jianshu.io/upload_images/4685968-3d1fbaa8fbe03d23.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/4685968-a771d49d749e1fc5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
