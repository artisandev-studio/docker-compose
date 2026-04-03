## Install Portainer dan Traefik
### 1. Buat Folder dan install docker
```bash
mkdir portainer
```
```bash
cd portainer
```
```bash
curl -fsSL https://get.docker.com -o get-docker.sh
```
```bash
sudo sh get-docker.sh
```
```bash
sudo usermod -aG docker $USER
```

### 2. Buat file YAML
```bash
docker-compose.yml
```
#### Silakan timpa file dengan konfigurasi di bawah ini
```bash
services:
  traefik:
    image: traefik:latest
    container_name: traefik
    restart: always
    # Menghubungkan Traefik ke jaringan eksternal
    networks:
      - webproxy
    command:
      - "--api.insecure=false"
      - "--providers.docker=true"
      - "--providers.docker.exposedbydefault=false"
      # Setup Entrypoints
      - "--entrypoints.web.address=:80"
      - "--entrypoints.web.http.redirections.entryPoint.to=websecure"
      - "--entrypoints.web.http.redirections.entryPoint.scheme=https"
      - "--entrypoints.websecure.address=:443"
      # Setup SSL menggunakan HTTP Challenge
      - "--certificatesresolvers.letsencrypt.acme.httpchallenge=true"
      - "--certificatesresolvers.letsencrypt.acme.httpchallenge.entrypoint=web"
      - "--certificatesresolvers.letsencrypt.acme.email=${EMAIL}"
      - "--certificatesresolvers.letsencrypt.acme.storage=/letsencrypt/acme.json"
    ports:
      # Port 80 wajib dibuka untuk jalur masuk verifikasi Let's Encrypt
      - "80:80"
      # Port 8800 untuk akses HTTPS custom Anda
      - "8800:443"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - traefik_certs:/letsencrypt

  portainer:
    image: portainer/portainer-ce:latest
    container_name: portainer
    restart: always
    # Menghubungkan Portainer ke jaringan eksternal yang sama
    networks:
      - webproxy
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - portainer_data:/data
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.portainer.rule=Host(`${DOMAIN}`)"
      - "traefik.http.routers.portainer.entrypoints=websecure"
      - "traefik.http.routers.portainer.tls.certresolver=letsencrypt"
      - "traefik.http.services.portainer.loadbalancer.server.port=9000"

volumes:
  portainer_data:
  traefik_certs:

# Mendefinisikan jaringan eksternal
networks:
  webproxy:
    external: true
```
### 3. Buat file .env
```bash
DOMAIN=sub.domain.com
EMAIL=your.email@mail.com
```
