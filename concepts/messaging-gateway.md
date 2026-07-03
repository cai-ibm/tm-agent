---
backlinks:
  - Сервер asb-0cj
  - Hermes Agent
updated: 2026-05-27T08:30:41.105239
---
# Messaging Gateway

**Роль:** Мост между Hermes Agent и мессенджерами (Telegram).

## Telegram
- **Основной чат:** DM с Roman Soboliev
- **Женя Панфилов:** chat_id = 1251343071
- **Telegram group IDs:** всегда с минусом (-XXXXXXXXX)

## История проблем
- **Systemd auto-restart loop** — Exit code 203/EXEC из-за пути к python под /root/
- **Решение:** ExecStart → `/home/cai/.local/bin/hermes gateway run --replace`
- **Старые процессы** нужно убивать явно перед запуском нового