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
