---
title: asb-timelapse — timelapse-сервер (motionEye + Veeam)
created: 2026-08-07
updated: 2026-08-07
type: entity
tags: [server, vm, camera, timelapse, veeam, inventory]
sources: []
confidence: high
---

# asb-timelapse (192.168.40.21)

**SSH:** cai@192.168.40.21 / Cai$2023#
**Роль:** Timelapse-сервер — motionEye (камеры), Samba-шара, Veeam Agent
**Виртуализация:** VMware VM (Chassis: vm, Virtualization: vmware)

---

## ХОСТ

| Параметр | Значение |
|---|---|
| Hostname | asb-timelapse (переименован с `debian` 2026-08-07) |
| ОС | Debian GNU/Linux 12 (bookworm) |
| Ядро | 6.1.0-37-amd64 |
| CPU | 4 vCPU — Intel Xeon E5-2680 v4 @ 2.40GHz |
| RAM | 3.8 GiB (982 MiB занято) |
| Swap | 974 MiB (0 занято) |
| Диск (/) | /dev/sda1 97G ext4, 4.0G занято (5%) |
| Uptime | 23 дня (на 2026-08-07) |
| Load | ~1.1 |

## СЕТЬ

| Параметр | Значение |
|---|---|
| IP | 192.168.40.21/24 (DHCP, ens192) |
| Шлюз | 192.168.40.254 |
| DNS | 192.168.40.44, 192.168.40.28 (search ca-ibm.org) |

## СЛУЖБЫ (running)

| Сервис | Назначение |
|---|---|
| motioneye.service | motionEye — веб-интерфейс камер (порт 8765) |
| smbd / nmbd | Samba (шара `[camera]`) |
| ssh | SSH (порт 22) |
| chrony | NTP |
| open-vm-tools / vgauth | VMware guest tools |
| veeamdeployment / veeamenvironment / veeamimmurepo / veeamtransport | Veeam Agent |

## ОТКРЫТЫЕ ПОРТЫ

| Порт | Сервис |
|---|---|
| 22 | SSH |
| 139, 445 | Samba |
| 9082 | motion — стрим камеры B2-Vesu |
| 8765 | motionEye web |
| 7999 | motion (localhost) |
| 6160, 6162 | Veeam transport |

## motionEye / КАМЕРЫ

- **Веб:** http://192.168.40.21:8765 (meyectl, venv `/opt/motioneye-venv`)
- **Конфиг:** `/etc/motioneye/` (motioneye.conf, motion.conf, camera-2.conf)
- **Медиа:** `/var/lib/motioneye/` (18M)

### Камера B2-Vesu (camera-2.conf)
- Разрешение: 3840×2160 (4K), 5 fps
- Стрим: порт 9082
- target_dir: `/var/lib/motioneye/Camera2`

### Каталоги
- `Camera1/` — снимки (892K, последний MK-2026-02-11)
- `Camera2/` — пусто
- `timelapse/` — готовые видео (timelapse_2025-07-02.mp4, 17 MB)

## TIMELAPSE (cron)

**Скрипт:** `/home/cai/create_timelapse.sh`
**Cron:** `59 23 * * *` — ежедневно в 23:59, лог `/home/cai/timelapse.log`

Логика: берёт снимки `Camera1/$TODAY/*.jpg` → ffmpeg (30 fps, libx264) → `timelapse/timelapse_$DATE.mp4`

## SMB

- Шара `[camera]` → `/var/lib/motioneye`
- Симлинк `/home/cai/camera_share` → `/var/lib/motioneye/Camera1`

## SMB — asb-fs1 (Media/Timelapse/K1)

- **Сервер:** asb-fs1.ca-ibm.org (192.168.40.5), шара `Media`
- **Путь:** `//192.168.40.5/Media` → `/mnt/asb-fs1-media` (в fstab используется **IP**, не hostname — DNS не готов на момент монтирования при загрузке)
- **Целевая папка снимков:** `/mnt/asb-fs1-media/Timelapse/K1`
- **Учётка:** `cai\hermes` / Superp@ss2020hermes
- **Credentials:** `/root/.smb-asb-fs1` (chmod 600)
- **Fstab:** `//192.168.40.5/Media /mnt/asb-fs1-media cifs credentials=/root/.smb-asb-fs1,vers=3.0,iocharset=utf8,nofail,uid=1000,gid=1000 0 0`
- **Пакеты:** установлены `cifs-utils`, `smbclient`

## Устойчивость к перезапускам (проверено 2026-08-13)

- В `motioneye.service` добавлена `RequiresMountsFor=/mnt/asb-fs1-media` — systemd поднимает SMB-монтирование **до** motionEye; если шара недоступна, motionEye не стартует
- **Pitfall:** при загрузке монтирование падало с `could not resolve address for asb-fs1.ca-ibm.org` (DNS не готов). Исправлено заменой hostname на IP `//192.168.40.5/Media` в fstab
- Проверено двумя перезагрузками: монтирование и motionEye поднимаются автоматически, запись в `Timelapse/K1` — WRITE OK

## Камера K1 (motionEye)

- **camera-1.conf:** RTSP `rtsp://192.168.51.100:554/`, user `admin:745745vF`, 4K 3840×2160, 2 fps
- **target_dir:** `/mnt/asb-fs1-media/Timelapse/K1` (пишет на SMB-шару)
- **stream_port:** 9081
- ⚠️ Камера K1 (192.168.51.100) **недоступна** («No route to host») — так и должно быть на текущий момент; motionEye ретраит подключение
- Камера B2-Vesu (camera-2.conf): RTSP `192.168.115.76`, target_dir `/var/lib/motioneye/Camera2`, stream 9082

## VEEAM

- Сервисы: deployment, environment, immurepo, transport
- Конфиг: `/etc/veeam/` (veeamdeployment.conf, veeamtransport.conf)
- Логи: `/var/log/VeeamBackup`

## ПРИМЕЧАНИЯ

- Docker и LXC **не установлены**
- `/etc/hosts`: добавлена `127.0.1.1 asb-timelapse` (2026-08-07, убирает предупреждение sudo)
- Пароль `cai` сменён на `Cai$2023#` (2026-08-07)

## СВЯЗИ

- [[entities/server-asb-timelapse|Сервер asb-timelapse]]
- [[concepts/network-subnets|Сеть и подсети]]
