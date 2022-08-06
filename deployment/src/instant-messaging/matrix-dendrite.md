# Matrix Dendrite

[Github](https://github.com/matrix-org/dendrite) - [Docker](https://hub.docker.com/r/matrixdotorg/dendrite-monolith)


## docker-compose
```yaml
version: "3.4"
services:
  dendrite:
    container_name: dendrite
    image: matrixdotorg/dendrite-monolith
    command: [ "--tls-cert=server.crt", "--tls-key=server.key" ]
    restart: "no"
    environment:
      - http_proxy=http://proxy:1087
      - https_proxy=http://proxy:1087
      - no_proxy=example.com,localhost,127.0.0.1
    volumes:
      - ./config:/etc/dendrite
      - ./data:/data
      - ./media:/var/dendrite/media
    logging:
      options:
        max-size: "10m"
        max-file: "1"
networks:
  default:
    external: true
    name: host-network
```

## scripts

### generate.sh
```shell
#!/bin/bash

#set -x

domain=$1

if [ "$domain" == "" ];then
  echo "must be input domain!!!!";
  exit 1;
fi

share_secret=
system=$(uname -s)
if [ $system == "Darwin" ];then
  share_secret=$(date +%s | shasum -a 256 | base64 | head -c 64 ; echo)
else
  share_secret=$(date +%s | sha256sum | base64 | head -c 32 ; echo)
fi

echo "delete old config data media..."
rm -rf config data media docker-compose.yaml
if [ $? != 0 ];then
  echo "failed to remove old data"
  exit 1;
fi

echo "create config data media..."
mkdir config data media

echo "write dendrite yaml file..."
cat > config/dendrite.yaml << EOF
# This is the Dendrite configuration file.
#
# The configuration is split up into sections - each Dendrite component has a
# configuration section, in addition to the "global" section which applies to
# all components.
#
# At a minimum, to get started, you will need to update the settings in the
# "global" section for your deployment, and you will need to check that the
# database "connection_string" line in each component section is correct.
#
# Each component with a "database" section can accept the following formats
# for "connection_string":
#   SQLite:     file:filename.db
#               file:///path/to/filename.db
#   PostgreSQL: postgresql://user:pass@hostname/database?params=...
#
# SQLite is embedded into Dendrite and therefore no further prerequisites are
# needed for the database when using SQLite mode. However, performance with
# PostgreSQL is significantly better and recommended for multi-user deployments.
# SQLite is typically around 20-30% slower than PostgreSQL when tested with a
# small number of users and likely will perform worse still with a higher volume
# of users.
#
# The "max_open_conns" and "max_idle_conns" settings configure the maximum
# number of open/idle database connections. The value 0 will use the database
# engine default, and a negative value will use unlimited connections. The
# "conn_max_lifetime" option controls the maximum length of time a database
# connection can be idle in seconds - a negative value is unlimited.

# The version of the configuration file.
version: 2

# Global Matrix configuration. This configuration applies to all components.
global:
  # The domain name of this homeserver.
  server_name: ${domain}

  # The path to the signing private key file, used to sign requests and events.
  private_key: matrix_key.pem

  # The paths and expiry timestamps (as a UNIX timestamp in millisecond precision)
  # to old signing private keys that were formerly in use on this domain. These
  # keys will not be used for federation request or event signing, but will be
  # provided to any other homeserver that asks when trying to verify old events.
  # old_private_keys:
  # - private_key: old_matrix_key.pem
  #   expired_at: 1601024554498

  # How long a remote server can cache our server signing key before requesting it
  # again. Increasing this number will reduce the number of requests made by other
  # servers for our key but increases the period that a compromised key will be
  # considered valid by other homeservers.
  key_validity_period: 168h0m0s

  # The server name to delegate server-server communications to, with optional port
  # e.g. localhost:443
  well_known_server_name: ""

  # Lists of domains that the server will trust as identity servers to verify third
  # party identifiers such as phone numbers and email addresses.
  trusted_third_party_id_servers:
    - matrix.org
    - vector.im

  # Configuration for Prometheus metric collection.
  metrics:
    # Whether or not Prometheus metrics are enabled.
    enabled: false

    # HTTP basic authentication to protect access to monitoring.
    basic_auth:
      username: metrics
      password: metrics

  # DNS cache options. The DNS cache may reduce the load on DNS servers
  # if there is no local caching resolver available for use.
  dns_cache:
    # Whether or not the DNS cache is enabled.
    enabled: false

    # Maximum number of entries to hold in the DNS cache, and
    # for how long those items should be considered valid in seconds.
    cache_size: 256
    cache_lifetime: 300

# Configuration for the Appservice API.
app_service_api:
  internal_api:
    listen: http://0.0.0.0:7777
    connect: http://appservice_api:7777
  database:
    # connection_string: postgresql://dendrite:itsasecret@postgres/dendrite_appservice?sslmode=disable
    connection_string: file:///data/dendrite_appservice.db
    max_open_conns: 10
    max_idle_conns: 2
    conn_max_lifetime: -1

  # Appservice configuration files to load into this homeserver.
  config_files: []

# Configuration for the Client API.
client_api:
  internal_api:
    listen: http://0.0.0.0:7771
    connect: http://client_api:7771
  external_api:
    listen: http://0.0.0.0:8071

  # Prevents new users from being able to register on this homeserver, except when
  # using the registration shared secret below.
  registration_disabled: true

  # If set, allows registration by anyone who knows the shared secret, regardless of
  # whether registration is otherwise disabled.
  registration_shared_secret: "${share_secret}"

  # Whether to require reCAPTCHA for registration.
  enable_registration_captcha: false

  # Settings for ReCAPTCHA.
  recaptcha_public_key: ""
  recaptcha_private_key: ""
  recaptcha_bypass_secret: ""
  recaptcha_siteverify_api: ""

  # TURN server information that this homeserver should send to clients.
  turn:
    turn_user_lifetime: ""
    turn_uris: []
    turn_shared_secret: ""
    turn_username: ""
    turn_password: ""

  # Settings for rate-limited endpoints. Rate limiting will kick in after the
  # threshold number of "slots" have been taken by requests from a specific
  # host. Each "slot" will be released after the cooloff time in milliseconds.
  rate_limiting:
    enabled: true
    threshold: 5
    cooloff_ms: 500

# Configuration for the Federation API.
federation_api:
  internal_api:
    listen: http://0.0.0.0:7772
    connect: http://federation_api:7772
  external_api:
    listen: http://0.0.0.0:8072
  database:
    # connection_string: postgresql://dendrite:itsasecret@postgres/dendrite_federationapi?sslmode=disable
    connection_string: file:///data/dendrite_federationapi.db
    max_open_conns: 10
    max_idle_conns: 2
    conn_max_lifetime: -1

  # How many times we will try to resend a failed transaction to a specific server. The
  # backoff is 2**x seconds, so 1 = 2 seconds, 2 = 4 seconds, 3 = 8 seconds etc.
  send_max_retries: 16

  # Disable the validation of TLS certificates of remote federated homeservers. Do not
  # enable this option in production as it presents a security risk!
  disable_tls_validation: false

  # Use the following proxy server for outbound federation traffic.
  proxy_outbound:
    enabled: false
    protocol: http
    host: localhost
    port: 8080

  # Perspective keyservers to use as a backup when direct key fetches fail. This may
  # be required to satisfy key requests for servers that are no longer online when
  # joining some rooms.
  key_perspectives:
    - server_name: matrix.org
      keys:
        - key_id: ed25519:auto
          public_key: Noi6WqcDj0QmPxCNQqgezwTlBKrfqehY1u2FyWP9uYw
        - key_id: ed25519:a_RXGa
          public_key: l8Hft5qXKn1vfHrg3p4+W8gELQVo8N13JkluMfmn2sQ

  # This option will control whether Dendrite will prefer to look up keys directly
  # or whether it should try perspective servers first, using direct fetches as a
  # last resort.
  prefer_direct_fetch: false

# Configuration for the Key Server (for end-to-end encryption).
key_server:
  internal_api:
    listen: http://0.0.0.0:7779
    connect: http://key_server:7779
  database:
    # connection_string: postgresql://dendrite:itsasecret@postgres/dendrite_keyserver?sslmode=disable
    connection_string: file:///data/dendrite_keyserver.db
    max_open_conns: 10
    max_idle_conns: 2
    conn_max_lifetime: -1

# Configuration for the Media API.
media_api:
  internal_api:
    listen: http://0.0.0.0:7774
    connect: http://media_api:7774
  external_api:
    listen: http://0.0.0.0:8074
  database:
    # connection_string: postgresql://dendrite:itsasecret@postgres/dendrite_mediaapi?sslmode=disable
    connection_string: file:///data/dendrite_mediaapi.db
    max_open_conns: 10
    max_idle_conns: 2
    conn_max_lifetime: -1

  # Storage path for uploaded media. May be relative or absolute.
  base_path: /var/dendrite/media

  # The maximum allowed file size (in bytes) for media uploads to this homeserver
  # (0 = unlimited).
  max_file_size_bytes: 10485760

  # Whether to dynamically generate thumbnails if needed.
  dynamic_thumbnails: false

  # The maximum number of simultaneous thumbnail generators to run.
  max_thumbnail_generators: 10

  # A list of thumbnail sizes to be generated for media content.
  thumbnail_sizes:
    - width: 32
      height: 32
      method: crop
    - width: 96
      height: 96
      method: crop
    - width: 640
      height: 480
      method: scale

# Configuration for experimental MSC's
mscs:
  # A list of enabled MSC's
  # Currently valid values are:
  # - msc2836    (Threading, see https://github.com/matrix-org/matrix-doc/pull/2836)
  # - msc2946    (Spaces Summary, see https://github.com/matrix-org/matrix-doc/pull/2946)
  mscs: []
  database:
    # connection_string: postgresql://dendrite:itsasecret@postgres/dendrite_mscs?sslmode=disable
    connection_string: file:///data/dendrite_mscs.db
    max_open_conns: 5
    max_idle_conns: 2
    conn_max_lifetime: -1

# Configuration for the Room Server.
room_server:
  internal_api:
    listen: http://0.0.0.0:7770
    connect: http://room_server:7770
  database:
    # connection_string: postgresql://dendrite:itsasecret@postgres/dendrite_roomserver?sslmode=disable
    connection_string: file:///data/dendrite_roomserver.db
    max_open_conns: 10
    max_idle_conns: 2
    conn_max_lifetime: -1

# Configuration for the Sync API.
sync_api:
  internal_api:
    listen: http://0.0.0.0:7773
    connect: http://sync_api:7773
  external_api:
    listen: http://0.0.0.0:8073
  database:
    # connection_string: postgresql://dendrite:itsasecret@postgres/dendrite_syncapi?sslmode=disable
    connection_string: file:///data/dendrite_syncapi.db
    max_open_conns: 10
    max_idle_conns: 2
    conn_max_lifetime: -1

# Configuration for the User API.
user_api:
  internal_api:
    listen: http://0.0.0.0:7781
    connect: http://user_api:7781
  account_database:
    # connection_string: postgresql://dendrite:itsasecret@postgres/dendrite_userapi_accounts?sslmode=disable
    connection_string: file:///data/dendrite_userapi_accounts.db
    max_open_conns: 10
    max_idle_conns: 2
    conn_max_lifetime: -1

# Configuration for the Push Server API.
push_server:
  internal_api:
    listen: http://localhost:7782
    connect: http://localhost:7782
  database:
    # connection_string: postgresql://dendrite:itsasecret@postgres/dendrite_pushserver?sslmode=disable
    connection_string: file:///data/dendrite_pushserver.db
    max_open_conns: 10
    max_idle_conns: 2
    conn_max_lifetime: -1

# Configuration for Opentracing.
# See https://github.com/matrix-org/dendrite/tree/master/docs/tracing for information on
# how this works and how to set it up.
tracing:
  enabled: false
  jaeger:
    serviceName: ""
    disabled: false
    rpc_metrics: false
    tags: []
    sampler: null
    reporter: null
    headers: null
    baggage_restrictions: null
    throttler: null

# Logging configuration, in addition to the standard logging that is sent to
# stdout by Dendrite.
logging:
  - type: file
    level: info
    params:
      path: /var/log/dendrite
EOF

echo "generate keys..."
docker run --rm  \
  -v $(pwd)/config:/mnt --entrypoint="" \
  matrixdotorg/dendrite-monolith:latest \
  /usr/bin/generate-keys \
  -private-key /mnt/matrix_key.pem \
  -tls-cert /mnt/server.crt \
  -tls-key /mnt/server.key


echo "write docker compose yaml file..."
cat > docker-compose.yaml << EOF
version: "3.4"
services:
  dendrite:
    container_name: dendrite
    image: matrixdotorg/dendrite-monolith
    command: [ "--tls-cert=server.crt", "--tls-key=server.key" ]
    restart: always
    environment:
      - http_proxy=http://proxy:1087
      - https_proxy=http://proxy:1087
      - no_proxy=$domain,localhost,127.0.0.1
    volumes:
      - ./config:/etc/dendrite
      - ./data:/data
      - ./media:/var/dendrite/media
    logging:
      options:
        max-size: "10m"
        max-file: "1"
networks:
  default:
    external: true
    name: host-network
EOF

echo "generate complete!"
```

### create_user.sh

```shell
#!/bin/bash

#set -x

name=$1
password=$(date +%s | sha256sum | base64 | head -c 32 ; echo)
is_admin="no"
admin=

# string=$(cat data/homeserver.yaml|grep 'registration_shared_secret:')
# prefix='registration_shared_secret: "'
# suffix='"'
# shared_secret=$(echo "$string" | sed -e "s/^$prefix//" -e "s/$suffix$//")

if [[ -z $name ]];then
    echo "name is empty!!!";
    exit 1;
fi

if [[ "$2" == "admin" ]]; then
    admin="--admin"
    is_admin="yes"
fi

read -r -p "name: ${name}, admin: ${is_admin}, OK? [yes/no] " input

case $input in
    [yY][eE][sS]|[yY])
        docker exec -it dendrite create-account \
            --config /etc/dendrite/dendrite.yaml \
            -username ${name} -password ${password} ${admin}
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

## Caddyfile
```Caddyfile
handle /.well-known/matrix/server {
    respond `{"m.server":"example.com:443"}`
}

handle /.well-known/matrix/client {
    respond `{"m.homeserver":{"base_url":"https://example.com"}}`
}

handle /_matrix/* {
    reverse_proxy http://dendrite:8008
}

handle /_synapse/* {
    reverse_proxy http://dendrite:8008
}
```