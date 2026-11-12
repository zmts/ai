# PostgreSQL Locks — Cheatsheet

См. также: [транзакции — границы, ошибки и savepoints](postgresql-transactions-cheatsheet.md).

## Содержание

1. [Что такое блокировка и зачем она нужна](#1-что-такое-блокировка-и-зачем-она-нужна)
2. [Optimistic vs Pessimistic Locking](#2-optimistic-vs-pessimistic-locking)
3. [MVCC и уровни изоляции](#3-mvcc-и-уровни-изоляции)
4. [Классификация блокировок](#4-классификация-блокировок)
5. [Row-level locks: FOR UPDATE / SHARE](#5-row-level-locks-for-update--share)
6. [NOWAIT и SKIP LOCKED](#6-nowait-и-skip-locked)
7. [Table-level locks](#7-table-level-locks)
8. [Advisory locks](#8-advisory-locks)
9. [Практические кейсы и выбор подхода](#9-практические-кейсы-и-выбор-подхода)
10. [Deadlocks, ожидания и retry](#10-deadlocks-ожидания-и-retry)
11. [Диагностика блокировок](#11-диагностика-блокировок)

---

## 1. Что такое блокировка и зачем она нужна

**Lock** — механизм координации параллельных транзакций, чтобы они не выполняли несовместимые операции над одними и теми же данными.

**Пример:** два HTTP-запроса одновременно пытаются списать деньги с одного счёта. Без корректной конкурентной логики можно получить некорректный баланс или потерю обновления.

- **Lock ≠ транзакция:** транзакция объединяет операции и задаёт границы фиксации; lock защищает конфликтующие действия.
- **Lock ≠ индекс:** индекс ускоряет поиск, но сам по себе не решает проблему конкурентного изменения данных.
- PostgreSQL автоматически ставит нужные блокировки для `INSERT`, `UPDATE`, `DELETE` и DDL. Явные блокировки требуются не всегда.

---

## 2. Optimistic vs Pessimistic Locking

Это **две стратегии**, а не названия SQL lock modes.

| Критерий | Optimistic | Pessimistic |
|---|---|---|
| Идея | Конфликты редки: проверяем при записи | Конфликты вероятны: заранее блокируем |
| Чтение | Без удержания блокировки | Обычно `SELECT ... FOR UPDATE` |
| Обнаружение конфликта | По `version` / условию `UPDATE` | При попытке взять несовместимую блокировку |
| Поведение при конфликте | `0 rows` → retry / 409 Conflict | Ожидание / `NOWAIT` / timeout |
| Подходит | Профиль, настройки, формы | Балансы, резервирование, обработка очереди |
| Риски | Конфликты при частых записях | Ожидания, deadlocks, длинные транзакции |

### Optimistic: проверка версии

```sql
-- 1. Клиент прочитал запись с version = 5
SELECT id, balance, version
FROM accounts
WHERE id = 1;

-- 2. Изменяем только если версия прежняя
UPDATE accounts
SET balance = 900,
    version = version + 1
WHERE id = 1
  AND version = 5
RETURNING id, balance, version;
```

- **Вернулась строка** → изменение прошло.
- **0 строк** → версия изменилась **или запись отсутствует**; нужно обработать конфликт, перечитать данные или повторить операцию по правилам бизнеса.
- Даже optimistic-`UPDATE` во время исполнения получает обычные внутренние блокировки PostgreSQL; «optimistic» означает **нет заранее удерживаемого lock между чтением и записью**.

### Pessimistic: блокируем перед read-modify-write

```sql
BEGIN;

SELECT balance
FROM accounts
WHERE id = 1
FOR UPDATE;

-- Проверяем бизнес-условия, рассчитываем новое значение
UPDATE accounts
SET balance = balance - 100
WHERE id = 1;

COMMIT;
```

- Конфликтующая транзакция будет ждать освобождения строки.
- Блокировка держится **до конца транзакции** (`COMMIT` / `ROLLBACK`).
- Обычный `SELECT` без locking clause чаще всего продолжит читать доступную версию строки благодаря MVCC.

---

## 3. MVCC и уровни изоляции

**MVCC (Multi-Version Concurrency Control)** — PostgreSQL хранит версии строк и использует snapshots. Благодаря этому обычный `SELECT` не обязан ждать конкурентный `UPDATE` и наоборот.

| Isolation level | В PostgreSQL | Для чего |
|---|---|---|
| `READ UNCOMMITTED` | Реализован как `READ COMMITTED` | Отдельного более слабого режима нет |
| **`READ COMMITTED`** | Snapshot на каждый statement; default | Типичный CRUD |
| **`REPEATABLE READ`** | Snapshot транзакции; в PG нет phantom reads | Согласованные чтения |
| **`SERIALIZABLE`** | SSI: проверяет, возможно ли сериализовать выполнение | Сложные инварианты между несколькими строками |

```sql
BEGIN ISOLATION LEVEL SERIALIZABLE;
-- связанные чтения и записи
COMMIT;
```

**Важно:** при `SERIALIZABLE` возможна ошибка `40001` (`serialization_failure`). Приложение должно уметь **повторить всю транзакцию**, а не только последний SQL-запрос. PostgreSQL использует для SSI также `SIReadLock` (predicate locking); он не блокирует обычную запись как `FOR UPDATE`.

---

## 4. Классификация блокировок

| Измерение | Варианты | Что означает |
|---|---|---|
| Стратегия | Optimistic / Pessimistic | Проверяем конфликт при записи или блокируем заранее |
| Совместимость | Shared / Exclusive | Могут ли другие транзакции брать конфликтующие блокировки |
| Объект | Row / Table / Advisory | Строка, таблица, пользовательский логический ключ |
| Время жизни | Transaction / Session | До конца транзакции или сессии |
| Поведение при занятости | Wait / `NOWAIT` / `SKIP LOCKED` | Ждать, ошибиться или пропустить занятые строки |
| Дополнительно | Predicate / `SIReadLock` | Отслеживание конфликтов сериализации при SSI |

> В PostgreSQL нет универсальных режимов с буквальными именами `S`/`X` на все случаи: у row-level и table-level свои **конкретные режимы и матрицы совместимости**.

---

## 5. Row-level locks: FOR UPDATE / SHARE

| Режим | Для чего | Существенная особенность |
|---|---|---|
| **`FOR UPDATE`** | Изменяем / удаляем выбранную строку | Самый строгий row lock из четырёх |
| **`FOR NO KEY UPDATE`** | Обновляем без изменения ключевых полей | Слабее `FOR UPDATE`; позволяет `FOR KEY SHARE` |
| **`FOR SHARE`** | Стабилизируем строки для дальнейшей логики | Несколько `FOR SHARE` совместимы |
| **`FOR KEY SHARE`** | Защищаем ключ от удаления/изменения | Не мешает `FOR NO KEY UPDATE` |

```sql
BEGIN;
SELECT * FROM orders WHERE id = 42 FOR UPDATE;
-- логика и изменение заказа
UPDATE orders SET status = 'paid' WHERE id = 42;
COMMIT;
```

**Важные нюансы:**

- `FOR UPDATE` — не то же самое, что «запретить любые чтения»: обычный `SELECT` может читать старую видимую версию.
- `UPDATE` автоматически берёт row lock: обычно `FOR NO KEY UPDATE`, а при изменении определённых уникальных ключевых столбцов — более сильный `FOR UPDATE`.
- При `SELECT ... FOR UPDATE` на множестве строк вы увеличиваете число конфликтующих объектов. Сужайте `WHERE` и держите транзакцию короткой.
- **Отсутствующую строку `FOR UPDATE` не блокирует.** Если проверка «записи ещё нет» вернула пустой результат, конкурентный `INSERT` остаётся возможным. Для уникальности используйте `UNIQUE`; для более сложных инвариантов — подходящую синхронизацию или `SERIALIZABLE` с retry.

### Упрощённая совместимость row locks

`✓` — совместимы, `✕` — конфликтуют (для разных транзакций на одной строке).

| Запрошено ↓ / Уже удерживается → | KEY SHARE | SHARE | NO KEY UPDATE | UPDATE |
|---|:---:|:---:|:---:|:---:|
| **KEY SHARE** | ✓ | ✓ | ✓ | ✕ |
| **SHARE** | ✓ | ✓ | ✕ | ✕ |
| **NO KEY UPDATE** | ✓ | ✕ | ✕ | ✕ |
| **UPDATE** | ✕ | ✕ | ✕ | ✕ |

---

## 6. NOWAIT и SKIP LOCKED

| Вариант | Что происходит | Когда использовать |
|---|---|---|
| Обычный `FOR UPDATE` | Ждёт, пока lock освободится | Надо обязательно обработать конкретную строку |
| `FOR UPDATE NOWAIT` | Сразу ошибка `55P03`, если строка занята | Fail-fast API / попытка захвата ресурса |
| `FOR UPDATE SKIP LOCKED` | Не возвращает строки, занятые другими | Многопоточная очередь заданий |

```sql
-- Не ждём занятую строку
SELECT * FROM jobs
WHERE id = 5
FOR UPDATE NOWAIT;

-- Разные workers забирают разные задания
SELECT * FROM jobs
WHERE status = 'pending'
ORDER BY id
LIMIT 10
FOR UPDATE SKIP LOCKED;
```

**Нюансы:** `SKIP LOCKED` даёт **неполную/несогласованную выборку**, поэтому обычно подходит для очередей, а не для обычной отчётности. `NOWAIT` и `SKIP LOCKED` касаются **row-level** блокировок; требуемый table-level `ROW SHARE` lock всё равно берётся обычным образом.

### Атомарный захват заданий

```sql
WITH picked AS (
  SELECT id
  FROM jobs
  WHERE status = 'pending'
  ORDER BY id
  LIMIT 10
  FOR UPDATE SKIP LOCKED
)
UPDATE jobs AS j
SET status = 'processing'
FROM picked
WHERE j.id = picked.id
RETURNING j.*;
```

Делайте это внутри короткой транзакции, затем фиксируйте. Если обработка длительная, храните lease/timeout и механизм повторной выдачи зависших заданий, а не держите транзакцию открытой всё время работы worker.

---

## 7. Table-level locks

**Table locks** ставятся как явно (`LOCK TABLE`), так и автоматически SQL-операциями. Название `ROW EXCLUSIVE` **не означает** блокировку одной строки — это именно режим блокировки **таблицы**.

| Режим | Типичный источник | На что обратить внимание |
|---|---|---|
| `ACCESS SHARE` | Обычный `SELECT` | Конфликтует с `ACCESS EXCLUSIVE` |
| `ROW SHARE` | `SELECT ... FOR UPDATE/SHARE` | Добавляется к row locks |
| `ROW EXCLUSIVE` | `INSERT`, `UPDATE`, `DELETE` | Не блокирует обычный `SELECT` |
| `SHARE UPDATE EXCLUSIVE` | `VACUUM` (без FULL), часть DDL | Ограничивает некоторые конкурирующие служебные операции |
| `SHARE` | Обычный `CREATE INDEX` | Конфликтует с модифицирующими транзакциями |
| `SHARE ROW EXCLUSIVE` | Некоторые операции DDL | Сильный самоконфликтующий lock |
| `EXCLUSIVE` | Некоторые явные операции | Позволяет обычный `SELECT`, но блокирует многие другие режимы |
| `ACCESS EXCLUSIVE` | `TRUNCATE`, `DROP TABLE`, часть `ALTER TABLE` | Блокирует даже обычный `SELECT` |

```sql
BEGIN;
LOCK TABLE orders IN SHARE MODE;
-- действия, требующие данного режима
COMMIT;
```

**Обычный `CREATE INDEX` может мешать записям;** `CREATE INDEX CONCURRENTLY` обычно позволяет параллельные `INSERT/UPDATE/DELETE`, но имеет ограничения и исполняется **вне transaction block**.

---

## 8. Advisory locks

**Advisory lock** — логическая блокировка по числовому ключу. PostgreSQL не связывает её автоматически с конкретной строкой: **все участники должны добровольно соблюдать один протокол ключей**.

| API | Срок жизни | Применение |
|---|---|---|
| `pg_advisory_xact_lock(key)` | До `COMMIT`/`ROLLBACK` | Защита операции в транзакции |
| `pg_try_advisory_xact_lock(key)` | До конца транзакции | Попытаться, не ожидая |
| `pg_advisory_lock(key)` | До освобождения/конца **сессии** | Долгий логический mutex |
| `pg_try_advisory_lock(key)` | До освобождения/конца сессии | Неблокирующая попытка |
| `pg_advisory_unlock(key)` | Явное освобождение | Только для session-scoped lock |

```sql
BEGIN;
SELECT pg_advisory_xact_lock(1001);
-- критическая секция
COMMIT; -- освободит автоматически
```

```sql
-- Session-scoped (в connection pool требует осторожности)
SELECT pg_advisory_lock(1001);
-- операции
SELECT pg_advisory_unlock(1001);
```

- Есть **shared**-варианты advisory locks; ключ можно задавать одним `bigint` или парой `int`.
- Transaction-scoped вариант обычно безопаснее в backend-коде с пулом соединений.
- Session-scoped lock живёт на **физическом соединении**, не на HTTP-запросе. Возвращая соединение в пул, можно случайно оставить lock удерживаемым.
- Advisory locks не гарантируют «один процесс навсегда»: сбой соединения освободит lock. Для надёжных фоновых задач также полезны lease, idempotency и проверки состояния.

---

## 9. Практические кейсы и выбор подхода

| Задача | Рекомендуемый подход | Почему |
|---|---|---|
| Редактирование профиля двумя клиентами | Optimistic `version` | Не держим транзакцию между чтением и сохранением формы |
| Простое списание баланса | **Один атомарный `UPDATE`** | Часто проще и безопаснее предварительного `SELECT` |
| Сложный read-modify-write с бизнес-условиями | `SELECT ... FOR UPDATE` | Стабилизируем конкретные строки на время проверки |
| Очередь SQL jobs с несколькими workers | `FOR UPDATE SKIP LOCKED` | Workers не ждут одни и те же задания |
| Одна логическая операция на ресурс | Advisory transaction lock | Ключ может не соответствовать одной строке |
| Проверка инварианта между несколькими строками | `SERIALIZABLE` + retry (либо аккуратная явная синхронизация) | Защита от write skew и других аномалий |
| Только аналитическое чтение | MVCC / подходящий isolation level | Обычно не нужен `FOR UPDATE` |

### Пример: атомарное списание денег (обычно лучше двух запросов)

```sql
UPDATE accounts
SET balance = balance - 100
WHERE id = 1
  AND balance >= 100
RETURNING id, balance;
```

- **Строка вернулась** → списание выполнено.
- **0 строк** → недостаточно средств **или счёт не найден**.
- PostgreSQL самостоятельно берёт нужные блокировки для `UPDATE`.
- Если нужно **перевести** деньги между двумя счетами, объединяйте списание и зачисление **в одну транзакцию**; сохраняйте одинаковый порядок доступа к счетам, чтобы уменьшать вероятность deadlock.

### Пример: optimistic-locking в API

```sql
UPDATE documents
SET content = $1,
    version = version + 1
WHERE id = $2
  AND version = $3
RETURNING id, version;
```

Проверьте `rowCount`. Если `0`, верните конфликт (часто HTTP `409`) и предложите перечитать запись, а не молча перезаписывайте чужие правки.

### Пример: блокировка нескольких счетов

```sql
BEGIN;

-- Блокируем в стабильном порядке по id
SELECT id, balance
FROM accounts
WHERE id IN (10, 20)
ORDER BY id
FOR UPDATE;

-- Проверяем остаток, обновляем оба счета
COMMIT;
```

**Не держите** транзакцию открытой во время сетевых вызовов к платёжному API: задержки создают очередь ожиданий и повышают риск deadlock/timeouts.

---

## 10. Deadlocks, ожидания и retry

**Deadlock** — циклическое ожидание:

```text
TX1: взяла lock(A) → ждёт lock(B)
TX2: взяла lock(B) → ждёт lock(A)
```

PostgreSQL обнаруживает deadlock и завершает одну транзакцию с SQLSTATE **`40P01`**.

**Профилактика:**

1. Захватывать ресурсы всегда в одинаковом порядке (`id ASC`).
2. Делать транзакции короткими: без внешних HTTP-запросов внутри.
3. Обновлять/блокировать только необходимые строки; правильно индексировать `WHERE`.
4. Ограничивать ожидания (`lock_timeout`) при необходимости.
5. Повторять **всю транзакцию** на `40P01` и `40001` с ограничением попыток и backoff.

```sql
-- Пример: ограничение ожидания блокировки для транзакции
BEGIN;
SET LOCAL lock_timeout = '2s';
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;
COMMIT;
```

| SQLSTATE | Значение | Что делать |
|---|---|---|
| `40P01` | `deadlock_detected` | Retry всей транзакции |
| `40001` | `serialization_failure` | Retry всей транзакции |
| `55P03` | `lock_not_available` (`NOWAIT`/`lock_timeout`) | Отдать busy, повторить позже по правилам API |

---

## 11. Диагностика блокировок

### Кто сейчас держит и ждёт locks

```sql
SELECT
    pid,
    locktype,
    relation::regclass AS relation,
    mode,
    granted
FROM pg_locks
WHERE relation = 'orders'::regclass;
```

`granted = false` означает, что lock **ожидается**.

Фильтр по `relation` не показывает все ожидания строк: они часто представлены ожиданием `transactionid`, без ссылки на таблицу. Для поиска блокирующего процесса используйте следующий запрос с `pg_blocking_pids()`.

### Кто блокирует конкретный процесс

```sql
SELECT
    pid,
    state,
    wait_event_type,
    wait_event,
    pg_blocking_pids(pid) AS blocked_by,
    query
FROM pg_stat_activity
WHERE wait_event_type = 'Lock';
```

### Быстрые настройки

```sql
SHOW lock_timeout;
SHOW deadlock_timeout;
SHOW statement_timeout;
```

- `lock_timeout` — сколько ждать **получения lock**.
- `statement_timeout` — лимит на **весь statement**.
- `deadlock_timeout` — задержка перед проверкой deadlock (обычно настраивается на уровне эксплуатации БД).

**`idle in transaction` тоже может удерживать блокировки.** Проверьте PID блокировщика в `pg_stat_activity`, его `state` и `xact_start`. `statement_timeout` не ограничивает простой между запросами; для этого существует `idle_in_transaction_session_timeout`, завершающий сессию. После SQL-ошибки внутри явной транзакции выполните `ROLLBACK` перед новой попыткой.

---

### Официальная документация

- [Explicit Locking — PostgreSQL](https://www.postgresql.org/docs/current/explicit-locking.html)
- [Transaction Isolation — PostgreSQL](https://www.postgresql.org/docs/current/transaction-iso.html)
- [MVCC Introduction — PostgreSQL](https://www.postgresql.org/docs/current/mvcc-intro.html)
- [Advisory Lock Functions — PostgreSQL](https://www.postgresql.org/docs/current/functions-admin.html#FUNCTIONS-ADVISORY-LOCKS)
- [Monitoring Database Activity — PostgreSQL](https://www.postgresql.org/docs/current/monitoring-stats.html)
- [pg_locks — PostgreSQL](https://www.postgresql.org/docs/current/view-pg-locks.html)
- [Client Connection Defaults — PostgreSQL](https://www.postgresql.org/docs/current/runtime-config-client.html)

> Примечание: SQL-примеры иллюстративные. Конкретный выбор режима, ожидание и поведение под нагрузкой проверяйте в транзакциях на вашей версии PostgreSQL.
