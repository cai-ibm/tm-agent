---
created: 2026-05-21
tags: [сервер, windows, sql, 1c, mssql]
---

# EU-E1C (eu-e1c.ca-ibm.org) — обзор

## Общая информация
- **IP**: 192.168.2.35
- **OS**: Windows Server 2022 Standard (10.0.20348), 64-bit
- **Домен**: ca-ibm.org
- **VM**: VMware20,1
- **RAM**: 34 GB
- **Диск C**: 299 GB (254.7 GB свободно — 85%)
- **Сеть**: Ethernet0, 192.168.2.35/24

## Доступ
- **WinRM (5985)**: ✅ открыт
- **SMB (445)**: ✅ открыт
- **RDP (3389)**: ✅ открыт
- **MS SQL (1433)**: ✅ открыт
- **Учётка**: cai\hermes (доменная, пароль Superp@ss2020hermes)
- **Подключение**: WinRM через pywinrm (NTLM)

## MS SQL Server
- **Версия**: SQL Server 2022 (MSSQL16) — 16.0.1000.6
- **Издание**: Enterprise Edition
- **Инстанс**: MSSQLSERVER
- **Коллация**: Cyrillic_General_CI_AS
- **Путь**: `C:\Program Files\Microsoft SQL Server\MSSQL16.MSSQLSERVER\`
- **sqlcmd**: `C:\Program Files\Microsoft SQL Server\Client SDK\ODBC\170\Tools\Binn\SQLCMD.EXE`
- **Служба**: MSSQLSERVER (Running, Auto)
- **SQL Agent**: SQLSERVERAGENT (Running, Auto)

## 1С:Предприятие 8.3
- **Версия**: 8.3.27.1989 (x86-64)
- **Путь**: `C:\Program Files\1cv8\8.3.27.1989\bin\`
- **Служба**: `1C:Enterprise 8.3 Server Agent (x86-64)`
  - **Состояние**: Running, Auto
  - **Бинарь**: `ragent.exe -srvc -agent -regport 1541 -port 1540 -range 1560:1591`
  - **Хранилище**: `C:\Program Files\1cv8\srvinfo`
- **rac.exe**: `C:\Program Files\1v8\8.3.27.1989\bin\rac.exe`

## Другое ПО
- **ManageEngine UEMS** — Agent (Endpoint Security)
- **VMware Tools** (VGAuthService, vm3dservice, VMTools)
- **WinRM** — активен

## Открытые порты
| Порт | Сервис |
|------|--------|
| 445 | SMB |
| 1433 | MS SQL |
| 3389 | RDP |
| 5985 | WinRM HTTP |
