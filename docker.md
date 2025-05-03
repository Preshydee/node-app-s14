
````markdown
# 🛠️ WordPress Deployment on `devops-s14.afex.dev` with Docker, Nginx, and Certbot

This project is a fully containerised WordPress setup deployed at **https://devops-s14.afex.dev**. Everything runs via a single `docker-compose.yml` file, including:

- WordPress and Mysql
- Nginx (reverse proxy)
- Certbot (SSL management via Let's Encrypt)

The goal was to automate the deployment of a production-grade WordPress instance with HTTPS and a custom domain.

---

## ⚙️ Overview

**Services (via Docker Compose):**
- `WordPress`: The WordPress app
- `mysql`: MySQL database
- `nginx`: Reverse proxy that routes traffic and handles TLS termination
- `certbot`: Issues and renews SSL certificates

**Volumes:**
- For persistent MySQL data
- For SSL certificates and challenge files
- For shared use between Certbot and Nginx

**Domain:** `devops-s14.afex.dev`  
**DNS Setup:** A record pointing to the server's public IP

---

## 🧾 What I Did

### 1. 📁 Environment Configuration (`.env`)

Defined secrets and runtime variables in a `.env` file:
```env
MYSQL_ROOT_PASSWORD=supersecret
MYSQL_DATABASE=wordpress
MYSQL_USER=wp_user
MYSQL_PASSWORD=wp_password
WORDPRESS_DB_HOST=db:3306
WORDPRESS_DB_NAME=wordpress
WORDPRESS_DB_USER=wp_user
WORDPRESS_DB_PASSWORD=wp_password
DOMAIN=devops-s14.afex.dev
EMAIL=admin@afex.dev
````

### 2. 🧱 Built the `docker-compose.yml`

Everything runs inside one file. Services include:

* WordPress app container
* MySQL container
* Nginx container configured to serve WordPress and Certbot challenges
* Certbot container to issue HTTPS certificates using the webroot method

All services are connected via the same Docker network and volumes.

---

### 3. 🌐 Domain Setup

Configured the A record for `devops-s1.afex.dev` to point to my server’s IP:

| Host  | Type | Value         |
| ----- | ---- | ------------- |
| `@`   | A    | `<server IP>` |
| `www` | A    | `<server IP>` |

---


Certificates are saved in a mounted volume shared with the Nginx container.

---

### 5. 🔄 Auto-Renewal

Renewal is handled by a scheduled job (e.g., cron or containerized scheduler):

```bash
docker compose run --rm certbot renew
docker compose exec nginx nginx -s reload
```

This ensures HTTPS certificates remain valid without downtime.

---

### 6. 🧩 Nginx Config (Mounted into Container)

Nginx reverse proxies requests to the WordPress container and serves SSL using the certs from Certbot:

```nginx
server {
    listen 80;
    server_name devops-s1.afex.dev;

    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

    location / {
        return 301 https://$host$request_uri;
    }
}

server {
    listen 443 ssl;
    server_name devops-s1.afex.dev;

    ssl_certificate /etc/letsencrypt/live/devops-s1.afex.dev/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/devops-s1.afex.dev/privkey.pem;

    location / {
        proxy_pass http://wordpress/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

---

## 📁 File Structure

```
wordpress-stack/
├── docker-compose.yml
├── .env
├── nginx/
│   └── devops-s1.afex.dev.conf
├── certbot/
│   └── webroot/
└── volumes/
    └── letsencrypt/
```

---

## ✅ Outcome

* WordPress is live and secure at `https://devops-s1.afex.dev`
* All traffic is routed through Nginx
* SSL is valid and auto-renews
* Deployment is reproducible and managed with Docker Compose

---

## 💡 Notes

* Certbot and Nginx share volumes to allow SSL validation and renewal.
* Everything is managed inside a single Docker Compose stack for simplicity and portability.
* Ensured  ports 80 and 443 are open on your server/firewall.

---


