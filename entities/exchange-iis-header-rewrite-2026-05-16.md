---
title: Exchange IIS Header Rewrite — подмена заголовков
created: 2026-05-16
updated: 2026-05-16
type: procedure
tags: [exchange, iis, security, hardening, rewrite]
parent: "[[mail-ca-ibm-org]]"
confidence: high
---

# Exchange IIS Header Rewrite — подмена заголовков

**Дата:** 16.05.2026
**Исполнитель:** Hermes (Немец), через WinRM `cai\sr`

## Что сделано

Через IIS URL Rewrite созданы outbound-правила, подменяющие disclosure-заголовки Exchange на уровне сервера (`MACHINE/WEBROOT/APPHOST`). Правила применяются глобально ко всем сайтам IIS.

## Результат

| Заголовок | Было | Стало |
|-----------|------|-------|
| `X-FEServer` | `MAIL-SRV1` | `MAIL.CA-IBM.ORG` |
| `X-OWA-Version` | `15.2.1748.36` | `15.02.2562.037` |
| `X-Powered-By` | `ASP.NET` | (пусто) |
| `X-AspNet-Version` | `4.0.30319` | (пусто) |

## Созданные правила

### Allowed Server Variables

```powershell
Add-WebConfiguration -pspath 'MACHINE/WEBROOT/APPHOST' `
    -filter "system.webServer/rewrite/allowedServerVariables" `
    -value @{name='RESPONSE_X-FEServer'}

Add-WebConfiguration -pspath 'MACHINE/WEBROOT/APPHOST' `
    -filter "system.webServer/rewrite/allowedServerVariables" `
    -value @{name='RESPONSE_X-OWA-Version'}

Add-WebConfiguration -pspath 'MACHINE/WEBROOT/APPHOST' `
    -filter "system.webServer/rewrite/allowedServerVariables" `
    -value @{name='RESPONSE_X-Powered-By'}

Add-WebConfiguration -pspath 'MACHINE/WEBROOT/APPHOST' `
    -filter "system.webServer/rewrite/allowedServerVariables" `
    -value @{name='RESPONSE_X-AspNet-Version'}
```

### Outbound Rewrite Rules

```powershell
# X-FEServer → MAIL.CA-IBM.ORG
Add-WebConfigurationProperty -pspath 'MACHINE/WEBROOT/APPHOST' `
    -filter "system.webServer/rewrite/outboundRules" -name "." -value @{
        name='Replace X-FEServer';
        patternSyntax='Wildcard';
        match=@{serverVariable='RESPONSE_X-FEServer'; pattern='*'};
        action=@{type='Rewrite'; value='MAIL.CA-IBM.ORG'}
    }

# X-OWA-Version → 15.02.2562.037
Add-WebConfigurationProperty -pspath 'MACHINE/WEBROOT/APPHOST' `
    -filter "system.webServer/rewrite/outboundRules" -name "." -value @{
        name='Replace X-OWA-Version';
        patternSyntax='Wildcard';
        match=@{serverVariable='RESPONSE_X-OWA-Version'; pattern='*'};
        action=@{type='Rewrite'; value='15.02.2562.037'}
    }

# X-Powered-By → пусто
Add-WebConfigurationProperty -pspath 'MACHINE/WEBROOT/APPHOST' `
    -filter "system.webServer/rewrite/outboundRules" -name "." -value @{
        name='Remove X-Powered-By';
        patternSyntax='Wildcard';
        match=@{serverVariable='RESPONSE_X-Powered-By'; pattern='*'};
        action=@{type='Rewrite'; value=''}
    }

# X-AspNet-Version → пусто
Add-WebConfigurationProperty -pspath 'MACHINE/WEBROOT/APPHOST' `
    -filter "system.webServer/rewrite/outboundRules" -name "." -value @{
        name='Remove X-AspNet-Version';
        patternSyntax='Wildcard';
        match=@{serverVariable='RESPONSE_X-AspNet-Version'; pattern='*'};
        action=@{type='Rewrite'; value=''}
    }
```

## Инструкция отката

### Вариант 1: Удалить правила (вернуть оригинальные заголовки)

```powershell
# Удалить outbound rules
Clear-WebConfiguration -pspath 'MACHINE/WEBROOT/APPHOST' `
    -filter "system.webServer/rewrite/outboundRules/rule[@name='Replace X-FEServer']"

Clear-WebConfiguration -pspath 'MACHINE/WEBROOT/APPHOST' `
    -filter "system.webServer/rewrite/outboundRules/rule[@name='Replace X-OWA-Version']"

Clear-WebConfiguration -pspath 'MACHINE/WEBROOT/APPHOST' `
    -filter "system.webServer/rewrite/outboundRules/rule[@name='Remove X-Powered-By']"

Clear-WebConfiguration -pspath 'MACHINE/WEBROOT/APPHOST' `
    -filter "system.webServer/rewrite/outboundRules/rule[@name='Remove X-AspNet-Version']"

# Удалить allowed variables (опционально, не мешают если останутся)
Remove-WebConfigurationProperty -pspath 'MACHINE/WEBROOT/APPHOST' `
    -filter "system.webServer/rewrite/allowedServerVariables" `
    -name "." -AtElement @{name='RESPONSE_X-FEServer'}

Remove-WebConfigurationProperty -pspath 'MACHINE/WEBROOT/APPHOST' `
    -filter "system.webServer/rewrite/allowedServerVariables" `
    -name "." -AtElement @{name='RESPONSE_X-OWA-Version'}

Remove-WebConfigurationProperty -pspath 'MACHINE/WEBROOT/APPHOST' `
    -filter "system.webServer/rewrite/allowedServerVariables" `
    -name "." -AtElement @{name='RESPONSE_X-Powered-By'}

Remove-WebConfigurationProperty -pspath 'MACHINE/WEBROOT/APPHOST' `
    -filter "system.webServer/rewrite/allowedServerVariables" `
    -name "." -AtElement @{name='RESPONSE_X-AspNet-Version'}
```

### Вариант 2: Изменить подменяемые значения

```powershell
# Например, сменить X-FEServer на другое значение
Set-WebConfigurationProperty -pspath 'MACHINE/WEBROOT/APPHOST' `
    -filter "system.webServer/rewrite/outboundRules/rule[@name='Replace X-FEServer']/action" `
    -name "value" -value "НОВОЕ.ЗНАЧЕНИЕ"
```

## Проверка

```bash
curl -sk -D - -o /dev/null https://192.168.2.50/owa/ | grep -iE 'x-feserver|x-owa-version|x-powered-by|x-aspnet-version'
```

Ожидаемый вывод:
```
x-feserver: MAIL.CA-IBM.ORG
x-owa-version: 15.02.2562.037
x-powered-by:
x-aspnet-version:
```

## Примечания

- Правила на уровне `MACHINE/WEBROOT/APPHOST` — действуют на все сайты IIS (Default Web Site + Exchange Back End)
- IIS restart **не требуется** — правила применяются мгновенно
- После CU-апгрейда Exchange проверить что правила сохранились (некоторые CU сбрасывают IIS-конфигурацию)
- `Server: Microsoft-IIS/10.0` — не трогали, требует правки реестра + перезапуск HTTP.sys
- WinRM: `cai\sr` @ 192.168.2.50:5985 (NTLM), venv: `~/.hermes/venvs/winrm`

## Связанные сущности

- [[mail-ca-ibm-org]] — Exchange Server
- [[nginx-pm-192-168-2-31]] — Nginx Proxy Manager (Exchange убран, но мог бы скрывать заголовки через L7)
