## docker 基本概念

​		提供应用打包，部署于运行应用的容器化平台。

#### 容器与镜像

##### 镜像

​		镜像是文件，只读，提供了运行程序完整的软硬件资源，是应用程序的"集装箱"。

##### 容器

​		是镜像的实力，由docker负责创建，容器之间彼此隔离。

#### docker执行流程

![image-20210425002430742](docker.assets/image-20210425002430742.png)



## docker 安装

```shell
# 安装 工具包 和 数据存储驱动包
yum install -y yum-utils device-mapper-persistent-data lvm2

# 用于设置安装源 修改安装源
yum-confing-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

# 自动检测哪个安装源最快优先使用
yum makecache fast

# 安装环节
yum -y install docker-ce

# 启动 docker
service docker start

# 验证是否启动 查看docker版本号
docker version 

# 从国外项目抽取到本地
docker pull hello-world

# 运行
docker run hello-world
```



## 阿里云docker镜像加速

```shell
# 访问阿里郧访问镜像服务
```

































