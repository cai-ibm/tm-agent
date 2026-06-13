---
title: Восстановление DAG01 после потери кворума
created: 2026-05-18
updated: 2026-05-18
type: entity
tags: [exchange, dag, cluster, incident, recovery]
sources: []
confidence: medium
---

# Восстановление DAG01 — 18.05.2026

## Инцидент

**Время:** 18.05.2026 08:43
**Суть:** Кластер DAG01 потерял кворум. Core Cluster Group (имя/IP кластера) — Offline.

## Диагностика

- Кластер: DAG01, 2 узла (mail-srv1 192.168.2.50, mail-srv2 192.168.40.38)
- Quorum: Node Majority
- Связность между узлами есть (пинг проходит)
- Event 1653: mail-srv1 failed to join cluster — не мог общаться с другими узлами
- Event 5398: Cluster failed to start — конфигурация кластера недоступна
- mail-srv1: DynamicWeight 0 (исключён из голосования)
- mail-srv2: DynamicWeight 1 (единственный голос)

## Восстановление

1. Попытка перезапуска ClusSvc на mail-srv1 — не помогла (конфиг устарел)
2. **Force quorum на mail-srv2:**
   ```
   Stop-Service ClusSvc -Force
   net start ClusSvc /forcequorum
   ```
3. Поднятие Core Cluster Group:
   ```
   Start-ClusterGroup -Name "????????? ?????????"
   ```
4. Результат: Core Group Online, оба узла Up в кластере

## Текущее состояние

- Core Cluster Group: Online (mail-srv2)
- Available Storage: Online (mail-srv2)
- mail-srv1: Up, NodeWeight=1, DynamicWeight=0 (восстановится автоматически)
- mail-srv2: Up, DynamicWeight=1

## Рекомендации

- Через несколько часов проверить DynamicWeight на mail-srv1 (должен стать 1)
- Рассмотреть добавление File Share Witness для 2-узлового DAG (предотвращает split-brain)
- Убедиться что heartbeat-сеть (192.168.40.0/24) стабильна между узлами

## Связанные сущности

- [[mail-ca-ibm-org]] — основной профиль Exchange
- [[exchange-iis-headers]] — управление HTTP-заголовками
