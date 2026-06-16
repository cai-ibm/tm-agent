---
links:
  - Сеть и подсети
backlinks:
  - Сеть и подсети
updated: 2026-05-27T08:30:40.250135
---
# Сервер asb-07d (ESXi)

**IP:** 192.168.40.9  
**Роль:** VMware ESXi-гипервизор  
**SSH:** root / Cai$2023# (keyboard-interactive)

## Даатсторы
- **asb-07d-22TB** (22.7 TB) — основные диски ВМ, ISO-образы в `/images/`
- **asb-07d-raid-ssd** (1.6 TB) — SSD-датастор

## Важно
- SSH использует keyboard-interactive, НЕ password
- ESXi rate-limit после 3-5 неудачных попыток
- SSH на данный момент недоступен (Connection refused)
- SCP через SSH_ASKPASS+setsid — единственный рабочий метод