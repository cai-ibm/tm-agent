---
title: Paperless-ngx (doc.ca-ibm.org)
created: 2026-05-22
updated: 2026-05-22
type: entity
tags: [server, paperless, documents, ocr, storage, cifs, hetzner]
sources: []
confidence: high
---

# Paperless-ngx — doc.ca-ibm.org

## Расположение

| Параметр | Значение |
|---|---|
| **Сервер** | [[server-94-130-51-161]] (Hetzner, Ubuntu 24.04) |
| **Домен** | `https://doc.ca-ibm.org` |
| **Тип установки** | Docker Compose |
| **Директория** | `/opt/paperless/` |
| **Сеть** | `127.0.0.1:8010` → nginx reverse proxy |

## Хранилище — Hetzner StorageBox (CIFS)

С 22.05.2026 media и consume живут на StorageBox, а не локально.

| Директория | Назначение | Расположение |
|---|---|---|
| `/mnt/storagebox/consume/` | Consume (drop-zone для документов) | StorageBox sub-account `u488607-sub1` |
| `/mnt/storagebox/paperless-media/` | Media (обработанные документы) | StorageBox sub-account `u488607-sub1` |
| `/opt/paperless/data/` | SQLite+поисковый индекс | Локально (сервер) |
| `/opt/paperless/db/` | PostgreSQL БД | Локально (сервер) |

### StorageBox подключение

| Параметр | Значение |
|---|---|
| **Хост** | `//u488607-sub1.your-storagebox.de/u488607-sub1` |
| **Пользователь** | `u488607-sub1` |
| **Credentials** | `/etc/smbcredentials/storagebox.cred` |
| **Опции монтирования** | `iocharset=utf8,vers=3.0,nofail,x-systemd.automount,uid=1000,gid=1000,forceuid,forcegid,file_mode=0644,dir_mode=0755` |
| **fstab** | Есть (автомонтирование) |

### Права доступа

Внутри контейнера Paperless запускается с UID/GID = 1000 (через `USERMAP_UID`/`USERMAP_GID`). CIFS смонтирован с `uid=1000,gid=1000,forceuid,forcegid`, так что права совпадают.

## Сервисы Docker

| Контейнер | Образ | Статус |
|---|---|---|
| `paperless-webserver-1` | `ghcr.io/paperless-ngx/paperless-ngx:latest` | healthy |
| `paperless-broker-1` | `redis:7-alpine` | up |
| `paperless-db-1` | `postgres:16-alpine` | up |

## Конфигурация

### docker-compose.yml

```yaml
volumes:
  - /opt/paperless/data:/usr/src/paperless/data
  - /mnt/storagebox/paperless-media:/usr/src/paperless/media
  - /opt/paperless/export:/usr/src/paperless/export
  - /mnt/storagebox/consume:/usr/src/paperless/consume
```

**Environment:**
- `PAPERLESS_URL: https://doc.ca-ibm.org`
- `PAPERLESS_TIME_ZONE: Europe/Moscow`
- `PAPERLESS_OCR_LANGUAGE: rus+eng`
- `PAPERLESS_OCR_LANGUAGES: rus eng`
- `USERMAP_UID: 1000`
- `USERMAP_GID: 1000`
- `PAPERLESS_REDIS: redis://broker:6379`
- PostgreSQL: `paperless` / `paperless_pass_2026`

### Админка

- URL: `https://doc.ca-ibm.org`
- Логин: `admin`
- Пароль: ~~был сброшен в процессе~~ (пересоздать через `docker compose exec webserver python3 manage.py createsuperuser`, если нужно)

## Использование

Просто кинуть PDF/JPG/TXT в `/opt/paperless/consume/` (или на StorageBox в директорию `consume` через SMB) — Paperless сам подхватит, OCR-нет и добавит в коллекцию.

Можно монтировать StorageBox SMB-шару с Windows/Mac/Linux и кидать файлы прямо в `consume/` — Paperless подберёт их оттуда.

## Бэкап

Раз в день в 3:00 ночи через cron.

| Что бэкапится | Куда | Размер | Retention |
|---|---|---|---|
| PostgreSQL (метаданные, теги, OCR-текст) | `StorageBox/paperless-backups/db/` | ~200 KB | 30 дней |
| Поисковый индекс + настройки | `StorageBox/paperless-backups/data/` | ~200 MB | 30 дней |
| Файлы документов (PDF/изображения) | StorageBox `paperless-media/` | (CIFS — снимки делает Hetzner) | ∞ |

**Скрипт:** `/opt/paperless/backup-paperless.sh`
**Лог:** `/var/log/paperless-backup.log`
**Cron:** `0 3 * * * /opt/paperless/backup-paperless.sh >> /var/log/paperless-backup.log 2>&1`

### Восстановление

```bash
# 1. Восстановить БД
gunzip -c /mnt/storagebox/paperless-backups/db/paperless-db-YYYY-MM-DD_HH-MM-SS.sql.gz | \
  docker exec -i paperless-db-1 psql -U paperless paperless

# 2. Восстановить данные (поисковый индекс)
tar xzf /mnt/storagebox/paperless-backups/data/paperless-data-YYYY-MM-DD_HH-MM-SS.tar.gz \
  -C /opt/paperless

# 3. Перезапустить Paperless
cd /opt/paperless && docker compose restart webserver
```

Файлы документов на StorageBox живут отдельно — их восстанавливать не нужно, они уже там.

## История изменений

- **22.05.2026** — Установка Paperless (на сервере 94.130.51.161)
- **22.05.2026** — Перенос media и consume на StorageBox (sub-account u488607-sub1)
- **22.05.2026** — Настроен ежедневный бэкап БД+данных на StorageBox в 3:00 (retention 30 дней)

## Связанные страницы

[[stargate-ca-ibm-org]]
