# Jellyfin

## 下载

[官方](https://jellyfin.org/) - [Github](https://github.com/jellyfin/jellyfin) - [Docker](https://hub.docker.com/r/jellyfin/jellyfin) - [第三方Docker](https://hub.docker.com/r/linuxserver/jellyfin)

## docker-compose配置

```yaml
version: "3"
services:
  jellyfin:
    image: jellyfin/jellyfin
    container_name: jellyfin
    restart: "no"
    ports:
      - 8096:8096
    environment:
      - GID=1000
      - UID=1000
    volumes:
      - ./config:/config
      - ./cache:/cache
      - ./media:/media
    logging:
      options:
        max-size: "10m"
        max-file: "1"
networks:
  default:
    external: true
    name: host-network
```

> [配置二级目录](https://jellyfin.org/docs/general/networking/caddy.html#subpath)`Admin > Networking and enter a path like /jellyfin in the Base URL field`


## 硬件加速

[link](https://jellyfin.org/docs/general/administration/hardware-acceleration.html)

## Caddyfile配置

二级目录格式

```yaml
reverse_proxy /jellyfin/* jellyfin:8096
```