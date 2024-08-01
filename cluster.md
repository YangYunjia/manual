
[TOC]

# 新集群使用说明

## 网络结构

|节点|name|ip|ib|Comment
|-|-|-|-|-|
|存储节点   |storage    |192.168.4.120|10.0.4.200
|主节点     |master     |192.168.4.220|10.0.4.220
|cpu1      |node01      | 192.168.4.221|10.0.4.221
|cpu2      |node02      | 192.168.4.222|10.0.4.222
|gpu1      |gpu01      | 192.168.4.223|10.0.4.223| nvidia A40 * 8
|gpu2      |gpu02      | 192.168.4.224|10.0.4.224| nvidia RTX 4090 * 8
|路由器     ||192.168.4.1|
|跳板机     ||192.168.4.245|

## 使用说明

### 注册账户

1. 登录管理员账号 -> 系统管理 -> 用户管理 -> 新增用户
2. 用初始密码登录账号： `123456a?`

### 登录系统

1. 连接方式
    - 有线连接：将网线从交换机换到路由器
    - 无线连接（随时会取消）
        - wifi: `lthpc`
        - pw:   `Lt@2023!`

2. 本地配置
    - 设置ip为手动分配
    - 本机ip地址设置为`192.168.4.2XX`，且与网络结构中ip不同
    - 本机网关设置为`255.255.255.0`

### 开发环境

    为了避免开发环境占用卡资源，不能自动释放。请每位使用者保留一个开发环境用于调试，调试好的训练调参请提交任务进行排队。


1. 创建环境

    环境就是一个docker；个人文件夹`<username>`被挂载在docker中
    - 业务管理 -> 开发环境 -> 创建
    - 选择镜像

        |镜像组|镜像|Comment|兼容性|Size|
        |-|-|-|-|-|
        |`pytorch`|`py3.9-openmpi-yangyunjia:v2.0`|配置好了`pytorch` `openmpi` | `cfl3d_mpi (v6.8)` `flowGen` `cgrid`|15.6G|
        |`pytorch`|`pytorch-yangyunjia-yangyunjia:v1.0`|配置好了`pytorch`||4.8G

    - 选择所需要的计算资源
    - 等待拉取结束

2. 在环境内使用

    就是一个普通的`linux`系统
    
    - 开启了`jupyter`服务，可以在`jupyter`内修改文件、复制粘贴
    - 在命令行中运行

3. 文件传送

    - 小文件可以通过`jupyter`或自带文件管理上传下载
    - 大文件需要通过ssh复制

        文件在根系统中的存储路径为`mnt/inpurfs/<username>`，通过`scp`命令/ftp工具（）从本地发送

4. **长期不用开发环境请释放（在列表中删除）**

### 提交任务

1. 创建任务

    任务就是在环境的基础上指定要运行的文件

    - 业务管理 -> 任务管理 -> 创建
    - 选择镜像 （与开发环境中相同，如果涉及到自己安装的库，请选择自己配置好的镜像）
    - 由于系统上python版本的问题，如果采用脚本模式运行常常不能识别到正确的python版本，故推荐采用命令模式运行
    
        输入命令如下：

        ``` 
        cd <folder_path> && <python_path> <file_path>
        ```
        - `<folder_path>` 是你想要运行的目录，推荐采用绝对路径，例如`/yangyunjia/wing_model
        - `python_path` 是python解释器路径，如果在训练中采用了anaconda的python环境，则路径应为`/opt/anaconda3_2022.10/bin/python`。可以在开发环境中通过以下命令查询解释器路径

            ```python
            import sys
            print(sys.executable)
            ```
        - `file_path` 是待执行的python文件路径

        实例命令

        ```
        cd /yangyunjia/wing_model && /opt/anaconda3_2022.10/bin/python run.py
        ```

2. 查看任务

    - 可以在运行的任务上，点击任务编号查看返回值

# Trouble Shooting

## 重新开机后文件没有挂载上

> 常见形式：能登录系统，但是不能新建环境（节点没有挂上），或进入个人账户后没有文件（master没有挂上）

解决方法1

1. 进入相应节点
2. 检查挂载情况：`df -h | grep mnt`。正常应该显示`10.0.4.200:/mnt/inspurfs  xxT  xxT  xxT    xx% /mnt/inspurfs`
3. 如果没有挂载按以下操作
    - 3.1. 运行`source /etc/rc.local`挂载
    - 3.2. 如果返回`mount.nfs: Stale file handle`，则目前挂载情况不对，需要先用`umount /mnt/inspurfs`解挂；然后再运行3.1
    - 3.3. 如果返回不正常，需要进入`storage`(192.168.4.200)重启nfs服务：`systemctl restart nfs`

解决方法2

在节点运行`aistationservice restart`重启节点服务（会非常长时间）

# Linux 操作技巧

## 上传下载

### `tar`

- 压缩和解压 tar 文件 - 使用 tar
- 压缩和解压 gz 文件 - 使用 gzip
- 压缩和解压 zip 文件 - 分别使用 zip、unzip

```sh
tar -cvf log.tar log2012.log            # 仅打包，不压缩
tar -zcvf log.tar.gz log2012.log        # 打包后，以 gzip 压缩
tar -jcvf log.tar.bz2 log2012.log       # 打包后，以 bzip2 压缩

tar -ztvf log.tar.gz                    # 查阅上述 tar 包内有哪些文件
tar -zxvf log.tar.gz                    # 将 tar 包解压缩
tar -zxvf log30.tar.gz log2013.log      # 只将 tar 内的部分文件解压出来
```

- `tar` 命令支持正则表达式，可以使用

    ```sh
    tar -zcvf logs.tar.gz 144_*.log 154_*
    ```

    压缩正则匹配的文件/文件夹。

# 软件安装和配置

## `CFL3D`

### 编译 `CFL3D`

1.	安装ifortran

    - 下载intel fortran [链接](https://www.intel.com/content/www/us/en/developer/articles/tool/oneapi-standalone-components.html#fortran)

    - 安装教程 [链接](https://zhuanlan.zhihu.com/p/498386759)

2. `ifort`的路径 `/opt/intel/oneapi/compiler/2023.2.1/linux/bin/intel64` ，要加到 `bsahrc` 里面 

3. 设置`ifort`的路径：`/opt/intel/oneapi# source setvars.sh`
（之后，mpifort就有东西了）

4. 设置`mpif90`（本质上是个wrapper）的对象是`ifort`：

    ```sh
    export OMPI_FC=/opt/intel/oneapi/compiler/2023.2.1/linux/bin/intel64/ifort
    ```
    查看目前的路径
    ```sh
    mpif90 –showme OMRI_FC
    ```