---
title: interbudmontazh.ch (Coolify)
created: 2026-06-03
updated: 2026-06-03
type: entity
tags: [wordpress, coolify, service, docker]
confidence: high
---

# interbudmontazh.ch — сайт на WordPress (Coolify)

Сайт-портфолио строительной компании. Мигрирован с fozzy.com на собственный сервер.

## Основные параметры

| Параметр | Значение |
|----------|----------|
| **Домен** | interbudmontazh.ch |
| **IP** | 192.168.2.39 (Coolify) |
| **Платформа** | Coolify v4.1.1 |
| **CMS** | WordPress (wordpress:latest) |
| **БД** | MariaDB 11 |
| **Тема** | Porto (платная) |
| **Страниц** | 24 |
| **Название** | CAI interbudmontazh |
| **Пользователи** | 1 (adm) |

## Креды MariaDB

| Параметр | Значение |
|----------|----------|
| Пользователь | `5A4ZfAEOmLWWXAZG` |
| Пароль | `qZJ9scdXRvnyy74q1pVWULIEvdyrFt9N` |
| БД | `wordpress` |
| Root | `yP0WBG9XEtMncji0yXO3Y2cqZDP0xXfF` |
| UUID | `dvr9kjj972k9ofsxei5hjb6x` |

## Проксирование через NPM (94.130.51.161)

| Параметр | Значение |
|----------|----------|
| **NPM Host ID** | 5 (удалён, заменён на редирект) |
| **Редирект** | interbudmontazh.ch → https://interbudmontazh.com (301, без сохранения пути) |
| **SSL** | Let's Encrypt (npm-18, до Sep 2026) |
| **Advanced** | `proxy_set_header X-Forwarded-Proto https;` |

### Схема трафика

```
Пользователь → interbudmontazh.ch (любая страница)
  → NPM (94.130.51.161) 
  → 301 редирект → https://interbudmontazh.com/
```

## Исходные данные (fozzy.com)

| Параметр | Значение |
|----------|----------|
| SSH | interbud@78.140.185.130 |
| БД (была) | interbud_cai |
| Пользователь БД | interbud_caiuser |
| Пароль БД | `^D1i}?o,];H}` |
| wp-content | 2.9 GB |

## Примечания

- wp-config.php принудительно включает HTTPS (`$_SERVER['HTTPS'] = 'on'`)
- На fozzy остались файлы, архив на 94.130.51.161 в /tmp/interbud_wp_content.tar.gz
- БД interbud_wp на 192.168.2.39 пока есть (старая, не в Coolify)

## Связанные сущности

- [[server-94-130-51-161]] — NPM
- [[coolify-192-168-2-39]] — Coolify сервер

## История изменений

- **2026-06-03** — Мигрирован с fozzy.com. Развёрнут в Coolify (UUID dvr9kjj972k9ofsxei5hjb6x). NPM proxy host ID 5, SSL LE npm-18.