# PostgreSQL Indexes

См. также: [транзакции — границы, ошибки и rollback](postgresql-transactions-cheatsheet.md).

См. также: [нормализация, денормализация и JOIN](postgresql-data-modeling-joins-cheatsheet.md).

## Содержание

1. [Что такое индекс](#1-что-такое-индекс)
2. [B-tree: как устроен и почему быстрый](#2-b-tree-как-устроен-и-почему-быстрый)
3. [Типы индексов PostgreSQL](#3-типы-индексов-postgresql)
4. [Разновидности индексов по назначению](#4-разновидности-индексов-по-назначению)
5. [Пример таблицы и составного индекса](#5-пример-таблицы-и-составного-индекса)
6. [Какие SELECT используют составной индекс](#6-какие-select-используют-составной-индекс)
7. [Как PostgreSQL комбинирует несколько индексов](#7-как-postgresql-комбинирует-несколько-индексов)
8. [Практические примеры других индексов](#8-практические-примеры-других-индексов)
9. [Как проверить индекс через EXPLAIN](#9-как-проверить-индекс-через-explain)
10. [Правила проектирования и типичные ошибки](#10-правила-проектирования-и-типичные-ошибки)
11. [Big O: сложность алгоритмов](#11-big-o-сложность-алгоритмов)

---

## 1. Что такое индекс

**Индекс** — дополнительная структура данных, позволяющая быстрее находить строки таблицы без полного перебора (`Seq Scan`).

**Аналогия:** указатель в книге: сначала находим тему в указателе, затем нужную страницу.

```sql
CREATE INDEX idx_users_email ON users(email);

SELECT * FROM users WHERE email = 'a@example.com';
```

Индекс хранит ключи и ссылки на строки таблицы. Индекс **не гарантирует**, что PostgreSQL выберет его: планировщик оценивает стоимость чтения, статистику, количество подходящих строк и другие факторы.

**Цена индекса:** дополнительное место на диске и накладные расходы на `INSERT`, `UPDATE`, `DELETE`.

## 2. B-tree: как устроен и почему быстрый

**B-tree (Balanced Tree)** — сбалансированное дерево с отсортированными ключами. PostgreSQL использует B-tree с организацией, близкой к **B+ tree**: данные индекса организованы по страницам, поиск идёт от корня к листовым страницам.

```text
                   [50]
                 /      \
           [10 | 30]  [70 | 90]
           /    ...     ...   \
        [5]   [25]    [60]   [95]
```

*Упрощённая иллюстрация принципа ветвления, не точный формат страниц PostgreSQL.*

- **Основа:** отсортированные ключи + несколько ветвей на узел + сбалансированная глубина.
- **Поиск:** на каждом уровне выбираем подходящую ветвь, отбрасывая остальные диапазоны.
- **Сложность:** ориентировочно `O(log N)` для поиска, вставки и удаления.
- **Зачем балансировка:** дерево не превращается в длинную цепочку.
- **Подходит:** `=`, `<`, `>`, `<=`, `>=`, `BETWEEN`, часто `IN`, `ORDER BY` и некоторые префиксные поиски.

```sql
CREATE INDEX idx_users_age ON users(age);
SELECT * FROM users WHERE age = 25;
SELECT * FROM users WHERE age BETWEEN 20 AND 30;
SELECT * FROM users ORDER BY age LIMIT 10;
```

## 3. Типы индексов PostgreSQL

| Тип | Где полезен | Типичные операции / особенности |
|---|---|---|
| **B-tree** (по умолчанию) | Обычные поля, сортировка, диапазоны | `=`, `<`, `>`, `BETWEEN`, `IN`, `ORDER BY` |
| **Hash** | Только точное равенство | `=`; обычно B-tree универсальнее |
| **GIN** | `jsonb`, массивы, полнотекстовый поиск | `@>`, `?`, `&&`, `@@` — в зависимости от типа и operator class |
| **GiST** | Геометрия, интервалы, nearest-neighbor | `&&`, `<@`, `<->` — при поддержке operator class |
| **SP-GiST** | Структуры пространственного разбиения | Например, поиск по точкам, префиксные структуры; зависит от operator class |
| **BRIN** | Огромные таблицы с корреляцией значения и физического порядка | Компактный индекс по диапазонам страниц, например по `created_at` |

> Конкретные операторы определяются **operator class**. Не все перечисленные операторы применимы к любому типу данных.

## 4. Разновидности индексов по назначению

Это **не отдельные методы доступа**, а способы создания индекса (обычно B-tree).

| Вид | Для чего |
|---|---|
| **Unique** | Обеспечивает уникальность ключей |
| **Composite / Multicolumn** | Несколько колонок в одном индексе |
| **Partial** | Индексирует только строки по условию `WHERE` |
| **Expression** | Индексирует выражение, например `lower(email)` |
| **Covering (`INCLUDE`)** | Добавляет неключевые колонки, потенциально позволяя `Index Only Scan` |

```sql
-- Уникальность
CREATE UNIQUE INDEX idx_users_email_unique ON users(email);

-- Составной
CREATE INDEX idx_orders_user_status ON orders(user_id, status);

-- Частичный
CREATE INDEX idx_orders_active ON orders(user_id)
WHERE status = 'active';

-- По выражению
CREATE INDEX idx_users_email_lower ON users(lower(email));

-- Покрывающий
CREATE INDEX idx_orders_cover ON orders(user_id) INCLUDE (status, amount);
```

`INCLUDE` не делает включённые колонки частью ключа сортировки/поиска. `Index Only Scan` возможен, только если индекс покрывает нужные данные **и** позволяет проверка видимости строк (visibility map).

## 5. Пример таблицы и составного индекса

```sql
CREATE TABLE orders (
    id BIGSERIAL PRIMARY KEY,
    user_id INT,
    status VARCHAR(20),
    amount NUMERIC,
    created_at TIMESTAMP
);

CREATE INDEX idx_orders_user_status_date
ON orders(user_id, status, created_at);
```

Порядок ключей:

```text
user_id → status → created_at
   1-й      2-й        3-й
```

Индекс упорядочен **сначала по `user_id`, внутри него — по `status`, внутри этой группы — по `created_at`**.

### Главное правило составного B-tree

Лучше всего индекс сужает поиск, когда запрос задаёт **левый префикс** ключей (особенно равенство по начальным колонкам). Пропуск промежуточной колонки обычно уменьшает эффективность поиска/сортировки.

## 6. Какие SELECT используют составной индекс

Предположим, есть только индекс:

```sql
CREATE INDEX idx_orders_user_status_date
ON orders(user_id, status, created_at);
```

| Условие запроса | Индекс поможет? | Почему |
|---|---|---|
| `user_id = 10` | ✅ Обычно хорошо | Первая колонка |
| `user_id = 10 AND status = 'paid'` | ✅ Обычно хорошо | Первые две колонки |
| `user_id = 10 AND status = 'paid' AND created_at > ...` | ✅ Обычно хорошо | Равенства + диапазон |
| `user_id = 10 AND created_at > ...` | ⚠️ Частично | Нет условия на `status` |
| `status = 'paid'` | ⚠️ Обычно слабо | Пропущена первая колонка |
| `created_at > ...` | ⚠️ Обычно слабо | Пропущены первые две колонки |

*⚠️ Не означает «индекс никогда не используется». Планировщик может выбрать полный/частичный проход по индексу, skip scan (PostgreSQL 18+, при подходящем распределении данных) или `Seq Scan`.*

**A. Поиск по первой колонке — хороший случай:**

```sql
SELECT * FROM orders WHERE user_id = 10;
```

**B. По первым двум колонкам — хороший случай:**

```sql
SELECT * FROM orders
WHERE user_id = 10 AND status = 'paid';
```

**C. Равенства + диапазон по третьей — хороший случай:**

```sql
SELECT * FROM orders
WHERE user_id = 10
  AND status = 'paid'
  AND created_at > '2026-01-01';
```

**D. Фильтрация + сортировка — хороший случай:**

```sql
SELECT * FROM orders
WHERE user_id = 10 AND status = 'paid'
ORDER BY created_at DESC
LIMIT 20;
```

B-tree может просканироваться в обратном направлении, чтобы удовлетворить `DESC` (без дополнительной сортировки).

**E. Пропущен `status` — частично:**

```sql
SELECT * FROM orders
WHERE user_id = 10
  AND created_at > '2026-01-01';
```

Индекс эффективно группирует по `user_id`, но `created_at` не образует единого подряд идущего диапазона среди разных `status`.

**F. Только `status` — обычно слабый случай:**

```sql
SELECT * FROM orders WHERE status = 'paid';
```

Если это частый и достаточно избирательный запрос, рассмотрим отдельный индекс:

```sql
CREATE INDEX idx_orders_status ON orders(status);
```

**Важно о диапазонах:**

```sql
SELECT * FROM orders
WHERE user_id > 10 AND status = 'paid';
```

Из-за диапазона по первому ключу второй ключ обычно не позволяет так эффективно сократить диапазон сканирования, как в случае `user_id = 10 AND status = 'paid'`.

## 7. Как PostgreSQL комбинирует несколько индексов

Можно создать отдельные и составной индексы:

```sql
CREATE INDEX idx_orders_user ON orders(user_id);
CREATE INDEX idx_orders_status ON orders(status);
CREATE INDEX idx_orders_user_status ON orders(user_id, status);
```

| Запрос | Возможный план |
|---|---|
| `WHERE user_id = 10` | `idx_orders_user` **или** составной |
| `WHERE status = 'paid'` | `idx_orders_status` |
| `WHERE user_id = 10 AND status = 'paid'` | Составной индекс **или** `BitmapAnd` двух отдельных |
| `WHERE user_id = 10 OR status = 'paid'` | `BitmapOr` по двум отдельным индексам |

**BitmapAnd / BitmapOr:** PostgreSQL может сначала получить наборы подходящих строк по разным индексам, затем пересечь (`AND`) либо объединить (`OR`) их и прочитать строки таблицы. Важно: bitmap-подход **обычно теряет порядок ключей**, поэтому для `ORDER BY` может потребоваться отдельная сортировка.

**Не создавай все три индекса автоматически:** каждый индекс занимает место и удорожает запись. Выбирай по реальным запросам и `EXPLAIN`.

## 8. Практические примеры других индексов

### GIN для JSONB

```sql
CREATE TABLE products (id BIGSERIAL PRIMARY KEY, metadata JSONB);
CREATE INDEX idx_products_metadata ON products USING GIN (metadata);

SELECT * FROM products
WHERE metadata @> '{"color":"red"}';
```

### GIN + pg_trgm для поиска подстроки

Обычный B-tree обычно не помогает с ведущим `%` в `LIKE '%term%'`.

```sql
CREATE EXTENSION IF NOT EXISTS pg_trgm;
CREATE INDEX idx_users_name_trgm
ON users USING GIN (name gin_trgm_ops);

SELECT * FROM users WHERE name ILIKE '%alex%';
```

### BRIN для большого журнала событий

```sql
CREATE INDEX idx_events_created_at
ON events USING BRIN (created_at);

SELECT * FROM events
WHERE created_at >= '2026-01-01'
  AND created_at < '2026-02-01';
```

BRIN особенно полезен, если новые события добавляются примерно по времени и физически лежат рядом. Он компактнее B-tree, но допускает чтение лишних блоков с последующей перепроверкой условий.

### Partial index

```sql
CREATE INDEX idx_orders_active_user
ON orders(user_id)
WHERE status = 'active';

SELECT * FROM orders
WHERE user_id = 10 AND status = 'active';
```

Partial index выгоден, когда условие запросов **подразумевает** условие индекса. Для произвольного `status` без этого условия он не подходит; с параметризованными запросами планировщик также не всегда может доказать соответствие.

### Expression index

```sql
CREATE INDEX idx_users_lower_email ON users(lower(email));

SELECT * FROM users WHERE lower(email) = 'a@example.com';
```

Обычный индекс на `email` не эквивалентен индексу на `lower(email)`.

### Covering index (`INCLUDE`)

```sql
CREATE INDEX idx_orders_user_cover
ON orders(user_id) INCLUDE (status, amount);

SELECT status, amount FROM orders WHERE user_id = 10;
```

Может обеспечить `Index Only Scan`, если visibility map позволяет избежать обращения к heap.

## 9. Как проверить индекс через EXPLAIN

```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT * FROM orders
WHERE user_id = 10 AND status = 'paid';
```

| Элемент плана | Что означает |
|---|---|
| `Seq Scan` | Чтение таблицы последовательно |
| `Index Scan` | Чтение подходящих строк с помощью индекса и таблицы |
| `Index Only Scan` | Данные берутся из индекса, но иногда нужны heap fetches |
| `Bitmap Index Scan` | Индекс создаёт bitmap подходящих строк |
| `Bitmap Heap Scan` | По bitmap читаются блоки таблицы |
| `BitmapAnd` / `BitmapOr` | Комбинация наборов строк от индексов |
| `Sort` | Дополнительная сортировка |

Смотри на `actual time`, `rows`, `loops`, `Buffers`, `Index Cond`, `Filter`, `Rows Removed by Filter`. Важно отличать **условие поиска в индексе** (`Index Cond`) от **фильтра после извлечения** (`Filter`).

> `EXPLAIN ANALYZE` действительно выполняет запрос. Для `UPDATE`/`DELETE` используй тестовые данные или транзакцию с `ROLLBACK` (побочные эффекты вроде вызовов внешних функций не всегда откатываются).

## 10. Правила проектирования и типичные ошибки

1. **Начинай с B-tree** для обычных `WHERE`, диапазонов и `ORDER BY`.
2. **Порядок колонок важен**: для частых составных запросов сначала обычно поля равенства, затем диапазон/сортировка; проверяй реальные планы.
3. **Один составной индекс не заменяет все одиночные**: `(user_id, status)` плохо подходит для частого `WHERE status = ...`.
4. **Не дублируй индексы без причины**: `(user_id, status)` часто обслуживает и `WHERE user_id = ...`; отдельный `(user_id)` может оказаться лишним, но это зависит от нагрузки.
5. **GIN** выбирай для `jsonb`, массивов, полнотекстового поиска; **BRIN** — для больших физически коррелированных диапазонов.
6. **Partial** особенно полезен для небольшой активно используемой доли таблицы.
7. **Expression** нужен, если фильтруешь по выражению (`lower(email)`), а не по самому полю.
8. **`LIKE '%text%'`**: обычный B-tree обычно не помогает; смотри `pg_trgm`.
9. **Индекс не всегда быстрее**: на маленькой таблице либо при выборке большой доли строк `Seq Scan` может быть оптимальным.
10. **На большой рабочей таблице** для снижения блокирования DML используй `CREATE INDEX CONCURRENTLY` (нельзя запускать внутри обычного transaction block; есть дополнительные требования и компромиссы).
11. **Проверяй `EXPLAIN (ANALYZE, BUFFERS)`**: окончательное решение принимает планировщик.

**Индексы и ограничения:** `PRIMARY KEY` и `UNIQUE` автоматически создают уникальные индексы; не дублируйте их. `FOREIGN KEY` не создаёт индекс на ссылающихся колонках. Например, при удалении пользователя проверка связанных `orders(user_id)` может сканировать всю таблицу заказов. Индекс оценивайте по размеру таблицы и реальным операциям.

**CONCURRENTLY не означает «без ожиданий»:** построение выполняет дополнительные проходы, создаёт нагрузку и может ждать транзакции. После ошибки может остаться `INVALID`-индекс: он не используется для поиска, но сохраняет накладные расходы на запись. Проверьте `pg_index.indisvalid`; после устранения причины индекс нужно пересоздать или перестроить. Неудачный уникальный индекс в некоторых случаях продолжает проверять уникальность.

### Конвенция именования индексов

Это соглашение проекта, а не требование PostgreSQL.

- Обычный индекс: `IDX_<table>__<column_1>-<column_2>`.
- Уникальный индекс: `UIDX_<table>__<column_1>-<column_2>`.
- Таблица и колонки — в `snake_case`, как в БД.
- `__` отделяет таблицу, `-` разделяет колонки в порядке их объявления в индексе.
- Для одной колонки: `IDX_orders__user_id`.
- Для partial index допустимо понятное имя назначения: `UIDX_orders__one_active_per_user`.
- Укладывайте имя в 63 байта (стандартный лимит PostgreSQL); длинные части сокращайте осмысленно, сохраняя уникальность имени в схеме.

В SQL такие имена заключайте в двойные кавычки:

```sql
CREATE INDEX "IDX_orders__user_id-created_at"
ON orders (user_id, created_at);

CREATE UNIQUE INDEX "UIDX_users__email"
ON users (email);

CREATE UNIQUE INDEX "UIDX_orders__one_active_per_user"
ON orders (user_id) WHERE status = 'active';
```

## 11. Big O: сложность алгоритмов

**Big O** показывает, как растёт объём работы с ростом количества элементов `N` (не точное время выполнения).

| Сложность | Название | Пример | Для `N = 1000` (условно) |
|---|---|---|---:|
| `O(1)` | Константная | Доступ к элементу массива по индексу | 1 |
| `O(log N)` | Логарифмическая | Бинарный поиск, B-tree | ≈ 10 |
| `O(N)` | Линейная | Перебор массива | 1 000 |
| `O(N log N)` | Линейно-логарифмическая | Merge sort | ≈ 10 000 |
| `O(N²)` | Квадратичная | Два вложенных цикла | 1 000 000 |
| `O(2^N)` | Экспоненциальная | Полный перебор подмножеств | ≈ 10³⁰¹ |

Запомнить:

- `O(1)` — рост `N` почти не меняет объём работы.
- `O(log N)` — удвоение `N` добавляет примерно один шаг (при логарифме по основанию 2).
- `O(N)` — удвоение `N` удваивает работу.
- `O(N²)` — удвоение `N` увеличивает работу примерно вчетверо.

---

### Документация PostgreSQL

- [Indexes](https://www.postgresql.org/docs/current/indexes.html)
- [Index Types](https://www.postgresql.org/docs/current/indexes-types.html)
- [Multicolumn Indexes](https://www.postgresql.org/docs/current/indexes-multicolumn.html)
- [Combining Multiple Indexes](https://www.postgresql.org/docs/current/indexes-bitmap-scans.html)
- [Partial Indexes](https://www.postgresql.org/docs/current/indexes-partial.html)
- [Index-Only Scans](https://www.postgresql.org/docs/current/indexes-index-only-scans.html)
- [Using EXPLAIN](https://www.postgresql.org/docs/current/using-explain.html)
- [Constraints](https://www.postgresql.org/docs/current/ddl-constraints.html)
- [CREATE INDEX: Building Indexes Concurrently](https://www.postgresql.org/docs/current/sql-createindex.html#SQL-CREATEINDEX-CONCURRENTLY)
- [PostgreSQL 18: skip scan](https://www.postgresql.org/docs/18/release-18.html)
