---
title: Twenty CRM
created: 2026-06-09
updated: 2026-06-09
type: entity
tags: [crm, twenty, coolify, docker, postgres]
sources: []
confidence: high
---

# Twenty CRM

**Twenty** — open-source CRM (альтернатива Salesforce), TypeScript-монорепа на NestJS + React + GraphQL. 49.5k звёзд на GitHub.

## Развёртывание

| Параметр | Значение |
|----------|----------|
| **Статус** | ✅ Опубликован через NPM (twenty.ca-ibm.org) |
| **Сервер** | docker-host2 (192.168.2.31) |
| **Метод** | Docker Compose вручную (`/opt/twenty/docker-compose.yml`) |
| **Публичный доступ** | `http://twenty.ca-ibm.org` (через NPM на Докер1) |
| **Путь к compose** | `/opt/twenty/` |
| **NPM proxy host ID** | 10 |
| **Coolify UUID** | `jj2aht57z9ngybhk96zhwzcd` (предыдущие: `aj2...`, `mdq...`, `b5e...`) |

## Контейнеры (Docker Compose)

### twenty-server-1
| Параметр | Значение |
|----------|----------|
| **Образ** | `twentycrm/twenty:latest` |
| **Порт** | `0.0.0.0:3000->3000/tcp` |
| **Статус** | Up |

### twenty-redis-1
| Параметр | Значение |
|----------|----------|
| **Образ** | `redis:7` |
| **Статус** | Up (healthy) |

## Переменные окружения server

| Переменная | Значение |
|-----------|----------|
| NODE_PORT | 3000 |
| PG_DATABASE_URL | `postgres://twenty:***@192.168.2.34:5433/twenty` |
| REDIS_URL | `redis://redis:6379` |
| SERVER_URL | `http://server:3000` |
| ENCRYPTION_KEY | `twenty2024-encryption-key-32chars!!` |

## База данных

| Параметр | Значение |
|----------|----------|
| **Тип** | Supabase PG (192.168.2.34) |
| **Порт** | 5433 (прямой socat-туннель, минуя supavisor) |
| **База** | `twenty` |
| **Пользователь** | `twenty` |
| **Пароль** | `twenty2024` |
| **Строка** | `postgres://twenty:***@192.168.2.34:5433/twenty` |

### Socat-туннель
- Контейнер: `pg-direct-tunnel` на 192.168.2.34
- Образ: `alpine/socat`
- Проброс: порт 5433 → TCP:supabase-db:5432
- Сеть: `supabase_default`

## Доступ

- **Локально:** `http://192.168.2.31:3000`
- **Публично:** `http://twenty.ca-ibm.org` (через NPM на Докер1)
- **NestJS** успешно стартует, миграции выполнены, 23 cron job зарегистрированы
- При первом входе нужно зарегистрировать workspace

## История изменений
- **15.06.2026** — Опубликован через NPM на Докер1: `http://twenty.ca-ibm.org` (proxy host id=10, HTTP без SSL)
- **09.06.2026 17:00** — Запущен вручную через Docker Compose (`/opt/twenty/`). HTTP 200.
- **09.06.2026** — Попытки развернуть через Coolify API (3 итерации). Неудачно — маскировка пароля.
- **09.06.2026** — Создан pg-direct-tunnel (порт 5433 → supabase-db:5432).
- **09.06.2026** — Создана БД `twenty`, пользователь `twenty` на Supabase PG.

## Связанные страницы

[[coolify-192-168-2-39]], [[server-94-130-51-161]]
