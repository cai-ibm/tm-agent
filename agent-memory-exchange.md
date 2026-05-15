---
title: Agent Memory — Exchange MAIL-SRV1
created: 2026-05-15
updated: 2026-05-15
type: entity
tags: [exchange, memory-offload, agent]
sources: [raw/memory/agent-knowledge-2026-05-15.md]
---

# Exchange MAIL-SRV1 — сводка из памяти агента

Вынесено из agent memory для освобождения места.

## Текущее состояние

- **Exchange 2019 CU15** (build 15.2.1748.36, Aug25SU)
- Хост: `MAIL-SRV1`, IP: `192.168.2.50`
- Доступ: WinRM через `CAI\hermes` (порт 5985), pywinrm venv: `~/.hermes/venvs/winrm`
- NPM: **убран**, Exchange на прямом NAT (94.130.51.188:443 → 192.168.2.50:443)

## IIS Rewrite

- **Активно**: Fix X-Frame-Options (SAMEORIGIN) — одно outbound-правило, убирает конфликт с DENY
- **Откачено**: скрытие disclosure-заголовков и версии в URL — outbound URL-правила ломали логин OWA
- URL Rewrite Module **установлен** на сервере

## Известные проблемы

- Disclosure-заголовки раскрыты: `Server: Microsoft-IIS/10.0`, `X-FEServer: MAIL-SRV1`, `X-OWA-Version`, `X-AspNet-Version`
- Версия в URL: `/owa/auth/15.2.1748/...`
- Premium OWA: отсутствуют locale JSON-файлы (`resources/locale/`), вызывает ошибки в консоли
- SMTP relay 2526 открыт наружу (анонимный)
- RDP 3389 открыт наружу
- WinRM HTTP (5985) без шифрования
- CSP минимальный

## Obsidian-заметки

- [[mail-ca-ibm-org]] — основной профиль сервера
- [[exchange-iis-header-hardening]] — попытка скрытия (откат)
- [[exchange-xframe-fix]] — исправление X-Frame-Options
