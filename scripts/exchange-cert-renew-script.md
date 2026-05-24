---
title: Скрипт обновления Exchange-сертификата (WACS + MikroTik)
created: 2026-05-24
updated: 2026-05-24
type: script
tags: [exchange, certificate, letsencrypt, wacs, mikrotik, automation]
sources: [raw/conversation/2026-05-24-cert-renew.md]
confidence: high
---

# exchange-cert-renew-script — Обновление сертификата Exchange через WACS

Скрипт автоматически открывает порт 80 на MikroTik, запускает WACS для HTTP-01 валидации, и закрывает порт обратно.

## Инфраструктура

| Компонент | Значение |
|-----------|----------|
| **Exchange** | MAIL-SRV1 (192.168.2.50), Windows Server 2022 |
| **MikroTik** | 192.168.2.1, SSH пользователь `hermes` (группа full) |
| **SSH-ключ** | `C:\Users\hermes\.ssh\id_mikrotik` (ED25519) |
| **WACS** | `C:\inetpub\letsencrypt\wacs.exe` версия 2.2.9.1701 |
| **NAT правило** | №14, dst-nat 94.130.51.188:80 → 192.168.2.50 |
| **IIS site ID** | 1 (Default Web Site) |

## Параметры WACS

```bash
wacs.exe --source manual ^
  --host mail.ca-ibm.org,autodiscover.ca-ibm.org,webmail.ca-ibm.org ^
  --validation selfhosting --validationmode http-01 ^
  --installation iis --installationsiteid 1 ^
  --accepttos --emailaddress support@ca-ibm.org ^
  --closeonfinish
```

## Scheduled Task (основной способ)

**Имя задачи:** `Exchange Certificate Renew`  
**Пользователь:** `NT AUTHORITY\SYSTEM`  
**Триггер:** Weekly, по пятницам в 23:00  
**Скрипт:** `C:\temp\renew.bat`

```batch
@echo off
cd /d C:\inetpub\letsencrypt

:: Enable NAT port 80
C:\Windows\System32\OpenSSH\ssh.exe -o StrictHostKeyChecking=no -o ConnectTimeout=5 -i C:\Users\hermes\.ssh\id_mikrotik hermes@192.168.2.1 /ip/firewall/nat/set numbers=14 disabled=no >> C:\temp\renew_log.txt 2>&1

:: Run WACS renewal
wacs.exe --renew --closeonfinish >> C:\temp\renew_log.txt 2>&1

:: Post-renew: move cert to My store (Exchange can't see WebHosting) + enable
powershell.exe -NoProfile -NonInteractive -Command "$cert = Get-ChildItem Cert:\LocalMachine\WebHosting | Where-Object {$_.Subject -like '*mail.ca-ibm.org*' -and $_.DnsNameList -contains 'autodiscover.ca-ibm.org'} | Sort-Object NotAfter -Descending | Select-Object -First 1; if ($cert) { $cert | Move-Item -Destination Cert:\LocalMachine\My -Force; Add-PSSnapin Microsoft.Exchange.Management.PowerShell.SnapIn; Enable-ExchangeCertificate -Thumbprint $cert.Thumbprint -Services IIS,SMTP -Confirm:`$false -Force }" >> C:\temp\renew_log.txt 2>&1

:: Disable NAT
C:\Windows\System32\OpenSSH\ssh.exe -o StrictHostKeyChecking=no -o ConnectTimeout=5 -i C:\Users\hermes\.ssh\id_mikrotik hermes@192.168.2.1 /ip/firewall/nat/set numbers=14 disabled=yes >> C:\temp\renew_log.txt 2>&1
```

### Ручной запуск
```batch
:: Однократный выпуск (не renewal)
wacs.exe --source manual --host mail.ca-ibm.org,autodiscover.ca-ibm.org,webmail.ca-ibm.org --validation selfhosting --validationmode http-01 --installation iis --installationsiteid 1 --accepttos --emailaddress support@ca-ibm.org --closeonfinish
```

## Запуск

Выполняется через Scheduled Task от `NT AUTHORITY\SYSTEM` с Highest RunLevel.

### Через WinRM (Python)

```python
import winrm
s = winrm.Session('192.168.2.50', auth=('CAI\\hermes', '...'), transport='ntlm')

script = """
$bat = @'
<содержимое renew.bat>
'@
Set-Content -Path "C:\\temp\\renew.bat" -Value $bat -Force -Encoding ASCII
$action = New-ScheduledTaskAction -Execute "cmd.exe" -Argument "/c C:\\temp\\renew.bat"
Register-ScheduledTask -TaskName "CertRenew" -Action $action -RunLevel Highest -User "NT AUTHORITY\\SYSTEM" -Force
Start-ScheduledTask -TaskName "CertRenew"
"""
s.run_ps(script)
```

## Примечания

- **Важно:** Scheduled Task от CAI\hermes не видит сертификаты (проблемы с правами). SYSTEM работает.
- **Важно:** WACS через WinRM напрямую зависает (нет интерактивной сессии). Scheduled Task решает проблему.
- **NAT выключен всегда:** порт 80 открыт только на время выполнения WACS (1-2 минуты).
- **MikroTik SSH-синтаксис:** команды через `/`, без кавычек. `:put [...]` для получения вывода.
- **Старый сертификат:** перед запуском рекомендуется удалить через `wacs.exe --cancel --friendlyname "..."` или `--delete`.
- **IIS:** после обновления сертификата IIS не требует перезапуска — WACS обновляет биндинги атомарно.

## Связанные сущности

- [[mail-ca-ibm-org]] — Exchange-сервер
- [[server-94-130-51-161]] — Hetzner (MikroTik)
