---
title: pfSense Captive Portal — настройка гостевой сети
created: 2026-07-24
updated: 2026-07-24
type: concept
tags: [network, firewall, howto, troubleshooting]
confidence: high
---

# pfSense Captive Portal — настройка гостевой сети

## Обзор

Captive Portal на pfSense — встроенный механизм для организации гостевого доступа в сеть. Пользователь подключается к Wi-Fi, открывает браузер, видит страницу авторизации (с кнопкой или логином/паролем), нажимает кнопку — и получает доступ в интернет.

В данной конфигурации используется:
- **VLAN 120** (192.168.120.0/24) — гостевая сеть CAI Free
- **Интерфейс:** LAN (vmx0) на pfSense
- **Аутентификация:** None (кнопка)
- **Pre-auth redirect:** http://google.com
- **Таймауты:** idle 60 мин, hard 480 мин
- **Порт портала:** 8002

---

## Пошаговая настройка

### 1. Базовая конфигурация через веб-интерфейс

**Services → Captive Portal → Add**

| Параметр | Значение |
|---|---|
| Zone name | `cai_free` |
| Interface | LAN |
| Authentication | None |
| Idle timeout | 60 |
| Hard timeout | 480 |
| Pre-auth redirect URL | `http://google.com` |
| Custom HTML page | `portal.html` |

После создания зоны pfSense автоматически:
- Создаёт pf-правила (блокировка неавторизованных, RDR на портал)
- Запускает nginx на порту 8000 + zoneid (по умолчанию 8000, если zoneid=0)
- Запускает minicron для чистки старых сессий

### 2. Кастомная страница портала

Страница загружается через веб-интерфейс (вкладка **Custom HTML**) или через редактирование конфига.

**Механизм работы:**
1. HTML-код кодируется в base64 и сохраняется в `/conf/config.xml` в теге `<page><htmltext>`
2. При `captiveportal_configure_zone()` pfSense декодирует base64, заменяет переменные `$PORTAL_...$` на `#PORTAL_...#` и пишет в `/var/etc/captiveportal_<zone>.html`
3. При запросе клиента `index.php` читает этот файл, подставляет реальные значения (`#PORTAL_ACTION#` → URL портала, `#PORTAL_REDIRURL#` → URL редиректа) и отдаёт клиенту

**Переменные в HTML:**
- `$PORTAL_ACTION$` — URL для отправки формы (POST)
- `$PORTAL_REDIRURL$` — URL для редиректа после авторизации
- `$PORTAL_MESSAGE$` — сообщение об ошибке
- `$CLIENT_MAC$` — MAC-адрес клиента
- `$CLIENT_IP$` — IP-адрес клиента

**Форма должна содержать:**
```html
<form method="post" action="$PORTAL_ACTION$" id="portalForm">
  <input type="hidden" name="accept" value="1">
  <input type="hidden" name="redirurl" value="$PORTAL_REDIRURL$">
  <button type="submit">Подключиться</button>
</form>
```

### 3. Firewall правила (passthru)

До авторизации клиент должен иметь доступ к:
- **DHCP** (порты 67-68) — чтобы получить IP
- **DNS** (порт 53) — чтобы резолвить домены
- **Порталу** (порт 8002) — чтобы открыть страницу

Эти правила должны стоять **перед** блокирующим правилом captive portal.

**Порядок правил в pf (критично!):**
```
1. pass ... port = domain (DNS passthru)     ← ДО блокировки
2. pass ... port = 8000 (доступ к порталу)   ← ДО блокировки
3. block drop ... ! tagged cpzoneid__auth    ← БЛОКИРУЮЩЕЕ ПРАВИЛО
4. anchor "userrules/*"                      ← ПОСЛЕ блокировки
```

### 4. NAT и RDR

NAT для гостевой сети настраивается автоматически при включении captive portal:
```
nat on vmx1 inet from 192.168.120.0/24 to any -> 192.168.40.1 port 1024:65535
```

RDR-правило для перехвата HTTP-трафика:
```
rdr on vmx0 inet proto tcp from any to ! <cpzoneid__cpips> port = http -> 192.168.120.1 port 8002
```

---

## Подводные камни и решения

### 🔴 1. Лишняя обёртка `<zone>` в конфиге

**Проблема:** При редактировании конфига вручную или через PHP может появиться лишняя структура:
```xml
<!-- НЕПРАВИЛЬНО -->
<captiveportal>
    <zone>                    ← ЛИШНЕЕ!
        <cai_free>...</cai_free>
    </zone>
</captiveportal>

<!-- ПРАВИЛЬНО -->
<captiveportal>
    <cai_free>...</cai_free>
</captiveportal>
```

**Симптомы:** `config warning: invalid path "captiveportal/"` в логах, портал не запускается.

**Решение:** Удалить обёртку `<zone>`:
```bash
sed -i '' -e '/<captiveportal>/,/<\/captiveportal>/{s/<zone>//;s/<\/zone>//}' /conf/config.xml
```

### 🔴 2. Порт nginx не совпадает с RDR

**Проблема:** RDR-правило редиректит на порт `8000 + zoneid`, а nginx может быть настроен на другой порт.

**Симптомы:** При переходе по IP редирект работает, страница не открывается, или nginx не стартует (Address already in use).

**Диагностика:**
```bash
# Проверить какой порт слушает nginx
grep 'listen' /var/etc/nginx-<zone>-CaptivePortal.conf

# Проверить RDR
pfctl -s nat | grep rdr

# Проверить что слушает
netstat -an -p tcp | grep -E '8000|8002'
```

**Решение:** Установить `listenporthttp` в конфиге:
```bash
sed -i '' 's|<listenporthttp>8000</listenporthttp>|<listenporthttp>8002</listenporthttp>|' /conf/config.xml
```

### 🔴 3. DNS заблокирован

**Проблема:** Блокирующее правило captive portal блокирует DNS-запросы (порт 53). Клиент не может резолвить домены, поэтому HTTP-запрос по домену не доходит до RDR.

**Симптомы:** По IP редирект работает, по домену — нет.

**Решение:** Добавить DNS passthru правила **перед** блокирующим правилом:
```bash
# Временно (до следующего filter_configure)
pfctl -s rules > /tmp/rules.txt
awk '/ridentifier 13003/ {
    print "pass in quick on vmx0 inet proto udp from any to any port = domain keep state (if-bound, sloppy) tag cpzoneid__auth"
    print "pass in quick on vmx0 inet proto tcp from any to any port = domain flags S/SA keep state (if-bound, sloppy) tag cpzoneid__auth"
    print $0; next
} {print}' /tmp/rules.txt | pfctl -f -
```

**Постоянное решение:** Добавить DNS passthru через allowed IPs в конфиге captive portal, или через firewall правило в XML.

### 🔴 4. Кастомная страница не отображается

**Проблема:** HTML-код сохранён в конфиге, но pfSense не генерирует файл `/var/etc/captiveportal_<zone>.html`.

**Причина:** `captiveportal_configure_zone()` проверяет `is_array($cpcfg['page'])`, но SimpleXML-объект не является массивом.

**Решение:** Скопировать файл вручную с заменой переменных:
```bash
cat portal.html | sed \
  -e 's/\$PORTAL_ZONE\$/#PORTAL_ZONE#/g' \
  -e 's/\$PORTAL_REDIRURL\$/#PORTAL_REDIRURL#/g' \
  -e 's/\$PORTAL_MESSAGE\$/#PORTAL_MESSAGE#/g' \
  -e 's/\$CLIENT_MAC\$/#CLIENT_MAC#/g' \
  -e 's/\$CLIENT_IP\$/#CLIENT_IP#/g' \
  -e 's/\$PORTAL_ACTION\$/#PORTAL_ACTION#/g' \
  > /var/etc/captiveportal_<zone>.html
```

### 🔴 5. Кнопка не отправляет форму

**Проблема:** В JavaScript кнопка отключается (`btn.disabled = true`) перед отправкой. Отключённая кнопка (`<button disabled>`) не отправляет форму.

**Решение:** Не отключать кнопку, а просто вызвать `form.submit()`:
```javascript
btn.addEventListener('click', (e) => {
    spinner.style.display = 'block';
    form.submit();  // вместо btn.disabled = true
});
```

### 🔴 6. Сброс pf-правил

**Проблема:** `pfctl -f` или `pfctl -i <interface> -f` перезаписывает ВЕСЬ ruleset, а не добавляет правила. После этого пропадают все правила, включая RDR, NAT, и базовые.

**Симптомы:** После ручного добавления правил через `pfctl -f` перестаёт работать весь firewall.

**Решение:** Всегда восстанавливать через `filter_configure()`:
```bash
php -r "
require_once('globals.inc');
require_once('filter.inc');
\$config = parse_config();
filter_configure();
"
```

### 🔴 7. PHP с require_once падает

**Проблема:** При выполнении PHP-скриптов через SSH с `require_once('globals.inc')` может быть ошибка (exit code 255 без вывода).

**Причина:** pfSense использует свою среду выполнения. Некоторые функции доступны только через веб-интерфейс или PHP-FPM.

**Решение:** Использовать `php -r` с короткими командами, или писать скрипты в файл и запускать их. Если не работает — редактировать XML напрямую через sed.

### 🔴 8. После перезагрузки pfSense портал не стартует

**Проблема:** После ребута captive portal может не запуститься, если конфиг повреждён.

**Решение:** Пересоздать зону через веб-интерфейс (удалить и создать заново), или через PHP:
```bash
php -r "
require_once('globals.inc');
require_once('captiveportal.inc');
require_once('filter.inc');
\$config = parse_config();
\$cpcfg = \$config['captiveportal']['cai_free'];
captiveportal_configure_zone(\$cpcfg, true);
captiveportal_init_webgui_zonename('cai_free');
filter_configure();
"
```

---

## Полезные команды

```bash
# Проверить статус портала
cat /var/run/nginx-<zone>-CaptivePortal.pid 2>/dev/null && echo "PID exists" || echo "No PID"
netstat -an -p tcp | grep -E '8000|8002'

# Проверить pf-правила
pfctl -s rules | grep -E 'captive|portal|8002|auth|allowed'
pfctl -s nat | grep rdr
pfctl -t cpzoneid__cpips -T show
pfctl -t cpzoneid__auth -T show

# Проверить логи
tail -20 /var/log/captiveportal.log
grep -i captive /var/log/system.log | tail -10

# Проверить сгенерированную страницу
cat /var/etc/captiveportal_<zone>.html | head -20

# Проверить конфиг
cat /conf/config.xml | grep -A 20 '<cai_free>'
```

---

## Связанные страницы

- [[concepts/network-subnets]] — Сеть и подсети
- [[inventory/cai-main-b2-tm]] — Инвентаризация сервера
