---
title: stargate.ca-ibm.org (Nextcloud)
created: 2026-05-15
updated: 2026-05-15
type: entity
tags: [server, nextcloud, onlyoffice, service, storage, database, nginx]
sources: [raw/memory/agent-knowledge-2026-05-15.md]
confidence: high
---

# stargate.ca-ibm.org — Nextcloud Server

Корпоративное облачное хранилище на Nextcloud. Выделенный физический сервер или VM. Доступ через [[server-94-130-51-161]] (Hetzner nginx reverse proxy) по маршруту `stargate.ca-ibm.org → Nextcloud`.

## Основные параметры

| Параметр | Значение |
|----------|----------|
| **FQDN** | stargate.ca-ibm.org |
| **Сервис** | Nextcloud 32.0.9 (обновлён с 32.0.3 08.05.2026) |
| **Доступ** | root (система), adm (админ Nextcloud) |
| **БД** | MariaDB 10.11.13 |
| **PHP** | 8.3.6 |
| **Хранилище** | S3 Object Storage (Hetzner fsn1.your-objectstorage.com, bucket cai-s3-1) |
| **Файлов** | ~2.5 млн |

## Интеграция с OnlyOffice

- **Плагин:** OnlyOffice v9.13.0 (v10 требует Nextcloud 33 — несовместим)
- **JWT:** задан (секрет KNJYTbUI^BRfkub654)
- **Document Server:** [[server-94-130-51-161]] (ofds.ca-ibm.org)

### Известная проблема: .docx открываются read-only

Файлы `.docx` открываются в режиме «только чтение» при редактировании через OnlyOffice. Причина на момент 08.05.2026 не найдена. `.xlsx` редактируются нормально.

См. навык Hermes: `onlyoffice-troubleshooting` — диагностика и исправление проблем интеграции OnlyOffice.

## Бэкап

- **Путь:** `/root/nextcloud-backup-20260508/`
- Дата последнего бэкапа: 08.05.2026

## Nginx-проксирование

Трафик к stargate.ca-ibm.org идёт через nginx на [[server-94-130-51-161]] (Hetzner). Конфигурация nginx на сервере Hetzner содержит маршруты:
- `stargate.ca-ibm.org` → Nextcloud
- `n8n.ca-ibm.org` → n8n
- `ofds.ca-ibm.org` → OnlyOffice Document Server
- `ocerp.ca-ibm.org` → OpenConstructionERP

## Связанные сущности

- [[server-94-130-51-161]] — Hetzner, nginx reverse proxy, OnlyOffice DS
- [[mail-ca-ibm-org]] — Exchange, почтовые уведомления Nextcloud могут идти через [[smtp-relay-ca-ibm]]
- [[smtp-relay-ca-ibm]] — SMTP-релей для исходящей почты

## История изменений

- **2026-05-15** — Страница создана при инициализации LLM Wiki
- **08.05.2026** — Обновление Nextcloud 32.0.3 → 32.0.9, OnlyOffice плагин 9.13.0
