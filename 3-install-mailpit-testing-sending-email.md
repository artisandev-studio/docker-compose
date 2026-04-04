### 1. Buat Folder
```bash
mkdir maildev
```
### 2. Buat file .env
```bash
sudo nano .env
```
Diisi dengan :
```bash
MAIL_DOMAIN=mailpit.voips.biz.id
MAIL_AUTH=admin:Password_Kamu_Yang_Sudah_Di_Hash
```
Note :
Untuk password bisa di buat dengan cara 
```bash
docker run --rm httpd:alpine htpasswd -nb admin password123 | sed -e 's/\$/\$\$/g'
```
### 3. Buat docker-compose.yml
```bahs
services:
  mailpit:
    image: axllent/mailpit:latest
    container_name: mailpit
    restart: always
    environment:
      - MP_MAX_MESSAGES=5000
      - MP_DATABASE=/data/mailpit.db
      # Port internal SMTP (untuk Laravel)
      - MP_SMTP_BIND_ADDR=0.0.0.0:1025
      # Port internal Web UI
      - MP_UI_BIND_ADDR=0.0.0.0:8025
    volumes:
      - ./data:/data
    networks:
      - webproxy
    ports:
      # Expose port SMTP ke VPS agar bisa diakses Laravel (Port 8100)
      # Format: Port_Luar:Port_Dalam_Container
      - "8100:1025"
    labels:
      - "traefik.enable=true"
      # Routing Dashboard via HTTPS
      - "traefik.http.routers.mailpit.rule=Host(`${MAIL_DOMAIN}`)"
      - "traefik.http.routers.mailpit.entrypoints=websecure"
      - "traefik.http.routers.mailpit.tls.certresolver=letsencrypt"
      # Middleware untuk Form Login (Basic Auth)
      - "traefik.http.routers.mailpit.middlewares=mailpit-auth"
      - "traefik.http.middlewares.mailpit-auth.basicauth.users=${MAIL_AUTH}"
      # Arahkan Traefik ke Port Web UI Mailpit (8025)
      - "traefik.http.services.mailpit.loadbalancer.server.port=8025"

networks:
  webproxy:
    external: true
```
4. Jalankan docker compose
```bash
docker compose up -d
```
