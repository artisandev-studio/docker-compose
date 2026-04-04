## Install Gitlab Alternative Github
### Spesifikasi Server
```bash
4 GB RAM
8 core vCPU
```
### 1. Buat Folder
```bash
mkdir gitlab
```
### 2. Buat Folder .env
```bash
sudo nano .env
```
```bash
# Domain & Admin
# Domain & Email
GITLAB_DOMAIN=git.domain.com
EMAIL=your.email@mail.com

# Lokasi Penyimpanan Data (Sesuaikan jika perlu)
GITLAB_HOME=/srv/gitlab

# Konfigurasi SSH (Agar tidak bentrok dengan SSH VPS port 22)
GITLAB_SSH_PORT=2222
```
### 2. Buat File docker-compose.yml
```bash
sudo nano docker-compose.yml
```
Diisi dengan :
```bash
services:
  # 1. TRAEFIK (Manajemen SSL & Routing)
  traefik:
    image: traefik:latest
    container_name: traefik
    restart: always
    command:
      - "--api.insecure=false"
      - "--providers.docker=true"
      - "--providers.docker.exposedbydefault=false"
      - "--entrypoints.web.address=:80"
      - "--entrypoints.web.http.redirections.entryPoint.to=websecure"
      - "--entrypoints.web.http.redirections.entryPoint.scheme=https"
      - "--entrypoints.websecure.address=:443"
      - "--certificatesresolvers.letsencrypt.acme.httpchallenge=true"
      - "--certificatesresolvers.letsencrypt.acme.httpchallenge.entrypoint=web"
      - "--certificatesresolvers.letsencrypt.acme.email=${EMAIL}"
      - "--certificatesresolvers.letsencrypt.acme.storage=/letsencrypt/acme.json"
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - traefik_certs:/letsencrypt
    networks:
      - webproxy

  # 2. GITLAB (Sesuai Konfigurasi Final Anda)
  gitlab:
    image: gitlab/gitlab-ce:latest
    container_name: gitlab
    restart: always
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'https://${GITLAB_DOMAIN}'
        nginx['listen_port'] = 80
        nginx['listen_https'] = false
        nginx['proxy_set_headers'] = {
          "X-Forwarded-Proto" => "https",
          "X-Forwarded-Ssl" => "on"
        }
        gitlab_rails['gitlab_shell_ssh_port'] = ${GITLAB_SSH_PORT}
    ports:
      - "${GITLAB_SSH_PORT}:22"
    volumes:
      - '${GITLAB_HOME}/config:/etc/gitlab'
      - '${GITLAB_HOME}/logs:/var/log/gitlab'
      - '${GITLAB_HOME}/data:/var/opt/gitlab'
    networks:
      - webproxy
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.gitlab.rule=Host(`${GITLAB_DOMAIN}`)"
      - "traefik.http.routers.gitlab.entrypoints=websecure"
      - "traefik.http.routers.gitlab.tls.certresolver=letsencrypt"
      - "traefik.http.services.gitlab.loadbalancer.server.port=80"
      - "traefik.docker.network=webproxy"

  # 3. PORTAINER AGENT (Untuk Manajemen dari Server Utama)
  portainer-agent:
    image: portainer/agent:latest
    container_name: portainer_agent
    restart: always
    ports:
      - "9001:9001"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /var/lib/docker/volumes:/var/lib/docker/volumes
    networks:
      - webproxy

# Definisi Volume Penyimpanan
volumes:
  traefik_certs:

# Definisi Jaringan Terpusat
networks:
  webproxy:
    external: true
```
### 3. Buat Network
```bash
docker network create webproxy
```
### 4. Jalankan docker compose
```bash
docker compose up -d
```
