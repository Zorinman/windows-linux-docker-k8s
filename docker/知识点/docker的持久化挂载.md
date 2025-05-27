## docker挂载方式
**K8S都是都是默认的覆盖的挂载模式**
### ⭐注意 挂载到的本地目录之前一定要先赋予本地目录一定权限 不然会导致挂载失败 ：`chmod 777 <挂载的文件夹>`
### 一、docker 中 -v 命令带有明确路径符`/`表示本地目录挂载到容器内，这会会直接覆盖容器内的原有文件 可能会导致文件丢失除非你先把容器内的文件复制出来放到本地目录
**问：本地目录为空 为什么 使用-v 挂载时 容器内挂载目录的一些文件还是存在？**

**答**：容器启动时的程序自动写入了文件，Docker 的挂载是立即生效的，但如果你运行的容器程序（比如 Kafka）在启动脚本中会自动写入一些配置文件，那么你看到的“容器内的文件”其实是程序运行时新创建的。即使 /root/kafka/config 是空的，只要 Kafka 启动脚本在 /opt/kafka/config 中自动创建了配置文件，那么你就会看到有文件了。这不是“保留了原本的文件”，而是新写进去的。

### 二、使用docker volume来进行挂载不会覆盖容器内原文件：
#### Volume挂载与本地目录挂载的区别
`-v /xxxx` 是将本地目录挂载到容器内
`-v xxxx`  是使用docker volume是将volume挂载到容器内
和本地挂载的区别是docker volume挂载点没有明确路径符`/`

---------
#### 挂载过程 
1. `docker volume create xxxx `创建一个volume（默认创建在以下目录
/var/lib/docker/volumes/xxxx/_data）


 2. 在启动容器时使用`-v config:/opt/kafka/config` (如果没有提前创建volume 那么Docker也会 会自动创建一个 volume config，挂载到容器里)

 这种 volume 的内容在首次挂载时，会：

- 如挂载点目录`/opt/kafka/config`非空，volume 会填充镜像中原本的内容（复制容器内的原始文件）；

- 如挂载点目录`/opt/kafka/config`空，volume 不会被填充。
（以上是 Docker 的 volume 预填充机制）

#### 常见volume操作命令:
- 查看所有volume:`docker volume ls`
- 查看与某容器绑定的volume:`docker inspect 容器id`
- 删除volume：`docker volume rm volume_name`

## docker挂载注意事项
### 常见问题：
如果docker 或者docker compose 成功启动容器并保持运行 本地宿主机可以访问容器 但是其他节点不可以访问容器
**那么可能是容器的持久化目录出现问题**
### 解决
#### 1 在容器持久化挂载时 一定要保证本地挂载的目录具有足够的权限：
创建需要挂载到容器的目录：mkdir xxxx 
给与一定权限: chmod 777 xxx

#### 2.持久化挂载到容器内的目录一定要正确,必要时参考docker hub 该镜像的文档


### 例子（使用docker 创建etcd服务）:

#### 错误操作
##### 1.本地创建挂载的文件夹`mkdir -p etcd/data`

##### 2.使用docker compose 创建服务

**yaml内容**:
```yaml
version: '3'

services:
  etcd:
    container_name: etcd1
    image: bitnami/etcd:3.5.21
    environment:
      - ALLOW_NONE_AUTHENTICATION=yes
      - ETCD_NAME=etcd1
      - ETCD_LISTEN_CLIENT_URLS=http://0.0.0.0:2379
      - ETCD_ADVERTISE_CLIENT_URLS=http://192.168.219.129:2379
      - ETCD_LISTEN_PEER_URLS=http://0.0.0.0:2380
      - ETCD_INITIAL_ADVERTISE_PEER_URLS=http://192.168.219.129:2380
      - ETCD_INITIAL_CLUSTER_TOKEN=etcd-cluster
      - ETCD_INITIAL_CLUSTER=etcd1=http://192.168.219.129:2380
      - ETCD_INITIAL_CLUSTER_STATE=new
      - ETCD_LOGGER=zap
      - ETCD_LOG_LEVEL=info
    volumes:
      - /root/etcd/data:/varvar/etcd
      - "/etc/localtime:/etc/localtime:ro"
    ports:
      - 2379:2379
      - 2380:2380
    restart: always


```

##### 报错:
宿主机 使用**curl 192.168.219.129:2379**能够正确访问，其它主机使用没有任何信息返回


##### 原因:
- 1.**没有给本地etcd/data 赋予足够权限**，导致文件不能同步，使得容器内对于数据文件被清空 最终导致访问失败
- 2.**挂载的容器数据位置不正确**，需要安装官方docker hub 的来进行挂载
`/root/etcd/data:/varvar/etcd`改为`/root/etcd/data:/bitnami/etcd`