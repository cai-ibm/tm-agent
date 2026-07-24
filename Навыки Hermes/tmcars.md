---
name: tmcars
title: TMCARS — парсинг объявлений
description: Извлечение данных с TMCARS (tmcars.info) — лоты недвижимости, фото, характеристики, перевод на русский
category: devops
triggers:
  - tmcars
  - tmcars.info
  - парсинг объявлений
  - аренда
  - недвижимость туркменистан
---

# TMCARS — Парсинг объявлений

Извлечение структурированных данных со страниц объявлений TMCARS (Next.js, RSC).

## URL-шаблон

```
https://tmcars.info/others/{id}/{slug}
https://tmcars.info/others/{id}
```

## Данные для извлечения

Страница содержит JSON-LD в `<script type="application/ld+json">` с типом `Product`:

```json
{
  "@type": "Product",
  "name": "Название лота",
  "description": "Описание",
  "image": ["url1", "url2", ...],
  "offers": {
    "price": 10000,
    "priceCurrency": "TMT"
  }
}
```

## Поля для карточки

- **Название** — из `<h1>` или `Product.name`
- **Цена** — из `Product.offers.price` + `TMT`
- **Локация** — Ашхабад / район (из текста страницы)
- **Дата** — `\d{2}\.\d{2}\.\d{4}` (первое совпадение)
- **Просмотры** — число после иконки `lucide-eye`
- **Фото** — массив `Product.image` (URL thumbnail)
- **Характеристики** — из таблицы на странице (тип, комнаты, этаж, ремонт, лифт)
- **Описание** — из `Product.description` или `<p class="text-text-muted...">`

## Команды для извлечения

### 1. Получить JSON-LD + метаданные

```bash
curl -s 'https://tmcars.info/others/{ID}' -H 'User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36' | python3 -c "
import sys, re, json
html = sys.stdin.read()
ld = re.findall(r'<script type=\"application/ld\+json\">(.*?)</script>', html, re.DOTALL)
for l in ld:
    try:
        data = json.loads(l)
        if data.get('@type') == 'Product':
            print('PRODUCT:', json.dumps(data, indent=2, ensure_ascii=False))
    except: pass
desc = re.search(r'<p class=\"text-text-muted[^\"]*\".*?>(.*?)</p>', html, re.DOTALL)
if desc: print('DESC:', desc.group(1).strip())
views = re.search(r'<svg[^>]*lucide-eye[^>]*>.*?</svg>\s*(\d+)', html, re.DOTALL)
if views: print('VIEWS:', views.group(1))
date = re.search(r'(\d{2}\.\d{2}\.\d{4})', html)
if date: print('DATE:', date.group(1))
price = re.search(r'(\d[\d ]*)\s*TMT', html)
if price: print('PRICE:', price.group(1), 'TMT')
images = re.findall(r'\"(https://media\.tapgo\.biz/tmcars-images/thumbnail/[^\"]+\.webp)\"', html)
seen = set()
for img in images:
    if img not in seen:
        seen.add(img)
        print('IMAGE:', img)
title = re.search(r'<h1[^>]*>(.*?)</h1>', html)
if title: print('TITLE:', title.group(1))
loc = re.search(r'Ашхабад\s*/\s*([^<]+)', html)
if loc: print('LOCATION: Ашхабад /', loc.group(1).strip())
"
```

### 2. Скачать фото

```bash
curl -s '{IMAGE_URL}' -o /tmp/tmcars_N.webp -H 'User-Agent: Mozilla/5.0'
```

### 3. Отправить фото в чат

Использовать `MEDIA:/tmp/tmcars_N.webp` в ответе.

## Перевод на русский

Описание на туркменском/смешанном языке переводится вручную:

| Туркменский | Русский |
|---|---|
| arenda | аренда |
| komnat | комнат |
| spalni | спален |
| banya-tualet | санузел |
| hemme serti bar | все условия есть |
| arassa | чистая |
| yashajak | для проживания |
| adam ucin | для человека |
| baha | цена |
| manat | манат |
| rieltor | риелтор |
| patent bar | патент имеется |
| yewro remont | евроремонт |
| dizaýner remont | дизайнерский ремонт |
| galan soraglar | остальные вопросы |
| janlashyp bilersiniz | можете звонить |

## Формат карточки

```
**Название лота**

📍 **Ашхабад** / район

💰 **Цена:** X TMT
📅 **Дата:** DD.MM.YYYY
👁 **Просмотров:** N
📸 **Фото:** N шт.

**Характеристики:**
- **Тип:** Элитка/Квартира
- **Комнат:** N
- **Этаж:** N / N
- **Ремонт:** ...
- **Лифт:** Есть/Нет

**Описание:**
(переведённый текст)

**Ссылка:** {url}
```

## Примечания

- Телефон скрыт за кнопкой «Показать» — требует авторизации на сайте
- Фото в формате webp, thumbnail-размер
- Для пакетного скачивания всех фото: взять массив `Product.image` и скачать каждый URL
