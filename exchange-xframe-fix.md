---
title: Exchange IIS — X-Frame-Options fix
created: 2026-05-15
updated: 2026-05-15
type: concept
tags: [exchange, iis, hardening, security, headers]
sources: []
---

# X-Frame-Options конфликт — PDF preview fix

## Проблема

Exchange 2019 CU15 генерирует два заголовка `X-Frame-Options` с разными значениями:
- `SAMEORIGIN` — от OWA auth модуля (разрешает iframe с того же origin)
- `DENY` — от OWA core модуля (запрещает все iframe)

Браузер при конфликте выбирает самое строгое (`DENY`), что ломает:
- PDF-превью в OWA (использует iframe)
- Любые встроенные фреймы

## Решение

Одно outbound-правило IIS URL Rewrite: перезаписать `RESPONSE_X-Frame-Options` в `SAMEORIGIN`.

```powershell
# 1. Разрешить переменную
Add-WebConfiguration -pspath 'MACHINE/WEBROOT/APPHOST' `
    -filter "system.webServer/rewrite/allowedServerVariables" `
    -value @{name='RESPONSE_X-Frame-Options'}

# 2. Создать правило
Add-WebConfigurationProperty -pspath 'MACHINE/WEBROOT/APPHOST' `
    -filter "system.webServer/rewrite/outboundRules" -name "." -value @{
        name='Fix X-Frame-Options (SAMEORIGIN)';
        patternSyntax='Wildcard';
        match=@{serverVariable='RESPONSE_X-Frame-Options'; pattern='*'};
        action=@{type='Rewrite'; value='SAMEORIGIN'}
    }
```

## Результат

- Все `X-Frame-Options` → `SAMEORIGIN`
- Дубликаты с одинаковым значением безопасны
- PDF-превью работает
- Clickjacking-защита сохранена (чужие origin блокируются)

## Связанные страницы

- [[mail-ca-ibm-org]] — Exchange Server 2019
- [[exchange-iis-header-hardening]] — попытка полного скрытия (откат)
