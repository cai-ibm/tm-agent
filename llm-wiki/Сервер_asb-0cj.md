---
links:
  - KVM Виртуальные машины
  - Сеть и подсети
  - Hermes Agent
  - Messaging Gateway
backlinks:
  - Сеть и подсети
updated: 2026-05-27T08:30:40.132711
---
# Сервер asb-0cj (cai-main-b2-tm)

**IP:** 192.168.30.12  
**Роль:** Основной KVM-гипервизор  
**SSH:** cai-root / Cai$2023#

## Характеристики
- **RAM:** 251 GiB, ~222–246 GiB занято
- **Сеть:** br0 (192.168.0.240/24), br30 (192.168.30.12/24), lxcbr0, virbr0, Docker br-8d8d057ca174 (172.18.0.1/16)

## Что работает
- 8 libvirt/KVM виртуальных машин (Windows Server)
- Docker: Odoo 19 + PostgreSQL 15 (остановлены, но bridge висит)
- Samba-шары: virt-images (read-only)

## Важно
- `virsh` по умолчанию идёт в `qemu:///session` — для системных ВМ всегда `-c qemu:///system`
- Диски ВМ: `/var/lib/libvirt/images/` и `/storage/virt-images/`, владелец libvirt-qemu:kvm