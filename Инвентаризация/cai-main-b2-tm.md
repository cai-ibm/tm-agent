---
title: cai-main-b2-tm — сервер виртуализации
created: 2026-06-16
updated: 2026-06-16
type: entity
tags: [server, vm, container, lxc, docker, inventory]
sources: []
confidence: high
---

# cai-main-b2-tm (192.168.30.12)

**SSH:** sobolevrv@192.168.30.12
**Роль:** Сервер виртуализации предприятия — QEMU/KVM, LXC, Docker

---

## ХОСТ

| Параметр | Значение |
|---|---|
| Hostname | cai-main-b2-tm |
| CPU | 56 ядер |
| RAM | 251 GiB (183 GiB занято, 73%) |
| Диск (/) | 147G / 93G занято (67%) |
| Uptime | 36 дней (на 2026-06-16) |
| Load | ~16.8 |

---

## QEMU/KVM — 7 RUNNING + 7 SHUT OFF

### Работающие

| VM | vCPU | RAM | Диски | Uptime |
|---|---|---|---|---|
| **win2k16-DC1** | 4 | 4 GB | 1x qcow2 | 36d |
| **win2k22-wsus** | 4 | 8 GB | 2x qcow2 (wsus, wsus-1) | 36d |
| **win2k22-printsrv** | 8 | 8 GB | 1x qcow2 | 36d |
| **win2k22-e1crdp** | 10 | 64 GB | 1x raw | 36d |
| **win2k22-sqlsrv** | 8 | 64 GB | 2x raw (sqlsrv, sqlsrv-1) | 36d |
| **win2k22-obmenka** | 8 | 8 GB | 3x qcow2 (obmenka, -1, -4) | 36d |
| **win2k22-e1csrv** | 12 | 58 GB | 2x raw (e1csrv, e1csrv-1) | 6d |

**Суммарно:** 54 vCPU, ~214 GB RAM

### Выключенные

| VM | vCPU | RAM |
|---|---|---|
| win10-ok-wapp | 4 | 8 GB |
| win10-test-rdp | 2 | 4 GB |
| win10test | 2 | 4 GB |
| win11-esit | 10 | 8 GB |
| win2k22-DC-1 | 4 | 8 GB |
| win2k22-e1c-test | 8 | 8 GB |
| win2k22-jas-rdp | 4 | 4 GB |
| win7anydesk | 1 | 4 GB |

**Образы дисков:** 4.5 TB в `/var/lib/libvirt/images/`

---

## LXC — 12 RUNNING + 9 STOPPED

### Работающие

| Контейнер | IP | Назначение |
|---|---|---|
| **cai-b2-rsync** | 192.168.0.22 | rsync |
| **cron-scripts** | 192.168.40.32 | cron-скрипты |
| **e1c-web** | 192.168.0.20 | 1С веб-доступ |
| **kv-scripts** | 192.168.40.30 | скрипты |
| **pg15-8-1-1c** | 192.168.0.19 | PostgreSQL 15 для 1С |
| **printtrap** | 192.168.40.24 | печать |
| **redmine** | 192.168.0.21 | Redmine |
| **scan-server2** | 192.168.30.23 | сканирование |
| **tenders** | 192.168.40.20 | тендеры |
| **tenders-old** | 192.168.0.8 | тендеры (старый) |
| **translator** | 192.168.30.103 | переводчик |
| **wasender** | 192.168.31.53 | WhatsApp sender |

### Остановленные

airprint, of-minio, of-redis, of-worker, scan-server, scan-server-t, softether, test-route

---

## DOCKER — 2 контейнера

| Имя | Образ | Статус | Порты |
|---|---|---|---|
| **odoo-app** | odoo:19.0 | Up 5 weeks | 127.0.0.1:8069, 8072 |
| **odoo-db** | postgres:15 | Up 5 weeks | 5432/tcp |

---

## wasender (LXC) — детально

- **ОС:** Ubuntu 24.04.3 LTS
- **IP:** 192.168.31.53/24 (был 192.168.40.25, изменён 2026-06-16)
- **Пользователи:** root, ubuntu
- **WhatsApp бот:** `/opt/wa-bot/index.js` (Node.js, от ubuntu)
- **rclone:** монтирует `cai.info.tm:services/ok-wa` → `/mnt/ok-wa`
- **Открытые порты:** 22 (SSH), 21 (FTP), 53 (DNS)
- **Доступ:** через `lxc-attach` с хоста (SSH-ключа нет)
