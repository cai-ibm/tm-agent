---
title: Поток с камеры Uniview через облако EZCloud — почему сервер не может его взять
created: 2026-10-23
updated: 2026-10-23
type: entity
tags: [server, service, network, camera, streaming]
sources: []
confidence: high
---

# Поток с камеры Uniview через облако EZCloud

**Задача:** сервер `asb-timelapse` ([[entities/server-asb-timelapse]]) должен принимать видеопоток с камеры
Uniview `IPC2122LB-AF28-DL-ECO`, когда камеру перенесут туда, где прямого доступа к ней не будет.
Единственный канал — облако `ulygayasec.com.tm` (EZCloud, белый лейбл Uniview).

**Статус на 2026-10-23: закрыто как технически невозможное на стороне сервера.** Облако не отдаёт
поток серверу ни по одной ручке; единственный работающий канал — клиентский P2P (мобильное
приложение, Windows-плагин), который серверу недоступен. Записи и облачного хранилища у аккаунта нет.

---

## 1. Что в аккаунте

Аккаунт `alexromanov4423@gmail.com`, площадка `ulygayasec.com.tm` (95.85.125.154), клиент `A240.1.0`.

| Устройство | SN | LAN | Состояние |
|---|---|---|---|
| D14 | `210235UNG63265000087` | 192.168.0.14 | online, supportLive:0 |
| D15 | `210235UNG63265000329` | 192.168.0.15 | online, supportLive:0 |

Обе камеры свои (`isShared:0`), организация `root` (organizationId 55708), `publicIp` 95.47.61.105.

## 2. Как облако отдаёт поток: только P2P-клиенту

- Приватная ручка `POST /private/web/device/access/url/get` отвечает 200 и выдаёт **не URL потока**, а
  пропуск локальному P2P-плагину:
  `http://localhost:9999/?t=…&u=KkRyrp&ds=210235UNG6326500008788263faa49&si=95.85.125.157&sc=2&pt=…&sk=…`
  где `si` — адрес релея, `pt`/`sk` — токены сессии.
- Веб-клиент сам опрашивает локальную службу: `http://localhost:9999/getP2pVersion`, а плагин
  `EZCloud_P2P` (последняя версия на площадке 1.21.0006) — Windows-only.
- Релей `95.85.125.157` доступен с обоих хостов по TCP 30002; порты 80/443/4000/8000/8080 закрыты.
  Протокол проприетарный, ffmpeg/MediaMTX его не понимают.

Именно поэтому в мобильном приложении видео есть: приложение — нативный P2P-клиент, а не HTTP-клиент.

## 3. Результаты проверки ручек (2026-10-23, с живым токеном)

| Ручка | Ответ |
|---|---|
| `/openapi/live/video/device/url/get`, `/get`, `/start`, `/url/refresh` | **503** (requestId 1f74d268, 63b19fc8, 93aa69e, cbf64ab) |
| `/openapi/live/trial/get`, `/openapi/cdn/activation/strategy/get`, `/openapi/cdn/device/list` | **503** (7abca8d7, 765350c6) |
| `/openapi/live/status/get` (путь выдуман) | **503** — весь префикс `/openapi/live/**` и `/openapi/cdn/**` уходит в мёртвый сервис |
| `/openapi/device/capacity/get` | 200, `supportLive:0`, `supportCdn:1`, `supportCos:1` |
| `/openapi/device/p2p/url/get` | 200, `1000 Server internal error` |
| `/openapi/device/media/url/get`, `/media/stream/get` | 200, `1006 Function not supported` |
| `/openapi/device/capture/get` | 200, `1009 Resource not found` |
| `/openapi/device/record/query` | **404** — ручки на сервере нет (клиент её всё ещё зовёт) |
| `/openapi/device/record/time/query` | 200, `recordList: []` — записей нет вообще |
| `/openapi/device/media/record/hls/get` | 200, `m3u8:null`, `expire:null` — отдавать нечего |
| `/openapi/cloud/storage/device/get` | 200, `configList: []` — облачное хранилище не подключено |

Без токена все `/openapi/*` отвечают 200 + `1001 Token authentication failed`, то есть авторизация
проверяется раньше маршрутизации; 503 виден только с сессией. `GET` на live-путях отдаёт 503 даже
анонимно — маршрут битый на уровне шлюза.

## 4. Попытка обойти через Android-клиент (2026-10-23)

Идея: раз приложение работает через P2P, поднять Android на сервере, залогиниться и снимать экран.

Инфраструктура: KVM нет ни на одном хосте (вложенная виртуализация выключена), поэтому только
контейнерный Android.

- **asb-timelapse (Debian 12, ядро 6.1.140):** `# CONFIG_ANDROID_BINDERFS is not set`, в актуальных
  конфигах Debian binder вообще выключен → Waydroid/redroid не запускаются. Отказ на уровне ядра.
- **tm-host4 (Ubuntu 22.04, 5.15):** `CONFIG_ANDROID_BINDERFS=m` — Android поднимается.
  - Waydroid 1.6.2 + LineageOS 20 x86_64: контейнер стартует, но SurfaceFlinger падает с
    `Abort message: 'output buffer not gpu writeable'` (RenderEngine/Skia GL, gralloc gbm на vmwgfx);
    в образе нет ARM-библиотек (`system/lib/arm` пуст) → ARM-only APK не запустился бы.
  - redroid 11 в Docker (gpu_mode=guest): Android 11 загружается, ARM-трансляция есть
    (`abilist: x86_64,x86,arm64-v8a,armeabi-v7a,armeabi`, libndk_translation.so), APK ставится,
    приложение EZLive CIS доходит до экрана логина.
  - **Вердикт:** arm64-сборка падает `SIGILL` в `libndk_translation.so`
    (`DecodeSimdScalarTwoRegMisc` → `DecodeDataProcessingSimdAndFp`, поток `2.raster`);
    32-битная сборка (`adb install --abi armeabi-v7a`) вешает UI — `ANR: Input dispatching timed out`,
    диалог согласия не реагирует. Ни одна ABI не даёт рабочего состояния — предел трансляции для
    Flutter + нативных медиабиблиотек (libNDPlayer, libMP2).

## 5. Что остаётся рабочим вариантом

1. **Тикет провайдеру** `ulygayasec`: восстановить сервис live/CDN, включить live устройствам,
   сверить версии API, выдать OpenAPI-ключи (`appKey`/`appSecret`). Только так сервер получит HLS/FLV-URL.
2. **Windows-клиент** (EZCloud PC / EZStation) на Windows-ВМ: нативное x86-приложение, P2P работает;
   забирать либо окно (`ffmpeg gdigrab`), либо — предпочтительно — выяснить, что P2P-плагин отдаёт
   на `127.0.0.1:9999`, и тянуть это напрямую в MediaMTX.
3. **Пуш из новой сети** (мини-ПК/NVR в LAN камеры → RTSP локально → SRT на asb-timelapse).
   Требует согласования: доступа к роутеру на новом месте не будет.

## 6. Ловушки

1. `503` на `/openapi/live/**` — не «камере не включили live», а мёртвый сервис на площадке;
   проверяется выдуманным путём под тем же префиксом.
2. `supportLive: 0` при `supportCdn: 1` — состояние учётки, а не сети; камера при этом online.
3. `localhost:9999` в ответах облака — это адрес P2P-плагина на машине клиента, а не сервера.
4. Android не поможет, если в образе нет ARM-трансляции: проверять `getprop ro.product.cpu.abilist`
   и наличие `libndk_translation.so` / libhoudini **до** установки APK.
5. `screencap` отдаёт чёрный кадр, если дисплей уснул — сначала `svc power stayon true`.
6. ARM-приложения под libndk_translation падают не всегда явно: `SIGILL` в 64-битной сборке,
   тихий `ANR` в 32-битной.

## Связи

- [[entities/server-asb-timelapse]] — сервер приёма (MediaMTX, архив, таймлапс)
- [[concepts/network-subnets]] — адресация площадок
