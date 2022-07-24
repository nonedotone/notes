# Caddy - [link](https://caddyserver.com/)

THE ULTIMATE SERVER

Caddy is a powerful, enterprise-ready, open source web server with automatic HTTPS written in Go

## 下载

[官方二进制文件](https://caddyserver.com/download) - [官方docker](https://hub.docker.com/_/caddy) - [自用docker](https://hub.docker.com/r/nonedotone/caddy) - [自用docker-hugo](https://hub.docker.com/r/nonedotone/caddy-hugo)


## 配置文件Caddyfile

#### 通用配置
```Caddyfile
localhost

file_server
reverse_proxy /api/* 127.0.0.1:9005
```

#### 自用配置

```Caddyfile
none.one {
    # 指定证书
    tls path/to/cert path/to/key
    # 自动获取证书
    tls {
    	dns cloudflare xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
    }
    # 日志
    log {
        output file /var/log/access.log {
            roll_size 10mb
            roll_keep 3
            roll_keep_for 720h
        }
    }

    # 导入配置
    import path/to/notes
    import path/to/blogger
}
```


## docker-compose配置文件

#### 通用配置
```yaml
version: '3'
services:
  caddy:
    container_name: caddy
    image: caddy
    restart: always
    ports:
      - 80:80
      - 443:443
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile
      - ./config:/config
      - ./data:/data
      - ./log:/var/log
    logging:
      options:
        max-size: "10m"
        max-file: "1"
networks:
  default:
    external: true
    name: host-network
```

#### 自用配置
```bash
version: '3'
services:
  none.one:
    container_name: none.one
    image: nonedotone/caddy-hugo
    restart: always
    ports:
      - 0.0.0.0:80:80
      - 0.0.0.0:443:443
    environment:
      - BLOG=https://github.com/nonedotone/blogger.git
      - NOTES=https://github.com/nonedotone/notes.git
    volumes:
      - ./config:/etc/caddy
      - ./.ssl:/data
      - ./log:/var/log
    logging:
      options:
        max-size: "10m"
        max-file: "1"
networks:
  default:
    external: true
    name: host-network
```

config目录结构
```bash
config
├── Caddyfile
└── snippet
    ├── blogger
    └── notes
```