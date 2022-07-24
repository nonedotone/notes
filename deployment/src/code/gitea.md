# Gitea

Git with a cup of tea, painless self-hosted git service

## 下载

[官网](https://gitea.io/zh-cn/) - [Github](https://github.com/go-gitea/gitea) - [Docker](https://hub.docker.com/r/gitea/gitea)


## docker-compose配置
```yaml
version: "3"
services:
  gitea:
    image: gitea/gitea
    container_name: gitea
    restart: "no"
    ports:
      - "127.0.0.1:2222:22"
    environment:
      - USER_UID=1000
      - USER_GID=1000
      - TZ=Asia/Shanghai
      - DISABLE_SSH=false
      - RUN_MODE=prod
      - APP_NAME=Code
      - DISABLE_REGISTRATION=true
      - ROOT_URL="https://example.com/code"
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /etc/timezone:/etc/timezone:ro
      # 需要git配置时使用
      #- /home/git/.ssh:/data/git/.ssh
      - ./data:/data
    logging:
      options:
        max-size: "3m"
        max-file: "1"
networks:
  default:
    external: true
    name: host-network
```

## Caddyfile配置
> 老版本 rewrite /code /code/
```Caddyfile
redir /code /code/
route /code/* {
    uri strip_prefix /code
    reverse_proxy gitea:3000 {
        header_up X-Real-IP {http.request.remote.host}
    }
}
```

## 配置ssh

...