---
name: padavan-vpn-config-monitor
description: Мониторинг изменений VPN-конфига на Padavan по SSH.
---

# Мониторинг конфига OpenVPN на Padavan (NK-VPN)

Когда: нужно отслеживать ИЗМЕНЕНИЯ конфигурации VPN-клиента на роутере с Padavan (Asuswrt-Mod),
а не сетевые события (активный endpoint, поднятие туннеля).

## Доступ (роутер NK, 192.168.8.1)

- SSH открыт (dropbear, порт 22), логин `admin`, пароль — тот же, что в веб-панели Padavan.
  Пароль хранить в `~/.hermes/scripts/.nk-monitor.env` как `NK_ROUTER_PASS`.
  `sshpass -p "$PW" ssh -o StrictHostKeyChecking=no admin@192.168.8.1 '<cmd>'`.
- HTTP-порт 80 отдаёт Basic realm `MI-R3GV2_SPI` (проверка «это тот самый девайс»), 443 закрыт.
- `nvram` НЕ содержит настроек VPN-клиента — не искать там домен/порт, их там нет.

## Где лежит конфиг и кто его меняет

- `/etc/storage/openvpn/client/client.conf` — ПОСТОЯННЫЙ (flash, `/etc/storage` персистентный).
- `/etc/openvpn/client/client.conf` — РАБОЧИЙ, в tmpfs: генерируется из storage при старте службы.
  openvpn запущен как `--cd /etc/openvpn/client --config client.conf`. /etc — tmpfs → правки рабочего
  файла теряются при перезагрузке, остаётся версия из storage.
- `/etc/storage/ca.sh` + cron роутера `*/10 * * * *` — ПРОВАЙДЕРСКИЙ автообновлятор: качает Google Doc,
  выдёргивает строки `remote IP PORT`, переписывает их в storage-конфиге, делает `client.conf.bak`
  и `mtd_storage.sh save`. Поэтому появление/изменение `.bak` = провайдер обновил список remote.
- `/etc/storage/openvpn/client/{ca.crt,client.crt,client.key,ta.key}` часто являются копиями одного
  и того же CA-плейсхолдера (`CN = ChangeMe`), а реальные ca/cert/key/tls-crypt лежат ИНЛАЙН в client.conf.
  Не удивляться одинаковым хешам этих четырёх файлов.
- busybox на роутере: есть `md5sum`, НЕТ `sha256sum` и `diff` — забирать файлы на хост и хешировать локально.

## Как сравнивать (правильный дизайн)

1. Забрать по одному SSH-вызову: оба client.conf, 4 файла ключей, `.bak`, `ca.sh` + `ps w | grep [o]penvpn`,
   `ip -4 addr show tun0`.
2. Снимок = sha256 каждого файла + разобранные параметры: список `remote` (как множество), `proto`,
   `dev`, `cipher`, `auth`, `scramble`, `redirect-gateway`, `verb`, маршруты, хеши инлайн-блоков
   `<ca>/<cert>/<key>/<tls-crypt>`, subject/notAfter сертификата, файловые ссылки ca/cert/key.
3. Сравнивать с baseline из `~/.hermes/state/nk-vpn-monitor/state.json`, печатать ТОЛЬКО дельту.
4. Пустой stdout = изменений нет (no_agent cron ничего не отправляет). Отчёт — только при изменении.
5. Отдельно ловить расхождение рабочий ≠ storage (по фронту, а не каждый день) и здоровье туннеля
   (процесс openvpn, tun0) — без этого мониторинг слепой.

## Ловушки

- НЕ алертить на смену активного endpoint: у провайдера пул из ~10 IP + домен, ротация внутри списка — норма.
  Тянуть conntrack с MikroTik ради этого не нужно (в первой версии было — давало ложные тревоги).
- Собирать удалённую команду из списка строк через `"\n".join(...)`; `" ".join` для `for ... do ... done`
  даёт `sh: syntax error: unexpected "then"`.
- Флаг состояния (`diverged`) читать из корня state, а не из вложенного snapshot — иначе алерт повторяется вечно.
- Сертификаты разбирать через локальный `openssl x509 -noout -subject -enddate` (stdin PEM).
- State держать в 700, файлы с ключами — 600: в снимках лежат выдержки PEM.

## Проверка перед сдачей

```bash
python3 nk_vpn_config_check.py --init      # записать baseline (1 строка в stdout)
python3 nk_vpn_config_check.py             # пусто = ок
python3 nk_vpn_config_check.py --report    # сводка «изменений нет»
```
Плюс тест детекта: подменить в `state.json` один `remote`/`cipher`/sha файла → отчёт должен перечислить
дельты, повторный запуск — снова пусто.

Готовый скрипт: `~/.hermes/scripts/nk_vpn_config_check.py` (cron-job 4796f332bc5a, ежедневно 20:00 UTC+5).
