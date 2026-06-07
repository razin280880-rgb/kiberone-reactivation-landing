# kiberone-reactivation-landing

Мини-лендинг под кампанию реактивации базы «пробник без договора» (2143 контакта).
Один файл `index.html` без зависимостей и без бэкенда — все заявки уходят
в WhatsApp с предзаполненным текстом через `wa.me/...`.

## Структура

- **Hero** — заголовок «новый формат», бейдж «Новый формат · Сентябрь 2026», блок персонализации (по UTM)
- **Видео-плейсхолдер** — место под 90-секундный ролик «до/после» (маркетолог подменит на YouTube embed)
- **Countdown** — авто-таймер «до 1 сентября осталось N дней»
- **3 CTA** в порядке убывания готовности:
  1. 🟡 Записаться на AI-пробный (модалка с 3 полями → отправляет в WhatsApp)
  2. 🟢 Получить расписание сентября (открывает WhatsApp с готовым текстом)
  3. 🔵 Задать вопрос менеджеру (открывает WhatsApp с приветствием)
- **Блок «Что нового»** — 3 буллета (сценарий с ИИ / озвучка / готовая игра за 1 час)
- **Social proof** — 7 городов / 3000+ детей / 6-14 лет / 0 ₽
- **Footer** — список городов

## UTM-параметры для персонализации

При запуске рекламы в VK Ads / Яндекс / SMS / WhatsApp ссылка должна быть вида:

```
https://landing.it-kiber.ru/?name=Айнур&child=Тимур&last=ноябрь%202024&city=Челны
```

| Параметр | Что делает |
|---|---|
| `name` | подставляется в обращение и в форму записи |
| `child` | имя ребёнка — попадает в hero и в WhatsApp-сообщение |
| `last` | период прошлого пробного (формирует текст «в [месяц год] [ребёнок] был у нас») |
| `city` | определяет, в какой филиальный WhatsApp ведёт кнопка |

⚠️ Городские параметры (lowercase): `челны`, `нижнекамск`, `казань`, `елабуга`, `краснодар`, `сургут`, `пермь`.

Если параметров нет — лендинг работает в общем режиме без персонализации, кнопки ведут на центральный WhatsApp (`DEFAULT_PHONE` в коде).

## Что должен сделать маркетолог перед стартом волны

1. **Заменить WhatsApp-номера в `BRANCHES` и `DEFAULT_PHONE`** (строки 540+ в index.html).
2. **Загрузить видео** — заменить плейсхолдер на YouTube embed (строка ~620, есть готовый комментарий-template).
3. **Зашить домен** (предложение — `reactivation.it-kiber.ru` или `start.it-kiber.ru`).
4. **Поставить Яндекс.Метрику и пиксель VK** для подсчёта конверсий.
5. **Подготовить UTM-таблицу** — для каждого контакта из базы своя ссылка с подставленным `name/child/last/city`. Генерируется скриптом (см. ниже).

## Деплой

Простейший вариант — Cloudflare Pages (как `kiber-summer-landing`):
1. Создать репо на GitHub `kiberone-reactivation-landing`
2. В Cloudflare Pages — Connect to Git → выбрать репо → root directory `/`
3. Получить домен `reactivation-landing.pages.dev` → CNAME на свой `reactivation.it-kiber.ru`
4. На push в `main` — автодеплой

Альтернатива — статика на любом хостинге, файл один.

## Генератор персональных ссылок

Чтобы для каждого контакта из xlsx сгенерировать персональную ссылку — будет в файле `gen_links.py` (TODO для маркетолога/помощника):

```python
import urllib.parse, openpyxl
wb = openpyxl.load_workbook('trial_dropoffs_2026-06-06.xlsx')
ws = wb['Пришли, не купили']
base = 'https://reactivation.it-kiber.ru/'
for row in ws.iter_rows(min_row=5, values_only=True):
    city, _, child, _, parent, phone, *_ = row
    last = ''  # можно вытащить из колонки «Дата посл. пробного»
    qs = urllib.parse.urlencode({
        'name': parent.split()[0] if parent else '',
        'child': child.split()[1] if child and len(child.split())>1 else child,
        'city': city,
    })
    print(f"{phone}\t{base}?{qs}")
```

## Связанные документы

- `kiberone-management/130-strategiya-omni-reaktivacii-bazy-bezdogovor.md` — общая стратегия
- `kiberone-management/131-skript-avgust-vozvrat-k-uchebnomu-godu.md` — скрипт обзвона
- `kiberone-management/132-metodika-rabota-s-bazoy-dropoffs.md` — методика работы с базой
- `salary_bot/trial_dropoffs_report.py` — источник базы
