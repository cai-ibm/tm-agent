# Wiki Log

> Хронология всех действий с вики. Только дополнение.
> Формат: `## [YYYY-MM-DD] действие | тема`
> Действия: ingest, update, query, lint, create, archive, delete
> При превышении 500 записей — ротация: переименовать в log-YYYY.md, начать новый.

## [2026-06-08] update | n8n 2.13.4 → 2.23.4, MCP SDK
- Обновлён n8n на Hetzner (94.130.51.161): pull образа, recreate контейнера
- SDK-валидация через MCP заработала
- Создан и выполнен workflow test-v2 с Manual Trigger через MCP
- В Obsidian задокументирована конфигурация MCP (URL, инструменты, конфиг)

## [2026-06-13] update | drawDB on Докер1, алиас сервера
- Добавлен алиас «Докер1» для [[server-94-130-51-161]]
- Развёрнут drawDB (drawdb.ca-ibm.org) на порту 3001, через NPM + LE
- Создана страница [[drawdb-ca-ibm-org]]

## [2026-06-01] create | IBT-files NFS
- Смонтирована NFS-шара `192.168.3.253:/IBT-files` → `/mnt/ibt-nfs` на hermes-eu
- Прописана в /etc/fstab: NFSv4.1, rw,hard,intr,_netdev
- Ёмкость: 32T (25T занято, 7.9T свободно)
- Создана страница: [[ibt-files-nfs]]
- Обновлён index.md: 11 → 12 страниц

## [2026-05-19] create | Claude Code Container
- Создан Docker-контейнер `claude-code` на хосте hermes-eu (192.168.2.34, внешний 94.130.51.188)
- Образ: `claude-code:latest` (Ubuntu 24.04 + Node.js 20 + Claude Code 2.1.144)
- Документация: [[entities/claude-code-container]]
- Dockerfile сохранён в `/workspace/claude-code-docker/Dockerfile`

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

## [2026-05-15] update | Повторный аудит mail.ca-ibm.org
- Свежее сканирование: порты, заголовки, TLS, OWA/ECP/API/Autodiscover
- 🔴 Обнаружен порт 2526 (SMTP relay) открытым наружу — был закрыт при аудите 09.05
- Обновлён [[mail-ca-ibm-org]]: +порт 2526 в таблицу, +CSP статус, +история

## [2026-05-15] create | Nginx Proxy Manager на 192.168.2.31
- Поднята отдельная ВМ Ubuntu 24.04, установлен Docker 29.5.0 + Compose v5.1.3
- Развёрнут Nginx Proxy Manager (jc21/nginx-proxy-manager:latest)
- Порты: 80, 81 (админка), 443. Веб-интерфейс доступен.
- Создана страница: [[nginx-pm-192-168-2-31]]
- Обновлён index.md: 4 → 5 страниц

## [2026-05-15] create | techbau.org за NPM
- Создан proxy host в NPM: techbau.org → 192.168.2.39:80 (id=1)
- WordPress на nginx/1.24.0. Прокси работает (HTTP 200).
- ✅ NAT 94.130.51.188 настроен, сайт доступен снаружи
- Создана страница: [[eu-sg-ca-ibm-org]]

## [2026-05-15] refactor | Упрощение: Exchange и SG убраны из NPM
- Exchange (id=7) и SecurityGateway (id=4) удалены из NPM — несовместимость NTLM с HTTP-прокси
- Схема: Exchange и SG на прямом NAT, NPM только для techbau.org
- Сертификаты Let's Encrypt для Exchange/SG удалены
- Обновлены страницы: [[mail-ca-ibm-org]], [[nginx-pm-192-168-2-31]]
- Удалена страница [[eu-sg-ca-ibm-org]] (SG на прямом доступе)
- Обновлён [[nginx-pm-192-168-2-31]]: добавлена таблица проксируемых хостов
- Обновлён index.md: 5 → 6 страниц

## [2026-05-15] create | Exchange за NPM (L7 proxy)
- Proxy host id=2: mail.ca-ibm.org, autodiscover.ca-ibm.org, webmail.ca-ibm.org → 192.168.2.50:443
- Advanced config: таймауты 600s, буферизация off, ssl_verify off, HSTS, WebSocket
- Let's Encrypt сертификат на все 3 домена
- Проверено: OWA 302, Autodiscover 401 (штатно) — все домены работают
- Обновлён [[nginx-pm-192-168-2-31]]: +Exchange в таблице прокси-хостов
- Обновлён [[mail-ca-ibm-org]]: секция «Проксирование через NPM»

## [2026-05-15] create | eu-sg.ca-ibm.org за NPM
- ALT-N SecurityGateway 11.0.3 (192.168.2.36:4443)
- Proxy host id=4, Let's Encrypt SSL
- HTTPS работает (HTTP 200)
- Создана страница: [[eu-sg-ca-ibm-org]]

## [2026-05-15] update | Exchange IIS header hardening
- Установлены outbound rewrite rules через IIS URL Rewrite Module на MAIL-SRV1
- Скрыты 4 заголовка: Server, X-FEServer, X-AspNet-Version, X-OWA-Version
- Значения обнулены (пустые строки), информация не утекает
- Доступ через WinRM (CAI\hermes, порт 5985)
- Server header полностью не удалён (HTTP.sys, требует реестр + перезапуск службы)
- Создана страница: [[exchange-iis-header-hardening]]

## [2026-05-15] update | mail.ca-ibm.org — исправление неточностей
- CU13 → CU15 (build 15.2.1748.36, Aug25SU)
- WinRM: cai\sr → cai\hermes (рабочая учётка)
- HSTS и X-Frame-Options: "отсутствуют" → присутствуют
- Disclosure-заголовки: "не исправлены" → исправлены через URL Rewrite
- X-Powered-By: убран (никогда не присутствовал, путаница с X-AspNet-Version)
- Добавлена ссылка на [[exchange-iis-header-hardening]]


## [2026-05-15] update | Exchange — скрытие версии в URL
- Outbound rule: вырезает 15.2.1748/ из HTML (precondition IsHTML)
- Inbound rule: пробрасывает /owa/auth/themes/ → /owa/auth/15.2.1748/themes/
- Версионированные пути больше не видны в исходном коде страницы
- Обновлены: exchange-iis-header-hardening.md, mail-ca-ibm-org.md


## [2026-05-15] fix | Exchange — сужен regex outbound-правила
- Старый: (owa/auth)/15\.2\.\d+(/.*) — ломал JS-переменные a_sLgn (редирект после логина → 404)
- Новый: (owa/auth)/15\.2\.\d+(/themes/.*) — только ресурсные пути, JS-переменные не трогает
- 15.2.1748: 0 вхождений в HTML, логин должен работать


## [2026-05-15] rollback | Exchange IIS Rewrite — откат всех изменений
- Удалены все outbound правила (header + URL stripping)
- Удалены все inbound правила
- Очищены allowed server variables
- Precondition IsHTML удалён
- Причина: outbound URL-правила ломают логин OWA (404 в браузере, curl работает)
- Обновлены: exchange-iis-header-hardening.md, mail-ca-ibm-org.md


## [2026-05-15] fix | Exchange — X-Frame-Options конфликт
- Одно outbound-правило: RESPONSE_X-Frame-Options → SAMEORIGIN
- Убирает конфликт SAMEORIGIN/DENY (браузер падал в DENY)
- Исправляет PDF-превью в OWA
- Создана страница: [[exchange-xframe-fix]]


## [2026-05-15] memory-offload | Exchange — выгрузка деталей в Obsidian
- Создана страница: [[agent-memory-exchange]]
- 3 записи agent memory заменены на 1 короткий указатель
- Память: 95% → 70%

## [2026-05-18] consolidate | Exchange — объединение заметок
- Удалено: agent-memory-exchange.md (устаревший дамп памяти)
- Объединены в concepts/exchange-iis-headers.md: exchange-iis-header-hardening.md + exchange-xframe-fix.md + entities/exchange-iis-header-rewrite-2026-05-16.md
- Перенесено: entities/exchange-extended-protection.md → concepts/exchange-extended-protection.md
- Обновлено: entities/mail-ca-ibm-org.md (DAG, EPA, перекрёстные ссылки), entities/dag01-recovery-2026-05-18.md (связанные сущности)
- Обновлено: index.md (10 страниц, убраны удалённые, добавлены dag01-recovery, exchange-iis-headers)
- Итого: 7 страниц → 4
- Commit: f03673f, push → origin/main

## [2026-05-19] update | PKI: настройка AIA/CDP/OCSP на Root-CA и Issue-CA
- Root-CA (192.168.2.33): установлен IIS, вирт. директории /aia и /crl → CertEnroll
- Root-CA: URL CDP/AIA заменены с pki.ca-ibm.org на certsrv.ca-ibm.org
- Root-CA: CRL опубликован, HTTP 200 на /crl/Root-CA.crl и /aia/Root-CA.crt
- Issue-CA: перевыпущен сертификат (RequestId: 4, новый thumbprint 7ED6..., до 19.05.2027)
- Issue-CA: вирт. директории /aia и /crl → CertEnroll (IIS уже был)
- Issue-CA: URL публикации оставлены на pki.ca-ibm.org + OCSP
- Issue-CA: настроен OCSP-ответчик (ocsp.msc → issue-ca-ocsp), POST 200
- WinRM на Root-CA: исправлен LocalAccountTokenFilterPolicy (reg add)
|- Оба CA: CRL автообновление 7 дней + delta 1 день
|- Создана страница: [[pki-ca-ibm-org]]
|
|## [2026-06-10] create | Инвентаризация 188.124.56.126 (Amnezia VPN)
|- Создан пользователь cai + sudo, настроен sshpass
|- Проведена полная инвентаризация: Ubuntu 24.04 LTS, 1 vCPU, 1GB RAM, 10GB disk
|- Docker 29.1.3, контейнеры: amnezia-xray (443), amnezia-awg2 (30558/udp)
|- Выполнена очистка места: journalctl → 200M, удалены старые syslog.gz, apt clean
|- Освобождено ~1.3 GB (74% → 58%)
|- Создана страница: [[серверы/188-124-56-126]]
- [[onlyoffice-docx-readonly]] — OnlyOffice word-редактор не активируется, таблицы работают
## [2026-06-10] update | stargate.ca-ibm.org — полная инвентаризация
- Разрезвлён DNS-запрос: IP 94.130.51.147
- Подключение по SSH (Superp@ss2020root)
- Обнаружен Apache2 (а не nginx), MariaDB 10.11.13, PHP 8.3.6, Redis, APCu, Let's Encrypt
- Приложения: collectives, contacts, external, groupfolders, onlyoffice, recognize
- SMTP: порт 2526, режим smtp, relay через smtp-relay-ca-ibm
- Бэкап: /root/nextcloud-backup-20260508/ (config.php + db.sql.gz + onlyoffice-app)
- Фоновые задачи: cron
- Режим логирования: DEBUG
- Обновлена страница: [[stargate-ca-ibm-org]]

## [2026-06-13] lint | Приведение вики в соответствие с паттерном Karpathy
- Созданы недостающие директории: comparisons/, queries/, raw/articles/, raw/papers/, raw/transcripts/, raw/assets/
- Добавлен frontmatter 4 страницам без него: ibt-files-nfs, ocerp-ca-ibm-org, Baku-Metro-Phase-I, Baku-Metro-Phase-II
- Исправлен нестандартный frontmatter: aiib-baku-metro (были произвольные поля вместо title/tags/sources)
- Дополнен frontmatter (+sources, +confidence) на 12 страницах
- Исправлены номера строк (read_file artefact): supabase-192-168-2-34, Парсер тендеров UNGM AIIB, raw/memory/agent-knowledge
- Добавлен sha256 в frontmatter: raw/memory/agent-knowledge-2026-05-15.md (для детекции дрейфа при re-ingest)
- Исправлены 4 битых [[wikilinks]]: Baku-metro-Faza-1→Phase-I, sql/cait-inv-w→cai-it-inv-w, escaped bracket в monitoring/index
- Добавлены недостающие >2 outbound links на 22 страницах
- Добавлены входящие ссылки (reduced orphans): 11 обратных ссылок от основных сущностей к дочерним
- Добавлен frontmatter вспомогательным страницам: серверы/, сертификаты/, sql/, СДЭК, Навыки Hermes, monitoring/
- Обновлён index.md: 14→39 страниц, все секции
- Всего затронуто: ~38 страниц
