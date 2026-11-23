---
title: ASB-1XS — HPE ProLiant DL380 Gen9 (192.168.40.107)
created: 2026-11-23
updated: 2026-11-23
type: entity
tags: [server, hardware, hpe, ilo, inventory, 40-net]
sources: [ilo-redfish-api, ilo-iml]
confidence: high
---

# ASB-1XS — HPE ProLiant DL380 Gen9 (iLO 192.168.40.107)

Новый сервер на площадке ASB, инвентаризация снята через iLO 4 (Redfish `/redfish/v1`, XML `/xmldata?item=All`,
журнал IML). На момент съёмки (23.11.2026) машина **выключена**, ОС не развёрнута (в BIOS `ServerName`
осталось `MININT-9BGQPLN` от установки Windows), Smart Array не сконфигурирован. Связанные узлы:
[[entities/server-asb-0cj]] (виртуализация), [[entities/server-asb-dc2]] (домен ca-ibm.org).

## Платформа

| Параметр | Значение |
|---|---|
| Модель | HPE ProLiant DL380 Gen9, 2U rack |
| Product / SKU | 823843-B21 |
| Серийный номер | CZ360821XS |
| UUID (System) | 38333238-3334-5A43-3336-303832315853 |
| Состояние | Power **Off**, Health OK (`Status.State = Disabled`) |
| Потребление / ёмкость | 0 Вт из 2800 Вт (замер при выключенном) |
| Блоки питания | 2 слота, модель 720620-B21 (Flex Slot); iLO показывает оба как **Absent / Health Warning**, 0 В на входе |
| iLO серийник / UUID | ILOCZ360821XS / 7ceeb08c-5717-5303-b5c6-7db4833a197e |

## Процессоры и память

| Параметр | Значение |
|---|---|
| CPU | 2× Intel Xeon E5-2643 v4 (6 ядер / 12 потоков, 3.4 ГГц, турбо до 3.7, TDP 135 Вт) |
| Hyper-Threading | **Disabled** в BIOS |
| RAM | 32 GiB: 2× 16 GB DDR4-2133 RDIMM (2Rx4, ECC multi-bit), слоты `PROC 1 DIMM 9` и `PROC 2 DIMM 9` |
| Прочее | AMP (Advanced Memory Protection) — `None`, режим не настроен |

Проблемы из POST 04.11.2026:
- `228-DIMM Configuration Error` — Processor 1 **и** Processor 2, Channel 2, DIMM 8: канал заполнен не по порядку
  (DIMM 8 пуст, а DIMM 9 занят). Модули стоят в девятых слотах, из-за чего теряется 4-канальная производительность.
- `Processor 1/2, DIMM 9 could not be authenticated as genuine HPE Memory` — модули не опознаны как HPE SmartMemory
  (неоригинальные либо с битым SPD), расширенные функции SmartMemory отключены.

## Сеть

| Интерфейс | MAC | Примечание |
|---|---|---|
| HP Ethernet 1Gb 4-port 331i (embedded LOM) | 14:02:ec:3d:ac:9c … 9f | 4 порта 1 GbE, `EmbNicEnable = Enabled` |
| HP InfiniBand FDR/Ethernet 40Gb 2-port 544+FLR-QSFP | 48:0f:cf:f2:2a:e1, …e2 | FlexLOM 40 Гбит/с (IB FDR или 40GbE), прошивка не проверялась |
| iLO 4 dedicated, порт 1 | 14:02:ec:39:40:0c | 192.168.40.107/24, static |
| iLO 4, порт 2 | 14:02:ec:39:40:0d | Disabled |

iLO: GW 192.168.40.254, DNS 192.168.40.28 / 40.66.162.225 / 40.66.162.223, домен `ca-ibm.org`, DDNS выключен.
**Записи `asb-1xs` / `asb-1xs.ca-ibm.org` в зоне ca-ibm.org нет** (NXDOMAIN с 192.168.40.2) — резолва нет ни у iLO-имени, ни у будущего хоста.

## iLO 4

| Параметр | Значение |
|---|---|
| Версия | iLO 4 v2.80 (26.09.2023 прошивалась 2.82, 02.10.2023 откат на 2.80) |
| Лицензия | iLO Advanced, Perpetual (ключ в iLO, здесь не приводится) |
| Сервисы | HTTP 80, HTTPS 443, SSH 22, SNMP 161 (trap 162), Virtual Media 17988, Remote Console 17990 |
| Отключено | IPMI 623, Remote Syslog, AlertMail (SMTP не задан) |
| XML API | `XMLResponseEnabled = true`, AES/3DES — Enforce |
| Время | `1970-01-01T00:08Z`, TZ Atlantic/Reykjavik (GMT+0), NTP-серверы пусты, `PropagateTimeToHost = false` |

Часы iLO сброшены в epoch — косвенный признак умершей батареи CMOS/RTC на системной плате (iLO берёт время у хоста).

## Накопители

- Контроллер **Smart Array в PCIe-слоте 3** (`PciSlot3Enable = Enabled`); точную модель (P440/P840) iLO 4 не отдаёт —
  видно только как «Slot 3 Drive Array».
- `1785-Slot 3 Drive Array Not Configured` — логические тома не созданы. Записи в IML: 01.04.2025, 04.11.2026 10:06,
  **23.11.2026 10:21 и 11:00** (два последних POST — сегодня).
- В журнале есть `No Drives Detected. Action: Reseat or add drives` — как минимум на одном включении контроллер
  не увидел ни одного диска (корзины/бэкплейн/кабели).
- `Smart Storage Battery pre-failure` (апрель 2020) — батарея кэша контроллера, скорее всего, требует замены.

## BIOS

| Параметр | Значение |
|---|---|
| Версия | **P89 v2.30 от 13.09.2016** (последняя правка настроек — 04.11.2026 10:06 UTC) |
| Режим загрузки | **Legacy BIOS** (UEFI Optimized Boot — Disabled) |
| Secure Boot | Disabled, TPM 1.2 присутствует и выключен |
| Питание | Power Regulator = Static High Performance, Power Profile = Custom, Thermal = Optimal Cooling |
| Автозапуск | `AutoPowerOn = RestoreLastState`, `PostF1Prompt = Delayed20Sec` |
| Часовой пояс BIOS | Utc0 (для площадки нужно UTC+5) |
| Прочее | Intelligent Provisioning включён, Boot order NIC 1–5 = NetworkBoot, `ServerName = MININT-9BGQPLN`, `ServerPrimaryOs` пуст |

## Журнал IML (225 записей с 2018 г., ключевое)

- 17.11.2018 — `PCI Bus Error, Slot 3` + `1719` сбой контроллера массива (lock up 0x13).
- 14.01.2020 — `Drive Array Controller Failure (Slot 3)`; 05–06.04.2020 — pre-failure батареи кэша,
  `1793` потеря данных в write-back кэше, `1797` кэширование отключено.
- 15.06.2022 — повторный `PCI Bus Error, Slot 3` (тот же слот с контроллером).
- 2023 — серия прошивок BIOS/ILO (2.61 → 2.70 → 2.82 → 2.80).
- 01.04.2025 — массив не сконфигурирован, диски не обнаружены (машина, похоже, стоит без дисковой части).
- 04.11.2026 — включение, настройка BIOS (Windows Setup, `MININT-9BGQPLN`), ошибки DIMM 228 на обоих процессорах,
  неаутентифицированная память, `Input Power Loss` по **обоим** БП, `1624` (БП 1 отключён).
- 23.11.2026 10:21 и 11:00 — два POST, снова `1785-Slot 3 Drive Array Not Configured`; после этого машина выключена.
- После потери питания часы iLO ушли в 1970 — запись `1970-01-01 ... Server Critical Fault (Input Power Loss, PS1, PS2)`.

## Что сделать перед вводом в эксплуатацию

1. **Питание:** проверить кабели/PDU обоих БП (iLO видит оба как Absent, в журнале Input Power Loss 04.11.2026,
   `1624` по БП 1). Без этого сервер не поднимется под нагрузкой.
2. **Батарея CMOS/RTC** — заменить (часы iLO в 1970, BIOS TimeZone = Utc0).
3. **Память:** переставить модули по порядку заполнения (DIMM 1/2 вместо DIMM 9, либо заполнить канал 2 целиком),
   проверить, что модули HPE SmartMemory — иначе убрать ошибки 228 и предупреждения аутентификации.
4. **Диски:** установить корзины/накопители и создать массив на Smart Array в слоте 3 (ошибка 1785 — томов нет),
   оценить состояние батареи кэша контроллера.
5. **Прошивки:** BIOS P89 2.30 (2016) и iLO 4 2.80 устарели — обновить (в iLO Advanced SPP/онлайн-каталог доступны),
   затем выставить NTP и корректное время.
6. **BIOS под сервер:** при необходимости — UEFI + Secure Boot вместо Legacy, включить Hyper-Threading
   (сейчас выключен — теряется половина потоков), TZ UTC+5.
7. **DNS:** добавить `A`-запись `asb-1xs` → 192.168.40.107 в зоне ca-ibm.org (сейчас NXDOMAIN), либо включить DDNS.
8. **iLO-безопасность:** сменить пароль Administrator, отключить HTTP (оставить HTTPS), решить по SNMP (161 открыт),
   настроить AlertMail/Remote Syslog на PRTG/почту.
