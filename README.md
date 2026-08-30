# WordPress Docker Starter Kit (Nginx + PHP-FPM Edition)

A **100% automated, high-performance** WordPress Docker Compose starter kit powered by **Nginx (Alpine) + WordPress PHP-FPM (Alpine)**.

Simply configure `.env` and run `docker compose up -d`. The system inside Docker automatically handles database health checks, file initialization, Nginx FastCGI routing, and WordPress installation via WP-CLI!

---

## ⚡ Architecture Overview

```
[ Browser / Client ]
         │ (Port ${HTTP_PORT})
         ▼
 ┌───────────────┐      FastCGI (port 9000)     ┌───────────────────────┐
 │  Nginx Web    │ ───────────────────────────> │  WordPress (PHP-FPM)  │
 │ (nginx:alpine)│                              │(wordpress:fpm-alpine) │
 └───────────────┘                              └───────────────────────┘
         │                                                  │
         │ (Static Assets: CSS, JS, Images)                 │ (MySQL TCP: 3306)
         ▼                                                  ▼
 ┌───────────────┐                              ┌───────────────────────┐
 │ Local Volume  │                              │  Database (MariaDB)   │
 │ (/wordpress)  │                              │    (mariadb:10.11)    │
 └───────────────┘                              └───────────────────────┘
```

* **Nginx (`nginx:alpine`)**: Menangani static assets (gambar, CSS, JS) secara langsung dan super cepat dengan browser caching, gzip compression, dan security rules.
* **WordPress (`wordpress:fpm-alpine`)**: Menjalankan eksekusi PHP murni via FastCGI tanpa overhead Apache, sangat hemat memory (RAM) & CPU.
* **MariaDB (`mariadb:10.11`)**: Database relasional cepat dan andal.
* **Auto-Installer (`wordpress:cli`)**: Melakukan instalasi otomatis WordPress begitu environment siap.

---

## 🚀 How to Use

### 1. Copy the Template Folder
Copy folder `wp-starter-nginx` ke project baru Anda:
```bash
cp -r wp-starter-nginx my-new-project
cd my-new-project
```

### 2. Configure `.env`
Copy `.env.example` ke `.env` (jika belum ada), lalu sesuaikan nilainya:
```dotenv
COMPOSE_PROJECT_NAME=my-new-project
HTTP_PORT=8080
PMA_PORT=8081
WP_TITLE="My Awesome Website"
WP_URL=http://localhost:8080
WP_ADMIN_USER=admin
WP_ADMIN_PASSWORD=SecretPassword123!
WP_ADMIN_EMAIL=admin@example.com
```

### 3. Run Docker Compose
Jalankan satu perintah ini di terminal:
```bash
docker compose up -d
```

🎉 **Selesai!**
Container `wp-auto-install` di dalam Docker akan secara otomatis:
1. Menunggu database MariaDB berstatus *healthy*.
2. Menunggu core files WordPress dan `wp-config.php` terbuat.
3. Menjalankan `wp core install` otomatis.
4. Berhenti dengan aman (*graceful exit*) tanpa membebani RAM/CPU.

---

## 🌐 Accessing the Services

- **WordPress Site**: `http://localhost:<HTTP_PORT>` (contoh: `http://localhost:8080`)
- **WordPress Admin**: `http://localhost:<HTTP_PORT>/wp-admin`
- **phpMyAdmin**: `http://localhost:<PMA_PORT>` (contoh: `http://localhost:8081`)

---

## 🛠️ Useful Commands

- **Check container status**: `docker compose ps`
- **View auto-installation logs**: `docker logs <COMPOSE_PROJECT_NAME>-auto-install`
- **Stop containers**: `docker compose stop`
- **Remove containers (data database & file web tetap aman)**: `docker compose down`
- **Run manual WP-CLI commands**:
  ```bash
  docker compose run --rm --entrypoint wp wp-auto-install plugin list
  docker compose run --rm --entrypoint wp wp-auto-install theme install generatepress --activate
  ```
- **Backup WordPress (`wp-content` + Database SQL)**:
  ```bash
  ./scripts/wp-backup.sh
  ```
- **Reset Environment (Hapus database lokal & file WordPress)**:
  ```bash
  # Standard reset (wordpress/ & db_data/)
  ./scripts/wp-reset.sh

  # Full reset (termasuk backups/ & .env)
  ./scripts/wp-reset.sh --all
  ```
