---
name: tmcell-check
title: TMCELL Check — ручная проверка и покупка пакетов
description: Ручная проверка баланса и пакетов TMCELL + покупка интернет-пакетов через buy_package_oneshot.py
category: devops
triggers:
  - tmcell проверь
  - tmcell check
  - tmcell сводка
  - tmcell купи
  - tmcell purchase
---

# TMCELL Check — ручная проверка и покупка

Скрипт для ручной проверки баланса и активных пакетов TMCELL с единым форматированным выводом.

## Использование — проверка

```bash
python3 ~/.hermes/skills/devops/tmcell-check/scripts/tmcell_check.py          # сводка по всем
python3 ~/.hermes/skills/devops/tmcell-check/scripts/tmcell_check.py К3       # один
python3 ~/.hermes/skills/devops/tmcell-check/scripts/tmcell_check.py К1,К2    # несколько
```

## Формат вывода (один аккаунт)

```
📊 **К3** (64602289)

▫️ Тариф: Sada
▫️ Баланс: 1.84 manat
▫️ Пакетов: 2

📦 **Internet-200 (20 Gb)**
   💾 Остаток: 11.36 Gb
   📅 Срок: до 23.09.2026 08:40
   ✅ Активен
📦 **Internet-200 (20 Gb)**
   💾 Остаток: 20 Gb
   📅 Срок: до 23.10.2026 08:35
   ✅ Активен
```

## Статусы

- `✅ Активен` — пакет есть и работает
- `⚠️ Нет пакета` — баланс есть, пакетов нет
- `❌ Ошибка` — не удалось проверить
- `⏳ Истекает` — осталось < 24ч

## Аккаунты

| Имя | Телефон |
|-----|---------|
| К1 | 62057470 |
| Дамба | 63275073 |
| К2 | 63886456 |
| К3 | 64602289 |
| Мишка | 65193617 |
| ВВП | 61424509 |
| Сарыбаев | 62068923 |

## Покупка пакетов — рабочий способ

**`~/.hermes/scripts/buy_package_oneshot.py`** — прямые POST через `requests` с CSRF. Проверено 23.09.2026 на К3 (Internet-200, списано 200 manat).

```bash
/usr/bin/python3 ~/.hermes/scripts/buy_package_oneshot.py --name К3 --package Internet-200
```

Порядок: GET `/` → CSRF → `sleep(1)` → POST login → GET `?internet_bukja` → POST `packetId=<id>&int_paket_al=ok` + CSRF → поллинг баланса.

### ID пакетов (data-id)

| Пакет | data-id | Цена | Трафик | Срок |
|-------|---------|------|--------|------|
| Internet-3 | 28 | 3 manat | 50 Mb | 30 дн |
| Internet-5 | 22 | 5 manat | 100 Mb | 30 дн |
| Internet-10 | 29 | 10 manat | 250 Mb | 30 дн |
| Internet-60 | 96 | 60 manat | 1500 Mb | 30 дн |
| Internet-160 | 6 | 160 manat | 4 Gb | 30 дн |
| Internet-200 | 97 | 200 manat | 20 Gb | 30 дн |

## Питфоллы

- **CSRF обязателен** — POST без CSRF даёт ложный «Üstünlik!» (заказ не обрабатывается). POST с CSRF работает по-настоящему.
- **Баланс списывается НЕ сразу** — карточка нового пакета появляется почти мгновенно (~0с), а списание с баланса приходит с задержкой 90–120 секунд. Скрипт детектит успех по `cards_after > cards_before`, поэтому в его отчёте баланс может остаться старым — это НЕ признак неудачи.
- **Верифицировать всегда отдельно** — `tmcell_check.py ИМЯ` с паузой ≥90с. Признак успеха: новая карточка со сроком сегодня+30д И списание ~цены пакета.
- **Işjeň поиск** — регистронезависимый, без привязки к `</span>`
- **Gozleg Protection** — `time.sleep(1.0)` между GET и POST
- **Read timed out** — случайные зависания Gozleg, до 5 ретраев
- **Браузерный путь недоступен в этом окружении** — Chrome не установлен (только Playwright-сборка), browser-harness демон падает с `chrome-not-running`. Покупки делать только через `buy_package_oneshot.py`.
- **Пакет не суммируется** с активным — старый продолжает тикать до истечения, баланс трафика не объединяется.

## Связанные страницы

- [[tmcell-accounts]]
- [[tmcell-monitoring]]
- [[tmcell-k3-64602289]]
