# Navidrome

[官网](https://www.navidrome.org/) - [Github](https://github.com/navidrome/navidrome) - [Docker](https://hub.docker.com/r/deluan/navidrome)


## docker-compose
```yaml
version: "3"
services:
  navidrome:
    image: deluan/navidrome:latest
    container_name: navidrome
    ports:
      - "4533:4533"
    restart: unless-stopped
    environment:
      ND_SCANSCHEDULE: 1h
      ND_LOGLEVEL: info
      ND_SESSIONTIMEOUT: 24h
      ND_BASEURL: "/music"
      ND_DEFAULTTHEME: Dark
    volumes:
      - "./data:/data"
      - "/data/navidrome/music:/music:ro"
    logging:
      options:
        max-size: "10m"
        max-file: "1"
networks:
  default:
    external: true
    name: host-network
```


## Caddyfile
```Caddyfile
rewrite /music /music/
reverse_proxy /music/* navidrome:4533
```