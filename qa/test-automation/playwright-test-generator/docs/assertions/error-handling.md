# Error Handling in Assertions

**Description**: Comprehensive guide to handling assertion failures, debugging techniques, and creating resilient test assertions. This document covers strategies for dealing with flaky tests, timeout issues, and providing meaningful error information.

## Table of Contents

- [Understanding Assertion Failures](#understanding-assertion-failures)
- [Debugging Strategies](#debugging-strategies)
- [Timeout Management](#timeout-management)
- [Retry Mechanisms](#retry-mechanisms)
- [Custom Error Messages](#custom-error-messages)
- [Flaky Test Mitigation](#flaky-test-mitigation)
- [Error Recovery Patterns](#error-recovery-patterns)
- [Logging and Diagnostics](#logging-and-diagnostics)

## Understanding Assertion Failures

### Common Assertion Failure Types

#### Element Not Found Errors

```typescript
// ❌ Common failure: Element doesn't exist
await expect(page.getByTestId('non-existent-element')).toBeVisible();
// Error: Locator.toBeVisible: Error: locator.isVisible: No such element

// ✅ Better: Check for existence first
const element = page.getByTestId('dynamic-element');
if ((await element.count()) > 0) {
  await expect(element).toBeVisible();
} else {
  console.log('Element not found, checking alternative location');
  await expect(page.getByTestId('alternative-element')).toBeVisible();
}

// ✅ Best: Use conditional assertion with context
await test.step('Validate search results display', async () => {
  const results = page.getByTestId('search-results');
  const noResults = page.getByTestId('no-results-message');

  // Handle both success and empty states
  const hasResults = (await results.count()) > 0;

  if (hasResults) {
    await expect(results).toBeVisible();
    await expect(results.getByTestId('result-item')).toHaveCount.greaterThan(0);
  } else {
    await expect(noResults).toBeVisible();
    await expect(noResults).toHaveText('No results found');
  }
});
```

#### Timing-Related Failures

```typescript
// ❌ Common failure: Element not ready
await expect(page.getByText('Loading complete')).toBeVisible();
// Error: Timeout 5000ms exceeded

// ✅ Better: Appropriate timeout with context
await expect(page.getByText('Loading complete'), 'Data loading should complete within 30 seconds').toBeVisible({
  timeout: 30000,
});

// ✅ Best: Progressive validation with clear steps
await test.step('Wait for data loading to complete', async () => {
  // First, ensure loading started
  await expect(page.getByText('Loading...')).toBeVisible();

  // Then wait for completion with appropriate timeout
  await expect(page.getByText('Loading complete')).toBeVisible({
    timeout: 30000,
  });

  // Finally, validate the loaded data
  await expect(page.getByTestId('data-container')).toBeVisible();
  await expect(page.getByTestId('data-item')).toHaveCount.greaterThan(0);
});
```

#### Text Content Mismatches

```typescript
// ❌ Common failure: Exact text doesn't match
await expect(page.getByTestId('username')).toHaveText('John Doe');
// Error: Expected 'John Doe', received 'John  Doe' (extra space)

// ✅ Better: Use regex for flexible matching
await expect(page.getByTestId('username')).toHaveText(/John\s+Doe/);

// ✅ Best: Normalize and validate with context
await test.step('Validate user name display', async () => {
  const username = page.getByTestId('username');
  const text = await username.textContent();
  const normalizedText = text?.trim().replace(/\s+/g, ' ');

  expect(normalizedText, 'Username should display correctly formatted').toBe('John Doe');
  await expect(username).toBeVisible();
});
```

## Debugging Strategies

### Enhanced Error Context

```typescript
// ✅ Provide detailed context for debugging
test('user profile update', async ({ page }) => {
  const profilePage = new ProfilePage(page);

  await test.step('Navigate to profile page', async () => {
    await profilePage.goto();

    // Capture current state for debugging
    await expect(page, 'Profile page should load successfully').toHaveURL('/profile');

    // Validate page prerequisites
    await expect(profilePage.loadingIndicator, 'Page should finish loading before proceeding').not.toBeVisible();
  });

  await test.step('Update profile information', async () => {
    const updateData = { name: 'Updated Name', email: 'updated@test.com' };

    try {
      await profilePage.updateProfile(updateData);

      // Validate success
      await expect(
        page.getByText('Profile updated successfully'),
        'Success message should appear after profile update',
      ).toBeVisible();
    } catch (error) {
      // Enhanced error reporting
      console.log('Profile update failed. Current page state:');
      console.log('URL:', await page.url());
      console.log('Form values:', {
        name: await profilePage.nameField.inputValue(),
        email: await profilePage.emailField.inputValue(),
      });

      // Check for error messages
      const errorMessages = await page.getByRole('alert').allTextContents();
      if (errorMessages.length > 0) {
        console.log('Error messages found:', errorMessages);
      }

      throw error;
    }
  });
});
```

### Screenshot and Trace Debugging

```typescript
// ✅ Capture debugging information on failure
test('complex workflow debugging', async ({ page }) => {
  const workflowPage = new WorkflowPage(page);

  try {
    await workflowPage.executeComplexWorkflow();

    // Validate success
    await expect(page.getByText('Workflow completed')).toBeVisible();
  } catch (error) {
    // Capture screenshot on failure
    await page.screenshot({
      path: `debug-workflow-failure-${Date.now()}.png`,
      fullPage: true,
    });

    // Capture page state
    console.log('Workflow failure debug info:');
    console.log('Current URL:', await page.url());
    console.log('Page title:', await page.title());

    // Check for specific error indicators
    const errorElements = await page.getByRole('alert').all();
    for (const error of errorElements) {
      console.log('Error found:', await error.textContent());
    }

    // Capture network state if relevant
    const failedRequests = [];
    page.on('response', (response) => {
      if (!response.ok()) {
        failedRequests.push({
          url: response.url(),
          status: response.status(),
        });
      }
    });

    if (failedRequests.length > 0) {
      console.log('Failed network requests:', failedRequests);
    }

    throw error;
  }
});
```

## Timeout Management

### Smart Timeout Strategies

```typescript
// ✅ Context-aware timeout configuration
class AssertionTimeouts {
  static readonly FAST = 2000; // Quick UI updates
  static readonly NORMAL = 5000; // Default Playwright timeout
  static readonly SLOW = 15000; // Form submissions, navigation
  static readonly VERY_SLOW = 30000; // Data processing, file uploads
  static readonly EXTRA_SLOW = 60000; // Large file operations, reports
}

test('timeout management examples', async ({ page }) => {
  const dataPage = new DataProcessingPage(page);

  await test.step('Fast UI interactions', async () => {
    await dataPage.toggleOption();

    // Fast timeout for immediate UI changes
    await expect(dataPage.optionStatus, 'Option status should update immediately').toHaveText('Enabled', {
      timeout: AssertionTimeouts.FAST,
    });
  });

  await test.step('Form submission', async () => {
    await dataPage.submitForm();

    // Normal timeout for form processing
    await expect(
      page.getByText('Form submitted successfully'),
      'Form submission should complete within normal timeframe',
    ).toBeVisible({ timeout: AssertionTimeouts.SLOW });
  });

  await test.step('Data processing', async () => {
    await dataPage.startDataProcessing();

    // Extended timeout for heavy operations
    await expect(
      page.getByText('Processing complete'),
      'Data processing is known to take up to 30 seconds',
    ).toBeVisible({ timeout: AssertionTimeouts.VERY_SLOW });
  });
});
```

### Conditional Timeout Adjustment

```typescript
// ✅ Environment-aware timeout configuration
function getEnvironmentTimeout(baseTimeout: number): number {
  const env = process.env.TEST_ENV || 'local';
  const multipliers = {
    local: 1,
    ci: 2, // CI environments are slower
    staging: 1.5, // Staging might be slower than local
    production: 1, // Production should be fast
  };

  return baseTimeout * (multipliers[env] || 1);
}

test('environment-aware timeouts', async ({ page }) => {
  const loginPage = new LoginPage(page);

  await loginPage.login('user@test.com', 'password');

  // Adjust timeout based on environment
  const profileTimeout = getEnvironmentTimeout(10000);

  await expect(
    page.getByText('Welcome to Profile'),
    'Profile should load within environment-appropriate time',
  ).toBeVisible({ timeout: profileTimeout });
});
```

## Retry Mechanisms

### Built-in Playwright Retries

```typescript
// Configure in playwright.config.ts
export default defineConfig({
  retries: process.env.CI ? 2 : 0, // Retry on CI only
  expect: {
    timeout: 10000,
  },
});

// ✅ Custom retry logic for specific scenarios
async function retryAssertion<T>(
  assertion: () => Promise<T>,
  maxRetries: number = 3,
  delay: number = 1000,
): Promise<T> {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await assertion();
    } catch (error) {
      if (attempt === maxRetries) {
        throw new Error(`Assertion failed after ${maxRetries} attempts: ${error.message}`);
      }

      console.log(`Assertion attempt ${attempt} failed, retrying in ${delay}ms...`);
      await new Promise((resolve) => setTimeout(resolve, delay));
    }
  }

  throw new Error('Unexpected error in retry logic');
}

// Usage example
test('retry assertion example', async ({ page }) => {
  const dynamicPage = new DynamicContentPage(page);

  await dynamicPage.loadContent();

  // Retry assertion for flaky dynamic content
  await retryAssertion(
    async () => {
      await expect(page.getByTestId('dynamic-content')).toBeVisible();
      await expect(page.getByTestId('dynamic-content')).not.toHaveText('Loading...');
    },
    3,
    2000,
  );
});
```

### Smart Retry with Conditions

```typescript
// ✅ Conditional retry based on error type
async function smartRetryAssertion(
  page: Page,
  assertion: () => Promise<void>,
  options: {
    maxRetries?: number;
    retryDelay?: number;
    retryCondition?: (error: Error) => boolean;
  } = {},
): Promise<void> {
  const { maxRetries = 3, retryDelay = 1000, retryCondition = () => true } = options;

  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      await assertion();
      return; // Success
    } catch (error) {
      const shouldRetry = attempt < maxRetries && retryCondition(error);

      if (!shouldRetry) {
        throw error;
      }

      console.log(`Attempt ${attempt} failed: ${error.message}`);
      console.log(`Retrying in ${retryDelay}ms...`);

      // Wait and optionally refresh page state
      await page.waitForTimeout(retryDelay);

      // Optional: Refresh page state before retry
      if (error.message.includes('detached')) {
        await page.reload();
      }
    }
  }
}

// Usage with specific retry conditions
test('smart retry example', async ({ page }) => {
  const searchPage = new SearchPage(page);

  await searchPage.performSearch('test query');

  await smartRetryAssertion(
    page,
    async () => {
      await expect(page.getByTestId('search-results')).toBeVisible();
      await expect(page.getByTestId('result-item')).toHaveCount.greaterThan(0);
    },
    {
      maxRetries: 3,
      retryDelay: 2000,
      retryCondition: (error) => {
        // Retry on timeout or element detached, but not on element not found
        return error.message.includes('Timeout') || error.message.includes('detached');
      },
    },
  );
});
```

## Custom Error Messages

### Descriptive Error Messages

```typescript
// ✅ Context-rich error messages
test('custom error messages', async ({ page }) => {
  const orderPage = new OrderPage(page);

  await test.step('Validate order submission', async () => {
    await orderPage.fillOrderDetails({
      product: 'Widget Pro',
      quantity: 5,
      customerEmail: 'test@example.com',
    });

    await orderPage.submitOrder();

    // Detailed error messages for different scenarios
    await expect(
      page.getByText('Order submitted successfully'),
      'Order submission should show success message immediately after form submission',
    ).toBeVisible();

    await expect(
      page.getByTestId('order-number'),
      'Order number should be generated and displayed after successful submission',
    ).toBeVisible();

    await expect(
      page.getByTestId('order-number'),
      'Order number should follow the format ORD-XXXXXXXX where X is a digit',
    ).toHaveText(/^ORD-\d{8}$/);

    await expect(
      page.getByText('test@example.com'),
      'Customer email should be displayed in the order confirmation for verification',
    ).toBeVisible();
  });
});
```

### Error Message Templates

```typescript
// ✅ Reusable error message templates
class ErrorMessages {
  static elementShouldBeVisible(elementName: string, context: string = ''): string {
    return `${elementName} should be visible${context ? ` ${context}` : ''}`;
  }

  static elementShouldHaveText(elementName: string, expectedText: string): string {
    return `${elementName} should display "${expectedText}"`;
  }

  static elementShouldBeEnabled(elementName: string, reason: string = ''): string {
    return `${elementName} should be enabled${reason ? ` because ${reason}` : ''}`;
  }

  static pageLoadTimeout(pageName: string, timeout: number): string {
    return `${pageName} should load completely within ${timeout}ms`;
  }

  static dataValidation(dataType: string, expectedFormat: string): string {
    return `${dataType} should match the expected format: ${expectedFormat}`;
  }
}

// Usage
test('error message templates', async ({ page }) => {
  const profilePage = new ProfilePage(page);

  await profilePage.goto();

  await expect(
    profilePage.editButton,
    ErrorMessages.elementShouldBeVisible('Edit Profile button', 'after page loads'),
  ).toBeVisible();

  await expect(
    profilePage.saveButton,
    ErrorMessages.elementShouldBeEnabled('Save button', 'form fields have been modified'),
  ).toBeDisabled(); // This will use the custom message

  await expect(page.getByTestId('user-id'), ErrorMessages.dataValidation('User ID', 'UUID format')).toHaveText(
    /^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$/i,
  );
});
```

## Flaky Test Mitigation

### Stabilization Techniques

```typescript
// ✅ Wait for stable state before assertions
async function waitForStableElement(locator: Locator, timeoutMs: number = 5000): Promise<void> {
  const startTime = Date.now();
  let previousBoundingBox: any = null;

  while (Date.now() - startTime < timeoutMs) {
    try {
      const currentBoundingBox = await locator.boundingBox();

      if (currentBoundingBox && previousBoundingBox) {
        // Check if element position is stable
        const isStable =
          Math.abs(currentBoundingBox.x - previousBoundingBox.x) < 1 &&
          Math.abs(currentBoundingBox.y - previousBoundingBox.y) < 1 &&
          Math.abs(currentBoundingBox.width - previousBoundingBox.width) < 1 &&
          Math.abs(currentBoundingBox.height - previousBoundingBox.height) < 1;

        if (isStable) {
          return; // Element is stable
        }
      }

      previousBoundingBox = currentBoundingBox;
      await new Promise((resolve) => setTimeout(resolve, 100));
    } catch (error) {
      // Element might not be visible yet, continue waiting
      await new Promise((resolve) => setTimeout(resolve, 100));
    }
  }

  throw new Error(`Element did not stabilize within ${timeoutMs}ms`);
}

test('stable element interaction', async ({ page }) => {
  const animatedPage = new AnimatedInterfacePage(page);

  await animatedPage.triggerAnimation();

  // Wait for element to stop moving before asserting
  const animatedElement = page.getByTestId('animated-button');
  await waitForStableElement(animatedElement);

  await expect(animatedElement).toBeVisible();
  await expect(animatedElement).toBeEnabled();

  // Now safe to interact
  await animatedElement.click();
});
```

### Anti-Flaky Patterns

```typescript
// ✅ Robust waiting patterns
test('anti-flaky assertion patterns', async ({ page }) => {
  const dataPage = new DataLoadingPage(page);

  await test.step('Wait for initial page load', async () => {
    await dataPage.goto();

    // Wait for specific load state, not just DOMContentLoaded
    await page.waitForLoadState('networkidle');

    // Ensure critical elements are loaded
    await expect(page.getByTestId('main-container')).toBeVisible();
    await expect(page.locator('body')).not.toHaveClass('loading');
  });

  await test.step('Wait for data loading completion', async () => {
    await dataPage.loadData();

    // Wait for loading indicator to appear first (ensures request started)
    await expect(page.getByTestId('loading-spinner')).toBeVisible();

    // Then wait for it to disappear (ensures request completed)
    await expect(page.getByTestId('loading-spinner')).not.toBeVisible({ timeout: 30000 });

    // Finally validate the loaded data
    await expect(page.getByTestId('data-container')).toBeVisible();
    await expect(page.getByTestId('data-item')).toHaveCount.greaterThan(0);
  });

  await test.step('Validate interactive elements', async () => {
    // Wait for all interactive elements to be ready
    await expect(page.getByRole('button', { name: 'Export' })).toBeEnabled();
    await expect(page.getByRole('button', { name: 'Refresh' })).toBeEnabled();
    await expect(page.getByLabel('Filter')).toBeEnabled();

    // Validate they're not just enabled but actually functional
    const exportButton = page.getByRole('button', { name: 'Export' });
    await exportButton.click();
    await expect(page.getByText('Export started')).toBeVisible();
  });
});
```

## Error Recovery Patterns

### Graceful Error Recovery

```typescript
// ✅ Error recovery with fallback strategies
test('error recovery patterns', async ({ page }) => {
  const searchPage = new SearchPage(page);

  await test.step('Primary search attempt', async () => {
    try {
      await searchPage.performSearch('test query');
      await expect(page.getByTestId('search-results')).toBeVisible();
    } catch (primaryError) {
      console.log('Primary search failed, attempting recovery...');

      // Recovery strategy 1: Refresh and retry
      await page.reload();
      await page.waitForLoadState('networkidle');

      try {
        await searchPage.performSearch('test query');
        await expect(page.getByTestId('search-results')).toBeVisible();
        console.log('Recovery successful after page refresh');
      } catch (recoveryError) {
        // Recovery strategy 2: Use alternative search method
        console.log('Standard recovery failed, trying alternative approach...');

        await searchPage.useAlternativeSearch('test query');
        await expect(page.getByTestId('alternative-results')).toBeVisible();
        console.log('Alternative search method successful');
      }
    }
  });
});
```

### Assertion Fallback Chains

```typescript
// ✅ Assertion fallback patterns
async function assertElementWithFallbacks(
  page: Page,
  primarySelector: string,
  fallbackSelectors: string[],
  expectedState: 'visible' | 'enabled' | 'text',
  expectedValue?: string,
): Promise<void> {
  const selectors = [primarySelector, ...fallbackSelectors];

  for (let i = 0; i < selectors.length; i++) {
    const selector = selectors[i];
    const isLastAttempt = i === selectors.length - 1;

    try {
      const element = page.locator(selector);

      switch (expectedState) {
        case 'visible':
          await expect(element).toBeVisible({ timeout: 5000 });
          break;
        case 'enabled':
          await expect(element).toBeEnabled({ timeout: 5000 });
          break;
        case 'text':
          if (expectedValue) {
            await expect(element).toHaveText(expectedValue, { timeout: 5000 });
          }
          break;
      }

      console.log(`Success with selector: ${selector}`);
      return;
    } catch (error) {
      if (isLastAttempt) {
        throw new Error(
          `All selector attempts failed. Selectors tried: ${selectors.join(', ')}. Last error: ${error.message}`,
        );
      }

      console.log(`Selector failed: ${selector}, trying next...`);
    }
  }
}

// Usage
test('fallback assertion example', async ({ page }) => {
  await page.goto('/dynamic-page');

  // Try multiple selectors for the same logical element
  await assertElementWithFallbacks(
    page,
    '[data-testid="submit-button"]', // Primary
    ['button[type="submit"]', '.submit-btn'], // Fallbacks
    'visible',
  );
});
```

## Logging and Diagnostics

### Comprehensive Error Logging

```typescript
// ✅ Enhanced diagnostic logging
class TestDiagnostics {
  static async capturePageState(page: Page, context: string): Promise<void> {
    console.log(`=== Page State Capture: ${context} ===`);
    console.log('URL:', await page.url());
    console.log('Title:', await page.title());
    console.log('Viewport:', await page.viewportSize());

    // Capture console logs
    const logs = [];
    page.on('console', (msg) => {
      logs.push(`${msg.type()}: ${msg.text()}`);
    });

    if (logs.length > 0) {
      console.log('Console logs:', logs);
    }

    // Capture network errors
    const networkErrors = [];
    page.on('response', (response) => {
      if (!response.ok()) {
        networkErrors.push(`${response.status()}: ${response.url()}`);
      }
    });

    if (networkErrors.length > 0) {
      console.log('Network errors:', networkErrors);
    }

    // Capture visible error messages
    try {
      const alerts = await page.getByRole('alert').all();
      if (alerts.length > 0) {
        const alertTexts = await Promise.all(alerts.map((alert) => alert.textContent()));
        console.log('Alert messages:', alertTexts);
      }
    } catch (error) {
      // No alerts found, continue
    }

    console.log('=== End Page State Capture ===');
  }

  static async captureOnFailure(page: Page, testName: string): Promise<void> {
    const timestamp = new Date().toISOString().replace(/:/g, '-');
    const screenshotPath = `debug-${testName}-${timestamp}.png`;

    await page.screenshot({
      path: screenshotPath,
      fullPage: true,
    });

    console.log(`Screenshot saved: ${screenshotPath}`);

    await this.capturePageState(page, `Failure in ${testName}`);
  }
}

// Usage in tests
test('diagnostic logging example', async ({ page }) => {
  const complexPage = new ComplexWorkflowPage(page);

  try {
    await complexPage.executeWorkflow();

    await expect(page.getByText('Workflow completed')).toBeVisible();
  } catch (error) {
    await TestDiagnostics.captureOnFailure(page, 'complex-workflow');
    throw error;
  }
});
```

By implementing these error handling strategies, your Playwright tests become more resilient, easier to debug, and provide better diagnostic information when issues occur.
