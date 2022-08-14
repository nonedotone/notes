# frp

[Docs](https://gofrp.org/docs/) - [Github](https://github.com/fatedier/frp) - [Docker](https://hub.docker.com/r/nonedotone/frp)


## docker-compose

## server
```yaml
version: '3'
services:
  frps-pi:
    container_name: frps-pi
    image: nonedotone/frp
    restart: always
    command: [ "/frp/frps", "-c", "/frp/frps.ini" ]
    ports:
      - 8000-9000:8000-9000
    volumes:
      - ./frps.ini:/frp/frps.ini
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

## client
```yaml
version: '3'
services:
  frpc-ssh:
    container_name: frpc-ssh
    image: nonedotone/frp
    restart: always
    command: [ "/frp/frpc", "-c", "/frp/frpc.ini" ]
    volumes:
      - ./frpc.ini:/frp/frpc.ini
      - ./config:/config
    extra_hosts:
      - "example.server.com:xxx.xxx.xxx.xxx"
    logging:
      options:
        max-size: "10m"
        max-file: "1"
networks:
  default:
    external: true
    name: host-network
```

## config

### server - frps.ini
```ini
[common]
bind_port = 8000
kcp_bind_port = 8000
token = xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# pool
max_pool_count = 20

# log
log_level = info

# tls
tls_enable = true
tls_cert_file = /config/server.crt
tls_key_file = /config/server.key
tls_trusted_ca_file = /config/ca.crt
```

### client - frpc.ini
```ini
[common]
# config
server_addr = example.server.com
server_port = 8000
token = xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

log_level = info

pool_count = 10

# tls
tls_enable = true
tls_cert_file = /config/client.crt
tls_key_file = /config/client.key
tls_trusted_ca_file = /config/ca.crt

# service
[ssh]
type = tcp
local_ip = host.network
local_port = 22
remote_port = 8888
use_compression = true
```

## 证书

下面简单示例如何用 openssl 生成 ca 和双方 SAN 证书。

准备默认 OpenSSL 配置文件于当前目录。此配置文件在 linux 系统下通常位于 /etc/pki/tls/openssl.cnf，在 mac 系统下通常位于 /System/Library/OpenSSL/openssl.cnf。

如果存在，则直接拷贝到当前目录，例如 cp /etc/pki/tls/openssl.cnf ./my-openssl.cnf。如果不存在可以使用下面的命令来创建

```bash
cat > my-openssl.cnf << EOF
[ ca ]
default_ca = CA_default
[ CA_default ]
x509_extensions = usr_cert
[ req ]
default_bits        = 2048
default_md          = sha256
default_keyfile     = privkey.pem
distinguished_name  = req_distinguished_name
attributes          = req_attributes
x509_extensions     = v3_ca
string_mask         = utf8only
[ req_distinguished_name ]
[ req_attributes ]
[ usr_cert ]
basicConstraints       = CA:FALSE
nsComment              = "OpenSSL Generated Certificate"
subjectKeyIdentifier   = hash
authorityKeyIdentifier = keyid,issuer
[ v3_ca ]
subjectKeyIdentifier   = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints       = CA:true
EOF
```


生成默认 ca:
```bash
openssl genrsa -out ca.key 2048
openssl req -x509 -new -nodes -key ca.key -subj "/CN=example.ca.com" -days 5000 -out ca.crt
```


生成 frps 证书:
```bash
openssl genrsa -out server.key 2048

openssl req -new -sha256 -key server.key \
    -subj "/C=XX/ST=DEFAULT/L=DEFAULT/O=DEFAULT/CN=server.com" \
    -reqexts SAN \
    -config <(cat my-openssl.cnf <(printf "\n[SAN]\nsubjectAltName=DNS:localhost,IP:127.0.0.1,DNS:example.server.com")) \
    -out server.csr

openssl x509 -req -days 3650 -sha256 \
	-in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial \
	-extfile <(printf "subjectAltName=DNS:localhost,IP:127.0.0.1,DNS:example.server.com") \
	-out server.crt
```

生成 frpc 的证书:
```bash
openssl genrsa -out client.key 2048
openssl req -new -sha256 -key client.key \
    -subj "/C=XX/ST=DEFAULT/L=DEFAULT/O=DEFAULT/CN=client.com" \
    -reqexts SAN \
    -config <(cat my-openssl.cnf <(printf "\n[SAN]\nsubjectAltName=DNS:client.com,DNS:example.client.com")) \
    -out client.csr

openssl x509 -req -days 3650 -sha256 \
    -in client.csr -CA ca.crt -CAkey ca.key -CAcreateserial \
	-extfile <(printf "subjectAltName=DNS:client.com,DNS:example.client.com") \
	-out client.crt
```

在本例中，server.crt 和 client.crt 都是由默认 ca 签发的，因此他们对默认 ca 是合法的。


* 默认采用了 sha1 的算法，只要在生成 *.crt 文件命令强制使用 sha256 算法即可