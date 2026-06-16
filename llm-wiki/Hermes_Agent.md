---
links:
  - Messaging Gateway
  - Obsidian Vault
  - TMCELL Мониторинг
backlinks:
  - Сервер asb-0cj
  - Obsidian Vault
updated: 2026-05-27T08:30:40.994934
---
# Hermes Agent

**Расположение:** `/home/cai/.hermes/`  
**Документация:** hermes-agent.nousresearch.com

## Компоненты
- **CLI:** `hermes` (через `~/.local/bin/hermes`)
- **Gateway:** Telegram/Discord-мост
- **Skills:** процедурная память в `~/.hermes/skills/`
- **Cron:** встроенный планировщик (Hermes cronjob)
- **Memory:** персистентная память (факты, предпочтения)

## Провайдеры
- **deepseek** — основной (deepseek-v4-flash)
- **openrouter** — запасной

## Особенности настройки
- Gateway systemd unit: ExecStart должен указывать на `/home/cai/.local/bin/hermes gateway run --replace` (не на /root/venv)
- `SSH_ASKPASS+setsid` для SSH c паролем