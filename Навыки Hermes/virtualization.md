# virtualization

**Категория:** devops
**Описание:** Hypervisor management and VM migration — KVM/libvirt, ESXi, disk format conversions, and cross-hypervisor migration workflows.

## Возможности

- Управление KVM/libvirt VM (virsh)
- Live snapshot (zero downtime) для миграции
- Конвертация qcow2 ↔ VMDK (monolithicSparse, streamOptimized)
- Загрузка больших файлов на ESXi (SCP / HTTP)
- Импорт VMDK в VMFS через vmkfstools
- Создание VMX конфигов для ESXi
- Windows Boot Repair после кросс-гипервизорной миграции
- Восстановление цепочки дисков (backing chain recovery)

## Этапы миграции KVM→ESXi

1. **Live snapshot** на KVM
2. **qemu-img convert** qcow2 → VMDK
3. **SCP/HTTP** на ESXi datastore
4. **vmkfstools -i** импорт в VMFS thin
5. **VMX файл** + регистрация
6. **Windows boot repair** (BCD fix)

## Связанные навыки

- [[kvm-to-esxi-migration]] — полный процесс миграции
- [[server-admin]] — SSH доступ к хостам

---

*Создано Hermes Agent. Obsidian — основной источник.*
