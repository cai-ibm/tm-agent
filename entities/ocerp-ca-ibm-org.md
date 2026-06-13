---
title: OpenConstructionERP (ocerp.ca-ibm.org) — удалён
created: 2026-05-25
updated: 2026-05-26
type: entity
tags: ["erp", "docker", "construction", "openconstructionerp", "deleted"]
sources: []
confidence: high
---
# ocerp.ca-ibm.org

**Статус:** УДАЛЁН 26.05.2026 (оба инстанса)

## Инстанс 1: 192.168.2.34 (cai@192.168.2.34)

Удалён утром 26.05.2026.

**Удалено:**
- Docker контейнеры: `openconstructionerp-app-1`, `openconstructionerp-postgres-1`
- Docker volumes: `app_data`, `pg_data`
- Docker network: `openconstructionerp_default`
- Nginx конфиг: `/etc/nginx/sites-available/openconstructionerp` (symlink из `sites-enabled`)
- Let's Encrypt сертификат для `ocerp.ca-ibm.org` (live, archive, renewal)
- Репозиторий: `/home/cai/OpenConstructionERP/`
- nginx перезагружен

**История:** v2.9.0 → переключён на v4.12.0, сборка не завершилась. Причина — больше не нужен.

## Инстанс 2: 94.130.51.161 (root@94.130.51.161)

Развёрнут 25.05.2026 на v4.9.1, порт 8082 (8080 занят OnlyOffice). Удалён 26.05.2026.

**Удалено:**
- Docker контейнеры: `openconstructionerp-app-1`, `openconstructionerp-postgres-1`
- Docker volumes: `app_data`, `pg_data`
- Docker image: `openconstructionerp-app`
- Docker network: `openconstructionerp_default`
- Nginx конфиг: `/etc/nginx/sites-available/ocerp` + `/etc/nginx/sites-enabled/ocerp`
- Nginx: `/etc/nginx/conf.d/large_headers.conf` (был создан для OCERP JWT)
- Let's Encrypt сертификат для `ocerp.ca-ibm.org` (live, archive, renewal)
- Репозиторий: `/opt/OpenConstructionERP/`
- nginx перезагружен

**История:** v4.9.1 → переключён на v4.12.0. Сборка падала из-за TS-ошибки в тестах (formatters.units.test.ts — импорт удалённых функций). Исправлено добавлением exclude в tsconfig. Не завершено — пользователь решил удалить.

## Связанные страницы

[[server-94-130-51-161]], [[mail-ca-ibm-org]]
