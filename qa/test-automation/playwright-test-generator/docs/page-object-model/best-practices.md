# Best Practices

**Description**: Comprehensive collection of proven best practices for Page Object Model implementation, covering design principles, maintenance strategies, performance optimization, and long-term sustainability of test automation code.

## Table of Contents

- [Design Principles](#design-principles)
- [Code Organization](#code-organization)
- [Performance Optimization](#performance-optimization)
- [Maintenance Strategies](#maintenance-strategies)
- [Team Collaboration](#team-collaboration)
- [Testing Guidelines](#testing-guidelines)
- [Documentation Standards](#documentation-standards)
- [Quick Checklist](#quick-checklist)

## Design Principles

### Single Responsibility Principle

Each page object should represent one logical page or component:

```typescript
// ✅ Good - Focused responsibility
export class LoginPage {
  // Only handles login-related functionality
  async goto(): Promise<void> {}
  async login(username: string, password: string): Promise<void> {}
  async getErrorMessage(): Promise<string | null> {}
}

export class DashboardPage {
  // Only handles dashboard-related functionality
  async goto(): Promise<void> {}
  async getWelcomeMessage(): Promise<string> {}
  async navigateToSearch(): Promise<void> {}
}

// ❌ Bad - Mixed responsibilities
export class ApplicationPage {
  // Too many responsibilities in one class
  async login(): Promise<void> {}
  async performSearch(): Promise<void> {}
  async createReport(): Promise<void> {}
  async manageUsers(): Promise<void> {}
}
```

### Immutable Locators

Use readonly properties to prevent accidental modification:

```typescript
// ✅ Good - Immutable locators
export class SearchPage {
  readonly page: Page;
  readonly searchField: Locator;
  readonly searchButton: Locator;
  readonly resultsList: Locator;

  constructor(page: Page) {
    this.page = page;
    this.searchField = page.getByPlaceholder('Enter search term');
    this.searchButton = page.getByRole('button', { name: 'Search' });
    this.resultsList = page.getByTestId('search-results');
  }
}
```

### Clear Method Naming

Use consistent, descriptive method names:

```typescript
// ✅ Good - Clear, consistent naming
export class UserPage {
  // Navigation methods
  async goto(): Promise<void> {}
  async gotoUserProfile(userId: string): Promise<void> {}

  // Action methods
  async clickSaveButton(): Promise<void> {}
  async clickCancelButton(): Promise<void> {}

  // Form methods
  async fillUsername(username: string): Promise<void> {}
  async fillEmail(email: string): Promise<void> {}

  // State methods
  async isFormValid(): Promise<boolean> {}
  async isLoading(): Promise<boolean> {}

  // Data retrieval methods
  async getUserName(): Promise<string> {}
  async getErrorCount(): Promise<number> {}
}
```

## Code Organization

### Directory Structure

Organize page objects by business domain:

```
pages/
├── auth/
│   ├── login.page.ts
│   ├── register.page.ts
│   └── forgot-password.page.ts
├── dashboard/
│   ├── dashboard.page.ts
│   ├── reports.page.ts
│   └── analytics.page.ts
├── search/
│   ├── search.page.ts
│   ├── results.page.ts
│   └── advanced-search.page.ts
├── admin/
│   ├── user-management.page.ts
│   ├── system-settings.page.ts
│   └── audit-logs.page.ts
└── shared/
    ├── navigation.component.ts
    ├── modal.component.ts
    └── data-table.component.ts
```

### File Naming Conventions

Use consistent naming patterns:

```
// Page objects
login.page.ts
user-management.page.ts
search-results.page.ts

// Components
navigation.component.ts
data-table.component.ts
modal.component.ts

// Utilities
form.utils.ts
date.utils.ts
api.helper.ts

// Constants
urls.constants.ts
test-data.constants.ts
selectors.constants.ts
```

### Import Organization

Keep imports organized and consistent:

```typescript
// External libraries first
import { type Page, type Locator } from '@playwright/test';

// Internal utilities
import { DateUtils } from '../utils/date.utils';
import { FormUtils } from '../utils/form.utils';

// Constants
import { PAGE_URLS } from '../constants/urls.constants';

// Types/interfaces
import type { UserProfile, SearchFilters } from '../types/common.types';

export class UserProfilePage {
  // Implementation
}
```

## Performance Optimization

### Lazy Initialization

Initialize expensive operations only when needed:

```typescript
export class ComplexDashboardPage {
  readonly page: Page;
  private _expensiveChart?: Locator;
  private _cachedUserData?: UserData;

  // Lazy locator initialization
  get expensiveChart(): Locator {
    if (!this._expensiveChart) {
      this._expensiveChart = this.page
        .locator('.chart-container')
        .filter({ has: this.page.locator('[data-chart-type="complex"]') });
    }
    return this._expensiveChart;
  }

  // Cached data retrieval
  async getUserData(): Promise<UserData> {
    if (!this._cachedUserData) {
      this._cachedUserData = await this.fetchUserData();
    }
    return this._cachedUserData;
  }

  // Cache invalidation
  invalidateCache(): void {
    this._cachedUserData = undefined;
  }
}
```

### Efficient Waiting Strategies

Use appropriate wait strategies for better performance:

```typescript
export class OptimizedPage {
  // Use built-in auto-waiting
  async performQuickAction(): Promise<void> {
    await this.button.click(); // Auto-waits for actionability
  }

  // Use specific wait states
  async navigateAndWait(): Promise<void> {
    await this.navigationLink.click();
    await this.page.waitForLoadState('domcontentloaded'); // Faster than networkidle
  }

  // Use conditional waits
  async handleOptionalElement(): Promise<boolean> {
    try {
      await this.optionalElement.waitFor({ state: 'visible', timeout: 2000 });
      return true;
    } catch {
      return false; // Element doesn't exist
    }
  }
}
```

### Minimize Locator Creation

Reuse locators when possible:

```typescript
// ✅ Good - Reuse locators
export class EfficientPage {
  readonly page: Page;
  readonly userCards: Locator;

  constructor(page: Page) {
    this.page = page;
    this.userCards = page.locator('[data-testid="user-card"]');
  }

  async getUserCardByName(name: string): Promise<Locator> {
    return this.userCards.filter({ hasText: name });
  }

  async getUserCardCount(): Promise<number> {
    return await this.userCards.count();
  }
}

// ❌ Bad - Creating new locators repeatedly
export class InefficientPage {
  async getUserCardByName(name: string): Promise<Locator> {
    return this.page.locator('[data-testid="user-card"]').filter({ hasText: name });
  }

  async getUserCardCount(): Promise<number> {
    return await this.page.locator('[data-testid="user-card"]').count();
  }
}
```

## Maintenance Strategies

### Version Control and Evolution

Plan for changes and maintain backward compatibility:

```typescript
export class EvolvingPage {
  readonly page: Page;

  constructor(page: Page) {
    this.page = page;
  }

  // New preferred method
  async submitFormV2(formData: FormData): Promise<void> {
    await this.fillForm(formData);
    await this.submitButton.click();
    await this.waitForSubmissionComplete();
  }

  // Deprecated but maintained for backward compatibility
  /** @deprecated Use submitFormV2() instead. Will be removed in v3.0 */
  async submitForm(formData: FormData): Promise<void> {
    console.warn('submitForm() is deprecated. Use submitFormV2() instead.');
    return this.submitFormV2(formData);
  }

  // Migration helper
  static migrateToV2(oldPageObject: any): EvolvingPage {
    // Helper to migrate from old version
    return new EvolvingPage(oldPageObject.page);
  }
}
```

### Health Monitoring

Implement page object health checks:

```typescript
// utils/page-health.checker.ts
export class PageHealthChecker {
  static async validatePage(
    page: Page,
    pageObject: any,
  ): Promise<{
    isHealthy: boolean;
    issues: string[];
    report: Record<string, any>;
  }> {
    const issues: string[] = [];
    const report: Record<string, any> = {};

    // Check if page is loaded
    try {
      await page.waitForLoadState('domcontentloaded', { timeout: 5000 });
      report.pageLoaded = true;
    } catch {
      issues.push('Page failed to load');
      report.pageLoaded = false;
    }

    // Validate critical locators
    const locatorProperties = Object.getOwnPropertyNames(pageObject).filter(
      (prop) => pageObject[prop]?.constructor?.name === 'Locator',
    );

    for (const prop of locatorProperties) {
      try {
        const locator = pageObject[prop] as Locator;
        const count = await locator.count();
        report[prop] = { count, status: count > 0 ? 'found' : 'missing' };

        if (count === 0) {
          issues.push(`Critical locator '${prop}' not found`);
        }
      } catch (error) {
        issues.push(`Locator '${prop}' validation failed: ${error}`);
        report[prop] = { status: 'error', error: String(error) };
      }
    }

    return {
      isHealthy: issues.length === 0,
      issues,
      report,
    };
  }
}

// Usage in tests
test.beforeEach(async ({ page }) => {
  const loginPage = new LoginPage(page);
  await loginPage.goto();

  const health = await PageHealthChecker.validatePage(page, loginPage);
  if (!health.isHealthy) {
    console.warn('Page health issues detected:', health.issues);
  }
});
```

### Refactoring Guidelines

Systematic approach to refactoring page objects:

```typescript
// Step 1: Create new implementation alongside old
export class SearchPageV2 {
  // New implementation
}

// Step 2: Add migration path
export class SearchPage {
  // Old implementation with migration warnings
  /** @deprecated Use SearchPageV2 instead */
  async oldMethod(): Promise<void> {
    console.warn('This method will be removed. Use SearchPageV2.newMethod()');
  }
}

// Step 3: Update tests gradually
test('search functionality - migrated', async ({ page }) => {
  const searchPage = new SearchPageV2(page);
  // Use new implementation
});

// Step 4: Remove old implementation after migration period
```

## Team Collaboration

### Code Review Guidelines

Establish clear review criteria:

```typescript
// Review checklist for page objects:
// ✅ Follows naming conventions
// ✅ Uses appropriate locator strategies
// ✅ Has proper TypeScript typing
// ✅ Includes error handling
// ✅ Methods have single responsibility
// ✅ No hardcoded values
// ✅ Proper documentation

export class ReviewableSearchPage {
  readonly page: Page;
  readonly searchField: Locator; // ✅ Readonly property

  constructor(page: Page) {
    this.page = page;
    // ✅ Semantic locator
    this.searchField = page.getByLabel('Search term');
  }

  // ✅ Descriptive name, proper typing, error handling
  async performSearch(query: string): Promise<number> {
    if (!query.trim()) {
      throw new Error('Search query cannot be empty');
    }

    await this.searchField.fill(query);
    await this.searchButton.click();

    return await this.getResultCount();
  }
}
```

### Documentation Standards

Document complex logic and business rules:

````typescript
export class DocumentedOrderPage {
  /**
   * Processes an order through the complete workflow
   *
   * @param orderData - Order information including items and customer details
   * @param options - Processing options
   * @param options.expedited - Whether to use expedited shipping
   * @param options.validateInventory - Whether to check inventory before processing
   * @returns Promise resolving to the order confirmation number
   *
   * @example
   * ```typescript
   * const confirmationNumber = await orderPage.processOrder({
   *   customerId: '12345',
   *   items: [{ productId: 'ABC', quantity: 2 }]
   * }, { expedited: true });
   * ```
   */
  async processOrder(
    orderData: OrderData,
    options: {
      expedited?: boolean;
      validateInventory?: boolean;
    } = {},
  ): Promise<string> {
    // Implementation with clear steps
    if (options.validateInventory) {
      await this.validateInventoryLevels(orderData.items);
    }

    await this.fillOrderForm(orderData);

    if (options.expedited) {
      await this.selectExpeditedShipping();
    }

    await this.submitOrder();
    return await this.getConfirmationNumber();
  }
}
````

## Testing Guidelines

### Integration with Test Framework

Structure page objects for optimal test integration:

```typescript
// tests/order-processing.spec.ts
import { test, expect } from '@playwright/test';
import { OrderPage } from '../pages/order.page';
import { TEST_DATA } from '../constants/test-data';

test.describe('Order Processing', () => {
  let orderPage: OrderPage;

  test.beforeEach(async ({ page }) => {
    orderPage = new OrderPage(page);
    await orderPage.goto();
  });

  test('should process standard order successfully', async () => {
    const confirmationNumber = await orderPage.processOrder(TEST_DATA.ORDERS.STANDARD_ORDER);

    expect(confirmationNumber).toMatch(/^ORD-\d{8}$/);
  });

  test('should handle expedited shipping', async () => {
    const confirmationNumber = await orderPage.processOrder(TEST_DATA.ORDERS.STANDARD_ORDER, { expedited: true });

    const shippingMethod = await orderPage.getSelectedShippingMethod();
    expect(shippingMethod).toBe('Expedited');
  });
});
```

### Error Recovery Patterns

Implement graceful error handling:

```typescript
export class ResilientPage {
  async performActionWithRetry<T>(
    action: () => Promise<T>,
    maxRetries: number = 3,
    retryDelay: number = 1000,
  ): Promise<T> {
    for (let attempt = 1; attempt <= maxRetries; attempt++) {
      try {
        return await action();
      } catch (error) {
        if (attempt === maxRetries) {
          throw new Error(`Action failed after ${maxRetries} attempts: ${error}`);
        }

        console.warn(`Attempt ${attempt} failed, retrying in ${retryDelay}ms...`);
        await this.page.waitForTimeout(retryDelay);
      }
    }

    throw new Error('This should never be reached');
  }

  async safeGetText(locator: Locator, defaultValue: string = ''): Promise<string> {
    try {
      return (await locator.textContent()) ?? defaultValue;
    } catch (error) {
      console.warn(`Failed to get text content: ${error}`);
      return defaultValue;
    }
  }
}
```

## Documentation Standards

### Code Comments

Use meaningful comments for complex logic:

```typescript
export class WellDocumentedPage {
  async complexBusinessWorkflow(data: BusinessData): Promise<void> {
    // Step 1: Validate user permissions before proceeding
    // This prevents unauthorized access to sensitive operations
    await this.validateUserPermissions(data.userId);

    // Step 2: Lock the record to prevent concurrent modifications
    // The lock is automatically released after 5 minutes or on completion
    await this.acquireRecordLock(data.recordId);

    try {
      // Step 3: Perform data transformation
      // Convert legacy format to new schema for compatibility
      const transformedData = this.transformLegacyData(data);

      // Step 4: Apply business rules validation
      // These rules are defined in the business requirements document
      await this.validateBusinessRules(transformedData);

      // Step 5: Submit for approval if amount exceeds threshold
      if (transformedData.amount > this.APPROVAL_THRESHOLD) {
        await this.submitForApproval(transformedData);
      } else {
        await this.processDirectly(transformedData);
      }
    } finally {
      // Always release the lock, even if an error occurs
      await this.releaseRecordLock(data.recordId);
    }
  }
}
```

## Quick Checklist

### ✅ Design Quality

- [ ] Each page object has single responsibility
- [ ] Locators are readonly and immutable
- [ ] Methods have clear, descriptive names
- [ ] Proper TypeScript typing throughout
- [ ] No hardcoded values or magic numbers

### ✅ Architecture

- [ ] Consistent directory structure
- [ ] Logical grouping by business domain
- [ ] Appropriate use of inheritance/composition
- [ ] Clear separation of concerns

### ✅ Maintainability

- [ ] Version control strategy in place
- [ ] Deprecation warnings for old methods
- [ ] Health monitoring capabilities
- [ ] Refactoring guidelines established

### ✅ Performance

- [ ] Lazy initialization where appropriate
- [ ] Efficient waiting strategies
- [ ] Locator reuse patterns
- [ ] Cache invalidation strategies

### ✅ Team Collaboration

- [ ] Code review guidelines defined
- [ ] Documentation standards followed
- [ ] Consistent naming conventions
- [ ] Clear migration paths for changes

### ✅ Testing Integration

- [ ] Proper test framework integration
- [ ] Error recovery patterns
- [ ] Health validation in tests
- [ ] Clear test data management

Following these best practices ensures that your Page Object Model implementation remains maintainable, scalable, and reliable as your test suite grows and evolves.
