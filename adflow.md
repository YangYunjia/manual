
# 安装ADFLOW （基于镜像）

credit from Yang XIAO

## Window下安装docker desktop + Ubuntu + WSL2 
[安装教程](https://www.jianshu.com/p/a20c2d58eaac?utm_campaign=haruki)
- 首先要安装Ubuntu系统
- 安装时按照wsl1安装，之后改为version 2 with `wsl --set-default-version 2`，不报错就运行后续命令，报错则需[更新WSL2的内核](https://aka.ms/wsl2kernel)
- 验证ubuntu是否在WSL2下启动：`wsl -l -v`，若显示version仍为1，则：
    1. 关闭wsl先：`wsl --shutdown``
    2. 然后执行：`wsl --set-version Ubuntu-18.04 2`
- WSL2默认支持docker desktop，不再需要在Ubuntu下按照docker

## 使用docker安装adflow

- 在docker desktop下找mdolab，按照需求下载对应版本（例如`mdolab/public:u20-gcc-ompi-stable`）

- 检查：`docker image ls`

- 从镜像（Image）建立容器（Container）`docker run -it --name <Container_name> --mount "type=bind,src=<source_folder_on_windows>,target=/home/mdolabuser/mount/" <image_name> /bin/bash`

    - `src`: windows下的共享文件夹路径，可以直接使用windows盘符和路径（如`D:\adflow`）。如果不行，则使用虚拟机中挂载的盘符（默认会自动挂载，如`/mnt/d/adflow`）
    - `image_name`: 镜像名，如`mdolab/public:u20-gcc-ompi-stable`
    - `bin/bash`: 命令行名，用于进入命令行

此时容器就可以用了。如果关闭后重新开启则：

- 可以在Docker Desktop中直接开启，也可以用命令`docker start <Container_name>`
- 进入命令行，使用`docker exec -it <Container_name> /bin/bash`就可以直接进入相应工作模式。
    > 管理员模式进入容器；`docker exec -it -u root ysrh  /bin/bash`不推荐，python用不了。。
- Mount目录为windows和docker之间的连接桥梁


## vscode 远程连接

[教程](https://blog.csdn.net/qq_21381465/article/details/134721992)

- 安装vscode插件：`docker`,`remote development`
- 与docker desktop兼容，直接在remote中找到container链接进去就成
- 在容器中安装python、fortran(`modern fortran`)插件。如果提示需要安装`fortls`则按照[教程](https://zhuanlan.zhihu.com/p/612056661)安装。