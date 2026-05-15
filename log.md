# Wiki Log

> Хронология всех действий с вики. Только дополнение.
> Формат: `## [YYYY-MM-DD] действие | тема`
> Действия: ingest, update, query, lint, create, archive, delete
> При превышении 500 записей — ротация: переименовать в log-YYYY.md, начать новый.

## [2026-05-15] create | Инициализация вики
- Пересобрана структура: убрана папка LLM Wiki/, всё в корне vault
- CLAUDE.md переписан для Hermes Agent (русский, инструменты Hermes, workflow)
- Создана страница: [[mail-ca-ibm-org]] — Exchange Server 2019, данные аудита и оценки апгрейда
- Источник: agent-knowledge-2026-05-15.md (дамп памяти Hermes)

## [2026-05-15] create | Заполнение сущностей инфраструктуры
- Создана страница: [[smtp-relay-ca-ibm]] — анонимный SMTP-релей 192.168.2.50:2526, sendmail.py
- Создана страница: [[stargate-ca-ibm-org]] — Nextcloud 32.0.9, S3, OnlyOffice, ~2.5M файлов
- Создана страница: [[server-94-130-51-161]] — Hetzner VPS, nginx, n8n, OnlyOffice DS
- Обновлён index.md: 1 → 4 страницы
- Все страницы перекрёстно связаны [[wikilinks]]
