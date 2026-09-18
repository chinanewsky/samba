# samba
docker samba 4.24.7 (20260918)

#### smb账号/密码：root/Aa.123456

============================================================

Samba 镜像通用部署模板

用法:
   1. 复制本文件为 docker-compose.yml
   2. 只改下面标注的两处 /mnt 改为需要共享的路径  SAMBA_PASS=Aa.123456 改为自已共享密码
   3. docker compose up -d
 访问:
   Windows 资源管理器输入  \\<服务器IP>   -> 能看到共享图标
   双击共享时按提示输入账号(SAMBA_USER)和密码(SAMBA_PASS)
说明:
   镜像已内置全部 Samba 配置(性能优化、匿名枚举共享列表等),
   即使 config 目录是空的, 容器首次启动也会自动生成 smb.conf, 无需手工准备。

============================================================

##### host网络模式
```
services:
  samba:
    image: ghcr.io/chinanewsky/samba:4.24.7
    container_name: samba
    hostname: samba
    restart: unless-stopped
    network_mode: host
    volumes:
      - /mnt:/share
      - ./config:/etc/samba
    environment:
	     # samba 账号 root 的密码 (容器内独立于宿主机, entrypoint 每次启动按此值重设)
      - SAMBA_PASS=Aa.123456
```


##### 任意网卡均可访问
```
services:
  samba:
    image: ghcr.io/chinanewsky/samba:4.24.7
    container_name: samba
    hostname: samba
    restart: unless-stopped
    ports:
      # 去掉前面IP，监听 0.0.0.0，本机所有网卡都可以访问
      - "137:137/udp"
      - "138:138/udp"
      - "139:139"
      - "445:445"
    volumes:
      - /mnt:/share
      - ./config:/etc/samba
    environment:
      - SAMBA_PASS=Aa.123456
```

##### 绑定指定网卡版本
```
services:
  samba:
    image: ghcr.io/chinanewsky/samba:4.24.7
    container_name: samba
    hostname: samba
    restart: unless-stopped
    ports:
      # 端口只绑定内网网卡 192.168.1.5 —— 本机另一块网卡(101.231.216.X)不开放 SMB。
      # docker-proxy 仅在该地址监听, 从其他网卡 IP 无法连到 139/445。
      - "10.8.205.160:137:137/udp"
      - "10.8.205.160:138:138/udp"
      - "10.8.205.160:139:139"
      - "10.8.205.160:445:445"
    volumes:
      # 注意: 当前运行中的容器挂的是宿主机 /mnt (共享整个磁盘根)。
      - /mnt:/share
      - ./config:/etc/samba
    environment:
      # samba 账号 root 的密码 (容器内独立于宿主机, entrypoint 每次启动按此值重设)
      - SAMBA_PASS=Aa.123456
```
