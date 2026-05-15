---
title: Exchange IIS — скрытие заголовков через URL Rewrite
created: 2026-05-15
updated: 2026-05-15
type: concept
tags: [exchange, iis, hardening, security, headers]
sources: []
---

# Exchange IIS — скрытие заголовков через URL Rewrite

## Контекст

Exchange 2019 CU15 (build 15.2.1748.36), MAIL-SRV1 (192.168.2.50), IIS 10.0.
Ранее Exchange был за Nginx Proxy Manager, где заголовки скрывались через `proxy_hide_header`.
После перехода на прямой NAT заголовки снова утекали.

## Проблема

Exchange/IIS отдавал 4 информационных заголовка в HTTP-ответах:

| Заголовок | Значение (до) | Утекает |
|-----------|--------------|---------|
| `Server` | `Microsoft-IIS/10.0` | Версия IIS |
| `X-FEServer` | `MAIL-SRV1` | Имя сервера |
| `X-AspNet-Version` | `4.0.30319` | Версия ASP.NET |
| `X-OWA-Version` | `15.2.1748.36` | Билд Exchange (→ версия CU/SU) |

## Решение

IIS URL Rewrite Module (уже был установлен на сервере). Созданы outbound rewrite rules
на уровне `MACHINE/WEBROOT/APPHOST` (глобально для всех сайтов).

### Шаг 1: Разрешить модификацию server variables

Через PowerShell (WinRM, учётка `CAI\hermes`):

```powershell
$headers = @('RESPONSE_Server', 'RESPONSE_X-FEServer',
             'RESPONSE_X-AspNet-Version', 'RESPONSE_X-OWA-Version')
foreach ($var in $headers) {
    Add-WebConfiguration -pspath 'MACHINE/WEBROOT/APPHOST' `
        -filter "system.webServer/rewrite/allowedServerVariables" `
        -value @{name=$var}
}
```

### Шаг 2: Создать outbound rewrite rules

```powershell
$rules = @(
    @{Name='Remove Server Header'; Var='RESPONSE_Server'},
    @{Name='Remove X-FEServer'; Var='RESPONSE_X-FEServer'},
    @{Name='Remove X-AspNet-Version'; Var='RESPONSE_X-AspNet-Version'},
    @{Name='Remove X-OWA-Version'; Var='RESPONSE_X-OWA-Version'}
)
foreach ($rule in $rules) {
    Add-WebConfigurationProperty -pspath 'MACHINE/WEBROOT/APPHOST' `
        -filter "system.webServer/rewrite/outboundRules" -name "." -value @{
            name=$rule.Name;
            patternSyntax='Wildcard';
            match=@{serverVariable=$rule.Var; pattern='*'};
            action=@{type='Rewrite'; value=''}
        }
}
```

### Результат (appcmd list)

```xml
<rule name="Remove Server Header" patternSyntax="Wildcard">
  <match serverVariable="RESPONSE_Server" pattern="*" />
  <action type="Rewrite" />
</rule>
<!-- + 3 аналогичных правила для остальных заголовков -->
```

## Результат

Все 4 заголовка присутствуют в ответе, но с **пустыми значениями**:

```
ДО:
  server: Microsoft-IIS/10.0
  x-feserver: MAIL-SRV1
  x-aspnet-version: 4.0.30319
  x-owa-version: 15.2.1748.36

ПОСЛЕ:
  server:
  x-feserver:
  x-aspnet-version:
  x-owa-version:
```

## Server header — полное удаление (НЕ ПРИМЕНЕНО)

Заголовок `Server` генерируется HTTP.sys (kernel-драйвер), не IIS.
Для полного удаления нужен ключ реестра:

```
HKLM\SYSTEM\CurrentControlSet\Services\HTTP\Parameters
  DisableServerHeader = 1 (DWORD)
```

**Требуется перезапуск HTTP.sys** (`net stop http /y && net start http`) —
это на секунды обрывает ВСЕ HTTP-сервисы Exchange.
Риск не подняться обратно → Exchange недоступен по вебу.

**Решение**: оставлено как есть. Пустых значений достаточно — 
ни версия IIS, ни имя сервера, ни билд Exchange не утекают.

## Скрытие версии Exchange из URL (15.05.2026)

OWA-страница логина содержит пути вида `/owa/auth/15.2.1748/themes/...` — 
трёхчастный билд раскрывает версию CU.

### Outbound rule — вырезать версию из HTML

Работает только для text/html-ответов (precondition `IsHTML`),
Regex: `(owa/auth)/15\\.2\\.\\d+(/themes/.*)` → `{R:1}{R:2}`.
Важно: матчится только `/themes/` — не задевает JS-переменные ASP.NET (a_sLgn, a_sUrl)
которые содержат полные пути с версией и ломали редирект после логина.

```powershell
# Precondition
Add-WebConfigurationProperty -pspath 'MACHINE/WEBROOT/APPHOST' `
    -filter "system.webServer/rewrite/outboundRules/preConditions" -name "." -value @{
        name='IsHTML'
    }
Add-WebConfigurationProperty -pspath 'MACHINE/WEBROOT/APPHOST' `
    -filter "system.webServer/rewrite/outboundRules/preConditions/preCondition[@name='IsHTML']" `
    -name "." -value @{input='{RESPONSE_CONTENT_TYPE}'; pattern='^text/html'}

# Outbound rule
Add-WebConfigurationProperty -pspath 'MACHINE/WEBROOT/APPHOST' `
    -filter "system.webServer/rewrite/outboundRules" -name "." -value @{
        name='Strip Exchange Version from URLs';
        preCondition='IsHTML';
        patternSyntax='ECMAScript';
        match=@{filterByTags='None';        pattern='(owa/auth)/15\\.2\\.\\d+(/themes/.*)'};
        action=@{type='Rewrite'; value='{R:1}{R:2}'}
    }
```

### Inbound rule — пробросить запросы без версии к реальным файлам

Браузер получает очищенный HTML и запрашивает `/owa/auth/themes/...`.
Inbound-правило перенаправляет эти запросы к реальным версионированным путям.

```powershell
Add-WebConfigurationProperty -pspath 'MACHINE/WEBROOT/APPHOST' `
    -filter "system.webServer/rewrite/rules" -name "." -value @{
        name='Map Exchange OWA Themes (hide version)';
        patternSyntax='ECMAScript';
        stopProcessing='false';
        match=@{url='^owa/auth/themes/(.*)'};
        action=@{type='Rewrite'; url='owa/auth/15.2.1748/themes/{R:1}'; appendQueryString='false'}
    }
```

### Результат

```
ДО:  /owa/auth/15.2.1748/themes/resources/favicon.ico
ПОСЛЕ: /owa/auth/themes/resources/favicon.ico
```

Inbound-проверка: `GET /owa/auth/themes/resources/favicon.ico` → HTTP 200, image/x-icon ✅

## Связанные страницы

- [[mail-ca-ibm-org]] — Exchange Server 2019, MAIL-SRV1
- [[server-94-130-51-161]] — Hetzner-сервер с nginx/n8n/OnlyOffice
