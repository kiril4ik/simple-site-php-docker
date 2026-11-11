# Simple multi-site PHP hosting (Coolify)

```
https://test-website.example.com  ->  /srv/sites/test-website  (host)  ->  /var/www/test-website
https://my-website.example.com    ->  /srv/sites/my-website    (host)  ->  /var/www/my-website
```

Services: `web` (nginx), `php` (PHP 8.5-FPM + composer), `mysql` (9.7 LTS), `phpmyadmin`.

- Folder name = first subdomain label (`www.` ignored).
- If `<site>/public/` exists, it becomes the docroot (Laravel/Symfony).
- Unknown subdomain or missing folder → 404.
- Dotfiles (`.env`, `.git`) and `*.sql/*.log/composer.json` are blocked.
- PHP gets `HTTPS=on` behind Traefik, real client IP, `SITE_NAME`, `DB_HOST=mysql`.
- Per-site PHP settings: put a `.user.ini` in the docroot (e.g. `memory_limit = 512M`).

## Deploy in Coolify

1. Push this repo to Git → Coolify → New Resource → your repo → Build Pack **Docker Compose**.
2. Environment variables (optional): `SITES_PATH=/srv/sites` (host folder), `TZ=Europe/Kyiv`.
3. Domains:
   - **web**: `https://test-website.example.com,https://my-website.example.com`
   - **phpmyadmin**: `https://pma.example.com`
4. DNS: `A *.example.com -> server IP` (wildcard), so new subdomains need no DNS change.
5. Deploy. MySQL root password = `SERVICE_PASSWORD_MYSQLROOT` (shown in Coolify env vars).

On the server: `sudo mkdir -p /srv/sites && sudo chown -R 82:82 /srv/sites`
(82 = `www-data` in Alpine PHP image; needed for uploads/cache writes).

## Add a new website

1. `mkdir /srv/sites/new-site` and upload files.
2. Coolify → web → Domains → append `,https://new-site.example.com` → Redeploy (only Traefik labels change).
3. Create DB + user in phpMyAdmin (or SQL below):

```sql
CREATE DATABASE new_site CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'new_site'@'%' IDENTIFIED BY 'strong-password';
GRANT ALL ON new_site.* TO 'new_site'@'%';
```

In the site config use host `mysql`, port `3306`.

## Local run

```bash
cp .env.example .env && mkdir -p sites/demo && echo '<?php phpinfo();' > sites/demo/index.php
docker compose up -d --build
# add a port mapping or curl inside: docker compose exec web wget -qO- --header 'Host: demo.localhost' http://127.0.0.1/
```
