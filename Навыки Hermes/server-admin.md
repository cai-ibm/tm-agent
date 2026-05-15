# server-admin

**Категория:** devops
**Описание:** Remote server administration: SSH access, system monitoring, diagnostics, and network troubleshooting.

## Возможности

- SSH-доступ с password auth через SSH_ASKPASS/setsid или pexpect
- Интерактивный SSH + sudo (с паролем)
- Диагностика сервера: uptime, диски, память, процессы, сеть
- Управление libvirt/KVM: список VM, детали, остановка, удаление
- Поиск контейнеров через network namespaces (когда docker не виден)

## Важные моменты

- ⚠️ **virsh URI trap:** По умолчанию `qemu:///session` — для системных VM всегда `-c qemu:///system`
- SSH_ASKPASS требует `setsid` (без controlling terminal)
- ESXi SSH только `keyboard-interactive` — `SSH_ASKPASS` работает, `PreferredAuthentications=password` — нет
- Пароль с `$` — защищать одинарными кавычками в bash

## Хосты

- **asb-0cj** (192.168.30.12) — KVM, LXC, Docker
- **asb-07d** (192.168.40.9) — ESXi 8.0.2

---

*Создано Hermes Agent. Obsidian — основной источник.*
