# Лабораторная работа №31-32: Введение в SQLite и Entity Framework Core

**Студент:** Катаржин Г.М. 

**Группа:** ИСП-232

**Дата выполнения:** 28.04.26

---

## Краткое описание работы

В данной лабораторной работе реализовано веб-API на ASP.NET Core с использованием Entity Framework Core и SQLite для хранения данных. Реализован полный CRUD для задач (TaskItem), включая фильтрацию, поиск, пагинацию, статистику, массовые операции и логирование SQL-запросов. Использован подход Code First с миграциями для управления схемой БД.

---

## Полезные команды `dotnet ef`

| Команда | Описание |
|--------|----------|
| `dotnet ef migrations add <Name>` | Создать новую миграцию |
| `dotnet ef database update` | Применить все неприменённые миграции |
| `dotnet ef migrations list` | Показать список миграций и их статус |
| `dotnet ef migrations remove` | Удалить последнюю миграцию (если не применена) |
| `dotnet ef database update <MigrationName>` | Откатиться до конкретной миграции |
| `dotnet ef migrations script` | Сгенерировать SQL-скрипт без применения к БД |

---

## Структура проекта (дерево файлов)

```
Lab31-32_EFCore/
├── .editorconfig 
├── README.md
├── img/
└── TaskDb/
    ├── TaskDb.csproj
    ├── taskdb.db
    ├── .gitignore    
    ├── Program.cs
    ├── appsettings.json
    ├── appsettings.Development.json
    ├── Models/
    │   ├── TaskItem.cs
    │   └── TaskDtos.cs
    ├── Data/
    │   └── AppDbContext.cs
    ├── Controllers/
    │   └── TasksController.cs
    └── Migrations/
        ├── 20260428013143_InitialCreate.cs
        ├── 20260428013143_InitialCreate.Designer.cs
        ├── 20260428013941_bebebe.cs
        ├── 20260428013941_bebebe.Designer.cs
        ├── 20260428025139_AddDueDateToTask.cs
        ├── 20260428025139_AddDueDateToTask.Designer.cs
        └── AppDbContextModelSnapshot.cs
```

---

## Список всех реализованных маршрутов

| Метод | Маршрут | Описание |
|-------|---------|----------|
| GET | `/api/tasks` | Получить все задачи с фильтрацией по `completed` и `priority` |
| GET | `/api/tasks/{id}` | Получить задачу по ID |
| POST | `/api/tasks` | Создать новую задачу |
| PUT | `/api/tasks/{id}` | Обновить задачу полностью |
| PATCH | `/api/tasks/{id}/complete` | Переключить статус выполнения задачи |
| DELETE | `/api/tasks/{id}` | Удалить задачу |
| GET | `/api/tasks/search` | Поиск задач по тексту, приоритету, статусу |
| GET | `/api/tasks/stats` | Получить статистику по задачам |
| GET | `/api/tasks/paged` | Пагинированный вывод задач |
| GET | `/api/tasks/overdue` | Получить просроченные задачи |
| PATCH | `/api/tasks/complete-all` | Пометить все невыполненные задачи как выполненные |
| DELETE | `/api/tasks/completed` | Удалить все выполненные задачи |

---

## Таблица применённых миграций

| Имя миграции | Дата создания | Описание изменений |
|--------------|---------------|---------------------|
| `InitialCreate` | 2026-01-01 | Создание таблицы `Tasks` с полями: Id, IsCompleted, CreatedAt, Title, Description, Priority + начальные данные (seed) |
| `AddDueDateToTask` | 2026-01-02 | Добавление nullable поля `DueDate` в таблицу `Tasks` |

---

## Сравнительная таблица LINQ vs SQL

| LINQ (C#) | SQL (генерируется EF Core) |
|-----------|----------------------------|
| `.Where(t => t.IsCompleted == false)` | `WHERE is_completed = 0` |
| `.OrderBy(t => t.CreatedAt)` | `ORDER BY created_at ASC` |
| `.OrderByDescending(t => t.CreatedAt)` | `ORDER BY created_at DESC` |
| `.Take(10)` | `LIMIT 10` |
| `.Skip(20).Take(10)` | `OFFSET 20 LIMIT 10` |
| `.Count()` | `SELECT COUNT(*)` |
| `.Any(t => t.Priority == "High")` | `SELECT EXISTS(... WHERE priority = 'High')` |
| `.Max(t => t.Id)` | `SELECT MAX(id)` |
| `.GroupBy(t => t.Priority)` | `GROUP BY priority` |
| `.Select(t => t.Title)` | `SELECT title` |
| `.Contains("SQL")` | `LIKE '%SQL%'` |

---

## Главные выводы

1. **EF Core — это переводчик между C# и SQL.** Вы пишете LINQ, он генерирует соответствующий SQL-запрос. Это позволяет работать с базой данных через объекты, не зная синтаксиса SQL.

2. **Миграции — это система контроля версий для структуры БД.** Как Git отслеживает изменения кода, так миграции отслеживают изменения схемы базы данных. Это критически важно для командной разработки.

3. **Code First удобнее ручного написания SQL.** Изменил класс → создал миграцию → применил её → база обновлена. Нет необходимости писать `ALTER TABLE` вручную.

4. **`SaveChangesAsync()` — ключевой момент фиксации.** До вызова этого метода все изменения живут только в памяти контекста. Только после него EF Core отправляет INSERT/UPDATE/DELETE в базу данных.

5. **`async/await` при работе с БД — обязательный стандарт.** Блокировка потока на время ожидания ответа от базы данных снижает производительность сервера. Асинхронность позволяет обрабатывать другие запросы параллельно.

---

## Итоговая сравнительная таблица: Хранение в памяти vs EF Core + SQLite

| Концепция             | Хранение в памяти          | EF Core + SQLite           |
|-----------------------|----------------------------|----------------------------|
| **Хранение данных**       | `static List<T>` в RAM     | Файл `.db` на диске        |
| **После перезапуска**     | Данные пропадают           | Данные сохраняются         |
| **Поиск по условию**      | LINQ to Objects            | LINQ to Entities → SQL     |
| **Создание структуры**    | Не нужно                   | Миграции (`dotnet ef`)     |
| **Начальные данные**      | Хардкод в коде             | `HasData()` в миграции     |
| **Получение данных**      | `list.FirstOrDefault(...)` | `await db.Table.FindAsync(id)` |
| **Добавление**            | `list.Add(item)`           | `db.Table.Add(item) + SaveChangesAsync()` |
| **Удаление**              | `list.Remove(item)`        | `db.Table.Remove(item) + SaveChangesAsync()` |
| **Масштабируемость**      | Ограничена RAM             | Гигабайты данных           |
| **Транзакции**            | Нет                        | Встроены в EF Core         |


