
# 新集群使用说明

## 网络结构

|节点|name|ip|ib|Comment
|-|-|-|-|-|
|存储节点   |storage    |192.168.4.120|10.0.4.120
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

### 系统使用

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