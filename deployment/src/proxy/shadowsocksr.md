# Shadowsocksr

A fast tunnel proxy that helps you bypass firewalls.

## 下载

[Github](https://github.com/nonedotone/shadowsocksr) - [DockerServer](https://hub.docker.com/r/nonedotone/shadowsocksr-server) - [DockerClient](https://hub.docker.com/r/nonedotone/shadowsocksr-client)

## 配置

#### 服务端配置
```json
{
    "server":"0.0.0.0",
    "server_ipv6":"::",
    "server_port":8000,
    "local_address":"127.0.0.1",
    "local_port":1080,
    "password":"xxxxxxxxxxxxxxxxxxxxxxxxxxxx",
    "timeout":120,
    "method":"none",
    "protocol":"origin",
    "protocol_param":"",
    "obfs":"tls1.2_ticket_auth",
    "obfs_param":"",
    "redirect":"*:8000#www.google.com:443",
    "dns_ipv6":false,
    "fast_open":false,
    "workers":1
}
```

#### 客户端配置
```json
{
    "server": "x.x.x.x",
    "server_port": 8000,
    "local_port": 1080,
    "password": "xxxxxxxxxxxxxxxxxxxxxxxxxxxx",
    "timeout": 120,
    "method": "none",
    "protocol": "origin",
    "protocol_param": "",
    "obfs": "tls1.2_ticket_auth",
    "obfs_param": "",
    "local_address": "0.0.0.0",
    "fast_open": false
}
```

## docker-compose配置

#### 服务端配置
```yaml
version: "3"
services:
  ssr:
    container_name: ssr
    image: "nonedotone/shadowsocksr-server"
    command: '-c /etc/shadowsocks-r/config.json'
    volumes:
      - ./config.json:/etc/shadowsocks-r/config.json
    restart: always
    logging:
      options:
        max-size: "10m"
        max-file: "1"
networks:
  default:
    external: true
    name: host-network
```

#### 客户端配置
```yaml
version: "3"
services:
  ssr:
    container_name: ssr
    image: "nonedotone/shadowsocksr-client"
    command: '-c /etc/shadowsocks-r/config.json'
    volumes:
      - ./config.json:/etc/shadowsocks-r/config.json
    restart: always
    logging:
      options:
        max-size: "10m"
        max-file: "1"
networks:
  default:
    external: true
    name: host-network
```