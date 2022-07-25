# Vaultwraden

Unofficial Bitwarden compatible server written in Rust, formerly known as bitwarden_rs

## 下载

[Github](https://github.com/dani-garcia/vaultwarden) - [Docker](https://hub.docker.com/r/vaultwarden/server)

## docker-compose配置

```yaml
version: "3"

services:
  vault:
    container_name: vault
    image: vaultwarden/server:alpine
    restart: "no"
    volumes:
      - ./data:/data
      - ./logs:/logs
    environment:
      - SIGNUPS_ALLOWED=true
      #- SIGNUPS_ALLOWED=false
      - DOMAIN=https://example.com/vault
      - DATABASE_URL=/data/vault.db
      - INVITATIONS_ALLOWED=true
      - ROCKET_LIMITS={json=10485760}
      - LOG_FILE=/logs/vault.log
      - LOG_LEVEL=info
      - ROCKET_WORKERS=4
      - WEBSOCKET_ENABLED=true
      - WEB_VAULT_ENABLED=true
      #- WEB_VAULT_ENABLED=false
      #- SMTP_SSL=false
      #- SMTP_SECURITY=off
      #- SMTP_PORT=25
      #- SMTP_HOST=smtp-proxy
      #- SMTP_FROM=vault@example.com
      #- HTTP_PROXY=http://proxy:1087
      #- HTTPS_PROXY=http://proxy:1087
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
route /vault/* {
    reverse_proxy /vault/notifications/hub vault:3012
    reverse_proxy vault:80 {
        header_up X-Real-IP {http.request.remote.host}
    }
}
```