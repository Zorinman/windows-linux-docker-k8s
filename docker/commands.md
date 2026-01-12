# Docker 常见命令备忘录

## 一、基本命令
| 命令 | 描述 | 示例 |
| :--- | :--- | :--- |
| `docker run` | 创建并启动一个容器 | `docker run -it ubuntu bash` |
| `docker ps` | 列出当前运行的容器 | `docker ps` |
| `docker ps -a` | 列出所有容器，包括未运行的 | `docker ps -a` |
| `docker stop` | 停止一个运行中的容器 | `docker stop [CONTAINER_ID]` |
| `docker start` | 启动一个已停止的容器 | `docker start [CONTAINER_ID]` |
| `docker restart` | 重启容器 | `docker restart [CONTAINER_ID]` |
| `docker rm` | 删除一个或多个容器 | `docker rm [CONTAINER_ID]` |
| `docker pull` | 从仓库拉取一个镜像 | `docker pull ubuntu` |
| `docker images` | 列出本地存储的镜像 | `docker images` |
| `docker rmi` | 删除一个或多个镜像 | `docker rmi [IMAGE_ID]` |
| `docker build` | 从Dockerfile构建镜像 | `docker build -t my-image:tag .` |
| `docker exec` | 在运行的容器中执行命令 | `docker exec -it [CONTAINER_ID] bash` |
| `docker logs` | 获取容器的日志 | `docker logs [CONTAINER_ID]` |

## 二、镜像管理指令
| 命令 | 描述 | 示例 |
| :--- | :--- | :--- |
| `docker images` | 列出本地的所有镜像 | `docker images` |
| `docker pull` | 从镜像仓库拉取指定的镜像 | `docker pull ubuntu:18.04` |
| `docker push` | 将本地镜像推送到镜像仓库 | `docker push myrepo/myimage:tag` |
| `docker rmi` | 删除一个或多个本地存储的镜像 | `docker rmi ubuntu` |
| `docker build` | 使用Dockerfile构建新的镜像 | `docker build -t myimage .` |
| `docker history` | 显示镜像的历史信息 | `docker history myimage` |
| `docker inspect` | 显示镜像的详细信息 | `docker inspect ubuntu` |
| `docker tag` | 为镜像添加一个新的标签 | `docker tag ubuntu:18.04 myubuntu:latest` |
| `docker save` | 将镜像保存为tar归档文件 | `docker save myimage > myimage.tar` |
| `docker load` | 从tar归档文件加载镜像 | `docker load < myimage.tar` |
| `docker import` | 从归档文件创建镜像 | `docker import mycontainer.tar myimage` |

## 三、容器管理指令
| 命令 | 描述 | 示例 |
| :--- | :--- | :--- |
| `docker run` | 创建一个新容器并运行一个命令 | `docker run -it ubuntu /bin/bash` |
| `docker ps` | 列出当前运行中的容器 | `docker ps` |
| `docker ps -a` | 列出所有容器，包括未运行的 | `docker ps -a` |
| `docker stop` | 停止一个或多个运行中的容器 | `docker stop [CONTAINER_ID]` |
| `docker start` | 启动一个或多个已停止的容器 | `docker start [CONTAINER_ID]` |
| `docker restart` | 重启一个或多个容器 | `docker restart [CONTAINER_ID]` |
| `docker kill` | 立即终止容器的运行 | `docker kill [CONTAINER_ID]` |
| `docker rm` | 删除一个或多个容器 | `docker rm [CONTAINER_ID]` |
| `docker exec` | 在运行的容器中执行命令 | `docker exec -it [CONTAINER_ID] /bin/bash` |
| `docker attach` | 连接到正在运行的容器 | `docker attach [CONTAINER_ID]` |

## 四、数据管理与卷指令
| 命令 | 描述 | 示例 |
| :--- | :--- | :--- |
| `docker volume create` | 创建一个新的卷 | `docker volume create my_volume` |
| `docker volume ls` | 列出所有卷 | `docker volume ls` |
| `docker volume inspect` | 显示指定卷的详细信息 | `docker volume inspect my_volume` |
| `docker volume rm` | 删除一个或多个卷 | `docker volume rm my_volume` |
| `docker volume prune` | 删除所有未使用的卷 | `docker volume prune` |
| `docker cp` | 从主机复制文件到容器，或者从容器复制到主机 | `docker cp foo.txt mycontainer:/foo.txt` |
| `docker create` | 创建一个新容器，用于数据卷 | `docker create -v /dbdata --name dbstore training/postgres` |
| `docker run -v` | 在运行容器时挂载卷 | `docker run -d -P --name web -v /webapp training/webapp python app.py` |

## 五、网络配置指令
| 命令 | 描述 | 示例 |
| :--- | :--- | :--- |
| `docker network create` | 创建一个新的网络 | `docker network create --driver bridge my_network` |
| `docker network ls` | 列出所有网络 | `docker network ls` |
| `docker network rm` | 删除一个或多个网络 | `docker network rm my_network` |
| `docker network inspect` | 显示一个或多个网络的详细信息 | `docker network inspect my_network` |
| `docker network connect` | 连接一个容器到网络 | `docker network connect my_network my_container` |
| `docker network disconnect` | 断开容器与网络的连接 | `docker network disconnect my_network my_container` |
| `docker run --network` | 在特定网络下运行一个新的容器 | `docker run --network=my_network my_image` |
| `docker network prune` | 删除所有未使用的网络 | `docker network prune` |
| `docker service create` | 在特定网络下创建一个服务 | `docker service create --network my_network my_service` |

## 六、安全与维护指令
| 命令 | 描述 | 示例 |
| :--- | :--- | :--- |
| `docker logs` | 获取容器的日志 | `docker logs [CONTAINER_ID]` |
| `docker stats` | 显示一个或多个容器的实时资源使用情况 | `docker stats` |
| `docker system df` | 显示Docker磁盘使用情况 | `docker system df` |
| `docker system prune` | 清理未使用的资源 | `docker system prune` |
| `docker system events` | 实时获取Docker系统的事件 | `docker system events` |
| `docker system info` | 显示系统范围的信息 | `docker system info` |
| `docker inspect` | 显示容器或镜像的详细信息 | `docker inspect [CONTAINER_ID] or [IMAGE_ID]` |
| `docker history` | 显示镜像的历史信息 | `docker history [IMAGE_ID]` |
| `docker checkpoint create` | 创建容器的检查点 | `docker checkpoint create [CONTAINER_ID] checkpoint01` |
| `docker checkpoint ls` | 列出容器的检查点 | `docker checkpoint ls [CONTAINER_ID]` |
