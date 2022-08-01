# Joplin

## 下载

[官网](https://joplinapp.org/) - [Github](https://github.com/laurent22/joplin)

服务端docker`florider89/joplin-server`


## docker-compose配置

```yaml
version: "3"

services:
  joplin:
    container_name: joplin
    image: florider89/joplin-server
    restart: "no"
    volumes:
      - ./db-prod.sqlite:/home/joplin/packages/server/db-prod.sqlite
    environment:
      - APP_BASE_URL=https://example.com/joplin
      - APP_PORT=8080
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

> 第一次启动从容器内部`docker cp`数据库文件，第二次映射sqlite文件启动


## Caddyfile配置
```Caddyfile
rewrite /joplin /joplin/
route /joplin/* {
    uri strip_prefix /joplin
    reverse_proxy joplin:8080 {
        header_up X-Real-IP {http.request.remote.host}
    }
}
```