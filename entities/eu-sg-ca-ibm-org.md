---
title: eu-sg.ca-ibm.org (SecurityGateway)
created: 2026-05-15
updated: 2026-05-15
type: entity
tags: [server, security, exchange, service, spam]
sources: []
confidence: high
---

# eu-sg.ca-ibm.org — ALT-N SecurityGateway

Почтовый шлюз безопасности (spam/virus filter) перед [[mail-ca-ibm-org]] (Exchange). Фильтрует входящую и исходящую почту.

## Основные параметры

| Параметр | Значение |
|----------|----------|
| **Домен** | eu-sg.ca-ibm.org |
| **IP (внутренний)** | 192.168.2.36 |
| **ПО** | ALT-N SecurityGateway 11.0.3 |
| **Веб-интерфейс** | Порт 4443 (HTTPS) |

## Проксирование через NPM

Проксируется через [[nginx-pm-192-168-2-31]]:

| Параметр | Значение |
|----------|----------|
| **Proxy Host ID** | 4 |
| **Домен** | eu-sg.ca-ibm.org |
| **Forward** | https://192.168.2.36:4443 |
| **SSL** | ✅ Let's Encrypt |
| **HSTS** | Включён |

## Связанные сущности

- [[nginx-pm-192-168-2-31]] — Nginx Proxy Manager
- [[mail-ca-ibm-org]] — Exchange, защищаемый шлюз

## История изменений

- **2026-05-15** — Добавлен в NPM (id=4), Let's Encrypt SSL, HTTPS работает
