# AT Logistika — API интеграция для отчётов по пробегу

**Сайт:** https://atlogistika.com  
**Логин:** srv / SuperP@ssword  
**SID-токен:** `1eec87edaf419106a95f13ec30e48aff`  
**Часовой пояс:** UTC+5 (Asia/Ashgabat)

## API endpoints

### Авторизация
```
GET /backend/ax/user/login.php?sid=<SID>
```
Возвращает cookie `PILOTID` (HttpOnly, Secure, SameSite=Strict).  
Редиректит на главную страницу. SID-логин не требует пароля.

### Список ТС (текущие данные)
```
GET /backend/ax/current_data.php
```
Возвращает JSON со всеми объектами. Ключевые поля:
- `id` — ID ТС
- `name` — название
- `veh` — госномер
- `lat`, `lon` — координаты
- `len` — одометр (общий пробег, км)
- `motor_hours` — моточасы
- `sensors` — датчики (топливо, зажигание, температура и т.д.)
- `last_event` — последнее событие (тип, скорость, время)
- `satsinview` — количество спутников
- `tags` — теги (включая ID датчиков)

### Отчёт по пробегу (Mileage Report)
```
POST /backend/ax/reports.php
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
X-Requested-With: XMLHttpRequest
```

**Параметры:**
| Параметр | Значение | Описание |
|---|---|---|
| `report_type` | `4` | Mileage Report |
| `veh_id` | ID ТС или пусто | Транспортное средство |
| `start_date` | `DD.MM.YYYY HH:MM` | Начало периода |
| `stop_date` | `DD.MM.YYYY HH:MM` | Конец периода |
| `start_time` | `HH:MM` | Не используется (всегда `00:00`) |
| `stop_time` | `HH:MM` | Не используется (всегда `00:00`) |
| `start_month` | `MM.YYYY` | Месяц начала |
| `stop_month` | `MM.YYYY` | Месяц конца |
| `explode` | `1` | Разбивка по дням |
| `lang` | `en` | Язык |
| `trip_types[]` | `1`, `2` | Типы поездок |
| `type` | `1` | |
| `template` | `1` | |

**Формат ответа:**
```json
{
  "data": {
    "1227 AGB": {
      "20.07.2026 - 21.07.2026": [
        {
          "fn": 2,
          "ts": 1784500231,
          "te": 1784500271,
          "agentid": 5668,
          "length": 0.01,
          "data": [
            "1227 AGB",           // [0] название ТС
            1784500231,           // [1] время начала (UTC timestamp)
            1784500271,           // [2] время конца (UTC timestamp)
            {"lat":"...","lon":"..."},  // [3] координаты начала
            {"lat":"...","lon":"..."},  // [4] координаты конца
            40,                   // [5] длительность (сек)
            0.01,                 // [6] расстояние (км)
            8,                    // [7] средняя скорость (км/ч)
            8                     // [8] макс скорость (км/ч)
          ]
        }
      ]
    }
  },
  "success": true,
  "start": "20.07.2026 00:00",
  "end": "22.07.2026 23:59"
}
```

**Важно:** API возвращает время в UTC. Конвертация в UTC+5 делается на стороне клиента.  
Фильтрация по времени суток (08:00-18:00 UTC+5) тоже на стороне клиента — API всегда получает полный день.

## Другие типы отчётов (report_type)
| ID | Название |
|---|---|
| 4 | Mileage Report |
| 23 | Summary Mileage Report |
| 1 | Trips and Parkings |
| 5 | Consolidated Report |
| 13 | Speed Report |
| 15 | Special Sensors Report |
| 16 | Fuel Report (sensors) |
| 19 | Fuel Report (in total) |
| 21 | Consolidated Report #3 |
| 24 | Fluids Consumption Report |
| 25 | Fluids Report |
| 3 | Geozones Report |
| 11 | Geozone Idling Report |
| 8 | Temperature Report |
| 14 | Speeding Report |
| 20 | Activity Report |
| 2 | Login History |

## Файлы проекта

### `/home/cai/atlog_web.py` — веб-форма (Flask)
- **Зависимости:** Flask (установлен в venv `/home/cai/atlog_web/`)
- **Запуск:** `source /home/cai/atlog_web/bin/activate && python3 atlog_web.py`
- **Порт:** 5000 (http://192.168.40.35:5000)
- **Функции:**
  - `get_session()` — авторизация через SID, получение cookie
  - `api_call()` — универсальный HTTP-запрос к API
  - `get_vehicles()` — список ТС с одометром
  - `get_mileage()` — запрос отчёта по пробегу
  - `format_trip()` — форматирование поездки (UTC → UTC+5)
  - `index()` — Flask route (GET/POST)
- **HTML-шаблон:** встроенный (render_template_string)
  - Чекбоксы с поиском по названию
  - Фильтр по дате и времени
  - Группировка по ТС с возможностью сворачивания
  - Сводка (ТС, поездки, км, время, средняя скорость)
  - Одометр в списке и в заголовке группы

### `/home/cai/atlog_mileage.py` — консольный скрипт
- **Зависимости:** только стандартная библиотека Python
- **Использование:**
  ```
  python3 atlog_mileage.py list
  python3 atlog_mileage.py "1227 AGB" 20.07.2026 22.07.2026
  python3 atlog_mileage.py "1227 AGB" 20.07.2026 22.07.2026 08:00 18:00
  python3 atlog_mileage.py all 20.07.2026 22.07.2026
  python3 atlog_mileage.py all 20.07.2026 22.07.2026 08:00 18:00
  ```

## Особенности реализации

1. **Фильтр по времени** — API не умеет корректно фильтровать по локальному времени. Решение: запрашивать полный день, фильтровать на стороне Python по времени начала поездки в UTC+5.

2. **Часовой пояс** — все timestamp-ы от API в UTC. Конвертация в UTC+5 через `datetime.fromtimestamp(ts, tz=TZ_UTC5)`.

3. **Одометр** — поле `len` в `current_data.php`. Это общий пробег ТС, не сбрасывается.

4. **Сессия** — SID-логин выдаёт новую cookie `PILOTID` при каждом запросе. Старая cookie перестаёт работать. Нужно каждый раз логиниться заново.
