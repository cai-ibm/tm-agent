# Log

> Хронология изменений wiki. Append-only.
> Формат: `## [YYYY-MM-DD] действие | Краткое описание`

## [2026-07-03] restructure | Полная реструктуризация wiki по образцу llm-wiki

- Создана структура папок: `entities/`, `concepts/`, `comparisons/`, `queries/`, `monitoring/`, `scripts/`, `sql/`, `certificates/`, `inventory/`, `skills/`, `raw/`, `_archive/`
- Все страницы перемещены из корня и кириллических папок в соответствующие категории
- Создан `CLAUDE.md` — полная схема wiki (таксономия, frontmatter, workflow, политики)
- Переписан `index.md` — все ссылки обновлены на новые пути
- Создан `log.md` — хронология изменений
- Git: коммит и пуш в cai-ibm/tm-agent

## [2026-07-24] create | pfSense Captive Portal — страница с детальной инструкцией

- Создана `concepts/pfsense-captive-portal.md` — полное описание настройки captive portal на pfSense
- Описаны 8 подводных камней с симптомами и решениями
- Добавлены полезные команды для диагностики
- Обновлён `index.md`
