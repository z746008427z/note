## 常用命令

- 从远程抽取镜像名

    ```shell
    docker pull 镜像名<:tags>
    ```

- 查看本地镜像

    ```shell
    docker images
    ```

- 创建容器，启动应用

    ```shell
    docker run 镜像名<:tags>
    # 前面是宿主机端口:容器内部8080端口
    docker run -p 8000:8080 tomcat
    # -d 代表后台运行
    docker run -p 8000:8080 -d tomcat
    ```

- 查看正在运行中得镜像

    ```shell
    docker ps
    ```

- 删除容器

    ```shell
    docker rm <-f> 容器id
    ```

- 删除指定版本得镜像

    ``````shell
    docker rmi <-f> 镜像名:<tags>
    ``````

- 优雅停止正在运行的容器

    ```shell
    docker stop 容器id
    ```

    









## 容器命令

**使用 docker container my_command**

- create —— 从镜像中创建一个容器

    ```shell
    # 从一个镜像中创建容器
    docker container create my_repo/my_image:my_tag
    # 下文中把 my_repo/my_image:my_tag 缩写为 my_image。
    
    # -a 是 -attach缩写，指将容器连接到 STDIN,STDOUT 或 STDERR
    docker container create -a STDIN my_image
    ```

- start —— 启动一个已有的容器

    ```shell
    # 启动一个已有的容器
    docker container start my_container
    ```

- run —— 创建一个新的容器并启动它

    ```shell
    # run命令 将 create 和 start 合并到一个命令
    docker container run my_image
    # -i 是 -interactive 缩写，即使未连接，也要保持 STDIN 打开
    # -t 是 -tty 缩写，会分配一个伪终端，将终端与容器的 STDIN 和 STDOUT 连接起来
    # -p 是 -port 缩写，1000：8000 将 Docker 端口 8000 映射到计算机上的端口 1000
    # -rm 自动删除停止运行的容器
    docker container run -i -t -p 1000:8000 -rm my_image
    # -d 是 -detach 缩写，旨在后台运行容器，允许在容器运行时将终端用于其他命令
    docker container run -d my_image
    ```

- ls —— 列出正在运行的容器

    ```shell
    # 列出运行中的容器，同时提供关于容器有用的信息
    docker container ls
    # -a 是 -all 的缩写，列出所有容器（不仅是正在运行的容器）
    # -s 是 -size 的缩写，列出每个容器的大小
    docker container ls -a -s
    ```

- inspect —— 查看关于容器的信息

    ```shell
    # 查看有关容器的信息
    docker container inspect my_container
    ```

- logs —— 打印日志

    ```shell
    # 列出容器日志
    docker container logs my_container
    ```

- stop —— 优雅停止正在运行的容器

    ```shell
    # 优雅地停止一个或多个正在运行的容器。在容器关闭之前提供默认 10 秒以完成任何进程。
    docker container stop my_container
    ```

- kill —— 立即停止容器中的主要进程

    ```shell
    # 立即停止一个或多个正在运行的容器。这就像拔掉电视上的插头一样。但是在大多数情况下，建议使用 stop 命令。
    docker container kill my_container
    # 终止所有运行中的容器
    docker container kill $(docker ps -q)
    ```

- rm —— 删除已经停止的容器

    ```shell
    # 删除所有不再运行中的容器
    docker container rm $(docker ps -a -q)
    ```



## 镜像命令

**使用 docker image my_command**

- build —— 构建一个镜像

    ```shell
    # 在指定路径或 url 的 Dockerfile 中构建一个名为 >my_image 的 Docker 镜像。
    # -t 是 tag 的缩写，是告诉 docker 用提供的标签来标记镜像
    # 结尾的句号(.) 是告诉docker 根据当前工作目录中的 dockerfile 构建镜像
    docker image build -t my_repo/my_image:my_tag .
    
    # 登录到 docker 镜像仓库，根据提示键入用户名密码
    docker login
    ```

- push —— 将镜像推送到远程镜像仓库中

    ```shell
    # 推送一个镜像到仓库
    docker image push my_repo/my_image:my_tag
    ```

- ls —— 列出镜像

    ```shell
    # 列出你的镜像以及每个镜像的大小
    docker image ls
    ```

- history —— 查看中间镜像信息

    ```shell
    # 显示镜像的中间镜像，包括大小及其创建方式
    docker image history my_image
    ```

- inspect —— 查看关于镜像的信息，包括层

    ```shell
    # 显示关于镜像的细节，包括组成镜像的层
    docker image inspect my_image
    ```

- rm —— 删除镜像

    ```shell
    # 删除指定镜像。如果镜像被保存在镜像仓库中，那么该镜像在那依旧可用。
    docker image rm my_image
    # 删除所有镜像。必须小心使用这一命令。请注意已经被推送到远程仓库的镜像依然能够保存，这是镜像仓库的一个优势。
    docker image rm $(docker images -a -q)
    ```

    

## 容器&镜像

- docker version —— 列出关于 docker 客户端以及服务器版本的信息
- docker login —— 登录到 docker 镜像仓库
- docker system prune —— 删除所有未使用的容器、网络及无名称的镜像