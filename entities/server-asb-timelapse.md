---
links:
  - Сеть и подсети
  - Hermes Agent
backlinks:
  - Сеть и подсети
updated: 2026-08-07
---
# Сервер asb-timelapse

**IP:** 192.168.40.21  
**Роль:** Timelapse-сервер (Debian)  
**SSH:** cai / Cai$2023#

## Характеристики
- **ОС:** Debian, ядро 6.1.0-37-amd64
- **Хостнейм:** asb-timelapse (переименован с `debian`)

## Изменения (2026-08-07)
- Пароль пользователя `cai` сменён на `Cai$2023#`
- Хостнейм переименован на `asb-timelapse`
- В `/etc/hosts` добавлена запись `127.0.1.1 asb-timelapse` (убирает предупреждение sudo «unable to resolve host»)

## Связи
- [[concepts/network-subnets|Сеть и подсети]]
- [[concepts/hermes-agent|Hermes Agent]]
