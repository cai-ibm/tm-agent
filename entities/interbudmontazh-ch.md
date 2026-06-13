---
title: interbudmontazh.ch (Coolify)
created: 2026-06-03
updated: 2026-06-03
type: entity
tags: [wordpress, coolify, service, docker]
sources: []
confidence: high
---

# interbudmontazh.ch — сайт на WordPress (Coolify)

Сайт-портфолио строительной компании. Мигрирован с fozzy.com на собственный сервер.

## Основные параметры

| Параметр | Значение |
|----------|----------|
| **Домен** | interbudmontazh.ch.ca-ibm.org |
| **IP** | 192.168.2.39 (Coolify) |
| **Платформа** | Coolify v4.1.1 |
| **CMS** | WordPress (wordpress:latest) |
| **БД** | MariaDB 11 |
| **Тема** | Porto (платная) |
| **Страниц** | 24 |
| **Название** | CAI interbudmontazh |
| **Пользователи** | 1 (adm) |
| **Протокол** | HTTP (без HTTPS, без редиректов) |

## Креды MariaDB

| Параметр | Значение |
|----------|----------|
| Пользователь | `5A4ZfAEOmLWWXAZG` |
| Пароль | `qZJ9scdXRvnyy74q1pVWULIEvdyrFt9N` |
| БД | `wordpress` |
| Root | `yP0WBG9XEtMncji0yXO3Y2cqZDP0xXfF` |
| UUID | `dvr9kjj972k9ofsxei5hjb6x` |

## Проксирование и редиректы

| Направление | Тип | Куда |
|-------------|-----|------|
| interbudmontazh.ch (любая страница) | 301 редирект | → https://interbudmontazh.com/ |
| interbudmontazh.ch.ca-ibm.org | Прямой доступ HTTP | → Caddy (Coolify на 192.168.2.39) |

**Прямой доступ без NPM**: `interbudmontazh.ch.ca-ibm.org` — Caddy на 192.168.2.39 принимает запросы по HTTP, без SSL, без редиректов.

SSL сертификат для `interbudmontazh.ch` есть (LE npm-18, до Sep 2026), используется только для 301 редиректа.

### Схема трафика

```
Пользователь → interbudmontazh.ch (любая страница)
  → NPM (94.130.51.161) 
  → 301 редирект → https://interbudmontazh.com/

Пользователь → interbudmontazh.ch.ca-ibm.org
  → Caddy (192.168.2.39)
  → WordPress container → MariaDB
```

## wp-config.php

Принудительное включение HTTPS (`$_SERVER['HTTPS'] = 'on'`) **удалено**. Сайт работает только по HTTP. В контенте остались старые ссылки на `https://interbudmontazh.ch/` (из Porto темы, элементов Elementor, меню), но сам движок не перенаправляет на HTTPS.

## Исходные данные (fozzy.com)

| Параметр | Значение |
|----------|----------|
| SSH | interbud@78.140.185.130 |
| БД (была) | interbud_cai |
| Пользователь БД | interbud_caiuser |
| Пароль БД | `^D1i}?o,];H}` |
| wp-content | 2.9 GB |

## Примечания

- На fozzy остались файлы, архив на 94.130.51.161 в /tmp/interbud_wp_content.tar.gz
- БД interbud_wp на 192.168.2.39 есть (старая, не в Coolify)
- Для запуска .com нужно создать отдельный ресурс в Coolify или домен в настройках

## Связанные сущности

- [[server-94-130-51-161]] — NPM
- [[coolify-192-168-2-39]] — Coolify сервер

## История изменений

- **2026-06-03** — Мигрирован с fozzy.com. Развёрнут в Coolify (UUID dvr9kjj972k9ofsxei5hjb6x). Добавлен домен interbudmontazh.ch.ca-ibm.org (только HTTP). wp-config: HTTPS принуждение удалено. 301 редирект .ch → .com через NPM.