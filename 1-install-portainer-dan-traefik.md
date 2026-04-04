## 1. Install Docker, Portainer dan Traefik
### 1. Buat Folder dan install docker
```bash
curl -fsSL https://get.docker.com -o get-docker.sh
```
```bash
sudo sh get-docker.sh
```
```bash
sudo usermod -aG docker $USER
```
```bash
mkdir portainer
```
```bash
cd portainer
```

### 2. Buat file YAML
```bash
sudo nano docker-compose.yml
```

#### Silakan tulis dengan konfigurasi di bawah ini
```bash
services:
  traefik:
    image: traefik:latest
    container_name: traefik
    restart: always
    networks:
      - webproxy
    command:
      - "--api.insecure=false"
      - "--providers.docker=true"
      - "--providers.docker.exposedbydefault=false"
      # Entrypoints
      - "--entrypoints.web.address=:80"
      - "--entrypoints.web.http.redirections.entryPoint.to=websecure"
      - "--entrypoints.web.http.redirections.entryPoint.scheme=https"
      - "--entrypoints.websecure.address=:443"
      # Let's Encrypt
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

  portainer:
    image: portainer/portainer-ce:latest
    container_name: portainer
    restart: always
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

networks:
  webproxy:
    external: true
```

### 3. Buat file .env
```bash
sudo nano .env
```
```bash
DOMAIN=sub.domain.com
EMAIL=your.email@mail.com
```

### 4. Buat Network External
```bash
sudo docker network create webproxy
```

### 5. Jalankan docker compose
```bash
sudo docker compose up -d
```
