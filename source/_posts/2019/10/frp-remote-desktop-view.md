---
title: 小米路由器3配置外网开机和远程桌面
date: 2019-10-12 10:00
tags:
- security
- server
- linux
- mi router
categories: linux
---

之前一直用TeamViewer作为远程软件，有一段时间帐号被识别为企业帐号连上之后频繁断开，花了点时间在小米路由器3和VPS上配置FRP，实现远程开机和远程桌面。

### 设备配置
1.可公网访问的VPS一台
2.小米路由器3一个
3.可远程唤醒的主机一台

### 原理说明
1.远程开机
大部分Intel的网卡支持网络唤醒，通过特定端口发送UDP数据包广播，即可唤醒指定MAC地址的电脑（需要修改主板和网卡配置）。

2.FRP
FRP [https://github.com/fatedier/frp](https://github.com/fatedier/frp) 是一款内网穿透软件，可将本地指定端口代理到远程服务器上，供外网连接。

3.部署方式
在远程服务器上部署frp server，在小米路由器3上部署frp client。在要发起远程的机器上安装frp client。

### 服务器部署
1.下载frp [https://github.com/fatedier/frp/releases](https://github.com/fatedier/frp/releases) 我服务器是CentOS 7,下载linux_amd64的包(32位系统为linux_386)。


2.解压并拷贝文件到指定目录，路径关系如下：
```
frps -> /usr/local/bin/frps
frps.ini -> /etc/frp/frps.ini
systemd/frps.service -> /lib/systemd/system/frps.service
```

3.修改配置
3.1修改 /lib/systemd/system/frps.service 文件，将 frps 的路径修改为 /usr/local/bin/frps。
修改后的配置文件如下：
```
[Unit]
Description=Frp Server Service
After=network.target

[Service]
Type=simple
User=nobody
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/frps -c /etc/frp/frps.ini

[Install]
WantedBy=multi-user.target
```

3.2修改 /etc/frp/frps.ini 文件，配置端口和token，可参考 [https://github.com/fatedier/frp/blob/master/README_zh.md](https://github.com/fatedier/frp/blob/master/README_zh.md)。我配置了 bind_port、bind_udp_port、tcp_mux、token几个字段。
修改配置如下：
```
[common]
bind_port = 2345
bind_udp_port = 2346
tcp_mux = true
token = abcd123
```
说明：
- 2345、2346为服务所使用的端口，可以任意配置，请保证防火墙对以上两个端口保持开放。
- token为客户端连接令牌，需和服务端保持一致。

4.启动服务
执行命令 `sudo systemctl start frps.service`启动服务
可通过 `sudo systemctl start frpc.service` 查看服务状态

### 小米路由器部署
1.小米路由器3先获取ssh root权限，具体操作网上搜。

2.在自己电脑上下载frpc linux mips客户端（路由器配置太低，操作耗时）。

3.解压文件，通过本地编辑器修改frpc.ini，分别配置ssh服务，远程唤醒服务，远程开机服务。
配置示例：
```
[common]
server_addr = <服务端ip或域名>
server_port = 2345
tcp_mux = true
token = abcd123
log_file = ./frpc.log

[ssh]
type = stcp
sk = aaa123
local_ip = 127.0.0.1
local_port = 22

[wol]
type = udp
local_ip = 192.168.31.255
local_port = 9
remote_port = 2347

[rdesk]
type = stcp
sk = aaa123
local_ip = <要远程的机器本地ip地址>
local_port = 3389
```

说明：
- common 配置服务连接模块
- ssh 将路由器的ssh暴露出来，方便远程操作路由器，sk为约定的密钥，连接的机器需要配置sk来保证配对。
- wol 将广播地址的UDP 9端口映射到VPS的2347端口。通过向VPS的2347端口发送UDP请求，唤醒局域网机器(VPS上2347端口要可访问)。local_ip为局域网广播地址，小米路由器默认为 192.168.31网段，如果有修改需要改到对应的地址。
- rdesk 将本地电脑的 3389 端口映射成服务。

4.创建服务文件frpc.d

内容如下：

```sh
#!/bin/sh /etc/rc.common
#
# frps:    FRP-Server Daemon
#
# description:  FRP-Server Daemon

START=99
EXTRA_COMMANDS="status"
EXTRA_HELP="        status  Check frp client service status"

PID_FILE=/userdisk/frp/frpc.pid
CONFIG_FILE=/userdisk/frp/frpc.ini
FRP_SERVER=/userdisk/frp/frpc

start()
{
    if [ ! -f $PID_FILE ]; then
        echo -n $"Starting FRP server..."
        nohup $FRP_SERVER -c $CONFIG_FILE < /dev/null > /dev/null 2> /dev/null &
        echo $! > $PID_FILE
        echo ""
    else
        PID=$(cat $PID_FILE)
        if [ ! -f /proc/$PID/cmdline ]; then

            echo -n $"Starting FRP server..."
            nohup $FRP_SERVER -c $CONFIG_FILE < /dev/null > /dev/null 2> /dev/null &
            echo $! > $PID_FILE
            echo ""
        else
            echo "FRP server is already running..."
        fi
    fi;
}
stop()
{
    if [[ -f $PID_FILE ]]; then
        echo -n $"Shutting down FRP server..."
        kill -9 $(cat $PID_FILE)
        rm -f $PID_FILE
        echo ""
    else
        echo "FRP server is not running..."
    fi;
}
status()
{
    if [ -f $PID_FILE ]; then
        PID=$(cat $PID_FILE)
        if [ -f /proc/$PID/cmdline ]; then
            echo "FRP server is running..."
        else
            echo "FRP server is not running..."
            rm -f $PID_FILE
        fi
    else
        echo "FRP server is not running..."
    fi;
}
```

5.将文件同步到小米路由器 /userdisk/frp 目录

```
frpc -> /userdisk/frp/frpc
frpc.ini -> /userdisk/frp/frpc.ini
frpc.d -> /userdisk/frp/frpc.d
```

*下面的操作在路由器上进行*
6.将 /userdisk/frp/frpc.d 通过软链接创建到 /etc/init.d/frpc.d。

```bash
ln -s /userdisk/frp/frpc.d /etc/init.d/frpc
```

7.启动服务

```
# 开启服务
/etc/init.d/frpc start
# 开机启动
/etc/init.d/frpc enable
```

通过 /etc/init.d/frpc status 查看状态。

### 访客端配置FRP

为了安全起见，ssh和rdesk配置的是stcp模式，需要本地机器上配置客户端。将服务映射成本地端口，然后使用。

1.安装FRP
mac电脑直接 brew install frpc

2.修改配置文件
mac电脑配置文件在 /usr/local/etc/frp/frpc.ini 
示例配置：
```
[common]
server_addr = <服务端ip或域名>
server_port = 2345
tcp_mux = true
token = abcd123
admin_addr = 127.0.0.1
admin_port = 7400
admin_user = admin
admin_pwd = admin123

[ssh_visitor]
type = stcp
role = visitor
server_name = ssh
sk = aaa123
bind_addr = 127.0.0.1
bind_port = 1122

[rdesk_visitor]
type = stcp
role = visitor
server_name = rdesk
sk = aaa123
bind_addr = 127.0.0.1
bind_port = 3389
```

说明：
- common字段配置了admin相关配置，可以通过web界面查看服务状态
- ssh_visitor将路由器上的ssh服务配置到本地的1122端口，使用ssh 127.0.0.1 -p 1122来访问
- rdesk_visitor将路由器上的rdesk服务配置到本地的1122端口，使用远程桌面工具连接127.0.0.1:3389访问

3.启动服务 `brew service start frpc`使用远程桌面尝试连接

### 配置远程唤醒

远程唤醒需要机器的网卡支持，如果不支持则无法使用远程唤醒功能，只能在机器开启的情况下使用远程桌面。

1.主板开启远程唤醒
在主板选项中的到 Wake On Lan 或者 WOL 选项并开启，各个电脑主板配置不同，可能需要修改多处。

2.在电脑网卡属性里面开启 魔包唤醒（Magic Packet）相关设置

![](/img/201910/2019-10-12_01.png)

3.测试关机唤醒
在电脑关机的情况下，手机下载一个 Wake On Lan的app，手机和要唤醒的电脑处在同一局域网。配置Mac地址和广播地址，然后尝试是否能唤醒电脑。如果本地局域网能唤醒，则说明配置成功。

![](/img/201910/2019-10-12_02.png)

4.远程唤醒
将ip地址改为VPS的ip地址，端口改为 2347，测试远程唤醒，正常情况下机器应该被唤醒。

### 链接
- frp [https://github.com/fatedier/frp](https://github.com/fatedier/frp)
- 小米路由器ssh [https://www.cnblogs.com/chenpingzhao/p/10935803.html](https://www.cnblogs.com/chenpingzhao/p/10935803.html)
- 设置网络唤醒电脑 [https://blog.csdn.net/xinluke/article/details/51169393](https://blog.csdn.net/xinluke/article/details/51169393)