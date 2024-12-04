+++
title = 'Headscale食用教程'
date = 2024-10-14T09:42:16+08:00
draft = false
enableToc = true
+++
{{< lead >}}
将使用容器安装headscale
{{< /lead >}}
## 首先安装容器也就是docker
安装过程就不记录，[详细教程](https://docs.docker.com/engine/install/)在官网。
## 配置并运行
1. 准备一个目录用来存放headscale的文件，它的最终形态是这样的
```bash		
cd headscale
mkdir config
mkdir data
touch docker-compose.yml
		
root@VM-A2J416XGSG53:~/headscale# tree
.
├── config
├── data
└── docker-compose.yml
```
2. 下载配置示例并将其保存到**config**目录下
```bash
wget https://github.com/juanfont/headscale/blob/main/config-example.yaml -O config.yaml
```
- 修改`server_url`，改为公网IP或域名，国内服务器需要备案，推荐使用老外的服务器
- `prefixes`字段，将ipv6注释，自定义私有地址网段：
	```yaml
	refixes:
    #v6: fd7a:115c:a1e0::/48
    v4: 10.0.0.0/8
	```
- 修改`nameservers`，添加几个国内dns：
	```yaml
	nameservers:
    global:
      - 223.5.5.5
      - 119.29.29.29
      - 8.8.8.8
      - 2606:4700:4700::1111
      - 2606:4700:4700::1001
    ```	
3. 编辑 **docker-compose.yaml** 文件，启动服务并验证
  ```yml
  version: "3.6"
  services:
    headscale:
      image: ghcr.io/juanfont/headscale
      container_name: headscale
      volumes:
        - ./config:/etc/headscale/
        - ./data:/var/lib/headscale
      ports:
        - 8080:8080 
        - 9090:9090
      command: serve
      restart: unless-stopped
        #cap_add:
        # - CAP_NET_BIND_SERVIC
  ```
  ```bash
  docker compose up -d
  ```
浏览器输入`http://公网IP或域名:8080/windows`或`http://公网IP或域名:8080/apple`，有内容返回证明服务启动成功

### 创建用户
```bash
docker exec -it headscale headscale user create USERNAME
```
`USERNAME` 为你想要设置的用户名称
```bash
docker exec -it hedscale headscale user list
```
## 终端接入
|操作系统                   |Headscale支持               |
|:-------------------------:|---------------------------|
|MacOS                     |yes                        |
|Windows                     |yes                        |
|linux                     |yes                        |
|IOS                     |yes                        |
|Android                     |yes                        |

### Mac & iPhone & iPad
 - `Mac` 浏览器打开http://公网IP或域名:8080/apple，下载`macOS AppStore profile`文件并运行![headscale/apple](https://zone.fitch.cloud/sW9JQZ5xVUGq.png)
 打开系统设置>通用>设备管理  双击打开，安装![systemsettings](https://zone.fitch.cloud/g2VIzgQlH7Av.png)
 -  在app商店(非国区ID)安装Tailscale客户端并运行它,在状态栏上点击应用图标按下`command+,`
点击`Add Account...`![appsettings](https://zone.fitch.cloud/Y10JnVCwIPum.png)
会发现浏览器会打开一个验证页面![confirm](https://zone.fitch.cloud/4J6J9iE00ecI.png)
此时我们回到headscale服务端运行:
```bash
docker exec -it headscale headscale nodes register --user USERNAME --key nodekey:345c366fe6de46a8ae56ba6dfe706573b45017a8432f08c9ad0459c9adf7ca77
Machine ffzmacbookpro14 registered
```
提示`Machine XXX registered`为接入成功；注意`USERNAME`应为上面创建用户时的`用户`
- `iPhone`和`iPad`同理使用appstore下载tailscale
打开软件点击右上角![iosapp-1](https://zone.fitch.cloud/MoqrrQFjS8um.png)
login > Use a cuntom... > 输入`http://公网IP或域名:8080/`
![iosapp-2](https://zone.fitch.cloud/kMU86ps8afvo.png)
依旧会弹出验证页面然后回到headscale服务端运行即可
