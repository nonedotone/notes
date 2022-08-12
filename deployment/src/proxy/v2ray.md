# V2ray - [link](https://github.com/v2fly/v2ray-core)

A platform for building proxies to bypass network restrictions.

## 下载
[官网](https://v2fly.org/) - [Github](https://github.com/v2fly/v2ray-core/releases) - [Docker](https://hub.docker.com/r/v2fly/v2fly-core)

## 配置

### 服务端配置

#### VMESS-WS协议
```json
{
    "log": {
        "loglevel": "debug"
    },
    "inbounds": [
        {
            "port": 8000,
            "protocol": "vmess",
            "settings": {
                "clients": [
                    {
                        "id": "00000000-0000-0000-0000-000000000000",
                        "email": "a@example.com"
                    }
                ]
            },
            "streamSettings": {
                "network": "ws",
                "security": "none",
                "wsSettings": {
                    "path": "/v2ray"
                }
            },
            "sniffing": {
                "enabled": true,
                "destOverride": [
                    "http",
                    "tls"
                ]
            }
        }
    ],
    "outbounds": [
        {
            "tag": "direct",
            "protocol": "freedom",
            "settings": {}
        },
        {
            "tag": "blocked",
            "protocol": "blackhole",
            "settings": {}
        }
    ],
    "routing": {
        "domainStrategy": "IPOnDemand",
        "rules": [
            {
                "type": "field",
                "outboundTag": "blocked",
                "ip": [
                    "geoip:private"
                ]
            }
        ]
    }
}
```

## docker-compose配置

### 服务端配置
```yaml
version: '3'
services:
  v2ray:
    image: v2fly/v2fly-core
    container_name: v2ray
    restart: always
    volumes:
      - ./config.json:/etc/v2ray/config.json
    logging:
      options:
        max-size: "10m"
        max-file: "1"
networks:
  default:
    external: true
    name: host-network
```


## 其他配置

### 根据目标地址或域名分流

```json
{
    "log": {
        "loglevel": "error"
    },
    "inbounds": [
        {
            "port": 8000,
            "protocol": "vmess",
            "settings": {
                "clients": [
                    {
                        "id": "0000-00000000-9001-000000000000",
                        "alterId": 64,
                        "level": 1
                    }
                ],
                "decryption": "none"
            },
            "streamSettings": {
                "network": "ws",
                "wsSettings": {
                    "path": "/ray"
                }
            }
        }
    ],
    "outbounds": [
        {
            "tag": "proxy-de",
            "protocol": "vmess",
            "settings": {
                "vnext": [
                    {
                        "address": "example.net",
                        "port": 80,
                        "users": [
                            {
                                "alterId": 64,
                                "level": 1,
                                "id": "xxxxxx-xxxx-xxxx-xxxxx-xxxxxxx"
                            }
                        ]
                    }
                ]
            },
            "streamSettings": {
                "network": "ws",
                "security": "tls",
                "wsSettings": {
                    "path": "/xxxx"
                }
            }
        },
        {
            "tag": "blocked",
            "protocol": "blackhole",
            "settings": {}
        },
        {
            "tag": "direct",
            "protocol": "freedom",
            "settings": {}
        }
    ],
    "routing": {
        "domainStrategy": "IPIfNonMatch",
        "rules": [
            {
                "type": "field",
                "outboundTag": "blocked",
                "ip": [
                    "geoip:private"
                ]
            },
            {
                "type": "field",
                "outboundTag": "proxy-de",
                "ip": [
                    "xxx.xxx.xxx.xxx"
                ]
            },
            {
                "type": "field",
                "outboundTag": "proxy-de",
                "domain": [
                    "domain:example.com"
                ]
            },
            {
                "type": "field",
                "network": "tcp,udp",
                "outboundTag": "direct"
            }
        ]
    }
}
```