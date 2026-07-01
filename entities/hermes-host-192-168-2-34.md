---
title: hermes-host (192.168.2.34)
created: 2026-06-30
updated: 2026-06-30
type: entity
tags: [server, hermes, docker, supabase, nfs]
confidence: high
---

# hermes-host — 192.168.2.34

Основной хост Hermes Agent. Хостнейм: `hermes-eu`. Внешний IP: 94.130.51.188.

## Доступ

- **SSH:** `cai@192.168.2.34`, ключ `id_ed25519`
- **sudo:** не настроен (пароль не задан)
- **Docker:** пользователь `cai` в группе `docker`

## Docker-контейнеры

| Контейнер | Назначение |
|-----------|-----------|
| `claude-code` | Claude Code CLI (Anthropic) |
| `hermes-d2ca1602` | Hermes Agent |
| `hermes-921e6cbe` | Hermes Agent (второй инстанс) |
| `camofox-browser` | Camofox антидетект браузер |
| `ocerp-app` | OpenConstructionERP |
| `ocerp-db` | PostgreSQL для OCERP |
| `pg-direct-tunnel` | Socat-туннель (5433 → supabase-db:5432) |
| `supabase-*` (13 шт) | Supabase stack (DB, Kong, Auth, Storage, Studio, Realtime и др.) |

## NFS

- **Шара:** `192.168.3.253:/IBT-files` → `/mnt/ibt-nfs`
- **Ёмкость:** 32T, занято 25T (76%)

## Связанные страницы

[[claude-code-container]], [[supabase-192-168-2-34]], [[server-94-130-51-161]]
