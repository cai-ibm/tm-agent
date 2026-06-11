---
title: OnlyOffice .docx — редактор не активируется (панели инструментов нет)
created: 2026-06-10
updated: 2026-06-10
type: entity
tags: [onlyoffice, nextcloud, debugging, docx, word, bug]
confidence: medium
---

# OnlyOffice .docx — редактор не активируется (панели инструментов нет)

## Проблема

Файлы .docx (и .doc) открываются через OnlyOffice, но панели инструментов редактирования не активированы. Файлы .xlsx (.xls) открываются и редактируются нормально.

## Проверенное и исключённое

### Серверная часть (Nextcloud → OnlyOffice)

- **Конфиг редактора** — полностью корректен (ONLYOFFICE-CONFIG от 17:56 10.06.2026):
  - `permissions.edit` = true
  - `editorConfig.mode` = (not set) → режим edit по умолчанию
  - `editable` = true, `canEdit_var` = true, `format_edit` = true
  - `isUpdateable` = true, `template` = false, `isTempLock` = false
  - `documentType` = "word", `fileType` = "docx"
  - JWT-токен присутствует (полный, HS256)
  - callbackUrl корректный: `https://stargate.ca-ibm.org/index.php/apps/onlyoffice/track?doc=...`

- **Формат .docx** — имеет actions `['view', 'edit', 'review', 'comment', 'encrypt']` (edit присутствует)

- **JWT-секрет** — совпадает на OnlyOffice и Nextcloud: `KNJYTbUI^BRfkub654`

- **Связность OnlyOffice → Nextcloud** — проверено из контейнера: curl до stargate.ca-ibm.org (HTTPS) — 200 OK

### История исследований

- **08.05.2026** — полное расследование (см. `references/2026-05-08-docx-readonly-investigation.md`):
  - Отключение JWT на обоих серверах → не помогло
  - Обновление OnlyOffice 9.1.0 → 9.3.1 → не помогло
  - Обновление Nextcloud 32.0.3 → 32.0.9 → не помогло
  - Явный `mode: "edit"` в конфиге → не помогло
  - Вывод: проблема не в JWT, не в версиях, не в серверной конфигурации

- **10.06.2026** — добавлен дебаг ONLYOFFICE-CONFIG (level 3) в EditorApiController.php

### Логи OnlyOffice (10.06.2026)

- PostgreSQL внутри контейнера был DOWN (volume удалён) → пересоздан контейнер (healthcheck=true, pg accepting connections)
- **После пересоздания:** при открытии .docx через Nextcloud — в логах OnlyOffice **ноль** запросов от браузера. Ни одного docId, ни одного обращения к nginx/docservice. Только старые checkJwt error (мои curl-тесты).
- Это означает, что **браузер не отправляет запросы на ofds.ca-ibm.org**

## Предполагаемая причина

Проблема **не на сервере**. Сервер (Nextcloud + OnlyOffice) работает корректно — конфиг правильный, JWT валидный, permissions.edit=true, callback URL задан.

Проблема на **стороне браузера/сети клиента**:
1. **Браузер не загружает iframe OnlyOffice** — тогда в логах OnlyOffice нет запросов
2. **Браузер блокирует iframe** — из-за CSP, CORS, или блокировщика рекламы
3. **Только для word-редактора** — возможно, js-скрипт word-редактора (web-apps) блокируется, а spreadsheet — нет

### Непроверенные гипотезы

1. **Консоль iframe** — открыть F12 → Console → в выпадающем списке контекстов (выше фильтра) выбрать `ofds.ca-ibm.org` — там будут ошибки
2. **AdBlock/uBlock/другое расширение** — может блокировать загрузку скриптов OnlyOffice
3. **CSP-заголовки** — Nextcloud может отправлять Content-Security-Policy, блокирующую загрузку iframe с ofds.ca-ibm.org
4. **Разные версии js-редактора** — word-редактор может использовать другие ресурсы, чем spreadsheet

### Следующие шаги

1. Проверить консоль браузера в контексте iframe ofds.ca-ibm.org
2. Временно отключить расширения (AdBlock/uBlock)
3. Проверить CSP-заголовки Nextcloud
4. Попробовать открыть ofds.ca-ibm.org/healthcheck напрямую (работает ли вообще)

## Команды управления

```bash
# Дебаг-лог конфига (level 3 — ERROR, попадает в nextcloud.log)
# Добавлен в /var/www/nextcloud/apps/onlyoffice/lib/Controller/EditorApiController.php
# Строка: $this->logger->error("ONLYOFFICE-CONFIG ...")
# Поиск: grep "ONLYOFFICE-CONFIG" /var/www/nextcloud/data/nextcloud.log

# Проверка коннектора
sudo -u www-data php /var/www/nextcloud/occ onlyoffice:documentserver --check
# → Document server https://ofds.ca-ibm.org/ version 9.3.1.10 is successfully connected

# Логи OnlyOffice
docker logs onlyoffice-document-server 2>&1 | grep -v notifyLicenseExpiration | tail -30

# Статус OnlyOffice
curl -s http://localhost:8080/healthcheck                # → true
curl -s -X POST http://localhost:8080/coauthoring/CommandService.ashx \
  -H "Content-Type: application/json" -d '{"c":"version"}'
# → {"error":6} — без JWT, ожидаемо

# PostgreSQL внутри OnlyOffice
docker exec onlyoffice-document-server pg_isready        # → accepting connections
```

## Связанные страницы

- [[stargate-ca-ibm-org]] — Nextcloud сервер
- [[server-94-130-51-161]] — OnlyOffice Document Server (Hetzner)

## История изменений

- **2026-06-10** — Страница создана. Диагностика: конфиг корректен, логи OnlyOffice пусты (браузер не шлёт запросы). Проблема на стороне браузера/сети.