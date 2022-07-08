---
title: 安装frp
date: 2022-07-08 16:41:02
tag: 
- vps
- 内网穿透
---
可以参考 [GitHub](https://github.com/fatedier/frp).

## 一、部署服务端

### 1、以x86`centos7`为例，前往`github`下载发布的安装包

``` bash
mkdir -p /data/soft
wget https://github.com/fatedier/frp/releases/download/v0.43.0/frp_0.43.0_linux_amd64.tar.gz
tar -xzvf frp_0.43.0_linux_amd64.tar.gz
mv frp_0.43.0_linux_amd64 frps
cd frps
```

### 2、修改配置文件`frps.ini`，编辑输入如下内容

``` bash
# 基础通用配置
[common]
# 用于与客户端建立通信连接的端口，一般使用bind_port即可，需开放防火墙（同时开通tcp、udp）
bind_addr = 0.0.0.0
bind_port = 17000
bind_udp_port = 17001
kcp_bind_port = 17000

# 外网访问使用的端口，使用时需要开放防火墙
vhost_http_port = 10080
vhost_https_port = 10443

# frps面板功能的配置（可忽略）
dashboard_addr = 0.0.0.0
dashboard_port = 17500
dashboard_user = admin
dashboard_pwd = password
enable_prometheus = true

# 日志输出配置，默认即可
log_file = ./frps.log
log_level = info
log_max_days = 3
disable_log_color = false
detailed_errors_to_client = true

# 身份验证配置，默认即可
authentication_method = token
authenticate_heartbeats = false
authenticate_new_work_conns = false

# 身份验证秘钥，需自定义及保密
token = 12345678

# 默认即可
oidc_issuer =
oidc_audience =
oidc_skip_expiry_check = false
oidc_skip_issuer_check = false
tls_only = false

# 泛域名后缀部分
subdomain_host = v1.100039.xyz
udp_packet_size = 1500
pprof_enable = false
```

### 3、开放防火墙端口

``` bash
firewall-cmd --zone=public --add-port=17000/tcp --permanent
firewall-cmd --zone=public --add-port=17000/udp --permanent
firewall-cmd --zone=public --add-port=10080/tcp --permanent
firewall-cmd --zone=public --add-port=10443/tcp --permanent
systemctl reload firewalld
firewall-cmd --list-ports
```
### 4、编写启动脚本`run.sh`
``` bash
#!/bin/bash
rm -f frps.log
nohup ./frps -c frps.ini &
```
### 5、编写关闭脚本`stop.sh`

``` bash
#/bin/bash
ps -ef|grep frps |grep -v grep |awk {'print $2'} |xargs kill -9
```
---
## 二、部署客户端

### 1、以64位`windows10`为例，前往`github`下载发布的安装包

``` bash
下载解压：https://github.com/fatedier/frp/releases/download/v0.43.0/frp_0.43.0_windows_amd64.zip
```

### 2、修改配置文件`frpc.ini`，编辑输入如下内容
``` bash
# 基础通用配置
[common]
tls_enable = true
server_addr = www.v1.100039.xyz
server_port = 17000
token = 12345678

# tcp穿透配置，可实现远程桌面，使用指定的remote_port访问
[desktop]
type = tcp
local_ip = 127.0.0.1
local_port = 3389
remote_port = 13389

# http穿透配置，使用frps.ini上配置的10080端口访问，域名为business.v1.100039.xyz
[business-dev]
type = http
local_ip = 127.0.0.1
local_port = 8080
subdomain = business

# https穿透配置，使用frps.ini上配置的10443端口访问，域名为rancher.v1.100039.xyz
[rancher-dev]
type = https
local_ip = 192.168.180.80
local_port = 443
subdomain = rancher
```
### 3、编写启动脚本`run.bat`
``` bash
frpc.exe
pause
```
