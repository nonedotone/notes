# RSSHub

RSSHub 是一个开源、简单易用、易于扩展的 RSS 生成器，可以给任何奇奇怪怪的内容生成 RSS 订阅源。RSSHub 借助于开源社区的力量快速发展中，目前已适配数百家网站的上千项内容

## 下载

[Github](https://github.com/DIYgod/RSSHub) - [Docker](https://hub.docker.com/r/diygod/rsshub) - [Docs](https://docs.rsshub.app/)


## docker-compose配置
```yaml
version: '3'

services:
  rsshub.app:
    image: diygod/rsshub
    container_name: rsshub.app
    restart: "always"
    environment:
      NODE_ENV: production
      DEBUG_INFO: 'false'
      DISALLOW_ROBOT: 'false'
      PORT: 80
      #PROXY_URI: 'socks://proxy:1086'
      GITHUB_ACCESS_TOKEN: 'xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx'
      DISQUS_API_KEY: 'xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx'
      TWITTER_CONSUMER_KEY: 'xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx'
      TWITTER_CONSUMER_SECRET: 'xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx'
      TWITTER_TOKEN_nonedotone: 'xxxxxxxxxxxxxxxx,xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx,xxxxxxxx-xxxxxxxx,xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx'
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
route /rsshub/* {
    uri strip_prefix /rsshub
    reverse_proxy rsshub:80
}
```