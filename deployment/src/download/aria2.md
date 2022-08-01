# Aria2

下载神器

## 下载

[Github](https://github.com/aria2/aria2) - [Docker](https://hub.docker.com/r/p3terx/aria2-pro)

## docker-compose配置
```yaml
version: '3'
services:
  aria2:
    image: p3terx/aria2-pro
    container_name: aria2
    restart: always
    volumes:
      - ./config:/config
      - ./downloads:/downloads
    environment:
      - RPC_SECRET=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
      - SPECIAL_MODE=rclone
    logging:
      options:
        max-size: "10m"
        max-file: "1"
networks:
  default:
    external: true
    name: host-network
```

> config目录下有个 script.conf 文件，通过它可以自定义文件删除功能、文件清理功能、文件过滤功能。文件上传设置和文件移动设置为特殊模式选项


## Caddyfile配置
```Caddyfile
reverse_proxy /jsonrpc aria2:6800 {
    header_up Host {http.reverse_proxy.upstream.hostport}
}
```

## 配套工具

### AriaNG

#### 下载

[Github](https://github.com/mayswind/AriaNg) - [Releases](https://github.com/mayswind/AriaNg/releases)

### Caddyfile配置
> 使用单文件版本
```Caddyfile
handle /ariang/* {
    uri strip_prefix /ariang
    root * /srv/ariang/
    file_server
}
```

### tele-aria2

#### 下载

[Github](https://github.com/HouCoder/tele-aria2) - [Docker](https://hub.docker.com/r/p3terx/tele-aria2)

#### docker-compose配置
```yaml
version: '3'
services:
  tele-aria2:
    container_name: tele-aria2
    image: p3terx/tele-aria2
    restart: "no"
    volumes:
      - ./config.json:/config.json
    logging:
      options:
        max-size: "10m"
        max-file: "1"
networks:
  default:
    external: true
    name: host-network
```


#### config配置
```json
{
  "aria2-server": "http://aria2:6800/jsonrpc",
  "aria2-key": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
  "proxy": "http://proxy:1087",
  "bot-key": "1234567890:qwertyuiopqwertyuiopqwertyuiopqwertyuiop",
  "user-id": "<user-id>",
  "max-index": 20
}
```


## 其他

* [第三方](https://p3terx.com/archives/docker-aria2-pro.html)