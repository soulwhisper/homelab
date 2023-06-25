# Bookstack
- for self notes
- for test, use "http://lib.homelab.arpa"

## Production
- for real, use "https://lib.homelab.arpa"
- deploy with k8s
- login with OIDC
- storage with local_secure
- caching with redis

## Prepare
```shell
mkdir -p /opt/docker/app/bookstack/configs
mkdir -p /opt/docker/database/mysql-bs

chown -R 1000:1000 /opt/docker/app
chown -R 1001 /opt/docker/database
```

## Env
```shell
BASE_DIR=/opt/docker

COMPOSE_SUBNET=192.168.112.0/20
COMPOSE_GATEWAY=192.168.112.1

VER_BOOKSTACK=23.05.2
VER_MARIADB=10.11-debian-11

DB_BOOKSTACK=bookstack
```

## Compose
```shell
version: "3"

# template
x-basics: &basics
  restart: "unless-stopped"
  security_opt:
    - "no-new-privileges=true"
  stop_grace_period: "10s"
  networks:
    - "compose"

# network
networks:
  compose:
    driver: "bridge"
    internal: false
    ipam:
      driver: "default"
      config:
        - subnet: "${COMPOSE_SUBNET}"
          gateway: "${COMPOSE_GATEWAY}"

services:
  bookstack:
    <<: [*basics]
    image: "lscr.io/linuxserver/bookstack:${VER_BOOKSTACK}"
    container_name: "bookstack"
    healthcheck:
      test: "wget --no-verbose --tries=1 --spider http://localhost:80 || exit 1"
      start_period: 1m
    depends_on:
      - mysql-bs
    ports:
      - 80:80
    volumes:
      - "${BASE_DIR}/app/bookstack/configs:/config"
    environment:
      - "PUID=1000"
      - "PGID=1000"
      - "APP_URL=https://lib.homelab.arpa"
      - "DB_HOST=mysql-bs"
      - "DB_PORT=3306"
      - "DB_USER=${DB_BOOKSTACK}"
      - "DB_PASS=${DB_BOOKSTACK}"
      - "DB_DATABASE=${DB_BOOKSTACK}"
      
  mysql-bs:
    <<: [*basics]
    image: "bitnami/mariadb:${VER_MARIADB}"
    container_name: "mysql-bs"
    healthcheck:
      test: ['CMD', '/opt/bitnami/scripts/mariadb/healthcheck.sh']
    volumes:
      - "${BASE_DIR}/database/mysql-bs:/bitnami/mariadb"
    environment:
      - "ALLOW_EMPTY_PASSWORD=yes"
      - "MARIADB_SKIP_TEST_DB=yes"
      - "MARIADB_USER=${DB_BOOKSTACK}"
      - "MARIADB_PASSWORD=${DB_BOOKSTACK}"
      - "MARIADB_DATABASE=${DB_BOOKSTACK}"
```
