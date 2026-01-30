# 使用shadowsocks 替代2ray 方案：
## server 端配置：
### 1、安装shadowsocks ，这时会同时安装server 和client <br>
sudo apt update   <br>
sudo apt upgrade <br>
sudo apt install shadowsocks-libev <br>
验证是否已安装成功: <br>
which ss-local   #验证客户端  <br>
which ss-server   # 验证server端 <br>

### 2、配置文件：
sudo nano  /etc/shadowsocks-libev/config.json  <br>
server端配置内容如下： <br>
{
    "server":"0.0.0.0",    # 监听所有请求的ip地址  <br>
    "mode":"tcp_and_udp",   #加密模式  <br>
    "server_port":8388,     #设置server监听端口，要确保防火墙已开通该端口  <br>
    "password":"yKxfj4GHjxac",   # 设置加密密码  <br>
    "timeout":86400,  <br>
    "method":"chacha20-ietf-poly1305"  # 固定设置  <br>
}
### 3.启动服务
临时启动： <br>
ss-server -c /etc/shadowsocks-libev/config.json  <br>
以守护模式启动ss-server <br>
sudo systemctl enable shadowsocks-libev <br>
sudo systemctl start shadowsocks-libev <br>
## 客户端配置：
