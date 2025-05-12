## General Clean Code Best Practices

### Based on "Clean Code" by Robert C. Martin

> *"Code is clean if it can be understood easily – by everyone on the team. Clean code can be read and enhanced by a developer other than its original author."*

---

### 📐 General Principles

* Follow standard conventions.
* Keep it simple — avoid unnecessary complexity.
* Apply the "Boy Scout Rule": Leave code better than you found it.
* Always search for the root cause of problems before patching symptoms.

---

### 🏗️ Design Principles

* Keep configurable data at high levels.
* Prefer polymorphism to `if`/`else` or `switch`/`case` statements.
* Separate multi-threaded concerns.
* Avoid over-configurability.
* Use dependency injection.
* Follow the Law of Demeter — a class should only know about its direct dependencies.

---

### 🔍 Understandability Tips

* Be consistent — do similar things in the same way.
* Use explanatory variables.
* Encapsulate boundary conditions in a single place.
* Prefer dedicated value objects over primitive types.
* Avoid logical dependencies between methods.
* Avoid negative conditionals.

---

### 🏷 Naming Conventions

* Choose descriptive and unambiguous names.
* Make meaningful distinctions between variables.
* Use pronounceable and searchable names.
* Replace magic numbers with named constants.
* Avoid encoded prefixes or type-based naming (e.g., `strName`, `iCount`).

---

### 🧩 Function Guidelines

* Keep functions small.
* Each function should do one thing.
* Use descriptive names.
* Prefer fewer arguments.
* Avoid side effects.
* Do not use flag arguments. Instead, split into multiple meaningful functions.

---

### 📝 Comments

* Prefer code that explains itself.
* Avoid redundant, obvious, or noisy comments.
* Don’t comment out code — delete it.
* Use comments to clarify *intent*, warn of consequences, or explain complex logic.

---

### 🧱 Code Structure

* Separate concepts vertically.
* Keep related code vertically dense.
* Declare variables close to where they’re used.
* Keep dependent and similar functions together.
* Order functions downward in a logical reading flow.
* Avoid horizontal alignment; use whitespace meaningfully.
* Maintain consistent indentation.

---

### 🧵 Objects and Data Structures

* Hide internal data structures.
* Prefer data structures or objects — not hybrids.
* Keep objects small and focused.
* Favor many methods over passing logic as parameters.
* Prefer non-static methods unless static is necessary.
* Ensure base classes know nothing about their children.

---

### ✅ Tests

* One assertion per test.
* Make tests readable, fast, independent, and repeatable.

---

### 🛑 Code Smells

* **Rigidity** – hard to change.
* **Fragility** – one change breaks multiple places.
* **Immobility** – hard to reuse.
* **Needless Complexity** – overengineering.
* **Needless Repetition** – copy-paste logic.
* **Opacity** – code is hard to understand.

---

---

### 💡 Examples of Clean Code in Practice

Below are examples grouped by Clean Code sections, each with bad and good implementations.

Below are examples demonstrating good and bad practices for each principle.

#### ✅ Descriptive and Unambiguous Names

**❌ Bad Example:**

```ts
let d = new Date();
```

**✅ Good Example:**

```ts
let currentTimestamp = new Date();
```

#### ✅ Use Explanatory Variables

**❌ Bad Example:**

```ts
const a = 5 * 24 * 60 * 60;
```

**✅ Good Example:**

```ts
const secondsPerDay = 86400;
```

#### ✅ Replace Magic Numbers with Named Constants

**❌ Bad Example:**

```ts
if (status === 4) return 'Suspended';
```

**✅ Good Example:**

```ts
const STATUS_SUSPENDED = 4;
if (status === STATUS_SUSPENDED) return 'Suspended';
```

#### ✅ Small, Focused Functions

**❌ Bad Example:**

```ts
function handleUserData(user) {
  validateUser(user);
  saveUser(user);
  notifyUser(user);
}
```

**✅ Good Example:**

```ts
function validateUser(user) { ... }
function saveUser(user) { ... }
function notifyUser(user) { ... }
```

#### ✅ Prefer Fewer Arguments

**❌ Bad Example:**

```ts
createUser('John', 'Doe', 'john.doe@example.com', 'admin', true);
```

**✅ Good Example:**

```ts
createUser({
  firstName: 'John',
  lastName: 'Doe',
  email: 'john.doe@example.com',
  role: 'admin',
  isActive: true
});
```

#### ✅ No Flag Arguments

**❌ Bad Example:**

```ts
function formatUser(user, isShort) {
  return isShort ? user.name : `${user.name} (${user.email})`;
}
```

**✅ Good Example:**

```ts
function formatShortUser(user) {
  return user.name;
}

function formatDetailedUser(user) {
  return `${user.name} (${user.email})`;
}
```

#### ✅ One Assertion per Test

**❌ Bad Example:**

```ts
test('user API returns expected data', () => {
  expect(res.status).toBe(200);
  expect(res.body.name).toBe('John');
});
```

**✅ Good Example:**

```ts
test('user API returns 200 status', () => {
  expect(res.status).toBe(200);
});

test('user API returns correct name', () => {
  expect(res.body.name).toBe('John');
});
```

#### ✅ Follow Standard Conventions

**❌ Bad Example:**

```ts
Function RunUSERlogin() {
  // badly capitalized, inconsistent
}
```

**✅ Good Example:**

```ts
function runUserLogin() {
  // consistent, camelCase
}
```

#### ✅ Keep Configurable Data at High Levels

**❌ Bad Example:**

```ts
function connect() {
  const dbUrl = 'mongodb://localhost:27017/dev';
}
```

**✅ Good Example:**

```ts
const config = { dbUrl: process.env.DB_URL };
function connect() {
  const dbUrl = config.dbUrl;
}
```

#### ✅ Prefer Polymorphism to Switch/Case

**❌ Bad Example:**

```ts
switch (animal.type) {
  case 'dog': return 'bark';
  case 'cat': return 'meow';
}
```

**✅ Good Example:**

```ts
class Animal { speak() {} }
class Dog extends Animal { speak() { return 'bark'; } }
class Cat extends Animal { speak() { return 'meow'; } }
```

#### ✅ Avoid Over-Configurability

**❌ Bad Example:**

```ts
function sendNotification(options) {
  if (options.useSlack && options.useEmail && options.useSms) { ... }
}
```

**✅ Good Example:**

```ts
function sendNotification(channel: 'slack' | 'email' | 'sms') { ... }
```

#### ✅ Use Dependency Injection

**❌ Bad Example:**

```ts
class Service {
  constructor() {
    this.db = new Database();
  }
}
```

**✅ Good Example:**

```ts
class Service {
  constructor(db) {
    this.db = db;
  }
}
```

#### ✅ Law of Demeter

The Law of Demeter ("Principle of Least Knowledge") encourages minimal knowledge between classes.

> A method should only call methods on:
>
> * itself
> * its own fields
> * objects passed in as arguments
> * objects it creates internally

This avoids chaining multiple method calls across unrelated objects.

**❌ Bad Example:**

```ts
user.getProfile().getAddress().getCity(); // too much knowledge of internal structure
```

**✅ Good Example:**

```ts
user.getCity(); // user internally delegates to profile/address if needed
```

**Why This Matters:**

* Reduces coupling
* Increases encapsulation
* Makes refactoring and testing easier

#### ✅ Use Value Objects Instead of Primitives

**❌ Bad Example:**

```ts
function calculateTotal(price: number, tax: number) { ... }
```

**✅ Good Example:**

```ts
class Money {
  constructor(public amount: number) {}
}
function calculateTotal(price: Money, tax: Money) { ... }
```

#### ✅ Avoid Logical Dependencies

**❌ Bad Example:**

```ts
this.methodA();
if (this.stateSetByA) this.methodB();
```

**✅ Good Example:**

```ts
const result = this.methodA();
this.methodB(result);
```

#### ✅ Avoid Negative Conditionals

**❌ Bad Example:**

```ts
if (!isValid) { return; }
```

**✅ Good Example:**

```ts
if (isValid) {
  // proceed
}
```

These examples demonstrate how clean code isn't just theoretical — it directly improves clarity, maintainability, and collaboration.
