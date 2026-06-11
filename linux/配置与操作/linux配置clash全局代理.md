### 需要自己有订阅的clash

⭐使用时要保证clash verge关闭，clash老版本打开。
更新新的config.yaml文件后，只需要移动到把新配置文件与移动到/root/clash
### 配置clash全局代理之后 docker和k8s可以直接拉取镜像 非常方便（k8s需要每个节点都配置）  
参考文章  
https://www.fuxi.info/archives/273  
https://robinxb.com/posts/2023/clash-on-linux/  
以下linux文件的传输均使用xtfp
### 1.下载clash
https://github.com/zhongfly/Clash-premium-backup/releases/download/2023-09-05-gdcc8d87/clash-linux-amd64-n2023-09-05-gdcc8d87.gz  
将其放入linux并解压  
`gzip -d clash-linux-amd64-n2023-09-05-gdcc8d87.gz`  
方便使用 将解压后文件重命名为clash   
`mv clash-linux-amd64-n2023-09-05-gdcc8d87 clash`  
赋予可执行权限  
`chmod +x clash`
将 clash 移动到系统的可执行路径 （如 /usr/local/bin）   
`sudo mv clash /usr/local/bin/`  
运行以下命令验证是否安装成功，如果显示版本信息，说明安装成功：  
`clash -v`
### 2.创建config.yaml文件
config.yaml为clash的代理规则和clash的一些其他设置。代理规则不需要我们自己编写，通过订阅地址直接下载即可  
- 1.在自己的calsh里根据订阅地址直接浏览器打开下载yaml文件  
- 2.直接在linux 使用wget+上图的订阅地址下载
- 
![image](https://github.com/user-attachments/assets/ee419d71-9953-4b68-af0d-55b7e79a349a)
![image](https://github.com/user-attachments/assets/9d60a4a9-f0a7-4af9-b279-1f70a5203d19)

将下载下来的yaml文件改名为config.yaml放入linux中   
`mv xxxxx config.yaml`  

在config.yaml的目录下运行该文件   
`clash -f config.yaml`  
此时会提示[MMDB] can't find DB, start download path=/root/.config/clash/Country.mmdb  
如果网络好可以直接下载下来并成功运行，如何下载不下来就直接Ctrl+c取消运行，先按照第3步去安装MMDB  

### 3.下载MMDB
Country.mmdb为全球IP库，可以实现各个国家的IP信息解析和地理定位，没有这个文件clash是无法运行的  
https://cdn.jsdelivr.net/gh/Dreamacro/maxmind-geoip@release/Country.mmdb  
将MMDB到系统提示的目录下  
`mv Country.mmdb /root/.config/clash/`   ⭐注意这里移动得目标位置请以之前`can't find DB, start download path=`的提示为准  
（.config是一个隐藏文件夹，`ls -a`可以查看，里面的clash文件夹最开始没有，需要手动去创建或者先执行config.yaml提示缺失MMDB之后会自动创建一个)
### 4.设置全局代理
设置全局代理需要在`/etc/profile`文件中设置  /etc/profile 是 Linux 系统的全局环境配置文件，它用于设置系统范围的环境变量。  
 
127.0.0.1:7890为本地代理地址  7890为clash的监听端口
![image](https://github.com/user-attachments/assets/ce917ef6-e548-4ac8-96fb-cae02faf5a2e)  
clash代理是使用使用 HTTP 协议配置的 因此 客户端(安装clash的linux虚拟机）和代理服务器（clash的订阅地址）使用的是HTTP协议通信  
因此添加代理配置，`http://127.0.0.1:7890`


```
$ vim /etc/profile
###在文件最后添加以下内容
export http_proxy=http://127.0.0.1:7890
export https_proxy=http://127.0.0.1:7890
保存退出
#使配置文件生效
$ source /etc/profile

```
⭐**如果希望把上面的内容清除关闭代理，那么执行`source /etc/profile`后你需要打开一个新的窗口才会生效**

转发流程：客户端收到http或https协议的请求，通过http协议将请求流量发送到clash代理服务器，代理服务器通常会将此请求转发给目标服务器，并且与外部服务器的通信协议根据客户端收到的请求的协议决定，代理服务器接收结果后再次通过HTTP协议返回结果给客户端  

实例：
linux虚拟机上输入命令:`curl www.gooogle.com` ，此HTTPS流量通过HTTP协议发送给代理服务器，代理服务器通过HTTPS协议将HTTPS流量发送给目标google服务器，代理服务器收到返回的结果后再将结果通过HTTP协议返回给linux虚拟机。  

--------
客户端和代理服务器之间的通信：使用 HTTP 协议（因为配置的是 `http://127.0.0.1:7890`）。

代理服务器和外部服务器之间的通信：使用 HTTPS 协议（因为请求流量目标是一个 HTTPS 网站，代理服务器会以 HTTPS 协议与外部服务器建立连接）。

文件中添加代理配置，`http://127.0.0.1:7890`,表示无论是http还是https请求都会通过HTTP协议将流量转到本地代理`127.0.0.1:7890`

### 5.设置clash在后台运行  
在config.yaml文件目录下临时直接运行`clash -f config.yaml`
临时运行clash的话，一旦ctrl+c关闭了终端，那么clash也会一并关闭。所以我们需要设置一下后台运行clash  
在/etc/systemd/system/目录新建一个clash.service文件  
⭐ 注意`ExecStart=/usr/local/bin/clash -d /root/clash` 中的/root/clash是你移动进来的config.yaml文件的所在位置，请以你实际放置的位置为准.  也可以统一放到/root/clash
```
$ vim /etc/systemd/system/clash.service
#添加以下内容（记得把#之后的注释全部删除）：
[Unit]
Description=Clash service
After=network.target

[Service]
Type=simple
User=root   #你的用户名，默认为root 可通过命令whoami查看
ExecStart=/usr/local/bin/clash -d /root/clash #/usr/local/bin/clashclash执行文件的绝对路径  /root/clash是config.yaml配置文件的所在目录 
Restart=on-failure
RestartPreventExitStatus=23

[Install]
WantedBy=multi-user.target
```
重新加载守护进程  
`systemctl daemon-reload`   
启动clash  
`systemctl start clash `  
检查运行状态  
`systemctl status clash`
设置开机自启clash服务  
`systemctl enable clash `  

查看运行日志  
`journalctl -u clash -f`

注意通过status查看内存大小，防止内存泄露，记得定时重启clash服务使内存清零
### systemctl常用命令
```
systemctl status clash 查看clash服务
systemctl start clash 启动clash服务
systemctl stop clash 停止clash服务
systemctl restart clash 重启clash服务
systemctl enable clash 设置开机自启clash服务
systemctl daemon-reload 如果修改了clash.service文件，需要此命令来重载被修改的服务文件
```

⭐⭐通常不用单独配置 Docker和container 即可直接拉取镜像 如果拉取不了可以单独配置一下（查看K8S的容器运行时，再为对于运行时配置）
---------------------------------------------------------------
**Docker 需要单独设置代理，因为 Docker 运行在自己的守护进程（daemon）中，它不会自动继承 /etc/profile 或 ~/.bashrc 里的环境变量  
containerd 作为 Docker 的底层运行时，不会自动继承 /etc/profile 的代理设置。**  

### 为 Docker 设置代理（Docker作为容器运行时）
在 /etc/systemd/system/docker.service.d/ 目录下创建代理配置  
`sudo mkdir -p /etc/systemd/system/docker.service.d`
创建一个 http-proxy.conf 文件  
`sudo vim /etc/systemd/system/docker.service.d/http-proxy.conf`  
添加以下内容  
```
[Service]
Environment="HTTP_PROXY=http://127.0.0.1:7890"
Environment="HTTPS_PROXY=http://127.0.0.1:7890"
Environment="NO_PROXY=localhost,127.0.0.1"
```
NO_PROXY 用于指定不走代理的地址，比如 localhost 和 127.0.0.1，防止本地通信受影响。  

重新加载配置并重启 Docker  
`sudo systemctl daemon-reload`
`sudo systemctl restart docker`

### 为containerd 设置代理（Containerd作为容器运行时）
创建 containerd 代理配置  
`sudo mkdir -p /etc/systemd/system/containerd.service.d`
创建一个 http-proxy.conf 文件   
`sudo vim /etc/systemd/system/containerd.service.d/http-proxy.conf` 
添加以下内容  
```
[Service]
Environment="HTTP_PROXY=http://127.0.0.1:7890"
Environment="HTTPS_PROXY=http://127.0.0.1:7890"
Environment="NO_PROXY=localhost,127.0.0.1"
```
⭐**注意这里只是添加了clash的配置，但是为了让 Containerd 拉取外部镜像（走代理），又不影响集群内部通信（直连）。我们需要根据之前 kubeadm init的参数实际修正，加入加入了你的k8s内部物理网段、Service网段、Pod网段和集群后缀** 详细请参考k8s参考的部署文档:`安装Debian 13.3并 部署k8s v1.34`

重新加载配置并重启 containerd  
`sudo systemctl daemon-reload`
`sudo systemctl restart containerd`



