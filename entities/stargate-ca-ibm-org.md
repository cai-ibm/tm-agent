---
title: stargate.ca-ibm.org (Nextcloud)
created: 2026-05-15
updated: 2026-06-20
type: entity
tags: [server, nextcloud, onlyoffice, service, storage, database, apache, letsencrypt, redis, mariadb]
sources: [raw/memory/agent-knowledge-2026-05-15.md]
confidence: high
---

# stargate.ca-ibm.org — Nextcloud Server (gate-cloud)

Корпоративное облачное хранилище на Nextcloud. Выделенный узел Hetzner.

## Основные параметры

| Параметр | Значение |
|----------|----------|
| **FQDN** | stargate.ca-ibm.org |
| **IP** | 94.130.51.147 (внешний), 192.168.2.42 (локальный) |
| **Хостнейм** | gate-cloud |
| **ОС** | Ubuntu 24.04.3 LTS |
| **RAM** | 7.8 GiB |
| **Диск /** | 638 ГБ (35 ГБ занято, 6%) |
| **Swap** | 4 GiB |
| **Uptime** | 184 дня (с декабря 2025) |
| **Провайдер** | Hetzner |

## Стек ПО

| Компонент | Версия |
|-----------|--------|
| **Nextcloud** | 32.0.9.2 |
| **Web-сервер** | Apache 2.4 + mod_ssl |
| **PHP** | 8.3.6 (FPM) |
| **MariaDB** | 10.11.13 (localhost:3306) |
| **Redis** | localhost:6379 |
| **Кеш** | APCu (local), Redis (locking) |
| **SSL** | Let's Encrypt (ECDSA, до 2026-07-16) |

## Веб-сервер (Apache2)

- **HTTP** — порт 80, редирект → HTTPS
- **HTTPS** — порт 443, сертификат Let's Encrypt
- **DocumentRoot:** `/var/www/nextcloud`
- **Конфиги:** `/etc/apache2/sites-enabled/nextcloud.conf`
- **SSL:** `/etc/letsencrypt/live/stargate.ca-ibm.org/`

Nextcloud.conf:
```
<VirtualHost *:80>
    ServerName stargate.ca-ibm.org
    DocumentRoot /var/www/nextcloud
    RewriteEngine on
    RewriteCond %{SERVER_NAME} =stargate.ca-ibm.org
    RewriteRule ^ https://%{SERVER_NAME}%{REQUEST_URI} [END,NE,R=permanent]
</VirtualHost>
<IfModule mod_headers.c>
  Header always set Strict-Transport-Security "max-age=15552000; includeSubDomains"
</IfModule>
```

SSL-конфиг включает Timeout 600, KeepAlive On.

## Мониторинг

### Лог-файл (19.06.2026)

- **Размер лога:** 17 ГБ (`/var/www/nextcloud/data/nextcloud.log`)
- Анализ последних 500+ строк через SSH с `Superp@ss2020root` — сотни `dirty table reads` в MariaDB
- Лог растёт: за последние 48 часов с ~72 MB → 101 MB → 17 ГБ (резкий скачок вечером 18.06)
- **Проблема:** стандартный Python-скрипт анализа зависает при чтении всего файла

### Доступность сервера (20.06.2026)

- **08:00 UTC** — SSH таймаут (60с), сервер не отвечает по SSH
- Причины: сервер выключен/перезагружается, смена IP, или SSH-демон завис
- Требуется ручная проверка доступности

## Хранилище

- **S3 Object Storage** (Hetzner) — `fsn1.your-objectstorage.com`, bucket `cai-s3-1`
- **Регион:** eu-central
- **use_path_style:** true
- **~2.5 млн файлов**

## Кеширование

- **local:** `\OC\Memcache\APCu`
- **locking:** `\OC\Memcache\Redis` (localhost:6379)

## Приложения (сторонние)

| Приложение | Версия | Назначение |
|------------|--------|------------|
| collectives | 4.4.0 | Совместные wiki-страницы |
| contacts | 8.3.9 | Контакты |
| external | 7.0.1 | Внешние сайты в меню |
| groupfolders | 20.1.13 | Групповые папки |
| onlyoffice | 9.13.0 | Документы (интеграция) |
| recognize | 10.0.7 | AI-тегирование фото |

## OnlyOffice интеграция

- **Плагин:** OnlyOffice v9.13.0 (v10 требует Nextcloud 33 — несовместим)
- **JWT:** задан (секрет KNJYTbUI^BRfkub654)
- **Document Server URL:** `https://ofds.ca-ibm.org/`
- **Storage URL:** не задан (используется по умолчанию)

### Известная проблема: .docx открываются read-only

Файлы `.docx` открываются в режиме «только чтение» при редактировании через OnlyOffice. Причина на момент 08.05.2026 не найдена. `.xlsx` редактируются нормально.

См. навык Hermes: `onlyoffice-troubleshooting` — диагностика и исправление проблем интеграции OnlyOffice.

## SMTP

- **Режим:** smtp
- **Порт:** 2526
- Отправка через [[smtp-relay-ca-ibm]]

## Фоновые задачи

Режим: **cron** (установлен через `occ background:cron`)

## Конфигурация (ключевые параметры occ)

| Параметр | Значение |
|----------|----------|
| `trusted_domains` | stargate.ca-ibm.org |
| `overwrite.cli.url` | https://stargate.ca-ibm.org |
| `default_phone_region` | RU |
| `loglevel` | 0 (DEBUG) |
| `maintenance_window_start` | 22 |
| `versions_retention_obligation` | 3, 14 |
| `operationTimeout` | 600 |
| `transferTimeout` | 600 |
| `forbidden_filename_basenames` | con, prn, aux, nul, com0-9, lpt0-9 |
| `memories.exiftool` | /var/www/nextcloud/apps/memories/bin-ext/exiftool-amd64-glibc |
| `memories.vod.path` | /var/www/nextcloud/apps/memories/bin-ext/go-vod-amd64 |
| `memories.db.triggers.fcu` | true |

## Команды управления (occ)

Путь: `/var/www/nextcloud/occ` (от root, через `sudo -u www-data php /var/www/nextcloud/occ`)

```bash
# Статус
sudo -u www-data php /var/www/nextcloud/occ status

# Конфиг
sudo -u www-data php /var/www/nextcloud/occ config:list system
sudo -u www-data php /var/www/nextcloud/occ config:app:list onlyoffice

# Обновление
sudo -u www-data php /var/www/nextcloud/occ upgrade

# Приложения
sudo -u www-data php /var/www/nextcloud/occ app:list --shipped=false
sudo -u www-data php /var/www/nextcloud/occ app:enable <app>
sudo -u www-data php /var/www/nextcloud/occ app:disable <app>

# Фоновые задачи
sudo -u www-data php /var/www/nextcloud/occ background:cron

# Пользователи
sudo -u www-data php /var/www/nextcloud/occ user:list

# Бэкап в maintenance
sudo -u www-data php /var/www/nextcloud/occ maintenance:mode --on
# ... backup ...
sudo -u www-data php /var/www/nextcloud/occ maintenance:mode --off

# OnlyOffice
sudo -u www-data php /var/www/nextcloud/occ config:app:get onlyoffice DocumentServerUrl
sudo -u www-data php /var/www/nextcloud/occ config:app:set onlyoffice jwt_secret --value=<secret>
```

## SSH-доступ (прямой)

```bash
sshpass -p 'Superp@ss2020root' ssh root@94.130.51.147
```

**⚠️ Парольная аутентификация включена**, ключи не настроены. Рекомендуется настроить key-based auth.

## Бэкап

- **Путь:** `/root/nextcloud-backup-20260508/`
- **Содержимое:**
  - `config.php` — конфиг Nextcloud
  - `nextcloud-db.sql.gz` — дамп MariaDB
  - `onlyoffice-app/` — настройки OnlyOffice
- Дата последнего бэкапа: 08.05.2026

**Рекомендуемая процедура бэкапа:**
```bash
# 1. Maintenance mode
sudo -u www-data php /var/www/nextcloud/occ maintenance:mode --on

# 2. Дамп БД
mysqldump --single-transaction nextcloud | gzip > /root/nextcloud-backup-$(date +%Y%m%d)/nextcloud-db.sql.gz

# 3. Копировать конфиг
cp /var/www/nextcloud/config/config.php /root/nextcloud-backup-$(date +%Y%m%d)/

# 4. Выключить maintenance
sudo -u www-data php /var/www/nextcloud/occ maintenance:mode --off
```

## Nginx-проксирование

Трафик к stargate.ca-ibm.org идёт через Nginx Proxy Manager на [[server-94-130-51-161]] (Hetzner 94.130.51.161) → проксируется на 94.130.51.147:443.

## Сеть

- Два интерфейса: внешний (94.130.51.147) и локальный (192.168.2.42)
- Локальный доступ: из сети CA-IBM через 192.168.2.42

## Связанные сущности

- [[server-94-130-51-161]] — Hetzner, nginx reverse proxy, OnlyOffice DS
- [[mail-ca-ibm-org]] — Exchange, почтовые уведомления
- [[smtp-relay-ca-ibm]] — SMTP-релей (порт 2526)

## История изменений

- **2026-06-10** — Полная инвентаризация сервера: IP (94.130.51.147, 192.168.2.42), Apache2, Redis, MariaDB 10.11.13, APCu, Let's Encrypt, приложения, SMTP (порт 2526), режим cron, бэкап, команды occ
- **2026-05-15** — Страница создана при инициализации LLM Wiki
- **08.05.2026** — Обновление Nextcloud 32.0.3 → 32.0.9, OnlyOffice плагин 9.13.0

## Связанные страницы

- [[onlyoffice-docx-readonly]]
