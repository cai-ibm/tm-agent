# nginx-proxy-manager (94.130.51.161)

## Схема

- **94.130.51.161** (Ubuntu 24.04, Hetzner)
- **NPM (jc21/nginx-proxy-manager:latest)** в Docker с `network_mode: host`
- Системный nginx (apt) **остановлен/отключён** (systemctl disable nginx)
- Веб-интерфейс NPM: `http://94.130.51.161:81`
- Admin: `sobolevrv@ca-ibm.org` / `changeme` → **сменить пароль!**

## Proxy hosts (все с LE сертификатами)

| Домен | Upstream | Порты | Статус |
|-------|----------|-------|--------|
| **n8n.ca-ibm.org** | 127.0.0.1:5678 (n8nio/n8n) | 80→443 LE | ✅ Up 8d |
| **doc.ca-ibm.org** | 127.0.0.1:8010 (paperless-ngx) | 80→443 LE | ✅ Up 8d |
| **ofds.ca-ibm.org** | 127.0.0.1:8080 (OnlyOffice) | 80→443 LE | ✅ Up 8d |

## Docker Compose stacks

| Стек | Путь | Запущен? |
|------|------|----------|
| NPM | `/opt/nginx-proxy-manager/docker-compose.yml` | ✅ |
| n8n | `/opt/n8n/docker-compose.yml` | ✅ |
| Paperless | `/opt/paperless/docker-compose.yml` | ✅ |
| OnlyOffice | `/opt/onlyoffice/docker-compose.yml` | ✅ |
| ~~OpenConstructionERP~~ | ~~/opt/OpenConstructionERP/~~ | ❌ Удалён |

## Firewall (UFW active)

**INPUT policy: DROP**

Разрешённые порты (внешние):
- `22/tcp` (SSH) — с **188.124.56.122** и **94.130.51.188**
- `80/tcp` (HTTP) — Anywhere
- `443/tcp` (HTTPS) — Anywhere
- `6160/tcp` (Veeam Deployment) — Anywhere
- `6162/tcp` (Veeam Transport) — Anywhere

Слушающие публичные порты:
- **80, 443, 81** — NPM (nginx внутри контейнера)
- **8080** — OnlyOffice (напрямую, не через NPM)
- **3000** — Node.js бэкенд NPM
- **6160, 6162** — Veeam

⚠️ Заметки по firewall:
- Порт **8080** напрямую открыт наружу — OnlyOffice доступен минуя NPM. Можно закрыть в UFW, так как проксируется через NPM
- Порт **6160/6162** Veeam — видимо нужны для бэкапов
- Порт **81** (NPM admin) — открыт всем. Если надо — ограничить по IP