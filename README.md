# uuplugin docker 版本

UU 加速提供了 OpenWrt 版本的插件，见 https://router.uu.163.com/app/baike/public/5f963c9304c215e129ca40e8.html。
该项目基于 OpenWrt 的 openwrtorg/rootfs 版本构完成了开启 UU 需要的配置，同时移除了一些无关服务，如 DHCP、SSH、Web luci。
你可以使用该镜像完成旁路由模式下 UU 加速服务快速部署。

因为不同的 OpenWrt 版本对路由规则配置不同（或者别的什么），导致检测不到游戏主机连入。主要表现为主机在 UU app 中出现后立即消失。
使用该 docker 镜像应当可以有效解决该问题。

# 通过Action编译
## 你只需要在 GitHub 仓库配置两个环境变量：

1. DOCKERHUB_USERNAME: 你的 Docker Hub 用户名
2. DOCKERHUB_TOKEN: 你的 Docker Hub 访问令牌 (Access Token)
## 获取 Docker Hub Token 的步骤：

1. 登录你的 Docker Hub 账号
2. 点击右上角的头像 -> Account Settings
3. 左侧菜单选择 Security
4. 点击 "New Access Token"
5. 给 token 起个名字（比如 "GitHub Actions"）
6. 选择权限范围（至少需要 "Read, Write, Delete"）
7. 点击生成，复制生成的 token
## 在 GitHub 仓库添加这些环境变量的步骤：

1. 打开你的 GitHub 仓库
2. 点击 "Settings" 标签
3. 左侧菜单找到 "Secrets and variables" -> "Actions"
4. 点击 "New repository secret"
5. 分别添加：
  Name: DOCKERHUB_USERNAME, Value: 你的 Docker Hub 用户名
  Name: DOCKERHUB_TOKEN, Value: 刚才生成的 Docker Hub token

## 环境准备

### 打开 IP 混杂模式

> 因为我没有多网口设备，如何在多网口设备下为容器提供独立 IP 你需要自行查询 Docker 相关文档。

如果你的物理设备为单网口，需要开启 IP 混杂模式，为容器提供独立 IP。

如下命令为开启方式：

```
ip link set eth0 promisc on
```

`eth0` 网卡根据实际情况选择，群晖系统命令应当如下：

```
ip link set ovs_eth0 promisc on
```

你可以通过 `ip addr` 判断该输入什么。

### 创建桥接网络 macvlan

命令示例如下，`subnet`、`gateway` 和 `parent` 应当根据实际情况选择：

```
docker network create -d macvlan \
--subnet=192.168.18.0/24 \
--gateway=192.168.18.1 \
-o parent=ovs_eth0 \
bridge-host
```

## 启动容器

容器可配置参数如下，一般情况只需要配置 `UU_LAN_IPADDR` 和 `UU_LAN_GATEWAY`。

```
ENV UU_LAN_IPADDR=
ENV UU_LAN_GATEWAY=
ENV UU_LAN_NETMASK="255.255.255.0"
ENV UU_LAN_DNS="119.29.29.29"
```

启动命令示例如下：

```
docker run -d --name uuplugin \
--network bridge-host \
--privileged \
-e UU_LAN_IPADDR=192.168.18.77 \
-e UU_LAN_GATEWAY=192.168.18.1 \
dianqk/uuplugin
```

- `UU_LAN_IPADDR` 为该容器使用的 IP，也是游戏主机的网关
- `UU_LAN_GATEWAY` 为该容器的上级网关

## 绑定 UU 服务

UU 主机加速 app 会检测手机网关进行通信判断是否安装的 UU 插件，所以你需要**将手机网关和 DNS 指向刚刚创建的容器使用的 IP**。
打开 app 点击安装路由器插件绑定即可。
此后为了避免影响手机上网，绑定完毕后，手机的网关和 DNS 可以改回原来的设定。

## 为游戏主机加速

将需要加速的游戏主机的网关和 DNS 设置为容器的 IP 后，打开 app 即可看到设备出现，点击加速即可。

## 补充

我的设备场景群晖连接主路由，小米 AX6000 AP 模式连接主路由，一切正常使用。
但将群晖连接小米 AX6000 时出现，旁路由网关出现连接不通问题，原因不明，还请网管大佬指教。
