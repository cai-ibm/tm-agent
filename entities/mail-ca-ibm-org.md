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

## Подключение и управление

### WinRM
- **Протокол:** NTLM, порт 5985
- **Пользователь:** `cai\sr`
- **Venv:** `~/.hermes/venvs/winrm` (pywinrm)
- **Ограничение:** нет прав **Organization Management** — Exchange-командлеты (`Get-ExchangeServer`, `Get-MailboxDatabase`) возвращают Access Denied. Для полноценного администрирования нужны повышенные права.

### SMTP Relay
- **Адрес:** 192.168.2.50:2526
- **Аутентификация:** анонимная
- **Banner:** `Microsoft ESMTP MAIL Service`
- **Отправка:** скрипт `/home/cai/sendmail.py`, отправитель по умолчанию `noreply@ca-ibm.org`
- См. навык Hermes `send-email` для деталей SMTP-релея

## Открытые порты

| Порт | Сервис |
|------|--------|
| 25 | SMTP |
| 80 | HTTP (редирект на HTTPS) |
| 143 | IMAP |
| 443 | HTTPS (OWA, ECP) |
| 465 | SMTPS |
| 587 | SMTP Submission |
| 993 | IMAPS |
| 3389 | RDP — **открыт наружу** |
| 5985 | WinRM HTTP |

## Сервисы Exchange

Все основные сервисы запущены и работают:
- **Transport** — транспортная служба
- **IS** — Information Store (хранилище почтовых ящиков)
- **MailboxAssistants**
- **Replication**
- **FastSearch**

Остановлен: **POP3**

## Уязвимости (аудит 09.05.2026)

### Критические
1. **CU13 устарел** — несколько известных CVE (ProxyShell, ProxyLogon). Текущая версия на несколько CU позади актуальной (CU15/CU16 на момент аудита)
2. **RDP (3389) открыт наружу** — небезопасно, рекомендуется VPN или RD Gateway
3. **WinRM HTTP (5985)** — без шифрования, NTLM-аутентификация

### Высокие / Средние
4. **Нет HSTS** — отсутствует Strict-Transport-Security header
5. **Нет X-Frame-Options** — возможен clickjacking OWA
6. **Нет CSP** — Content Security Policy не задан
7. **Version disclosure** — `x-owa-version: 15.2.1748.36`, `x-powered-by: ASP.NET`
8. **`/api/v2.0/` доступен без аутентификации** — отвечает 401 с NTLM/Negotiate, но сам endpoint достижим без авторизации

## Оценка рисков апгрейда CU13 → CU15/16

| Риск | Описание | Критичность |
|------|----------|-------------|
| **Нет DAG** | Все почтовые ящики на одном сервере. Простой 1-3 часа | Критично |
| **Мало места на C:** | CU требует 10-20 ГБ под распаковку + backup старых бинарников. 63 ГБ на грани | Высокая |
| **Необратимая схема AD** | Схема Active Directory обновляется до установки бинарников. Сбой = без отката | Критично |
| **Нет отката CU** | CU нельзя деинсталлировать. Восстановление только через снапшот VM | Критично |
| **Сброс настроек** | Возможен сброс OWA/ECP web.config, virtual directories, certificate bindings | Средняя |

### Рекомендации перед апгрейдом
1. Сделать снапшот виртуальной машины
2. Очистить/расширить диск C: (минимум 80 ГБ свободно)
3. Запланировать maintenance window (1-3 часа)
4. Получить права Organization Management для `cai\sr`
5. Подготовить PowerShell-скрипт для восстановления настроек после CU

## Связанные сущности

- [[smtp-relay-ca-ibm]] — SMTP-релей 192.168.2.50:2526
- [[stargate-ca-ibm-org]] — Nextcloud, корпоративное облако
- [[server-94-130-51-161]] — Hetzner, nginx reverse proxy

## История изменений

- **2026-05-15** — Страница создана при инициализации LLM Wiki (источник: agent-knowledge-2026-05-15)
- **09.05.2026** — Аудит безопасности (port scan, headers, TLS)
- **09.05.2026** — Оценка рисков апгрейда CU (WinRM + pywinrm)
