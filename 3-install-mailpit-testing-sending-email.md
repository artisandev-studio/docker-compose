## Install Mailpit
### 1. Buat Folder
```bash
mkdir maildev
```
### 2. Buat file .env
```bash
sudo nano .env
```
Diisi dengan :

Note :
Untuk password bisa di buat dengan cara 
```bash
docker run --rm httpd:alpine htpasswd -nb admin password123 | sed -e 's/\$/\$\$/g'
```

```bash
MAIL_DOMAIN=maildev.domainkamu.com
MAIL_AUTH=admin:Password_Kamu_Yang_Sudah_Di_Hash
```
### 3. Buat docker-compose.yml
```bash
sudo nano docker-compose.yml
```
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
### 5. Test Seending
## Online SMTP Tester
## Inline code
Menggunakan Website "Online SMTP Tester"
Ada beberapa website yang bisa mengirim email percobaan ke server SMTP kustom. Salah satu yang paling stabil adalah Smtper.net.
Cara Mengisinya:
```
SMTP Host: maildev.domain.com (atau IP VPS Anda)
```
```
Port: 8100
```
```
SSL/TLS: No (pilih None karena Mailpit di port 8100 biasanya tanpa enkripsi SMTP, enkripsi sudah diurus Traefik hanya untuk Dashboard).
```
```
Authentication: No (kosongkan, karena Mailpit secara default menerima kiriman tanpa password).
```
```
Sender/Receiver: Isi bebas (misal: test@pma.id ke saya@domain.com).
```

Note :
Digunakan Project Laravel
```bash
MAIL_MAILER=smtp
MAIL_HOST=ip_vps_anda_atau_domain
MAIL_PORT=8100
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
MAIL_FROM_ADDRESS="admin@domain.com"
MAIL_FROM_NAME="${APP_NAME}"
```
