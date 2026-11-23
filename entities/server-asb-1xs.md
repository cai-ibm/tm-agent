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
| Версия | **iLO 4 v2.82** — прошит 23.11.2026 (было 2.80; пакет CP053894, `ilo4_282.bin`) |
| Лицензия | iLO Advanced, Perpetual (ключ в iLO, здесь не приводится) |
| Имя | `ILO-ASB-1XS` / `ILO-ASB-1XS.ca-ibm.org` (с 23.11.2026; подчёркивание iLO 4 отвергает) |
| Сервисы | HTTP 80, HTTPS 443, SSH 22, SNMP 161 (trap 162), Virtual Media 17988, Remote Console 17990 |
| Отключено | IPMI 623, Remote Syslog, AlertMail (SMTP не задан) |
| XML API | `XMLResponseEnabled = true`, AES/3DES — Enforce |
| Время | NTP `192.168.40.2` + `192.168.30.9`, TZ Asia/Ashgabat (+05), `PropagateTimeToHost = Yes` (с 23.11.2026) |

Часы iLO были сброшены в epoch — после настройки NTP время корректно; сам сброс — признак умершей
батареи CMOS/RTC на системной плате (iLO берёт время у хоста).

## Накопители

- Контроллер: **HP Smart Array P840** в PCIe-слоте 3 (определён 23.11.2026 через System Utilities → System Configuration;
  iLO 4 модель не отдаёт, видно только как «Slot 3 Drive Array»).
- Диски: **2× 1 TB SATA SSD** — Port 2I, Box 4, Bay 1 и Bay 2 (оба были Unassigned).
- **23.11.2026 создан Logical Drive 1 = RAID 1**, 953.84 GiB (1024.18 GB), 100 % ёмкости массива,
  «HP SSD Smart Path is enabled». Создано через SSA (iLO Remote Console → F9 → System Configuration →
  Slot 3: Smart Array P840 → Launch HPSSA).
- До этого: `1785-Slot 3 Drive Array Not Configured` на каждом POST (01.04.2025, 04.11.2026, 23.11.2026 10:21 и 11:00);
  один раз было `No Drives Detected` — тогда диски ещё не были установлены.
- `Smart Storage Battery pre-failure` (апрель 2020) — батарея кэша контроллера; сейчас в POST пункт
  «Smart Storage Battery» отображается, состояние требует отдельной проверки (SmartCache для SSD не критичен).

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

## Порядок заселения DIMM (HPE DL380 Gen9 User Guide)

Слоты нумеруются 1–12 на каждый процессор. **Буквы = порядок заполнения**, цифры = номер слота:

| Канал | Порядок | Номера слотов |
|---|---|---|
| 1 | **A** E I | 12 11 10 |
| 2 | **B** F J | 9 8 7 |
| 3 | **C** G K | 1 2 3 |
| 4 | **D** H L | 4 5 6 |

Белые слоты (первые в канале) — **12, 9, 1, 4** = A, B, C, D. Заполнять строго A → B → C → D …
При двух процессорах память балансируется между ними (по одинаковому набору на каждый).

- 2 модуля на процессор (4 всего): **слоты 12 и 9** (каналы 1 и 2) — сейчас модули стоят только в слоте 9,
  отсюда ошибка 228.
- 4 модуля на процессор (8 всего): слоты **12, 9, 1, 4** — все четыре канала, полная пропускная способность.
- Не смешивать RDIMM и LRDIMM, ставить модули одного типа/ранга/объёма на оба процессора.

## План подготовки (23.11.2026, до сборки)

Целевые версии: **iLO 4 v2.82** (финальная, пакет CP053894 → `ilo4_282.bin`, 12.02.2023) и
**BIOS P89 3.40 от 29.08.2024** (SHA-256 `907f09f86840c8524c140a5c4bf66c14987a697993a610cd64dba5e3f105e050`).
BIOS на Gen9 не прошивается через iLO (нет UpdateService в Redfish) — только SPP/ISO через virtual media
или из установленной ОС; iLO прошивается из веб-интерфейса.

Удалённо, до сборки:
1. iLO: hostname → `ILO_ASB-1XS` (по образцу `ILO_ASB-07D` на 192.168.40.102), домен ca-ibm.org,
   NTP → 192.168.40.2 / 192.168.30.9, часовой пояс UTC+5, флаг «передавать время хосту».
2. Прошить iLO 2.80 → 2.82 (хост выключен — безопасно).
3. DNS: `A ILO_ASB-1XS → 192.168.40.107`; отдельно решить имя/IP для ОС (сейчас iLO занял имя `asb-1xs`).
4. BIOS через Redfish `PATCH /redfish/v1/Systems/1/Bios/Settings` (применится при следующем старте):
   Hyper-Threading = Enabled, PostF1Prompt = Disabled (чтобы POST не вставал на ошибках памяти),
   ServerName/ServerPrimaryOs, режим загрузки (UEFI — если ОС это позволяет), TZ UTC+5.
5. PRTG: завести iLO (SNMP 161 / IPMI), настроить AlertMail или SNMP-trap на 192.168.40.29.

При сборке: проверить БП и кабели, заменить батарею CMOS, поставить память по таблице выше,
убедиться в наличии корзин/дисков, проверить батарею кэша Smart Array.

## Выполнено 23.11.2026 (удалённо, через iLO)

- **iLO 2.80 → 2.82**: файл `ilo4_282.bin` (CP053894) раздан по HTTP и залит через SSH-CLI
  `load /map1/firmware1 -source http://…/ilo4_282.bin`. Раздатчик обязан отвечать `206 Partial Content`
  на Range-запросы — простой `python -m http.server` iLO отвергает («Unable to retrieve a valid image»).
- **Имя**: `ILO-ASB-1XS.ca-ibm.org` (подчёркивание, как у `ILO_ASB-07D`, iLO 4 не принимает:
  «DNS_NAME must contain only letters, numbers and non-hanging hyphens»).
- **NTP/время**: `192.168.40.2`, `192.168.30.9`; TZ `Asia/Ashgabat`; `PropagateTimeToHost = Yes`;
  после синхронизации время верное (сверено с локальным UTC).
- **BIOS (pending, применится при первом старте)**: `ProcHyperthreading = Enabled`,
  `PostF1Prompt = Disabled`, `ServerName = ASB-1XS`, `TimeZone = UtcP5`.
  `BootMode` оставлен `LegacyBios` — меняется после решения по ОС.
- **RAID 1 создан** на Smart Array P840 (2× 1 TB SATA SSD → Logical Drive 1, 953.84 GiB) через
  Remote Console: F9 → System Configuration → Slot 3 → Launch HPSSA → Configure → Create Array → RAID 1.
  После перезагрузки POST проходит без ошибки 1785.
- **Network Boot отключён** (`NicBoot1..5 = Disabled`, `NetworkBootRetry = Disabled`) — машина больше не
  висит в PXE-цикле, пробует HDD и пишет `No bootable devices detected` (ОС ещё нет).
- **Сеть:** FLR-карта воткнута в CRS354 `SW.B2.Mod3.Server.CRS354.HG909V1GC61` (mgmt 192.168.100.6),
  порт **qsfpplus2-1**: 40G, DAC Molex 1110402304 3 м, untagged PVID 1 — тот же VLAN 1, что у iLO (ether16).
  На свиче выучен MAC **48:0F:CF:F2:2A:E2** — это второй порт карты, а PXE ходил с первого (`…:E1`),
  откуда и «Link:down». Побочный эффект: каждый retry FlexBoot сбрасывал PHY, и линк 40G флапал
  каждые ~30 с (`link-downs` до 58). После отключения Network Boot флап прекратился — линк стабилен.

Осталось (в работе): DNS-запись iLO/хоста, смена пароля и отключение HTTP, PRTG/AlertMail,
BIOS P89 3.40 через SPP, замена батареи CMOS, память по таблице DIMM.

## Что сделать перед вводом в эксплуатацию
1. **Питание:** проверить кабели/PDU обоих БП (iLO видит оба как Absent, в журнале Input Power Loss 04.11.2026,
   `1624` по БП 1). Без этого сервер не поднимется под нагрузкой.
2. **Батарея CMOS/RTC** — заменить (часы iLO в 1970, BIOS TimeZone = Utc0).
3. **Память:** поставить модули в слоты **12 и 9** каждого процессора (сейчас занят только 9 — отсюда 228);
   проверить, что модули HPE SmartMemory — иначе останутся предупреждения аутентификации.
4. **Диски:** установить корзины/накопители и создать массив на Smart Array в слоте 3 (ошибка 1785 — томов нет),
   оценить состояние батареи кэша контроллера.
5. **Прошивки:** BIOS P89 2.30 (2016) и iLO 4 2.80 устарели — обновить (в iLO Advanced SPP/онлайн-каталог доступны),
   затем выставить NTP и корректное время.
6. **BIOS под сервер:** при необходимости — UEFI + Secure Boot вместо Legacy, включить Hyper-Threading
   (сейчас выключен — теряется половина потоков), TZ UTC+5.
7. **DNS:** добавить `A`-запись `asb-1xs` → 192.168.40.107 в зоне ca-ibm.org (сейчас NXDOMAIN), либо включить DDNS.
8. **iLO-безопасность:** сменить пароль Administrator, отключить HTTP (оставить HTTPS), решить по SNMP (161 открыт),
   настроить AlertMail/Remote Syslog на PRTG/почту.
