---
title: BookStack — корпвикипедия
created: 2026-07-03
updated: 2026-07-03
type: entity
tags: [service, wiki, bookstack, ldap, docker]
---

# BookStack — корпвикипедия

**BookStack** — self-hosted wiki/база знаний, развёрнута на tm-host1.

## Параметры

| Параметр | Значение |
|---|---|
| Сервер | tm-host1 (192.168.40.28) |
| Домен | http://wiki.ca-ibm.org |
| Прямой доступ | http://192.168.40.28 (через Apache reverse proxy) |
| Тип развёртывания | Docker Compose |
| Образ | `lscr.io/linuxserver/bookstack:latest` (v26.05.2) |
| БД | MariaDB 11 (контейнер bookstack-db) |
| Конфиг | `/opt/bookstack/docker-compose.yml` |

## Аутентификация

**Режим:** LDAP (Active Directory)

| Параметр LDAP | Значение |
|---|---|
| Сервер | `ldap://192.168.30.9` (gc1.ca-ibm.org) |
| Base DN | `DC=ca-ibm,DC=org` |
| Bind DN | `CN=ldap,OU=Admin,OU=EU,DC=ca-ibm,DC=org` |
| Фильтр | `(&(objectClass=user)(sAMAccountName={user}))` |
| ID атрибут | `sAMAccountName` |
| Email атрибут | `mail` |
| Display name | `displayName` |

**Вход:** доменный логин (sAMAccountName) + доменный пароль.

## Администраторы

| Пользователь | Тип | Роль |
|---|---|---|
| `sr` | LDAP (sAMAccountName=sr) | Admin |
| `root@ca-ibm.org` | Локальный (отключён при LDAP) | Admin |

## Развёртывание

Wiki.js был удалён, на его место установлен BookStack. Apache2 переключен в reverse proxy для wiki.ca-ibm.org.

**Контейнеры:**
- `bookstack-app` — порт 127.0.0.1:8080
- `bookstack-db` — MariaDB

## Связанные страницы

- [[server-e1c-web-tm-host1]] — сервер tm-host1
- [[mail-ca-ibm-org]] — Exchange/AD (LDAP-сервер)
