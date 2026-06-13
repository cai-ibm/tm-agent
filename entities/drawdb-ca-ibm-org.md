---
title: drawDB (drawdb.ca-ibm.org)
aliases: [drawDB, drawdb]
created: 2026-06-13
updated: 2026-06-13
type: entity
tags: [service, docker, database, erd, sql]
sources: []
confidence: high
---

# drawDB (drawdb.ca-ibm.org)

Бесплатный онлайн-редактор ER-диаграмм и SQL-генератор. Запущен в Docker на [[server-94-130-51-161|Докер1]] за Nginx Proxy Manager.

## Основные параметры

| Параметр | Значение |
|----------|----------|
| **URL** | https://drawdb.ca-ibm.org/ |
| **Репозиторий** | https://github.com/drawdb-io/drawdb |
| **Лицензия** | Apache 2.0 (NVIDIA SkillSpector), свободная |
| **Тип** | Static SPA (React + Vite), сервится через nginx |

## Инфраструктура

| Параметр | Значение |
|----------|----------|
| **Сервер** | [[server-94-130-51-161|Докер1]] (94.130.51.161) |
| **Стек** | `/opt/drawdb/docker-compose.yml` |
| **Контейнер** | `drawdb` — собран из исходников |
| **Внутренний порт** | `127.0.0.1:3001` → 80 (в контейнере) |
| **Restart policy** | `unless-stopped` |
| **Прокси** | NPM → drawdb.ca-ibm.org (Let's Encrypt) |

## Домен и NPM

- **Домен:** `drawdb.ca-ibm.org`
- **NPM Proxy Host ID:** 7
- **NPM SSL Cert ID:** 20 (Let's Encrypt)
- **SSL Forced:** ✅
- **HTTP/2:** ✅

## Возможности

- Визуальный редактор ER-диаграмм в браузере
- Поддержка SQL-генерации (экспорт/импорт)
- Миграции
- Кастомизация редактора
- **Без регистрации** — всё в браузере
- Презентационный режим
- Diagram versioning

## Поддерживаемые СУБД

- MySQL / MariaDB
- PostgreSQL
- SQLite
- Microsoft SQL Server
- Oracle SQL

## Развёртывание

Клонирован репозиторий → `docker compose build` → `docker compose up -d` на Докер1.

**Построено:** 2026-06-13
**Теги:** нет (только последний коммит из main)

## Связанные сущности

- [[server-94-130-51-161]] — сервер, на котором работает drawDB
- [[nginx-pm-192-168-2-31]] — Nginx Proxy Manager