
---
# Developer Guide: Workflow для программистов (CRM C# + Blazor)

Этот документ — шпаргалка по Git и правилам коммитов в проекте CRM. Следуй ему на каждой задаче.

## 1. Общие правила

- Ты работаешь **только в своей feature-ветке**.
- **Никогда не делай push** напрямую в `develop` или `main`. У тебя нет на это прав, да и процесс так не работает.
- **Не мержь** свои MR — это делает Bazilio после ревью.
- **Не ставь теги** и **не запускай деплой**.
- **Не пиши в CHANGELOG.md** — это зона ответственности Bazilio.
- Если ветка висит больше 3–5 дней — разбей задачу на части. Длинные ветки почти всегда приводят к конфликтам.

---

## 2. Именование веток

Формат: `тип/ТИКЕТ-краткое-описание`

Используй только эти префиксы:

| Префикс | Когда использовать | Пример |
|---------|--------------------|--------|
| `feature/` | Новая функциональность | `feature/CRMT-12-export-to-excel` |
| `bugfix/` | Исправление бага (не срочно) | `bugfix/CRMT-15-fix-filter-reset` |
| `refactor/` | Рефакторинг без смены поведения | `refactor/CRMT-18-extract-export-service` |
| `chore/` | Настройки, зависимости, конфиги | `chore/CRMT-20-update-nuget-packages` |

**Правила:**
- Английский язык.
- Слова через дефис.
- Номер тикета (CRMT‑XX) обязателен!!!
- 2–4 слова, отражающих суть.

---

## 3. Именование коммитов

Формат: `тип: описание (ТИКЕТ)`

| Префикс | Назначение | Пример |
|---------|------------|--------|
| `feat:` | Новая функциональность | `feat: add Excel export (CRMT-12)` |
| `fix:` | Исправление бага | `fix: handle null in login (CRMT-99)` |
| `refactor:` | Рефакторинг | `refactor: extract export to service (CRMT-18)` |
| `docs:` | Документация | `docs: add deployment guide (CRMT-21)` |
| `chore:` | Настройки, зависимости | `chore: update NuGet packages (CRMT-20)` |
| `test:` | Тесты | `test: add export service tests (CRMT-12)` |
| `style:` | Форматирование | `style: fix indentation in ExportController` |

**Как писать правильно:**
- Описание в настоящем времени: `add`, `fix`, `update`, `remove`.
- Тикет в скобках в конце.
- Одна логическая задача = один коммит.
- Не более ~72 символов в первой строке.
- Английский язык в описании, русский/английский в пояснении.
- Если нужно пояснение — пустая строка и текст ниже:

fix: handle null in login service (CRMT-99)

When username is null, the service threw NullReferenceException.
Added explicit null check and return BadRequest.

---

## 2. Полный цикл работы над задачей

### Шаг 1. Старт: создать ветку

```bash
git checkout develop
git pull origin develop
git checkout -b feature/CRMT-12-export-to-excel
```

### Шаг 2. Работа: коммиты

```bash
git add .
git commit -m "feat: add export service skeleton (CRMT-12)"
# ... пишешь код ...
git add .
git commit -m "feat: implement Excel generation (CRMT-12)"
git add .
git commit -m "fix: handle empty dataset (CRMT-12)"
```

### Шаг 3. Пуш ветки

```bash
git push origin feature/CRMT-12-export-to-excel
```

### Шаг 4. Перед MR: синхронизация с develop (ОБЯЗАТЕЛЬНО)

Это нужно, чтобы избежать конфликтов. Делай это каждый раз перед созданием MR и если ветка долго висит.

```bash
git checkout develop
git pull origin develop
git checkout feature/CRMT-12-export-to-excel
git rebase develop
```

Если `rebase` остановился с `CONFLICT` — смотри раздел «Разрешение конфликтов».

После успешного rebase:

```bash
git push origin feature/CRMT-12-export-to-excel --force-with-lease
```

> `--force-with-lease` — безопасный форс-пуш. Он не перезапишет чужую историю, если кто-то успел запушить в эту ветку.

### Шаг 5. Создать Merge Request в GitLab

- **Source branch:** `feature/CRMT-12-export-to-excel`
- **Target branch:** `develop`
- **Заголовок MR:** `feat: add Excel export for contacts (CRMT-12)`
- **Описание MR:** обязательно напиши:
  - Что сделано.
  - Как проверить (страница, шаги).
  - Есть ли миграции БД (если да — укажи, какие).
  - Ссылка на тикет: `Closes CRMT-12` или `Closes #12`.

### Шаг 6. Ревью и правки

Bazilio оставит комментарии. Если есть замечания:

```bash
# Правь код
git add .
git commit -m "fix: address review comments (CRMT-12)"
git push origin feature/CRMT-12-export-to-excel
```

Если Bazilio просит сделать rebase (появился конфликт) — повтори шаги 4–5.

### Шаг 7. После мержа: удалить ветку

Когда Bazilio смержил MR в GitLab:

```bash
git checkout develop
git pull origin develop
git branch -D feature/CRMT-12-export-to-excel
```

Удалённую ветку на сервере GitLab удалит автоматически (настроено Bazilio).

---

## 5. Разрешение конфликтов при rebase

Если при `git rebase develop` видишь: `CONFLICT (content): Merge conflict in add_for_test.txt`

1. Открой файл с конфликтом в редакторе.
2. Найди маркеры конфликта:

   ```text
   <<<<<<< HEAD
   ... (код из develop)
   =======
   ... (код из твоей ветки)
   >>>>>>> feature/CRMT-12-export-to-excel
   ```

3. Оставь нужный вариант или объедини вручную.
4. Сохрани файл.
5. Добавь исправленный файл:

   ```bash
   git add add_for_test.txt
   ```

6. Продолжи rebase:

   ```bash
   git rebase --continue
   ```

7. Повторяй шаги 1–6, пока rebase не завершится.

8. Пушь обновлённой ветки:

   ```bash
   git push origin feature/CRMT-12-export-to-excel --force-with-lease
   ```

---

## 6. Частые ошибки и как их избежать

| Ошибка | Как правильно |
|--------|---------------|
| Работа напрямую в `develop` | Всегда создавай feature-ветку от `develop`. |
| Коммит `update` или `fix bug` | Используй префиксы и формат: `fix: handle null in login (CRMT-99)`. |
| MR без описания | Пиши: что сделано, как проверить, есть ли миграции. |
| Забыли rebase перед MR | Всегда делай rebase на свежий `develop` перед MR. |
| Force-push без `--force-with-lease` | Только `--force-with-lease`. |
| Долгая ветка (неделя+) | Разбивай задачу на части по 1–3 дня. |
| Конфликт в MR, ждёшь Bazilio | Сам сделай rebase и разреши конфликт. |

---

## 7. Шпаргалка: быстрый старт

```bash
# Старт
git checkout develop && git pull origin develop
git checkout -b feature/ТИКЕТ-описание

# Работа
git add . && git commit -m "тип: что сделал (ТИКЕТ)"
git push origin feature/ТИКЕТ-описание

# Перед MR
git checkout develop && git pull origin develop
git checkout feature/ТИКЕТ-описание
git rebase develop
# Если есть конфликты — реши их
git push origin feature/ТИКЕТ-описание --force-with-lease

# Создать MR в GitLab → ждать ревью

# Правки по ревью
git add . && git commit -m "fix: address review (ТИКЕТ)"
git push origin feature/ТИКЕТ-описание

# После мержа
git checkout develop && git pull origin develop
git branch -D feature/ТИКЕТ-описание
```

---

## 8. Особенности CRM-проекта 

- **Совместимость со статусами и полями.** В новой CRM (C# + Blazor) значения статусов и кодов должны совпадать с теми, что были в PowerBuilder. Иначе отчёты и интеграции будут работать некорректно. Если сомневаешься — спроси Bazilio.
- **Тестовые файлы.** Файлы вроде `add_for_test.*`, которые ты создаёшь локально для проверки, не должны попадать в репозиторий. Добавь их в `.gitignore` или удали из индекса.

---

Если есть вопросы по конкретному модулю (офисный, диспетчер, инженеры СТП) или по работе с БД — спроси Bazilio или посмотри примеры в истории коммитов.
```