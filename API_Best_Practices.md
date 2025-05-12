
## API Automation Best Practices

### General Principles

*Inspired by: Clean Code by Robert C. Martin*

* Functions and flows should be small and do one thing.
* Names should reveal intent.
* Avoid duplication.
* Tests should be readable, fast, and independent.
* Avoid logical dependencies and magic values.

---

### 1. Keep Request Builders Focused

*Inspired by: SRP - Single Responsibility Principle*

* Separate responsibilities for request configuration, execution, and validation.

**✅ Good Structure:**

```ts
await fluentRest()
  .setBaseUrl('https://api.example.com')
  .givenAuth('Bearer token')
  .givenBody({ name: 'John' })
  .whenPost('/users')
  .thenExpectStatus(201);
```

**Why This Matters:**

* Keeps each function focused on a single concern.
* Encourages composability.
* Makes tests easier to understand and maintain.

---

### 2. Avoid Assertions in Wrappers

*Inspired by: Separation of Concerns*

* Let your test assert the outcome, not the utility.

**❌ Bad Practice:**

```ts
await myApiWrapper.createUser('John'); // internally asserts status 201
```

**✅ Good Practice:**

```ts
const res = await myApiWrapper.createUser('John');
res.thenExpectStatus(201);
```

**Why This Matters:**

* Keeps test ownership explicit.
* Makes error handling and negative testing more flexible.

---

### 3. Use Schema Validation for API Responses

*Inspired by: Self-verifying systems*

**❌ Bad Example:**

```ts
expect(res.body.name).toBe('John');
```

**✅ Good Example:**

```ts
await fluentRest()
  .whenGet('/users/123')
  .thenValidateBody(
    Joi.object({
      id: Joi.number().required(),
      name: Joi.string().required(),
      email: Joi.string().email()
    })
  );
```

**Why This Matters:**

* Reduces maintenance cost by validating structure instead of individual values.
* Immediately flags missing or malformed fields.

---

### 4. Provide Safe Error Handling

*Inspired by: Avoid rigidity and fragility*

* Wrap risky assertions and expose meaningful error logs.

**✅ Example:**

```ts
await response.catchAndLog(() => {
  expect(response.status).toBe(200);
  expect(response.data.length).toBeGreaterThan(0);
});
```

**Why This Matters:**

* Prevents a single failed assertion from hiding other valuable diagnostics.
* Encourages clearer failure output for debugging.

---

### 5. Abstract Test Setup, Not Assertions

*Inspired by: DRY and test clarity*

* Centralize config logic and avoid duplicating boilerplate.

**✅ Good Practice:**

```ts
function createRequestWithAuth() {
  return fluentRest()
    .setBaseUrl(API_BASE)
    .givenAuth(getToken());
}

await createRequestWithAuth()
  .whenGet('/profile')
  .thenExpectStatus(200);
```

---

### 6. Avoid Magic Strings and Numbers

*Inspired by: Replace magic numbers with named constants*

**❌ Bad Example:**

```ts
expect(response.status).toBe(201);
```

**✅ Good Example:**

```ts
const CREATED = 201;
expect(response.status).toBe(CREATED);
```

---

### 7. Prefer Named Utilities Over Inline Logic

*Inspired by: Code should explain itself*

**❌ Bad Example:**

```ts
const res = await fluentRest().whenPost('/users');
expect(JSONPath({ path: '$.id', json: res.getResponse().data })[0]).toBeDefined();
```

**✅ Good Example:**

```ts
function extractUserId(response) {
  return JSONPath({ path: '$.id', json: response.getResponse().data })[0];
}

const res = await fluentRest().whenPost('/users');
expect(extractUserId(res)).toBeDefined();
```

---

These practices ensure your API test suite remains clean, modular, expressive, and maintainable over time.

---

### 8. Design Reusable Fluent Flows Without Embedding Assertions

*Inspired by: Clean Code - Reusability and Separation of Concerns*

* If you create reusable flows like `login()`, they should *not* assert success internally.
* Allow each test to control its assertions, including negative cases.

**❌ Bad Example (login with internal assertion):**

```ts
async function login() {
  return fluentRest()
    .givenBody({ username: 'admin', password: 'secret' })
    .whenPost('/auth/login')
    .thenExpectStatus(200); // ❌ assertion here
}

// In test
await login(); // Fails test immediately if status is not 200, even in negative scenarios
```

**✅ Good Example (reusable login flow):**

```ts
function login() {
  return fluentRest()
    .givenBody({ username: 'admin', password: 'secret' })
    .whenPost('/auth/login');
}

// In success test
const res = await login();
res.thenExpectStatus(200);

// In failure test
const res = await login();
res.thenExpectStatus(401); // reused logic, different expectation
```

**Why This Matters:**

* Keeps reusable flows flexible for both positive and negative tests.
* Prevents unnecessary coupling of utilities and test logic.
* Encourages transparency: the test defines its expected outcome, not the helper.

**Bonus:**
You can still wrap reusable flows with optional assertions in the test layer if needed:

```ts
function loginAndExpectSuccess() {
  return login().thenExpectStatus(200);
}
```

This gives the best of both worlds: clean helper logic + optional clarity in test code.

---

### 9. Handle Test Data and State Cleanly

*Inspired by: Avoid shared mutable state and fragility*

* Avoid depending on pre-existing data in the environment.
* Use factories or setup steps to provision required data for each test.

**❌ Bad Example (dependent on external state):**

```ts
// Assumes user with ID 123 already exists
await fluentRest()
  .whenGet('/users/123')
  .thenExpectStatus(200);
```

**✅ Good Example (creates its own state):**

```ts
const createRes = await fluentRest()
  .givenBody({ name: 'John' })
  .whenPost('/users');

const userId = createRes.thenExtract('$.id');

await fluentRest()
  .whenGet(`/users/${userId}`)
  .thenExpectStatus(200);
```

**Why This Matters:**

* Ensures tests are self-contained and repeatable.
* Avoids reliance on test order or flaky data.

---

### 10. Make Environment and Configuration Explicit

*Inspired by: Eliminate ambiguity and magic behavior*

* Configuration should be passed clearly, not inferred from global state.
* Avoid global variables or hidden environment toggles.

**✅ Good Practice:**

```ts
configureDefaults({
  baseURL: process.env.API_BASE_URL,
  timeout: 10000,
  logLevel: 'debug'
});
```

* Document required variables and expected formats.
* Use `.env` or environment injection tools consistently.

**Why This Matters:**

* Helps CI and test runners behave consistently.
* Reduces onboarding time for new contributors.

---

### 11. Introduce Mocking for Unstable or External Dependencies

*Inspired by: Make tests fast, independent, and repeatable*

* Use mocking when testing APIs that rely on unstable third-party services, are slow, or involve payments/side effects.

**✅ Example using MSW or similar tooling:**

```ts
setupServer(
  rest.post('/auth/login', (req, res, ctx) => {
    return res(ctx.status(200), ctx.json({ token: 'mock-token' }));
  })
);
```

**Why This Matters:**

* Prevents flakiness from real external systems.
* Allows focused testing of your own logic.
* Reduces CI test time and avoids quota limits.

---

### 12. Validate Contracts in Integration or Pre-prod

*Inspired by: Don't rely on assumptions — verify structure early*

* Use tools like Pact or OpenAPI validators to verify the response format matches consumer expectations.

**✅ Contract Example using Joi in test:**

```ts
await fluentRest()
  .whenGet('/orders/789')
  .thenValidateBody(
    Joi.object({
      id: Joi.number(),
      items: Joi.array().min(1),
      total: Joi.number().required()
    })
  );
```

**Why This Matters:**

* Prevents breaking consumers with unnoticed changes.
* Encourages alignment between frontend/backend/API consumers.

---

### 13. Apply Test Layering: Smoke, Integration, Regression

*Inspired by: Separation of test intent and cost*

* Not all tests serve the same purpose. Categorize them for clarity and CI performance.

| Layer       | Purpose                   | Scope             |
| ----------- | ------------------------- | ----------------- |
| Smoke       | Health check on key flows | Fast, shallow     |
| Integration | Component + flow coverage | Business-critical |
| Regression  | Comprehensive coverage    | All edge cases    |

**Best Practice:**

* Use test tags or separate files/folders.
* Run smoke on every PR, integration daily, regression in full pipeline.

### 14. Use Descriptive and Searchable Test Names

*Inspired by: Names should reveal intent*

* Test names should describe what the test is validating.
* Helps identify failures quickly and improves readability in test reports.

**❌ Bad Example:**

```ts
test('test1', async () => { ... });
```

**✅ Good Example:**

```ts
test('Should return 401 for invalid login credentials', async () => { ... });
```

**Why This Matters:**

* Clarifies test purpose at a glance.
* Improves traceability in CI reports and logs.

---

### 15. Integrate with CI to Run the Right Tests at the Right Time

*Inspired by: Fast feedback loops and scalability*

* Integrate your test strategy with GitLab/GitHub Actions/Jenkins or similar.

**✅ Example:**

* Run smoke tests on pull request.
* Run integration tests on merges.
* Run regression tests on scheduled/nightly builds.

```yml
# Sample GitHub Actions job
jobs:
  run-api-tests:
    runs-on: ubuntu-latest
    steps:
      - name: Run smoke tests
        run: npm run test:smoke
```

**Why This Matters:**

* Ensures fast feedback.
* Prevents regressions from merging into main.
* Optimizes use of CI/CD resources.

---

### 16. Implement Effective Logging and Debugging Tools

*Inspired by: Understandability and maintainability*

* Log request/response metadata.
* Include correlation IDs, timing, headers, and error codes.
* Avoid logging sensitive data.

**✅ Good Example:**

```ts
configureDefaults({
  logLevel: 'debug',
  logToFile: true
});
```

* Snapshot full requests/responses on failure.
* Use meaningful error codes in custom assertions (e.g., `ERR_VALIDATION_SCHEMA`).

**Why This Matters:**

* Simplifies root cause analysis.
* Helps debug flaky tests and environment-specific failures.

---

### 17. How to Balance Best Practices Without Conflict

*Inspired by: Clean Code and Practical Testing Tradeoffs*

Some best practices might appear to conflict at first glance, but they actually complement each other when applied with intent:

| Practice A                         | Practice B                             | How to Balance                                                                |
| ---------------------------------- | -------------------------------------- | ----------------------------------------------------------------------------- |
| Avoid assertions in reusable flows | Reuse common `fluentRest()` calls      | Keep assertions in test layer, but offer optional `.thenExpectXYZ()` wrappers |
| Use mocking for external services  | Validate real contracts with consumers | Use mocks in local/unit tests, and contract tests in integration/staging      |
| Create isolated test data          | Avoid duplication (DRY)                | Use shared factories/setup helpers to reduce code repetition                  |
| Abstract setup logic               | Avoid hiding assertions                | Configure base/auth/etc. once; leave result validation in the test            |

**Why This Matters:**

* Clean code is not about rigid rules — it's about *clarity, intent, and maintainability*.
* Conflicting principles often reflect different goals (readability vs. reuse vs. flexibility).
* Use comments, naming, and structure to make your intentions clear.

By recognizing these tradeoffs and designing with clarity, you can confidently apply these best practices in a practical, scalable way.

---
