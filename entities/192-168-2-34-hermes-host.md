---
title: hermes-host (192.168.2.34)
created: 2026-06-30
updated: 2026-07-01
type: entity
tags: [server, hermes, docker, supabase, nfs, monitoring]
confidence: high
---

# hermes-host — 192.168.2.34

Основной хост Hermes Agent. Хостнейм: `hermes-eu`. Внешний IP: 94.130.51.188.

- **OS:** Ubuntu 24.04.4 LTS, kernel 6.8.0-124-generic
- **Uptime:** 20 days
- **CPU:** Load avg 1.01 / 0.90 / 0.75
- **RAM:** 15G total, 5.6G used, 10G available (Swap: 0)
- **Disk:** 96G (LVM), 58G used (64%)

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
| `camofox-browser` | Camofox антидетект браузер (порт 9377) |
| `ocerp-db` | PostgreSQL для OCERP |
| `pg-direct-tunnel` | Socat-туннель (5433 → supabase-db:5432) |
| `cadvisor` | Контейнерные метрики (порт 8080) |
| `grafana` | Дашборды мониторинга (порт 3000) |
| `prometheus` | Сбор метрик (порт 9090) |
| `node-exporter` | Метрики хоста (порт 9100) |
| `supabase-*` (13 шт) | Supabase stack (DB, Kong, Auth, Storage, Studio, Realtime, Pooler, REST, Meta, Analytics, Vector, Imgproxy, Edge Functions) |

## NFS

- **Шара:** `192.168.3.253:/IBT-files` → `/mnt/ibt-nfs`
- **Ёмкость:** 32T, занято 25T (76%)

## Мониторинг (Prometheus + Grafana)

Стек развёрнут через Docker Compose в `/home/cai/monitoring/`.

| Сервис | Порт | Статус |
|--------|------|--------|
| **Grafana** | 3000 | admin / Superp@ss2020grafana |
| **Prometheus** | 9090 | Собирает метрики |
| **Node Exporter** | 9100 | Локальный + удалённые хосты |
| **cAdvisor** | 8080 | Контейнерные метрики |

**Дашборды:**
- `Хосты — Мониторинг ресурсов` (UID: `aj7rdx`) — CPU/RAM/Disk/Network по всем хостам (фильтр по имени хоста)
- `Node Exporter Full` (ID 1860) — детальные метрики хостов
- `Docker and Host Monitoring w/ Prometheus` (ID 179) — контейнерные метрики

**Мониторимые хосты:**
- `hermes-host` — локальный (192.168.2.34)
- `hz-host1` — 192.168.2.37
- `hz-host2` — 192.168.2.31
- `coolify` — 192.168.2.39 (11 контейнеров)

**Всего:** 4 хоста, 52 контейнера под мониторингом.

**Дашборд MS SQL Server (eu-e1c):** UID `abstn5` — I/O stall, wait stats, buffer pool, log growths, PLE.

## Связанные страницы

[[claude-code-container]], [[192-168-2-34-supabase]], [[192-168-2-37-hz-host1]]
