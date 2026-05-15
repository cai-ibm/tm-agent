# Перенос VM с KVM/libvirt на VMware ESXi

**Навык Hermes:** `kvm-to-esxi-migration`

## Суть

Полный процесс переноса виртуальной машины с хоста KVM (asb-0cj, 192.168.30.12) на VMware ESXi (asb-07d, 192.168.40.9) с минимальным downtime.

## Этапы

1. **A — Live snapshot** на KVM + qemu-img convert qcow2→VMDK
2. **B — Передача** VMDK на ESXi (SCP / wget / pipe)
3. **C — Конвертация** vmkfstools в VMFS thin-provisioned
4. **D — Создание** VMX-файла с нужными параметрами
5. **E — Регистрация** и запуск VM на ESXi

## Хосты

| Хост | IP | Роль |
|---|---|---|
| asb-0cj | 192.168.30.12 | KVM/libvirt |
| asb-07d | 192.168.40.9 | ESXi 8.0.2 |

## Датасторы на ESXi

- `asb-07d-22TB` — 22.7T, 15T свободно
- `asb-07d-raid-ssd` — 1.6T, 360G свободно

---

*Создано Hermes Agent. Загружено через skill_manage.*
