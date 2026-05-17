---
title: mail.ca-ibm.org (Exchange 2019)
created: 2026-05-15
updated: 2026-05-15
type: entity
tags: [server, exchange, service, security, vulnerability, audit, winrm, smtp, hardening]
sources: [raw/memory/agent-knowledge-2026-05-15.md]
confidence: high
---

# mail.ca-ibm.org — Exchange Server 2019

Почтовый сервер организации. Обслуживает домен `ca-ibm.org`. Развёрнут на Windows Server 2022 Standard, одиночный сервер без DAG.

## Основные параметры

| Параметр | Значение |
|----------|----------|
| **Хостнейм** | MAIL-SRV1 |
| **FQDN** | mail.ca-ibm.org |
| **Локальный IP** | 192.168.2.50 |
| **ОС** | Windows Server 2022 Standard |
| **Exchange** | Microsoft Exchange Server 2019 CU15 |
| **Build** | 15.2.1748.36 (Aug25SU, август 2025) |
| **CPU** | AMD EPYC 7401P (24 ядра) |
| **RAM** | 32 ГБ |
| **Диск C:** | 199 ГБ (свободно 63.6 ГБ / 32%) |
| **Диск D:** | 200 ГБ (свободно 174.1 ГБ) |
| **.NET** | 4.8.04161 |
| **DAG** | Отсутствует — одиночный сервер |
| **IIS** | Exchange Back End на портах 81/444 |

## Схема доступа

```
Интернет → 94.130.51.188:443 → 192.168.2.50:443 (Exchange напрямую, NAT)
```

Exchange **убран из Nginx Proxy Manager** (15.05.2026) из-за несовместимости NTLM.
NPM обслуживает только techbau.org.

## Подключение и управление

### WinRM
- **Протокол:** NTLM, порт 5985
- **Пользователь:** `cai\hermes` (рабочая учётка для автоматизации)
- **Резерв:** `cai\sr` (ограниченные права — Exchange-командлеты возвращают Access Denied)
- **Venv:** `~/.hermes/venvs/winrm` (pywinrm)

### SMTP Relay
- **Внутренний адрес:** 192.168.2.50:2526
- **Аутентификация:** анонимная
- **Banner:** `Microsoft ESMTP MAIL Service`
- **Отправка:** скрипт `/home/cai/sendmail.py`

## Открытые порты

| Порт | Сервис | Статус |
|------|--------|--------|
| 25 | SMTP | Открыт (421 — anti-spam) |
| 80 | HTTP | Открыт (403 / редирект) |
| 143 | IMAP | Открыт |
| 443 | HTTPS (OWA, ECP, EWS, Autodiscover...) | Открыт |
| 465 | SMTPS | Открыт |
| 587 | SMTP Submission | Открыт |
| 993 | IMAPS | Открыт |
| 2526 | SMTP Relay (анонимный) | 🔴 Открыт наружу (15.05) |
| 3389 | RDP | Открыт |
| 5985 | WinRM HTTP | Открыт |

## Уязвимости (аудит 15.05.2026)

### 🔴 КРИТИЧЕСКИЕ

1. **Порт 2526 (SMTP relay) открыт наружу** — анонимная отправка от @ca-ibm.org
2. **CU15 не обновлён** — Aug25SU (15.2.1748.36), доступен Feb26SU (15.2.1748.43) и Exchange SE (15.2.2562.41)
3. **RDP (3389) открыт наружу**
4. **WinRM HTTP (5985)** — без шифрования

### 🟠 ВЫСОКИЕ / СРЕДНИЕ

5. **CSP минимальный** — только `script-src-attr 'none'`
6. **Disclosure-заголовки:** `X-FEServer: MAIL-SRV1`, `X-AspNet-Version: 4.0.30319`, `X-OWA-Version: 15.2.1748.36`, `Server: Microsoft-IIS/10.0`
7. **Версия Exchange в URL:** `/owa/auth/15.2.1748/`
8. **Autodiscover с Basic auth**

### ⚠️ ПОПЫТКА ИСПРАВЛЕНИЯ (ОТКАТ 15.05.2026)

Попытка скрыть заголовки и версию в URL через IIS URL Rewrite откачена — outbound-правила ломают логин OWA (404 после ввода УЗ). Документировано в [[exchange-iis-header-hardening]].

### ✅ ПОЛОЖИТЕЛЬНЫЕ

- HSTS присутствует: `max-age=63072000; preload`
- X-Frame-Options: `DENY` / `SAMEORIGIN`
- TLS 1.0/1.1 отключены
- Сертификат Let's Encrypt R13 (wildcard *.ca-ibm.org, до 14.06.2026)
- SMTP 25 защищён anti-spam

## Связанные сущности

- [[smtp-relay-ca-ibm]] — SMTP-релей 192.168.2.50:2526
- [[stargate-ca-ibm-org]] — Nextcloud
- [[server-94-130-51-161]] — Hetzner, nginx
- [[nginx-pm-192-168-2-31]] — Nginx Proxy Manager (только для HTTP-сервисов)
- [[exchange-iis-header-hardening]] — Скрытие заголовков через URL Rewrite
- [[exchange-extended-protection]] — Extended Protection (EPA)

## История изменений

- **17.05.2026 10:52** — Включён Extended Protection (EPA) на всех виртуальных директориях: Autodiscover, PowerShell (Allow), server root (Allow). OWA/ECP/MAPI (Require), EWS/ActiveSync (Allow) — были ранее.
- **15.05.2026 22:12** — Скрыты disclosure-заголовки через IIS URL Rewrite (Server, X-FEServer, X-AspNet-Version, X-OWA-Version). Обнаружено: CU15, build 15.2.1748.36, HSTS и X-Frame-Options уже присутствуют.
- **15.05.2026 15:00** — Exchange убран из NPM (несовместимость NTLM). Прямой NAT 94.130.51.188:443 → 192.168.2.50. NPM оставлен только для techbau.org.
- **15.05.2026 13:30** — Скрыты disclosure-заголовки и версия в URL через NPM (позже отменено — NTLM несовместимость)
- **15.05.2026 12:00** — Exchange размещён за NPM: proxy host id=2, 3 домена, Let's Encrypt
- **15.05.2026 11:00** — Повторный аудит: обнаружен порт 2526 (SMTP relay) открытым наружу
- **09.05.2026** — Первичный аудит безопасности, оценка рисков апгрейда CU
