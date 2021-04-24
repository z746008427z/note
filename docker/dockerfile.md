## 基本结构

- 基础镜像信息
- 维护者信息
- 镜像操作指令
- 容器启动时执行指令
- #为 dockerfile 中的注释



## 创建容器基本步骤

1. 编写 dockerfile 文件
2. docker build
3. docker run

### 企业案例

1. 通过 dockerfile 自定义 centos 镜像

   ```shell
   docker pull centos
   ```

   ![image-20210424124153981](/Users/qixin06/note_local/note/docker/Untitled.assets/image-20210424124153981.png)

   官方镜像

   ```shell
   docker run -i -t 470671670cac
   ```

   官方镜像不支持，我们需要自定义一个镜像来支持 vim、ifconfig、并且登录后的默认路径做修改。



## 常见命令

#### FROM

​		基础镜像 —— 他的妈妈是谁

#### MAINTAINER

​		运维人员/维护人员 —— 维护者信息

#### RUN

​		你想让他做什么

#### ADD

​		向镜像中添加点什么东西 —— copy文件/会自动解压

#### WORKDIR

​		当前的工作目录

#### VOLUME

​		挂载卷 —— 你给我一个地方存放行李箱

#### EXPOSE

​		开放容器端口

#### ENV

​		环境变量

#### USER

​		用谁来运行 —— root

#### 参考文档

​		https://www.jianshu.com/p/53123da7af41



## 定义镜像

#### 编写dockerfile

```shell
# 从标准centos构建
FROM centos

# 定义作者信息
MAINTAINER tim<azkaban@163.com>

# 定义一个变量
ENV newpath /tmp

# 设置登录后的落脚点
WORKDIR $newpath

# 安装vim和net-tools工具
RUN yum -y install vim
RUN yun -y install net-tools

EXPOSE 80

CMD echo $newpath
CMD echo "success--------ok"
CMD /bin/bash
```

#### 开始构建

```shell
docker build -f dockerfile -t azkaban/custom_centos:dev
```

构建镜像成功 docker images查看

![image-20210424131548220](/Users/qixin06/note_local/note/docker/Untitled.assets/image-20210424131548220.png)

#### 创建容器及验证

```shell
docker run -i -t azkaban/custom centos:dev
ifconfig
mtu 65535
```

#### 查看构建过程

```shell
docker history azkaban/custom_centos:dev
```





























