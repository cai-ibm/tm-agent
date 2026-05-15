---
title: SMTP Relay ca-ibm.org (192.168.2.50:2526)
created: 2026-05-15
updated: 2026-05-15
type: entity
tags: [smtp, exchange, service, network, script]
sources: [raw/memory/agent-knowledge-2026-05-15.md]
confidence: high
---

# SMTP Relay ca-ibm.org

Внутренний анонимный SMTP-релей для отправки почты из скриптов и сервисов в домене `ca-ibm.org`. Работает на том же хосте, что и [[mail-ca-ibm-org]] (Exchange Server 2019).

## Параметры

| Параметр | Значение |
|----------|----------|
| **IP** | 192.168.2.50 |
| **Порт** | 2526 |
| **Аутентификация** | Нет (анонимный) |
| **Banner** | `Microsoft ESMTP MAIL Service` |
| **Роль** | Exchange Frontend Transport / IIS SMTP |
| **Отправитель по умолчанию** | `noreply@ca-ibm.org` |
| **Доступ** | Только из внутренней сети 192.168.2.0/24 |

## Отправка писем

### Скрипт sendmail.py

```bash
python3 /home/cai/sendmail.py 'user@ca-ibm.org' 'Тема' 'Текст письма'
```

Опции:
- `--from security@ca-ibm.org` — задать отправителя
- `--html` — HTML-формат
- `--attach /path/to/file` — вложение
- Поддерживает кириллицу

Навык Hermes: `send-email` — полная документация по SMTP-релею и скрипту.

### Примеры

```bash
# Простое письмо
python3 /home/cai/sendmail.py 'sobolevrv@ca-ibm.org' 'Тест' 'Проверка связи'

# С вложением
python3 /home/cai/sendmail.py 'user@ca-ibm.org' 'Отчёт' /tmp/report.txt --attach /tmp/data.csv

# С кастомным отправителем
python3 /home/cai/sendmail.py 'user@ca-ibm.org' 'Аудит' /tmp/audit.txt --from security@ca-ibm.org
```

## Особенности

- Без шифрования (plain SMTP, порт 2526 — нестандартный)
- Без аутентификации — только внутри сети
- Принимает любые адреса `@ca-ibm.org`

## Связанные сущности

- [[mail-ca-ibm-org]] — Exchange Server, на котором работает релей
- [[stargate-ca-ibm-org]] — Nextcloud, может использовать релей для уведомлений
- [[server-94-130-51-161]] — n8n на Hetzner, может отправлять через этот релей при наличии доступа к сети

## История изменений

- **2026-05-15** — Страница создана при инициализации LLM Wiki
