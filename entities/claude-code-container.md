---
title: Claude Code Container
created: 2026-05-19
updated: 2026-06-30
type: entity
tags: [docker, claude-code, container, ai-agent]
sources: []
confidence: high
---

# Claude Code Container

Отдельный Docker-контейнер с Claude Code CLI (Anthropic), запущенный на хосте `hermes-eu` (192.168.2.34 / внешний 94.130.51.188) рядом с Hermes Agent.

## Основные параметры

| Параметр | Значение |
|----------|---------|
| Имя контейнера | `claude-code` |
| Образ | `claude-code:latest` |
| Claude Code | v2.1.144 |
| Node.js | v20.20.2 |
| npm | 11.14.1 |
| Базовый образ | Ubuntu 24.04 |
| Рабочая директория | `/workspace` |
| Restart policy | `unless-stopped` |
| Dockerfile | `/workspace/claude-code-docker/Dockerfile` (в контейнере Hermes) |

## Доступ

SSH через ключ `id_ed25519`, пользователь `cai@192.168.2.34`. Пароль `sudo` не настроен — команды от пользователя `cai` (в группе `docker`).

## FCC + Ollama Cloud (с 30.06.2026)

На хосте установлен **free-claude-code v2.4.1** — прокси для подключения Claude Code CLI к **Ollama Cloud API**.

**FCC-сервер** запущен на порту 8082 с моделью `ollama/kimi-k2.7-code`. API-ключ Ollama Cloud передаётся через `OLLAMA_API_KEY`.

**Контейнер `claude-code`** пересоздан с переменными:
- `ANTHROPIC_BASE_URL=http://172.17.0.1:8082` (FCC-прокси)
- `ANTHROPIC_API_KEY=fcc-no-auth`

**Патчи FCC** (применены вручную, сбрасываются при переустановке):
- `providers/ollama/client.py` — переписан на `OpenAIChatTransport`
- `config/provider_catalog.py` — ollama: `openai_chat` transport, `credential_attr`
- `config/settings.py` — добавлен `ollama_api_key`, удалён валидатор URL

Подробнее: [[fcc-ollama-cloud]] (навык Hermes)

## Команды управления

```bash
# Интерактивный режим (TUI)
docker exec -it claude-code claude

# Print mode (одноразовая задача)
docker exec -e ANTHROPIC_API_KEY=sk-ant-... claude-code claude -p "задача"

# Проверка версии
docker exec claude-code claude --version

# Логи
docker logs claude-code

# Перезапуск
docker restart claude-code
```

## Аутентификация

Требуется `ANTHROPIC_API_KEY` (передаётся через `-e` при `docker exec`) или OAuth (требует браузер, в headless-контейнере неудобно). Ключ пока не настроен — добавить при необходимости.

## Volume mount (пока не настроен)

Для доступа Claude Code к коду нужно пересоздать контейнер:
```bash
docker stop claude-code && docker rm claude-code
docker run -d --name claude-code --restart unless-stopped \
  -v /путь/к/проекту:/workspace \
  -e ANTHROPIC_API_KEY=sk-ant-... \
  claude-code:latest
```

## Dockerfile

```dockerfile
FROM ubuntu:24.04
ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install -y --no-install-recommends \
    curl ca-certificates git && rm -rf /var/lib/apt/lists/*
RUN curl -fsSL https://deb.nodesource.com/setup_20.x | bash - \
    && apt-get install -y --no-install-recommends nodejs \
    && npm install -g npm@latest && rm -rf /var/lib/apt/lists/*
RUN npm install -g @anthropic-ai/claude-code
WORKDIR /workspace
CMD ["sleep", "infinity"]
```

## Связанные страницы

[[mail-ca-ibm-org]], [[server-94-130-51-161]]
