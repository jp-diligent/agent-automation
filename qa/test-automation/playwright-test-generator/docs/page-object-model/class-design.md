# Class Design Principles

> **Purpose**: Learn how to structure page object classes with proper locator organization, TypeScript integration, and maintainable design patterns.

## Core Structure Template

```typescript
import { type Locator, type Page } from '@playwright/test';

export class ExamplePage {
  readonly page: Page;

  // 1. Group locators by functional purpose
  // Navigation elements
  readonly logo: Locator;
  readonly homeLink: Locator;
  readonly userMenu: Locator;

  // Form elements
  readonly usernameField: Locator;
  readonly passwordField: Locator;
  readonly submitButton: Locator;

  // Status/feedback elements
  readonly loadingIndicator: Locator;
  readonly errorMessage: Locator;
  readonly successMessage: Locator;

  constructor(page: Page) {
    this.page = page;

    // 2. Initialize locators with semantic selectors first
    this.logo = page.getByRole('link', { name: 'Company Logo' });
    this.homeLink = page.getByRole('link', { name: 'Home' });
    this.usernameField = page.getByPlaceholder('Username');
    this.submitButton = page.getByRole('button', { name: 'Submit' });

    // 3. Use CSS selectors for unique IDs when needed
    this.loadingIndicator = page.locator('#loading-spinner');
    this.errorMessage = page.locator('#error-message');
  }

  // 4. Group methods by purpose
  // Navigation methods
  async goto(): Promise<void> {
    await this.page.goto('/example');
    await this.page.waitForLoadState('domcontentloaded');
  }

  // Action methods
  async fillCredentials(username: string, password: string): Promise<void> {
    await this.usernameField.fill(username);
    await this.passwordField.fill(password);
  }

  // Utility methods - return data for tests to assert
  async getErrorMessage(): Promise<string | null> {
    return await this.errorMessage.textContent();
  }
}
```

## Locator Organization Strategies

### 1. Functional Grouping (Recommended)

Group locators by their purpose rather than their type:

```typescript
export class DashboardPage {
  readonly page: Page;

  // Navigation cluster
  readonly logo: Locator;
  readonly searchLink: Locator;
  readonly monitorLink: Locator;
  readonly profileMenu: Locator;
  readonly logoutLink: Locator;

  // Search functionality cluster
  readonly searchField: Locator;
  readonly searchButton: Locator;
  readonly searchFilters: Locator;
  readonly searchResults: Locator;

  // Content panels cluster
  readonly latestSearchesPanel: Locator;
  readonly savedReportsPanel: Locator;
  readonly monitoringPanel: Locator;
  readonly watchlistStatusPanel: Locator;

  // Status and feedback cluster
  readonly loadingIndicator: Locator;
  readonly errorNotification: Locator;
  readonly successNotification: Locator;
}
```

### 2. Complex Page with Sub-components

For pages with many elements, consider sub-grouping:

```typescript
export class SearchResultsPage {
  readonly page: Page;

  // Main search interface
  readonly searchBar: {
    readonly field: Locator;
    readonly button: Locator;
    readonly clearButton: Locator;
  };

  // Filters section
  readonly filters: {
    readonly dateRange: Locator;
    readonly category: Locator;
    readonly status: Locator;
    readonly applyButton: Locator;
    readonly clearButton: Locator;
  };

  // Results display
  readonly results: {
    readonly container: Locator;
    readonly items: Locator;
    readonly pagination: Locator;
    readonly countDisplay: Locator;
  };

  constructor(page: Page) {
    this.page = page;

    // Initialize grouped locators
    this.searchBar = {
      field: page.getByPlaceholder('Search...'),
      button: page.getByRole('button', { name: 'Search' }),
      clearButton: page.getByRole('button', { name: 'Clear' }),
    };

    this.filters = {
      dateRange: page.getByLabel('Date Range'),
      category: page.getByLabel('Category'),
      status: page.getByLabel('Status'),
      applyButton: page.getByRole('button', { name: 'Apply Filters' }),
      clearButton: page.getByRole('button', { name: 'Clear Filters' }),
    };

    this.results = {
      container: page.locator('.results-container'),
      items: page.locator('.result-item'),
      pagination: page.locator('.pagination'),
      countDisplay: page.locator('.result-count'),
    };
  }

  // Methods can reference grouped locators
  async performSearch(query: string): Promise<void> {
    await this.searchBar.field.fill(query);
    await this.searchBar.button.click();
  }

  async applyFilters(options: { category?: string; status?: string }): Promise<void> {
    if (options.category) {
      await this.filters.category.selectOption(options.category);
    }
    if (options.status) {
      await this.filters.status.selectOption(options.status);
    }
    await this.filters.applyButton.click();
  }
}
```

## TypeScript Integration

### 1. Interface Definitions

Define interfaces for complex data structures:

```typescript
// types/user.types.ts
export interface UserCredentials {
  username: string;
  password: string;
}

export interface UserProfile {
  firstName: string;
  lastName: string;
  email: string;
  role: 'admin' | 'user' | 'viewer';
}

export interface SearchFilters {
  dateRange?: {
    from: Date;
    to: Date;
  };
  category?: string;
  status?: 'active' | 'inactive' | 'pending';
  tags?: string[];
}
```

### 2. Typed Page Objects

Use interfaces in your page objects:

```typescript
import { UserCredentials, UserProfile, SearchFilters } from '../types/user.types';

export class UserManagementPage {
  readonly page: Page;

  // Explicit typing for method parameters
  async createUser(profile: UserProfile, credentials: UserCredentials): Promise<string> {
    await this.fillUserForm(profile);
    await this.setCredentials(credentials);
    await this.submitForm();

    // Return user ID for test validation
    return await this.getUserId();
  }

  async searchUsers(filters: SearchFilters): Promise<string[]> {
    await this.applySearchFilters(filters);
    return await this.getUserListIds();
  }

  // Explicit return types for all methods
  async getUserCount(): Promise<number> {
    const countText = await this.userCountElement.textContent();
    return parseInt(countText?.match(/(\d+)/)?.[1] || '0', 10);
  }

  async isUserActive(userId: string): Promise<boolean> {
    const userRow = this.page.locator(`[data-user-id="${userId}"]`);
    const statusElement = userRow.locator('.status');
    const status = await statusElement.textContent();
    return status?.toLowerCase() === 'active';
  }
}
```

### 3. Generic Utilities

Create reusable generic types:

```typescript
// types/common.types.ts
export type ActionResult<T = void> = Promise<T>;
export type ValidationResult = { isValid: boolean; message?: string };

// utils/form.utils.ts
export class FormUtilities {
  static async fillForm<T extends Record<string, any>>(
    page: Page,
    formData: T,
    fieldMapping: Record<keyof T, string>,
  ): Promise<void> {
    for (const [key, selector] of Object.entries(fieldMapping)) {
      const value = formData[key];
      if (value !== undefined) {
        await page.locator(selector).fill(String(value));
      }
    }
  }
}
```

## Method Organization Patterns

### 1. Method Categories

Organize methods into clear categories:

```typescript
export class LoginPage {
  readonly page: Page;
  // ... locators ...

  // === NAVIGATION METHODS ===
  async goto(): Promise<void> {
    await this.page.goto('/login');
    await this.page.waitForLoadState('domcontentloaded');
  }

  async gotoWithRedirect(returnUrl: string): Promise<void> {
    await this.page.goto(`/login?return=${encodeURIComponent(returnUrl)}`);
    await this.page.waitForLoadState('domcontentloaded');
  }

  // === ACTION METHODS ===
  async login(username: string, password: string): Promise<void> {
    await this.fillUsername(username);
    await this.fillPassword(password);
    await this.clickSubmitButton();
  }

  async fillUsername(username: string): Promise<void> {
    await this.usernameField.fill(username);
  }

  async fillPassword(password: string): Promise<void> {
    await this.passwordField.fill(password);
  }

  async clickSubmitButton(): Promise<void> {
    await this.submitButton.click();
  }

  async clickForgotPasswordLink(): Promise<void> {
    await this.forgotPasswordLink.click();
  }

  // === UTILITY METHODS ===
  async getErrorMessage(): Promise<string | null> {
    return await this.errorMessage.textContent();
  }

  async isSubmitButtonEnabled(): Promise<boolean> {
    return await this.submitButton.isEnabled();
  }

  async waitForLoginForm(): Promise<void> {
    await this.usernameField.waitFor({ state: 'visible' });
  }

  // === VALIDATION METHODS ===
  async validateLoginForm(): Promise<ValidationResult> {
    const isUsernameVisible = await this.usernameField.isVisible();
    const isPasswordVisible = await this.passwordField.isVisible();
    const isSubmitVisible = await this.submitButton.isVisible();

    const isValid = isUsernameVisible && isPasswordVisible && isSubmitVisible;
    return {
      isValid,
      message: isValid ? undefined : 'Login form is not completely visible',
    };
  }
}
```

### 2. Business Logic Encapsulation

Encapsulate complex workflows in single methods:

```typescript
export class MonitorListPage {
  async createMonitorList(
    name: string,
    options: {
      negativeNews?: boolean;
      financialNews?: boolean;
      watchlist?: boolean;
      peps?: boolean;
    } = {},
  ): Promise<string> {
    // Generate unique name
    const uniqueName = `${name}_${Date.now()}`;

    await this.fillListName(uniqueName);

    // Configure options
    if (options.negativeNews) {
      await this.negativeNewsCheckbox.check();
    }
    if (options.financialNews) {
      await this.financialNewsCheckbox.check();
    }
    if (options.watchlist) {
      await this.watchlistCheckbox.check();
    }
    if (options.peps) {
      await this.pepsCheckbox.check();
    }

    await this.clickCreateButton();
    await this.page.waitForLoadState('networkidle');

    // Return created list ID for test validation
    return await this.getCreatedListId();
  }

  // Usage in tests becomes simple:
  // await monitorListPage.createMonitorList('Test List', { negativeNews: true, watchlist: true });
}
```

## Readonly Properties and Immutability

### Why Readonly Matters

```typescript
export class SearchPage {
  // ✅ Good - Readonly ensures locators can't be reassigned
  readonly searchField: Locator;
  readonly searchButton: Locator;

  // ❌ Bad - Could be accidentally reassigned
  searchField: Locator;
  searchButton: Locator;

  constructor(page: Page) {
    this.page = page;
    this.searchField = page.getByPlaceholder('Search...');

    // This would cause TypeScript error with readonly:
    // this.searchField = page.getByRole('textbox'); // Error!
  }
}
```

### Page Reference Management

```typescript
export class BasePage {
  // Make page readonly to prevent accidental reassignment
  readonly page: Page;

  constructor(page: Page) {
    this.page = page;
  }

  // Provide controlled access to page instance
  protected getPage(): Page {
    return this.page;
  }

  // Common functionality available to all page objects
  async getPageTitle(): Promise<string> {
    return await this.page.title();
  }

  async getCurrentUrl(): Promise<string> {
    return this.page.url();
  }

  async refreshPage(): Promise<void> {
    await this.page.reload();
    await this.page.waitForLoadState('domcontentloaded');
  }
}
```

## Error Handling in Page Objects

```typescript
export class RobustPage {
  readonly page: Page;

  // Wrapper methods with error handling
  async safeClick(locator: Locator, description: string): Promise<void> {
    try {
      await locator.waitFor({ state: 'visible' });
      await locator.click();
    } catch (error) {
      throw new Error(`Failed to click ${description}: ${error instanceof Error ? error.message : 'Unknown error'}`);
    }
  }

  async safeGetText(locator: Locator, defaultValue: string = ''): Promise<string> {
    try {
      await locator.waitFor({ state: 'visible' });
      return (await locator.textContent()) || defaultValue;
    } catch (error) {
      console.warn(`Could not get text content: ${error instanceof Error ? error.message : 'Unknown error'}`);
      return defaultValue;
    }
  }

  // Retry mechanisms for flaky operations
  async retryOperation<T>(
    operation: () => Promise<T>,
    maxRetries: number = 3,
    description: string = 'operation',
  ): Promise<T> {
    for (let attempt = 1; attempt <= maxRetries; attempt++) {
      try {
        return await operation();
      } catch (error) {
        if (attempt === maxRetries) {
          throw new Error(
            `${description} failed after ${maxRetries} attempts: ${
              error instanceof Error ? error.message : 'Unknown error'
            }`,
          );
        }
        console.warn(`${description} attempt ${attempt} failed, retrying...`);
      }
    }
    throw new Error(`Retry logic failed for ${description}`);
  }
}
```

## Design Principles Summary

### ✅ Do's

1. **Single Responsibility**: Each page object represents one page or component
2. **Readonly Properties**: Make all locators and page references readonly
3. **Functional Grouping**: Group locators by purpose, not by type
4. **Explicit Typing**: Use TypeScript interfaces and explicit return types
5. **Clear Method Names**: Use consistent naming conventions
6. **Business Logic**: Encapsulate complex workflows in page objects
7. **Error Handling**: Include proper error handling for robustness

### ❌ Don'ts

1. **No Assertions**: Don't include test assertions in page objects
2. **No Hardcoded Data**: Don't store test data in page objects
3. **No Deep Inheritance**: Avoid complex inheritance chains
4. **No Side Effects**: Methods should be predictable and focused
5. **No Tight Coupling**: Page objects should be independently testable

---

_Well-designed page object classes form the foundation of maintainable test automation. Focus on clarity, type safety, and encapsulation of business logic._
