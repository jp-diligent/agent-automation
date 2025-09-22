# Assertion Anti-Patterns

**Description**: Common mistakes and anti-patterns to avoid when writing Playwright assertions. This guide helps identify problematic patterns and provides better alternatives for reliable, maintainable test assertions.

## Table of Contents

- [Timing Anti-Patterns](#timing-anti-patterns)
- [Locator Anti-Patterns](#locator-anti-patterns)
- [Assertion Structure Anti-Patterns](#assertion-structure-anti-patterns)
- [Error Handling Anti-Patterns](#error-handling-anti-patterns)
- [Performance Anti-Patterns](#performance-anti-patterns)
- [Maintainability Anti-Patterns](#maintainability-anti-patterns)
- [Test Organization Anti-Patterns](#test-organization-anti-patterns)
- [Debugging Anti-Patterns](#debugging-anti-patterns)

## Timing Anti-Patterns

### ❌ Manual Waits Before Assertions

```typescript
// ❌ Don't use manual waits before assertions
test('manual wait anti-pattern', async ({ page }) => {
  await page.goto('/dashboard');

  // BAD: Manual timeout before assertion
  await page.waitForTimeout(3000);
  await expect(page.getByText('Welcome')).toBeVisible();

  // BAD: Arbitrary delay
  await new Promise((resolve) => setTimeout(resolve, 2000));
  await expect(page.getByTestId('data-loaded')).toBeVisible();
});

// ✅ Use Playwright's auto-waiting instead
test('auto-waiting pattern', async ({ page }) => {
  await page.goto('/dashboard');

  // GOOD: Let Playwright wait automatically
  await expect(page.getByText('Welcome')).toBeVisible();
  await expect(page.getByTestId('data-loaded')).toBeVisible();

  // GOOD: Use specific wait conditions when needed
  await expect(page.getByText('Data processing complete')).toBeVisible({
    timeout: 30000,
  });
});
```

### ❌ Checking States with Manual Polling

```typescript
// ❌ Don't manually poll for element states
test('manual polling anti-pattern', async ({ page }) => {
  await page.goto('/async-content');

  // BAD: Manual polling loop
  let attempts = 0;
  while (attempts < 10) {
    try {
      const element = await page.locator('[data-testid="async-content"]');
      if (await element.isVisible()) {
        break;
      }
    } catch (e) {
      // Ignore error and retry
    }
    await page.waitForTimeout(500);
    attempts++;
  }

  await expect(page.getByTestId('async-content')).toBeVisible();
});

// ✅ Use built-in waiting mechanisms
test('built-in waiting pattern', async ({ page }) => {
  await page.goto('/async-content');

  // GOOD: Single assertion with auto-waiting
  await expect(page.getByTestId('async-content')).toBeVisible();

  // GOOD: Custom timeout if needed
  await expect(page.getByTestId('slow-content')).toBeVisible({
    timeout: 15000,
  });
});
```

## Locator Anti-Patterns

### ❌ Overly Specific Selectors

```typescript
// ❌ Don't use brittle, overly specific selectors
test('brittle selectors anti-pattern', async ({ page }) => {
  await page.goto('/users');

  // BAD: Position-dependent selectors
  await expect(page.locator('div > div > div:nth-child(3) > span')).toBeVisible();
  await expect(page.locator('tr:nth-of-type(2) td:nth-of-type(3)')).toHaveText('Admin');

  // BAD: Implementation-dependent class names
  await expect(page.locator('.css-1a2b3c4 .MuiButton-root-5d6e7f8')).toBeVisible();
});

// ✅ Use semantic, stable selectors
test('semantic selectors pattern', async ({ page }) => {
  await page.goto('/users');

  // GOOD: Semantic locators
  await expect(page.getByRole('button', { name: 'Add User' })).toBeVisible();
  await expect(page.getByText('John Doe')).toBeVisible();

  // GOOD: Test IDs for complex elements
  await expect(page.getByTestId('user-management-table')).toBeVisible();

  // GOOD: Stable attributes
  await expect(page.getByLabel('Search users')).toBeVisible();
});
```

### ❌ XPath Selectors

```typescript
// ❌ Avoid XPath selectors
test('xpath anti-pattern', async ({ page }) => {
  await page.goto('/complex-form');

  // BAD: XPath selectors are hard to maintain
  await expect(page.locator('//div[@class="form-group"]//input[@type="text"][1]')).toBeVisible();
  await expect(page.locator('//button[contains(text(), "Submit") and @type="submit"]')).toBeEnabled();
  await expect(page.locator('//table//tr[position()>1]//td[3]')).toHaveText('Active');
});

// ✅ Use CSS selectors or semantic locators
test('css and semantic locators pattern', async ({ page }) => {
  await page.goto('/complex-form');

  // GOOD: CSS selectors
  await expect(page.locator('input[name="firstName"]')).toBeVisible();

  // GOOD: Semantic locators (preferred)
  await expect(page.getByRole('button', { name: 'Submit' })).toBeEnabled();
  await expect(page.getByLabel('First Name')).toBeVisible();

  // GOOD: Test IDs for complex elements
  await expect(page.getByTestId('status-cell')).toHaveText('Active');
});
```

### ❌ Generic Selectors Without Context

```typescript
// ❌ Don't use overly generic selectors
test('generic selectors anti-pattern', async ({ page }) => {
  await page.goto('/dashboard');

  // BAD: Too generic, could match many elements
  await expect(page.locator('button')).toBeVisible();
  await expect(page.locator('div')).toHaveText('Welcome');
  await expect(page.locator('span')).toBeVisible();
  await expect(page.locator('.error')).not.toBeVisible(); // Which error?
});

// ✅ Use specific, contextual selectors
test('specific selectors pattern', async ({ page }) => {
  await page.goto('/dashboard');

  // GOOD: Specific role-based selectors
  await expect(page.getByRole('button', { name: 'Save Settings' })).toBeVisible();
  await expect(page.getByRole('heading', { name: 'Dashboard' })).toBeVisible();

  // GOOD: Contextual selectors
  await expect(page.getByTestId('welcome-message')).toHaveText('Welcome');
  await expect(page.getByTestId('user-form').getByText('Error')).not.toBeVisible();
});
```

## Assertion Structure Anti-Patterns

### ❌ Assertions in Page Objects

```typescript
// ❌ Don't put assertions in page objects
class LoginPageAntiPattern {
  constructor(private page: Page) {}

  async login(username: string, password: string): Promise<void> {
    await this.page.getByLabel('Username').fill(username);
    await this.page.getByLabel('Password').fill(password);
    await this.page.getByRole('button', { name: 'Login' }).click();

    // BAD: Assertions in page object
    await expect(this.page).toHaveURL('/dashboard');
    await expect(this.page.getByText('Welcome')).toBeVisible();
  }
}

// ✅ Keep assertions in tests, actions in page objects
class LoginPage {
  constructor(private page: Page) {}

  async goto(): Promise<void> {
    await this.page.goto('/login');
  }

  async login(username: string, password: string): Promise<void> {
    await this.page.getByLabel('Username').fill(username);
    await this.page.getByLabel('Password').fill(password);
    await this.page.getByRole('button', { name: 'Login' }).click();
  }

  // GOOD: Return data for tests to assert on
  async getWelcomeMessage(): Promise<string | null> {
    return await this.page.getByTestId('welcome-message').textContent();
  }
}

test('proper separation pattern', async ({ page }) => {
  const loginPage = new LoginPage(page);

  await loginPage.goto();
  await loginPage.login('user@test.com', 'password');

  // GOOD: Assertions in test
  await expect(page).toHaveURL('/dashboard');
  await expect(page.getByText('Welcome')).toBeVisible();
});
```

### ❌ Testing Implementation Details

```typescript
// ❌ Don't test implementation details
test('implementation details anti-pattern', async ({ page }) => {
  await page.goto('/todo-app');

  // BAD: Testing internal CSS classes
  await expect(page.locator('.todo-item-component-v2')).toBeVisible();
  await expect(page.locator('.state-active')).toHaveCount(3);

  // BAD: Testing internal data attributes
  await expect(page.locator('[data-react-component="TodoItem"]')).toBeVisible();

  // BAD: Testing framework-specific attributes
  await expect(page.locator('[ng-repeat]')).toHaveCount(5);
});

// ✅ Test user-facing behavior
test('user behavior pattern', async ({ page }) => {
  await page.goto('/todo-app');

  // GOOD: Test what users see and do
  await expect(page.getByText('Buy groceries')).toBeVisible();
  await expect(page.getByRole('checkbox', { name: 'Buy groceries' })).not.toBeChecked();

  // GOOD: Test functional behavior
  await page.getByRole('checkbox', { name: 'Buy groceries' }).check();
  await expect(page.getByRole('checkbox', { name: 'Buy groceries' })).toBeChecked();

  // GOOD: Test user-visible counts
  await expect(page.getByText('3 items remaining')).toBeVisible();
});
```

### ❌ Vague Assertion Messages

```typescript
// ❌ Don't use vague assertion messages
test('vague messages anti-pattern', async ({ page }) => {
  await page.goto('/profile');

  // BAD: No context for failure
  await expect(page.locator('.profile-card')).toBeVisible();
  await expect(page.locator('input')).toHaveValue('John');
  await expect(page.locator('button')).toBeEnabled();
});

// ✅ Use descriptive assertion messages
test('descriptive messages pattern', async ({ page }) => {
  await page.goto('/profile');

  // GOOD: Clear context for debugging
  await expect(page.getByTestId('profile-card'), 'Profile card should be visible after page loads').toBeVisible();

  await expect(page.getByLabel('First Name'), 'First name field should display current user name').toHaveValue('John');

  await expect(
    page.getByRole('button', { name: 'Save Changes' }),
    'Save button should be enabled when form is valid',
  ).toBeEnabled();
});
```

## Error Handling Anti-Patterns

### ❌ Swallowing Assertion Errors

```typescript
// ❌ Don't catch and ignore assertion errors
test('swallowing errors anti-pattern', async ({ page }) => {
  await page.goto('/unreliable-page');

  try {
    // BAD: Catching assertion errors without proper handling
    await expect(page.getByText('Important Content')).toBeVisible();
  } catch (error) {
    console.log('Content not found, continuing...');
    // BAD: Ignoring the error continues the test in invalid state
  }

  // BAD: Silent fallback without validation
  try {
    await expect(page.getByTestId('primary-button')).toBeVisible();
  } catch {
    await expect(page.getByTestId('fallback-button')).toBeVisible();
    // What if both fail?
  }
});

// ✅ Handle errors with proper fallback validation
test('proper error handling pattern', async ({ page }) => {
  await page.goto('/unreliable-page');

  // GOOD: Explicit conditional logic
  const importantContent = page.getByText('Important Content');
  if (await importantContent.isVisible()) {
    await expect(importantContent).toBeVisible();
  } else {
    // GOOD: Validate expected alternative state
    await expect(page.getByText('Content loading...')).toBeVisible();
  }

  // GOOD: Validate one of multiple expected states
  const primaryButton = page.getByTestId('primary-button');
  const fallbackButton = page.getByTestId('fallback-button');

  const primaryVisible = await primaryButton.isVisible();
  const fallbackVisible = await fallbackButton.isVisible();

  expect(primaryVisible || fallbackVisible, 'Either primary or fallback button should be visible').toBe(true);

  if (primaryVisible) {
    await expect(primaryButton).toBeEnabled();
  } else {
    await expect(fallbackButton).toBeEnabled();
  }
});
```

### ❌ Generic Error Messages

```typescript
// ❌ Don't use generic error messages that don't help debugging
test('generic error messages anti-pattern', async ({ page }) => {
  await page.goto('/form');

  // BAD: Generic messages don't provide context
  await expect(page.locator('input')).toBeVisible(); // Which input?
  await expect(page.locator('button')).toBeEnabled(); // Which button?
  await expect(page.locator('.error')).not.toBeVisible(); // Which error?
});

// ✅ Use specific, actionable error messages
test('specific error messages pattern', async ({ page }) => {
  await page.goto('/form');

  // GOOD: Specific, actionable messages
  await expect(
    page.getByLabel('Email Address'),
    'Email input field should be visible in registration form',
  ).toBeVisible();

  await expect(
    page.getByRole('button', { name: 'Submit' }),
    'Submit button should be enabled after filling required fields',
  ).toBeEnabled();

  await expect(
    page.getByTestId('email-validation-error'),
    'Email validation error should not be visible for valid email',
  ).not.toBeVisible();
});
```

## Performance Anti-Patterns

### ❌ Redundant Locator Creation

```typescript
// ❌ Don't create the same locator multiple times
test('redundant locators anti-pattern', async ({ page }) => {
  await page.goto('/dashboard');

  // BAD: Creating same locator repeatedly
  await expect(page.getByTestId('user-card')).toBeVisible();
  await expect(page.getByTestId('user-card')).toHaveText('John Doe');
  await expect(page.getByTestId('user-card')).toHaveClass('active');
  await expect(page.getByTestId('user-card')).toHaveAttribute('data-user-id', '123');
});

// ✅ Reuse locators for efficiency
test('locator reuse pattern', async ({ page }) => {
  await page.goto('/dashboard');

  // GOOD: Create locator once, reuse multiple times
  const userCard = page.getByTestId('user-card');

  await expect(userCard).toBeVisible();
  await expect(userCard).toHaveText('John Doe');
  await expect(userCard).toHaveClass('active');
  await expect(userCard).toHaveAttribute('data-user-id', '123');
});
```

### ❌ Inefficient Assertion Ordering

```typescript
// ❌ Don't order assertions inefficiently
test('inefficient ordering anti-pattern', async ({ page }) => {
  await page.goto('/data-table');

  // BAD: Checking specific content before ensuring container exists
  await expect(page.getByText('John Doe')).toBeVisible();
  await expect(page.getByText('jane@example.com')).toBeVisible();
  await expect(page.getByTestId('data-table')).toBeVisible(); // Should be first

  // BAD: Expensive operations before quick checks
  await expect(page.getByTestId('complex-chart')).toHaveScreenshot('chart.png');
  await expect(page.getByTestId('loading-spinner')).not.toBeVisible(); // Quick check
});

// ✅ Order assertions logically for efficiency
test('efficient ordering pattern', async ({ page }) => {
  await page.goto('/data-table');

  // GOOD: Check container exists first
  await expect(page.getByTestId('data-table')).toBeVisible();

  // GOOD: Quick state checks before content validation
  await expect(page.getByTestId('loading-spinner')).not.toBeVisible();

  // GOOD: Then validate specific content
  await expect(page.getByText('John Doe')).toBeVisible();
  await expect(page.getByText('jane@example.com')).toBeVisible();

  // GOOD: Expensive operations last
  await expect(page.getByTestId('complex-chart')).toHaveScreenshot('chart.png');
});
```

## Maintainability Anti-Patterns

### ❌ Hardcoded Values Everywhere

```typescript
// ❌ Don't hardcode values throughout tests
test('hardcoded values anti-pattern', async ({ page }) => {
  await page.goto('/user/12345');

  // BAD: Hardcoded values make tests fragile
  await expect(page.getByText('John Doe')).toBeVisible();
  await expect(page.getByText('john.doe@company.com')).toBeVisible();
  await expect(page.getByText('Administrator')).toBeVisible();
  await expect(page.getByText('Active since: 2023-01-15')).toBeVisible();
});

// ✅ Use parameterized, reusable data
test('parameterized data pattern', async ({ page }) => {
  const testUser = {
    id: '12345',
    name: 'John Doe',
    email: 'john.doe@company.com',
    role: 'Administrator',
    activeDate: '2023-01-15',
  };

  await page.goto(`/user/${testUser.id}`);

  // GOOD: Parameterized assertions
  await expect(page.getByText(testUser.name)).toBeVisible();
  await expect(page.getByText(testUser.email)).toBeVisible();
  await expect(page.getByText(testUser.role)).toBeVisible();
  await expect(page.getByText(`Active since: ${testUser.activeDate}`)).toBeVisible();
});
```

### ❌ Copy-Paste Assertion Blocks

```typescript
// ❌ Don't copy-paste assertion blocks
test('copy-paste anti-pattern', async ({ page }) => {
  await page.goto('/users');

  // BAD: Repeated assertion patterns
  const firstRow = page.locator('tr').nth(1);
  await expect(firstRow.locator('td').nth(0)).toBeVisible();
  await expect(firstRow.locator('td').nth(1)).toBeVisible();
  await expect(firstRow.locator('td').nth(2)).toBeVisible();

  const secondRow = page.locator('tr').nth(2);
  await expect(secondRow.locator('td').nth(0)).toBeVisible();
  await expect(secondRow.locator('td').nth(1)).toBeVisible();
  await expect(secondRow.locator('td').nth(2)).toBeVisible();
});

// ✅ Extract reusable assertion functions
async function validateTableRow(page: Page, rowIndex: number): Promise<void> {
  const row = page.locator('tr').nth(rowIndex);

  // Validate all cells in the row
  const cells = row.locator('td');
  const cellCount = await cells.count();

  for (let i = 0; i < cellCount; i++) {
    await expect(cells.nth(i)).toBeVisible();
  }
}

test('reusable assertions pattern', async ({ page }) => {
  await page.goto('/users');

  // GOOD: Reusable function for common patterns
  await validateTableRow(page, 1);
  await validateTableRow(page, 2);

  // Or even better: validate all rows
  const rowCount = await page.locator('tbody tr').count();
  for (let i = 1; i <= rowCount; i++) {
    await validateTableRow(page, i);
  }
});
```

## Test Organization Anti-Patterns

### ❌ Giant Test Cases

```typescript
// ❌ Don't create massive test cases that test everything
test('giant test anti-pattern', async ({ page }) => {
  // BAD: One test doing too many things
  await page.goto('/');
  await expect(page.getByText('Welcome')).toBeVisible();

  await page.getByRole('link', { name: 'Login' }).click();
  await expect(page).toHaveURL('/login');
  await expect(page.getByRole('form')).toBeVisible();

  await page.getByLabel('Username').fill('user@test.com');
  await page.getByLabel('Password').fill('password');
  await page.getByRole('button', { name: 'Login' }).click();

  await expect(page).toHaveURL('/dashboard');
  await expect(page.getByText('Dashboard')).toBeVisible();

  await page.getByRole('link', { name: 'Users' }).click();
  await expect(page).toHaveURL('/users');
  await expect(page.getByTestId('users-table')).toBeVisible();

  await page.getByRole('button', { name: 'Add User' }).click();
  await expect(page.getByRole('dialog')).toBeVisible();

  // ... continues for 50+ more lines
});

// ✅ Break into focused, single-purpose tests
test.describe('User Management Workflow', () => {
  test('should display welcome page correctly', async ({ page }) => {
    await page.goto('/');
    await expect(page.getByText('Welcome')).toBeVisible();
    await expect(page.getByRole('link', { name: 'Login' })).toBeVisible();
  });

  test('should allow user login', async ({ page }) => {
    const loginPage = new LoginPage(page);

    await loginPage.goto();
    await loginPage.login('user@test.com', 'password');

    await expect(page).toHaveURL('/dashboard');
    await expect(page.getByText('Dashboard')).toBeVisible();
  });

  test('should navigate to users page', async ({ page }) => {
    // Assume user is logged in via setup
    await page.goto('/dashboard');

    await page.getByRole('link', { name: 'Users' }).click();

    await expect(page).toHaveURL('/users');
    await expect(page.getByTestId('users-table')).toBeVisible();
  });
});
```

## Debugging Anti-Patterns

### ❌ Poor Debug Information

```typescript
// ❌ Don't leave assertions without context for debugging
test('poor debug info anti-pattern', async ({ page }) => {
  await page.goto('/complex-page');

  // BAD: No context when assertion fails
  await expect(page.locator('.content')).toBeVisible();
  await expect(page.locator('button')).toBeEnabled();
  await expect(page.locator('input')).toHaveValue('expected');
});

// ✅ Provide rich context for debugging
test('rich debug info pattern', async ({ page }) => {
  await page.goto('/complex-page');

  await test.step('Validate page structure', async () => {
    await expect(page.getByTestId('main-content'), 'Main content area should be visible after page load').toBeVisible();

    await expect(
      page.getByRole('button', { name: 'Submit' }),
      'Submit button should be enabled when form is valid',
    ).toBeEnabled();

    await expect(page.getByLabel('Search Query'), 'Search input should contain the expected default value').toHaveValue(
      'expected',
    );
  });

  // Add debug information on failure
  page.on('pageerror', (error) => {
    console.log('Page error occurred:', error.message);
  });

  // Capture state on assertion failure
  try {
    await expect(page.getByTestId('critical-element')).toBeVisible();
  } catch (error) {
    console.log('Critical element assertion failed');
    console.log('Current URL:', await page.url());
    console.log('Page title:', await page.title());

    // Take screenshot for debugging
    await page.screenshot({ path: 'debug-critical-element-failure.png' });

    throw error;
  }
});
```

### ❌ Not Using Test Steps

```typescript
// ❌ Don't skip test steps for complex workflows
test('no test steps anti-pattern', async ({ page }) => {
  await page.goto('/checkout');

  // BAD: All assertions at same level, hard to debug which step failed
  await expect(page.getByTestId('cart-summary')).toBeVisible();
  await page.getByLabel('First Name').fill('John');
  await page.getByLabel('Last Name').fill('Doe');
  await expect(page.getByRole('button', { name: 'Continue' })).toBeEnabled();
  await page.getByRole('button', { name: 'Continue' }).click();
  await expect(page.getByText('Payment')).toBeVisible();
  await page.getByLabel('Card Number').fill('4111111111111111');
  await expect(page.getByRole('button', { name: 'Place Order' })).toBeEnabled();
});

// ✅ Use test steps for clear workflow organization
test('test steps pattern', async ({ page }) => {
  await page.goto('/checkout');

  await test.step('Validate checkout page initialization', async () => {
    await expect(page.getByTestId('cart-summary')).toBeVisible();
    await expect(page.getByText('Shipping Information')).toBeVisible();
  });

  await test.step('Fill shipping information', async () => {
    await page.getByLabel('First Name').fill('John');
    await page.getByLabel('Last Name').fill('Doe');

    await expect(page.getByRole('button', { name: 'Continue' })).toBeEnabled();
  });

  await test.step('Proceed to payment', async () => {
    await page.getByRole('button', { name: 'Continue' }).click();

    await expect(page.getByText('Payment')).toBeVisible();
    await expect(page.getByLabel('Card Number')).toBeVisible();
  });

  await test.step('Fill payment information', async () => {
    await page.getByLabel('Card Number').fill('4111111111111111');

    await expect(page.getByRole('button', { name: 'Place Order' })).toBeEnabled();
  });
});
```

By avoiding these anti-patterns and following the recommended alternatives, your Playwright assertions will be more reliable, maintainable, and provide better debugging information when tests fail.
