# Performance Considerations

**Description**: Guide to optimizing assertion performance while maintaining test reliability. This document covers strategies for efficient assertion patterns, performance monitoring, and scalable test execution.

## Table of Contents

- [Assertion Performance Fundamentals](#assertion-performance-fundamentals)
- [Efficient Locator Strategies](#efficient-locator-strategies)
- [Batch Assertion Patterns](#batch-assertion-patterns)
- [Timeout Optimization](#timeout-optimization)
- [Memory Management](#memory-management)
- [Parallel Assertion Execution](#parallel-assertion-execution)
- [Performance Monitoring](#performance-monitoring)
- [Scalability Patterns](#scalability-patterns)

## Assertion Performance Fundamentals

### Understanding Assertion Costs

```typescript
// ❌ Inefficient - Multiple DOM queries for same element
test('inefficient assertions', async ({ page }) => {
  await page.goto('/dashboard');

  // Each assertion creates a new locator and queries DOM
  await expect(page.getByTestId('user-name')).toBeVisible();
  await expect(page.getByTestId('user-name')).toHaveText('John Doe');
  await expect(page.getByTestId('user-name')).toHaveClass('highlighted');
  await expect(page.getByTestId('user-name')).toHaveAttribute('data-user-id', '123');
});

// ✅ Efficient - Reuse locator for multiple assertions
test('efficient assertions', async ({ page }) => {
  await page.goto('/dashboard');

  // Create locator once, reuse for multiple assertions
  const userName = page.getByTestId('user-name');

  await expect(userName).toBeVisible();
  await expect(userName).toHaveText('John Doe');
  await expect(userName).toHaveClass('highlighted');
  await expect(userName).toHaveAttribute('data-user-id', '123');
});
```

### Locator Optimization Patterns

```typescript
// ✅ Efficient locator patterns
export class PerformantAssertions {
  static async validateUserCard(
    page: Page,
    userData: {
      name: string;
      email: string;
      role: string;
      status: string;
    },
  ): Promise<void> {
    // Single container lookup
    const userCard = page.getByTestId('user-card');

    // Validate container exists first
    await expect(userCard).toBeVisible();

    // Use child selectors from container (more efficient than global search)
    await expect(userCard.getByTestId('name')).toHaveText(userData.name);
    await expect(userCard.getByTestId('email')).toHaveText(userData.email);
    await expect(userCard.getByTestId('role')).toHaveText(userData.role);
    await expect(userCard.getByTestId('status')).toHaveText(userData.status);
  }

  static async validateTableStructure(page: Page): Promise<void> {
    // Cache table reference
    const table = page.getByRole('table');

    // Validate table existence once
    await expect(table).toBeVisible();

    // Use table context for all child validations
    const headers = table.locator('thead th');
    const rows = table.locator('tbody tr');

    await expect(headers).toHaveCount(5);
    await expect(rows).toHaveCount.greaterThan(0);

    // Validate specific content using cached references
    await expect(headers.nth(0)).toHaveText('Name');
    await expect(headers.nth(1)).toHaveText('Email');
    await expect(headers.nth(2)).toHaveText('Role');
  }
}
```

## Efficient Locator Strategies

### Semantic Locator Performance

```typescript
// ✅ Performance-optimized semantic locators
export class OptimizedLocators {
  static getUserDashboard(page: Page) {
    // Cache commonly used containers
    const dashboard = page.getByRole('main');

    return {
      container: dashboard,
      navigation: dashboard.getByRole('navigation'),
      content: dashboard.getByTestId('dashboard-content'),
      sidebar: dashboard.getByTestId('sidebar'),

      // Nested locators for efficiency
      getUserInfo: () => dashboard.getByTestId('user-info'),
      getNotifications: () => dashboard.getByTestId('notifications'),
      getQuickActions: () => dashboard.getByTestId('quick-actions'),
    };
  }

  static getDataTable(page: Page, tableId: string) {
    const table = page.getByTestId(tableId);

    return {
      container: table,
      headers: table.locator('thead th'),
      rows: table.locator('tbody tr'),
      cells: (row: number, col: number) => table.locator(`tbody tr:nth-child(${row + 1}) td:nth-child(${col + 1})`),
      pagination: table.locator('+ [data-testid="pagination"]'),
    };
  }
}

// Usage for efficient batch assertions
test('optimized locator usage', async ({ page }) => {
  await page.goto('/dashboard');

  const dashboard = OptimizedLocators.getUserDashboard(page);

  // Efficient batch validation using cached locators
  await expect(dashboard.container).toBeVisible();
  await expect(dashboard.navigation).toBeVisible();
  await expect(dashboard.content).toBeVisible();

  const userInfo = dashboard.getUserInfo();
  await expect(userInfo).toBeVisible();
  await expect(userInfo.getByText('John Doe')).toBeVisible();
  await expect(userInfo.getByText('Administrator')).toBeVisible();
});
```

### Locator Filtering Performance

```typescript
// ❌ Inefficient filtering
test('inefficient filtering', async ({ page }) => {
  await page.goto('/users');

  // Multiple DOM queries with different filters
  await expect(page.getByRole('row').filter({ hasText: 'John' })).toBeVisible();
  await expect(page.getByRole('row').filter({ hasText: 'Admin' })).toHaveCount(3);
  await expect(page.getByRole('row').filter({ hasText: 'Active' })).toHaveCount(15);
});

// ✅ Efficient filtering with cached base locator
test('efficient filtering', async ({ page }) => {
  await page.goto('/users');

  // Cache base locator
  const tableRows = page.getByRole('row');

  // Reuse base locator for filtering
  await expect(tableRows.filter({ hasText: 'John' })).toBeVisible();
  await expect(tableRows.filter({ hasText: 'Admin' })).toHaveCount(3);
  await expect(tableRows.filter({ hasText: 'Active' })).toHaveCount(15);

  // Even better: combine filters when possible
  const adminRows = tableRows.filter({ hasText: 'Admin' });
  await expect(adminRows).toHaveCount(3);
  await expect(adminRows.filter({ hasText: 'Active' })).toHaveCount(2);
});
```

## Batch Assertion Patterns

### Parallel Safe Assertions

```typescript
// ✅ Parallel assertion execution for independent validations
export class BatchAssertions {
  static async validatePageStructure(page: Page): Promise<void> {
    // These assertions are independent and can run in parallel
    await Promise.all([
      expect(page.getByRole('banner')).toBeVisible(),
      expect(page.getByRole('navigation')).toBeVisible(),
      expect(page.getByRole('main')).toBeVisible(),
      expect(page.getByRole('contentinfo')).toBeVisible(),
    ]);
  }

  static async validateFormFields(page: Page, formData: Record<string, string>): Promise<void> {
    // Parallel validation of independent form fields
    const fieldValidations = Object.entries(formData).map(([label, value]) =>
      expect(page.getByLabel(label)).toHaveValue(value),
    );

    await Promise.all(fieldValidations);
  }

  static async validateTableHeaders(page: Page, expectedHeaders: string[]): Promise<void> {
    const headerElements = page.locator('thead th');

    // Validate count first
    await expect(headerElements).toHaveCount(expectedHeaders.length);

    // Parallel validation of header texts
    const headerValidations = expectedHeaders.map((header, index) =>
      expect(headerElements.nth(index)).toHaveText(header),
    );

    await Promise.all(headerValidations);
  }
}

// Usage
test('batch assertions example', async ({ page }) => {
  await page.goto('/users');

  // Parallel structure validation
  await BatchAssertions.validatePageStructure(page);

  // Parallel header validation
  await BatchAssertions.validateTableHeaders(page, ['Name', 'Email', 'Role', 'Status', 'Actions']);

  // Parallel form validation after filling
  await page.getByLabel('Search').fill('john');
  await page.getByLabel('Role Filter').selectOption('admin');

  await BatchAssertions.validateFormFields(page, {
    Search: 'john',
    'Role Filter': 'admin',
  });
});
```

### Sequential vs Parallel Decision Making

```typescript
// ✅ Smart assertion ordering
export class SmartAssertions {
  static async validateUserWorkflow(page: Page): Promise<void> {
    // Sequential assertions that depend on each other
    await expect(page.getByTestId('login-form')).toBeVisible();

    // Fill form sequentially (each step depends on previous)
    await page.getByLabel('Username').fill('john@example.com');
    await expect(page.getByLabel('Username')).toHaveValue('john@example.com');

    await page.getByLabel('Password').fill('password123');
    await expect(page.getByLabel('Password')).toHaveValue('password123');

    // Submit button should now be enabled
    await expect(page.getByRole('button', { name: 'Login' })).toBeEnabled();

    await page.getByRole('button', { name: 'Login' }).click();

    // After login, these validations can run in parallel
    await Promise.all([
      expect(page).toHaveURL('/dashboard'),
      expect(page.getByText('Welcome')).toBeVisible(),
      expect(page.getByRole('button', { name: 'Logout' })).toBeVisible(),
      expect(page.getByTestId('user-menu')).toBeVisible(),
    ]);
  }
}
```

## Timeout Optimization

### Dynamic Timeout Adjustment

```typescript
// ✅ Performance-aware timeout configuration
export class TimeoutOptimization {
  private static readonly TIMEOUT_PROFILES = {
    fast: 2000, // UI updates, cached data
    normal: 5000, // Standard operations
    slow: 15000, // Network requests, form submissions
    verySlow: 30000, // Large data loads, file operations
  };

  static async validateWithAppropriateTimeout(
    page: Page,
    validationType: 'ui' | 'network' | 'data' | 'file',
  ): Promise<void> {
    const timeoutMap = {
      ui: this.TIMEOUT_PROFILES.fast,
      network: this.TIMEOUT_PROFILES.normal,
      data: this.TIMEOUT_PROFILES.slow,
      file: this.TIMEOUT_PROFILES.verySlow,
    };

    const timeout = timeoutMap[validationType];

    switch (validationType) {
      case 'ui':
        await expect(page.getByTestId('ui-element')).toBeVisible({ timeout });
        break;
      case 'network':
        await expect(page.getByText('Data loaded')).toBeVisible({ timeout });
        break;
      case 'data':
        await expect(page.getByTestId('large-dataset')).toBeVisible({ timeout });
        break;
      case 'file':
        await expect(page.getByText('File processed')).toBeVisible({ timeout });
        break;
    }
  }

  static async adaptiveTimeout<T>(
    assertion: () => Promise<T>,
    baseTimeout: number,
    retryCount: number = 0,
  ): Promise<T> {
    const adaptedTimeout = baseTimeout * Math.pow(1.5, retryCount);

    try {
      return await assertion();
    } catch (error) {
      if (retryCount < 2 && error.message.includes('Timeout')) {
        console.log(`Assertion timed out, retrying with ${adaptedTimeout}ms timeout...`);
        return this.adaptiveTimeout(assertion, adaptedTimeout, retryCount + 1);
      }
      throw error;
    }
  }
}
```

### Conditional Timeout Strategies

```typescript
// ✅ Environment-aware timeout optimization
export function getOptimizedTimeout(baseTimeout: number, operation: 'ui' | 'network' | 'computation'): number {
  const environment = process.env.TEST_ENV || 'local';
  const isHeadless = process.env.HEADLESS !== 'false';
  const isCi = process.env.CI === 'true';

  let multiplier = 1;

  // Environment adjustments
  if (isCi) multiplier *= 2; // CI is typically slower
  if (!isHeadless) multiplier *= 1.3; // Headed mode has rendering overhead

  // Operation-specific adjustments
  const operationMultipliers = {
    ui: 1, // UI operations are usually fast
    network: 1.5, // Network operations need more time
    computation: 2, // Heavy computations need most time
  };

  multiplier *= operationMultipliers[operation];

  return Math.round(baseTimeout * multiplier);
}

// Usage
test('optimized timeout example', async ({ page }) => {
  await page.goto('/data-processing');

  const uiTimeout = getOptimizedTimeout(5000, 'ui');
  const networkTimeout = getOptimizedTimeout(10000, 'network');
  const computationTimeout = getOptimizedTimeout(20000, 'computation');

  // UI validation with appropriate timeout
  await expect(page.getByTestId('processing-form')).toBeVisible({
    timeout: uiTimeout,
  });

  // Trigger data processing
  await page.getByRole('button', { name: 'Start Processing' }).click();

  // Network operation validation
  await expect(page.getByText('Processing started')).toBeVisible({
    timeout: networkTimeout,
  });

  // Computation completion validation
  await expect(page.getByText('Processing complete')).toBeVisible({
    timeout: computationTimeout,
  });
});
```

## Memory Management

### Locator Lifecycle Management

```typescript
// ✅ Memory-efficient locator management
export class MemoryEfficientAssertions {
  private locatorCache = new Map<string, Locator>();

  constructor(private page: Page) {}

  getLocator(key: string, selector: string): Locator {
    if (!this.locatorCache.has(key)) {
      this.locatorCache.set(key, this.page.locator(selector));
    }
    return this.locatorCache.get(key)!;
  }

  async validateWithCache(
    validations: Array<{
      key: string;
      selector: string;
      assertion: (locator: Locator) => Promise<void>;
    }>,
  ): Promise<void> {
    for (const { key, selector, assertion } of validations) {
      const locator = this.getLocator(key, selector);
      await assertion(locator);
    }
  }

  clearCache(): void {
    this.locatorCache.clear();
  }

  async cleanup(): Promise<void> {
    this.clearCache();
  }
}

// Usage with proper cleanup
test('memory efficient assertions', async ({ page }) => {
  const assertions = new MemoryEfficientAssertions(page);

  try {
    await page.goto('/complex-page');

    await assertions.validateWithCache([
      {
        key: 'header',
        selector: '[data-testid="header"]',
        assertion: async (locator) => await expect(locator).toBeVisible(),
      },
      {
        key: 'navigation',
        selector: '[data-testid="nav"]',
        assertion: async (locator) => await expect(locator).toBeVisible(),
      },
      {
        key: 'content',
        selector: '[data-testid="content"]',
        assertion: async (locator) => await expect(locator).toBeVisible(),
      },
    ]);
  } finally {
    await assertions.cleanup();
  }
});
```

### Resource Cleanup Patterns

```typescript
// ✅ Resource cleanup for long-running tests
export class ResourceManagedAssertions {
  private static activeResources = new Set<string>();

  static async withResourceTracking<T>(resourceId: string, operation: () => Promise<T>): Promise<T> {
    this.activeResources.add(resourceId);

    try {
      return await operation();
    } finally {
      this.activeResources.delete(resourceId);
    }
  }

  static getActiveResourceCount(): number {
    return this.activeResources.size;
  }

  static async cleanupAllResources(): Promise<void> {
    this.activeResources.clear();
  }
}

// Usage
test('resource managed test', async ({ page }) => {
  await ResourceManagedAssertions.withResourceTracking('main-test', async () => {
    await page.goto('/resource-intensive-page');

    // Your assertions here
    await expect(page.getByTestId('heavy-component')).toBeVisible();

    // Resource count should be managed automatically
    console.log('Active resources:', ResourceManagedAssertions.getActiveResourceCount());
  });
});
```

## Parallel Assertion Execution

### Safe Parallel Patterns

```typescript
// ✅ Safe parallel assertion execution
export class ParallelAssertions {
  static async validatePageSections(page: Page, sections: string[]): Promise<void> {
    // Safe parallel validation - each section is independent
    const sectionValidations = sections.map(async (section) => {
      const element = page.getByTestId(section);
      await expect(element).toBeVisible();
      return element;
    });

    const validatedSections = await Promise.all(sectionValidations);

    // Sequential validation that depends on parallel results
    for (const section of validatedSections) {
      await expect(section.getByRole('heading')).toBeVisible();
    }
  }

  static async validateFormFieldsParallel(
    page: Page,
    fieldValidations: Array<{
      label: string;
      expectedValue: string;
      shouldBeEnabled: boolean;
    }>,
  ): Promise<void> {
    // Parallel field validations
    const validations = fieldValidations.map(async ({ label, expectedValue, shouldBeEnabled }) => {
      const field = page.getByLabel(label);

      // Each field validation is independent
      await Promise.all([
        expect(field).toHaveValue(expectedValue),
        shouldBeEnabled ? expect(field).toBeEnabled() : expect(field).toBeDisabled(),
      ]);

      return field;
    });

    await Promise.all(validations);
  }

  static async validateTableDataParallel(
    page: Page,
    tableSelector: string,
    expectedData: Array<{ column: string; value: string }>,
  ): Promise<void> {
    const table = page.locator(tableSelector);
    await expect(table).toBeVisible();

    // Parallel validation of table cells
    const cellValidations = expectedData.map(({ column, value }) =>
      expect(table.locator(`[data-column="${column}"]`).first()).toHaveText(value),
    );

    await Promise.all(cellValidations);
  }
}
```

## Performance Monitoring

### Assertion Performance Tracking

```typescript
// ✅ Performance monitoring for assertions
export class AssertionPerformanceMonitor {
  private static metrics: Array<{
    name: string;
    duration: number;
    timestamp: number;
  }> = [];

  static async measureAssertion<T>(name: string, assertion: () => Promise<T>): Promise<T> {
    const startTime = performance.now();

    try {
      const result = await assertion();
      const duration = performance.now() - startTime;

      this.metrics.push({
        name,
        duration,
        timestamp: Date.now(),
      });

      // Log slow assertions
      if (duration > 5000) {
        console.warn(`Slow assertion detected: ${name} took ${duration.toFixed(2)}ms`);
      }

      return result;
    } catch (error) {
      const duration = performance.now() - startTime;
      console.error(`Assertion failed after ${duration.toFixed(2)}ms: ${name}`);
      throw error;
    }
  }

  static getMetrics() {
    return [...this.metrics];
  }

  static getSlowAssertions(threshold: number = 3000) {
    return this.metrics.filter((metric) => metric.duration > threshold);
  }

  static generateReport(): string {
    const totalAssertions = this.metrics.length;
    const averageDuration = this.metrics.reduce((sum, m) => sum + m.duration, 0) / totalAssertions;
    const slowAssertions = this.getSlowAssertions();

    return `
    Performance Report:
    - Total Assertions: ${totalAssertions}
    - Average Duration: ${averageDuration.toFixed(2)}ms
    - Slow Assertions (>3s): ${slowAssertions.length}
    - Slowest Assertion: ${Math.max(...this.metrics.map((m) => m.duration)).toFixed(2)}ms
    `;
  }
}

// Usage
test('performance monitored test', async ({ page }) => {
  await page.goto('/dashboard');

  await AssertionPerformanceMonitor.measureAssertion('page-load-validation', async () => {
    await expect(page.getByTestId('dashboard')).toBeVisible();
    await expect(page.getByTestId('user-info')).toBeVisible();
    await expect(page.getByTestId('navigation')).toBeVisible();
  });

  await AssertionPerformanceMonitor.measureAssertion('data-load-validation', async () => {
    await expect(page.getByTestId('data-table')).toBeVisible();
    await expect(page.getByTestId('chart')).toBeVisible();
  });

  // Generate performance report
  console.log(AssertionPerformanceMonitor.generateReport());
});
```

## Scalability Patterns

### Large Dataset Validation

```typescript
// ✅ Scalable patterns for large datasets
export class ScalableAssertions {
  static async validateLargeTable(
    page: Page,
    tableSelector: string,
    options: {
      sampleSize?: number;
      validateAllRows?: boolean;
      expectedMinRows?: number;
    } = {},
  ): Promise<void> {
    const { sampleSize = 10, validateAllRows = false, expectedMinRows = 1 } = options;

    const table = page.locator(tableSelector);
    const rows = table.locator('tbody tr');

    // Quick validation of table presence and minimum rows
    await expect(table).toBeVisible();
    await expect(rows).toHaveCount.greaterThanOrEqual(expectedMinRows);

    const totalRows = await rows.count();

    if (validateAllRows && totalRows <= 100) {
      // For small datasets, validate all rows
      for (let i = 0; i < totalRows; i++) {
        await expect(rows.nth(i)).toBeVisible();
      }
    } else {
      // For large datasets, validate a sample
      const indicesToCheck = this.generateSampleIndices(totalRows, sampleSize);

      for (const index of indicesToCheck) {
        await expect(rows.nth(index)).toBeVisible();
      }
    }
  }

  private static generateSampleIndices(totalCount: number, sampleSize: number): number[] {
    if (totalCount <= sampleSize) {
      return Array.from({ length: totalCount }, (_, i) => i);
    }

    const indices: number[] = [];
    const step = Math.floor(totalCount / sampleSize);

    for (let i = 0; i < sampleSize; i++) {
      indices.push(i * step);
    }

    return indices;
  }

  static async validateInfiniteScroll(
    page: Page,
    containerSelector: string,
    options: {
      itemSelector: string;
      expectedMinItems: number;
      maxScrollAttempts: number;
    },
  ): Promise<void> {
    const container = page.locator(containerSelector);
    const items = container.locator(options.itemSelector);

    let scrollAttempts = 0;
    let previousCount = 0;

    while (scrollAttempts < options.maxScrollAttempts) {
      const currentCount = await items.count();

      if (currentCount >= options.expectedMinItems) {
        break;
      }

      if (currentCount === previousCount) {
        // No new items loaded, might have reached the end
        break;
      }

      // Scroll to load more items
      await container.scrollIntoViewIfNeeded();
      await page.keyboard.press('End');
      await page.waitForTimeout(1000); // Wait for items to load

      previousCount = currentCount;
      scrollAttempts++;
    }

    // Validate final state
    await expect(items).toHaveCount.greaterThanOrEqual(options.expectedMinItems);
  }
}
```

By implementing these performance optimization strategies, your Playwright assertions will run faster, use resources more efficiently, and scale better with your application's complexity.
