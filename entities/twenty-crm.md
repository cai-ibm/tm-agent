---
title: Twenty CRM
created: 2026-06-09
updated: 2026-06-09
type: entity
tags: [crm, twenty, coolify, service, docker]
sources: []
confidence: high
---

# Twenty CRM

**Twenty** — open-source CRM (альтернатива Salesforce), TypeScript-монорепа на NestJS + React + GraphQL. 49.5k звёзд на GitHub.

## Развёртывание

| Параметр | Значение |
|----------|----------|
| **Статус** | Развёрнут через Coolify (HTTP 201) |
| **Coolify UUID** | `aj2wfwftl3bav589z3jjq1it` |
| **Сервер** | docker-host2 (192.168.2.31, uuid: `b14cjjl628nbf5z15r2vwxg9`) |
| **Проект** | CAI (`vvz540r0j7qaa4i9ongoq5dt`) → production |
| **Тип** | Сервис (docker-compose) |

## Контейнеры

### server
| Параметр | Значение |
|----------|----------|
| **Образ** | `twentycrm/twenty:latest` |
| **Порт** | 3000 |
| **Домен** | (будет присвоен Coolify) |

### redis
| Параметр | Значение |
|----------|----------|
| **Образ** | `redis:7` |
| **Назначение** | Очереди и кеш |

## База данных (внешняя)

| Параметр | Значение |
|----------|----------|
| **Хост** | 192.168.2.34 (Supabase PG) |
| **Порт** | 5432 |
| **База** | `twenty` |
| **Пользователь** | `twenty` |
| **Пароль** | `twenty2024` |
| **Строка** | `postgres://twenty:***@192.168.2.34:5432/twenty` |

## Доступ

Адрес: `http://192.168.2.31:3000` (после завершения деплоя)

## История изменений
- **09.06.2026** — Развёрнут через Coolify API (POST /api/v1/services). Сервис UUID: `aj2wfwftl3bav589z3jjq1it`.
