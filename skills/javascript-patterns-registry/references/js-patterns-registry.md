# Patterns Registry

The patterns in this registry apply to both JavaScript and TypeScript.
TypeScript can additionally express their contracts through interfaces,
generics, readonly, and discriminated unions; the principles governing objects,
data structures, resources, and errors remain the same.

When using these examples in TypeScript, keep in mind:

- Parameters, fields, and results may need explicit types;
  collections and reusable algorithms may need generics.
- Static types do not replace runtime validation of external data.
- readonly restricts mutation through typed access but does not provide
  deep immutability of objects at runtime.
- Performance guidance concerns the executed JavaScript and depends on
  the target runtime and workload.

## Table of Contents

- [1. Object Creation and State Scope](#1-object-creation-and-state-scope)
  - [Factory](#factory)
  - [Factory Method / Type Selection](#factory-method--type-selection)
  - [Abstract Factory](#abstract-factory)
  - [Builder](#builder)
  - [Prototype](#prototype)
  - [Singleton](#singleton)
  - [Multiton](#multiton)
  - [Context](#context)
- [2. Interface Boundaries, Composition, and Behavior Management](#2-interface-boundaries-composition-and-behavior-management)
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
- [3. Memory Sharing and Resource Management](#3-memory-sharing-and-resource-management)
  - [Flyweight](#flyweight)
  - [Object Pool](#object-pool)
  - [Resource Pool / Lease](#resource-pool--lease)
  - [Bounded Memory Retention](#bounded-memory-retention)
  - [Explicit Subscription Lifecycle](#explicit-subscription-lifecycle)
- [4. Data Representation and Collection Selection](#4-data-representation-and-collection-selection)
  - [Object as a Record](#object-as-a-record)
  - [Map for Dynamic Keys](#map-for-dynamic-keys)
  - [Set for Uniqueness / Membership](#set-for-uniqueness--membership)
  - [Array for Sequences](#array-for-sequences)
  - [WeakMap / WeakSet](#weakmap--weakset)
  - [TypedArray / ArrayBuffer](#typedarray--arraybuffer)
  - [DataView](#dataview)
  - [Lookup table / registry dispatch](#lookup-table--registry-dispatch)
  - [Alias Normalization at the Boundary](#alias-normalization-at-the-boundary)
  - [Choose Structures by Operation Semantics](#choose-structures-by-operation-semantics)
  - [Snapshot / live view](#snapshot--live-view)
  - [Safe Dictionary](#safe-dictionary)
- [5. Data Structures, Search, Traversal, and Distributed State](#5-data-structures-search-traversal-and-distributed-state)
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
  - [Struct / Schema-Based Record](#struct--schema-based-record)
  - [Prototype overlay / branch](#prototype-overlay--branch)
  - [CRDT G-Counter](#crdt-g-counter)
- [6. Errors and Recovery](#6-errors-and-recovery)
  - [Error Classification](#error-classification)
  - [Error Translation at the Boundary](#error-translation-at-the-boundary)
  - [Propagation / Handling Ownership](#propagation--handling-ownership)
  - [Retry + exponential backoff](#retry--exponential-backoff)
  - [Process-level failure handling](#process-level-failure-handling)
  - [Graceful Shutdown](#graceful-shutdown)
- [7. Code Organization and Data Ownership](#7-code-organization-and-data-ownership)
  - [Data first](#data-first)
  - [Named steps](#named-steps)
  - [Split then compose](#split-then-compose)
  - [Localizing Semantic Complexity](#localizing-semantic-complexity)
  - [Input Ownership](#input-ownership)
  - [Preserving Effect Order](#preserving-effect-order)
  - [Composition over inheritance](#composition-over-inheritance)
  - [Explicit Dependencies / IoC and DI](#explicit-dependencies--ioc-and-di)
  - [Separating Domain and System Code](#separating-domain-and-system-code)
  - [Contracts Before Mechanical Reuse](#contracts-before-mechanical-reuse)
- [8. Performance and Measurement](#8-performance-and-measurement)
  - [Stable object shape](#stable-object-shape)
  - [Dense Arrays / Consistent Element Types](#dense-arrays--consistent-element-types)
  - [Monomorphic hot paths](#monomorphic-hot-paths)
  - [Reducing Escaping Allocations](#reducing-escaping-allocations)
  - [Measure Before Optimizing](#measure-before-optimizing)
  - [Isolating Microbenchmarks](#isolating-microbenchmarks)

The examples show a specific solution and a specific mistake. Names such as
`client`, `cache`, `logger`, and `signal` represent application dependencies.
Run each programming-example block separately from the adjacent block.
Examples target modern Node.js with ES modules and top-level await.
Some lifecycle and structural models intentionally show only the contract
under discussion; their limitations are explained alongside them. “Bad” means
a problem in the stated context, not an unconditional ban on a JavaScript construct.
Performance depends on workload and runtime.

Strategy, Iterator, Chain of Responsibility, Mediator, Multiton, and B-tree
have no dedicated full entries in the source material.
Their examples explain the references; they do not imply corresponding
implementations exist in Metaskills.

## 1. Object Creation and State Scope

### Factory

Centralize defaults and object creation.

#### When to Use

Objects serving the same purpose are created in several places, and their defaults, input validation, or fields must remain consistent.

#### When It Is Excessive

There is one obvious creation site and a stable constructor; a separate factory merely repeats its parameters.

#### Simpler Alternative

Create an object literal or call the existing constructor directly. Extract a small creation function when shared creation rules appear.

#### Good Example

```js
const createUser = (name, role = 'reader') => ({ name, role });
const canRead = (user) => ['reader', 'admin'].includes(user.role);

const fromRegistration = createUser('Anna');
const fromImport = createUser('Boris');
console.log(canRead(fromRegistration), canRead(fromImport)); // true true
```

#### Bad Example

```js
const canRead = (user) => ['reader', 'admin'].includes(user.role);
const fromRegistration = { name: 'Anna', role: 'reader' };
const fromImport = { name: 'Boris' };

console.log(canRead(fromRegistration), canRead(fromImport)); // true false
```

#### What Makes the Difference

The factory defines defaults in one place. In the second version, the import path omits role: users with the same intended role behave differently. A separate factory may be unnecessary for a single stable creation site.

### Factory Method / Type Selection

Localize the choice of a concrete product.

#### When to Use

An existing shared workflow needs to vary the product it creates while keeping the other steps. An overridable method fits when subtype-based extension is already part of the model.

#### When It Is Excessive

Only a simple key-based choice of class varies, with no shared polymorphic workflow. A creator hierarchy adds more code than object creation itself.

#### Simpler Alternative

Use a function with a small conditional, a constructor registry, or pass a creation function into the existing workflow.

#### Good Example

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

#### Bad Example

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

#### What Makes the Difference

In the good version, the shared run workflow does not know the concrete product: an overridable method creates it. In the bad version, format selection is duplicated, so export supports CSV while preview does not. Inheritance is justified by an existing workflow; a registry is enough for simple selection.

### Abstract Factory

Create a consistent family of products.

#### When to Use

Configuration switches an entire family of compatible products, such as a store, cursor, and transaction for one backend.

#### When It Is Excessive

There is only one family implementation, or only one product varies; multiple compatible roles do not need coordination.

#### Simpler Alternative

Pass the concrete dependency directly, or use a single factory for the product that varies.

#### Good Example

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

#### Bad Example

```js
const createSqlStore = () => ({ backend: 'sql' });
const createFileCursor = (store) => {
  if (store.backend !== 'file') throw new Error('Expected file store');
  return { store };
};
const store = createSqlStore();
// createFileCursor(store); // Expected file store
```

#### What Makes the Difference

Choosing one family keeps all products compatible. Selecting the store and cursor independently allows an incompatible combination; the failure appears during assembly or use. With a single family, an ordinary factory is simpler.

### Builder

Assemble a complex product step by step.

#### When to Use

A product is assembled gradually from many optional parts, step order matters, or the final build must check an overall invariant.

#### When It Is Excessive

All parameters are known at once and fit comfortably in one options object. A method chain merely renames the fields of that object.

#### Simpler Alternative

Pass options into a function or constructor and validate the final configuration in one place.

#### Good Example

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

#### Bad Example

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
console.log(query.filters.length); // 2: the finished query changed
```

#### What Makes the Difference

The first build returns a separate list of immutable conditions. In the second version, the finished query shares the builder's mutable array. Copying is shallow here: the example contract assumes scalar condition values.

### Prototype

Clone a prepared object with the required reference semantics.

#### When to Use

Variants of a prepared exemplar are needed, and cloning rules and preservation of shared references are part of the domain model.

#### When It Is Excessive

Creating the object from its original parameters is cheap and clear; copying introduces uncertainty about identity and nested references.

#### Simpler Alternative

Call a constructor or factory. A shallow copy is enough for simple records when shared nested references are acceptable; structuredClone can be used for supported types when its semantics explicitly fit.

#### Good Example

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

#### Bad Example

```js
const original = { start: { x: 0 }, end: { x: 10 } };
const copy = { ...original };
copy.start.x = 5;
console.log(original.start.x); // 5: shared nested point
```

#### What Makes the Difference

The good clone preserves the required independence of nested points. Spread copies only the outer object. It works for a shallow copy but violates the contract of an independent geometric copy.

### Singleton

Share one instance within an explicitly defined scope.

#### When to Use

A defined scope needs one shared coordinator or resource, and multiple instances would break consistency.

#### When It Is Excessive

State belongs to a request, tenant, or test, or multiple independent instances are valid. Global access then only hides dependencies.

#### Simpler Alternative

Create an instance at application startup and pass it to consumers. Use separate instances or a request context for independent requests.

#### Good Example

```js
// One application instance; request state is kept separately.
const counters = new Map();
const recordRequest = (userId) => {
  counters.set(userId, (counters.get(userId) ?? 0) + 1);
};
recordRequest('anna');
recordRequest('boris');
console.log(counters.get('anna')); // 1
```

#### Bad Example

```js
let currentUser;
const handle = async (user) => {
  currentUser = user;
  await Promise.resolve();
  return currentUser;
};
console.log(await Promise.all([handle('anna'), handle('boris')]));
// ['boris', 'boris']: Anna's request read someone else's state
```

#### What Makes the Difference

A shared counter may intentionally belong to the application. The current user belongs to a request: storing it in a singleton mixes concurrent operations. Define what “one” means explicitly: a module, application, or process.

### Multiton

Keep one instance per key.

#### When to Use

A reusable instance is needed for each stable key, such as a tenant, endpoint, or configuration, with clear sharing and shutdown rules.

#### When It Is Excessive

There is one consumer, creation is cheap, or every request needs an independent instance. A registry introduces hidden shared state and an eviction obligation.

#### Simpler Alternative

Create and inject the required instance during application assembly, or create it per operation. A local Map with a bounded lifetime is enough for a short series of calls.

#### Good Example

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

#### Bad Example

```js
const client = { tenantId: null };
const query = async (tenantId) => {
  client.tenantId = tenantId;
  await Promise.resolve();
  return client.tenantId;
};
console.log(await Promise.all([query('anna'), query('boris')]));
// ['boris', 'boris']: the shared client was reconfigured during the request
```

#### What Makes the Difference

A multiton retains a separate instance per key. Reconfiguring a singleton for each tenant causes a race between requests. A large tenant population requires lifetime limits, eviction, and correct client shutdown.

### Context

Pass dependencies and request state explicitly.

#### When to Use

Several related operations within one request or job use a small shared set of capabilities and state that must cross boundaries explicitly.

#### When It Is Excessive

Only one or two arguments are needed, or the dependencies do not share a lifetime. A universal context becomes a hidden service locator.

#### Simpler Alternative

Pass named parameters directly, or capture stable dependencies in a service factory; keep request state as an operation argument.

#### Good Example

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

#### Bad Example

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

#### What Makes the Difference

A context is created per request and passed explicitly. A shared mutable context turns request state into global state. A context must also not become an uncontrolled service locator containing every system dependency.

In the source, Factory Method also covers static factories and registry dispatch.
These are related ways to select the type being created, but a registry alone
is not the classic polymorphic Factory Method.
A module-level singleton does not guarantee one instance across processes or workers.

## 2. Interface Boundaries, Composition, and Behavior Management

### Adapter

Convert an external interface into the expected one.

#### When to Use

At an external-library or legacy boundary, signatures, data shapes, callback/Promise models, or errors differ, and the external side cannot be changed.

#### When It Is Excessive

The contracts are already compatible, or the conversion is obvious and used only once. A separate class adds no isolation.

#### Simpler Alternative

Call the API directly or perform a local conversion. For a standard callback API, use an existing promisify utility while preserving the required receiver.

#### Good Example

```js
import { promisify } from 'node:util';

const client = {
  prefix: 'user:',
  read(id, callback) { callback(null, this.prefix + id); },
};
const read = promisify(client.read.bind(client));
console.log(await read(7)); // user:7
```

#### Bad Example

```js
import { promisify } from 'node:util';

const client = {
  prefix: 'user:',
  read(id, callback) { callback(null, this.prefix + id); },
};
const read = promisify(client.read);
try { await read(7); } catch (error) {
  console.log(error.name); // TypeError: receiver was lost
}
```

#### What Makes the Difference

Both versions convert a callback into a Promise. Only the first preserves the receiver that client.read depends on. Adapt the entire contract: this, results, errors, and required parameters.

### Wrapper

Add behavior around a call.

#### When to Use

The same action, such as timing, logging, or validation, must be added around multiple calls while preserving the original function contract.

#### When It Is Excessive

The action is needed in one place, or is so specific to an operation that the wrapper needs many exceptions.

#### Simpler Alternative

Keep a few explicit statements before and after a direct call; express cleanup with try/finally when necessary.

#### Good Example

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

#### Bad Example

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
// calculator.multiply(3); // TypeError: this was lost
// Even a function independent of this would lose its return value.
```

#### What Makes the Difference

A wrapper must preserve arguments, receiver, and result. The second version breaks this and loses the return value; for an async function it also loses the Promise the caller must await.

### Decorator

Compose behaviors through a shared contract.

#### When to Use

Several optional behaviors must be combined independently through one interface, and the order of layers has a defined meaning.

#### When It Is Excessive

There is one fixed behavior, or only a few unchanging combinations. A wrapper chain makes tracing harder without practical benefit.

#### Simpler Alternative

Use an ordinary function with sequential checks, or an explicit array of steps if they share the same contract.

#### Good Example

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

#### Bad Example

```js
const email = {
  validate: (value) => value.includes('@') ? true : 'Invalid email',
};
const submit = (value) => email.validate(value) ? 'saved' : 'rejected';
console.log(submit('invalid')); // saved: the error string is truthy
```

#### What Makes the Difference

The first set of layers obeys a boolean contract. In the second version, an error string looks like a successful truthy result. Another error format is possible, but all layers and callers must support it consistently.

### Chain of Responsibility

Pass work through a chain of handlers that can stop processing.

#### When to Use

Several handlers decide in order whether to complete a request or pass it on; ordering and termination are part of the contract.

#### When It Is Excessive

Every fixed step always runs and no step selects who handles the request. A next chain then hides an ordinary sequence.

#### Simpler Alternative

Write a direct function with sequential checks and early returns. An explicit pipeline is enough for transformations that cannot terminate processing.

#### Good Example

```js
const response = [];
const auth = (request, next) => {
  if (!request.user) { response.push('401'); return; }
  next();
};
const handler = () => response.push('200');
auth({ user: null }, handler);
console.log(response); // ['401']: the chain stopped
```

#### Bad Example

```js
const response = [];
const auth = (request, next) => {
  if (!request.user) response.push('401');
  next();
};
const handler = () => response.push('200');
auth({ user: null }, handler);
console.log(response); // ['401', '200']: the protected handler ran anyway
```

#### What Makes the Difference

A handler either completes processing or delegates it. A missing return after rejection can both duplicate the response and allow the request into a protected operation.

### Proxy

Control access to an object through a compatible interface.

#### When to Use

Access must pass through a controlled point for access policy, lazy resource acquisition, or transparent observation.

#### When It Is Excessive

One check in one caller is enough, and transparent replacement of the whole interface is unnecessary. Subclassing a platform class can introduce an unnecessary dependency on its lifecycle.

#### Simpler Alternative

Call the check explicitly before the operation or wrap the specific function. Use the built-in JavaScript Proxy only when its interception capabilities are needed.

#### Good Example

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
console.log(emitter.emit('message', 'hello')); // hello, then true
```

#### Bad Example

```js
import { EventEmitter } from 'node:events';

class ObservedEmitter extends EventEmitter {
  count = 0;
  emit(name, ...args) { this.count++; }
}
const emitter = new ObservedEmitter();
emitter.on('message', console.log);
console.log(emitter.emit('message', 'hello')); // only undefined
```

#### What Makes the Difference

The interceptor adds observation while preserving event delivery and the boolean return value of emit. The bad version swallows the call: listeners never run. This is a proxy by object role, not the built-in JavaScript Proxy.

### Bridge

Separate two independently varying dimensions.

#### When to Use

Two dimensions and their combinations vary independently, such as device type and transport. A separate class for every pair is already causing duplication.

#### When It Is Excessive

One dimension is fixed, there are few combinations, or their independence is only hypothetical.

#### Simpler Alternative

Inject a function or implementation object into the existing component without separate abstraction hierarchies.

#### Good Example

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

#### Bad Example

```js
class MqttLight {
  on() { return 'mqtt:light:on'; }
  off() { return 'mqtt:light:off'; }
}
class HttpLight {
  on() { return 'http:light:on'; }
  // Added off only to MqttLight.
}
console.log(typeof new HttpLight().off); // undefined
```

#### What Makes the Difference

Device logic changes independently of transport. Classes for every combination duplicate each device operation; a new off operation must be added consistently to every transport variant.

### Strategy

Select behavior, including through a lookup table.

#### When to Use

One operation needs several interchangeable implementations, and choosing the rule should be separate from the rest of the workflow.

#### When It Is Excessive

There is only one algorithm or one short stable branch. Separate strategy classes add no useful variability.

#### Simpler Alternative

Keep if/else in one place or pass a function argument. A small table of functions is enough for key-based selection.

#### Good Example

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

#### Bad Example

```js
const checkoutPrice = (kind, total) => kind === 'member' ? total * 0.9 : total;
const invoicePrice = (kind, total) => kind === 'member' ? total * 0.95 : total;
console.log(checkoutPrice('member', 100), invoicePrice('member', 100)); // 90 95
```

#### What Makes the Difference

One selected strategy defines the pricing rule for all callers. Independent checkout/invoice branches drift apart. Strategy does not require a class hierarchy: functions and a table sufficiently express variable behavior.

### Composite

Handle a leaf and a composite node uniformly.

#### When to Use

Data truly forms a hierarchy, and clients must perform the same operation on an individual element and a group.

#### When It Is Excessive

The structure is flat, or a leaf and a group have no sensible shared contract. A class hierarchy for a single traversal may be unnecessary.

#### Simpler Alternative

Use an array with reduce/map, or a single recursive traversal function over an ordinary data tree.

#### Good Example

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

#### Bad Example

```js
const order = { children: [{ price: 10 }, { children: [{ price: 5 }] }] };
const total = (group) => group.children.reduce((sum, child) => {
  if ('price' in child) return sum + child.price;
  return sum; // this caller missed nested groups
}, 0);
console.log(total(order)); // 10 instead of 15
```

#### What Makes the Difference

Leaves and groups share total, so recursion is concentrated in the structure. In the second version, callers must recognize every node shape, and one caller misses the nested group. A correct generic traversal can also be sufficient for simple JSON.

### Facade

Expose a small API over a scenario involving several subsystems.

#### When to Use

A repeated scenario must coordinate several subsystems, and callers should receive a small stable API rather than know the internal call order.

#### When It Is Excessive

The scenario is one clear call, or the facade merely renames a dependency API without changing the boundary.

#### Simpler Alternative

Call the dependency directly or extract one scenario function with explicit parameters.

#### Good Example

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

#### Bad Example

```js
const cache = new Map();
const db = { load: async (id) => ({ id, name: 'Anna' }) };
const profileRoute = async (id) => {
  if (cache.has(id)) return cache.get(id);
  const user = await db.load(id);
  cache.set(id, user);
  return user;
};
const exportRoute = async (id) => cache.get(id); // missed the database fallback
console.log(await exportRoute(1)); // undefined, although the user exists
```

#### What Makes the Difference

The facade gives callers one loading contract. Without it, the scenario is reproduced manually: the export route loses the fallback. A facade need not include every system operation; its boundary should stay small.

### Mediator

Coordinate interactions between components.

#### When to Use

Several participants interact in complex ways, and coordination rules should live in one place rather than in hidden calls between participants.

#### When It Is Excessive

Two components interact through one direct call, or the coordinator becomes the center of unrelated scenarios.

#### Simpler Alternative

Keep the direct call or use a small scenario function. A facade may suffice for a single linear use case without event-driven coordination.

#### Good Example

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

#### Bad Example

```js
const events = [];
let mailer;
const inventory = {
  reserve(id) { events.push(`reserve:${id}`); mailer.send(id); },
};
mailer = {
  send(id) { events.push(`email:${id}`); inventory.reserve(id); },
};
// inventory.reserve(1); // endless mutual calls -> stack overflow
```

#### What Makes the Difference

The coordinator defines the interaction, and participants do not call each other implicitly. In the bad version, mutual coordination creates a recursive cycle. In this simple synchronous scenario the coordinator also acts as a facade; a mediator is especially useful for event-based interactions.

### Boxing / Value Object

Centralize the invariants of a domain value.

#### When to Use

A value has a stable domain meaning, invariants, and operations that would otherwise need checks in many places: money, units, or addresses.

#### When It Is Excessive

It is a scalar field used once and validated at the boundary; a separate object protects no additional invariant and complicates serialization.

#### Simpler Alternative

Validate the primitive at the boundary and pass it with a precise name and units. In TypeScript, a branded type can complement the contract when appropriate, while retaining runtime validation of external input.

#### Good Example

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

#### Bad Example

```js
const createMoney = (cents) => ({ cents });
const amount = createMoney(NaN);
const total = amount.cents + createMoney(50).cents;
console.log(total); // NaN reached the result
```

#### What Makes the Difference

A value object checks its invariant at creation and preserves it through add. A plain wrapper without validation only changes the value's shape. The example assumes a single currency and non-negative integer cents; currency operations need a separate contract.

Adapter changes a contract; Wrapper adds an action around a call;
Decorator composes behaviors through a shared contract; Proxy controls access;
Facade combines a subsystem scenario. One implementation can combine these roles.
Boxing here means a domain value object, not `new String(value)`.

## 3. Memory Sharing and Resource Management

### Flyweight

Share common immutable state.

#### When to Use

Many objects duplicate a significant amount of identical immutable data, and data specific to each use can be stored separately.

#### When It Is Excessive

Duplication is small, values mutate frequently, or the cost of identity, keys, and eviction exceeds the memory savings.

#### Simpler Alternative

Keep ordinary independent records. If there is truly one immutable shared value, use a shared constant without a factory cache.

#### Good Example

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

#### Bad Example

```js
const shared = { char: 'A', row: 0 };
const first = { glyph: shared };
const second = { glyph: shared };
first.glyph.row = 10;
console.log(second.glyph.row); // 10: another instance's position changed
```

#### What Makes the Difference

Shared glyph data is immutable, while coordinates belong to individual uses. The bad version also shares mutable position. A flyweight cache with many keys needs a memory-retention policy.

### Object Pool

Reuse expensive mutable objects.

#### When to Use

Object creation or initialization is measurably expensive, objects support reliable reset, and usage and return rights can be clearly bounded.

#### When It Is Excessive

Objects are small and cheap, allocation has not been shown to be a problem, or reset is more complex than safe recreation.

#### Simpler Alternative

Create an object per operation and let GC reclaim it. A strictly sequential owner may only need one working buffer rather than a shared pool.

#### Good Example

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

#### Bad Example

```js
const buffer = new Uint8Array([7, 0]);
const send = async (bytes) => {
  await Promise.resolve();
  return [...bytes];
};
const pending = send(buffer);
// The resource was returned and reused before send completed.
buffer.fill(9);
console.log(await pending); // [9, 9]: different data was sent
```

#### What Makes the Difference

The resource is returned after every user has finished and is cleared before reuse. Returning it before await completes creates a shared-memory race. The teaching pool does not bound concurrency or idle capacity; production policy is defined separately.

### Resource Pool / Lease

Issue a resource with a controlled return.

#### When to Use

External resources are limited or expensive: connections, workers, or devices. Concurrent access, acquisition waiting, and mandatory release need control.

#### When It Is Excessive

A resource is needed by only one short operation, or the existing client already manages a pool; a second pool complicates limits and ownership.

#### Simpler Alternative

Use the library pool, or open and close one resource with try/finally. Pass an already open resource directly to a single owner.

#### Good Example

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
const next = pool.acquire(); // the resource is available again
next.release();
```

#### Bad Example

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
  lease.release(); // unreachable
} catch (error) { console.log(error.message); }
// acquire(); // Pool exhausted after the first failure
```

#### What Makes the Difference

finally returns the lease regardless of the work outcome; preventing duplicate release preserves ownership. In the bad version, the first failure permanently occupies the only slot.

### Bounded Memory Retention

Prevent unbounded growth of long-lived state.

#### When to Use

A long-lived collection accepts a growing set of keys or heavy values, so its limit, lifetime, and behavior on reaching the limit must be defined.

#### When It Is Excessive

Boundedness remains useful; a separate eviction system is excessive for a small static set or data that lives for only one short operation.

#### Simpler Alternative

Keep data in operation-local variables, clear it on completion, or use a fixed set. Remove a cache that provides no measurable benefit.

#### Good Example

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

#### Bad Example

```js
const cache = new Map();
const remember = (key, value) => cache.set(key, value);
for (let id = 0; id < 10000; id++) remember(id, { id });
console.log(cache.size); // 10000; growth follows request history
```

#### What Makes the Difference

The first cache bounds entry count with an explicit FIFO policy. The second retains all values until process termination. An entry limit does not bound exact bytes; heavy values may require an additional budget.

### Explicit Subscription Lifecycle

Remove listeners using the same function object.

#### When to Use

An event source may outlive its subscriber, subscriptions are created repeatedly, or callbacks retain state from a completed operation.

#### When It Is Excessive

A separate lifecycle manager is unnecessary for one subscription when the existing framework already guarantees teardown. Releasing a long-lived subscription does not thereby become optional.

#### Simpler Alternative

Keep the handler reference and remove it in the existing completion hook. If exactly one event is required, consider once; use AbortSignal where the API supports it.

#### Good Example

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

#### Bad Example

```js
import { EventEmitter } from 'node:events';
const stream = new EventEmitter();
const received = [];
stream.on('data', (chunk) => received.push(chunk));
stream.off('data', (chunk) => received.push(chunk));
stream.emit('data', 'after-close');
console.log(received, stream.listenerCount('data')); // ['after-close'] 1
```

#### What Makes the Difference

off compares function references. A new callback with the same source text is not the registered callback. The subscription continues running and retaining captured values after its owner finishes.

Flyweight shares immutable data concurrently. A pool reissues a mutable resource
after the previous use has finished. Limiting idle objects does not limit
the number of resources currently checked out.

## 4. Data Representation and Collection Selection

These are data structures and usage techniques, not GoF patterns.

### Object as a Record

Store a known set of fields.

#### When to Use

The fields are known, their values have distinct roles, and data is passed as a DTO, options, or a serializable record.

#### When It Is Excessive

Keys are arbitrary and change dynamically, or object identity is needed for keys. A fixed schema does not match that task.

#### Simpler Alternative

Pass a scalar for one value. Function arguments may suffice for a few explicitly named parameters; use Map for a dynamic dictionary.

#### Good Example

```js
const createUser = (id, name) => ({ id, name, active: false });
const user = createUser(1, 'Anna');
const metadata = new Map([['source', 'import']]);
console.log(user.id, metadata.get('source')); // 1 import
```

#### Bad Example

```js
const user = { id: 1, name: 'Anna', active: false };
const customField = 'id';
user[customField] = 'external-note';
console.log(user.id); // external-note: metadata overwrote the identifier
```

#### What Makes the Difference

Record fields have stable meanings, while arbitrary metadata lives separately. Mixing namespaces allows an external key to overwrite a DTO field. A dynamic object is valid when that behavior is its explicit contract.

### Map for Dynamic Keys

Store a mutable key-to-value mapping.

#### When to Use

A mutable key-to-value mapping, object keys, size, and insertion/deletion operations are needed without mixing entries with record fields.

#### When It Is Excessive

There are a few fixed named fields, or data must serialize directly to JSON. Map introduces unnecessary conversions.

#### Simpler Alternative

Use an Object literal for a known schema; an object with correct own-property checks is enough for a small static mapping of string keys.

#### Good Example

```js
const firstSocket = { id: 1 };
const secondSocket = { id: 2 };
const sessions = new Map();
sessions.set(firstSocket, 'Anna');
sessions.set(secondSocket, 'Boris');
console.log(sessions.get(firstSocket), sessions.size); // Anna 2
```

#### Bad Example

```js
const firstSocket = { id: 1 };
const secondSocket = { id: 2 };
const sessions = {};
sessions[firstSocket] = 'Anna';
sessions[secondSocket] = 'Boris';
console.log(sessions[firstSocket], Object.keys(sessions).length); // Boris 1
```

#### What Makes the Difference

Map distinguishes keys by object identity. Object converts both keys to the string [object Object], so the second entry overwrites the first.

### Set for Uniqueness / Membership

Check membership and eliminate duplicates.

#### When to Use

Uniqueness is part of the model, or many membership checks are needed against a changing set.

#### When It Is Excessive

The set has a few immutable elements, checks are rare, and a separate Set is not justified. It is unsuitable when duplicates must be preserved.

#### Simpler Alternative

Use a small array with includes or a direct conditional. If membership is already represented by the keys of an existing Map, a separate Set is unnecessary.

#### Good Example

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

#### Bad Example

```js
const seen = [];
for (const id of [1, 1, 2]) seen.push(id);
console.log(seen.length); // 3 instead of two unique IDs
```

#### What Makes the Difference

Set expresses uniqueness explicitly; an array does not enforce it. An array with includes can be correct, but many checks require evaluating the cost of linear search.

### Array for Sequences

Preserve order, duplicates, and indexed access.

#### When to Use

Order, repeated elements, indexed access, and standard sequence transformations are needed.

#### When It Is Excessive

Only one value, a fixed record with named fields, or dynamic key-based lookup is needed. Positional conventions can hide the meaning of data.

#### Simpler Alternative

Pass a single element directly; use an object for several distinct roles; choose Map for key-based lookup.

#### Good Example

```js
const steps = ['open', 'read', 'read', 'close'];
const executed = [];
for (const step of steps) executed.push(step);
console.log(executed); // ['open', 'read', 'read', 'close']
```

#### Bad Example

```js
const steps = new Set(['open', 'read', 'read', 'close']);
const executed = [];
for (const step of steps) executed.push(step);
console.log(executed); // ['open', 'read', 'close']: one read was lost
```

#### What Makes the Difference

The operations form a sequence in which repetition matters. Set preserves the order of unique elements but removes the duplicate, changing the scenario.

### WeakMap / WeakSet

Attach supplementary data to an object's lifetime.

#### When to Use

Supplementary data or a presence marker should exist only while the object key is otherwise alive; enumeration and size are not needed.

#### When It Is Excessive

The owner already controls lifetime, data naturally belongs in fields of an owned object, or enumeration and deterministic removal are required.

#### Simpler Alternative

Store a field on an application-owned object, or use a regular Map with explicit removal at lifecycle completion. Do not rely on GC to close resources promptly.

#### Good Example

```js
const metadata = new WeakMap();
let node = { text: 'hello' };
metadata.set(node, { selected: true });
console.log(metadata.get(node).selected); // true
node = null; // metadata does not hold a strong reference to the former key
```

#### Bad Example

```js
const metadata = new Map();
let node = { text: 'hello' };
metadata.set(node, { selected: true });
node = null;
console.log(metadata.size); // 1: Map still retains the object
```

#### What Makes the Difference

WeakMap does not keep a key alive after all other strong references disappear. GC timing is not guaranteed, so the example does not check when reclamation occurs. Map is appropriate for enumeration but requires explicit entry removal.

### TypedArray / ArrayBuffer

Represent numeric and binary data.

#### When to Use

Data has a specific binary or numeric representation: protocol bytes, samples, a shared buffer, or interaction with a low-level API.

#### When It Is Excessive

It is an ordinary list of mixed values or numbers without binary-layout requirements. Range limits and type conversions only complicate the work.

#### Simpler Alternative

Use a regular Array. If a Node.js API already returns a suitable Buffer, do not copy it into another TypedArray unnecessarily.

#### Good Example

```js
const buffer = new ArrayBuffer(4);
const samples = new Uint16Array(buffer);
samples[0] = 300;
samples[1] = 1000;
console.log([...samples], buffer.byteLength); // [300, 1000] 4
```

#### Bad Example

```js
const samples = new Uint8Array(2);
samples[0] = 300;
samples[1] = 1000;
console.log([...samples]); // [44, 232]: values were truncated to 8 bits
```

#### What Makes the Difference

The representation is selected for the value range. Uint8Array does not validate an arbitrary number: it converts it to an 8-bit value, silently corrupting measurements.

### DataView

Read a binary format with explicit byte order.

#### When to Use

Binary fields must be read or written with explicit byte order, different widths, or unaligned offsets.

#### When It Is Excessive

Only individual bytes are involved, or an existing parser already decodes the protocol correctly. Additional manual parsing increases the risk of error.

#### Simpler Alternative

Use Uint8Array for bytes or an existing parser. Node Buffer offers suitable read/write methods with explicit byte order.

#### Good Example

```js
const bytes = new Uint8Array([0x01, 0x02]);
const view = new DataView(bytes.buffer);
const value = view.getUint16(0, true); // little-endian protocol
console.log(value); // 513
```

#### Bad Example

```js
const bytes = new Uint8Array([0x01, 0x02]);
const view = new DataView(bytes.buffer);
const value = view.getUint16(0); // default big-endian
console.log(value); // 258: a different number was read
```

#### What Makes the Difference

DataView lets the protocol byte order be specified explicitly. Omitting the argument changes interpretation; a multibyte TypedArray also does not replace a protocol endian specification because it uses platform byte order.

### Lookup table / registry dispatch

Centralize key-based selection.

#### When to Use

An action or implementation must be selected by key, especially when one mapping is reused or changes as a set of data.

#### When It Is Excessive

There are one or two simple branches, or conditions cannot be reduced to an exact key: ranges, sequential checks, or dependent effects.

#### Simpler Alternative

Keep a clear if/switch in one place or pass the already selected function; do not build a registry without a need for selection.

#### Good Example

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

#### Bad Example

```js
const exportValue = (kind, value) => {
  if (kind === 'json') return JSON.stringify(value);
  if (kind === 'text') return `${value}`;
  throw new Error('Unknown kind');
};
const previewValue = (kind, value) => {
  if (kind === 'json') return JSON.stringify(value);
  return ''; // missed text
};
console.log(exportValue('text', 42), previewValue('text', 42)); // '42' and ''
```

#### What Makes the Difference

The table has one owner and explicitly rejects unknown keys. Duplicated branching diverges when formats are added. One small switch without duplication is not inherently a problem.

### Alias Normalization at the Boundary

Convert alternative input names into one canonical name.

#### When to Use

External data allows several names for one entity, but internal logic should work with one canonical value.

#### When It Is Excessive

The contract allows only one name, or aliases are unused. Supporting invented synonyms expands the API and creates compatibility obligations.

#### Simpler Alternative

Accept only the canonical name and explicitly reject unknown names. One transitional alias only needs a local conversion at a single input boundary.

#### Good Example

```js
const aliases = new Map([['colour', 'color'], ['COLOR', 'color']]);
const normalize = (input) => aliases.get(input) ?? input;
const save = (input) => ({ field: normalize(input) });
const validate = (input) => normalize(input) === 'color';
console.log(save('colour').field, validate('colour')); // color true
```

#### Bad Example

```js
const save = (input) => ({ field: input === 'colour' ? 'color' : input });
const validate = (input) => input === 'color';
console.log(save('colour').field, validate('colour')); // color false
```

#### What Makes the Difference

After shared normalization, business operations see one canonical key. In the bad example, save accepts an alias while validate rejects the same input.

### Choose Structures by Operation Semantics

Choose between records, ordering, membership, and indexing.

#### When to Use

Before implementation, determine whether order, duplicates, membership, keys, and ownership matter; the correctness of the representation depends on them.

#### When It Is Excessive

The principle is not excessive; lengthy classification or a universal collection factory is excessive when the meaning is obvious.

#### Simpler Alternative

Choose a familiar built-in type and briefly state the essential contract. Do not add a separate structure-selection layer.

#### Good Example

```js
const steps = ['read', 'read', 'close'];
const allowedModes = ['read', 'write'];
console.log(steps.length); // 3 operations
console.log(allowedModes.includes('read')); // true
```

#### Bad Example

```js
const steps = new Set(['read', 'read', 'close']);
console.log(steps.size); // 2: uniqueness was imposed on the sequence
```

#### What Makes the Difference

An ordered sequence with duplicates needs Array; includes is enough for a small static allowlist. Choosing a structure for its popularity rather than semantics can change data or introduce unnecessary complexity.

### Snapshot / live view

Make state sharing in the result explicit.

#### When to Use

A collection crosses an ownership boundary, and the caller must know whether the result reflects later changes and whether it can modify internal state.

#### When It Is Excessive

An independent snapshot is unnecessary when the caller intentionally works with shared state under one ownership policy. Deep copying everything can be particularly expensive and incorrect for identity.

#### Simpler Alternative

Pass an explicitly documented live view or iterator, or compute the required scalar. If only the array membership/order must be detached, use a shallow copy rather than a deep clone.

#### Good Example

```js
const items = [{ id: 1 }];
const snapshot = () => [...items];
const copy = snapshot();
copy.push({ id: 2 });
console.log(items.length); // 1
console.log(copy[0] === items[0]); // true: elements are shared by contract
```

#### Bad Example

```js
const items = [{ id: 1 }];
const snapshot = () => items;
const copy = snapshot();
copy.push({ id: 2 });
console.log(items.length); // 2: the "copy" changed the owner's data
```

#### What Makes the Difference

The first result separates the sequence contents without cloning elements. The second returns a live view under the name snapshot. Independent nested objects need a stricter copying contract.

### Safe Dictionary

Keep external keys separate from prototype properties.

#### When to Use

Arbitrary external strings are used as keys and must be distinguished from prototype properties while preserving a correct write policy.

#### When It Is Excessive

Keys are fixed and controlled by code, or data is already in a Map. Separate dictionary infrastructure does not improve that contract.

#### Simpler Alternative

Use Map for external keys, or a regular object with Object.hasOwn and a consistent safe-write policy. A literal with known fields is enough for a fixed DTO.

#### Good Example

```js
const dictionary = Object.create(null);
dictionary['name'] = 'Anna';
console.log(Object.hasOwn(dictionary, 'toString')); // false
console.log(Object.hasOwn(dictionary, 'name')); // true
```

#### Bad Example

```js
const dictionary = {};
const hasEntry = (key) => key in dictionary;
console.log(hasEntry('toString')); // true: the entry was never added
```

#### What Makes the Difference

A prototype-free dictionary and own-key checks separate entries from inherited properties. In the second version, an external key can incorrectly appear to be an existing entry. A regular object with Object.hasOwn also works with an agreed key-write policy.

## 5. Data Structures, Search, Traversal, and Distributed State

### Singly Linked List

Insert and remove elements at the head.

#### When to Use

Head operations are frequent, or nodes must be linked without moving the remaining elements; indexed access is not the main operation.

#### When It Is Excessive

Frequent indexed access, sorting, or compact sequential storage is needed. A separate object per node complicates traversal and reference management.

#### Simpler Alternative

Use Array; push/pop usually suffice for LIFO. Introduce a custom list only for required operations or sharing constraints.

#### Good Example

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

#### Bad Example

```js
let head = null;
const push = (value) => {
  if (!head) { head = { value, next: null }; return; }
  let tail = head;
  while (tail.next) tail = tail.next;
  tail.next = { value, next: null };
};
push(1); push(2);
console.log(head.value); // 1: appended at the tail instead of pushing at the head
```

#### What Makes the Difference

The first version performs head push/pop in a constant number of steps. The second traverses the entire list and changes the operation: it appends rather than pushing at the head. For append, retain a tail pointer and name the operation explicitly.

### Doubly Linked List

Remove or move known nodes.

#### When to Use

Already known nodes must be removed or moved, both ends must be used, or traversal must work in both directions.

#### When It Is Excessive

One-ended operations suffice, or the list is small and rarely changes. Maintaining prev/next, head/tail, and size introduces additional invariants.

#### Simpler Alternative

Use Array for small sequences, a singly linked list for one-way links, or an existing deque for end operations.

#### Good Example

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

#### Bad Example

```js
const a = { value: 'A', prev: null, next: null };
const b = { value: 'B', prev: a, next: null };
const c = { value: 'C', prev: b, next: null };
a.next = b; b.next = c;
a.next = c; // updated only the forward link
console.log(a.next.value, c.prev.value); // C B: traversals disagree
```

#### What Makes the Difference

Removal must restore both links and detach the removed node. This example shows an internal node; a complete operation also handles head/tail and size. Updating only one direction leaves backward traversal corrupted.

### Immutable Cons List

Create sequence versions that share a tail.

#### When to Use

Independent sequence versions, frequent prepend, and structural tail sharing are needed for history, persistent state, or branching computations.

#### When It Is Excessive

History is unnecessary, changes must be in-place, or indexed access dominates. A shared tail complicates the work without useful copying savings.

#### Simpler Alternative

Use Array copies for a small history, or push/pop for an ordinary mutable stack. Prefer an existing persistent collection over a custom one when a suitable dependency is available.

#### Good Example

```js
const prepend = (value, tail) => Object.freeze({ value, tail });
const history = prepend('initial', null);
const first = prepend('first', history);
const second = prepend('second', history);
console.log(first.tail === second.tail); // true: shared immutable tail
console.log(history.value); // initial
```

#### Bad Example

```js
const history = { value: 'initial', tail: null };
const first = { value: 'first', tail: history };
const second = { value: 'second', tail: history };
first.tail.value = 'changed';
console.log(second.tail.value); // changed: another version was corrupted
```

#### What Makes the Difference

Structural sharing is safe with immutable nodes. The bad version shares a mutable tail. Freeze protects the node in the example, but mutation of object values needs a separate policy.

### Circular Buffer

Reuse array positions in a ring.

#### When to Use

A data stream is processed sequentially, storage must be reused, capacity and overflow policy are known, and frequent end operations matter.

#### When It Is Excessive

The queue is small and rarely used, or the complete history must be retained. Manual indices and full/empty distinction provide no practical gain.

#### Simpler Alternative

Use Array for a small volume or an existing queue/deque. A read cursor is enough for a single pass through an accumulated array.

#### Good Example

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
// A, successful write of C, then B and C
```

#### Bad Example

```js
const buffer = new Array(2);
let head = 0;
let tail = 0;
const write = (value) => {
  buffer[tail] = value;
  tail = (tail + 1) % buffer.length;
};
write('A'); write('B');
console.log(head === tail); // true: a full ring was mistaken for an empty one
```

#### What Makes the Difference

Size distinguishes empty from full and prevents overwriting unread data. Head/tail alone are ambiguous without additional state. The teaching Ring assumes capacity has already been checked as a positive integer.

### Queue / FIFO

Process elements in arrival order.

#### When to Use

Jobs or messages must be processed in arrival order; older elements must not yield to newer ones because of LIFO.

#### When It Is Excessive

There is one element, data can be processed immediately, or processing is a single pass over an existing array. A separate Queue object is unnecessary.

#### Simpler Alternative

Use for...of for an existing batch, or Array with push/shift for a small queue. For a large stream, choose an existing queue or ring without changing the FIFO contract.

#### Good Example

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

#### Bad Example

```js
const jobs = [];
jobs.push('first'); jobs.push('second');
console.log(jobs.pop(), jobs.pop()); // second first: this is LIFO
```

#### What Makes the Difference

A queue preserves arrival order. Pop turns scheduling into a stack and can delay old jobs under continuous arrivals. Array.shift is used here for clarity; a large hot queue needs a ring or another specialized implementation.

### Deque

Work efficiently at both ends.

#### When to Use

Insertion and removal at both ends are needed for a sliding window, bidirectional traversal, or an algorithm alternating front/back operations.

#### When It Is Excessive

Only one end is used, or searching and indexed changes in the middle dominate.

#### Simpler Alternative

Use Array push/pop for a stack, a regular Array for a small window, or restrict the API to an ordinary queue if only FIFO is needed.

#### Good Example

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

#### Bad Example

```js
const window = [];
const limit = 3;
const record = (value) => {
  window.push(value);
  if (window.length > limit) window.pop();
};
for (const value of [1, 2, 3, 4]) record(value);
console.log(window); // [1, 2, 3]: discarded the new value instead of the old one
```

#### What Makes the Difference

The window adds on the right and removes on the left. Removing at the same end retains stale data. The array demonstrates the contract; frequent operations at both ends need a deque implementation with suitable complexity.

### Stack / LIFO

Process the most recently added element first.

#### When to Use

The last added element must be processed first: undo, nested operations, or iterative depth-first traversal.

#### When It Is Excessive

Order does not matter, FIFO is required, or there is no chain of nested actions. A separate Stack class over a couple of calls is unnecessary.

#### Simpler Alternative

Use Array with push/pop. Clear recursion can remain for a shallow tree when its depth is safe for the runtime.

#### Good Example

```js
const undo = [];
undo.push('type A');
undo.push('type B');
console.log(undo.pop()); // type B: undo the latest command
```

#### Bad Example

```js
const undo = [];
undo.push('type A');
undo.push('type B');
console.log(undo.shift()); // type A: undo in the wrong order
```

#### What Makes the Difference

Undo history needs LIFO. FIFO removal reverses an earlier command that a later command may depend on.

### Unrolled List

Store a queue in blocks to reduce node allocations.

#### When to Use

Measured enqueue/dequeue traffic is high, and block storage reduces node allocations or improves locality compared with a per-element linked list.

#### When It Is Excessive

Volume is moderate, random access is needed, or allocation pressure has not been measured. Block sizing, reset, and pooling add complexity without confirmed benefit.

#### Simpler Alternative

Use an existing Queue/CircularBuffer or Array for a small volume. Introduce block storage only after comparing it with a simpler queue.

#### Good Example

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

#### Bad Example

```js
const block = { values: ['A', 'B'], read: 0 };
const first = block.values[block.read++];
// The block was returned to the pool after one read.
block.values = [];
block.read = 0;
console.log(first, block.values[block.read]); // A undefined: B was lost
```

#### What Makes the Difference

A block is released only after every element has been read. The bad reset destroys pending values. The good example shows block storage, not a production node pool or optimal cleanup of an empty tail.

### Binary Search Tree

Order elements for search and traversal.

#### When to Use

Ordered storage, comparator-based search, and order-dependent operations are needed. Guaranteed logarithmic depth requires a balanced implementation.

#### When It Is Excessive

Only exact-key lookup is needed, the set is small, or data is loaded as a whole and rarely changes. A custom unbalanced BST can lose to simpler options.

#### Simpler Alternative

Use Map/Set for exact keys, or a sorted Array with binary search for a small or rarely changing ordered set.

#### Good Example

```js
const insert = (node, value) => {
  if (!node) return { value, left: null, right: null };
  if (value < node.value) node.left = insert(node.left, value);
  else if (value > node.value) node.right = insert(node.right, value);
  return node; // duplicates are ignored
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

#### Bad Example

```js
const root = { value: 4, left: null, right: { value: 2 } };
const find = (node, value) => {
  if (!node) return false;
  if (node.value === value) return true;
  return find(value < node.value ? node.left : node.right, value);
};
console.log(find(root, 2)); // false: the node was inserted into the wrong branch
```

#### What Makes the Difference

Insertion and lookup use the same ordering rule and an explicit duplicate policy. Inconsistent insertion makes an existing value unreachable. This BST is unbalanced: worst-case search and recursion depth are linear.

### B-tree

A structure briefly mentioned in the data-structures description.

#### When to Use

A large ordered index must work efficiently with storage blocks and support lookup and updates; use a proven B-tree implementation with balancing.

#### When It Is Excessive

Data is small, fits in a simple list, or only exact lookup in memory is needed. Implementing split/merge yourself introduces substantial error risk.

#### Simpler Alternative

Use an existing database index. In memory, choose Map for exact keys or a sorted Array for a small ordered set; do not implement a B-tree from the teaching fragment.

#### Good Example

```js
// Node-model and lookup fragment, not a full B-tree implementation.
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

#### Bad Example

```js
const root = {
  keys: [10, 20],
  children: [
    { keys: [2, 5], children: [] },
    { keys: [25, 30], children: [] },
  ],
};
console.log(root.children.length === root.keys.length + 1); // false
// The third child range is missing: lookup cannot choose a branch correctly.
```

#### What Makes the Difference

An internal node with k keys must separate k+1 ranges. The bad model violates even this structural invariant. A full B-tree also needs occupancy limits, equal leaf depth, and correct split/merge; the example does not implement them. Metaskills only briefly mentions B-tree.

### Heap / Priority Queue

Retrieve the element with minimum or maximum priority.

#### When to Use

Elements must be inserted repeatedly and the next minimum/maximum extracted without sorting the entire collection after every change.

#### When It Is Excessive

The minimum is needed once, the set is small, or one full sort before processing is enough.

#### Simpler Alternative

Find the minimum once with a linear scan, or sort an Array and traverse it sequentially. Use an existing priority queue for sustained priority-based processing.

#### Good Example

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
console.log(heap[0]); // 1: a min-heap after each insertion
```

#### Bad Example

```js
const heap = [];
const push = (value) => heap.push(value);
push(5); push(2); push(1);
console.log(heap[0]); // 5 instead of the minimum 1
```

#### What Makes the Difference

Sift-up restores the parent ≤ child invariant. An ordinary append creates an array, not a heap. Only insertion is shown; removing the root needs a separate sift-down, not a simple shift.

### Trie

Search by string prefix.

#### When to Use

The main queries involve string prefixes: autocomplete, finding all continuations, or routing by a character sequence.

#### When It Is Excessive

Only exact-key lookup is needed, the set is small, or prefix queries are rare. Trie nodes can consume more memory than the strings themselves.

#### Simpler Alternative

Use Map/Set for exact keys, or array filter with startsWith for small autocomplete. Complicate the index only after checking the real workload.

#### Good Example

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

#### Bad Example

```js
const root = { c: { a: { r: {} } } };
const has = (word) => {
  let node = root;
  for (const char of word) {
    node = node[char];
    if (!node) return false;
  }
  return true; // any path is treated as a word
};
console.log(has('ca')); // true: only car was inserted
```

#### What Makes the Difference

Terminal distinguishes a complete word from an existing prefix. Without this flag, every prefix automatically becomes a key.

### Graph / adjacency list + BFS

Traverse connections in layers.

#### When to Use

The model has arbitrary connections, and layered traversal or the shortest edge count in an unweighted graph is needed. Cycles and repeated paths require visited tracking.

#### When It Is Excessive

The structure is a simple list, only direct neighbors need processing, or BFS order is unnecessary. A full graph model and generic traversal are excessive.

#### Simpler Alternative

Use a direct array pass or one operation over neighbors. A specialized traversal is enough for an ordinary tree; weighted shortest-path search needs a different algorithm.

#### Good Example

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

#### Bad Example

```js
const graph = new Map([['A', ['B']], ['B', ['A']]]);
const queue = ['A'];
// The limit of 5 prevents this teaching example from looping forever.
for (let i = 0; i < queue.length && i < 5; i++) {
  queue.push(...(graph.get(queue[i]) ?? []));
}
console.log(queue); // A, B, A, B, A, B
```

#### What Makes the Difference

Vertices are marked visited when enqueued so each vertex is added once. Without it, a cycle causes infinite traversal; an artificial limit merely interrupts incorrect work.

### Iterator

Provide a uniform sequential traversal interface.

#### When to Use

The caller must traverse data sequentially without knowing its representation; lazy traversal or for...of support is useful.

#### When It Is Excessive

The collection is already an Array or iterable, and a new iterator merely duplicates its traversal. An iterator also does not replace random access.

#### Simpler Alternative

Return an existing iterable or traverse an ordinary array. A generator function is enough for one specialized traversal without a separate iterator class.

#### Good Example

```js
class Sequence {
  constructor(values) { this.values = values; }
  *[Symbol.iterator]() { yield* this.values; }
}
const collection = new Sequence([1, 2, 3]);
let sum = 0;
for (const value of collection) sum += value;
console.log(sum); // 6: the caller uses public traversal
```

#### Bad Example

```js
const oldCollection = { buffer: [1, 2, 3] };
const sum = (collection) => collection.buffer.reduce((total, n) => total + n, 0);
console.log(sum(oldCollection)); // 6
const newCollection = { head: { value: 1, next: null } };
// sum(newCollection); // TypeError: the caller depends on the internal buffer
```

#### What Makes the Difference

An iterator separates traversal from storage. A caller reading the internal buffer breaks when an array is replaced by linked storage, even if the collection still means the same thing.

### LRU Cache

Evict the least recently used entry.

#### When to Use

A bounded cache is needed, recent accesses are reasonably likely to repeat, and recency-based eviction suits the workload.

#### When It Is Excessive

Data is cheaper to recompute, the set is static, repeated accesses are rare, or age matters more than recency. Maintaining LRU without a benefit is unnecessary.

#### Simpler Alternative

Remove the cache, or use local memoization for a short operation. Choose explicit TTL when freshness matters, or FIFO when insertion order matters.

#### Good Example

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

#### Bad Example

```js
const cache = new Map([['A', 1], ['B', 2]]);
const get = (key) => cache.get(key); // ordering is not updated
get('A');
cache.set('C', 3);
cache.delete(cache.keys().next().value);
console.log([...cache.keys()]); // ['B', 'C']: the recently accessed A was evicted
```

#### What Makes the Difference

LRU updates recency on reads. The bad version evicts by insertion order, implementing a different policy. A full implementation must also define how set updates an existing key.

### Struct / Schema-Based Record

Control fields, types, and mutability.

#### When to Use

Records with the same fields, defaults, type checks, and defined mutable/immutable semantics are created in several places.

#### When It Is Excessive

There is one simple record, its schema is already validated at the boundary, or the framework provides the required model. A custom universal Struct duplicates that validation.

#### Simpler Alternative

Use an object literal or a small factory with explicit validation. In TypeScript, describe the static type while retaining runtime checks for untrusted data.

#### Good Example

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

#### Bad Example

```js
const defaults = { name: '', tags: [] };
const createRecord = (input = {}) => ({ ...defaults, ...input });
const first = createRecord();
const second = createRecord();
first.tags.push('new');
console.log(second.tags); // ['new']: the mutable default is shared
console.log(createRecord({ nam: 123 })); // a typo and an invalid type were accepted
```

#### What Makes the Difference

Creation validates fields and separates mutable defaults across instances. Spreading defaults neither validates the schema nor copies nested arrays. Seal fixes the field set but does not make the record deeply immutable.

### Prototype overlay / branch

Create a variant that shares base state.

#### When to Use

Variants of a stable base need only a few differences, and inherited reads and sharing intentionally belong to the contract.

#### When It Is Excessive

An independent copy, ordinary JSON serialization of all fields, or a frequently mutated base is involved. Prototype lookup hides dependencies between versions.

#### Simpler Alternative

For a small record, use spread with explicit overrides; configuration can retain base and overrides separately and resolve values through one function.

#### Good Example

```js
const base = Object.freeze({ color: 'red', size: 10 });
const variant = Object.create(base, {
  color: { value: 'blue', enumerable: true },
});
console.log(variant.color, variant.size); // blue 10
console.log(Object.hasOwn(variant, 'size')); // false: sharing is explicit
```

#### Bad Example

```js
const base = { color: 'red', size: 10 };
const independentCopy = Object.create(base);
base.size = 20;
console.log(independentCopy.size); // 20: this is an overlay, not an independent copy
```

#### What Makes the Difference

An overlay intentionally inherits base values. Freezing the base makes sharing predictable for the scalar schema shown. Object.create(base) cannot be called an independent copy: reads still depend on the prototype.

### CRDT G-Counter

Merge monotonic replica counters.

#### When to Use

Several replicas independently increment counters, states can arrive repeatedly and out of order, and merging must converge without a central increment operation.

#### When It Is Excessive

There is one writer or an available central transactional authority. G-Counter also does not support decrements, reset, or arbitrary value changes.

#### Simpler Alternative

Use a local counter or an atomic increment in a shared database. If decrement is required, choose another consistent contract rather than modifying G-Counter with simple subtraction.

#### Good Example

```js
const merge = (local, remote) => local.map((value, i) => Math.max(value, remote[i]));
const anna = [2, 0];
const boris = [0, 3];
const once = merge(anna, boris);
const twice = merge(once, boris);
console.log(once, twice); // [2, 3] [2, 3]
console.log(twice.reduce((sum, n) => sum + n, 0)); // 5
```

#### Bad Example

```js
const merge = (local, remote) => local.map((value, i) => value + remote[i]);
const once = merge([2, 0], [0, 3]);
const twice = merge(once, [0, 3]);
console.log(once, twice); // [2, 3] [2, 6]: duplicate delivery was counted twice
```

#### What Makes the Difference

Componentwise max makes merge idempotent. Adding states counts previously known increments again. The example assumes a fixed, agreed replica set and non-negative monotonic components.

Built-in and library versions of a structure can have different empty-value,
iteration, and mutation contracts. For example, `undefined` from dequeue is
indistinguishable from a stored `undefined` unless the API provides an additional
presence indicator. These differences cannot be removed by mechanically replacing values.

## 6. Errors and Recovery

### Error Classification

Distinguish programming errors from expected environmental failures.

#### When to Use

The error kind determines the action: retry an operation, return an agreed rejection, use a fallback, or stop because of a programming failure.

#### When It Is Excessive

A separate class hierarchy and large code catalog are unnecessary when all errors in a local operation propagate in the same way. Distinguishing valid from invalid recovery remains necessary.

#### Simpler Alternative

Propagate the original error without catch, or check one documented error.code where recovery is genuinely possible.

#### Good Example

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
// TypeError 1: the programming error is not retried
```

#### Bad Example

```js
let calls = 0;
const load = async () => { calls++; throw new TypeError('Broken logic'); };
const run = async () => {
  for (let attempt = 1; attempt <= 3; attempt++) {
    try { return await load(); }
    catch { /* retry any error */ }
  }
  return null;
};
console.log(await run(), calls); // null 3: a bug was disguised as missing data
```

#### What Makes the Difference

Classification determines permitted recovery. Retrying does not fix a TypeError; the first version propagates it. The second wastes attempts and hides the bug. Specific transient codes depend on the library used.

### Error Translation at the Boundary

Translate an internal error into a public code.

#### When to Use

An internal error crosses a public or service boundary; the external consumer needs a stable code and an acceptable level of detail.

#### When It Is Excessive

The code is inside one component and the error meaning does not change. Translating at every internal layer makes tracing harder and can lose the original cause.

#### Simpler Alternative

Propagate the original error to one existing API boundary and translate it there using the framework's standard handler.

#### Good Example

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
console.log(toResponse(error)); // 500, public code only
```

#### Bad Example

```js
const toResponse = (error) => ({
  status: 500,
  body: { message: error.message, stack: error.stack },
});
const error = new Error('SELECT failed on db.internal');
console.log(toResponse(error).body.message); // internal details in the API
```

#### What Makes the Difference

The API boundary exposes consistent public codes while keeping internal details for internal observation. Directly serializing message/stack exposes service internals and couples external clients to internal wording.

### Propagation / Handling Ownership

Recover, translate, or propagate an error.

#### When to Use

The code must establish which layer can recover, add context, or translate the error, and which should simply pass it on.

#### When It Is Excessive

The local function has no useful recovery or translation: a catch that immediately throws the same error adds no behavior.

#### Simpler Alternative

Remove the empty catch and let the exception or rejected Promise propagate. Keep cleanup in finally and observation with the chosen owner.

#### Good Example

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

#### Bad Example

```js
const listUsers = async (repository) => {
  try { return await repository.list(); }
  catch { return []; }
};
const repository = { list: async () => { throw new Error('Database offline'); } };
console.log(await listUsers(repository)); // []: looks like "there are no users"
```

#### What Makes the Difference

Recovery is allowed only for an agreed optional source. A primary-database failure must remain an error. A universal empty result can cause incorrect caller decisions, such as clearing data as “no longer existing.”

### Retry + exponential backoff

Retry an operation that failed temporarily.

#### When to Use

There is a known transient failure, retrying is safe, attempts and waiting time are bounded, and the dependency does not already perform the same retry.

#### When It Is Excessive

The error is permanent, an earlier write may already have succeeded without idempotency, or retrying only worsens overload. A second retry layer over the client's retry may also be unnecessary.

#### Simpler Alternative

Make one attempt and return an explicit error; configure a suitable built-in client policy instead of a custom loop when available. First establish a safe retry contract for writes.

#### Good Example

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

#### Bad Example

```js
let charges = 0;
const charge = async () => {
  charges++; // the payment already succeeded, but the response was lost
  throw new Error('Timeout after commit');
};
for (let attempt = 0; attempt < 2; attempt++) {
  try { await charge(); break; }
  catch { /* retry without an idempotency key */ }
}
console.log(charges); // 2 charges
```

#### What Makes the Difference

The first helper is intended for safe reads and retries only a known transient failure with bounded backoff. Retrying a write after a lost response can double its effect. The teaching helper adds no overall deadline/cancellation: when needed, the policy must pass a signal and bound the entire operation budget.

### Process-level failure handling

Record an unhandled process failure.

#### When to Use

The application owns the process and must define policy for an unhandled fatal failure: recording the cause, exit status, and interaction with a supervisor.

#### When It Is Excessive

A library installs the handler in someone else's process, or the runtime/framework already defines the required policy. Duplicate process handlers can conflict.

#### Simpler Alternative

Use the runtime's normal fatal termination and a supervisor; libraries should propagate errors to callers rather than install global handlers. Connect additional diagnostics at one application entry point.

#### Good Example

```js
// Test model of a supervisor API: the actual process does not exit.
const actions = [];
const runtime = { exit: (code) => actions.push(`exit:${code}`) };
const onFatal = (error) => {
  actions.push(`fatal:${error.message}`);
  runtime.exit(1);
};
onFatal(new Error('Corrupted state'));
console.log(actions); // ['fatal:Corrupted state', 'exit:1']
// In the application, onFatal is registered with process.on('uncaughtException', ...).
```

#### Bad Example

```js
const actions = [];
const onFatal = (error) => actions.push(`fatal:${error.message}`);
onFatal(new Error('Corrupted state'));
actions.push('accept next request');
console.log(actions); // the process continued serving requests after a fatal error
```

#### What Makes the Difference

An unhandled programming failure can violate process invariants. The good scenario signals unsuccessful termination to the supervisor; the bad one only logs and continues. This is a policy model: a real path must coordinate bounded termination, synchronous error recording, and exit status.

### Graceful Shutdown

Stop accepting work and finish current work.

#### When to Use

A long-lived service accepts concurrent work and owns resources that cannot be closed before active operations finish.

#### When It Is Excessive

A separate coordinator is unnecessary for a short CLI command without background work, or an application whose framework already provides the required shutdown. Do not duplicate lifecycle management.

#### Simpler Alternative

Use standard framework hooks and close resources in try/finally. One bounded close suffices for one resource; a service must retain stop-admission behavior and deadline-bounded draining.

#### Good Example

```js
const events = [];
let finish;
const activeRequest = new Promise((resolve) => { finish = resolve; });
const server = { stopAdmission: () => events.push('stop admission') };
const shutdown = async () => {
  server.stopAdmission();
  await activeRequest; // production waiting is bounded by a deadline
  events.push('close database');
};
const closing = shutdown();
events.push('request completed');
finish();
await closing;
console.log(events); // stop admission → request completed → close database
```

#### Bad Example

```js
const events = [];
const shutdown = () => {
  events.push('destroy active socket');
  events.push('close database');
};
shutdown();
events.push('request tries to commit');
console.log(events); // the request continued after its resources were closed
```

#### What Makes the Difference

Orderly shutdown rejects new work, waits for active work, and only then closes dependencies. Immediately destroying resources interrupts current operations. Production needs a deadline and a separate forced path: waiting forever for a stuck request is also wrong.

The source retry example does not filter transient errors; the shutdown example
closes connections immediately. Good examples in this registry show the required
behavior of the solution, not a claim that the source implementations are correct.

## 7. Code Organization and Data Ownership

### Data first

Choose the data relationships before the algorithm.

#### When to Use

Algorithm correctness depends on data relationships: uniqueness, order, keys, identity, and state ownership.

#### When It Is Excessive

Turning an obvious built-in type choice into a separate design phase with a universal model is excessive. Understanding the data remains necessary.

#### Simpler Alternative

Choose the smallest suitable literal, Array, Map, or Set and record only the essential invariant. Do not introduce another data layer without a separate need.

#### Good Example

```js
const orders = [{ id: 1, total: 10 }, { id: 1, total: 10 }, { id: 2, total: 5 }];
const byId = new Map();
for (const order of orders) byId.set(order.id, order);
const total = [...byId.values()].reduce((sum, order) => sum + order.total, 0);
console.log(total); // 15: one order per ID
```

#### Bad Example

```js
const ids = [1, 1, 2];
const totals = [10, 10, 5];
const uniqueIds = [...new Set(ids)];
const total = uniqueIds.reduce((sum, id, index) => sum + totals[index], 0);
console.log(total); // 20: after deduplication, indices no longer match orders
```

#### What Makes the Difference

The Map representation preserves the ID-to-record relationship before computation. Independently transforming parallel arrays destroys that correspondence. Conflicting records for one ID need an explicit policy: the last one wins here.

### Named steps

Make intermediate meanings explicit.

#### When to Use

An expression mixes several meanings, evaluation order is difficult to see, or intermediate values need independent checking.

#### When It Is Excessive

The expression is short and obvious, and intermediate variables only repeat syntax without adding meaning.

#### Simpler Alternative

Keep a direct return, a short chain, or a clear conditional. Name an independent concept, not every operator.

#### Good Example

```js
const items = [
  { name: 'Anna', visible: true, selected: false },
  { name: 'Boris', visible: false, selected: true },
];
const visible = items.filter((item) => item.visible);
const labels = visible.map((item) => item.selected ? `* ${item.name}` : item.name);
console.log(labels.join(', ')); // Anna
```

#### Bad Example

```js
const items = [
  { name: 'Anna', visible: true, selected: false },
  { name: 'Boris', visible: false, selected: true },
];
console.log(items.filter((x) => x.visible).map((x) => x.selected ? `* ${x.name}` : x.name).join(', '));
// Anna: the result is the same, but stages are hidden in one long line.
```

#### What Makes the Difference

The bad version does not change the result here. The problem is maintenance: visibility filtering, formatting, and assembly are mixed into one expression, making intermediate values harder to inspect and one stage harder to change locally. Short clear chains are valid.

### Split then compose

Separate independent stages and compose them explicitly.

#### When to Use

An operation contains independent stages with different rules or repeated semantics that need separate checks and changes.

#### When It Is Excessive

Splitting creates functions for every trivial step, hides linear order, or requires passing a large bag of intermediate state.

#### Simpler Alternative

Keep one sequential function with a few meaningful local variables; extract only a genuinely independent algorithm.

#### Good Example

```js
const tokenize = (text) => text.trim().split(/\s+/);
const pack = (tokens) => ({ tokens, count: tokens.length });
const parse = (text) => pack(tokenize(text));
console.log(parse(' one   two ')); // tokens: ['one', 'two'], count: 2
```

#### Bad Example

```js
const parse = (text) => ({
  tokens: text.trim().split(/\s+/),
  count: text.split(' ').length,
});
console.log(parse(' one   two ')); // tokens: 2 elements, count: 6
```

#### What Makes the Difference

Decomposition separates the independent tokenization rule from result packaging. The second version repeats the rule differently, so count disagrees with tokens. A separate function is not needed for every trivial action.

### Localizing Semantic Complexity

Keep one rule system in one place.

#### When to Use

A business rule, alias set, or set of protocol cases repeats and must change consistently.

#### When It Is Excessive

The similarity is merely syntactic, while the rules belong to different domains and should change independently. A common abstraction couples them artificially.

#### Simpler Alternative

Keep a local conditional for one rule; use a small table or function for one shared rule set, without a rules framework.

#### Good Example

```js
const aliases = new Map([['js', 'javascript'], ['ts', 'typescript']]);
const normalizeLanguage = (name) => aliases.get(name) ?? name;
const validate = (name) => ['javascript', 'typescript'].includes(normalizeLanguage(name));
const save = (name) => ({ language: normalizeLanguage(name) });
console.log(validate('ts'), save('ts')); // true, language: typescript
```

#### Bad Example

```js
const validate = (name) => ['js', 'javascript', 'ts', 'typescript'].includes(name);
const save = (name) => ({ language: name === 'js' ? 'javascript' : name });
console.log(validate('ts'), save('ts')); // true, language: ts
```

#### What Makes the Difference

The alias rule lives in one place and serves several operations. With local lists, the validator already recognizes ts, but saving does not yet normalize it. This is no reason for a generic rules framework: one table is enough.

### Input Ownership

Mutate only state owned by the operation.

#### When to Use

A function receives shared data or promises to compute a result without changing caller state. Permitted mutations must be explicit.

#### When It Is Excessive

Copying data the operation already owns exclusively, or enforcing immutability against an agreed in-place contract, is excessive. Ownership must still be clear.

#### Simpler Alternative

Mutate a local working collection or an explicitly named in-place argument. A shallow copy, not a deep clone, suffices if only array order changes.

#### Good Example

```js
const users = [{ score: 1 }, { score: 3 }];
const rank = (items) => [...items].sort((a, b) => b.score - a.score);
const ranked = rank(users);
console.log(ranked[0].score, users[0].score); // 3 1
```

#### Bad Example

```js
const users = [{ score: 1 }, { score: 3 }];
const rank = (items) => items.sort((a, b) => b.score - a.score);
const ranked = rank(users);
console.log(ranked[0].score, users[0].score); // 3 3: the input changed
```

#### What Makes the Difference

If rank promises a new ordering, it must own the array it sorts. Sort mutates the input array. The second version is valid for an agreed in-place operation, but the caller must know that contract. Copying the array does not copy user objects.

### Preserving Effect Order

Evaluate only the required branch.

#### When to Use

Refactoring affects conditional calls, short-circuit evaluation, I/O, mutation, or error handling; ordering is part of behavior.

#### When It Is Excessive

An additional effect-management mechanism is unnecessary for a simple branch. Preserving behavior itself is never optional.

#### Simpler Alternative

Keep an explicit if/else or short ternary, evaluating only the required branch. Do not precompute effectful calls merely for presentation.

#### Good Example

```js
const calls = [];
const load = () => { calls.push('load'); return 'data'; };
const fallback = () => { calls.push('fallback'); return 'cached'; };
const ready = true;
const result = ready ? load() : fallback();
console.log(result, calls); // data ['load']
```

#### Bad Example

```js
const calls = [];
const load = () => { calls.push('load'); return 'data'; };
const fallback = () => { calls.push('fallback'); return 'cached'; };
const loaded = load();
const cached = fallback();
const result = true ? loaded : cached;
console.log(result, calls); // data ['load', 'fallback']
```

#### What Makes the Difference

A ternary evaluates only the selected branch. Extracting both calls into variables changes semantics: fallback runs unnecessarily. In real code this can mean an extra request or a side-effecting write.

### Composition over inheritance

Build variable behavior through delegation.

#### When to Use

A component uses a capability but is not a valid subtype of its implementation; behaviors must be replaceable independently.

#### When It Is Excessive

One direct call needs no composition object; inheritance may also already express a valid stable subtype relationship and framework extension point.

#### Simpler Alternative

Pass a function, call a module directly, or retain existing valid inheritance. Do not build multiple wrapper layers solely to follow the composition principle.

#### Good Example

```js
class UserService {
  constructor(transport) { this.transport = transport; }
  load(id) { return this.transport.get(`/users/${id}`); }
}
const service = new UserService({ get: (path) => `GET ${path}` });
console.log(service.load(1)); // GET /users/1
console.log(typeof service.close); // undefined: the unrelated API is not exposed
```

#### Bad Example

```js
class Transport {
  get(path) { return `GET ${path}`; }
  close() { this.closed = true; }
}
class UserService extends Transport {
  load(id) { return this.get(`/users/${id}`); }
}
const service = new UserService();
service.close(); // the caller gained transport control through the domain API
```

#### What Makes the Difference

The service uses a transport but is not a transport. Composition preserves a small public contract. Inheritance in the example exposes unrelated operations and ties domain API evolution to the transport class. A valid subtype relationship can justify inheritance.

### Explicit Dependencies / IoC and DI

Make the environment replaceable and testable.

#### When to Use

Time, I/O, storage, or external behavior must be substituted, varied between environments, or controlled by lifecycle.

#### When It Is Excessive

A DI container and interfaces for every function are unnecessary for a small local operation or a stable pure dependency.

#### Simpler Alternative

Pass one function or object as a parameter, or use a small factory with a closure. A direct import is valid for an immutable pure utility.

#### Good Example

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
console.log(service.start('anna').createdAt); // 1000: a deterministic test
```

#### Bad Example

```js
const storage = [];
const startSession = (userId) => {
  const session = { userId, createdAt: Date.now() };
  storage.push(session);
  return session;
};
console.log(startSession('anna').createdAt); // depends on the real clock
// The caller cannot inject different storage or a fixed clock.
```

#### What Makes the Difference

The first service receives the capabilities it needs explicitly. The test controls time and storage without replacing globals. In the second version, the environment is hidden inside the implementation. DI requires no container or framework: factory arguments suffice here.

### Separating Domain and System Code

Separate domain rules from I/O mechanics.

#### When to Use

A domain rule must be checked and reused independently of HTTP, database, and transport, or is already mixed with I/O details.

#### When It Is Excessive

Formal layers, DTO transitions, and separate packages are unnecessary for a small scenario without independent domain rules.

#### Simpler Alternative

Keep a short handler and extract one pure computation function. Do not introduce repository/service/use-case layers if they only forward calls.

#### Good Example

```js
const discountedTotal = (price, discount) => price * (1 - discount);
const handle = async (request, loadPrice) => {
  const price = await loadPrice(request.id);
  return { total: discountedTotal(price, request.discount) };
};
console.log(discountedTotal(100, 0.2)); // 80 without I/O
console.log(await handle({ id: 1, discount: 0.2 }, async () => 100));
```

#### Bad Example

```js
const discountedTotal = async (request, response, database) => {
  const price = await database.loadPrice(request.id);
  response.end(`${price * (1 - request.discount)}`);
};
const output = [];
await discountedTotal({ id: 1, discount: 0.2 },
  { end: (value) => output.push(value) }, { loadPrice: async () => 100 });
// Testing the formula requires assembling an HTTP and database environment.
```

#### What Makes the Difference

A pure domain formula is tested separately from transport and storage. In the second version, computation, database access, and response completion are fused: reusing the formula requires unrelated interfaces. Separation does not require many architectural layers.

### Contracts Before Mechanical Reuse

Reuse a library only when behavior matches.

#### When to Use

Local code will be replaced by a library helper or implementations combined; compare sync/async behavior, errors, defaults, units, and ownership.

#### When It Is Excessive

A separate audit process is excessive for an obviously identical pure operation. Checking the essential contract remains necessary when behavior changes.

#### Simpler Alternative

Keep a small local helper if the library does something different; call the library directly without a universal adapter when contracts match.

#### Good Example

```js
const readText = () => '{"id":1}';
const parseConfig = () => JSON.parse(readText());
console.log(parseConfig().id); // 1: both steps are synchronous
```

#### Bad Example

```js
const readTextAsync = async () => '{"id":1}';
const parseConfig = () => JSON.parse(readTextAsync());
try { parseConfig(); }
catch (error) { console.log(error.name); } // SyntaxError: attempted to parse a Promise
```

#### What Makes the Difference

Interchangeability follows from the contract, not a similar helper name. Replacing sync with async requires changes along the entire caller chain. Likewise, check UTC/local, throw/null, and fallback rules.

The source names GRASP, SOLID, contract programming, declarative DSLs,
referential transparency, and platform-agnostic code as guiding principles.
They do not form a separate detailed catalog there. For referential transparency,
for example, `const total = (items) => items.reduce((s, x) => s + x.price, 0)`
is a good choice; adding hidden mutation of a global sum inside that computation
while expecting identical results for identical inputs is a bad choice.

## 8. Performance and Measurement

Apply these techniques in the context of the target Node.js/V8 version and measured workload.

### Stable object shape

Stabilize the representation of records of the same kind.

#### When to Use

There are many records of the same kind and profiling identifies hot property accesses or allocation; consistent initialization does not harm the data model.

#### When It Is Excessive

The schema is inherently dynamic, code is not hot, or stabilizing it requires artificial fields and complicates the API without measured benefit.

#### Simpler Alternative

Create ordinary objects with a clear schema; use Map for dynamic keys. Do not introduce classes and factories solely for a presumed V8 optimization.

#### Good Example

```js
const createPoint = (x, y, label = null) => ({ x, y, label });
const first = createPoint(1, 2);
const second = createPoint(3, 4, 'target');
console.log(Object.keys(first), Object.keys(second));
// ['x', 'y', 'label'] for both records
```

#### Bad Example

```js
const first = { x: 1, y: 2 };
const second = { y: 4, x: 3 };
second.label = 'target';
delete second.x;
second.x = 3;
console.log(Object.keys(first), Object.keys(second)); // different shapes and orders
```

#### What Makes the Difference

The first version creates the same schema for records of the same kind from the start. The second needlessly changes the field set and order. This is a potential hot-access cost and schema complication, not a proven slowdown: measure on the target V8 version.

### Dense Arrays / Consistent Element Types

Avoid unnecessary holes and representation changes.

#### When to Use

Data is a dense sequence, and accidental holes or unexpected type conversions break traversal or interfere with measured hot processing.

#### When It Is Excessive

A truly sparse model is required or mixed types are part of the contract. Artificially filling a huge range can waste memory.

#### Simpler Alternative

Use a regular Array with push; do not micro-optimize a small set. Consider Map for sparse numeric indices instead of allocating a dense buffer.

#### Good Example

```js
const values = [];
for (let i = 0; i < 3; i++) values.push(i);
console.log(values.length, Object.keys(values)); // 3 ['0', '1', '2']
```

#### Bad Example

```js
const values = [0, 1, 2];
values[1000000] = 3;
let visited = 0;
values.forEach(() => { visited++; });
console.log(values.length, visited); // 1000001 4: almost all indices are holes
```

#### What Makes the Difference

A dense sequence has a value at every index. Writing far beyond the end changes length and creates holes; traversal and serialization methods may treat them differently. A sparse array is valid when the data model requires it.

### Monomorphic hot paths

Reduce type variation in a measured hot path.

#### When to Use

A measured hot operation regularly receives consistent types, and that contract can be maintained without losing the meaning of the model.

#### When It Is Excessive

Different types express a real requirement or the code does not affect performance. Adapter hierarchies introduced for monomorphism can cost more than they save.

#### Simpler Alternative

Keep ordinary polymorphic code with correct checks. Validating or normalizing numeric input at the boundary may suffice without changing the architecture.

#### Good Example

```js
const records = [{ amount: 10 }, { amount: 20 }];
const total = records.reduce((sum, record) => sum + record.amount, 0);
console.log(total); // 30: a numeric contract
```

#### Bad Example

```js
const records = [{ amount: 10 }, { amount: '20' }];
const total = records.reduce((sum, record) => sum + record.amount, 0);
console.log(total, typeof total); // '1020' string
```

#### What Makes the Difference

Consistent types simplify optimization and preserve numeric semantics. The second example already has a correctness bug: + performs concatenation. Not all polymorphism is wrong; performance conclusions require profiling and measurement.

### Reducing Escaping Allocations

Reduce allocations of objects that outlive an iteration.

#### When to Use

Profiling shows significant allocation/GC costs, and identical immutable data or callbacks are created in a hot loop and outlive its iterations.

#### When It Is Excessive

Objects are short-lived, load is low, or reuse would require unsafe sharing of mutable state. Allocation alone does not demonstrate a problem.

#### Simpler Alternative

Keep local allocations and GC. One extracted constant suffices for an obviously identical immutable table; introduce pooling only with separate justification.

#### Good Example

```js
const rules = Object.freeze({ read: 1, write: 2 });
const results = [];
for (const kind of ['read', 'write', 'read']) {
  results.push({ kind, rules });
}
console.log(new Set(results.map((item) => item.rules)).size); // 1 table
```

#### Bad Example

```js
const results = [];
for (const kind of ['read', 'write', 'read']) {
  const rules = { read: 1, write: 2 };
  results.push({ kind, rules });
}
console.log(new Set(results.map((item) => item.rules)).size); // 3 copies
```

#### What Makes the Difference

An immutable table can be shared safely. In the second version, every copy escapes into results and remains reachable after the iteration, increasing retained memory. Do not mechanically reuse a mutable result that must be independent.

### Measure Before Optimizing

Check the benefit on a real workload.

#### When to Use

A decision aims to improve speed or memory use, and correct alternatives must be compared on the target runtime and workload.

#### When It Is Excessive

A full benchmark suite is excessive for a change with no performance goal or for removing an obviously unnecessary action. Without measurement, do not promise a speedup.

#### Simpler Alternative

Start with an existing profiler and real metrics; a small reproducible comparison suffices for a narrow hypothesis instead of new benchmark infrastructure.

#### Good Example

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

#### Bad Example

```js
import { performance } from 'node:perf_hooks';
const values = [1, 2, 3];
const start = performance.now();
values.reduce((sum, n) => sum + n, 0);
console.log('Always fast:', performance.now() - start);
// One run, no baseline, warmup, or result validation.
```

#### What Makes the Difference

The good example shows result validation, repetition, warmup, and a median; comparison requires a second variant under comparable conditions. The bad example generalizes from one noisy cold call. Even a good microbenchmark does not replace profiling the real workload.

### Isolating Microbenchmarks

Avoid distorting comparisons through shared call-site state.

#### When to Use

Variants can affect shared JIT, GC, or call-site history, and the goal is to assess each under isolated conditions.

#### When It Is Excessive

The measurement targets a real mixed workload where shared type history is part of the scenario, or does not involve JIT state.

#### Simpler Alternative

Use one representative end-to-end benchmark or an existing harness. Separate processes address a specific distortion risk, not a mandatory rule for every measurement.

#### Good Example

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

#### Bad Example

```js
const read = (item) => item;
const cases = [[1, 2, 3], ['1', '2', '3']];
for (const values of cases) {
  for (let i = 0; i < 10000; i++) read(values[i % values.length]);
}
// Subsequent read measurements also depend on the shared call-site type history.
```

#### What Makes the Difference

A separate process isolates each variant's optimization history. A shared harness can mix types and affect subsequent measurements. These fragments demonstrate isolation, not a complete timed comparative benchmark.


---

This registry is based on the `metarhia/metaskills` repository at revision `ab13efe6c7ce56e8d576a3b7e42b90d408b68375`.
Compiled on: 2026-11-12.
