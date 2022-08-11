# Nextcloud

[官网](https://nextcloud.com/) - [Github](github.com/nextcloud) - [Docker](https://hub.docker.com/_/nextcloud) - [Docker](https://hub.docker.com/r/linuxserver/nextcloud)

## docker-compose配置

```yaml
version: "3"

services:
  nextcloud:
    container_name: nextcloud
    image: linuxserver/nextcloud
    restart: "always"
    ports:
    - 8880:80
    - 8443:443
    volumes:
      - ./data:/data
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
    external: true
    name: host-network
```

## 二级目录配置

先正常启动一次，初始化所需数据以及文件

替换`<sub-path>`为需要的二级目录

复制`config/www/nextcloud`为`config/www/<sub-path>`

### Caddyfile

```Caddyfile
reverse_proxy /<sub-path>/* nextcloud:443 {
    transport http {
        tls_insecure_skip_verify
    }
}
```

### 容器中的nginx-conf文件

```conf
upstream php-handler {
    server 127.0.0.1:9000;
    #server unix:/var/run/php/php7.4-fpm.sock;
}

map $arg_v $asset_immutable {
    "" "";
    default "immutable";
}

server {
    listen 80;
    listen [::]:80;
    server_name _;

    location /<sub-path> {
        return 301 https://$server_name$request_uri;
    }
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name _;
    ssl_certificate /config/keys/cert.crt;
    ssl_certificate_key /config/keys/cert.key;

    root /config/www;

    location = /robots.txt {
        allow all;
        log_not_found off;
        access_log off;
    }

    location ^~ /.well-known {
        location = /.well-known/carddav { return 301 /<sub-path>/remote.php/dav/; }
        location = /.well-known/caldav  { return 301 /<sub-path>/remote.php/dav/; }

        location /.well-known/acme-challenge    { try_files $uri $uri/ =404; }
        location /.well-known/pki-validation    { try_files $uri $uri/ =404; }

        return 301 /<sub-path>/index.php$request_uri;
    }

    location ^~ /<sub-path> {
        client_max_body_size 2048M;
        client_body_timeout 300s;
        fastcgi_buffers 64 4K;

        gzip on;
        gzip_vary on;
        gzip_comp_level 4;
        gzip_min_length 256;
        gzip_proxied expired no-cache no-store private no_last_modified no_etag auth;
        gzip_types application/atom+xml application/javascript application/json application/ld+json application/manifest+json application/rss+xml application/vnd.geo+json application/vnd.ms-fontobject application/wasm application/x-font-ttf application/x-web-app-manifest+json application/xhtml+xml application/xml font/opentype image/bmp image/svg+xml image/x-icon text/cache-manifest text/css text/plain text/vcard text/vnd.rim.location.xloc text/vtt text/x-component text/x-cross-domain-policy;

        add_header Referrer-Policy                      "no-referrer"   always;
        add_header X-Content-Type-Options               "nosniff"       always;
        add_header X-Download-Options                   "noopen"        always;
        add_header X-Frame-Options                      "SAMEORIGIN"    always;
        add_header X-Permitted-Cross-Domain-Policies    "none"          always;
        add_header X-Robots-Tag                         "none"          always;
        add_header X-XSS-Protection                     "1; mode=block" always;

        fastcgi_hide_header X-Powered-By;

        index index.php index.html /<sub-path>/index.php$request_uri;

        location = /<sub-path> {
            if ( $http_user_agent ~ ^DavClnt ) {
                return 302 /<sub-path>/remote.php/webdav/$is_args$args;
            }
        }

        location ~ ^/<sub-path>/(?:build|tests|config|lib|3rdparty|templates|data)(?:$|/)    { return 404; }
        location ~ ^/<sub-path>/(?:\.|autotest|occ|issue|indie|db_|console)                  { return 404; }

        location ~ \.php(?:$|/) {
            # Required for legacy support
            rewrite ^/<sub-path>/(?!index|remote|public|cron|core\/ajax\/update|status|ocs\/v[12]|updater\/.+|oc[ms]-provider\/.+|.+\/richdocumentscode\/proxy) /<sub-path>/index.php$request_uri;

            fastcgi_split_path_info ^(.+?\.php)(/.*)$;
            set $path_info $fastcgi_path_info;

            try_files $fastcgi_script_name =404;

            include /etc/nginx/fastcgi_params;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            fastcgi_param PATH_INFO $path_info;
            fastcgi_param HTTPS on;

            fastcgi_param modHeadersAvailable true;         # Avoid sending the security headers twice
            fastcgi_param front_controller_active true;     # Enable pretty urls
            fastcgi_pass php-handler;

            fastcgi_intercept_errors on;
            fastcgi_request_buffering off;

            fastcgi_max_temp_file_size 0;
        }

        location ~ \.(?:css|js|svg|gif|png|jpg|ico|wasm|tflite|map)$ {
            try_files $uri /<sub-path>/index.php$request_uri;
            add_header Cache-Control "public, max-age=15778463, $asset_immutable";
            access_log off;     # Optional: Don't log access to assets

            location ~ \.wasm$ {
                default_type application/wasm;
            }
        }

        location ~ \.woff2?$ {
            try_files $uri /<sub-path>/index.php$request_uri;
            expires 7d;         # Cache-Control policy borrowed from `.htaccess`
            access_log off;     # Optional: Don't log access to assets
        }

        location /<sub-path>/remote {
            return 301 /<sub-path>/remote.php$request_uri;
        }

        location /<sub-path> {
            try_files $uri $uri/ /<sub-path>/index.php$request_uri;
        }
    }
}
```

### nextcloud配置文件config.php

路径: `config/www/<sub-path>/config/config.php`

* 添加或者修改`trusted_domains`
* 添加或者修改`overwritewebroot`
* 添加或者修改`overwrite.cli.url`

```php
<?php
$CONFIG = array (
  'memcache.local' => '\\OC\\Memcache\\APCu',
  'datadirectory' => '/data',
  'instanceid' => 'xxxxxxxx',
  'passwordsalt' => 'xxxxxxxx+xxxxxxxx',
  'secret' => 'xxxxxxxx/xxxxxxxx+xxxxxxxx+xxxxxxxx/xxxxxxxx+xxxxxxxx',
  'trusted_domains' =>
  array (
    0 => 'example.com',
  ),
  'dbtype' => 'sqlite3',
  'version' => '24.0.2.1',
  'overwritewebroot' => '/<sub-path>',
  'overwrite.cli.url' => 'https://example.com/<sub-path>',
  'installed' => true,
);
```

## 其他

扫描文件
```bash
# 扫描所有
docker exec -u 1000 -it nextcloud bash -c "php /config/www/nextcloud/occ files:scan --all"

# 指定文件夹
docker exec -u 1000 -it nextcloud bash -c "php /config/www/nextcloud/occ files:scan --path <name>/files/<path/to/directory>"
```

* [file-operations](https://docs.nextcloud.com/server/latest/admin_manual/configuration_server/occ_command.html#file-operations)