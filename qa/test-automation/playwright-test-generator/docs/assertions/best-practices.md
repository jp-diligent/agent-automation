# Assertion Best Practices

**Description**: Comprehensive guide to writing effective, maintainable, and reliable assertions in Playwright tests. This document covers fundamental principles, common patterns, and proven strategies for creating robust test validation.

## Table of Contents

- [Core Principles](#core-principles)
- [Assertion Structure](#assertion-structure)
- [Locator Best Practices](#locator-best-practices)
- [Timing and Waiting](#timing-and-waiting)
- [Error Messages](#error-messages)
- [Test Organization](#test-organization)
- [Performance Considerations](#performance-considerations)
- [Debugging and Maintenance](#debugging-and-maintenance)

## Core Principles

### 1. Clear Intent and Purpose

Every assertion should clearly communicate what is being validated:

```typescript
// ✅ Good - Clear what's being validated
await expect(page.getByRole('heading', { name: 'User Profile' })).toBeVisible();
await expect(page.getByText('Profile updated successfully')).toBeVisible();
await expect(page.getByRole('button', { name: 'Save Changes' })).toBeDisabled();

// ❌ Bad - Unclear intent
await expect(page.locator('h1')).toBeVisible();
await expect(page.locator('.message')).toBeVisible();
await expect(page.locator('button')).toHaveAttribute('disabled');
```

### 2. Comprehensive Validation

Validate complete user scenarios, not just individual elements:

```typescript
// ✅ Good - Complete workflow validation
test('user login success', async ({ page }) => {
  const loginPage = new LoginPage(page);
  const dashboardPage = new DashboardPage(page);

  await loginPage.login('validuser', 'validpass');

  // Validate complete success state
  await expect(page).toHaveURL('/dashboard');
  await expect(dashboardPage.welcomeMessage).toBeVisible();
  await expect(dashboardPage.welcomeMessage).toHaveText('Welcome, Valid User');
  await expect(dashboardPage.navigationMenu).toBeVisible();
  await expect(page.getByText('Login successful')).not.toBeVisible(); // Temporary message gone
});

// ❌ Bad - Incomplete validation
test('user login', async ({ page }) => {
  await page.goto('/login');
  await page.fill('[name="username"]', 'user');
  await page.fill('[name="password"]', 'pass');
  await page.click('button[type="submit"]');

  // Only checks URL, misses user experience validation
  await expect(page).toHaveURL('/dashboard');
});
```

### 3. Robust and Reliable

Use Playwright's auto-waiting capabilities and avoid brittle patterns:

```typescript
// ✅ Good - Leverages auto-waiting
await expect(page.getByRole('button', { name: 'Submit' })).toBeEnabled();
await expect(page.getByText('Processing...')).toBeVisible();
await expect(page.getByText('Operation completed')).toBeVisible();
await expect(page.getByText('Processing...')).not.toBeVisible();

// ❌ Bad - Manual timing that can fail
await page.waitForTimeout(2000);
expect(await page.locator('button').isEnabled()).toBe(true);
```

## Assertion Structure

### Grouping Related Assertions

Group assertions logically using test steps:

```typescript
test('order submission workflow', async ({ page }) => {
  const orderPage = new OrderPage(page);

  await test.step('Fill order details', async () => {
    await orderPage.fillOrderForm({
      product: 'Widget A',
      quantity: 5,
      customerEmail: 'test@example.com',
    });

    // Validate form state
    await expect(orderPage.productField).toHaveValue('Widget A');
    await expect(orderPage.quantityField).toHaveValue('5');
    await expect(orderPage.submitButton).toBeEnabled();
  });

  await test.step('Submit order', async () => {
    await orderPage.submitOrder();

    // Validate submission state
    await expect(page.getByText('Order submitted successfully')).toBeVisible();
    await expect(page.getByText('Order #')).toBeVisible();
    await expect(orderPage.submitButton).toBeDisabled();
  });

  await test.step('Verify order confirmation', async () => {
    // Validate complete success state
    await expect(page).toHaveURL(/\/orders\/\d+/);
    await expect(page.getByText('Widget A')).toBeVisible();
    await expect(page.getByText('Quantity: 5')).toBeVisible();
    await expect(page.getByText('test@example.com')).toBeVisible();
  });
});
```

### Assertion Hierarchy

Structure assertions from general to specific:

```typescript
// ✅ Good - General to specific validation
await test.step('Validate search results', async () => {
  // 1. General page state
  await expect(page).toHaveURL(/\/search/);
  await expect(page.getByRole('main')).toBeVisible();

  // 2. Results container
  await expect(page.getByTestId('search-results')).toBeVisible();
  await expect(page.getByText('Found 15 results')).toBeVisible();

  // 3. Specific result items
  await expect(page.getByTestId('result-item').first()).toBeVisible();
  await expect(page.getByTestId('result-item')).toHaveCount(15);

  // 4. Individual result content
  const firstResult = page.getByTestId('result-item').first();
  await expect(firstResult.getByRole('heading')).toBeVisible();
  await expect(firstResult.getByText(/Updated:/)).toBeVisible();
});
```

## Locator Best Practices

### Semantic Locators in Assertions

Prefer semantic locators for better assertion readability:

```typescript
// ✅ Good - Semantic locators
await expect(page.getByRole('button', { name: 'Save Changes' })).toBeEnabled();
await expect(page.getByRole('alert')).toHaveText('Changes saved successfully');
await expect(page.getByLabel('Email Address')).toHaveValue('user@example.com');
await expect(page.getByRole('table')).toBeVisible();

// ❌ Bad - Generic selectors
await expect(page.locator('button.save-btn')).toBeEnabled();
await expect(page.locator('.alert-success')).toHaveText('Changes saved successfully');
await expect(page.locator('input[type="email"]')).toHaveValue('user@example.com');
await expect(page.locator('table')).toBeVisible();
```

### Precise Targeting

Use specific locators to avoid false positives:

```typescript
// ✅ Good - Precise targeting
await expect(page.getByRole('button', { name: 'Delete User' })).toBeVisible();
await expect(page.getByText('User "John Doe" has been deleted')).toBeVisible();
await expect(page.getByRole('row').filter({ hasText: 'John Doe' })).not.toBeVisible();

// ❌ Bad - Ambiguous targeting
await expect(page.getByText('Delete')).toBeVisible(); // Could match multiple elements
await expect(page.getByText('deleted')).toBeVisible(); // Too generic
await expect(page.locator('tr:has-text("John")')).not.toBeVisible(); // XPath alternative is better
```

## Timing and Waiting

### Leverage Auto-Waiting

Playwright's assertions include auto-waiting - use it effectively:

```typescript
// ✅ Good - Leverages auto-waiting
await expect(page.getByRole('button', { name: 'Submit' })).toBeEnabled();
await expect(page.getByText('Loading...')).toBeVisible();
await expect(page.getByText('Data loaded')).toBeVisible();
await expect(page.getByText('Loading...')).not.toBeVisible();

// ❌ Bad - Unnecessary manual waiting
await page.waitForSelector('button:not([disabled])');
expect(await page.locator('button').isEnabled()).toBe(true);

await page.waitForTimeout(1000);
await expect(page.getByText('Data loaded')).toBeVisible();
```

### Custom Timeouts for Slow Operations

Use custom timeouts only when necessary:

```typescript
// ✅ Good - Custom timeout for known slow operation
await expect(page.getByText('Report generated successfully')).toBeVisible({
  timeout: 30000, // Report generation is known to be slow
});

// ✅ Good - Custom timeout with context
await test.step('Wait for large file upload', async () => {
  await expect(page.getByText('Upload completed')).toBeVisible({
    timeout: 60000,
  });
});

// ❌ Bad - Arbitrary timeout without justification
await expect(page.getByText('Welcome')).toBeVisible({ timeout: 10000 });
```

## Error Messages

### Descriptive Custom Messages

Provide context when assertion failures need explanation:

```typescript
// ✅ Good - Descriptive error messages
await expect(
  page.getByRole('button', { name: 'Submit' }),
  'Submit button should be enabled after filling required fields',
).toBeEnabled();

await expect(page.getByTestId('user-count'), 'User count should reflect the newly added user').toHaveText(
  'Total Users: 11',
);

// ✅ Good - Context in test steps
await test.step('Verify user was added to the system', async () => {
  await expect(page.getByRole('cell', { name: 'john.doe@example.com' })).toBeVisible();
});

// ❌ Bad - Generic error messages don't help debugging
await expect(page.locator('button')).toBeEnabled();
await expect(page.locator('.count')).toHaveText('11');
```

### Assertion Failure Context

Structure tests to provide context when assertions fail:

```typescript
test('user profile update', async ({ page }) => {
  const userPage = new UserPage(page);

  await test.step('Navigate to user profile', async () => {
    await userPage.goto();
    await expect(page).toHaveURL('/profile');
  });

  await test.step('Update user information', async () => {
    await userPage.updateProfile({
      name: 'Updated Name',
      email: 'updated@example.com',
    });

    // Clear context for what should happen
    await expect(
      page.getByText('Profile updated successfully'),
      'Success message should appear after profile update',
    ).toBeVisible();
  });

  await test.step('Verify changes are persisted', async () => {
    await page.reload();

    // Context for why this assertion matters
    await expect(page.getByDisplayValue('Updated Name'), 'Updated name should persist after page reload').toBeVisible();
    await expect(
      page.getByDisplayValue('updated@example.com'),
      'Updated email should persist after page reload',
    ).toBeVisible();
  });
});
```

## Test Organization

### Logical Grouping

Group assertions by their purpose and relationship:

```typescript
test('shopping cart functionality', async ({ page }) => {
  const cartPage = new CartPage(page);

  await test.step('Add items to cart', async () => {
    await cartPage.addItem('Product A', 2);
    await cartPage.addItem('Product B', 1);

    // Validate cart state after additions
    await expect(cartPage.itemCount).toHaveText('3 items');
    await expect(cartPage.subtotal).toHaveText('$75.00');
  });

  await test.step('Apply discount code', async () => {
    await cartPage.applyDiscountCode('SAVE10');

    // Validate discount application
    await expect(page.getByText('Discount applied')).toBeVisible();
    await expect(cartPage.discountAmount).toHaveText('-$7.50');
    await expect(cartPage.total).toHaveText('$67.50');
  });

  await test.step('Proceed to checkout', async () => {
    await cartPage.proceedToCheckout();

    // Validate checkout page state
    await expect(page).toHaveURL('/checkout');
    await expect(page.getByText('Order Summary')).toBeVisible();
    await expect(page.getByText('Total: $67.50')).toBeVisible();
  });
});
```

### Negative Assertions

Include negative assertions to validate what shouldn't be present:

```typescript
test('user authentication flow', async ({ page }) => {
  const loginPage = new LoginPage(page);

  await test.step('Invalid login attempt', async () => {
    await loginPage.login('user', 'wrongpassword');

    // Positive assertions - what should be present
    await expect(page.getByRole('alert')).toBeVisible();
    await expect(page.getByText('Invalid credentials')).toBeVisible();

    // Negative assertions - what should NOT be present
    await expect(page).not.toHaveURL('/dashboard');
    await expect(page.getByText('Welcome')).not.toBeVisible();
    await expect(page.getByRole('button', { name: 'Logout' })).not.toBeVisible();
  });
});
```

## Performance Considerations

### Efficient Assertion Patterns

Optimize assertion performance without sacrificing reliability:

```typescript
// ✅ Good - Efficient single locator with multiple assertions
const submitButton = page.getByRole('button', { name: 'Submit' });
await expect(submitButton).toBeVisible();
await expect(submitButton).toBeEnabled();
await expect(submitButton).toHaveText('Submit Order');

// ❌ Less efficient - Multiple locator lookups
await expect(page.getByRole('button', { name: 'Submit' })).toBeVisible();
await expect(page.getByRole('button', { name: 'Submit' })).toBeEnabled();
await expect(page.getByRole('button', { name: 'Submit' })).toHaveText('Submit Order');
```

### Batch Related Validations

Group related assertions for better performance:

```typescript
// ✅ Good - Batch validation of form state
await test.step('Validate form completion', async () => {
  const form = page.getByRole('form', { name: 'User Registration' });

  // Validate form container first
  await expect(form).toBeVisible();

  // Batch field validations
  await expect(form.getByLabel('First Name')).toHaveValue('John');
  await expect(form.getByLabel('Last Name')).toHaveValue('Doe');
  await expect(form.getByLabel('Email')).toHaveValue('john@example.com');
  await expect(form.getByRole('button', { name: 'Register' })).toBeEnabled();
});
```

## Debugging and Maintenance

### Assertion Debugging

Structure assertions to make debugging easier:

```typescript
test('complex data validation', async ({ page }) => {
  await test.step('Load user data', async () => {
    await page.goto('/users/123');

    // Validate page loaded correctly first
    await expect(page).toHaveURL('/users/123');
    await expect(page.getByRole('main')).toBeVisible();
  });

  await test.step('Validate user information display', async () => {
    const userCard = page.getByTestId('user-card');

    // Validate container exists before checking contents
    await expect(userCard).toBeVisible();

    // Then validate specific content
    await expect(userCard.getByRole('heading')).toHaveText('John Doe');
    await expect(userCard.getByText('john.doe@example.com')).toBeVisible();
    await expect(userCard.getByText('Member since: 2023')).toBeVisible();
  });
});
```

### Maintainable Assertion Patterns

Create patterns that are easy to maintain and update:

```typescript
// ✅ Good - Maintainable validation helper
class ValidationHelper {
  static async validateUserCard(page: Page, userData: { name: string; email: string; memberSince: string }) {
    const userCard = page.getByTestId('user-card');

    await expect(userCard).toBeVisible();
    await expect(userCard.getByRole('heading')).toHaveText(userData.name);
    await expect(userCard.getByText(userData.email)).toBeVisible();
    await expect(userCard.getByText(`Member since: ${userData.memberSince}`)).toBeVisible();
  }
}

// Usage in test
await ValidationHelper.validateUserCard(page, {
  name: 'John Doe',
  email: 'john.doe@example.com',
  memberSince: '2023',
});
```

## Quick Checklist

### Pre-Assertion Checklist

- [ ] **Clear Intent**: Does the assertion clearly communicate what's being validated?
- [ ] **Semantic Locators**: Using role-based or label-based selectors when possible?
- [ ] **Auto-Waiting**: Leveraging Playwright's built-in waiting instead of manual timeouts?
- [ ] **Comprehensive**: Validating complete user scenarios, not just individual elements?
- [ ] **Robust**: Will this assertion be reliable across different test runs?

### Post-Assertion Review

- [ ] **Error Messages**: Will assertion failures provide clear debugging information?
- [ ] **Performance**: Are locators being reused efficiently?
- [ ] **Maintenance**: Can this assertion be easily updated if the UI changes?
- [ ] **Coverage**: Are both positive and negative scenarios covered?
- [ ] **Context**: Is the assertion grouped logically with related validations?

## Common Anti-Patterns to Avoid

### ❌ Brittle Timing Patterns

```typescript
// Don't do this
await page.waitForTimeout(1000);
expect(await page.locator('button').isEnabled()).toBe(true);
```

### ❌ Vague Error Messages

```typescript
// Don't do this
await expect(page.locator('.message')).toBeVisible();
```

### ❌ Over-Generic Selectors

```typescript
// Don't do this
await expect(page.locator('div')).toContainText('success');
```

### ❌ Incomplete Validation

```typescript
// Don't do this - only checks URL, misses UX validation
await expect(page).toHaveURL('/dashboard');
```

By following these best practices, your Playwright assertions will be more reliable, maintainable, and provide better debugging information when tests fail.
