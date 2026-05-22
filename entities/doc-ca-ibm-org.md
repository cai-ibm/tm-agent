---
title: Paperless-ngx (doc.ca-ibm.org)
created: 2026-05-22
updated: 2026-05-22
type: entity
tags: [server, paperless, documents, ocr, storage, cifs, hetzner]
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

## История изменений

- **22.05.2026** — Установка Paperless (на сервере 94.130.51.161)
- **22.05.2026** — Перенос media и consume на StorageBox (sub-account u488607-sub1)
