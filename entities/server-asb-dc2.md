---
title: ASB-DC2 — контроллер домена (192.168.40.2)
created: 2026-11-09
updated: 2026-11-09
type: entity
tags: [server, network, service, winrm, print, vm]
sources: []
confidence: high
---
# ASB-DC2 — контроллер домена TM (192.168.40.2)

**FQDN:** asb-dc2.ca-ibm.org · **IP:** 192.168.40.2/24 (шлюз 192.168.40.254) · **DNS:** 192.168.30.9, 127.0.0.1
**Роль:** AD DS + DNS + Global Catalog, дополнительный контроллер домена ca-ibm.org (сайт TM).
**Платформа:** ВМ QEMU/KVM (модель `QEMU Standard PC (Q35 + ICH9, 2009)`), 4 GB RAM, C: 29 GB (14 свободно).

## Система
| Параметр | Значение |
|---|---|
| ОС | Windows Server 2022 Standard, 10.0.20348 |
| Службы | NTDS Running, DNS Running, Spooler Running/Automatic |
| Роли | AD-Domain-Services **Installed**, Print-Server **Installed** (с 09.11.2026) |
| DNS-зоны (DS-integrated) | ca-ibm.org, _msdcs.ca-ibm.org, interbudmontaz.local, **server**, TrustAnchors |
| Часовой пояс | West Asia Standard Time (UTC+5) — с 09.11.2026 (было UTC+3) |
| Доступ | WinRM 5985 (cai\hermes, ntlm); RDP-порт слушается |

## Печать (резерв принт-сервера)
Полная копия конфигурации [[entities/192-168-20-253-asb-prtsrv]]: **67 очередей**, те же IP/порты/драйверы,
SDDL-права очередей совпадают. Источник истины — asb-prtsrv; репликация ежедневно через `printbrm`:

| Задача | Где | Время | Что делает |
|---|---|---|---|
| `PrintReplica-Export` | asb-prtsrv | 03:00 | `printbrm -b` → `C:\PrintBackup\replica\current.printerExport` (201 MB, хранится 7 копий) → копия на `\\ASB-DC2\PrintReplica$` |
| `PrintReplica-Import` | asb-dc2 | 03:20 | `printbrm -r -o force` (аддитивно + перезапись, очереди не удаляются), лог `C:\PrintReplica\import.log` |

Скрипты: `C:\PrintBackup\replica-export.ps1` (prtsrv), `C:\PrintReplica\import.ps1` (dc2), обе задачи — SYSTEM.
Шара `PrintReplica$` на dc2: Change только для `CAI\ASB-PRTSRV$` (NTFS Modify по SID).

**Имя `printer.server`:** CNAME в зоне `server` → сейчас указывает на asb-prtsrv. Для приёма этого имени на dc2:
`LanmanServer\Parameters\DisableStrictNameChecking=1`, `OptionalNames=printer.server` уже прописаны.
**SPN `cifs/printer.server` сейчас принадлежит ASB-PRTSRV** (на кого указывает CNAME, у того и SPN) — при переключении
CNAME на dc2 SPN надо перенести (`setspn -D ... ASB-PRTSRV` + `setspn -S ... ASB-DC2`), иначе клиенты получат ошибку 1396.
Переключение — `C:\PrintReplica\switch-printer-server.ps1 -Target dc2` (обратно `-Target prtsrv`).
Клиентский TTL DNS — 285 c, поэтому переключение «доезжает» за ~5 минут.

## Время (GPO)
- Политика **`tm-Часовой_пояс_UTC+5_DC`** (Id bd84c4c5, Enforced) на `OU=TM,OU=Domain Controllers`:
  `HKLM\SYSTEM\CurrentControlSet\Control\TimeZoneInformation\TimeZoneKeyName = West Asia Standard Time` +
  включён NTP-сервер (`TimeProviders\NtpServer\Enabled=1`), чтобы DC отдавал время клиентам.
- **NTP-синхронизация DC** уже была настроена раньше enforced-политикой «Политика синхронизации времени
  для контроллера домена» (linked на `OU=Domain Controllers`, NoOverride): `Type=NTP`,
  `NtpServer=time.windows.com,0x9`, `SpecialPollInterval=1024`. Она перекрывает любые NTP-настройки
  на уровне OU — поэтому NTP в `tm-Часовой_пояс_UTC+5_DC` не дублируется.
- Проверено 09.11.2026: ASB-DC2 и GC1 — UTC+5, источник `time.windows.com`; GC-EU (OU=EU) — UTC+3, политика не применяется.
- Применение к уже работающему DC требует `tzutil /s "West Asia Standard Time"` (или перезагрузки) — GPO
  пишет только ключ реестра.

## Замечания
- `Get-GPO`/`Get-GPInheritance` на dc2 и GC1 падают с `0x80072020` (operations error); GPMC работает только на GC-EU.
  Управление GPO вести с GC-EU.
- На dc2 не транслируются локальные/групповые имена (`BUILTIN\Administrators`, `CAI\Domain Admins`, `Everyone`)
  через ACL API — права задавать по SID.
- Планировщик учитывает локальный TZ: задача импорта DC2 «03:20» = 03:20 по UTC+5 только после смены пояса.

## Связи
- [[entities/192-168-20-253-asb-prtsrv]] — основной принт-сервер (источник реплики)
- [[concepts/network-subnets]] — подсети 192.168.40.0/24 и 192.168.20.0/24
- [[inventory/cai-main-b2-tm]] — KVM-хост asb-0cj (вероятная площадка ВМ)
