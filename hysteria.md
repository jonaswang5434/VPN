又是一个 `404` 错误。这说明甬哥（yonggekkk）把他的 GitHub 仓库也做了一次大清理，或者把项目移到了其他地方（这是开源脚本作者经常干的事，经常突然更换路径）。

别慌，针对你“回国网络慢、极度看重性能（Hysteria 2）”的核心需求，我们完全不需要死磕这一个脚本。

目前业界最稳定、更新最频繁、绝不会 404 的 **Hysteria 2 官方一键脚本** 才是最好的选择。这个脚本直接由 Hysteria 官方/核心社区维护，纯 C/Go 底层架构，没有任何花里胡哨的封装，**性能和吞吐量反而是最高的**。

请改用以下官方首选方案：

---

## 🛠️ 1. 切换为 Hysteria 2 官方纯净版脚本

我们同样按照先下载、后执行的安全步骤：

**第一步：下载官方安装脚本**

```bash
wget -O hy2-install.sh https://get.hy2.io/

```

**第二步：检查脚本内容（可选）**

```bash
less hy2-install.sh

```

*(按 `q` 键退出)*

**第三步：赋予权限并执行安装**

```bash
chmod +x hy2-install.sh
sudo ./hy2-install.sh

```

*这个脚本非常快，它会自动检测你的服务器架构，下载官方最新的 Hysteria 2 二进制文件并注册为系统服务。*

---

## ⚙️ 2. 极简配置（针对回国加速优化）

官方脚本安装完成后，默认配置文件在 `/etc/hysteria/config.yaml`。为了让你能用 v2rayN 连上，并且发挥最大性能，我们直接来配置它。

运行以下命令编辑配置文件（如果你不习惯 nano，也可以用你熟悉的编辑器）：

```bash
sudo nano /etc/hysteria/config.yaml

```

将里面的内容清空，直接粘贴下面这套**专为高并发、弱网回国优化**的配置（你只需要修改里面的密码）：

```yaml
# /etc/hysteria/config.yaml
listen: :34567 # 服务端监听端口（记得去云厂商后台放行这个 UDP 端口）

# 使用自签证书（免去申请域名的麻烦，对弱网加速性能毫无影响）
tls:
  cert: /etc/hysteria/server.crt
  key: /etc/hysteria/server.key

auth:
  type: password
  password: your_secure_password_here # 👈 改成你自己的密码（给客户端连接用）

# 性能调优加速参数
quic:
  initStreamReceiveWindow: 8388608
  maxStreamReceiveWindow: 8388608
  initConnectionReceiveWindow: 16777216
  maxConnectionReceiveWindow: 16777216
  maxIdleTimeout: 30s

```

*(在 nano 中，粘贴后按 `Ctrl + O` 保存，按 `Enter` 确认，再按 `Ctrl + X` 退出)*

---

## 🔑 3. 生成自签证书并启动

因为配置里指定了自签证书路径，我们需要一行命令来生成它（有效期 10 年）：

```bash
sudo openssl req -x509 -nodes -days 3650 -newkey rsa:2048 -keyout /etc/hysteria/server.key -out /etc/hysteria/server.crt -subj "/CN=www.bing.com"

```

生成后，启动 Hysteria 2 服务并设置开机自启：

```bash
sudo chmod 644 /etc/hysteria/server.crt /etc/hysteria/server.key
sudo systemctl enable hysteria-server
sudo systemctl restart hysteria-server

```

检查一下状态，如果显示绿色的 `active (running)`，说明服务端已经完美运行：

```bash
sudo systemctl status hysteria-server

```

---

## 📲 4. v2rayN 客户端如何连接？

由于这是官方纯净版，不会主动吐出 `hysteria2://` 链接，你需要手动在 v2rayN 里添加：

1. 打开 **v2rayN** -> 点击顶部 **服务器** -> 选择 **添加 Hysteria2 服务器**。
2. 填入以下关键参数：
* **别名**：回国加速Hy2
* **地址 (Address)**：你的服务器公网 IP
* **端口 (Port)**：`34567`
* **密码 (Auth payload / Password)**：你在配置文件里写的那个密码


3. **关键性能设置：**
* **跳过证书验证 (AllowInsecure)**：必须勾选为 `true`（因为我们用的是自签证书）。
* **SNI**：填写 `www.bing.com`（与你刚才生成证书的域名一致）。
* **Up Mbps / Down Mbps**：手动填上你客户端本地宽带的数值（例如下行填 `200`，上行填 `30`），这一步是 Hysteria 2 跑满弱网带宽的底层核心。



点击确定，选择该节点，按下 `Ctrl + U` 测试真连接延迟。只要你服务器的 UDP 端口放行了，此时你就可以享受到极速的回国网络了。
