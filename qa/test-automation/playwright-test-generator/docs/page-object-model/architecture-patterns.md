# Architecture Patterns

> **Purpose**: Choose between Standard Page Object Pattern and App Fixture Pattern based on your project needs. This guide helps you understand the trade-offs and implementation details of each approach.

## Pattern Comparison

| Aspect             | Standard Pattern                   | App Fixture Pattern        |
| ------------------ | ---------------------------------- | -------------------------- |
| **Best For**       | Small-medium projects (< 50 tests) | Large projects (50+ tests) |
| **Page Objects**   | Manual instantiation               | Centralized management     |
| **Complexity**     | Simple, explicit                   | More setup, powerful       |
| **Learning Curve** | Easy                               | Moderate                   |
| **Maintenance**    | More boilerplate                   | Less repetition            |

## Standard Page Object Pattern

### When to Use

- Small to medium test suites
- Simple page interactions
- Team prefers explicit control
- Getting started with Page Object Model

### Implementation

```typescript
// tests/login.spec.ts
import { test, expect } from '@playwright/test';
import { LoginPage } from '../pages/login/login.page';
import { DashboardPage } from '../pages/dashboard/dashboard.page';

test('user login flow', async ({ page }) => {
  // Manual instantiation
  const loginPage = new LoginPage(page);
  const dashboardPage = new DashboardPage(page);

  await loginPage.goto();
  await loginPage.login('user@example.com', 'password');
  await expect(dashboardPage.welcomeMessage).toBeVisible();
});
```

### Pros & Cons

**✅ Advantages:**

- Simple and straightforward
- No additional setup required
- Easy to understand for beginners
- Full control over instantiation

**❌ Disadvantages:**

- Repetitive instantiation code
- More boilerplate in tests
- Harder to manage many page objects
- No centralized configuration

## App Fixture Pattern (Recommended for Large Projects)

### When to Use

- Large test suites (50+ tests)
- Many page objects (10+ pages)
- Complex cross-page workflows
- Want centralized page object management
- Need shared test data or utilities

### Implementation

#### 1. Create Fixtures

```typescript
// fixtures/fixtures.ts
import { test as base, expect } from '@playwright/test';
import { App } from '../pages/app.page';

export const test = base.extend<{ app: App }>({
  app: async ({ page }, use, testInfo) => {
    // Optional: Add error tracking
    page.on('pageerror', (error) => {
      console.log(`JavaScript error in ${testInfo.title}: ${error.message}`);
    });

    // Create and provide app instance
    const app = new App(page);
    await use(app);
  },
});

export { expect };
```

#### 2. Create App Aggregator

```typescript
// pages/app.page.ts
import { type Page } from '@playwright/test';
import { TestData } from '../test_data/test.data';

// Import all page objects
import { LoginPage } from './auth/login.page';
import { DashboardPage } from './dashboard/dashboard.page';
import { SearchPage } from './search/search.page';
import { MonitorPage } from './monitor/monitor.page';

export class App {
  readonly page: Page;
  readonly testData: TestData;

  // Declare all page objects
  readonly loginPage: LoginPage;
  readonly dashboardPage: DashboardPage;
  readonly searchPage: SearchPage;
  readonly monitorPage: MonitorPage;

  constructor(page: Page) {
    this.page = page;
    this.testData = new TestData();

    // Instantiate all page objects
    this.loginPage = new LoginPage(page);
    this.dashboardPage = new DashboardPage(page);
    this.searchPage = new SearchPage(page);
    this.monitorPage = new MonitorPage(page);
  }
}
```

#### 3. Use in Tests

```typescript
// tests/login.spec.ts
import { test, expect } from '../fixtures/fixtures';

test('user login flow', async ({ app }) => {
  // All page objects available through app
  await app.loginPage.goto();
  await app.loginPage.login('user@example.com', 'password');
  await expect(app.dashboardPage.welcomeMessage).toBeVisible();

  // Access test data
  await app.searchPage.search(app.testData.searchTerms.johnDoe);
});
```

### Advanced App Fixture Features

#### Error Tracking

```typescript
// fixtures/fixtures.ts - Enhanced version
export const test = base.extend<{ app: App }>({
  app: async ({ page }, use, testInfo) => {
    const jsErrors: string[] = [];
    const consoleWarnings: string[] = [];

    // Capture JavaScript errors
    page.on('pageerror', (error) => {
      const errorMsg = `JS Error: ${error.message}`;
      jsErrors.push(errorMsg);
      console.log(errorMsg);
    });

    // Capture console warnings
    page.on('console', (msg) => {
      if (msg.type() === 'warning') {
        consoleWarnings.push(msg.text());
      }
    });

    const app = new App(page);
    await use(app);

    // Report errors after test
    if (jsErrors.length > 0) {
      console.log(`Test ${testInfo.title} had ${jsErrors.length} JS errors`);
    }
  },
});
```

#### Shared Test Data

```typescript
// test_data/test.data.ts
export class TestData {
  readonly users = {
    admin: { username: 'admin@example.com', password: 'admin123' },
    regular: { username: 'user@example.com', password: 'user123' },
  };

  readonly searchTerms = {
    johnDoe: 'John Doe',
    janeDoe: 'Jane Doe',
  };

  readonly monitorLists = {
    testList: 'Test Monitor List',
    negativeList: 'Negative News List',
  };
}

// Usage in tests
test('search with test data', async ({ app }) => {
  await app.searchPage.search(app.testData.searchTerms.johnDoe);
  await app.monitorPage.createList(app.testData.monitorLists.testList);
});
```

### Pros & Cons

**✅ Advantages:**

- Centralized page object management
- Less boilerplate in tests
- Shared test data and utilities
- Easier to manage large test suites
- Built-in error tracking capabilities
- Consistent page object availability

**❌ Disadvantages:**

- More initial setup required
- Higher learning curve
- Additional abstraction layer
- Overkill for small projects

## Migration Strategy

### From Standard to App Fixture

1. **Create fixtures.ts**

```typescript
// Start simple
export const test = base.extend<{ app: App }>({
  app: async ({ page }, use) => {
    await use(new App(page));
  },
});
```

2. **Create app.page.ts gradually**

```typescript
// Start with key pages
export class App {
  readonly loginPage: LoginPage;
  readonly dashboardPage: DashboardPage;

  constructor(page: Page) {
    this.loginPage = new LoginPage(page);
    this.dashboardPage = new DashboardPage(page);
  }
}
```

3. **Update tests incrementally**

```typescript
// Before
test('login', async ({ page }) => {
  const loginPage = new LoginPage(page);
  // ...
});

// After
test('login', async ({ app }) => {
  // ...
});
```

### Backward Compatibility

```typescript
// Keep both patterns during migration
export class App {
  constructor(public readonly page: Page) {
    // App fixture properties
  }

  // Static methods for standard pattern compatibility
  static createLoginPage(page: Page): LoginPage {
    return new LoginPage(page);
  }
}
```

## Best Practices

### App Fixture Organization

```typescript
// Group related page objects
export class App {
  // Authentication
  readonly loginPage: LoginPage;
  readonly registrationPage: RegistrationPage;

  // Main application
  readonly dashboardPage: DashboardPage;
  readonly searchPage: SearchPage;

  // Admin
  readonly adminDashboard: AdminDashboardPage;
  readonly userManagement: UserManagementPage;
}
```

### Conditional Page Objects

```typescript
// Only instantiate when needed
export class App {
  private _adminPages?: AdminPages;

  get adminPages(): AdminPages {
    if (!this._adminPages) {
      this._adminPages = new AdminPages(this.page);
    }
    return this._adminPages;
  }
}
```

## Decision Matrix

Choose your pattern based on these factors:

| Factor               | Standard       | App Fixture   |
| -------------------- | -------------- | ------------- |
| Team size            | 1-3 developers | 3+ developers |
| Test count           | < 50 tests     | 50+ tests     |
| Page objects         | < 10 pages     | 10+ pages     |
| Cross-page workflows | Simple         | Complex       |
| Test data sharing    | Minimal        | Extensive     |
| Error tracking       | Basic          | Advanced      |

---

_Choose the pattern that best fits your current needs. You can always migrate from Standard to App Fixture as your test suite grows._
