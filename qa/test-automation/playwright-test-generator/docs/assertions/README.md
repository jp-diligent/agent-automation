# Assertions Documentation

**Description**: Comprehensive guide to writing effective, maintainable, and reliable assertions in Playwright tests. This documentation covers assertion strategies, patterns, and best practices for creating robust test validation.

## 📚 Documentation Structure

### Core Guides

- **[Best Practices](best-practices.md)** - Fundamental principles for writing effective assertions
- **[Assertion Types](assertion-types.md)** - Complete reference of Playwright assertion methods
- **[Patterns & Strategies](patterns-strategies.md)** - Common assertion patterns and when to use them
- **[Error Handling](error-handling.md)** - Managing assertion failures and debugging techniques

### Advanced Topics

- **[Custom Assertions](custom-assertions.md)** - Creating reusable custom assertion methods
- **[Performance Considerations](performance.md)** - Optimizing assertion performance and reliability
- **[Anti-Patterns](anti-patterns.md)** - Common mistakes and how to avoid them

### Quick References

- **[Assertion Checklist](../quick-references/assertion-checklist.md)** - Quick validation checklist
- **[Common Examples](../quick-references/assertion-examples.md)** - Frequently used assertion patterns

## 🎯 Core Principles

### 1. Clear and Descriptive

Assertions should clearly communicate what is being validated:

```typescript
// ✅ Good - Clear intent
await expect(page.getByRole('heading', { name: 'Dashboard' })).toBeVisible();
await expect(page.getByText('Welcome, John Doe')).toBeVisible();

// ❌ Bad - Unclear what's being validated
await expect(page.locator('h1')).toBeVisible();
await expect(page.locator('.user-info')).toContainText('John');
```

### 2. Robust and Reliable

Use Playwright's auto-waiting capabilities and avoid brittle assertions:

```typescript
// ✅ Good - Auto-waiting with clear conditions
await expect(page.getByRole('button', { name: 'Submit' })).toBeEnabled();
await expect(page.getByText('Data loaded successfully')).toBeVisible();

// ❌ Bad - Race conditions and timing issues
await page.waitForTimeout(1000);
expect(await page.locator('button').isEnabled()).toBe(true);
```

### 3. Comprehensive Coverage

Validate both positive and negative scenarios:

```typescript
// ✅ Good - Comprehensive validation
await expect(page.getByRole('alert')).toBeVisible();
await expect(page.getByRole('alert')).toHaveText('Invalid credentials');
await expect(page.getByRole('button', { name: 'Submit' })).toBeDisabled();

// ❌ Bad - Incomplete validation
await expect(page.getByRole('alert')).toBeVisible();
```

## 🚀 Quick Start

### Basic Assertion Structure

```typescript
import { test, expect } from '@playwright/test';

test('user login validation', async ({ page }) => {
  // Navigate and perform actions
  await page.goto('/login');
  await page.getByLabel('Username').fill('testuser');
  await page.getByLabel('Password').fill('wrongpassword');
  await page.getByRole('button', { name: 'Login' }).click();

  // Assertions - validate the expected outcome
  await expect(page.getByRole('alert')).toBeVisible();
  await expect(page.getByRole('alert')).toHaveText('Invalid username or password');
  await expect(page).toHaveURL('/login'); // Still on login page
});
```

### Using Page Objects with Assertions

```typescript
test('dashboard content validation', async ({ page }) => {
  const loginPage = new LoginPage(page);
  const dashboardPage = new DashboardPage(page);

  // Actions through page objects
  await loginPage.goto();
  await loginPage.login('validuser', 'validpass');

  // Assertions - validate page object states
  await expect(dashboardPage.welcomeMessage).toBeVisible();
  await expect(dashboardPage.welcomeMessage).toHaveText('Welcome, Valid User');
  await expect(dashboardPage.searchButton).toBeEnabled();
});
```

## 🔧 Integration with Testing Workflow

### Test Structure with Assertions

```typescript
test.describe('User Management', () => {
  test('create user successfully', async ({ page }) => {
    // Arrange
    const userPage = new UserPage(page);
    const userData = { name: 'John Doe', email: 'john@example.com' };

    // Act
    await userPage.goto();
    await userPage.createUser(userData);

    // Assert - Multiple validation layers
    await expect(page.getByText('User created successfully')).toBeVisible();
    await expect(userPage.userTable).toContainText(userData.name);
    await expect(userPage.userTable).toContainText(userData.email);
    await expect(userPage.createButton).toBeEnabled(); // Ready for next action
  });
});
```

## 🎯 Best Practices Summary

1. **Use semantic locators** in assertions for better readability
2. **Leverage auto-waiting** - avoid manual waits before assertions
3. **Write descriptive test messages** that explain failures clearly
4. **Group related assertions** logically within test steps
5. **Validate complete user workflows** not just individual elements
6. **Handle async operations** properly with await
7. **Use appropriate assertion methods** for different validation types
8. **Create custom assertions** for complex business logic validation

## 📖 Further Reading

For detailed information on specific topics, refer to the individual documentation files in this folder. Each file provides comprehensive examples, patterns, and guidelines for its respective area.

---

**Next Steps**: Start with [Best Practices](best-practices.md) for fundamental assertion principles, then explore [Assertion Types](assertion-types.md) for a complete reference of available assertion methods.
