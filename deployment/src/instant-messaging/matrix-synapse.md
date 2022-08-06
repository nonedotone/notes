# Matrix Synapse

[Github](https://github.com/matrix-org/synapse) - [Docker](https://hub.docker.com/r/matrixdotorg/synapse/)


## 步骤
```bash
# 第一步初始化
./generate.sh

# 启动
docker-compose up -d

# 注册用户

# 注册一个名称i管理员用户
register_user.sh i admin

# 注册一个test用户
register_user.sh  test
```


## docker-compose配置
```yaml
version: '3'
services:
  synapse:
    image: matrixdotorg/synapse
    container_name: synapse
    restart: always
    volumes:
      - ./data:/data
      - ./media:/data/media_store
    environment:
      - UID=1000
      - GID=1000
      - TZ=Asia/Shanghai
    healthcheck:
      disable: true
    logging:
      options:
        max-size: "10m"
        max-file: "1"
networks:
  default:
    external: true
    name: host-network
```

## 脚本
### 初始化 generate.sh

```shell
docker run -it --rm \
    -v ${HOME}/.deploy/matrix/synapse/data:/data \
    -v /data/matrix/synapse/media:/data/media_store \
    -e SYNAPSE_SERVER_NAME=example.com \
    -e SYNAPSE_REPORT_STATS=no \
    -e UID=1000 \
    -e GID=1000 \
    matrixdotorg/synapse:latest generate
```

### 注册 register_user.sh
```shell
#!/bin/bash
#set -x

name=$1
password=$(date +%s | sha256sum | base64 | head -c 32 ; echo)
is_admin="--no-admin"

# string=$(cat data/homeserver.yaml|grep 'registration_shared_secret:')
# prefix='registration_shared_secret: "'
# suffix='"'
# shared_secret=$(echo "$string" | sed -e "s/^$prefix//" -e "s/$suffix$//")

if [[ -z $name ]];then
    echo "name is empty!!!";
    exit 1;
fi

if [[ "$2" == "admin" ]]; then
    is_admin="--admin"
fi

read -r -p "name: ${name}, admin: ${is_admin}, OK? [yes/no] " input

case $input in
    [yY][eE][sS]|[yY])
        docker exec -it synapse register_new_matrix_user http://localhost:8008 \
            --config /data/homeserver.yaml \
            --user ${name} \
            --password ${password} ${is_admin}
        echo "register user success, name ${name}, password ${password}"
        ;;

    [nN][oO]|[nN])
        echo "exit"
        ;;

    *)
        echo "Invalid input..."
        exit 1
        ;;
esac
```