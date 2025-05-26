## 下载docker-compose
**下载`docker-compose-linux-x86_64`名称的，这是针对linux专门编译好的二进制可执行文件**
-   1.可以去github官网下载文件再通过xftp传输到linux
-   2.可以直接在linux里使用wget:`wget https://github.com/docker/compose/releases/download/v2.16.0/docker-compose-linux-x86_64`


## 移动文件至 系统bin目录
 `mv docker-compose-linux-x86_64 /usr/local/bin/docker-compos`

## 文件夹赋权
`chmod +x /usr/local/bin/docker-compose`

## 测试安装是否成功
```
[root@localhost ~]# docker-compose --version
Docker Compose version v2.16.0
```
