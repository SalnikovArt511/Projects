# Tasks Board

Личный kanban-трекер на чистом HTML — без бэкенда, без базы данных, без фреймворков. Хостится на GitHub Pages, встраивается в Notion через embed.

**Живой сайт:** https://salnikovart511.github.io/Projects/

---

## Что это и как работает

```
index.html  ←  единственный файл всего проекта
    │
    ├── Данные (TASKS[])      — все задачи прямо в коде
    ├── Рендер (render())     — строит HTML из данных
    ├── Действия              — toggleSub, addTask, applyEdit...
    └── GitHub API            — кнопка "Сохранить" делает git commit
```

Когда ты нажимаешь **"💾 Сохранить"**, страница:
1. Берёт текущий `TASKS[]` из памяти
2. Вставляет его в свой собственный HTML-код
3. Отправляет этот HTML в GitHub API как новый коммит
4. GitHub Pages публикует обновление за ~30 секунд

Данные живут прямо в файле — никакой базы данных не нужно.

---

## Структура репозитория

```
Projects/
├── index.html      ← весь проект
└── README.md       ← этот файл
```

---

## Как работать с задачами

### Ежедневная работа (браузер)

| Что сделать | Как |
|---|---|
| Отметить подзадачу | Кликнуть на чекбокс |
| Переместить задачу в другую колонку | ✏️ Изменить → Статус → выбрать → Сохранить |
| Добавить задачу | ✏️ Редактировать → форма в нужной колонке |
| Добавить подзадачу | ✏️ Редактировать → поле внизу карточки |
| Изменить название / приоритет / дедлайн | ✏️ Изменить на карточке |
| Удалить задачу | ✏️ Изменить → Удалить |
| Сохранить всё на GitHub | 💾 Сохранить |

### Колонки

| Колонка | Смысл |
|---|---|
| 🔄 In progress | Активные задачи, работа идёт прямо сейчас |
| 📥 Inbox | Входящие, нужно разобрать |
| 📋 Plans | Запланировано, но не началось |
| ✔ Done | Завершено |

---

## Настройка GitHub Token (один раз)

Токен нужен, чтобы кнопка "Сохранить" могла делать коммиты.

1. Открой https://github.com/settings/tokens/new?scopes=repo&description=kanban
2. Нажми **Generate token**
3. Скопируй токен (показывается только один раз)
4. На сайте нажми **⚙️** → вставь токен → **Сохранить настройки**

Токен хранится в `localStorage` браузера — вводить повторно не нужно.  
Если работаешь с другого устройства — нужно ввести токен снова.

> **Безопасность:** токен имеет доступ только к репозиторию (`repo` scope). Не коммить токен в код.

---

## Как обновить файл вручную (запасной способ)

Если GitHub API недоступен или нужно сделать большие изменения:

1. Внести правки в код
2. Открыть https://github.com/salnikovart511/Projects
3. Нажать на `index.html` → карандаш ✏️ → вставить код → **Commit changes**

Или через git:
```bash
git clone https://github.com/salnikovart511/Projects.git
# внести правки
git add index.html
git commit -m "update tasks"
git push
```

---

## Как добавить новую колонку

Найти в `index.html` массив `COLS` (строка ~85) и добавить объект:

```js
const COLS = [
  {key:"inprogress", label:"🔄 In progress"},
  {key:"inbox",      label:"📥 Inbox"},
  {key:"plans",      label:"📋 Plans"},
  {key:"waiting",    label:"⏳ Waiting"},   // ← новая колонка
  {key:"done",       label:"✔ Done"},
];
```

Всё остальное (рендер, форма добавления, select статуса) подхватится автоматически.

---

## Как добавить новое поле к задаче

Пример: добавить поле `link` (ссылка на документ).

**1. Добавить в данные задачи:**
```js
{id:5, name:"Новая задача", ..., link:"https://docs.google.com/..."}
```

**2. Показать на карточке** — в функции `renderCard()`:
```js
// после card-meta
${task.link ? `<a href="${task.link}" target="_blank" style="font-size:11px;color:var(--accent)">🔗 Открыть документ</a>` : ""}
```

**3. Добавить в редактор** — в функции `inlineEditPanel()`:
```js
<div class="ie-row">
  <span class="ie-label">Ссылка</span>
  <input class="ie-input" id="ie_link_${task.id}" value="${task.link||""}">
</div>
```

**4. Сохранить в `applyEdit()`:**
```js
const lnk = document.getElementById(`ie_link_${id}`)?.value.trim();
t.link = lnk;
```

---

## Как добавить новый фильтр

Пример: фильтр "Только горящие дедлайны".

**1. Добавить кнопку в HTML:**
```html
<button class="filter-btn" data-filter="hot" onclick="setFilter('hot')">🔥 Горит</button>
```

**2. Добавить логику в функцию `visible()`:**
```js
function visible(task) {
  if(currentFilter === "hot") return task.deadlineHot === true;
  // ... остальные фильтры
}
```

---

## Как встроить в Notion

1. На странице Notion напечатай `/embed`
2. Вставь ссылку: `https://salnikovart511.github.io/Projects/`
3. Нажми **Embed link**
4. Растяни блок до нужного размера

**Если в Notion показывается старая версия:**
- Кликни на embed → `•••` → **Reload**
- Или добавь `?v=2` к ссылке (меняй цифру при каждом обновлении)

---

## Архитектурные решения и почему так

| Решение | Почему |
|---|---|
| Один HTML-файл | Нет сборки, нет зависимостей, легко редактировать и деплоить |
| Данные в коде, не в БД | Нет бэкенда, нет хостинга, полный контроль над данными |
| GitHub API для сохранения | Бесплатно, есть история изменений, не нужен отдельный сервер |
| GitHub Pages для хостинга | Бесплатно, HTTPS из коробки, обновляется при каждом push |
| Нет фреймворков (React/Vue) | Меньше кода, быстрее загрузка, легче читать и менять |

---

## Возможные интеграции

### Telegram-бот → канбан
Бот при создании задачи в Notion может параллельно добавлять задачу в `TASKS[]` через GitHub API:

```python
# В planning_bot.py после create_notion_task()
import base64, requests, json

def update_kanban(task_name, priority, deadline, owner):
    token = os.getenv("GITHUB_TOKEN")
    repo  = "salnikovart511/Projects"
    file  = "index.html"
    
    # Получить текущий файл и SHA
    r = requests.get(f"https://api.github.com/repos/{repo}/contents/{file}",
                     headers={"Authorization": f"Bearer {token}"})
    sha     = r.json()["sha"]
    content = base64.b64decode(r.json()["content"]).decode()
    
    # Вставить новую задачу в TASKS[]
    new_task = f'''  {{id:{int(time.time())},name:"{task_name}",priority:"{priority}",deadline:"{deadline}",deadlineHot:false,status:"inbox",owner:"{owner}",subs:[]}},\n'''
    content  = content.replace("let TASKS = [\n", f"let TASKS = [\n{new_task}")
    
    # Закоммитить
    requests.put(f"https://api.github.com/repos/{repo}/contents/{file}",
        headers={"Authorization": f"Bearer {token}", "Content-Type": "application/json"},
        json={"message": f"add task: {task_name}", "content": base64.b64encode(content.encode()).decode(), "sha": sha})
```

### Reclaim.ai → дедлайны
Reclaim синхронизируется с Notion напрямую — отдельной интеграции с канбаном не нужно.

### Google Calendar
Можно добавить кнопку "Добавить в календарь" на карточку — она откроет prefilled форму Google Calendar:
```js
const gcalUrl = `https://calendar.google.com/calendar/render?action=TEMPLATE&text=${encodeURIComponent(task.name)}&dates=${deadline}/${deadline}`;
```

---

## Переменные окружения (для Telegram-бота)

Хранятся в файле `.env` рядом с `planning_bot.py`:

```env
TELEGRAM_TOKEN=        # от @BotFather
GEMINI_API_KEY=        # от aistudio.google.com
NOTION_TOKEN=          # от notion.so/my-integrations
NOTION_DB_ID=          # ID базы Tasks из URL
MANAGER_CHAT_ID=       # Telegram ID руководителя
ASSISTANT_CHAT_ID=     # Telegram ID ассистента
GITHUB_TOKEN=          # для интеграции бота с канбаном (опционально)
```
