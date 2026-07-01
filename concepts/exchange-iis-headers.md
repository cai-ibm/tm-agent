---
title: Exchange IIS — управление HTTP-заголовками
created: 2026-05-18
updated: 2026-05-18
type: concept
tags: [exchange, iis, security, hardening, headers, rewrite]
sources: []
confidence: high
---

# Exchange IIS — управление HTTP-заголовками

Хронология всех изменений HTTP-заголовков Exchange 2019 (MAIL-SRV1, 192.168.2.50) через IIS URL Rewrite Module. Контекст: после ухода из-под Nginx Proxy Manager (15.05.2026) заголовки снова утекали, потребовалось управление на уровне IIS.

## Исходные заголовки (до всех изменений)

| Заголовок | Значение | Что утекает |
|-----------|----------|-------------|
| `Server` | `Microsoft-IIS/10.0` | Версия IIS |
| `X-FEServer` | `MAIL-SRV1` | Имя сервера |
| `X-OWA-Version` | `15.2.1748.36` | Билд Exchange (→ версия CU/SU) |
| `X-AspNet-Version` | `4.0.30319` | Версия ASP.NET |
| `X-Powered-By` | `ASP.NET` | Технологический стек |
| `X-Frame-Options` | `DENY` + `SAMEORIGIN` (конфликт!) | — |

---

## Фаза 1 — X-Frame-Options fix (15.05.2026) ✅ АКТИВНО

**Проблема:** Exchange генерирует два заголовка `X-Frame-Options` с разными значениями: `SAMEORIGIN` (от OWA auth) и `DENY` (от OWA core). Браузер выбирает самое строгое (`DENY`) → ломает PDF-превью в OWA.

**Решение:** одно outbound-правило — перезаписать `RESPONSE_X-Frame-Options` в `SAMEORIGIN`.

```powershell
# Разрешить переменную
Add-WebConfiguration -pspath 'MACHINE/WEBROOT/APPHOST' `
    -filter "system.webServer/rewrite/allowedServerVariables" `
    -value @{name='RESPONSE_X-Frame-Options'}

# Создать правило
Add-WebConfigurationProperty -pspath 'MACHINE/WEBROOT/APPHOST' `
    -filter "system.webServer/rewrite/outboundRules" -name "." -value @{
        name='Fix X-Frame-Options (SAMEORIGIN)';
        patternSyntax='Wildcard';
        match=@{serverVariable='RESPONSE_X-Frame-Options'; pattern='*'};
        action=@{type='Rewrite'; value='SAMEORIGIN'}
    }
```

**Результат:** X-Frame-Options → `SAMEORIGIN`, PDF-превью работает, clickjacking-защита сохранена.

---

## Фаза 2 — Первая попытка скрытия заголовков (15.05.2026) ⚠️ ОТКАТ

**Цель:** скрыть 4 disclosure-заголовка + версию Exchange из URL.

### Заголовки — через server variable rules

Созданы outbound-правила, обнуляющие значения (не удаляющие заголовки):

| Правило | Переменная | Результат |
|---------|-----------|-----------|
| Remove Server Header | `RESPONSE_Server` | `server:` (пусто) |
| Remove X-FEServer | `RESPONSE_X-FEServer` | `x-feserver:` (пусто) |
| Remove X-AspNet-Version | `RESPONSE_X-AspNet-Version` | `x-aspnet-version:` (пусто) |
| Remove X-OWA-Version | `RESPONSE_X-OWA-Version` | `x-owa-version:` (пусто) |

```powershell
$headers = @('RESPONSE_Server', 'RESPONSE_X-FEServer',
             'RESPONSE_X-AspNet-Version', 'RESPONSE_X-OWA-Version')
foreach ($var in $headers) {
    Add-WebConfiguration -pspath 'MACHINE/WEBROOT/APPHOST' `
        -filter "system.webServer/rewrite/allowedServerVariables" `
        -value @{name=$var}
}
# Затем 4 Add-WebConfigurationProperty с action=@{type='Rewrite'; value=''}
```

### Версия в URL — через outbound rewrite

HTML OWA содержит пути вида `/owa/auth/15.2.1748/themes/...`. Созданы два правила:

1. **Outbound** — вырезать версию из HTML (только `text/html`, матчится `/themes/` чтобы не задеть JS-переменные ASP.NET).
2. **Inbound** — пробросить запросы без версии к реальным файлам.

```powershell
# Outbound: (owa/auth)/15\.2\.\d+(/themes/.*) → {R:1}{R:2}
# Inbound:  ^owa/auth/themes/(.*) → owa/auth/15.2.1748/themes/{R:1}
```

### Причина отката

Outbound URL-правила ломают логин OWA: после ввода учётных данных — HTTP 404. Точная причина не выявлена (curl-тесты работают, в браузере — 404). Все правила (и header, и URL) удалены.

**Урок:** outbound URL rewrite на Exchange OWA нестабилен. Header-правила безопасны.

---

## Фаза 3 — Успешная подмена заголовков (16.05.2026) ✅ АКТИВНО

**Исполнитель:** Hermes через WinRM `cai\sr`. Правила на уровне `MACHINE/WEBROOT/APPHOST` (глобально для всех сайтов IIS). IIS restart не требуется.

### Результат

| Заголовок | Было | Стало |
|-----------|------|-------|
| `X-FEServer` | `MAIL-SRV1` | `MAIL.CA-IBM.ORG` |
| `X-OWA-Version` | `15.2.1748.36` | `15.02.2562.037` |
| `X-Powered-By` | `ASP.NET` | (пусто) |
| `X-AspNet-Version` | `4.0.30319` | (пусто) |

### Созданные правила

```powershell
# 1. Allowed Server Variables
$vars = @('RESPONSE_X-FEServer', 'RESPONSE_X-OWA-Version',
          'RESPONSE_X-Powered-By', 'RESPONSE_X-AspNet-Version')
foreach ($var in $vars) {
    Add-WebConfiguration -pspath 'MACHINE/WEBROOT/APPHOST' `
        -filter "system.webServer/rewrite/allowedServerVariables" `
        -value @{name=$var}
}

# 2. Rules
Add-WebConfigurationProperty ... -value @{
    name='Replace X-FEServer'; match=@{serverVariable='RESPONSE_X-FEServer'; pattern='*'};
    action=@{type='Rewrite'; value='MAIL.CA-IBM.ORG'}
}
Add-WebConfigurationProperty ... -value @{
    name='Replace X-OWA-Version'; match=@{serverVariable='RESPONSE_X-OWA-Version'; pattern='*'};
    action=@{type='Rewrite'; value='15.02.2562.037'}
}
Add-WebConfigurationProperty ... -value @{
    name='Remove X-Powered-By'; match=@{serverVariable='RESPONSE_X-Powered-By'; pattern='*'};
    action=@{type='Rewrite'; value=''}
}
Add-WebConfigurationProperty ... -value @{
    name='Remove X-AspNet-Version'; match=@{serverVariable='RESPONSE_X-AspNet-Version'; pattern='*'};
    action=@{type='Rewrite'; value=''}
}
```

### Откат (если потребуется)

```powershell
Clear-WebConfiguration -pspath 'MACHINE/WEBROOT/APPHOST' `
    -filter "system.webServer/rewrite/outboundRules/rule[@name='Replace X-FEServer']"
# ... аналогично для остальных правил
```

### Проверка

```bash
curl -sk -D - -o /dev/null https://192.168.2.50/owa/ | grep -iE 'x-feserver|x-owa-version|x-powered-by|x-aspnet-version'
```

---

## Server header — не трогали

Заголовок `Server: Microsoft-IIS/10.0` генерируется HTTP.sys (kernel-драйвер), не IIS. Для удаления нужен ключ реестра:

```
HKLM\SYSTEM\CurrentControlSet\Services\HTTP\Parameters
  DisableServerHeader = 1 (DWORD)
```

Требуется перезапуск HTTP.sys (`net stop http /y && net start http`) — обрывает ВСЕ HTTP-сервисы Exchange на секунды. Риск не подняться обратно. Оставлено как есть.

---

## Примечания

- После CU-апгрейда Exchange проверить, что правила сохранились (некоторые CU сбрасывают IIS-конфигурацию)
- Правила на уровне `MACHINE/WEBROOT/APPHOST` действуют на все сайты IIS
- WinRM: `cai\hermes` / `cai\sr` @ 192.168.2.50:5985 (NTLM), venv: `~/.hermes/venvs/winrm`

## Связанные сущности

- [[mail-ca-ibm-org]] — Exchange Server 2019, MAIL-SRV1
- [[exchange-extended-protection]] — Extended Protection (EPA)
-  — Nginx Proxy Manager (мог бы скрывать заголовки через L7, но Exchange убран)
- [[server-94-130-51-161]] — Hetzner, внешний IP для NAT
