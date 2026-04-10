## sing-box tuic协议手动安装教程

TUIC 协议核心特性与优势：
极速连接： 依托 QUIC 协议，实现 1-RTT TCP 中继和 0-RTT UDP 中继，显著减少网络握手延迟。
网络加速： 内置用户空间拥塞控制，在任意平台可实现双向 BBR 加速，特别适合游戏场景，中外节点延迟可稳定在 120ms 以内。
FullCone NAT： 提供高可靠的 UDP 数据转发，使主机游戏跨平台联机成功率达到 95%。
高平滑切换： 切换 Wi-Fi 和移动数据时，连接不会像 TCP 一样断开，保证了用户体验的连续性。
多路复用： 客户端与服务器间仅需一条 QUIC 连接，所有任务在该连接的“流”中传输，降低连接开销。
安全性： 所有数据均通过 TLS 加密。

开始搞事～打开你的vps！

官方一键

```bash
curl -fsSL https://sing-box.app/install.sh | sh 
```

手动按照

```bash
wget https://sourceforge.net/projects/sing-box.mirror/files/v1.12.20/sing-box-1.12.20-linux-amd64.tar.gz
tar -xzf sing-box-1.12.20-linux-amd64.tar.gz
mv sing-box-1.12.20-linux-amd64/sing-box /usr/local/bin/
chmod +x /usr/local/bin/sing-box
```

查看版本

```bash
sing-box version
```

创建配置目录

```bash
mkdir -p /etc/sing-box
mkdir -p /var/log/sing-box
```

建立证书

```bash
cd /etc/sing-box
openssl genrsa -out tls.key 2048

openssl req -new -x509 -key tls.key -out tls.cer -days 3650
```

看看有没有文件

```bash
ls -l /etc/sing-box
```

测试运行

```bash
sing-box run -c /etc/sing-box/config.json
```

创建开机启动

```bash
vim /etc/systemd/system/sing-box.service
```

```txt
[Unit]
Description=sing-box Service
Documentation=https://sing-box.sagernet.org/
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
ExecStart=/usr/bin/sing-box run -c /etc/sing-box/config.json
Restart=on-failure
RestartSec=5
LimitNOFILE=512000
User=root

[Install]
WantedBy=multi-user.target
```

相关命令

```bash
systemctl daemon-reload

systemctl enable sing-box
systemctl start sing-box
systemctl restart sing-box
systemctl status sing-box
sing-box generate reality-keypair #生成Reality 密钥对
```

卸载sing-box

```bash
systemctl stop sing-box

systemctl disable sing-box

rm -f /etc/systemd/system/sing-box.service

systemctl daemon-reload

rm -f /usr/local/bin/sing-box

rm -rf /etc/sing-box

rm -f /var/log/sing-box/access.log
```

客户端配置：
```bash
proxies:
  - name: 名称
    type: tuic
    server: 你的ip
    port: 端口
    uuid: 3a8b6e09-cea9-4267-a8e2-a2d956a6e78e
    password: 3a8b6e09-cea9-4267-a8e2-a2d956a6e78e
    alpn:
      - h3
    congestion-controller: bbr
    skip-cert-verify: true
    disable-sni: false
```



## VLESS + Reality 最牛的加密模式

VLESS + Reality 是一种顶级的代理技术组合，通过无需域名和证书的 REALITY 协议配合无状态的 VLESS，实现了极致的隐蔽性和高效能。其核心特征包括：TLS 指纹掩护（无真实证书）、极致的流量伪装、无需域名与服务器配置，以及结合 XTLS Vision 实现的低延迟、高性能传输。 

生成Reality 密钥对，会有PrivateKey和PublicKey，PrivateKey用于服务器配置，PublicKey用于客户端配置，一定要生成的配对！

```bash
sing-box generate reality-keypair
```

服务器config.json的配置
```
{
  "log": { "level": "info", "timestamp": true },
  "inbounds": [
    {
      "type": "vless",
      "listen": "::",
      "listen_port": 54321,
      "tag": "vless-reality",
      "users": [
        {
          "name": "longlikun",
          "uuid": "cc11afe7-4af8-4bb1-a99d-ee51cc34e6c9"
        }
      ],
      "tls": {
        "enabled": true,
        "server_name": "www.bing.com",
        "reality": {
          "enabled": true,
          "handshake": {
            "server": "www.bing.com",
            "server_port": 443
          },
          "private_key": "mOKbAI-_VhFFx9XD9qvRlfU5exJmJ2yRcj8s_OZb838",
          "short_id": ["0123456789abcdef"]
        }
      }
    }
  ],
  "outbounds": [
    {
      "type": "direct",
      "tag": "direct"
    }
  ],
  "route": {
    "rules": [
      { "inbound": ["vless"], "outbound": "direct" }
    ]
  }
}
```
客户端配置
```bash
  - name: 名称
    type: vless
    server: 你的vps的ip
    port: 服务器配置端口
    uuid: 3a8b6e09-cea9-4267-a8e2-a2d956a6e78e
    network: tcp
    udp: true
    tls: true
    flow: xtls-rprx-vision
    servername: www.bing.com                
    reality-opts:
      public-key: _DdkyDiYY0vAZFU_bso4ssx2zXCFL7Aa2grwPyvmizE
      short-id: 0123456789abcdef                
    client-fingerprint: firefox
```
