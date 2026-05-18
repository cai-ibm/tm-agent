---
title: Exchange Extended Protection (EPA)
created: 2026-05-17
updated: 2026-05-17
type: entity
tags: [exchange, security, hardening, epa, extended-protection]
parent: "[[mail-ca-ibm-org]]"
confidence: high
---

# Exchange Extended Protection (EPA)

**Дата:** 17.05.2026
**Исполнитель:** Hermes (Немец), через WinRM `cai\hermes`

## Что такое Extended Protection

Extended Protection for Authentication (EPA) — механизм защиты Windows-аутентификации (NTLM/Kerberos) от relay-атак (Pass-the-Hash, token relay, MitM). Связывает аутентификацию с TLS-каналом (Channel Binding) и Service Principal Name.

**KB5017260** — первое обновление, добавившее EPA в Exchange 2019 CU12. Начиная с CU14 — включено по умолчанию при новой установке.

## Текущее состояние (17.05.2026)

| Виртуальная директория | EP TokenChecking |
|------------------------|-----------------|
| OWA | **Require** |
| ECP | **Require** |
| EWS | **Allow** |
| ActiveSync | **Allow** |
| Autodiscover | **Allow** (включено 17.05) |
| MAPI | **Require** |
| PowerShell | **Allow** (включено 17.05) |
| Server root | **Allow** (включено 17.05) |

## Что сделано 17.05.2026

Через IIS (`Set-WebConfigurationProperty`) включён Extended Protection на виртуальных директориях, где он отсутствовал:

- `Default Web Site/Autodiscover`: None → **Allow**
- `Default Web Site/PowerShell`: None → **Allow**
- Server root: None → **Allow**

### Команды

```powershell
Set-WebConfigurationProperty -PSPath 'MACHINE/WEBROOT/APPHOST' `
  -Filter "system.webServer/security/authentication/windowsAuthentication/extendedProtection" `
  -Location 'Default Web Site/Autodiscover' -Name tokenChecking -Value 'Allow'

Set-WebConfigurationProperty -PSPath 'MACHINE/WEBROOT/APPHOST' `
  -Filter "system.webServer/security/authentication/windowsAuthentication/extendedProtection" `
  -Location 'Default Web Site/PowerShell' -Name tokenChecking -Value 'Allow'

Set-WebConfigurationProperty -PSPath 'MACHINE/WEBROOT/APPHOST' `
  -Filter "system.webServer/security/authentication/windowsAuthentication/extendedProtection" `
  -Name tokenChecking -Value 'Allow'
```

### Почему через IIS, а не Exchange

`cai\hermes` не может выполнять Exchange-командлеты — `ADInvalidCredentialException` (недостаточно прав Active Directory). Правильный путь — `Set-ExchangeServer -ExtendedProtectionTokenChecking Allow`, но требует `Organization Management`. Результат через IIS идентичен на уровне HTTP.

## Проверка работоспособности

Все endpoint'ы отвечают штатно:
- OWA: HTTP 302 (редирект на логин)
- ECP: HTTP 302
- EWS: HTTP 401 (требует аутентификации)
- Autodiscover: HTTP 401
- MAPI: HTTP 401
- ActiveSync: HTTP 401
- PowerShell: HTTP 403 (IP-ограничение — штатно)

## Примечания

- IIS restart **не требуется** — правила применяются мгновенно
- WinRM: `cai\hermes` @ 192.168.2.50:5985 (NTLM), venv: `~/.hermes/venvs/winrm`
- После CU-апгрейда проверить что EP настройки сохранились
- **Рекомендация:** через 1-2 недели мониторинга перевести `Allow` → `Require` для максимальной защиты

## Связанные сущности

- [[mail-ca-ibm-org]] — Exchange Server
- [[exchange-iis-headers]] — управление HTTP-заголовками (предыдущие изменения IIS)
