# icloudpd

## 下载

[Github](https://github.com/boredazfcuk/docker-icloudpd) - [Docker](https://hub.docker.com/r/boredazfcuk/icloudpd)


## docker-compose配置
```yaml
version: '3'
services:
  icloudpd:
    container_name: icloudpd
    image: boredazfcuk/icloudpd
    restart: "no"
    environment:
      - apple_id=apple-id@gmail.com
      - download_path=/icloud
      - config_dir=/config
    volumes:
      - ./data:/icloud
      - ./config:/config
    logging:
      options:
        max-size: "10m"
        max-file: "1"
```

## 初始化
```bash
docker exec -it icloudpd sync-icloud.sh --Initialise
```

## 故障保护功能

此容器添加了故障保护功能，因此它不会对文件系统进行任何更改，除非它可以验证它正在写入的卷是否已正确安装。容器将在容器内的**下载目标目录**(download_path)中查找名为“/home/${user}/iCloud/.mounted”的文件（请注意 iCloud 的大写）。如果此文件不存在，它将不会从 iCloud 下载任何内容。这样，如果底层磁盘/卷/任何东西被卸载，同步将不会发生，这可以防止脚本填满主机的根卷。该文件必须手动创建，没有它就无法开始同步