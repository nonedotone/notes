# RSSBot

Lightweight Telegram RSS notification bot. 用于消息通知的轻量级 Telegram RSS 机器人


## 下载

[Github](https://github.com/iovxw/rssbot) - [自用Docker](https://hub.docker.com/r/nonedotone/rssbot)

## docker-compose配置

```yaml
version: '3'
services:
  rssbot:
    image: nonedotone/rssbot
    container_name: rssbot
    command: '<BOT_TOKEN> --insecure --restricted --database /config/rssbot.json --max-feed-size 0 --max-interval 7200 --min-interval 3600 --admin <id> --admin <id>'
    restart: always
    volumes:
      - ./config:/config
    logging:
      options:
        max-size: "10m"
        max-file: "1"
networks:
  default:
    external: true
    name: host-network
```

## 使用
```bash
命令列表：
/rss       - 显示当前订阅的 RSS 列表
/sub       - 订阅一个 RSS：/sub http://example.com/feed.xml
/unsub     - 退订一个 RSS：/unsub http://example.com/feed.xml
/export    - 导出为 OPML
所有命令均可在后面跟上频道 ID 来管理频道订阅
例如 /sub @BotNews http://example.com/feed.xml
如果是私有频道 /sub -1001745000000 http://example.com/feed.xml

```

## 运行
```bash
USAGE:
    rssbot [FLAGS] [OPTIONS] <token>

FLAGS:
    -h, --help          Prints help information
        --insecure      DANGER: Insecure mode, accept invalid TLS certificates
        --restricted    Make bot commands only accessible for group admins
    -V, --version       Prints version information

OPTIONS:
        --admin <user id>...        Private mode, only specified user can use this bot. This argument can be passed
                                    multiple times to allow multiple admins
    -d, --database <path>           Path to database [default: ./rssbot.json]
        --max-feed-size <bytes>     Maximum feed size, 0 is unlimited [default: 2097152]
        --max-interval <seconds>    Maximum fetch interval [default: 43200]
        --min-interval <seconds>    Minimum fetch interval [default: 300]

ARGS:
    <token>    Telegram bot token

NOTE: You can get <user id> using bots like @userinfobot @getidsbot
```