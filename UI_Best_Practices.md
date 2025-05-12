## UI Automation Best Practices

### General Best Practices for UI Testing (Playwright + TypeScript)

#### 1. Use Intention-Revealing Selectors

*Inspired by: Clean Code - "Names should reveal intent"*

* Prefer `data-testid` or accessible roles for targeting elements.
* Avoid fragile selectors based on structure or CSS.

**❌ Bad Example:**

```ts
await page.click('div:nth-child(2) > span.button');
```

**✅ Good Example:**

```ts
await page.locator('[data-testid="submit-button"]').click();
```

---

#### 2. Avoid Hard Waits (Flaky Tests)

*Inspired by: Keep it simple / eliminate unnecessary complexity*

* Use Playwright's auto-waiting features.

**❌ Bad Example:**

```ts
await page.waitForTimeout(3000);
```

**✅ Good Example:**

```ts
await expect(page.locator('.message')).toHaveText('Success');
```

---

#### 3. One Test = One Responsibility

*Inspired by: Single Responsibility Principle (SRP)*

* Tests should verify one specific behavior.

**❌ Bad Example:**

```ts
test('User logs in, creates post, and logs out', async () => { ... });
```

**✅ Good Example:**

```ts
test('User can log in', async () => { ... });
test('User can create post', async () => { ... });
test('User can log out', async () => { ... });
```

---

#### 4. Use Meaningful Test Names

*Inspired by: Code should be self-explanatory*

* Write test names like use case descriptions.

**❌ Bad Example:**

```ts
test('Test 01', async () => { ... });
```

**✅ Good Example:**

```ts
test('Should display error on invalid credentials', async () => { ... });
```

---

#### 5. Structure Test Suites Logically

*Inspired by: Vertical structure, clarity, and order*

* Group tests by feature, page, or business scenario.
* Use `describe()` to organize related tests into logical blocks.
* Place common setup in `beforeEach()` to reduce duplication.

**✅ Example:**

```ts
describe('User Authentication', () => {
  beforeEach(async () => {
    await page.goto('/login');
  });

  test('should log in with valid credentials', async () => {
    await page.fill('#username', 'admin');
    await page.fill('#password', 'secret');
    await page.click('button[type="submit"]');
    await expect(page.locator('.success')).toBeVisible();
  });

  test('should show error with invalid credentials', async () => {
    await page.fill('#username', 'wrong');
    await page.fill('#password', 'wrong');
    await page.click('button[type="submit"]');
    await expect(page.locator('.error')).toBeVisible();
  });
});
```

**Why This Matters:**

* Logical grouping improves readability and navigation.
* Encourages shared setup/teardown.
* Makes test intent easier to understand and maintain.

---

#### 6. Abstract Repetitive Logic

*Inspired by: DRY - Don't Repeat Yourself*

* Use helper functions for repetitive login/setup tasks.

**❌ Bad Example:**

```ts
await page.goto('/login');
await page.fill('#user', 'admin');
await page.fill('#pass', '123');
await page.click('button');
```

**✅ Good Example:**

```ts
await loginAsAdmin(page);
```

---

### Page Object Model (POM) Best Practices

---

### Isolate Browser Context Per Test

*Inspired by: Test isolation and repeatability*

* Each test should use a fresh browser context (or even browser instance) to prevent shared state, session leakage, and side effects.

**Why This Matters:**

* Tests become independent and reproducible.
* Parallel execution is safer and avoids cross-test contamination.
* Login sessions, local storage, and cookies stay scoped to the test.

**❌ Bad Practice:**

* Reusing the same browser/page across multiple unrelated tests.

```ts
let browser;
let page;

beforeAll(async () => {
  browser = await chromium.launch();
  page = await browser.newPage();
});

// Tests may accidentally depend on shared session or leftover data
```

**✅ Good Practice:**

* Launch a new context or page per test to guarantee isolation.

```ts
test('user A can login', async ({ browser }) => {
  const context = await browser.newContext();
  const page = await context.newPage();
  // ...test logic
});

test('user B login fails', async ({ browser }) => {
  const context = await browser.newContext();
  const page = await context.newPage();
  // ...test logic
});
```

**Playwright Built-in Fixture Example:**

* Use Playwright's built-in `page` fixture which handles this automatically.

```ts
test('should navigate securely', async ({ page }) => {
  await page.goto('/secure');
});
```

**Caveats of Shared Browser Context:**

* One test’s failure might corrupt the state for others.
* You may unknowingly rely on cookies, session state, or DOM remnants.
* Tests become order-dependent (flaky in parallel or CI runs).

---

#### 1. Don’t Include Assertions in Page Object Methods

*Inspired by: SRP and Separation of Concerns*

* Page Objects should encapsulate UI structure and actions — not test logic.
* Assertions should live in the test files to clearly express test intent.

**Why This Matters:**

* Keeps tests flexible: you can reuse POM methods across many tests.
* If assertions are embedded in POMs, they may assert things irrelevant or even harmful in certain scenarios.
* Makes test failures easier to debug: assertion context is near the test.
* Prevents POM methods from blocking negative test cases by enforcing a specific success path.

**Common Problem:**
When assertions are built into POM methods, they can interfere with tests meant to validate negative or edge-case scenarios.

**❌ Bad Example (Blocks Negative Testing):**

```ts
async login(user: string, pass: string) {
  await this.page.fill('#username', user);
  await this.page.fill('#password', pass);
  await this.page.click('#submit');
  await expect(this.page.locator('.success')).toBeVisible(); // ❌ always expects success
}

// In test
await loginPage.login('lockedUser', 'wrongPass');
// This test will fail prematurely even if the goal is to check for an error
```

**✅ Good Example (Flexible for Negative Testing):**

```ts
async login(user: string, pass: string) {
  await this.page.fill('#username', user);
  await this.page.fill('#password', pass);
  await this.page.click('#submit');
}

// In test - success path
await loginPage.login('admin', 'adminpass');
await expect(page.locator('.success')).toBeVisible();

// In test - failure path
await loginPage.login('lockedUser', 'wrongPass');
await expect(page.locator('.error')).toContainText('Account is locked');
```

**Benefits of Keeping Assertions in the Test:**

* Assertions match the intent of the individual test case.
* You can express both positive and negative expectations clearly.
* You avoid maintaining multiple variants of the same POM method just to bypass built-in assertions.

Corrected Reusable Design:\*\*

```ts
await loginPage.login('lockedUser', 'wrongPass');
await expect(page.locator('.error')).toBeVisible();
```

---

#### 2. Keep POMs Simple and Focused

*Inspired by: "Functions should do one thing"*

* Avoid chaining multiple business flows in a single method.
* Each method in the POM should reflect one user action or one UI control.

**❌ Bad Example:**

```ts
async completeUserRegistration(user) {
  await this.fillForm(user);
  await this.submit();
  await this.confirmEmail();
  await expect(this.page.locator('.dashboard')).toBeVisible();
}
```

**✅ Good Example:**

```ts
await registrationPage.fillForm(user);
await registrationPage.submit();
await registrationPage.confirmEmail();
await expect(page.locator('.dashboard')).toBeVisible();
```

**Why This Matters:**

* Keeps page methods test-agnostic.
* Allows reuse of smaller actions in different combinations.
* Encourages separation between UI behavior and test flow.

---

#### 3. Use Getters for Locators

*Inspired by: Clarity and intent revealing*

* Group and name locators logically.

**✅ Good Example:**

```ts
get usernameField() {
  return this.page.locator('#username');
}
```

---

#### 4. Avoid Deep Logic in POMs

* Let test files define *how* tests are executed.

**❌ Bad Example:**

```ts
async registerUser(user) {
  await this.fillForm(user);
  await this.submit();
  await this.verifySuccess();
}
```

**✅ Good Example:**

```ts
await registrationPage.fillForm(user);
await registrationPage.submit();
await expect(page.locator('.success')).toBeVisible();
```

---

#### 5. Favor Composition Over Inheritance in POMs

*Inspired by: Prefer composition to inheritance*

* Inheritance leads to tight coupling and fragile base classes.
* Composition allows you to combine small, focused behaviors flexibly.

**❌ Bad Example (Inheritance):**

```ts
class BasePage {
  constructor(protected page: Page) {}

  async navigateTo(path: string) {
    await this.page.goto(path);
  }
}

class LoginPage extends BasePage {
  async login(user: string, pass: string) {
    await this.page.fill('#username', user);
    await this.page.fill('#password', pass);
    await this.page.click('button[type="submit"]');
  }
}
```

**✅ Good Example (Composition):**

```ts
class Navigator {
  constructor(private page: Page) {}
  async goTo(path: string) {
    await this.page.goto(path);
  }
}

class LoginPage {
  private navigator: Navigator;

  constructor(private page: Page) {
    this.navigator = new Navigator(page);
  }

  async login(user: string, pass: string) {
    await this.page.fill('#username', user);
    await this.page.fill('#password', pass);
    await this.page.click('button[type="submit"]');
  }

  async open() {
    await this.navigator.goTo('/login');
  }
}
```

#### Dependency Injection (DI) Example

*Inspired by: Clean Code - Dependency Inversion Principle*

* DI allows for better testability and flexibility.

**✅ Good Example:**

```ts
class LoginPage {
  constructor(
    private page: Page,
    private navigator: Navigator // injected
  ) {}

  async open() {
    await this.navigator.goTo('/login');
  }

  async login(username: string, password: string) {
    await this.page.fill('#username', username);
    await this.page.fill('#password', password);
    await this.page.click('button[type="submit"]');
  }
}

// Usage
const navigator = new Navigator(page);
const loginPage = new LoginPage(page, navigator);
```

---

#### 6. Co-locate Tests and POMs by Feature

* Improves discoverability and organization.
* Example:

```ts
tests/login/login.spec.ts
tests/login/LoginPage.ts
```
