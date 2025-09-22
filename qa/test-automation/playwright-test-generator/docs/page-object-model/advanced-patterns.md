# Advanced Patterns

**Description**: Advanced architectural patterns and strategies for scaling Page Object Model implementations, including App fixture pattern, component reusability, error handling, accessibility integration, and maintenance strategies for large test suites.

## Table of Contents

- [App Fixture Pattern](#app-fixture-pattern)
- [Base Page Pattern](#base-page-pattern)
- [Component Pattern](#component-pattern)
- [Constants and Data Management](#constants-and-data-management)
- [Method Chaining](#method-chaining)
- [Error Handling Strategies](#error-handling-strategies)
- [Accessibility Integration](#accessibility-integration)
- [Testing Strategies](#testing-strategies)
- [Maintenance Patterns](#maintenance-patterns)

## App Fixture Pattern

The App fixture pattern centralizes all page objects, providing a single entry point for tests. Ideal for large test suites.

### Implementation

```typescript
// fixtures/fixtures.ts
import { test as base } from '@playwright/test';
import { App } from '../pages/app.page';

export const test = base.extend<{ app: App }>({
  app: async ({ page }, use, testInfo) => {
    // Add global error tracking
    page.on('pageerror', (error) => {
      console.log(`JavaScript error in ${testInfo.title}: ${error.message}`);
    });

    // Add request monitoring if needed
    page.on('requestfailed', (request) => {
      console.log(`Failed request: ${request.url()}`);
    });

    const app = new App(page);
    await use(app);
  },
});

export { expect } from '@playwright/test';
```

### Central App Class

```typescript
// pages/app.page.ts - Central page object aggregator
import { type Page } from '@playwright/test';
import { LoginPage } from './auth/login.page';
import { DashboardPage } from './dashboard/dashboard.page';
import { SearchPage } from './search/search.page';
import { MonitorPage } from './monitor/monitor.page';
import { ToolsPage } from './tools/tools.page';

export class App {
  readonly page: Page;

  // Authentication pages
  readonly loginPage: LoginPage;

  // Main application pages
  readonly dashboardPage: DashboardPage;
  readonly searchPage: SearchPage;
  readonly monitorPage: MonitorPage;
  readonly toolsPage: ToolsPage;

  constructor(page: Page) {
    this.page = page;

    // Instantiate all page objects
    this.loginPage = new LoginPage(page);
    this.dashboardPage = new DashboardPage(page);
    this.searchPage = new SearchPage(page);
    this.monitorPage = new MonitorPage(page);
    this.toolsPage = new ToolsPage(page);
  }

  // Global navigation helpers
  async navigate(path: string): Promise<void> {
    await this.page.goto(path);
    await this.page.waitForLoadState('domcontentloaded');
  }

  // Common workflows that span multiple pages
  async loginAndNavigateTo(credentials: LoginCredentials, path: string): Promise<void> {
    await this.loginPage.goto();
    await this.loginPage.login(credentials.username, credentials.password);
    await this.navigate(path);
  }
}
```

### Usage in Tests

```typescript
// tests/search/advanced-search.spec.ts
import { test, expect } from '../../fixtures/fixtures';

test('user can perform advanced search with filters', async ({ app }) => {
  // Clean, centralized access to all page objects
  await app.loginPage.goto();
  await app.loginPage.login('testuser', 'password');

  await app.searchPage.goto();
  await app.searchPage.performAdvancedSearch({
    query: 'John Doe',
    category: 'individuals',
    dateRange: { from: new Date('2023-01-01'), to: new Date('2023-12-31') },
  });

  const resultCount = await app.searchPage.getResultCount();
  expect(resultCount).toBeGreaterThan(0);
});
```

## Base Page Pattern

Shared functionality across all page objects. Use sparingly to avoid over-engineering.

```typescript
// pages/base.page.ts
import { type Page, type Locator } from '@playwright/test';

export abstract class BasePage {
  readonly page: Page;

  // Common elements present on most pages
  readonly loadingIndicator: Locator;
  readonly errorNotification: Locator;
  readonly successNotification: Locator;
  readonly navigationMenu: Locator;
  readonly userMenu: Locator;

  constructor(page: Page) {
    this.page = page;
    this.loadingIndicator = page.locator('.loading-spinner');
    this.errorNotification = page.locator('.error-notification');
    this.successNotification = page.locator('.success-notification');
    this.navigationMenu = page.getByRole('navigation');
    this.userMenu = page.getByTestId('user-menu');
  }

  // Common utility methods
  async waitForPageLoad(): Promise<void> {
    await this.page.waitForLoadState('domcontentloaded');
    await this.loadingIndicator.waitFor({ state: 'hidden' }).catch(() => {
      // Loading indicator might not exist on all pages
    });
  }

  async getPageTitle(): Promise<string> {
    return await this.page.title();
  }

  async refreshPage(): Promise<void> {
    await this.page.reload();
    await this.waitForPageLoad();
  }

  async getErrorMessage(): Promise<string | null> {
    try {
      await this.errorNotification.waitFor({ state: 'visible', timeout: 1000 });
      return await this.errorNotification.textContent();
    } catch {
      return null;
    }
  }

  async getSuccessMessage(): Promise<string | null> {
    try {
      await this.successNotification.waitFor({ state: 'visible', timeout: 1000 });
      return await this.successNotification.textContent();
    } catch {
      return null;
    }
  }
}

// Implementation in concrete page
export class SearchPage extends BasePage {
  readonly searchField: Locator;
  readonly searchButton: Locator;

  constructor(page: Page) {
    super(page);
    this.searchField = page.getByPlaceholder('Enter search term');
    this.searchButton = page.getByRole('button', { name: 'Search' });
  }

  async goto(): Promise<void> {
    await this.page.goto('/search');
    await this.waitForPageLoad(); // Inherited method
  }

  async performSearch(query: string): Promise<void> {
    await this.searchField.fill(query);
    await this.searchButton.click();
    await this.waitForPageLoad();
  }
}
```

## Component Pattern

For reusable UI components that appear across multiple pages.

```typescript
// components/modal.component.ts
import { type Page, type Locator } from '@playwright/test';

export class ModalComponent {
  readonly container: Locator;
  readonly title: Locator;
  readonly closeButton: Locator;
  readonly confirmButton: Locator;
  readonly cancelButton: Locator;
  readonly content: Locator;

  constructor(page: Page, containerSelector: string = '.modal') {
    this.container = page.locator(containerSelector);
    this.title = this.container.locator('.modal-title');
    this.content = this.container.locator('.modal-body');
    this.closeButton = this.container.getByRole('button', { name: 'Close' });
    this.confirmButton = this.container.getByRole('button', { name: 'Confirm' });
    this.cancelButton = this.container.getByRole('button', { name: 'Cancel' });
  }

  async waitForModal(): Promise<void> {
    await this.container.waitFor({ state: 'visible' });
  }

  async getTitle(): Promise<string> {
    return (await this.title.textContent()) || '';
  }

  async getContent(): Promise<string> {
    return (await this.content.textContent()) || '';
  }

  async confirm(): Promise<void> {
    await this.confirmButton.click();
    await this.container.waitFor({ state: 'hidden' });
  }

  async cancel(): Promise<void> {
    await this.cancelButton.click();
    await this.container.waitFor({ state: 'hidden' });
  }

  async close(): Promise<void> {
    await this.closeButton.click();
    await this.container.waitFor({ state: 'hidden' });
  }
}

// components/data-table.component.ts
export class DataTableComponent {
  readonly container: Locator;
  readonly headers: Locator;
  readonly rows: Locator;
  readonly paginationContainer: Locator;

  constructor(page: Page, tableSelector: string = '[data-testid="data-table"]') {
    this.container = page.locator(tableSelector);
    this.headers = this.container.locator('thead th');
    this.rows = this.container.locator('tbody tr');
    this.paginationContainer = page.locator('.pagination');
  }

  async getRowCount(): Promise<number> {
    return await this.rows.count();
  }

  async getRowByIndex(index: number): Promise<Locator> {
    return this.rows.nth(index);
  }

  async getRowByText(text: string): Promise<Locator> {
    return this.rows.filter({ hasText: text });
  }

  async getCellValue(rowIndex: number, columnIndex: number): Promise<string> {
    const row = this.rows.nth(rowIndex);
    const cell = row.locator('td').nth(columnIndex);
    return (await cell.textContent()) || '';
  }

  async sortByColumn(columnName: string): Promise<void> {
    const header = this.headers.filter({ hasText: columnName });
    await header.click();
  }
}

// Usage in page objects
export class UserManagementPage {
  readonly deleteModal: ModalComponent;
  readonly userTable: DataTableComponent;

  constructor(page: Page) {
    this.page = page;
    this.deleteModal = new ModalComponent(page, '#delete-confirmation-modal');
    this.userTable = new DataTableComponent(page, '#users-table');
  }

  async deleteUser(userName: string): Promise<void> {
    const userRow = await this.userTable.getRowByText(userName);
    const deleteButton = userRow.getByRole('button', { name: 'Delete' });

    await deleteButton.click();
    await this.deleteModal.waitForModal();
    await this.deleteModal.confirm();
  }

  async getUserCount(): Promise<number> {
    return await this.userTable.getRowCount();
  }
}
```

## Constants and Data Management

Centralize configuration and constants for maintainability.

```typescript
// constants/page.constants.ts
export const PAGE_CONSTANTS = {
  URLS: {
    LOGIN: '/login',
    DASHBOARD: '/dashboard',
    SEARCH: '/search',
    MONITOR: '/monitor',
    TOOLS: '/tools',
  },
  SELECTORS: {
    LOADING: '.loading-spinner',
    ERROR_MESSAGE: '.error-message',
    SUCCESS_MESSAGE: '.success-message',
    MODAL_BACKDROP: '.modal-backdrop',
  },
  TIMEOUTS: {
    SHORT: 5000,
    MEDIUM: 10000,
    LONG: 30000,
  },
} as const;

// constants/test.data.ts
export const TEST_DATA = {
  USERS: {
    ADMIN: {
      username: 'admin@example.com',
      password: 'AdminPass123!',
      role: 'admin',
    },
    REGULAR_USER: {
      username: 'user@example.com',
      password: 'UserPass123!',
      role: 'user',
    },
  },
  SEARCH_TERMS: {
    INDIVIDUAL: 'John Doe',
    COMPANY: 'Acme Corporation',
    COMPLEX: 'Financial Crime Investigation',
  },
} as const;

// Using constants in page objects
export class LoginPage {
  constructor(page: Page) {
    this.page = page;
    this.errorMessage = page.locator(PAGE_CONSTANTS.SELECTORS.ERROR_MESSAGE);
  }

  async goto(): Promise<void> {
    await this.page.goto(PAGE_CONSTANTS.URLS.LOGIN);
    await this.page.waitForLoadState('domcontentloaded');
  }

  async loginAsAdmin(): Promise<void> {
    const admin = TEST_DATA.USERS.ADMIN;
    await this.login(admin.username, admin.password);
  }
}
```

## Method Chaining

Enable fluent interfaces for complex workflows.

```typescript
// Fluent interface implementation
export class SearchPage {
  async search(query: string): Promise<SearchPage> {
    await this.searchField.fill(query);
    await this.searchButton.click();
    await this.page.waitForLoadState('networkidle');
    return this;
  }

  async applyFilter(filterType: string, value: string): Promise<SearchPage> {
    const filter = this.page.getByTestId(`filter-${filterType}`);
    await filter.selectOption(value);
    await this.page.waitForLoadState('networkidle');
    return this;
  }

  async sortBy(option: string): Promise<SearchPage> {
    await this.sortDropdown.selectOption(option);
    await this.page.waitForLoadState('networkidle');
    return this;
  }

  async setDateRange(from: Date, to: Date): Promise<SearchPage> {
    await this.dateFromField.fill(from.toISOString().split('T')[0]);
    await this.dateToField.fill(to.toISOString().split('T')[0]);
    return this;
  }

  // Terminal methods that don't return SearchPage
  async getResultCount(): Promise<number> {
    const countText = await this.resultCountElement.textContent();
    const match = countText?.match(/(\\d+) results/);
    return match ? parseInt(match[1], 10) : 0;
  }

  async getResults(): Promise<string[]> {
    return await this.resultsList.allTextContents();
  }
}

// Usage with method chaining
test('advanced search with chaining', async ({ page }) => {
  const searchPage = new SearchPage(page);

  const resultCount = await searchPage
    .goto()
    .then((page) => page.search('John Doe'))
    .then((page) => page.applyFilter('location', 'New York'))
    .then((page) => page.applyFilter('category', 'individual'))
    .then((page) => page.sortBy('relevance'))
    .then((page) => page.getResultCount());

  expect(resultCount).toBeGreaterThan(0);
});
```

## Error Handling Strategies

Robust error handling for production-ready tests.

```typescript
// pages/robust.page.ts
export class RobustPage {
  readonly page: Page;

  constructor(page: Page) {
    this.page = page;
  }

  // Safe click with error handling
  async safeClick(locator: Locator, options?: { timeout?: number; retries?: number }): Promise<void> {
    const { timeout = 5000, retries = 3 } = options || {};

    for (let attempt = 1; attempt <= retries; attempt++) {
      try {
        await locator.waitFor({ state: 'visible', timeout });
        await locator.click({ timeout });
        return; // Success
      } catch (error) {
        if (attempt === retries) {
          throw new Error(
            `Failed to click element after ${retries} attempts: ${
              error instanceof Error ? error.message : 'Unknown error'
            }`,
          );
        }
        console.warn(`Click attempt ${attempt} failed, retrying...`);
        await this.page.waitForTimeout(1000);
      }
    }
  }

  // Safe text retrieval
  async safeGetText(locator: Locator, defaultValue: string = ''): Promise<string> {
    try {
      await locator.waitFor({ state: 'visible', timeout: 5000 });
      return (await locator.textContent()) || defaultValue;
    } catch (error) {
      console.warn(`Could not get text content: ${error instanceof Error ? error.message : 'Unknown error'}`);
      return defaultValue;
    }
  }

  // Conditional actions
  async clickIfExists(locator: Locator): Promise<boolean> {
    try {
      await locator.waitFor({ state: 'visible', timeout: 2000 });
      await locator.click();
      return true;
    } catch {
      return false; // Element doesn't exist or isn't clickable
    }
  }

  // Retry pattern for flaky operations
  async retryOperation<T>(operation: () => Promise<T>, maxRetries: number = 3, delayMs: number = 1000): Promise<T> {
    for (let attempt = 1; attempt <= maxRetries; attempt++) {
      try {
        return await operation();
      } catch (error) {
        if (attempt === maxRetries) {
          throw error;
        }
        console.warn(`Operation attempt ${attempt} failed, retrying in ${delayMs}ms...`);
        await this.page.waitForTimeout(delayMs);
      }
    }
    throw new Error('This should never be reached');
  }

  // Network error handling
  async performActionWithNetworkResilience<T>(action: () => Promise<T>): Promise<T> {
    return await this.retryOperation(async () => {
      try {
        return await action();
      } catch (error) {
        if (error instanceof Error && error.message.includes('net::')) {
          throw new Error(`Network error occurred: ${error.message}`);
        }
        throw error;
      }
    });
  }
}
```

## Accessibility Integration

Integrate accessibility testing into page objects.

```typescript
// pages/shared/aria.snapshots.ts
export const ariaSnapshots = {
  loginForm: `
    - textbox "Username"
    - textbox "Password" [type=password]
    - button "Sign In"
    - link "Forgot Password?"
  `,
  dashboardNavigation: `
    - link "Search"
    - link "Monitor"
    - link "Reports"
    - link "Tools"
    - link "Help"
  `,
  searchForm: `
    - textbox "Search term"
    - button "Search"
    - combobox "Category"
    - button "Advanced Filters"
  `,
} as const;

// Accessibility-aware page object
export class AccessibleLoginPage {
  readonly page: Page;

  constructor(page: Page) {
    this.page = page;
  }

  get expectedAriaStructure() {
    return ariaSnapshots.loginForm;
  }

  async validateAccessibility(): Promise<any> {
    // Return accessibility tree for test validation
    return await this.page.accessibility.snapshot();
  }

  async validateFormLabels(): Promise<boolean> {
    // Check that form elements have proper labels
    const usernameLabel = await this.page.getByLabel('Username').isVisible();
    const passwordLabel = await this.page.getByLabel('Password').isVisible();
    return usernameLabel && passwordLabel;
  }

  async checkKeyboardNavigation(): Promise<boolean> {
    // Test tab navigation through form
    await this.page.keyboard.press('Tab');
    const activeElement = await this.page.evaluate(() => document.activeElement?.tagName);
    return activeElement === 'INPUT';
  }
}
```

## Testing Strategies

### Page Object Health Monitoring

```typescript
// utils/page.health.ts
export class PageHealthChecker {
  static async validateLocators(page: Page, pageObject: any): Promise<string[]> {
    const issues: string[] = [];
    const locatorProperties = Object.getOwnPropertyNames(pageObject).filter(
      (prop) => pageObject[prop]?.constructor?.name === 'Locator',
    );

    for (const prop of locatorProperties) {
      try {
        const locator = pageObject[prop] as Locator;
        await locator.waitFor({ state: 'attached', timeout: 1000 });
      } catch (error) {
        issues.push(`Locator '${prop}' failed validation: ${error instanceof Error ? error.message : 'Unknown error'}`);
      }
    }

    return issues;
  }

  static async generateLocatorReport(page: Page, pageObject: any): Promise<object> {
    const report = {
      pageUrl: page.url(),
      timestamp: new Date().toISOString(),
      locators: {} as Record<string, any>,
    };

    const locatorProperties = Object.getOwnPropertyNames(pageObject).filter(
      (prop) => pageObject[prop]?.constructor?.name === 'Locator',
    );

    for (const prop of locatorProperties) {
      try {
        const locator = pageObject[prop] as Locator;
        const isVisible = await locator.isVisible().catch(() => false);
        const count = await locator.count().catch(() => 0);

        report.locators[prop] = {
          visible: isVisible,
          count,
          status: count > 0 ? 'found' : 'not-found',
        };
      } catch (error) {
        report.locators[prop] = {
          status: 'error',
          error: error instanceof Error ? error.message : 'Unknown error',
        };
      }
    }

    return report;
  }
}
```

## Maintenance Patterns

### Deprecation and Migration

```typescript
export class UserPage {
  // New preferred method
  async updateUserProfile(data: UserProfile): Promise<void> {
    await this.fillUserForm(data);
    await this.saveChanges();
  }

  // Deprecated method with migration guidance
  /**
   * @deprecated Use updateUserProfile() instead. Will be removed in v2.0
   * @see updateUserProfile
   */
  async editProfile(data: UserProfile): Promise<void> {
    console.warn('editProfile() is deprecated. Use updateUserProfile() instead.');
    return this.updateUserProfile(data);
  }

  // Version-specific implementations
  async saveChanges(options: { waitForConfirmation?: boolean } = {}): Promise<void> {
    await this.saveButton.click();

    if (options.waitForConfirmation !== false) {
      await this.confirmationMessage.waitFor({ state: 'visible' });
    }
  }
}
```

### Locator Evolution Tracking

```typescript
// Track locator changes over time
export class SearchPageLocators {
  // Current implementation
  static current(page: Page) {
    return {
      searchField: page.getByTestId('search-input'),
      searchButton: page.getByRole('button', { name: 'Search' }),
      filters: page.getByTestId('search-filters'),
    };
  }

  // Legacy locators for backward compatibility
  static legacy(page: Page) {
    return {
      searchField: page.locator('#search_single_term'),
      searchButton: page.locator('input[type="submit"]'),
      filters: page.locator('.search-filters'),
    };
  }

  // Auto-fallback implementation
  static adaptive(page: Page) {
    const current = this.current(page);
    const legacy = this.legacy(page);

    return {
      async searchField() {
        try {
          await current.searchField.waitFor({ state: 'attached', timeout: 1000 });
          return current.searchField;
        } catch {
          return legacy.searchField;
        }
      },
    };
  }
}
```

These advanced patterns provide the foundation for building maintainable, scalable test automation suites that can evolve with your application while maintaining reliability and clarity.
