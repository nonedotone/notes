# H5ai

[Github](https://github.com/lrsjng/h5ai) - [Docker](https://hub.docker.com/r/awesometic/h5ai)


## docker-compose
```ymal
version: "3"
services:
  h5ai:
    container_name: h5ai
    image: awesometic/h5ai
    restart: always
    volumes:
      - /data/h5ai:/h5ai
      - ./config:/config
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Shanghai
    logging:
      options:
        max-size: "10m"
        max-file: "1"
networks:
  default:
    external:
      name: host-network
```


## 二级目录配置
### Caddyfile

```Caddyfile
redir /h5ai /h5ai/
route /h5ai/* {
    uri strip_prefix /h5ai
    reverse_proxy h5ai:80
}
```

### config配置

修改文件`config/h5ai/_h5ai/private/php/core/class-setup.php`

```
$this->set('SCRIPT_NAME', $_SERVER['SCRIPT_NAME']);

# to

$this->set('SCRIPT_NAME', "/h5ai" . $_SERVER['SCRIPT_NAME']);
```

## cache文件夹(先正常部署再转移)

```
# chmod 777
config/h5ai/_h5ai/public/cache
```

## 关闭多文件下载

配置文件`config/h5ai/_h5ai/private/conf/options.json`

```
# 在配置文件中搜索select，将true改为false

"select": {
	"enabled": true,
	"clickndrag": true,
	"checkboxes": true
},
```

## 修改右上角标识链接

文件：`config/h5ai/_h5ai/public/js/scripts.js`

搜索`powered by`和`h5ai`，分为两个部分

## other
* https://cnxiaobai.com/articles/2021/04/21/1619015365926.html