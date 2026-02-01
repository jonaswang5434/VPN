# 使用shadowsocks 替代2ray 方案：
## server端和客户端都需要以相同的方式安装 shadowsocks：
### 1、安装shadowsocks ，这时会同时安装server 和client
sudo apt update   <br>
sudo apt upgrade <br>
sudo apt install shadowsocks-libev <br>
验证是否已安装成功: <br>
which ss-local   #验证客户端  <br>
which ss-server   # 验证server端 <br>

### 2、配置文件：
sudo nano  /etc/shadowsocks-libev/config.json  <br>
server端配置内容如下： <br>
```
{
    "server":"0.0.0.0",    # 监听所有请求的ip地址  
    "mode":"tcp_and_udp",   #加密模式 
    "server_port":8388,     #设置server监听端口，要确保防火墙已开通该端口  
    "password":"yKxfj4GHjxac",   # 设置加密密码  
    "timeout":86400,  <br>
    "method":"chacha20-ietf-poly1305"  # 固定设置  
}
```
### 3.启动服务
临时启动： <br>
ss-server -c /etc/shadowsocks-libev/config.json  <br>
以守护模式启动ss-server <br>
sudo systemctl enable shadowsocks-libev <br>
sudo systemctl start shadowsocks-libev <br>
## 客户端配置：
sudo nano /etc/shadowsocks-libev/config.json 
```
{
    "server":"170.106.188.73",    # server ip 地址
    "local_address":"127.0.0.1",   # 监听的本地ip 地址
    "server_port":8388,             # server 设置的端口
    "local_port":1080,             # 本地监听的端口
    "password":"yKxfj4GHjxac",     # 要和server 端的password一致
    "timeout":86400,
    "method":"chacha20-ietf-poly1305"  # 固定设置加密算法
}
```
### 客户端启动：
临时启动： <br>
ss-local -c /etc/shadowsocks-libev/config.json  <br>
守护模式启动：
```
sudo systemctl enable shadowsocks-libev-local@config.service 
sudo systemctl start shadowsocks-libev-local@config.service
```
测试是否成功：<br>
curl --socks5-hostname 127.0.0.1:1080 http://www.google.com <br>
浏览器配置：<br>
firefox: <br>
settings- 滚动到最后，network settings, 点击settings ,将socks host 设置为127.0.0.1,port:1080,选择 socks_v5 及 proxy DNS When using socks v5 <br>
- 切忌不要对http proxy, https_proxy 进行设置，该部分内容必须设置为空。<br>
google浏览器：<br>
同样的设置，不能对http proxy, https_proxy  进行设置。<br>
如果使用插件：SwitchyOmega <br>
- 那么所有的https,http,都需要设置为socks5 才行。<br>
- 最后点击apply
# Docker 内配置代理：
### step1: 确认local machine ip 地址
- query localhost machine ip: `ip addr` <br>
通常默认minikube 的ip 地址是192.168.49.1:， 不确定可以让ai 确认
- 确保shadowsocks 配置的local ip 监听地址是0.0.0.0,监听所有请求（包含docker内)所有请求
### step2: config proxy and test 
```
minikube ssh
sudo mkdir -p /etc/systemd/system/docker.service.d

sudo tee /etc/systemd/system/docker.service.d/http-proxy.conf <<EOF
[Service]
Environment="HTTP_PROXY=socks5h://192.168.49.1:1080"
Environment="HTTPS_PROXY=socks5h://192.168.49.1:1080"
Environment="NO_PROXY=localhost,127.0.0.1,192.168.49.1"
EOF
# restart docker ,make it effective
sudo systemctl daemon-reload
sudo systemctl restart docker

# 测试 DNS 和连接
curl -I https://registry.k8s.io
```

