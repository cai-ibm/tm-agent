---
title: mail.ca-ibm.org (Exchange 2019)
created: 2026-05-15
updated: 2026-05-15
type: entity
tags: [server, exchange, service, security, vulnerability, audit, winrm, smtp]
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
| **Exchange** | Microsoft Exchange Server 2019 CU13 |
| **Build** | 15.2.1748.036 (июнь 2024) |
| **CPU** | AMD EPYC 7401P (24 ядра) |
| **RAM** | 32 ГБ |
| **Диск C:** | 199 ГБ (свободно 63.6 ГБ / 32%) |
| **Диск D:** | 200 ГБ (свободно 174.1 ГБ) |
| **.NET** | 4.8.04161 |
| **DAG** | Отсутствует — одиночный сервер |
| **IIS** | Exchange Back End на портах 81/444 |

## Схема доступа

```
Интернет → 94.130.51.188:443 → 192.168.2.50:443 (Exchange напрямую)
```

## Подключение и управление

### WinRM
- **Протокол:** NTLM, порт 5985
- **Пользователь:** `cai\sr`
- **Venv:** `~/.hermes/venvs/winrm` (pywinrm)
- **Ограничение:** нет прав **Organization Management** — Exchange-командлеты возвращают Access Denied

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
2. **CU13 устарел** — известные CVE (ProxyShell, ProxyLogon)
3. **RDP (3389) открыт наружу**
4. **WinRM HTTP (5985)** — без шифрования

### 🟠 ВЫСОКИЕ / СРЕДНИЕ (требуют исправления на IIS)

5. **Нет HSTS** — Strict-Transport-Security
6. **Нет X-Frame-Options**
7. **CSP минимальный** — только `script-src-attr 'none'`
8. **Disclosure-заголовки:** `X-FEServer: MAIL-SRV1`, `X-Powered-By: ASP.NET`, `X-AspNet-Version: 4.0.30319`, `X-OWA-Version: 15.2.1748.36`
9. **Версия Exchange в URL:** `/owa/auth/15.2.1748/`
10. **Autodiscover с Basic auth**

### ✅ ПОЛОЖИТЕЛЬНЫЕ

- TLS 1.0/1.1 отключены
- Сертификат Let's Encrypt R13 (wildcard *.ca-ibm.org, до 14.06.2026)
- SMTP 25 защищён anti-spam

## Исправление уязвимостей на IIS

Для скрытия disclosure-заголовков и версии в URL — URL Rewrite на IIS. Для HSTS/X-Frame-Options/CSP — HTTP Response Headers.

См. навык `onlyoffice-troubleshooting` (раздел IIS hardening) или отдельную процедуру.

## Связанные сущности

- [[smtp-relay-ca-ibm]] — SMTP-релей 192.168.2.50:2526
- [[stargate-ca-ibm-org]] — Nextcloud
- [[server-94-130-51-161]] — Hetzner, nginx
- [[nginx-pm-192-168-2-31]] — Nginx Proxy Manager (только для HTTP-сервисов)

## История изменений

- **15.05.2026 15:00** — Exchange убран из-за NPM (несовместимость NTLM). Прямой NAT 94.130.51.188:443 → 192.168.2.50. NPM оставлен только для techbau.org.
- **15.05.2026 13:30** — Скрыты disclosure-заголовки и версия в URL через NPM (позже отменено)
- **15.05.2026 12:00** — Exchange размещён за NPM: proxy host id=2, 3 домена, Let's Encrypt
- **15.05.2026 11:00** — Повторный аудит: обнаружен порт 2526 (SMTP relay) открытым наружу
- **09.05.2026** — Первичный аудит безопасности, оценка рисков апгрейда CU
