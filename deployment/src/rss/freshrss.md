# FreshRSS

A free, self-hostable aggregator… probably the best! (in our opinion)

## 下载

[官网](https://freshrss.org/) - [Github](https://github.com/FreshRSS/FreshRSS) - [Docker](https://hub.docker.com/r/freshrss/freshrss)

## docker-compose配置
```yaml
version: "3"

services:
  freshrss:
    container_name: freshrss
    image: freshrss/freshrss:arm
    restart: "no"
    volumes:
      - ./data:/var/www/FreshRSS/data
      - ./extensions:/var/www/FreshRSS/extensions
    environment:
      - CRON_MIN=*/20
      - TZ=Asia/Shanghai
    logging:
      options:
        max-size: "10m"
        max-file: "1"
networks:
  default:
    external: true
    name: host-network
```

## Caddyfile配置
```Caddyfile
route /freshrss/* {
    uri strip_prefix /freshrss
    reverse_proxy freshrss:80 {
        header_down Set-Cookie "path=/i/" "path=/freshrss/i/"
    }
}
```