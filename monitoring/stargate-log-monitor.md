---
title: Stargate Log Monitor
created: 2026-06-16
updated: 2026-06-21
type: entity
tags: [monitoring, stargate, nextcloud, logs, script, cron]
sources: []
confidence: high
---

# Stargate Log Monitor — мониторинг лога Nextcloud

Мониторинг лога Nextcloud (stargate.ca-ibm.org) на предмет ошибок и предупреждений. 
Автоматический анализ каждые 2 часа с доставкой отчёта в Telegram.

## Архитектура

```
stargate (94.130.51.147)                    Hermes Agent
┌─────────────────────────┐       ┌─────────────────────────────┐
│ /root/stargate-log-     │       │ ~/.hermes/scripts/          │
│   monitor.py            │ ←SSH← │   stargate-log-monitor.sh   │
│                         │       │                             │
│ Анализирует последние   │       │ Cron: 0 */2 * * *          │
│ 10000 строк nextcloud.log│      │ Telegram доставка           │
│ Выводит (stdout):       │       │ -5277048106                 │
│ - 🚨 level≥3 с traceback│       └─────────────────────────────┘
│ - ⚠️ level=2            │
│ - 📊 статистика         │
└─────────────────────────┘
```

## Файлы

| Файл | Путь | Назначение |
|------|------|------------|
| Python-скрипт | `/root/stargate-log-monitor.py` (stargate) | Анализ лога |
| Wrapper-скрипт | `~/.hermes/scripts/stargate-log-monitor.sh` | SSH-обёртка |
| Cron-задача | `Stargate Log Monitor` | Каждые 2 часа |

## Расписание

- **Cron:** `0 */2 * * *` (каждые 2 часа)
- **Доставка:** Telegram `-5277048106`
- **Тихий режим:** если нет ошибок level≥3 — пустой stdout, уведомление не шлётся

## Детектируемые события

| Уровень | Тип | Действие |
|---------|-----|----------|
| 🚨 **level≥3** | Ошибки (исключения, падения, 500-е) | Показываются с traceback |
| ⚠️ **level=2** | Предупреждения | Показываются |
| 📊 Статистика | deprecations, dirty reads, размер лога | Всегда в отчёте |

## Игнорируемый шум

- `dirty table reads` — особенность Nextcloud 32 + MySQL, не ошибка
- `deprecated` — устаревшие DI-алиасы в 3rdparty библиотеках

## История

- **2026-06-21 08:02** — **Мониторинг отключён пользователем** (cron paused). Причина: сервер stargate недоступен, скрипт не может выполниться.
- **2026-06-21 06:00–08:00** — SSH-соединение со stargate обрывается на этапе аутентификации (Connection timed out during banner exchange). Порт 22 открыт, хост пингуется. Вероятная причина: зависший SSH-демон или fail2ban/rate-limit.
- **2026-06-20 08:00** — **Сервер недоступен по SSH** (таймаут 60с). Скрипт мониторинга не может соединиться со stargate. Требуется ручная проверка.
- **2026-06-19** — Лог вырос до 17 ГБ; SSH с `Superp@ss2020root` работает, скрипт на stargate зависает (логика анализа читает весь файл ≈17 ГБ в память). Ошибки SMB 6-8 за час (level≥3), предупреждения unscanned files (level=2). Deprecations 5 000-7 200, dirty reads 900-4 000 за 10k строк.
- **2026-06-16 18:01** — cron пересоздан с agent-driven доставкой (предыдущая версия падала с error)
- **2026-06-16 11:45** — создан скрипт `stargate-log-monitor.py`, обёртка, cron (первая версия, no_agent)

## Связанные страницы

- [[stargate-ca-ibm-org]] — Nextcloud сервер
- [[monitoring/index|Мониторинг — сводка]]
