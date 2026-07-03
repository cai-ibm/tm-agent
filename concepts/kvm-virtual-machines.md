---
backlinks:
  - Сервер asb-0cj
updated: 2026-05-27T08:30:40.353167
---
# KVM Виртуальные машины (asb-0cj)

## Работающие (8)
| ВМ | RAM | vCPU | Назначение |
|----|-----|------|------------|
| win2k16-DC1 | 4G | 4 | Старый домен-контроллер |
| win2k22-wsus | 8G | 4 | WSUS-сервер |
| win2k22-printsrv | 8G | 8 | Print-сервер |
| win2k22-e1crdp | 64G | 10 | 1C RDP |
| win2k22-sqlsrv | 63G | 8 | SQL Server |
| win2k22-obmenka | 8G | 8 | Обмен данными |
| win2k22-DC-1 | 8G | 4 | Новый домен-контроллер (выключен) |
| win2k22-e1csrv | 58G | 12 | 1C Сервер |

## Выключенные (8)
win10-ok-wapp, win10-test-rdp, win10test, win11-esit, win2k22-e1c-test, win2k22-ims, win2k22-jas-rdp, win2k22-nfs, win7anydesk

## Сеть
Все ВМ подключены к br0 (192.168.0.x) и/или br30 (192.168.30.x)