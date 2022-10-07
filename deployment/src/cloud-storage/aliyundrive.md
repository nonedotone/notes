# Aliyundrive

阿里云盘webdav [Github](https://github.com/messense/aliyundrive-webdav)

## docker-compose
```yaml
version: '3'
services:
  aliyundrive:
    container_name: aliyundrive
    image: messense/aliyundrive-webdav
    restart: "no"
    ports:
      - '8080:8080'
    volumes:
      - ./data/:/etc/aliyundrive-webdav/
    environment:
      - WEBDAV_AUTH_USER=xxxxx
      - WEBDAV_AUTH_PASSWORD=xxxxx
      - REFRESH_TOKEN=xxxxx
    logging:
      options:
        max-size: "10m"
        max-file: "1"
networks:
  default:
    external: true
    name: host-network
```

> REFRESH_TOKEN 环境变量为你的阿里云盘 refresh_token，WEBDAV_AUTH_USER 和 WEBDAV_AUTH_PASSWORD 为连接 WebDAV 服务的用户名和密码

### 获取 refresh_token
自动获取: 登录阿里云盘后，控制台粘贴 JSON.parse(localStorage.token).refresh_token