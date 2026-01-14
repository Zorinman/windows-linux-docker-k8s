
⭐ **address，netmask，gateway的查看和之前一样不再赘述** 

# 1.确认你的网卡名称（通常是 ens33, eth0 等）：
```bash
ip addr
```
# 编辑networking的网络配置文件
```bash
sudo vim /etc/network/interfaces
```
![alt text](imgs/image-5.png)
**添加内容如下**
dns-nameservers和gateway相同即可
```
auto ens33
iface ens33 inet static
    address 192.168.219.128
    netmask 255.255.255.0
    gateway 192.168.219.2
    dns-nameservers 192.168.219.2

```
# 2.重启网络服务
```bash
sudo systemctl restart networking
```

# 2.如果重启网络服务失败则需要停止NetworkManager服务，释放网卡，重启虚拟机
```bash
sudo systemctl stop NetworkManager
sudo systemctl stop networking
sudo ip addr flush dev ens33
```
**重启虚拟机**

## **解释**：

### 1.冲突的根源：

- NetworkManager：这是 Debian 桌面版（带有 GNOME/KDE 等界面）默认的智能网络管理器。它倾向于自动接管所有网卡，并通过 DHCP 获取 IP。

- networking (ifupdown)：这是传统的通过 /etc/network/interfaces 文件配置网络的服务。

- 当你修改了 interfaces 文件想设静态 IP，但 NetworkManager 还在运行并控制着网卡时，networking 服务试图接管网卡就会失败（报错），因为设备被占用了。

**2.为什么以上操作会成功**

- 当你执行 stop NetworkManager 和 flush 时，你强制释放了网卡。

- 关键在于重启：当你重启 Debian 后，系统启动脚本会按照优先级运行。

- NetworkManager 有一个默认机制：它通常会自动忽略已经在 /etc/network/interfaces 中定义过的网卡。

- 所以，重启后，networking 服务率先读取配置文件并配置了静态 IP。随后 NetworkManager 启动，发现 ens33 已经在 interfaces 里写死了，它就会很“识趣”地把这个网卡标记为 Unmanaged（不托管），从而不再去干涉它。


# 3.验证配置是否生效
- 1.检查ens33显示的信息inet 192.168.219.129/24 brd 192.168.219.255 `scope global dynamic noprefixroute` ens33 变为`scope global`

```bash
ip addr show ens33
```
- 2.测试外网连通性
```bash
ping baidu.com
```
