## Install Kuma Monitoring Web App Alternative Uptime Robot
### 1. Buat Folder
```bash
mkdir uptime
```
### 2. Buat Folder .env
```bash
sudo nano .env
```
```bash
KUMA_DOMAIN=sub.domain.com
```
### 2. Buat Folder .env
```bash
sudo nano docker-compose.yml
```
Diisi dengan :
```bash
services:
  uptime-kuma:
    image: louislam/uptime-kuma:1
    container_name: uptime-kuma
    restart: always
    volumes:
      - ./data:/app/data
      # Opsional: Memungkinkan Kuma memonitor status Docker lain di server ini
      - /var/run/docker.sock:/var/run/docker.sock
    networks:
      - webproxy
    labels:
      - "traefik.enable=true"
      # Menggunakan domain dari file .env
      - "traefik.http.routers.uptime-kuma.rule=Host(`${KUMA_DOMAIN}`)"
      - "traefik.http.routers.uptime-kuma.entrypoints=websecure"
      - "traefik.http.routers.uptime-kuma.tls.certresolver=letsencrypt"
      # Port internal Uptime Kuma adalah 3001
      - "traefik.http.services.uptime-kuma.loadbalancer.server.port=3001"

networks:
  webproxy:
    external: true
```
### 3. Jalankan docker compose
```bash
docker compose up -d
```
