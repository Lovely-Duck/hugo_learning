+++
title = 'Hysteria2的安装使用'
date = 2024-10-28T14:08:16+08:00
draft = false
enableToc = true
+++

### 安装Hysteria2服务端
使用官方服务端端安装脚本
```bash
bash <(curl -fsSL https://get.hy2.sh/)
```
### 生成自签证书
```bash
openssl req -x509 -nodes -newkey ec:<(openssl ecparam -name prime256v1) -keyout /etc/hysteria/server.key -out /etc/hysteria/server.crt -subj "/CN=bing.com" -days 36500 && sudo chown hysteria /etc/hysteria/server.key && sudo chown hysteria /etc/hysteria/server.crt
```
### 服务端配置
编辑配置文件
```bash 
vim /etc/hysteria/config.yaml 
```
```bash
listen: :443
 
tls:
  cert: /etc/hysteria/server.crt
  key: /etc/hysteria/server.key
 
auth:
  type: password
  password: gRa6gZACqwQms6   # 请及时更改密码
 
masquerade:
  type: proxy
  proxy:
    url: https://bing.com # 伪装网站
    rewriteHost: true
```
服务管理
```bash
systemctl enable --now hysteria-server.service
systemctl restart hysteria-server.service
systemctl status hysteria-server.service
```
### 客户端配置
#### Linux
sing-box 配置文件
```bash
      {
          "type": "hysteria2",
          "up_mbps": 500, #按照实际带宽
          "down_mbps": 500, #按照实际带宽
          "server": "服务端IP或域名",
          "password":"gRa6gZACqwQms6",
          "tls": {
            "enabled": true,
            "server_name": "bing.com",
            "insecure": true
          },
          "server_port": 443,
          "tag": "xxx"
      },
```
#### clash-verge
我使用的是代理集合[proxy-providers](https://wiki.metacubex.one/config/proxy-providers/)，只需添加节点即可
```bash
- name: "xxx"
  type: hysteria2
  server: 服务端IP或域名
  port: 443
  #  up和down均不写或为0则使用BBR流控
  up: "500 Mbps" # 若不写单位，默认为 Mbps
  down: "500 Mbps" # 若不写单位，默认为 Mbps
  password: gRa6gZACqwQms6
  sni: bing.com
  skip-cert-verify: true
  ```
  