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

## [2026-07-25] fix | Captive Portal — исправлена кнопка «Подключиться»

- Диагностирована проблема: JS отключал кнопку (`btn.disabled = true`) без вызова `form.submit()`
- Исправлен скрипт на странице портала — убран `btn.disabled`, добавлен `form.submit()`
- Обновлён конфиг `/conf/config.xml` на pfSense (base64)
- Переписана сгенерированная страница `/var/etc/captiveportal_cai_free.html`
- Перезагружен nginx портала
- Дополнен подводный камень #5 в wiki — добавлены симптомы и порядок применения на pfSense
