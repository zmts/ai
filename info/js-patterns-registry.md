# Реестр паттернов

Паттерны этого реестра применимы и к JavaScript, и к TypeScript.
TypeScript позволяет дополнительно выразить их контракты через interface,
generics, readonly и discriminated unions; принципы работы объектов,
структур данных, ресурсов и ошибок остаются теми же.

При использовании примеров в TypeScript учитывайте:

- Параметрам, полям и результатам могут потребоваться явные типы;
  коллекциям и общим алгоритмам — generics.
- Статические типы не заменяют runtime-валидацию внешних данных.
- readonly ограничивает изменение через типизированный доступ, но не
  обеспечивает глубокую неизменяемость объектов во время выполнения.
- Рекомендации по производительности относятся к исполняемому JavaScript
  и зависят от целевого runtime и нагрузки.

## Оглавление

- [1. Создание объектов и область состояния](#1-создание-объектов-и-область-состояния)
  - [Factory](#factory)
  - [Factory Method / выбор типа](#factory-method--выбор-типа)
  - [Abstract Factory](#abstract-factory)
  - [Builder](#builder)
  - [Prototype](#prototype)
  - [Singleton](#singleton)
  - [Multiton](#multiton)
  - [Context](#context)
- [2. Границы интерфейсов, композиция и управление поведением](#2-границы-интерфейсов-композиция-и-управление-поведением)
  - [Adapter](#adapter)
  - [Wrapper](#wrapper)
  - [Decorator](#decorator)
  - [Chain of Responsibility](#chain-of-responsibility)
  - [Proxy](#proxy)
  - [Bridge](#bridge)
  - [Strategy](#strategy)
  - [Composite](#composite)
  - [Facade](#facade)
  - [Mediator](#mediator)
  - [Boxing / Value Object](#boxing--value-object)
- [3. Разделение памяти и управление ресурсами](#3-разделение-памяти-и-управление-ресурсами)
  - [Flyweight](#flyweight)
  - [Object Pool](#object-pool)
  - [Resource Pool / Lease](#resource-pool--lease)
  - [Ограничение удерживаемой памяти](#ограничение-удерживаемой-памяти)
  - [Явный lifecycle подписки](#явный-lifecycle-подписки)
- [4. Представление данных и выбор коллекции](#4-представление-данных-и-выбор-коллекции)
  - [Object как запись](#object-как-запись)
  - [Map для динамических ключей](#map-для-динамических-ключей)
  - [Set для уникальности / membership](#set-для-уникальности--membership)
  - [Array для последовательности](#array-для-последовательности)
  - [WeakMap / WeakSet](#weakmap--weakset)
  - [TypedArray / ArrayBuffer](#typedarray--arraybuffer)
  - [DataView](#dataview)
  - [Lookup table / registry dispatch](#lookup-table--registry-dispatch)
  - [Нормализация aliases на границе](#нормализация-aliases-на-границе)
  - [Структура по смыслу операции](#структура-по-смыслу-операции)
  - [Snapshot / live view](#snapshot--live-view)
  - [Безопасный словарь](#безопасный-словарь)
- [5. Структуры данных, поиск, обход и распределённое состояние](#5-структуры-данных-поиск-обход-и-распределённое-состояние)
  - [Singly Linked List](#singly-linked-list)
  - [Doubly Linked List](#doubly-linked-list)
  - [Immutable Cons List](#immutable-cons-list)
  - [Circular Buffer](#circular-buffer)
  - [Queue / FIFO](#queue--fifo)
  - [Deque](#deque)
  - [Stack / LIFO](#stack--lifo)
  - [Unrolled List](#unrolled-list)
  - [Binary Search Tree](#binary-search-tree)
  - [B-tree](#b-tree)
  - [Heap / Priority Queue](#heap--priority-queue)
  - [Trie](#trie)
  - [Graph / adjacency list + BFS](#graph--adjacency-list--bfs)
  - [Iterator](#iterator)
  - [LRU Cache](#lru-cache)
  - [Struct / запись со схемой](#struct--запись-со-схемой)
  - [Prototype overlay / branch](#prototype-overlay--branch)
  - [CRDT G-Counter](#crdt-g-counter)
- [6. Ошибки и восстановление](#6-ошибки-и-восстановление)
  - [Классификация ошибок](#классификация-ошибок)
  - [Перевод ошибки на границе](#перевод-ошибки-на-границе)
  - [Propagation / владелец обработки](#propagation--владелец-обработки)
  - [Retry + exponential backoff](#retry--exponential-backoff)
  - [Process-level failure handling](#process-level-failure-handling)
  - [Graceful Shutdown](#graceful-shutdown)
- [7. Организация кода и владение данными](#7-организация-кода-и-владение-данными)
  - [Data first](#data-first)
  - [Named steps](#named-steps)
  - [Split then compose](#split-then-compose)
  - [Локализация семантической сложности](#локализация-семантической-сложности)
  - [Владение входами](#владение-входами)
  - [Сохранение порядка эффектов](#сохранение-порядка-эффектов)
  - [Composition over inheritance](#composition-over-inheritance)
  - [Явные зависимости / IoC и DI](#явные-зависимости--ioc-и-di)
  - [Разделение domain/system code](#разделение-domainsystem-code)
  - [Контракт важнее механического reuse](#контракт-важнее-механического-reuse)
- [8. Производительность и измерение](#8-производительность-и-измерение)
  - [Stable object shape](#stable-object-shape)
  - [Dense arrays / согласованный тип элементов](#dense-arrays--согласованный-тип-элементов)
  - [Monomorphic hot paths](#monomorphic-hot-paths)
  - [Снижение escaping allocations](#снижение-escaping-allocations)
  - [Измерение до оптимизации](#измерение-до-оптимизации)
  - [Изоляция микробенчмарков](#изоляция-микробенчмарков)

Примеры показывают конкретное решение и конкретную ошибку. Имена `client`,
`cache`, `logger`, `signal` и подобные обозначают зависимости приложения.
Каждый блок программного примера запускается отдельно от соседнего.
Примеры рассчитаны на современный Node.js с ES modules и top-level await.
Отдельные модели lifecycle и структур намеренно показывают только
обсуждаемый контракт; их ограничения указаны в пояснении. «Плохой» означает
проблему в указанном контексте, а не безусловный запрет конструкции JavaScript.
Производительность зависит от нагрузки и runtime.

Strategy, Iterator, Chain of Responsibility, Mediator, Multiton и B-tree
в исходном материале не имеют собственной полноценной карточки.
Их примеры поясняют упоминание и не обозначают наличие соответствующей
реализации в Metaskills.

## 1. Создание объектов и область состояния

### Factory

Централизовать defaults и создание объекта.

#### Когда применять

Объекты одного назначения создаются в нескольких местах, а defaults, проверка входов или состав полей должны оставаться согласованными.

#### Когда избыточен

Есть одно очевидное место создания и стабильный конструктор; отдельная factory только повторяет его параметры.

#### Более простое решение

Создать объект литералом или вызвать существующий конструктор напрямую. Небольшую функцию создания выделить при появлении повторяющихся правил.

#### Хороший пример

```js
const createUser = (name, role = 'reader') => ({ name, role });
const canRead = (user) => ['reader', 'admin'].includes(user.role);

const fromRegistration = createUser('Anna');
const fromImport = createUser('Boris');
console.log(canRead(fromRegistration), canRead(fromImport)); // true true
```

#### Плохой пример

```js
const canRead = (user) => ['reader', 'admin'].includes(user.role);
const fromRegistration = { name: 'Anna', role: 'reader' };
const fromImport = { name: 'Boris' };

console.log(canRead(fromRegistration), canRead(fromImport)); // true false
```

#### В чём разница

Factory задаёт defaults в одном месте. Во втором варианте путь импорта забыл role: одинаковые по смыслу пользователи получают разное поведение. Для единственного стабильного места создания отдельная factory может не понадобиться.

### Factory Method / выбор типа

Локализовать выбор конкретного продукта.

#### Когда применять

В существующем общем workflow нужно менять создаваемый продукт, сохраняя остальные шаги. Переопределяемый метод подходит, если расширение через подтипы уже является частью модели.

#### Когда избыточен

Различается только простой выбор класса по ключу, а общего полиморфного workflow нет. Иерархия creators добавит больше кода, чем сама операция создания.

#### Более простое решение

Использовать функцию с небольшим условием, registry конструкторов или передать функцию создания в готовый workflow.

#### Хороший пример

```js
class JsonReport {
  render(rows) { return JSON.stringify(rows); }
}
class ReportJob {
  createReport() { throw new Error('Override createReport'); }
  run(rows) { return this.createReport().render(rows); }
}
class JsonReportJob extends ReportJob {
  createReport() { return new JsonReport(); }
}
console.log(new JsonReportJob().run([{ count: 2 }])); // [{"count":2}]
```

#### Плохой пример

```js
const exportReport = (kind, rows) => {
  if (kind === 'json') return JSON.stringify(rows);
  if (kind === 'csv') return rows.map((row) => row.count).join('\n');
  throw new Error('Unknown format');
};
const previewReport = (kind, rows) => {
  if (kind === 'json') return JSON.stringify(rows);
  throw new Error('Unknown format');
};
console.log(exportReport('csv', [{ count: 2 }])); // 2
// previewReport('csv', [{ count: 2 }]); // Unknown format
```

#### В чём разница

В хорошем варианте общий workflow run не знает конкретный продукт: его создаёт переопределяемый метод. В плохом выбор формата продублирован, поэтому export поддерживает CSV, а preview — нет. Наследование оправдано при существующем workflow; для простого выбора достаточно registry.

### Abstract Factory

Создавать согласованное семейство продуктов.

#### Когда применять

Конфигурация переключает целое семейство совместимых продуктов: например, store, cursor и transaction для одного backend.

#### Когда избыточен

Есть одна реализация семейства либо меняется только один продукт; согласовывать несколько совместимых ролей не требуется.

#### Более простое решение

Передать конкретную зависимость напрямую или использовать одну factory для изменяемого продукта.

#### Хороший пример

```js
const createFamily = (backend) => ({
  createStore: () => ({ backend }),
  createCursor(store) {
    if (store.backend !== backend) throw new Error('Incompatible store');
    return { backend, store };
  },
});
const families = { sql: createFamily('sql'), file: createFamily('file') };
const family = families.sql;
const store = family.createStore();
console.log(family.createCursor(store).backend); // sql
```

#### Плохой пример

```js
const createSqlStore = () => ({ backend: 'sql' });
const createFileCursor = (store) => {
  if (store.backend !== 'file') throw new Error('Expected file store');
  return { store };
};
const store = createSqlStore();
// createFileCursor(store); // Expected file store
```

#### В чём разница

Один выбор family согласует все продукты. Независимый выбор store и cursor допускает несовместимую комбинацию; ошибка проявится при сборке или использовании. Если семейство одно, обычная factory проще.

### Builder

Собирать сложный продукт по шагам.

#### Когда применять

Продукт собирается постепенно из множества необязательных частей, порядок шагов имеет смысл или финальная сборка должна проверять общий инвариант.

#### Когда избыточен

Все параметры известны сразу и удобно помещаются в один options object. Цепочка методов лишь переименовывает поля этого объекта.

#### Более простое решение

Передать options в функцию или конструктор и проверить итоговую конфигурацию в одном месте.

#### Хороший пример

```js
class QueryBuilder {
  constructor() { this.filters = []; }
  where(field, value) {
    this.filters.push(Object.freeze({ field, value }));
    return this;
  }
  build() { return { filters: [...this.filters] }; }
}
const builder = new QueryBuilder().where('active', true);
const query = builder.build();
builder.where('role', 'admin');
console.log(query.filters.length); // 1
```

#### Плохой пример

```js
class QueryBuilder {
  constructor() { this.filters = []; }
  where(field, value) {
    this.filters.push({ field, value });
    return this;
  }
  build() { return { filters: this.filters }; }
}
const builder = new QueryBuilder().where('active', true);
const query = builder.build();
builder.where('role', 'admin');
console.log(query.filters.length); // 2: готовый query изменился
```

#### В чём разница

build в первом примере выдаёт отдельный список immutable условий. Во втором готовый query разделяет изменяемый массив builder. Копирование здесь поверхностное: контракт примера предполагает скалярные значения условий.

### Prototype

Клонировать подготовленный объект с нужной семантикой ссылок.

#### Когда применять

Нужно создавать варианты подготовленного образца, а правила клонирования и сохранения общих ссылок являются частью доменной модели.

#### Когда избыточен

Объект дешёво и понятно создаётся из исходных параметров; копирование добавляет неясность вокруг identity и вложенных ссылок.

#### Более простое решение

Вызвать конструктор или factory. Для простых записей достаточно shallow copy, если общие вложенные ссылки допустимы; для поддерживаемых типов можно использовать structuredClone с явно подходящей семантикой.

#### Хороший пример

```js
class Point {
  constructor(x) { this.x = x; }
  clone() { return new Point(this.x); }
}
class Line {
  constructor(start, end) { this.start = start; this.end = end; }
  clone() { return new Line(this.start.clone(), this.end.clone()); }
}
const original = new Line(new Point(0), new Point(10));
const copy = original.clone();
copy.start.x = 5;
console.log(original.start.x); // 0
```

#### Плохой пример

```js
const original = { start: { x: 0 }, end: { x: 10 } };
const copy = { ...original };
copy.start.x = 5;
console.log(original.start.x); // 5: общая вложенная точка
```

#### В чём разница

Хороший clone сохраняет требуемую независимость вложенных точек. Spread копирует только внешний объект. Он подходит для shallow copy, но нарушает контракт независимой геометрической копии.

### Singleton

Разделять один экземпляр в явно заданной области.

#### Когда применять

В пределах явно определённой области нужен один общий координатор или ресурс, и несколько экземпляров нарушили бы согласованность.

#### Когда избыточен

Состояние относится к запросу, tenant или тесту либо несколько независимых экземпляров допустимы. Глобальный доступ тогда только скрывает зависимости.

#### Более простое решение

Создать экземпляр при запуске приложения и передать его потребителям. Для независимых запросов использовать отдельные экземпляры или request context.

#### Хороший пример

```js
// Один экземпляр приложения; request state хранится отдельно.
const counters = new Map();
const recordRequest = (userId) => {
  counters.set(userId, (counters.get(userId) ?? 0) + 1);
};
recordRequest('anna');
recordRequest('boris');
console.log(counters.get('anna')); // 1
```

#### Плохой пример

```js
let currentUser;
const handle = async (user) => {
  currentUser = user;
  await Promise.resolve();
  return currentUser;
};
console.log(await Promise.all([handle('anna'), handle('boris')]));
// ['boris', 'boris']: запрос Anna прочитал чужое состояние
```

#### В чём разница

Общий счётчик может намеренно принадлежать приложению. Текущий пользователь принадлежит запросу: хранение его в singleton смешивает параллельные операции. Область «один» нужно задавать явно: модуль, приложение или процесс.

### Multiton

Один экземпляр на ключ.

#### Когда применять

Нужен повторно используемый экземпляр на устойчивый ключ: tenant, endpoint или конфигурацию; правила совместного использования и завершения ясны.

#### Когда избыточен

Потребитель один, создание дешёвое либо экземпляр должен быть независим на каждый запрос. Реестр создаст скрытое разделяемое состояние и обязанность eviction.

#### Более простое решение

Создать и передать нужный экземпляр при сборке приложения или создавать его на операцию. Для краткой серии вызовов достаточно локального Map с ограниченным lifetime.

#### Хороший пример

```js
const clients = new Map();
const forTenant = (tenantId) => {
  if (!clients.has(tenantId)) {
    clients.set(tenantId, { tenantId, query() { return this.tenantId; } });
  }
  return clients.get(tenantId);
};
console.log(forTenant('anna') === forTenant('anna')); // true
console.log(forTenant('anna').query(), forTenant('boris').query()); // anna boris
```

#### Плохой пример

```js
const client = { tenantId: null };
const query = async (tenantId) => {
  client.tenantId = tenantId;
  await Promise.resolve();
  return client.tenantId;
};
console.log(await Promise.all([query('anna'), query('boris')]));
// ['boris', 'boris']: общий клиент перенастроили во время запроса
```

#### В чём разница

Multiton сохраняет отдельный экземпляр на ключ. Перенастройка singleton по tenant создаёт межзапросную гонку. Для большого числа tenant нужны ограничения lifetime, eviction и корректное закрытие клиентов.

### Context

Явно передавать зависимости и состояние запроса.

#### Когда применять

Несколько связанных операций одного запроса или job используют общий небольшой набор capabilities и состояния, который нужно явно передать через границы.

#### Когда избыточен

Нужны один-два аргумента или зависимости не образуют общую область жизни. Универсальный context превращается в скрытый service locator.

#### Более простое решение

Передать именованные параметры напрямую или замкнуть постоянные зависимости в factory сервиса; request state оставить аргументом операции.

#### Хороший пример

```js
const createService = ({ user, logger }) => ({
  async owner() {
    await Promise.resolve();
    logger.push(user.id);
    return user.id;
  },
});
const log = [];
const anna = createService({ user: { id: 'anna' }, logger: log });
const boris = createService({ user: { id: 'boris' }, logger: log });
console.log(await Promise.all([anna.owner(), boris.owner()]));
// ['anna', 'boris']
```

#### Плохой пример

```js
const context = { user: null };
const owner = async (user) => {
  context.user = user;
  await Promise.resolve();
  return context.user.id;
};
console.log(await Promise.all([
  owner({ id: 'anna' }), owner({ id: 'boris' }),
])); // ['boris', 'boris']
```

#### В чём разница

Context создаётся на запрос и передаётся явно. Общий mutable context превращает request state в глобальное состояние. Контекст также не должен становиться бесконтрольным service locator со всеми зависимостями системы.

Factory Method в источнике также охватывает static factory и registry dispatch.
Это связанные варианты выбора создаваемого типа, но registry сам по себе
не равен классическому полиморфному Factory Method.
Модульный Singleton не гарантирует один экземпляр между процессами и workers.

## 2. Границы интерфейсов, композиция и управление поведением

### Adapter

Привести чужой интерфейс к ожидаемому.

#### Когда применять

На границе с внешней библиотекой или legacy-кодом различаются сигнатуры, форма данных, callback/Promise-модель либо ошибки, а изменить внешнюю сторону нельзя.

#### Когда избыточен

Контракты уже совместимы или преобразование одноразовое и очевидное. Отдельный класс не добавляет изоляции.

#### Более простое решение

Вызвать API напрямую либо выполнить локальное преобразование. Для стандартного callback API использовать готовый promisify с сохранением необходимого receiver.

#### Хороший пример

```js
import { promisify } from 'node:util';

const client = {
  prefix: 'user:',
  read(id, callback) { callback(null, this.prefix + id); },
};
const read = promisify(client.read.bind(client));
console.log(await read(7)); // user:7
```

#### Плохой пример

```js
import { promisify } from 'node:util';

const client = {
  prefix: 'user:',
  read(id, callback) { callback(null, this.prefix + id); },
};
const read = promisify(client.read);
try { await read(7); } catch (error) {
  console.log(error.name); // TypeError: receiver потерян
}
```

#### В чём разница

Оба варианта переводят callback в Promise. Только первый сохраняет receiver, от которого зависит client.read. При адаптации нужно переносить весь контракт: this, результаты, ошибки и необходимые параметры.

### Wrapper

Добавить поведение вокруг вызова.

#### Когда применять

Нужно повторно добавлять одинаковое действие вокруг вызовов: измерение времени, логирование или проверку, сохраняя исходный контракт функции.

#### Когда избыточен

Действие нужно в одном месте или оно настолько специфично для операции, что wrapper требует множества исключений.

#### Более простое решение

Оставить несколько явных строк до и после прямого вызова; cleanup при необходимости выразить через try/finally.

#### Хороший пример

```js
const withLog = (fn, log) => function (...args) {
  log.push(args);
  return fn.apply(this, args);
};
const calculator = {
  factor: 2,
  multiply(value) { return this.factor * value; },
};
calculator.multiply = withLog(calculator.multiply, []);
console.log(calculator.multiply(3)); // 6
```

#### Плохой пример

```js
const withLog = (fn, log) => function (...args) {
  log.push(args);
  fn(...args);
};
const calculator = {
  factor: 2,
  multiply(value) { return this.factor * value; },
};
calculator.multiply = withLog(calculator.multiply, []);
// calculator.multiply(3); // TypeError: this потерян
// Даже независимая от this функция потеряла бы return.
```

#### В чём разница

Wrapper должен сохранить аргументы, receiver и результат. Во втором варианте ломается this и исчезает возвращаемое значение; для async-функции исчезнет также Promise, который caller должен ожидать.

### Decorator

Комбинировать поведения через общий контракт.

#### Когда применять

Нужно независимо комбинировать несколько дополнительных поведений через один интерфейс, причём порядок слоёв имеет определённый смысл.

#### Когда избыточен

Есть одно фиксированное поведение либо комбинаций мало и они не меняются. Цепочка обёрток затруднит трассировку без практической пользы.

#### Более простое решение

Использовать обычную функцию с последовательными проверками или явный массив шагов, если их контракт одинаков.

#### Хороший пример

```js
const accept = { validate: () => true };
const required = (next) => ({
  validate: (value) => value !== '' && next.validate(value),
});
const email = (next) => ({
  validate: (value) => value.includes('@') && next.validate(value),
});
const validator = email(required(accept));
console.log(validator.validate('anna@example.org')); // true
console.log(validator.validate('invalid')); // false
```

#### Плохой пример

```js
const email = {
  validate: (value) => value.includes('@') ? true : 'Invalid email',
};
const submit = (value) => email.validate(value) ? 'saved' : 'rejected';
console.log(submit('invalid')); // saved: строка ошибки truthy
```

#### В чём разница

Слои первого варианта соблюдают boolean-контракт. Строка ошибки во втором выглядит как успешный truthy результат. Другой формат ошибок возможен, но его должны согласованно поддерживать все слои и callers.

### Chain of Responsibility

Цепь обработчиков с возможностью остановки.

#### Когда применять

Несколько обработчиков по порядку решают, завершить запрос или передать его дальше; порядок и остановка входят в контракт.

#### Когда избыточен

Всегда выполняются все фиксированные шаги и ни один не выбирает владельца обработки. Цепочка next тогда скрывает обычную последовательность.

#### Более простое решение

Написать прямую функцию с последовательными проверками и ранними return. Для преобразований без остановки достаточно явного pipeline.

#### Хороший пример

```js
const response = [];
const auth = (request, next) => {
  if (!request.user) { response.push('401'); return; }
  next();
};
const handler = () => response.push('200');
auth({ user: null }, handler);
console.log(response); // ['401']: цепочка остановлена
```

#### Плохой пример

```js
const response = [];
const auth = (request, next) => {
  if (!request.user) response.push('401');
  next();
};
const handler = () => response.push('200');
auth({ user: null }, handler);
console.log(response); // ['401', '200']: protected handler всё равно запущен
```

#### В чём разница

Обработчик либо завершает обработку, либо передаёт дальше. Отсутствующий return после отказа не только дублирует ответ, но и может пропустить запрос в защищённую операцию.

### Proxy

Контролировать доступ к объекту через совместимый интерфейс.

#### Когда применять

Доступ к объекту должен проходить через контролируемую точку: для политики доступа, ленивого получения ресурса или прозрачного наблюдения.

#### Когда избыточен

Достаточно одной проверки в одном caller, а прозрачная подмена всего интерфейса не требуется. Наследование платформенного класса может создать ненужную связь с его внутренней жизнью.

#### Более простое решение

Явно вызвать проверку перед операцией или обернуть конкретную функцию. Встроенный JavaScript Proxy использовать только при необходимости его возможностей перехвата.

#### Хороший пример

```js
import { EventEmitter } from 'node:events';

class ObservedEmitter extends EventEmitter {
  count = 0;
  emit(name, ...args) {
    this.count++;
    return super.emit(name, ...args);
  }
}
const emitter = new ObservedEmitter();
emitter.on('message', console.log);
console.log(emitter.emit('message', 'hello')); // hello, затем true
```

#### Плохой пример

```js
import { EventEmitter } from 'node:events';

class ObservedEmitter extends EventEmitter {
  count = 0;
  emit(name, ...args) { this.count++; }
}
const emitter = new ObservedEmitter();
emitter.on('message', console.log);
console.log(emitter.emit('message', 'hello')); // только undefined
```

#### В чём разница

Перехват добавляет наблюдение, сохраняя доставку события и boolean-результат emit. Плохой вариант поглощает вызов: слушатели не запускаются. Это Proxy по роли объекта, не встроенный JavaScript Proxy.

### Bridge

Разделить две независимо меняющиеся оси.

#### Когда применять

Есть две независимо меняющиеся оси и их комбинации: например, тип устройства и транспорт. Отдельный класс на каждую пару уже ведёт к дублированию.

#### Когда избыточен

Одна ось фиксирована, комбинаций мало или независимость изменений только предполагается.

#### Более простое решение

Передать одну функцию или объект реализации в существующий компонент без отдельных абстрактных иерархий.

#### Хороший пример

```js
class SmartLight {
  constructor(transport) { this.transport = transport; }
  on() { return this.transport.send('light:on'); }
}
const mqtt = { send: (command) => `mqtt:${command}` };
const http = { send: (command) => `http:${command}` };
console.log(new SmartLight(mqtt).on()); // mqtt:light:on
console.log(new SmartLight(http).on()); // http:light:on
```

#### Плохой пример

```js
class MqttLight {
  on() { return 'mqtt:light:on'; }
  off() { return 'mqtt:light:off'; }
}
class HttpLight {
  on() { return 'http:light:on'; }
  // Добавили off только в MqttLight.
}
console.log(typeof new HttpLight().off); // undefined
```

#### В чём разница

Логика устройства меняется независимо от транспорта. При классах на каждую комбинацию одна функция устройства дублируется; новый off нужно синхронно добавить во все транспортные варианты.

### Strategy

Выбор поведения, в том числе по lookup.

#### Когда применять

Одну операцию нужно выполнять несколькими взаимозаменяемыми способами, а выбор правила должен быть отделён от остального workflow.

#### Когда избыточен

Есть единственный алгоритм или одна короткая стабильная ветка. Стратегии как отдельные классы не добавляют полезной вариативности.

#### Более простое решение

Оставить if/else в одном месте либо передать функцию-параметр. Для выбора по ключу достаточно небольшой таблицы функций.

#### Хороший пример

```js
const strategies = new Map([
  ['regular', (total) => total],
  ['member', (total) => total * 0.9],
]);
const price = (kind, total) => {
  const strategy = strategies.get(kind);
  if (!strategy) throw new Error('Unknown pricing policy');
  return strategy(total);
};
console.log(price('member', 100)); // 90
```

#### Плохой пример

```js
const checkoutPrice = (kind, total) => kind === 'member' ? total * 0.9 : total;
const invoicePrice = (kind, total) => kind === 'member' ? total * 0.95 : total;
console.log(checkoutPrice('member', 100), invoicePrice('member', 100)); // 90 95
```

#### В чём разница

Одна выбранная стратегия задаёт правило расчёта для всех callers. Независимые ветки checkout/invoice расходятся. Strategy не требует class hierarchy: функции и таблица достаточно выражают изменяемое поведение.

### Composite

Одинаково обрабатывать лист и составной узел.

#### Когда применять

Данные действительно образуют иерархию, а клиенты должны одинаково выполнять операцию над отдельным элементом и группой.

#### Когда избыточен

Структура плоская либо лист и группа не имеют разумного общего контракта. Иерархия классов ради одного обхода может оказаться лишней.

#### Более простое решение

Использовать массив с reduce/map или одну функцию рекурсивного обхода обычного дерева данных.

#### Хороший пример

```js
class Item {
  constructor(price) { this.price = price; }
  total() { return this.price; }
}
class Group {
  constructor(children) { this.children = children; }
  total() { return this.children.reduce((sum, child) => sum + child.total(), 0); }
}
const order = new Group([new Item(10), new Group([new Item(5)])]);
console.log(order.total()); // 15
```

#### Плохой пример

```js
const order = { children: [{ price: 10 }, { children: [{ price: 5 }] }] };
const total = (group) => group.children.reduce((sum, child) => {
  if ('price' in child) return sum + child.price;
  return sum; // этот caller забыл вложенные группы
}, 0);
console.log(total(order)); // 10 вместо 15
```

#### В чём разница

Лист и группа имеют общий total, поэтому рекурсия сосредоточена в структуре. Во втором варианте callers должны знать все формы узлов, и один из них пропускает вложенную группу. Для простого JSON корректный общий обход также может быть достаточен.

### Facade

Дать небольшой API над сценарием нескольких подсистем.

#### Когда применять

Повторяющийся сценарий требует согласовать несколько подсистем, а callers должны получать небольшой устойчивый API вместо знания порядка внутренних вызовов.

#### Когда избыточен

Сценарий состоит из одного понятного вызова или фасад лишь переименовывает API зависимости без изменения границы.

#### Более простое решение

Вызывать зависимость напрямую либо выделить одну функцию сценария с явными параметрами.

#### Хороший пример

```js
const createUsersApi = (db, cache) => ({
  async load(id) {
    if (cache.has(id)) return cache.get(id);
    const user = await db.load(id);
    cache.set(id, user);
    return user;
  },
});
const db = { load: async (id) => ({ id, name: 'Anna' }) };
const cache = new Map();
const api = createUsersApi(db, cache);
console.log((await api.load(1)) === (await api.load(1))); // true
```

#### Плохой пример

```js
const cache = new Map();
const db = { load: async (id) => ({ id, name: 'Anna' }) };
const profileRoute = async (id) => {
  if (cache.has(id)) return cache.get(id);
  const user = await db.load(id);
  cache.set(id, user);
  return user;
};
const exportRoute = async (id) => cache.get(id); // забыли DB fallback
console.log(await exportRoute(1)); // undefined, хотя пользователь существует
```

#### В чём разница

Facade даёт callers один контракт загрузки. Без него сценарий воспроизводится вручную: export route теряет fallback. Фасад не обязан включать все операции системы; его граница должна оставаться небольшой.

### Mediator

Координация взаимодействий.

#### Когда применять

Несколько участников взаимодействуют сложным образом, и правила координации нужно собрать в одном месте вместо взаимных скрытых вызовов.

#### Когда избыточен

Взаимодействуют два компонента одним прямым вызовом или координатор превращается в центр всех несвязанных сценариев.

#### Более простое решение

Оставить прямой вызов или небольшую функцию сценария. Для одного линейного use case может хватить Facade без event-driven координации.

#### Хороший пример

```js
const events = [];
const inventory = { reserve: (id) => events.push(`reserve:${id}`) };
const mailer = { send: (id) => events.push(`email:${id}`) };
const checkout = {
  complete(id) {
    inventory.reserve(id);
    mailer.send(id);
  },
};
checkout.complete(1);
console.log(events); // ['reserve:1', 'email:1']
```

#### Плохой пример

```js
const events = [];
let mailer;
const inventory = {
  reserve(id) { events.push(`reserve:${id}`); mailer.send(id); },
};
mailer = {
  send(id) { events.push(`email:${id}`); inventory.reserve(id); },
};
// inventory.reserve(1); // взаимные вызовы без остановки → stack overflow
```

#### В чём разница

Координатор задаёт взаимодействие, участники не вызывают друг друга скрыто. В плохом варианте взаимная координация образовала рекурсивный цикл. В этом простом синхронном сценарии координатор также играет роль Facade; Mediator особенно полезен при взаимодействии через события.

### Boxing / Value Object

Собрать в одном месте инварианты доменного значения.

#### Когда применять

Значение имеет устойчивый доменный смысл, инварианты и операции, которые иначе приходится проверять во многих местах: деньги, единицы измерения, адреса.

#### Когда избыточен

Это одноразовое скалярное поле с проверкой на входе, а отдельный объект не защищает дополнительных инвариантов и усложняет сериализацию.

#### Более простое решение

Проверить primitive на границе и передавать его с точным именем и единицами. В TypeScript при подходящей задаче можно дополнить контракт branded type, сохранив runtime-валидацию внешнего ввода.

#### Хороший пример

```js
class Money {
  constructor(cents) {
    if (!Number.isSafeInteger(cents) || cents < 0) {
      throw new TypeError('Expected non-negative integer cents');
    }
    this.cents = cents;
    Object.freeze(this);
  }
  add(other) { return new Money(this.cents + other.cents); }
}
console.log(new Money(100).add(new Money(50)).cents); // 150
// new Money(NaN); // TypeError
```

#### Плохой пример

```js
const createMoney = (cents) => ({ cents });
const amount = createMoney(NaN);
const total = amount.cents + createMoney(50).cents;
console.log(total); // NaN дошёл до результата
```

#### В чём разница

Value Object проверяет инвариант при создании и сохраняет его при операции add. Простая обёртка без валидации только меняет форму значения. В примере фиксированы одна валюта и неотрицательные целые cents; валютные операции требуют отдельного контракта.

Adapter меняет контракт; Wrapper добавляет действие вокруг вызова;
Decorator составляет поведения через общий контракт; Proxy управляет доступом;
Facade объединяет сценарий подсистем. Одна реализация может сочетать эти роли.
Boxing здесь означает доменный Value Object, а не `new String(value)`.

## 3. Разделение памяти и управление ресурсами

### Flyweight

Разделять общее неизменяемое состояние.

#### Когда применять

Большое число объектов дублирует значимый объём одинаковых неизменяемых данных, а данные конкретного использования можно хранить отдельно.

#### Когда избыточен

Дублирование мало, значения часто мутируют или усложнение identity, ключей и eviction дороже экономии памяти.

#### Более простое решение

Хранить обычные независимые записи. Если общее значение действительно одно и неизменно, достаточно общей константы без factory-кэша.

#### Хороший пример

```js
const glyphs = new Map();
const getGlyph = (char) => {
  if (!glyphs.has(char)) glyphs.set(char, Object.freeze({ char }));
  return glyphs.get(char);
};
const first = { glyph: getGlyph('A'), row: 0 };
const second = { glyph: getGlyph('A'), row: 1 };
first.row = 10;
console.log(first.glyph === second.glyph, second.row); // true 1
```

#### Плохой пример

```js
const shared = { char: 'A', row: 0 };
const first = { glyph: shared };
const second = { glyph: shared };
first.glyph.row = 10;
console.log(second.glyph.row); // 10: чужая позиция изменилась
```

#### В чём разница

Общие данные символа неизменяемы, координаты принадлежат отдельным использованиям. Плохой вариант разделяет также изменяемую позицию. Для большого набора ключей flyweight-кэшу нужна политика удержания памяти.

### Object Pool

Повторно использовать дорогие изменяемые объекты.

#### Когда применять

Создание или инициализация объектов измеримо дороги, объекты допускают надёжный reset, а право использования и возврата можно чётко ограничить.

#### Когда избыточен

Объекты малы и дешевы, нагрузка не создаёт подтверждённой проблемы allocation либо reset сложнее безопасного создания заново.

#### Более простое решение

Создавать объект для каждой операции и позволить GC освобождать его. Для строго последовательного владельца иногда достаточно одного рабочего буфера без общего pool.

#### Хороший пример

```js
const idle = [new Uint8Array(2)];
const pool = {
  acquire: () => idle.pop() ?? new Uint8Array(2),
  release(buffer) { buffer.fill(0); idle.push(buffer); },
};
const send = async (buffer) => {
  await Promise.resolve();
  return [...buffer];
};
const buffer = pool.acquire();
buffer[0] = 7;
try { console.log(await send(buffer)); } // [7, 0]
finally { pool.release(buffer); }
```

#### Плохой пример

```js
const buffer = new Uint8Array([7, 0]);
const send = async (bytes) => {
  await Promise.resolve();
  return [...bytes];
};
const pending = send(buffer);
// Ресурс уже вернули и переиспользовали до завершения send.
buffer.fill(9);
console.log(await pending); // [9, 9]: отправлены другие данные
```

#### В чём разница

Ресурс возвращается после окончания всех пользователей и очищается перед следующим использованием. Возврат до await создаёт гонку на общей памяти. Учебный pool не ограничивает concurrency и idle capacity; промышленная политика задаётся отдельно.

### Resource Pool / Lease

Выдать ресурс с контролируемым возвратом.

#### Когда применять

Внешние ресурсы ограничены или дороги: соединения, workers, устройства. Нужны контроль конкурентного доступа, ожидание выдачи и обязательное освобождение.

#### Когда избыточен

Ресурс нужен только одной короткой операции или существующий клиент уже управляет pool; второй pool усложнит лимиты и ownership.

#### Более простое решение

Использовать pool библиотеки либо открыть и закрыть один ресурс в try/finally. Для единственного владельца передать уже открытый ресурс напрямую.

#### Хороший пример

```js
const createPool = () => {
  let busy = false;
  return {
    acquire() {
      if (busy) throw new Error('Pool exhausted');
      busy = true;
      let released = false;
      return { resource: { id: 1 }, release() {
        if (released) throw new Error('Already released');
        released = true;
        busy = false;
      } };
    },
  };
};
const pool = createPool();
const lease = pool.acquire();
try { throw new Error('Work failed'); }
catch (error) { console.log(error.message); }
finally { lease.release(); }
const next = pool.acquire(); // ресурс снова доступен
next.release();
```

#### Плохой пример

```js
let busy = false;
const acquire = () => {
  if (busy) throw new Error('Pool exhausted');
  busy = true;
  return { release() { busy = false; } };
};
try {
  const lease = acquire();
  throw new Error('Work failed');
  lease.release(); // недостижимо
} catch (error) { console.log(error.message); }
// acquire(); // Pool exhausted после первой ошибки
```

#### В чём разница

finally возвращает lease при любом исходе работы; защита от повторного release удерживает контракт владения. В плохом варианте первая ошибка навсегда занимает единственный слот.

### Ограничение удерживаемой памяти

Не допускать неограниченного роста долгоживущего состояния.

#### Когда применять

Долгоживущая коллекция принимает растущий набор ключей или тяжёлых значений, поэтому нужно определить предел, lifetime и поведение при его достижении.

#### Когда избыточен

Само требование ограниченности остаётся полезным; избыточна отдельная eviction-система для маленького статического набора или данных, живущих только одну короткую операцию.

#### Более простое решение

Оставить данные локальными переменными операции, очищать их при завершении либо использовать фиксированный набор. Если кэш не даёт измеримой пользы, убрать его.

#### Хороший пример

```js
const cache = new Map();
const maxEntries = 2;
const remember = (key, value) => {
  cache.set(key, value);
  if (cache.size > maxEntries) cache.delete(cache.keys().next().value);
};
for (const id of [1, 2, 3]) remember(id, { id });
console.log([...cache.keys()]); // [2, 3]: FIFO eviction
```

#### Плохой пример

```js
const cache = new Map();
const remember = (key, value) => cache.set(key, value);
for (let id = 0; id < 10000; id++) remember(id, { id });
console.log(cache.size); // 10000; история запросов определяет рост
```

#### В чём разница

Первый кэш ограничивает число записей явной FIFO-политикой. Второй удерживает все значения до завершения процесса. Предел entries не ограничивает точное число байтов; для тяжёлых значений может понадобиться дополнительный бюджет.

### Явный lifecycle подписки

Удалять слушателей тем же объектом функции.

#### Когда применять

Источник событий может пережить подписчика, подписки создаются повторно или их callback удерживает состояние завершённой операции.

#### Когда избыточен

Отдельный менеджер lifecycle не нужен для одной подписки, если существующий framework уже гарантирует teardown. Освобождение долгоживущей подписки при этом не становится необязательным.

#### Более простое решение

Сохранить ссылку на handler и снять его в существующем hook завершения. Если требуется ровно одно событие, рассмотреть once; при поддержке API можно использовать AbortSignal.

#### Хороший пример

```js
import { EventEmitter } from 'node:events';
const stream = new EventEmitter();
const received = [];
const onData = (chunk) => received.push(chunk);
stream.on('data', onData);
stream.emit('data', 'first');
stream.off('data', onData);
stream.emit('data', 'second');
console.log(received, stream.listenerCount('data')); // ['first'] 0
```

#### Плохой пример

```js
import { EventEmitter } from 'node:events';
const stream = new EventEmitter();
const received = [];
stream.on('data', (chunk) => received.push(chunk));
stream.off('data', (chunk) => received.push(chunk));
stream.emit('data', 'after-close');
console.log(received, stream.listenerCount('data')); // ['after-close'] 1
```

#### В чём разница

off сравнивает ссылки на функции. Новый callback с тем же текстом не равен зарегистрированному. Подписка продолжает работать и удерживать захваченные значения после завершения владельца.

Flyweight разделяет неизменяемые данные одновременно. Pool повторно выдаёт
изменяемый ресурс после окончания предыдущего использования. Ограничение
idle-объектов pool не означает ограничения одновременно выданных ресурсов.

## 4. Представление данных и выбор коллекции

Это структуры данных и приёмы их использования, а не GoF-паттерны.

### Object как запись

Хранить известный набор полей.

#### Когда применять

Набор полей известен, значения имеют разные роли, а данные передаются как DTO, options или сериализуемая запись.

#### Когда избыточен

Ключи произвольны, появляются и исчезают динамически либо требуется identity объектов как ключей. Фиксированная схема тогда не соответствует задаче.

#### Более простое решение

Для одного значения передать scalar. Для пары явно именованных параметров иногда достаточно аргументов функции; для динамического словаря использовать Map.

#### Хороший пример

```js
const createUser = (id, name) => ({ id, name, active: false });
const user = createUser(1, 'Anna');
const metadata = new Map([['source', 'import']]);
console.log(user.id, metadata.get('source')); // 1 import
```

#### Плохой пример

```js
const user = { id: 1, name: 'Anna', active: false };
const customField = 'id';
user[customField] = 'external-note';
console.log(user.id); // external-note: metadata затёрла идентификатор
```

#### В чём разница

Поля записи имеют стабильный смысл, произвольные metadata живут отдельно. Смешивание пространств имён позволяет внешнему ключу перезаписать поле DTO. Динамический object возможен, если такое поведение является его явным контрактом.

### Map для динамических ключей

Хранить изменяемое соответствие ключ → значение.

#### Когда применять

Нужны изменяемое соответствие ключ → значение, ключи-объекты, size и операции добавления/удаления без смешения с полями записи.

#### Когда избыточен

Есть несколько фиксированных именованных полей или данные должны напрямую сериализоваться в JSON. Map потребует лишних преобразований.

#### Более простое решение

Использовать литерал Object для известной схемы; для маленького статического соответствия строковых ключей достаточно объекта с корректной проверкой собственных свойств.

#### Хороший пример

```js
const firstSocket = { id: 1 };
const secondSocket = { id: 2 };
const sessions = new Map();
sessions.set(firstSocket, 'Anna');
sessions.set(secondSocket, 'Boris');
console.log(sessions.get(firstSocket), sessions.size); // Anna 2
```

#### Плохой пример

```js
const firstSocket = { id: 1 };
const secondSocket = { id: 2 };
const sessions = {};
sessions[firstSocket] = 'Anna';
sessions[secondSocket] = 'Boris';
console.log(sessions[firstSocket], Object.keys(sessions).length); // Boris 1
```

#### В чём разница

Map различает ключи по identity объектов. Object преобразует оба ключа в строку [object Object], поэтому вторая запись уничтожает первую.

### Set для уникальности / membership

Проверять присутствие и устранять дубликаты.

#### Когда применять

Уникальность является частью модели либо требуется много проверок присутствия в изменяемом наборе.

#### Когда избыточен

Набор состоит из нескольких неизменяемых элементов, проверяется редко, а стоимость отдельного Set не оправдана. Для сохранения повторов он вообще не подходит.

#### Более простое решение

Использовать маленький массив с includes или прямое условие. Если membership уже представлена ключами существующего Map, отдельный Set не нужен.

#### Хороший пример

```js
const seen = new Set();
const unique = [];
for (const id of [1, 1, 2]) {
  if (seen.has(id)) continue;
  seen.add(id);
  unique.push(id);
}
console.log(unique); // [1, 2]
```

#### Плохой пример

```js
const seen = [];
for (const id of [1, 1, 2]) seen.push(id);
console.log(seen.length); // 3 вместо двух уникальных ID
```

#### В чём разница

Set выражает уникальность явно; массив сам её не обеспечивает. Если использовать массив с includes, результат может быть корректным, но при большом количестве проверок потребуется оценить стоимость линейного поиска.

### Array для последовательности

Хранить порядок, дубликаты и доступ по индексу.

#### Когда применять

Нужны порядок, повторяющиеся элементы, доступ по индексу и стандартные преобразования последовательности.

#### Когда избыточен

Нужно только одно значение, фиксированная запись с именованными полями либо динамический поиск по ключу. Позиционные соглашения могут скрыть смысл данных.

#### Более простое решение

Для одного элемента передать его напрямую; для нескольких разных ролей использовать object; для поиска по ключу выбрать Map.

#### Хороший пример

```js
const steps = ['open', 'read', 'read', 'close'];
const executed = [];
for (const step of steps) executed.push(step);
console.log(executed); // ['open', 'read', 'read', 'close']
```

#### Плохой пример

```js
const steps = new Set(['open', 'read', 'read', 'close']);
const executed = [];
for (const step of steps) executed.push(step);
console.log(executed); // ['open', 'read', 'close']: один read потерян
```

#### В чём разница

Операции образуют последовательность, в которой повтор имеет смысл. Set сохраняет порядок уникальных элементов, но удаляет дубликат, меняя сценарий.

### WeakMap / WeakSet

Связать дополнительные данные с временем жизни объекта.

#### Когда применять

Дополнительные данные или признак присутствия должны существовать только пока объект-ключ жив сам по себе; перечислять ключи и считать size не требуется.

#### Когда избыточен

Lifetime уже контролируется владельцем, данные естественно являются полями собственного объекта или нужны enumeration и предсказуемое удаление.

#### Более простое решение

Хранить поле в принадлежащем приложению объекте либо использовать обычный Map с явным удалением в конце lifecycle. Не рассчитывать на GC как на механизм своевременного закрытия ресурса.

#### Хороший пример

```js
const metadata = new WeakMap();
let node = { text: 'hello' };
metadata.set(node, { selected: true });
console.log(metadata.get(node).selected); // true
node = null; // metadata не создаёт сильную ссылку на бывший ключ
```

#### Плохой пример

```js
const metadata = new Map();
let node = { text: 'hello' };
metadata.set(node, { selected: true });
node = null;
console.log(metadata.size); // 1: Map по-прежнему удерживает объект
```

#### В чём разница

WeakMap не продлевает жизнь ключа, когда остальных сильных ссылок не осталось. Срок GC не гарантирован, поэтому пример не проверяет момент освобождения. Map подходит при необходимости перечисления, но требует явного удаления записей.

### TypedArray / ArrayBuffer

Представлять числовые и бинарные данные.

#### Когда применять

Данные имеют конкретное бинарное или числовое представление: байты протокола, samples, shared buffer или взаимодействие с низкоуровневым API.

#### Когда избыточен

Это обычный список разнородных значений или чисел без требований к бинарному layout. Ограничения диапазона и преобразования типов только усложнят работу.

#### Более простое решение

Использовать обычный Array. Если API Node.js уже возвращает подходящий Buffer, не копировать его в отдельный TypedArray без необходимости.

#### Хороший пример

```js
const buffer = new ArrayBuffer(4);
const samples = new Uint16Array(buffer);
samples[0] = 300;
samples[1] = 1000;
console.log([...samples], buffer.byteLength); // [300, 1000] 4
```

#### Плохой пример

```js
const samples = new Uint8Array(2);
samples[0] = 300;
samples[1] = 1000;
console.log([...samples]); // [44, 232]: значения усечены до 8 бит
```

#### В чём разница

Тип представления выбран по диапазону значений. Uint8Array не валидирует произвольное число: он преобразует его к 8-битному значению, что незаметно искажает измерения.

### DataView

Читать бинарный формат с явным byte order.

#### Когда применять

Нужно читать или записывать поля бинарного формата с явным byte order, разными ширинами значений или невыравненными offsets.

#### Когда избыточен

Работа ограничена отдельными байтами либо готовый parser уже корректно разбирает протокол. Дополнительный ручной разбор увеличит вероятность ошибки.

#### Более простое решение

Использовать Uint8Array для байтов или готовый parser. Для Node Buffer можно пользоваться его подходящими read/write-методами с явным порядком байтов.

#### Хороший пример

```js
const bytes = new Uint8Array([0x01, 0x02]);
const view = new DataView(bytes.buffer);
const value = view.getUint16(0, true); // протокол little-endian
console.log(value); // 513
```

#### Плохой пример

```js
const bytes = new Uint8Array([0x01, 0x02]);
const view = new DataView(bytes.buffer);
const value = view.getUint16(0); // default big-endian
console.log(value); // 258: прочитано другое число
```

#### В чём разница

DataView позволяет явно задать byte order протокола. Пропущенный аргумент переключает интерпретацию; multi-byte TypedArray также не заменяет протокольное указание endian, поскольку использует порядок платформы.

### Lookup table / registry dispatch

Централизовать выбор по ключу.

#### Когда применять

Нужно выбирать действие или реализацию по ключу, особенно если одно соответствие используется в нескольких местах или меняется как набор данных.

#### Когда избыточен

Есть одна-две простые ветки или условия не сводятся к точному ключу: диапазоны, последовательные проверки и зависимые эффекты.

#### Более простое решение

Оставить ясный if/switch в одном месте либо передать уже выбранную функцию; не строить registry без потребности в выборе.

#### Хороший пример

```js
const handlers = new Map([
  ['json', (value) => JSON.stringify(value)],
  ['text', (value) => `${value}`],
]);
const render = (kind, value) => {
  const handler = handlers.get(kind);
  if (!handler) throw new Error(`Unknown kind: ${kind}`);
  return handler(value);
};
console.log(render('json', 42)); // 42
```

#### Плохой пример

```js
const exportValue = (kind, value) => {
  if (kind === 'json') return JSON.stringify(value);
  if (kind === 'text') return `${value}`;
  throw new Error('Unknown kind');
};
const previewValue = (kind, value) => {
  if (kind === 'json') return JSON.stringify(value);
  return ''; // забыли text
};
console.log(exportValue('text', 42), previewValue('text', 42)); // '42' и ''
```

#### В чём разница

Таблица имеет одного владельца и явно отвергает неизвестный ключ. Дублированные ветвления расходятся при добавлении форматов. Один небольшой switch без дублирования сам по себе не является проблемой.

### Нормализация aliases на границе

Свести разные входные имена к одному каноническому.

#### Когда применять

Внешние данные допускают несколько имён одной сущности, но внутренняя логика должна работать с одним каноническим значением.

#### Когда избыточен

Контракт разрешает только одно имя или aliases не используются. Поддержка выдуманных синонимов расширяет API и создаёт обязательства совместимости.

#### Более простое решение

Принимать только каноническое имя и явно отвергать неизвестное. Для одного переходного alias достаточно локального преобразования на одной входной границе.

#### Хороший пример

```js
const aliases = new Map([['colour', 'color'], ['COLOR', 'color']]);
const normalize = (input) => aliases.get(input) ?? input;
const save = (input) => ({ field: normalize(input) });
const validate = (input) => normalize(input) === 'color';
console.log(save('colour').field, validate('colour')); // color true
```

#### Плохой пример

```js
const save = (input) => ({ field: input === 'colour' ? 'color' : input });
const validate = (input) => input === 'color';
console.log(save('colour').field, validate('colour')); // color false
```

#### В чём разница

После общей нормализации бизнес-операции видят один канонический ключ. В плохом примере save принимает alias, а validate отвергает тот же ввод.

### Структура по смыслу операции

Выбирать между записью, порядком, membership и индексом.

#### Когда применять

До реализации нужно определить, важны ли порядок, повторы, membership, ключи и ownership; от этого зависит корректность выбранного представления.

#### Когда избыточен

Сам принцип не избыточен; избыточна длительная классификация или универсальная фабрика коллекций там, где смысл очевиден.

#### Более простое решение

Выбрать привычный встроенный тип и кратко зафиксировать существенный контракт. Не вводить отдельный слой выбора структур.

#### Хороший пример

```js
const steps = ['read', 'read', 'close'];
const allowedModes = ['read', 'write'];
console.log(steps.length); // 3 операции
console.log(allowedModes.includes('read')); // true
```

#### Плохой пример

```js
const steps = new Set(['read', 'read', 'close']);
console.log(steps.size); // 2: уникальность навязана последовательности
```

#### В чём разница

Для очередности с повторами нужен Array; для небольшого статического allowlist достаточно includes. Выбор структуры по её популярности, а не по семантике, может менять данные или добавлять ненужную сложность.

### Snapshot / live view

Явно определить, разделяет ли результат состояние.

#### Когда применять

Коллекция пересекает границу владельцев и caller должен знать, отражает ли результат последующие изменения и может ли сам менять внутреннее состояние.

#### Когда избыточен

Независимый snapshot не нужен, если caller намеренно работает с общим состоянием под единым ownership. Глубокое копирование всех данных может быть особенно дорогим и неверным для identity.

#### Более простое решение

Передать явно оговорённую live view, iterator или вычислить нужный scalar. Если достаточно отделить состав массива, сделать shallow copy без deep clone.

#### Хороший пример

```js
const items = [{ id: 1 }];
const snapshot = () => [...items];
const copy = snapshot();
copy.push({ id: 2 });
console.log(items.length); // 1
console.log(copy[0] === items[0]); // true: элементы общие по контракту
```

#### Плохой пример

```js
const items = [{ id: 1 }];
const snapshot = () => items;
const copy = snapshot();
copy.push({ id: 2 });
console.log(items.length); // 2: «копия» изменила владельца
```

#### В чём разница

Первый результат отделяет состав последовательности, но не клонирует элементы. Второй возвращает live view под именем snapshot. Для независимости вложенных объектов нужен более строгий контракт копирования.

### Безопасный словарь

Не смешивать внешние ключи с prototype properties.

#### Когда применять

Произвольные внешние строки используются как ключи, и их нужно отличать от prototype-свойств, сохраняя корректную политику записи.

#### Когда избыточен

Ключи фиксированы и контролируются кодом либо данные уже хранятся в Map. Отдельная инфраструктура словаря не улучшит такой контракт.

#### Более простое решение

Использовать Map для внешних ключей или обычный object с Object.hasOwn и согласованной безопасной записью. Для фиксированного DTO достаточно литерала с известными полями.

#### Хороший пример

```js
const dictionary = Object.create(null);
dictionary['name'] = 'Anna';
console.log(Object.hasOwn(dictionary, 'toString')); // false
console.log(Object.hasOwn(dictionary, 'name')); // true
```

#### Плохой пример

```js
const dictionary = {};
const hasEntry = (key) => key in dictionary;
console.log(hasEntry('toString')); // true: запись не добавляли
```

#### В чём разница

Словарь без prototype и проверка собственных ключей отделяют записи от унаследованных свойств. Во втором варианте внешний ключ может ошибочно считаться существующей записью. Обычный object с Object.hasOwn также подходит при согласованной политике записи ключей.

## 5. Структуры данных, поиск, обход и распределённое состояние

### Singly Linked List

Добавлять и извлекать элементы с головы.

#### Когда применять

Часто выполняются операции с головы или нужно связывать узлы без перемещения оставшихся элементов; доступ по индексу не является основной операцией.

#### Когда избыточен

Нужны частый индексный доступ, сортировка или компактное последовательное хранение. Отдельный объект на каждый узел усложняет обход и управление ссылками.

#### Более простое решение

Использовать Array; для LIFO обычно достаточно push/pop. Собственный список вводить только при нужных операциях или требованиях к sharing.

#### Хороший пример

```js
class StackList {
  head = null;
  push(value) { this.head = { value, next: this.head }; }
  pop() {
    if (!this.head) return undefined;
    const node = this.head;
    this.head = node.next;
    return node.value;
  }
}
const list = new StackList();
list.push(1); list.push(2);
console.log(list.pop(), list.pop()); // 2 1
```

#### Плохой пример

```js
let head = null;
const push = (value) => {
  if (!head) { head = { value, next: null }; return; }
  let tail = head;
  while (tail.next) tail = tail.next;
  tail.next = { value, next: null };
};
push(1); push(2);
console.log(head.value); // 1: добавление в хвост вместо head push
```

#### В чём разница

Первый вариант реализует head push/pop за постоянное число действий. Второй обходит весь список и меняет семантику операции: это append, а не push в голову. Для append стоит хранить tail и явно назвать операцию.

### Doubly Linked List

Удалять или перемещать известные узлы.

#### Когда применять

Нужно удалять или перемещать уже известные узлы, работать с обоими концами либо обходить структуру в обе стороны.

#### Когда избыточен

Достаточно операций с одного конца или список мал и редко меняется. Поддержка prev/next, head/tail и size создаёт дополнительные инварианты.

#### Более простое решение

Использовать Array для небольших последовательностей, singly linked list для односторонних связей или готовый deque для операций с концами.

#### Хороший пример

```js
const a = { value: 'A', prev: null, next: null };
const b = { value: 'B', prev: a, next: null };
const c = { value: 'C', prev: b, next: null };
a.next = b; b.next = c;
const removeMiddle = (node) => {
  node.prev.next = node.next;
  node.next.prev = node.prev;
  node.prev = null;
  node.next = null;
};
removeMiddle(b);
console.log(a.next.value, c.prev.value); // C A
```

#### Плохой пример

```js
const a = { value: 'A', prev: null, next: null };
const b = { value: 'B', prev: a, next: null };
const c = { value: 'C', prev: b, next: null };
a.next = b; b.next = c;
a.next = c; // обновили только прямую ссылку
console.log(a.next.value, c.prev.value); // C B: обходы расходятся
```

#### В чём разница

Удаление должно восстановить обе связи и отсоединить удалённый узел. Здесь показан внутренний узел; полноценная операция дополнительно обрабатывает head/tail и size. Одностороннее исправление оставляет повреждённый обратный обход.

### Immutable Cons List

Создавать версии последовательности с общим хвостом.

#### Когда применять

Нужны независимые версии последовательности, частое prepend и структурное разделение хвостов: история, persistent state или ветвление вычислений.

#### Когда избыточен

История не нужна, изменения должны быть in-place или основной доступ идёт по индексу. Shared tail усложнит работу без полезной экономии копирования.

#### Более простое решение

Для небольшой истории использовать копии Array; для обычного mutable stack — push/pop. Готовая persistent collection предпочтительнее собственной при наличии подходящей зависимости.

#### Хороший пример

```js
const prepend = (value, tail) => Object.freeze({ value, tail });
const history = prepend('initial', null);
const first = prepend('first', history);
const second = prepend('second', history);
console.log(first.tail === second.tail); // true: общий неизменяемый хвост
console.log(history.value); // initial
```

#### Плохой пример

```js
const history = { value: 'initial', tail: null };
const first = { value: 'first', tail: history };
const second = { value: 'second', tail: history };
first.tail.value = 'changed';
console.log(second.tail.value); // changed: повреждена другая версия
```

#### В чём разница

Structural sharing безопасен при неизменяемых узлах. Плохой вариант разделяет изменяемый хвост. Freeze в примере защищает узел, но для object values нужно отдельно определить допустимость мутаций содержимого.

### Circular Buffer

Переиспользовать позиции массива по кругу.

#### Когда применять

Поток данных обрабатывается последовательно, хранилище нужно переиспользовать, а политика capacity и overflow известна; важны частые операции с концами.

#### Когда избыточен

Очередь маленькая и используется редко либо история должна храниться целиком. Ручные индексы и различение full/empty не дают практического выигрыша.

#### Более простое решение

Использовать Array для небольшого объёма или готовую очередь/deque. Для одноразового прохода по накопленному массиву достаточно read cursor.

#### Хороший пример

```js
class Ring {
  constructor(capacity) {
    this.buffer = new Array(capacity).fill(undefined);
    this.head = 0; this.size = 0;
  }
  write(value) {
    if (this.size === this.buffer.length) return false;
    this.buffer[(this.head + this.size) % this.buffer.length] = value;
    this.size++;
    return true;
  }
  read() {
    if (!this.size) return { found: false };
    const value = this.buffer[this.head];
    this.buffer[this.head] = undefined;
    this.head = (this.head + 1) % this.buffer.length;
    this.size--;
    return { found: true, value };
  }
}
const ring = new Ring(2);
ring.write('A'); ring.write('B');
console.log(ring.read(), ring.write('C'), ring.read(), ring.read());
// A, успешная запись C, затем B и C
```

#### Плохой пример

```js
const buffer = new Array(2);
let head = 0;
let tail = 0;
const write = (value) => {
  buffer[tail] = value;
  tail = (tail + 1) % buffer.length;
};
write('A'); write('B');
console.log(head === tail); // true: полное кольцо приняли за пустое
```

#### В чём разница

Размер отличает empty от full и не допускает перезаписи непрочитанных данных. Только head/tail без дополнительного состояния неоднозначны. Учебный Ring предполагает заранее проверенную положительную целую capacity.

### Queue / FIFO

Обрабатывать элементы в порядке поступления.

#### Когда применять

Задачи или сообщения должны обрабатываться в порядке поступления; старые элементы не должны уступать место новым из-за LIFO.

#### Когда избыточен

Есть один элемент, данные можно обработать сразу или обработка представляет один прямой проход по готовому массиву. Отдельный объект Queue тогда не нужен.

#### Более простое решение

Использовать for...of для готового набора; для маленькой очереди — Array с push/shift. При большом потоке выбрать готовую очередь или ring, не меняя FIFO-контракт.

#### Хороший пример

```js
class Queue {
  items = [];
  enqueue(value) { this.items.push(value); }
  dequeue() { return this.items.shift(); }
}
const jobs = new Queue();
jobs.enqueue('first'); jobs.enqueue('second');
console.log(jobs.dequeue(), jobs.dequeue()); // first second
```

#### Плохой пример

```js
const jobs = [];
jobs.push('first'); jobs.push('second');
console.log(jobs.pop(), jobs.pop()); // second first: это LIFO
```

#### В чём разница

Queue сохраняет порядок поступления. Pop превращает планирование в stack и может задерживать старые задачи при постоянном притоке новых. Array.shift здесь выбран для ясной демонстрации; горячей большой очереди подходит ring или другая специализированная реализация.

### Deque

Эффективно работать с обоими концами.

#### Когда применять

Нужны добавление и удаление с обоих концов: скользящее окно, двусторонний обход или алгоритм, чередующий front/back операции.

#### Когда избыточен

Используется только один конец или нужны главным образом поиск и индексные изменения в середине.

#### Более простое решение

Для stack использовать Array push/pop; для маленького окна — обычный Array; если требуется лишь FIFO, ограничить API обычной очередью.

#### Хороший пример

```js
const window = [];
const limit = 3;
const record = (value) => {
  window.push(value);
  if (window.length > limit) window.shift();
};
for (const value of [1, 2, 3, 4]) record(value);
console.log(window); // [2, 3, 4]
```

#### Плохой пример

```js
const window = [];
const limit = 3;
const record = (value) => {
  window.push(value);
  if (window.length > limit) window.pop();
};
for (const value of [1, 2, 3, 4]) record(value);
console.log(window); // [1, 2, 3]: потеряли новое, а не старое
```

#### В чём разница

Окно добавляет справа и удаляет слева. Удаление с того же конца оставляет устаревшие данные. Массив демонстрирует контракт; для частых операций с обоих концов нужна реализация deque с подходящей сложностью.

### Stack / LIFO

Обрабатывать последнее добавленное первым.

#### Когда применять

Последний добавленный элемент должен обрабатываться первым: undo, вложенные операции, итеративный обход в глубину.

#### Когда избыточен

Порядок не имеет значения, нужна FIFO-обработка или нет цепочки вложенных действий. Отдельный класс Stack поверх пары вызовов будет лишним.

#### Более простое решение

Использовать Array с push/pop. Для неглубокого дерева можно оставить ясную рекурсию, если её глубина безопасна для runtime.

#### Хороший пример

```js
const undo = [];
undo.push('type A');
undo.push('type B');
console.log(undo.pop()); // type B: отмена последней команды
```

#### Плохой пример

```js
const undo = [];
undo.push('type A');
undo.push('type B');
console.log(undo.shift()); // type A: отмена в неправильном порядке
```

#### В чём разница

История undo требует LIFO. FIFO-извлечение отменяет раннюю команду, от которой может зависеть более поздняя.

### Unrolled List

Хранить очередь блоками с меньшим числом node allocations.

#### Когда применять

Измеренный поток enqueue/dequeue велик, и блочное хранение помогает снизить число node allocations или улучшить локальность по сравнению с поэлементным linked list.

#### Когда избыточен

Объём умеренный, нужен random access или нет измеренного давления allocation. Размеры блоков, reset и pool усложнят структуру без подтверждённой пользы.

#### Более простое решение

Использовать готовую Queue/CircularBuffer или Array для малого объёма. Блочное хранение вводить после сравнения с более простой очередью.

#### Хороший пример

```js
class BlockQueue {
  constructor() { this.head = this.tail = { values: [], read: 0, next: null }; }
  enqueue(value) {
    if (this.tail.values.length === 2) {
      this.tail.next = { values: [], read: 0, next: null };
      this.tail = this.tail.next;
    }
    this.tail.values.push(value);
  }
  dequeue() {
    if (this.head.read === this.head.values.length && this.head.next) {
      this.head = this.head.next;
    }
    if (this.head.read === this.head.values.length) return undefined;
    const index = this.head.read++;
    const value = this.head.values[index];
    this.head.values[index] = undefined;
    return value;
  }
}
const queue = new BlockQueue();
for (const value of ['A', 'B', 'C']) queue.enqueue(value);
console.log(queue.dequeue(), queue.dequeue(), queue.dequeue()); // A B C
```

#### Плохой пример

```js
const block = { values: ['A', 'B'], read: 0 };
const first = block.values[block.read++];
// Блок вернули в pool после одного чтения.
block.values = [];
block.read = 0;
console.log(first, block.values[block.read]); // A undefined: B потерян
```

#### В чём разница

Блок освобождается только после чтения всех его элементов. Плохой reset уничтожает ещё ожидающие значения. Хороший пример показывает блочное хранение, но не production pool узлов или оптимальную очистку пустого хвоста.

### Binary Search Tree

Упорядочить элементы для поиска и обхода.

#### Когда применять

Нужны упорядоченное хранение, поиск по comparator и операции, зависящие от порядка. Для гарантированной логарифмической глубины нужна балансируемая реализация.

#### Когда избыточен

Нужна только проверка по точному ключу, набор небольшой либо данные загружаются целиком и редко изменяются. Собственный несбалансированный BST может проиграть простым вариантам.

#### Более простое решение

Для точного ключа использовать Map/Set; для небольшого или редко меняющегося упорядоченного набора — отсортированный Array с бинарным поиском.

#### Хороший пример

```js
const insert = (node, value) => {
  if (!node) return { value, left: null, right: null };
  if (value < node.value) node.left = insert(node.left, value);
  else if (value > node.value) node.right = insert(node.right, value);
  return node; // дубликаты игнорируются
};
const find = (node, value) => {
  if (!node) return false;
  if (node.value === value) return true;
  return find(value < node.value ? node.left : node.right, value);
};
let root = null;
for (const value of [4, 2, 6, 2]) root = insert(root, value);
console.log(find(root, 6), find(root, 3)); // true false
```

#### Плохой пример

```js
const root = { value: 4, left: null, right: { value: 2 } };
const find = (node, value) => {
  if (!node) return false;
  if (node.value === value) return true;
  return find(value < node.value ? node.left : node.right, value);
};
console.log(find(root, 2)); // false: узел вставлен не в ту ветку
```

#### В чём разница

Вставка и поиск используют одно правило порядка и явную политику duplicates. Несогласованная вставка делает существующее значение недоступным. Этот BST не балансируется: worst-case поиск и глубина рекурсии линейны.

### B-tree

Упоминание в description структур данных.

#### Когда применять

Большой упорядоченный индекс должен эффективно работать с блоками хранения и поддерживать поиск и изменения; подходят проверенные B-tree реализации с контролем балансировки.

#### Когда избыточен

Данные малы, помещаются в простой список или нужен только точный поиск в памяти. Собственная реализация split/merge создаёт значительный риск ошибок.

#### Более простое решение

Использовать индекс существующей DB. В памяти выбрать Map для точного ключа или отсортированный Array для небольшого упорядоченного набора; не реализовывать B-tree по учебному фрагменту.

#### Хороший пример

```js
// Фрагмент модели узла и поиска, не полная реализация B-tree.
const root = {
  keys: [10, 20],
  children: [
    { keys: [2, 5], children: [] },
    { keys: [12, 15], children: [] },
    { keys: [25, 30], children: [] },
  ],
};
const has = (node, key) => {
  let i = 0;
  while (i < node.keys.length && key > node.keys[i]) i++;
  if (node.keys[i] === key) return true;
  if (!node.children.length) return false;
  return has(node.children[i], key);
};
console.log(has(root, 15), has(root, 17)); // true false
```

#### Плохой пример

```js
const root = {
  keys: [10, 20],
  children: [
    { keys: [2, 5], children: [] },
    { keys: [25, 30], children: [] },
  ],
};
console.log(root.children.length === root.keys.length + 1); // false
// Нет третьего диапазона children: поиск не может корректно выбрать ветку.
```

#### В чём разница

Внутренний узел с k ключами должен разделять k+1 диапазонов. Плохая модель нарушает даже этот структурный invariant. Для полноценного B-tree дополнительно нужны ограничения заполненности, одинаковая глубина листьев и корректные split/merge; пример их не реализует. В Metaskills B-tree лишь кратко упомянут.

### Heap / Priority Queue

Извлекать элемент с минимальным или максимальным приоритетом.

#### Когда применять

Нужно многократно добавлять элементы и извлекать следующий минимум/максимум, не сортируя всю коллекцию после каждого изменения.

#### Когда избыточен

Минимум требуется один раз, набор маленький или достаточно одной полной сортировки перед обработкой.

#### Более простое решение

Один раз найти минимум линейным проходом либо отсортировать Array и последовательно пройти его. При постоянной работе с приоритетами использовать готовую priority queue.

#### Хороший пример

```js
const heap = [];
const push = (value) => {
  heap.push(value);
  let i = heap.length - 1;
  while (i > 0) {
    const parent = Math.floor((i - 1) / 2);
    if (heap[parent] <= heap[i]) break;
    [heap[parent], heap[i]] = [heap[i], heap[parent]];
    i = parent;
  }
};
push(5); push(2); push(1);
console.log(heap[0]); // 1: min-heap после каждой вставки
```

#### Плохой пример

```js
const heap = [];
const push = (value) => heap.push(value);
push(5); push(2); push(1);
console.log(heap[0]); // 5 вместо минимального 1
```

#### В чём разница

Sift-up восстанавливает invariant parent ≤ child. Обычный append создаёт массив, но не heap. Здесь показана вставка; извлечение корня требует отдельного sift-down, а не простого shift.

### Trie

Искать по строковому префиксу.

#### Когда применять

Основные запросы связаны с префиксами строк: autocomplete, поиск всех продолжений или маршрутизация по последовательности символов.

#### Когда избыточен

Нужен только точный поиск ключа, набор мал либо prefix-запросы редки. Узлы trie могут занимать больше памяти, чем сами строки.

#### Более простое решение

Для точных ключей использовать Map/Set; для небольшого autocomplete — filter со startsWith по массиву. Усложнять индекс после проверки реальной нагрузки.

#### Хороший пример

```js
const root = { terminal: false, children: new Map() };
const insert = (word) => {
  let node = root;
  for (const char of word) {
    if (!node.children.has(char)) {
      node.children.set(char, { terminal: false, children: new Map() });
    }
    node = node.children.get(char);
  }
  node.terminal = true;
};
const has = (word) => {
  let node = root;
  for (const char of word) {
    node = node.children.get(char);
    if (!node) return false;
  }
  return node.terminal;
};
insert('car');
console.log(has('car'), has('ca')); // true false
```

#### Плохой пример

```js
const root = { c: { a: { r: {} } } };
const has = (word) => {
  let node = root;
  for (const char of word) {
    node = node[char];
    if (!node) return false;
  }
  return true; // любой путь считается словом
};
console.log(has('ca')); // true: добавляли только car
```

#### В чём разница

Terminal отделяет завершённое слово от существующего префикса. Без этого флага все префиксы автоматически становятся ключами.

### Graph / adjacency list + BFS

Обходить связи по слоям.

#### Когда применять

Модель содержит произвольные связи, требуется обход по слоям или кратчайшее число рёбер в невзвешенном графе. Циклы и повторные пути требуют контроля visited.

#### Когда избыточен

Структура — простой список, нужно только обработать прямых соседей или задача не требует порядка BFS. Полная модель графа и универсальный обход тогда лишние.

#### Более простое решение

Использовать прямой проход по массиву или одну операцию над соседями. Для обычного дерева достаточно специализированного обхода; для поиска пути со взвешенными рёбрами нужен другой алгоритм.

#### Хороший пример

```js
const graph = new Map([['A', ['B']], ['B', ['A']]]);
const queue = ['A'];
const visited = new Set(queue);
for (let i = 0; i < queue.length; i++) {
  for (const next of graph.get(queue[i]) ?? []) {
    if (visited.has(next)) continue;
    visited.add(next);
    queue.push(next);
  }
}
console.log(queue); // ['A', 'B']
```

#### Плохой пример

```js
const graph = new Map([['A', ['B']], ['B', ['A']]]);
const queue = ['A'];
// Ограничение 5 не даёт учебному примеру зациклиться.
for (let i = 0; i < queue.length && i < 5; i++) {
  queue.push(...(graph.get(queue[i]) ?? []));
}
console.log(queue); // A, B, A, B, A, B
```

#### В чём разница

Visited отмечается при постановке в очередь, чтобы каждая вершина добавлялась один раз. Без него цикл графа порождает бесконечный обход; искусственный лимит лишь обрывает ошибочную работу.

### Iterator

Единый способ последовательного обхода.

#### Когда применять

Caller должен последовательно обходить данные, не зная их внутреннего представления; полезны lazy traversal или поддержка for...of.

#### Когда избыточен

Коллекция уже является Array или готовым iterable, а новый iterator лишь дублирует существующий обход. Также он не заменяет необходимость random access.

#### Более простое решение

Вернуть существующий iterable или пройти обычный массив. Для одного специального обхода достаточно generator-функции без отдельного класса iterator.

#### Хороший пример

```js
class Sequence {
  constructor(values) { this.values = values; }
  *[Symbol.iterator]() { yield* this.values; }
}
const collection = new Sequence([1, 2, 3]);
let sum = 0;
for (const value of collection) sum += value;
console.log(sum); // 6: caller использует публичный обход
```

#### Плохой пример

```js
const oldCollection = { buffer: [1, 2, 3] };
const sum = (collection) => collection.buffer.reduce((total, n) => total + n, 0);
console.log(sum(oldCollection)); // 6
const newCollection = { head: { value: 1, next: null } };
// sum(newCollection); // TypeError: caller привязан к внутреннему buffer
```

#### В чём разница

Iterator отделяет способ обхода от хранения. Caller, читающий внутренний buffer, ломается при замене array на linked representation, даже если смысл коллекции прежний.

### LRU Cache

Удалять давно не использовавшуюся запись.

#### Когда применять

Есть ограниченный кэш, недавние обращения с разумной вероятностью повторяются, а вытеснение по recency соответствует требованиям нагрузки.

#### Когда избыточен

Данные дешевле пересчитать, набор статичен, повторных обращений мало или важнее возраст данных, чем recency. Поддерживать LRU без выигрыша не нужно.

#### Более простое решение

Убрать кэш; для короткой операции использовать локальную memoization. Если важнее срок свежести — выбрать явную TTL-политику, если порядок добавления — FIFO.

#### Хороший пример

```js
const cache = new Map([['A', 1], ['B', 2]]);
const get = (key) => {
  if (!cache.has(key)) return undefined;
  const value = cache.get(key);
  cache.delete(key);
  cache.set(key, value);
  return value;
};
get('A');
cache.set('C', 3);
cache.delete(cache.keys().next().value);
console.log([...cache.keys()]); // ['A', 'C']
```

#### Плохой пример

```js
const cache = new Map([['A', 1], ['B', 2]]);
const get = (key) => cache.get(key); // порядок не обновляется
get('A');
cache.set('C', 3);
cache.delete(cache.keys().next().value);
console.log([...cache.keys()]); // ['B', 'C']: недавно прочитанную A удалили
```

#### В чём разница

LRU обновляет recency при чтении. Плохой вариант удаляет по порядку вставки, то есть реализует другую политику. Для полной реализации нужно согласовать также обновление существующего ключа через set.

### Struct / запись со схемой

Контролировать поля, типы и мутабельность.

#### Когда применять

В нескольких местах создаются записи с одинаковыми полями, defaults, проверкой типов и определённой mutable/immutable-семантикой.

#### Когда избыточен

Есть одна простая запись, схема уже проверяется на границе или framework предоставляет нужную модель. Собственный универсальный Struct дублирует эту проверку.

#### Более простое решение

Использовать object literal или небольшую factory с явной валидацией. В TypeScript описать статический тип, сохранив runtime-проверку недоверенных данных.

#### Хороший пример

```js
const createRecord = (input = {}) => {
  for (const key of Object.keys(input)) {
    if (key !== 'name' && key !== 'tags') throw new TypeError('Unknown field');
  }
  const name = input.name ?? '';
  const tags = input.tags ?? [];
  if (typeof name !== 'string' || !Array.isArray(tags)) {
    throw new TypeError('Invalid fields');
  }
  return Object.seal({ name, tags: [...tags] });
};
const first = createRecord();
const second = createRecord();
first.tags.push('new');
console.log(second.tags); // []
```

#### Плохой пример

```js
const defaults = { name: '', tags: [] };
const createRecord = (input = {}) => ({ ...defaults, ...input });
const first = createRecord();
const second = createRecord();
first.tags.push('new');
console.log(second.tags); // ['new']: mutable default общий
console.log(createRecord({ nam: 123 })); // опечатка и неверный тип приняты
```

#### В чём разница

Создание проверяет поля и отделяет mutable defaults между экземплярами. Spread defaults не валидирует схему и не копирует вложенные массивы. Seal фиксирует набор полей, но не делает запись глубоко неизменяемой.

### Prototype overlay / branch

Создавать вариант с общим базовым состоянием.

#### Когда применять

Нужно создавать варианты стабильной базы с небольшим числом отличий, а наследуемое чтение и sharing намеренно входят в контракт.

#### Когда избыточен

Нужна независимая копия, обычная JSON-сериализация всех полей или база активно мутирует. Prototype lookup скрывает зависимости между версиями.

#### Более простое решение

Для небольшой записи сделать spread с явными overrides; для конфигурации можно хранить base и overrides отдельно и разрешать значения одной функцией.

#### Хороший пример

```js
const base = Object.freeze({ color: 'red', size: 10 });
const variant = Object.create(base, {
  color: { value: 'blue', enumerable: true },
});
console.log(variant.color, variant.size); // blue 10
console.log(Object.hasOwn(variant, 'size')); // false: sharing явный
```

#### Плохой пример

```js
const base = { color: 'red', size: 10 };
const independentCopy = Object.create(base);
base.size = 20;
console.log(independentCopy.size); // 20: это overlay, не независимая копия
```

#### В чём разница

Overlay намеренно наследует значения base. Замороженная база делает это sharing предсказуемым в показанной скалярной схеме. Назвать Object.create(base) независимой копией нельзя: чтение продолжает зависеть от prototype.

### CRDT G-Counter

Сливать монотонные счётчики реплик.

#### Когда применять

Несколько реплик независимо увеличивают счётчики, состояния могут приходить повторно и не по порядку, а после merge требуется сходимость без центрального инкремента.

#### Когда избыточен

Есть один автор записи или доступна центральная транзакционная точка. Также G-Counter не подходит для уменьшений, reset и произвольного изменения значения.

#### Более простое решение

Использовать локальный counter либо атомарный increment в общей DB. При необходимости decrement выбрать другой согласованный контракт, не модифицировать G-Counter простым вычитанием.

#### Хороший пример

```js
const merge = (local, remote) => local.map((value, i) => Math.max(value, remote[i]));
const anna = [2, 0];
const boris = [0, 3];
const once = merge(anna, boris);
const twice = merge(once, boris);
console.log(once, twice); // [2, 3] [2, 3]
console.log(twice.reduce((sum, n) => sum + n, 0)); // 5
```

#### Плохой пример

```js
const merge = (local, remote) => local.map((value, i) => value + remote[i]);
const once = merge([2, 0], [0, 3]);
const twice = merge(once, [0, 3]);
console.log(once, twice); // [2, 3] [2, 6]: повторная доставка учтена дважды
```

#### В чём разница

Покомпонентный max делает merge идемпотентным. Суммирование состояний повторно учитывает уже известные increments. Пример предполагает фиксированный согласованный набор реплик и неотрицательные монотонные компоненты.

Встроенные и библиотечные варианты одной структуры могут иметь разные
empty-value, iteration и mutation contracts. Например, `undefined` из dequeue
неотличим от сохранённого `undefined`, если API не предоставляет дополнительный
признак присутствия. Эти различия нельзя устранять механической заменой значений.

## 6. Ошибки и восстановление

### Классификация ошибок

Различать ошибки программы и ожидаемые сбои среды.

#### Когда применять

От вида ошибки зависит действие: повторить операцию, вернуть согласованный отказ, перейти на fallback или прекратить работу из-за программного сбоя.

#### Когда избыточен

Отдельная иерархия классов и большой каталог кодов не нужны, если в локальной операции все ошибки одинаково передаются выше. Само различение корректного и некорректного восстановления остаётся необходимым.

#### Более простое решение

Передавать исходную ошибку без catch либо проверять один документированный error.code там, где действительно возможно восстановление.

#### Хороший пример

```js
let calls = 0;
const load = async () => {
  calls++;
  throw new TypeError('Broken application logic');
};
const run = async () => {
  for (let attempt = 1; attempt <= 3; attempt++) {
    try { return await load(); }
    catch (error) {
      if (error.code !== 'ETIMEDOUT' || attempt === 3) throw error;
    }
  }
};
try { await run(); } catch (error) { console.log(error.name, calls); }
// TypeError 1: программная ошибка не повторяется
```

#### Плохой пример

```js
let calls = 0;
const load = async () => { calls++; throw new TypeError('Broken logic'); };
const run = async () => {
  for (let attempt = 1; attempt <= 3; attempt++) {
    try { return await load(); }
    catch { /* повторяем любую ошибку */ }
  }
  return null;
};
console.log(await run(), calls); // null 3: баг замаскирован как отсутствие данных
```

#### В чём разница

Классификация определяет разрешённое восстановление. TypeError не исправляется повтором; первый вариант передаёт его выше. Второй тратит попытки и скрывает ошибку. Конкретные transient codes зависят от используемой библиотеки.

### Перевод ошибки на границе

Преобразовать внутреннюю ошибку в публичный код.

#### Когда применять

Внутренняя ошибка пересекает публичную или межсервисную границу; внешнему потребителю нужны стабильный код и допустимый объём деталей.

#### Когда избыточен

Код находится внутри одного компонента и ошибка не меняет смысл. Перевод на каждом внутреннем слое усложняет трассировку и может потерять исходную причину.

#### Более простое решение

Передавать исходную ошибку до одной существующей API-границы и переводить её там стандартным обработчиком framework.

#### Хороший пример

```js
const toResponse = (error) => {
  if (error.code === 'USER_NOT_FOUND') {
    return { status: 404, body: { code: 'ENOTFOUND' } };
  }
  return { status: 500, body: { code: 'EINTERNAL' } };
};
const error = Object.assign(new Error('SELECT failed on db.internal'), {
  code: 'DB_FAILURE',
});
console.log(toResponse(error)); // 500, только публичный код
```

#### Плохой пример

```js
const toResponse = (error) => ({
  status: 500,
  body: { message: error.message, stack: error.stack },
});
const error = new Error('SELECT failed on db.internal');
console.log(toResponse(error).body.message); // внутренние детали в API
```

#### В чём разница

Граница API выдаёт согласованные публичные коды, сохраняя внутренние детали для внутреннего наблюдения. Прямая сериализация message/stack раскрывает устройство сервиса и делает внешних клиентов зависимыми от внутренних формулировок.

### Propagation / владелец обработки

Восстановить, перевести или передать ошибку дальше.

#### Когда применять

Нужно решить, какой слой может восстановиться, добавить контекст или перевести ошибку, а какой должен просто передать её дальше.

#### Когда избыточен

В локальной функции нет полезного восстановления или перевода: catch с немедленным throw той же ошибки не добавляет поведения.

#### Более простое решение

Убрать пустой catch и позволить исключению или rejected Promise распространиться. Cleanup оставить в finally; наблюдение — у выбранного владельца.

#### Хороший пример

```js
const listUsers = async (repository) => {
  try { return await repository.list(); }
  catch (error) {
    if (error.code === 'OPTIONAL_SOURCE_UNAVAILABLE') return [];
    throw error;
  }
};
const repository = { list: async () => { throw new Error('Database offline'); } };
try { await listUsers(repository); }
catch (error) { console.log(error.message); } // Database offline
```

#### Плохой пример

```js
const listUsers = async (repository) => {
  try { return await repository.list(); }
  catch { return []; }
};
const repository = { list: async () => { throw new Error('Database offline'); } };
console.log(await listUsers(repository)); // []: выглядит как «пользователей нет»
```

#### В чём разница

Восстановление разрешено только для согласованного необязательного источника. Сбой основной DB должен остаться ошибкой. Универсальный пустой результат может вызвать неверные решения вызывающего кода, например очистку данных как «больше не существующих».

### Retry + exponential backoff

Повторить временно неуспешную операцию.

#### Когда применять

Есть известный временный сбой, повтор безопасен, число попыток и время ожидания ограничены, а зависимость не выполняет тот же retry самостоятельно.

#### Когда избыточен

Ошибка постоянная, предыдущая запись могла успешно завершиться без идемпотентности или повтор только увеличивает перегрузку. Второй retry поверх клиентского также может быть лишним.

#### Более простое решение

Выполнить одну попытку и вернуть явную ошибку; при наличии подходящей встроенной политики настроить retry клиента вместо собственного цикла. Для writes сначала обеспечить безопасный контракт повторения.

#### Хороший пример

```js
const sleep = (ms) => new Promise((resolve) => setTimeout(resolve, ms));
const retryRead = async (read, { attempts = 3, delayMs = 1 } = {}) => {
  for (let attempt = 1; ; attempt++) {
    try { return await read(); }
    catch (error) {
      if (error.code !== 'ETIMEDOUT' || attempt >= attempts) throw error;
      await sleep(Math.min(100, delayMs * 2 ** (attempt - 1)));
    }
  }
};
let calls = 0;
const value = await retryRead(async () => {
  if (++calls < 2) throw Object.assign(new Error('Timeout'), { code: 'ETIMEDOUT' });
  return 42;
});
console.log(value, calls); // 42 2
```

#### Плохой пример

```js
let charges = 0;
const charge = async () => {
  charges++; // платёж уже выполнен, но ответ потерян
  throw new Error('Timeout after commit');
};
for (let attempt = 0; attempt < 2; attempt++) {
  try { await charge(); break; }
  catch { /* повтор без idempotency key */ }
}
console.log(charges); // 2 списания
```

#### В чём разница

Первый helper предназначен для безопасного чтения и повторяет только известный transient failure с ограниченным backoff. Повтор записи после потери ответа может удвоить эффект. Учебный helper не добавляет общий deadline/cancellation: при их необходимости политика должна передать сигнал и ограничить весь бюджет операции.

### Process-level failure handling

Зафиксировать необработанный сбой процесса.

#### Когда применять

Приложение владеет процессом и должно определить политику необработанного fatal failure: фиксацию причины, exit status и взаимодействие с supervisor.

#### Когда избыточен

Обработчик добавляется библиотекой в чужой процесс либо runtime/framework уже задаёт нужную политику. Дублирующие process handlers могут конфликтовать.

#### Более простое решение

Использовать штатное аварийное завершение runtime и supervisor; библиотеке передавать ошибку caller, а не устанавливать глобальные handlers. Дополнительную диагностику подключать в одной точке запуска приложения.

#### Хороший пример

```js
// Тестовая модель supervisor API: настоящий процесс не завершается.
const actions = [];
const runtime = { exit: (code) => actions.push(`exit:${code}`) };
const onFatal = (error) => {
  actions.push(`fatal:${error.message}`);
  runtime.exit(1);
};
onFatal(new Error('Corrupted state'));
console.log(actions); // ['fatal:Corrupted state', 'exit:1']
// В приложении onFatal подключается к process.on('uncaughtException', ...).
```

#### Плохой пример

```js
const actions = [];
const onFatal = (error) => actions.push(`fatal:${error.message}`);
onFatal(new Error('Corrupted state'));
actions.push('accept next request');
console.log(actions); // после fatal процесс продолжил обслуживать запросы
```

#### В чём разница

Необработанный программный сбой может нарушить инварианты процесса. Хороший сценарий сигнализирует об ошибочном завершении supervisor, плохой только пишет лог и продолжает работу. Это модель политики: реальный путь должен согласовать ограниченное завершение, синхронную фиксацию ошибки и exit status.

### Graceful Shutdown

Прекратить приём работы и завершить текущую.

#### Когда применять

Долгоживущий сервис принимает конкурентную работу и владеет ресурсами, которые нельзя закрывать до завершения активных операций.

#### Когда избыточен

Отдельный coordinator не нужен короткой CLI-команде без фоновых операций или приложению, где framework уже обеспечивает требуемый shutdown. Дублировать lifecycle не стоит.

#### Более простое решение

Использовать штатные framework hooks и закрытие ресурсов в try/finally. Для одного ресурса достаточно одного ограниченного close; для сервиса сохранить stop admission и deadline draining.

#### Хороший пример

```js
const events = [];
let finish;
const activeRequest = new Promise((resolve) => { finish = resolve; });
const server = { stopAdmission: () => events.push('stop admission') };
const shutdown = async () => {
  server.stopAdmission();
  await activeRequest; // в production ожидание ограничено deadline
  events.push('close database');
};
const closing = shutdown();
events.push('request completed');
finish();
await closing;
console.log(events); // stop admission → request completed → close database
```

#### Плохой пример

```js
const events = [];
const shutdown = () => {
  events.push('destroy active socket');
  events.push('close database');
};
shutdown();
events.push('request tries to commit');
console.log(events); // запрос продолжил работу после закрытия его ресурсов
```

#### В чём разница

Штатное завершение запрещает новую работу, ожидает активную и только потом закрывает зависимости. Немедленное уничтожение ресурсов обрывает текущие операции. В production нужен deadline с отдельным принудительным сценарием: ожидать зависший запрос бесконечно тоже неправильно.

Retry-пример источника не реализует фильтрацию transient errors; shutdown-пример
немедленно закрывает соединения. Хорошие примеры в этом реестре показывают
требуемое поведение решения, а не утверждают корректность исходных реализаций.

## 7. Организация кода и владение данными

### Data first

Сначала определить отношение данных, затем алгоритм.

#### Когда применять

Корректность алгоритма зависит от связей данных: уникальности, порядка, ключей, identity и принадлежности состояния.

#### Когда избыточен

Избыточно превращать выбор очевидного встроенного типа в отдельный этап проектирования с универсальной моделью. Само понимание данных не отменяется.

#### Более простое решение

Выбрать минимальный literal, Array, Map или Set и записать только существенный инвариант. Не вводить дополнительный data layer без отдельной необходимости.

#### Хороший пример

```js
const orders = [{ id: 1, total: 10 }, { id: 1, total: 10 }, { id: 2, total: 5 }];
const byId = new Map();
for (const order of orders) byId.set(order.id, order);
const total = [...byId.values()].reduce((sum, order) => sum + order.total, 0);
console.log(total); // 15: один заказ на ID
```

#### Плохой пример

```js
const ids = [1, 1, 2];
const totals = [10, 10, 5];
const uniqueIds = [...new Set(ids)];
const total = uniqueIds.reduce((sum, id, index) => sum + totals[index], 0);
console.log(total); // 20: после dedup индексы больше не соответствуют заказам
```

#### В чём разница

Представление Map сохраняет связь ID с записью до вычисления. Независимое преобразование параллельных массивов разрушает соответствие. Выбор поведения при конфликтующих записях одного ID нужно задать отдельно: здесь побеждает последняя.

### Named steps

Сделать промежуточные смыслы явными.

#### Когда применять

Выражение смешивает несколько смыслов, порядок вычислений трудно увидеть или промежуточные значения нужно независимо проверить.

#### Когда избыточен

Выражение короткое и очевидное, а промежуточные переменные лишь повторяют синтаксис без нового смысла.

#### Более простое решение

Оставить прямой return, короткую цепочку или понятное условие. Выделять имя для самостоятельного понятия, а не для каждого оператора.

#### Хороший пример

```js
const items = [
  { name: 'Anna', visible: true, selected: false },
  { name: 'Boris', visible: false, selected: true },
];
const visible = items.filter((item) => item.visible);
const labels = visible.map((item) => item.selected ? `* ${item.name}` : item.name);
console.log(labels.join(', ')); // Anna
```

#### Плохой пример

```js
const items = [
  { name: 'Anna', visible: true, selected: false },
  { name: 'Boris', visible: false, selected: true },
];
console.log(items.filter((x) => x.visible).map((x) => x.selected ? `* ${x.name}` : x.name).join(', '));
// Anna — результат тот же, но этапы скрыты в одной длинной строке.
```

#### В чём разница

Здесь плохой вариант не меняет результат. Проблема в сопровождении: критерий видимости, форматирование и сборка смешаны в одном выражении; сложнее проверить промежуточное значение и локально изменить один этап. Короткие понятные цепочки сами по себе допустимы.

### Split then compose

Разделить самостоятельные этапы и собрать их явно.

#### Когда применять

Операция содержит самостоятельные этапы с разными правилами или повторяющуюся семантику, которую нужно проверять и изменять отдельно.

#### Когда избыточен

Разбиение создаёт функции на каждый тривиальный шаг, прячет линейный порядок или вынуждает передавать большой мешок промежуточного состояния.

#### Более простое решение

Оставить одну последовательную функцию с несколькими понятными локальными переменными; извлечь только действительно самостоятельный алгоритм.

#### Хороший пример

```js
const tokenize = (text) => text.trim().split(/\s+/);
const pack = (tokens) => ({ tokens, count: tokens.length });
const parse = (text) => pack(tokenize(text));
console.log(parse(' one   two ')); // tokens: ['one', 'two'], count: 2
```

#### Плохой пример

```js
const parse = (text) => ({
  tokens: text.trim().split(/\s+/),
  count: text.split(' ').length,
});
console.log(parse(' one   two ')); // tokens: 2 элемента, count: 6
```

#### В чём разница

Декомпозиция отделяет самостоятельное правило токенизации от упаковки результата. Во втором варианте правило повторяется по-разному, поэтому count не соответствует tokens. Не требуется выделять функцию для каждого тривиального действия.

### Локализация семантической сложности

Держать одну систему правил в одном месте.

#### Когда применять

Одно бизнес-правило, набор aliases или протокольных случаев повторяется и должен изменяться согласованно.

#### Когда избыточен

Сходство только синтаксическое, но правила принадлежат разным доменам и должны меняться независимо. Общая абстракция свяжет их искусственно.

#### Более простое решение

Для единственного правила оставить локальное условие; для одного общего набора — небольшую таблицу или функцию без framework правил.

#### Хороший пример

```js
const aliases = new Map([['js', 'javascript'], ['ts', 'typescript']]);
const normalizeLanguage = (name) => aliases.get(name) ?? name;
const validate = (name) => ['javascript', 'typescript'].includes(normalizeLanguage(name));
const save = (name) => ({ language: normalizeLanguage(name) });
console.log(validate('ts'), save('ts')); // true, language: typescript
```

#### Плохой пример

```js
const validate = (name) => ['js', 'javascript', 'ts', 'typescript'].includes(name);
const save = (name) => ({ language: name === 'js' ? 'javascript' : name });
console.log(validate('ts'), save('ts')); // true, language: ts
```

#### В чём разница

Правило aliases живёт в одном месте и используется разными операциями. При локальных списках валидатор уже знает ts, а сохранение ещё не нормализует его. Это не повод строить общий framework правил: одной таблицы достаточно.

### Владение входами

Мутировать только принадлежащее операции состояние.

#### Когда применять

Функция получает разделяемые данные или обещает вычислить результат без изменения состояния caller. Нужно явно определить разрешённые мутации.

#### Когда избыточен

Избыточно копировать данные, которыми операция уже владеет исключительно, или сохранять immutable-форму вопреки согласованному in-place контракту. Ownership при этом всё равно должен быть понятен.

#### Более простое решение

Мутировать локальную рабочую коллекцию или явно названный in-place аргумент. Если меняется только порядок массива, достаточно shallow copy, а не deep clone.

#### Хороший пример

```js
const users = [{ score: 1 }, { score: 3 }];
const rank = (items) => [...items].sort((a, b) => b.score - a.score);
const ranked = rank(users);
console.log(ranked[0].score, users[0].score); // 3 1
```

#### Плохой пример

```js
const users = [{ score: 1 }, { score: 3 }];
const rank = (items) => items.sort((a, b) => b.score - a.score);
const ranked = rank(users);
console.log(ranked[0].score, users[0].score); // 3 3: вход изменился
```

#### В чём разница

Если rank обещает вернуть новый порядок, он должен владеть сортируемым массивом. Sort мутирует исходный массив. Для согласованной in-place операции второй вариант допустим, но caller должен знать этот контракт. Копирование массива не копирует объекты пользователей.

### Сохранение порядка эффектов

Вычислять только необходимую ветку.

#### Когда применять

Рефакторинг затрагивает условные вызовы, short-circuit, IO, изменение состояния или обработку ошибок; порядок является частью поведения.

#### Когда избыточен

Дополнительный механизм управления эффектами не нужен для простой ветки. Сохранение самого поведения не бывает необязательным.

#### Более простое решение

Оставить явный if/else или короткий тернарный оператор, вычисляя только нужную ветку. Не выносить эффектные вызовы заранее ради оформления.

#### Хороший пример

```js
const calls = [];
const load = () => { calls.push('load'); return 'data'; };
const fallback = () => { calls.push('fallback'); return 'cached'; };
const ready = true;
const result = ready ? load() : fallback();
console.log(result, calls); // data ['load']
```

#### Плохой пример

```js
const calls = [];
const load = () => { calls.push('load'); return 'data'; };
const fallback = () => { calls.push('fallback'); return 'cached'; };
const loaded = load();
const cached = fallback();
const result = true ? loaded : cached;
console.log(result, calls); // data ['load', 'fallback']
```

#### В чём разница

Тернарное выражение вычисляет только выбранную ветку. Вынос обоих вызовов в переменные изменяет семантику: fallback запускается без необходимости. В реальном коде это может означать лишний запрос или побочную запись.

### Composition over inheritance

Собирать вариативное поведение делегированием.

#### Когда применять

Компонент использует чужую возможность, но не является корректным подтипом её реализации; поведения нужно заменять независимо.

#### Когда избыточен

Один прямой вызов не требует объекта-компоновщика; также наследование может уже корректно выражать устойчивое отношение подтипов и framework extension point.

#### Более простое решение

Передать функцию, вызвать модуль напрямую или оставить существующее корректное наследование. Не строить несколько wrapper-слоёв только ради принципа композиции.

#### Хороший пример

```js
class UserService {
  constructor(transport) { this.transport = transport; }
  load(id) { return this.transport.get(`/users/${id}`); }
}
const service = new UserService({ get: (path) => `GET ${path}` });
console.log(service.load(1)); // GET /users/1
console.log(typeof service.close); // undefined: чужой API не выставлен
```

#### Плохой пример

```js
class Transport {
  get(path) { return `GET ${path}`; }
  close() { this.closed = true; }
}
class UserService extends Transport {
  load(id) { return this.get(`/users/${id}`); }
}
const service = new UserService();
service.close(); // caller получил транспортное управление через domain API
```

#### В чём разница

Сервис использует transport, но не является transport. Композиция сохраняет небольшой публичный контракт. Наследование в примере выставляет нерелевантные операции и связывает развитие domain API с транспортным классом. Корректное отношение подтипов может оправдывать наследование.

### Явные зависимости / IoC и DI

Сделать окружение заменяемым и проверяемым.

#### Когда применять

Время, IO, хранилище или внешнее поведение нужно подменять, разделять между окружениями либо контролировать по lifecycle.

#### Когда избыточен

DI-контейнер и интерфейсы на каждую функцию лишние для небольшой локальной операции или стабильной чистой зависимости.

#### Более простое решение

Передать одну функцию или объект параметром, использовать небольшую factory с closure. Для неизменяемой чистой utility допустим прямой import.

#### Хороший пример

```js
const createSession = ({ now, save }) => ({
  start(userId) {
    const session = { userId, createdAt: now() };
    save(session);
    return session;
  },
});
const stored = [];
const service = createSession({ now: () => 1000, save: (value) => stored.push(value) });
console.log(service.start('anna').createdAt); // 1000: детерминированный тест
```

#### Плохой пример

```js
const storage = [];
const startSession = (userId) => {
  const session = { userId, createdAt: Date.now() };
  storage.push(session);
  return session;
};
console.log(startSession('anna').createdAt); // зависит от реальных часов
// Caller не может передать другое хранилище или фиксированные часы.
```

#### В чём разница

Первый сервис получает необходимые возможности явно. Тест контролирует время и сохранение без подмены глобальных объектов. Во втором окружение скрыто внутри реализации. DI не требует контейнера или framework: здесь достаточно аргументов factory.

### Разделение domain/system code

Отделить правила предметной области от IO-механики.

#### Когда применять

Доменное правило требуется проверять и использовать независимо от HTTP, DB и транспорта либо оно уже смешано с деталями IO.

#### Когда избыточен

Формальные слои, DTO-переходы и отдельные пакеты не нужны маленькому сценарию без самостоятельных доменных правил.

#### Более простое решение

Оставить короткий handler и выделить одну чистую функцию вычисления. Не вводить repository/service/use-case слои, если они только пересылают вызовы.

#### Хороший пример

```js
const discountedTotal = (price, discount) => price * (1 - discount);
const handle = async (request, loadPrice) => {
  const price = await loadPrice(request.id);
  return { total: discountedTotal(price, request.discount) };
};
console.log(discountedTotal(100, 0.2)); // 80 без IO
console.log(await handle({ id: 1, discount: 0.2 }, async () => 100));
```

#### Плохой пример

```js
const discountedTotal = async (request, response, database) => {
  const price = await database.loadPrice(request.id);
  response.end(`${price * (1 - request.discount)}`);
};
const output = [];
await discountedTotal({ id: 1, discount: 0.2 },
  { end: (value) => output.push(value) }, { loadPrice: async () => 100 });
// Чтобы проверить формулу, приходится собирать HTTP и DB окружение.
```

#### В чём разница

Чистая доменная формула проверяется отдельно от транспорта и хранения. Во втором варианте вычисление, чтение DB и завершение response срослись: повторное использование формулы требует чужих интерфейсов. Разделение не обязывает вводить много архитектурных слоёв.

### Контракт важнее механического reuse

Повторно использовать библиотеку только при совпадении поведения.

#### Когда применять

Планируется заменить локальный код библиотечным helper или объединить реализации; нужно сравнить sync/async, ошибки, defaults, units и ownership.

#### Когда избыточен

Избыточен отдельный процесс аудита для очевидно идентичной чистой операции. Проверка существенного контракта всё равно нужна при изменении поведения.

#### Более простое решение

Оставить небольшой локальный helper, если библиотека делает другое; при совпадении контракта вызвать её напрямую без универсального adapter.

#### Хороший пример

```js
const readText = () => '{"id":1}';
const parseConfig = () => JSON.parse(readText());
console.log(parseConfig().id); // 1: оба шага синхронны
```

#### Плохой пример

```js
const readTextAsync = async () => '{"id":1}';
const parseConfig = () => JSON.parse(readTextAsync());
try { parseConfig(); }
catch (error) { console.log(error.name); } // SyntaxError: парсили Promise
```

#### В чём разница

Взаимозаменяемость определяется контрактом, а не похожим названием helper. Замена sync на async требует изменения всей цепочки callers. Аналогично нужно проверять UTC/local, throw/null и правила fallback.

GRASP, SOLID, contract programming, declarative DSL, referential transparency
и platform-agnostic code названы в источнике как ориентиры. Они не образуют
там самостоятельного подробного каталога. Например, для referential transparency
хорошо `const total = (items) => items.reduce((s, x) => s + x.price, 0)`;
плохо добавлять внутрь этой вычислительной функции скрытую мутацию глобальной
суммы и при этом ожидать одинаковый результат при одинаковом входе.

## 8. Производительность и измерение

Эти приёмы применяются с учётом целевой версии Node.js/V8 и измеренной нагрузки.

### Stable object shape

Стабилизировать представление однотипных записей.

#### Когда применять

Есть много однотипных записей и профиль указывает на горячие property accesses или allocation; согласованная инициализация не ухудшает модель данных.

#### Когда избыточен

Схема по смыслу динамическая, код не горячий либо стабилизация требует искусственных полей и усложняет API без измеренной пользы.

#### Более простое решение

Создавать обычные объекты с понятной схемой; для динамических ключей использовать Map. Не вводить классы и factories только ради предполагаемой V8-оптимизации.

#### Хороший пример

```js
const createPoint = (x, y, label = null) => ({ x, y, label });
const first = createPoint(1, 2);
const second = createPoint(3, 4, 'target');
console.log(Object.keys(first), Object.keys(second));
// ['x', 'y', 'label'] для обеих записей
```

#### Плохой пример

```js
const first = { x: 1, y: 2 };
const second = { y: 4, x: 3 };
second.label = 'target';
delete second.x;
second.x = 3;
console.log(Object.keys(first), Object.keys(second)); // разная форма и порядок
```

#### В чём разница

Первый вариант создаёт одинаковую схему однотипных записей сразу. Второй без необходимости меняет набор и порядок полей. Это потенциальная цена в горячем доступе и усложнение схемы, а не доказанный slowdown: эффект нужно измерять на целевой версии V8.

### Dense arrays / согласованный тип элементов

Не создавать ненужные holes и смены представления.

#### Когда применять

Данные являются плотной последовательностью, а случайные holes или неожиданные преобразования типов нарушают обход либо мешают измеренной горячей обработке.

#### Когда избыточен

Нужна настоящая разрежённая модель или типы по контракту разнородны. Искусственное заполнение огромного диапазона может расходовать лишнюю память.

#### Более простое решение

Использовать обычный Array с push; для небольшого набора не заботиться о micro-оптимизации. Для редких числовых индексов рассмотреть Map, не создавая плотный буфер.

#### Хороший пример

```js
const values = [];
for (let i = 0; i < 3; i++) values.push(i);
console.log(values.length, Object.keys(values)); // 3 ['0', '1', '2']
```

#### Плохой пример

```js
const values = [0, 1, 2];
values[1000000] = 3;
let visited = 0;
values.forEach(() => { visited++; });
console.log(values.length, visited); // 1000001 4: почти все индексы — holes
```

#### В чём разница

Плотная последовательность содержит значение на каждом индексе. Запись далеко за концом меняет length и создаёт holes; разные способы обхода и сериализации могут обрабатывать их по-разному. Разрежённый массив допустим, если именно это требуется модели данных.

### Monomorphic hot paths

Снизить вариативность типов на измеряемом участке.

#### Когда применять

Измеренная горячая операция регулярно получает однотипные данные, и можно поддерживать этот контракт без потери смысла модели.

#### Когда избыточен

Разные типы выражают реальную задачу или участок не влияет на производительность. Иерархии адаптеров ради мономорфизма могут стоить дороже.

#### Более простое решение

Оставить обычный полиморфный код с корректными проверками. Для числового вычисления достаточно валидировать или нормализовать вход на границе, не меняя всю архитектуру.

#### Хороший пример

```js
const records = [{ amount: 10 }, { amount: 20 }];
const total = records.reduce((sum, record) => sum + record.amount, 0);
console.log(total); // 30: числовой контракт
```

#### Плохой пример

```js
const records = [{ amount: 10 }, { amount: '20' }];
const total = records.reduce((sum, record) => sum + record.amount, 0);
console.log(total, typeof total); // '1020' string
```

#### В чём разница

Согласованный тип упрощает оптимизацию и сохраняет числовую семантику. Во втором примере есть уже ошибка корректности: + выполняет конкатенацию. Не всякий полиморфизм ошибочен; performance-вывод требует профиля и измерения.

### Снижение escaping allocations

Уменьшить создание объектов, переживающих итерацию.

#### Когда применять

Профиль показывает значимые allocation/GC costs, а одинаковые неизменяемые данные или callbacks создаются в часто выполняемом цикле и переживают его итерации.

#### Когда избыточен

Объекты короткоживущие, нагрузка мала или reuse потребует опасного разделения mutable state. Сам факт создания объекта не доказывает проблему.

#### Более простое решение

Оставить локальные allocations и GC. Для очевидно одинаковой неизменяемой таблицы достаточно вынести одну константу; pool добавлять только при отдельном обосновании.

#### Хороший пример

```js
const rules = Object.freeze({ read: 1, write: 2 });
const results = [];
for (const kind of ['read', 'write', 'read']) {
  results.push({ kind, rules });
}
console.log(new Set(results.map((item) => item.rules)).size); // 1 таблица
```

#### Плохой пример

```js
const results = [];
for (const kind of ['read', 'write', 'read']) {
  const rules = { read: 1, write: 2 };
  results.push({ kind, rules });
}
console.log(new Set(results.map((item) => item.rules)).size); // 3 копии
```

#### В чём разница

Неизменяемая таблица может безопасно разделяться. Во втором варианте каждая копия уходит в results и остаётся доступна после итерации, увеличивая удерживаемую память. Нельзя механически переиспользовать mutable результат, который должен быть независимым.

### Измерение до оптимизации

Проверить пользу на реальной нагрузке.

#### Когда применять

Решение принимается ради скорости или памяти и нужно сравнить корректные альтернативы на целевом runtime и workload.

#### Когда избыточен

Полный benchmark suite избыточен для изменения без performance-цели или для устранения очевидного лишнего действия. Отсутствие замера не позволяет обещать ускорение.

#### Более простое решение

Сначала использовать существующий profiler и реальные метрики; для узкой гипотезы достаточно небольшого воспроизводимого сравнения вместо новой benchmark-инфраструктуры.

#### Хороший пример

```js
import { performance } from 'node:perf_hooks';

const sum = (items) => items.reduce((total, n) => total + n, 0);
const values = Array.from({ length: 1000 }, (_, i) => i);
if (sum(values) !== 499500) throw new Error('Incorrect result');
for (let i = 0; i < 1000; i++) sum(values); // warmup
const samples = [];
let sink = 0;
for (let run = 0; run < 5; run++) {
  const start = performance.now();
  for (let i = 0; i < 1000; i++) sink += sum(values);
  samples.push(performance.now() - start);
}
samples.sort((a, b) => a - b);
console.log({ medianMs: samples[2], sink, runtime: process.version });
```

#### Плохой пример

```js
import { performance } from 'node:perf_hooks';
const values = [1, 2, 3];
const start = performance.now();
values.reduce((sum, n) => sum + n, 0);
console.log('Always fast:', performance.now() - start);
// Один запуск, нет baseline, warmup и проверки результата.
```

#### В чём разница

Хороший пример показывает проверку результата, повторение, warmup и медиану; для сравнения нужен второй вариант в сопоставимых условиях. Плохой делает общий вывод по шумному холодному вызову. Даже хороший микробенчмарк не заменяет профиль реального workload.

### Изоляция микробенчмарков

Не искажать сравнение общим состоянием call site.

#### Когда применять

Сравниваемые варианты могут влиять на общую историю JIT, GC или call sites, а цель — оценить каждый в изолированных условиях.

#### Когда избыточен

Измеряется реальная смешанная нагрузка, где общая история типов является частью сценария, либо измерение не связано с JIT-состоянием.

#### Более простое решение

Использовать один репрезентативный end-to-end benchmark или существующий harness. Отдельные процессы нужны для конкретной угрозы искажения, а не как обязательное правило всех измерений.

#### Хороший пример

```js
import { spawnSync } from 'node:child_process';

const cases = {
  numbers: 'const values = [1, 2, 3];',
  strings: "const values = ['1', '2', '3'];",
};
for (const [name, setup] of Object.entries(cases)) {
  const source = `${setup}
    const read = (item) => item;
    for (let i = 0; i < 10000; i++) read(values[i % values.length]);
    console.log(typeof read(values[0]));`;
  const result = spawnSync(process.execPath, ['-e', source], { encoding: 'utf8' });
  if (result.status !== 0) throw new Error(result.stderr);
  console.log(name, result.stdout.trim());
}
```

#### Плохой пример

```js
const read = (item) => item;
const cases = [[1, 2, 3], ['1', '2', '3']];
for (const values of cases) {
  for (let i = 0; i < 10000; i++) read(values[i % values.length]);
}
// Последующие замеры read зависят также от общей истории типов call site.
```

#### В чём разница

Отдельный процесс изолирует историю оптимизации каждого варианта. Общий harness может смешивать типы и влиять на последующие замеры. Фрагменты демонстрируют изоляцию, а не содержат полноценный сравнительный benchmark с таймером.


---

Реестр составлен на базе репозитория `metarhia/metaskills` на ревизии `ab13efe6c7ce56e8d576a3b7e42b90d408b68375`.
Дата составления: 2026-11-12.
