## Install Nextcloud Alternative Google Drive
### 1. Buat Folder
```bash
mkdir drivecloud
```
### 2. Buat Folder .env
```bash
sudo nano .env
```
```bash
# Domain & Admin
NEXTCLOUD_DOMAIN=sub.domain.com
NEXTCLOUD_ADMIN_USER=your.email@mail.com
NEXTCLOUD_ADMIN_PASSWORD=Password@123

# Database (Internal)
DB_NAME=nextcloud_db
DB_USER=nextcloud_user
DB_PASSWORD=Password@123
DB_ROOT_PASSWORD=Password@123
```
### 2. Buat File docker-compose.yml
```bash
sudo nano docker-compose.yml
```
Diisi dengan :
```bash
services:
  nextcloud-db:
    image: mariadb:10.11
    container_name: nextcloud_db
    restart: always
    command: --transaction-isolation=READ-COMMITTED --binlog-format=ROW
    volumes:
      - ./db_data:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=${DB_ROOT_PASSWORD}
      - MYSQL_PASSWORD=${DB_PASSWORD}
      - MYSQL_DATABASE=${DB_NAME}
      - MYSQL_USER=${DB_USER}
    networks:
      - nextcloud_network

  nextcloud-app:
    image: nextcloud:latest
    container_name: nextcloud_app
    restart: always
    depends_on:
      - nextcloud-db
    environment:
      - MYSQL_PASSWORD=${DB_PASSWORD}
      - MYSQL_DATABASE=${DB_NAME}
      - MYSQL_USER=${DB_USER}
      - MYSQL_HOST=nextcloud-db
      - NEXTCLOUD_ADMIN_USER=${NEXTCLOUD_ADMIN_USER}
      - NEXTCLOUD_ADMIN_PASSWORD=${NEXTCLOUD_ADMIN_PASSWORD}
      - NEXTCLOUD_TRUSTED_DOMAINS=${NEXTCLOUD_DOMAIN}
      - OVERWRITEPROTOCOL=https
      - TRUSTED_PROXIES=172.16.0.0/12 192.168.0.0/16 10.0.0.0/8
    volumes:
      - ./app_data:/var/www/html
    networks:
      - nextcloud_network
      - webproxy
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.nextcloud.rule=Host(`${NEXTCLOUD_DOMAIN}`)"
      - "traefik.http.routers.nextcloud.entrypoints=websecure"
      - "traefik.http.routers.nextcloud.tls.certresolver=letsencrypt"
      - "traefik.http.services.nextcloud.loadbalancer.server.port=80"
      - "traefik.docker.network=webproxy"

networks:
  nextcloud_network:
  webproxy:
    external: true
```
### 3. Jalankan docker compose
```bash
docker compose up -d
```
