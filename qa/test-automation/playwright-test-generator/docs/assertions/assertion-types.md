# Assertion Types Reference

**Description**: Complete reference guide for all Playwright assertion methods, including syntax, use cases, and practical examples. This guide serves as a comprehensive lookup for choosing the right assertion method for different validation scenarios.

## Table of Contents

- [Element State Assertions](#element-state-assertions)
- [Text Content Assertions](#text-content-assertions)
- [Attribute Assertions](#attribute-assertions)
- [Value Assertions](#value-assertions)
- [Page State Assertions](#page-state-assertions)
- [Collection Assertions](#collection-assertions)
- [Screenshot Assertions](#screenshot-assertions)
- [Custom Assertion Patterns](#custom-assertion-patterns)

## Element State Assertions

### Visibility Assertions

#### `toBeVisible()`

Validates that an element is visible to the user.

```typescript
// ✅ Common usage patterns
await expect(page.getByRole('button', { name: 'Submit' })).toBeVisible();
await expect(page.getByText('Welcome back!')).toBeVisible();
await expect(page.getByTestId('loading-spinner')).toBeVisible();

// ✅ With custom timeout for slow-loading elements
await expect(page.getByText('Report generated')).toBeVisible({ timeout: 30000 });

// ✅ In combination with page object
await expect(profilePage.welcomeMessage).toBeVisible();
```

#### `not.toBeVisible()`

Validates that an element is not visible or doesn't exist.

```typescript
// ✅ Validating element disappearance
await expect(page.getByText('Loading...')).not.toBeVisible();
await expect(page.getByRole('dialog')).not.toBeVisible();
await expect(page.getByTestId('error-banner')).not.toBeVisible();

// ✅ Validating conditional content
await expect(page.getByText('Admin Panel')).not.toBeVisible(); // For non-admin users
```

#### `toBeHidden()`

Validates that an element exists in DOM but is not visible.

```typescript
// ✅ Validating hidden but present elements
await expect(page.getByTestId('collapsible-content')).toBeHidden();
await expect(page.getByRole('menu')).toBeHidden(); // Dropdown menu closed

// ✅ Useful for elements that toggle visibility
const sidebar = page.getByTestId('sidebar');
await expect(sidebar).toBeHidden();
await page.getByRole('button', { name: 'Toggle Sidebar' }).click();
await expect(sidebar).toBeVisible();
```

### Interaction State Assertions

#### `toBeEnabled()`

Validates that an interactive element can be interacted with.

```typescript
// ✅ Form validation
await expect(page.getByRole('button', { name: 'Submit' })).toBeEnabled();
await expect(page.getByLabel('Email')).toBeEnabled();
await expect(page.getByRole('checkbox', { name: 'Terms' })).toBeEnabled();

// ✅ Dynamic state validation
const saveButton = page.getByRole('button', { name: 'Save' });
await expect(saveButton).toBeDisabled(); // Initially disabled
await page.getByLabel('Name').fill('John');
await expect(saveButton).toBeEnabled(); // Enabled after input
```

#### `toBeDisabled()`

Validates that an interactive element cannot be interacted with.

```typescript
// ✅ Form validation states
await expect(page.getByRole('button', { name: 'Submit' })).toBeDisabled();
await expect(page.getByLabel('Readonly Field')).toBeDisabled();

// ✅ Conditional disabling
await expect(page.getByRole('button', { name: 'Delete' })).toBeDisabled(); // No selection
await page.getByRole('checkbox').first().check();
await expect(page.getByRole('button', { name: 'Delete' })).toBeEnabled(); // After selection
```

#### `toBeEditable()`

Validates that an input element can be edited.

```typescript
// ✅ Input field validation
await expect(page.getByLabel('Username')).toBeEditable();
await expect(page.getByRole('textbox', { name: 'Description' })).toBeEditable();

// ✅ Content editable elements
await expect(page.getByTestId('rich-editor')).toBeEditable();
```

#### `toBeChecked()`

Validates checkbox or radio button selection state.

```typescript
// ✅ Checkbox validation
await expect(page.getByRole('checkbox', { name: 'Subscribe' })).toBeChecked();
await expect(page.getByRole('checkbox', { name: 'Terms' })).not.toBeChecked();

// ✅ Radio button validation
await expect(page.getByRole('radio', { name: 'Premium' })).toBeChecked();
await expect(page.getByRole('radio', { name: 'Basic' })).not.toBeChecked();

// ✅ Dynamic state changes
const agreeCheckbox = page.getByRole('checkbox', { name: 'I agree' });
await expect(agreeCheckbox).not.toBeChecked();
await agreeCheckbox.check();
await expect(agreeCheckbox).toBeChecked();
```

#### `toBeFocused()`

Validates that an element has keyboard focus.

```typescript
// ✅ Focus validation
await expect(page.getByLabel('Search')).toBeFocused();

// ✅ Tab navigation validation
await page.keyboard.press('Tab');
await expect(page.getByRole('button', { name: 'Next' })).toBeFocused();

// ✅ Focus after interaction
await page.getByLabel('Username').click();
await expect(page.getByLabel('Username')).toBeFocused();
```

## Text Content Assertions

### `toHaveText()`

Validates exact text content of an element.

```typescript
// ✅ Exact text matching
await expect(page.getByRole('heading')).toHaveText('User Profile');
await expect(page.getByTestId('counter')).toHaveText('5');
await expect(page.getByText('Status:')).toHaveText('Status: Active');

// ✅ With array for multiple elements
await expect(page.getByTestId('menu-item')).toHaveText(['Home', 'Profile', 'Settings', 'Logout']);

// ✅ Using regular expressions
await expect(page.getByTestId('timestamp')).toHaveText(/\d{4}-\d{2}-\d{2}/);
```

### `toContainText()`

Validates that element contains specific text (partial matching).

```typescript
// ✅ Partial text matching
await expect(page.getByRole('alert')).toContainText('successfully');
await expect(page.getByTestId('description')).toContainText('updated');

// ✅ Case-insensitive matching
await expect(page.getByTestId('status')).toContainText('ACTIVE', {
  ignoreCase: true,
});

// ✅ Multiple text patterns
await expect(page.getByTestId('summary')).toContainText(['users', 'total']);
```

### `toHaveValue()`

Validates input element values.

```typescript
// ✅ Input field values
await expect(page.getByLabel('Username')).toHaveValue('john.doe');
await expect(page.getByLabel('Age')).toHaveValue('25');
await expect(page.getByRole('textbox', { name: 'Email' })).toHaveValue('john@example.com');

// ✅ Select element values
await expect(page.getByLabel('Country')).toHaveValue('US');

// ✅ After form filling
await page.getByLabel('First Name').fill('John');
await expect(page.getByLabel('First Name')).toHaveValue('John');
```

## Attribute Assertions

### `toHaveAttribute()`

Validates element attributes and their values.

```typescript
// ✅ Attribute existence and value
await expect(page.getByRole('link')).toHaveAttribute('href', '/profile');
await expect(page.getByTestId('image')).toHaveAttribute('alt', 'User avatar');
await expect(page.getByRole('button')).toHaveAttribute('type', 'submit');

// ✅ Attribute existence only
await expect(page.getByLabel('Required Field')).toHaveAttribute('required');
await expect(page.getByTestId('external-link')).toHaveAttribute('target');

// ✅ Dynamic attributes
await expect(page.getByRole('button', { name: 'Toggle' })).toHaveAttribute('aria-expanded', 'false');
await page.getByRole('button', { name: 'Toggle' }).click();
await expect(page.getByRole('button', { name: 'Toggle' })).toHaveAttribute('aria-expanded', 'true');
```

### `toHaveClass()`

Validates CSS classes on elements.

```typescript
// ✅ Single class validation
await expect(page.getByRole('button')).toHaveClass('btn-primary');
await expect(page.getByTestId('status')).toHaveClass('active');

// ✅ Multiple classes (all must be present)
await expect(page.getByRole('button')).toHaveClass(['btn', 'btn-large', 'btn-primary']);

// ✅ Class pattern matching
await expect(page.getByTestId('notification')).toHaveClass(/alert-/);

// ✅ State-dependent classes
const toggleButton = page.getByRole('button', { name: 'Toggle' });
await expect(toggleButton).toHaveClass('inactive');
await toggleButton.click();
await expect(toggleButton).toHaveClass('active');
```

### `toHaveCSS()`

Validates computed CSS properties.

```typescript
// ✅ Color validation
await expect(page.getByRole('button')).toHaveCSS('background-color', 'rgb(0, 123, 255)');
await expect(page.getByTestId('error-text')).toHaveCSS('color', 'rgb(220, 53, 69)');

// ✅ Layout properties
await expect(page.getByTestId('sidebar')).toHaveCSS('width', '300px');
await expect(page.getByRole('main')).toHaveCSS('display', 'flex');

// ✅ Responsive design validation
await page.setViewportSize({ width: 768, height: 1024 });
await expect(page.getByTestId('mobile-menu')).toHaveCSS('display', 'block');
```

## Page State Assertions

### `toHaveURL()`

Validates current page URL.

```typescript
// ✅ Exact URL matching
await expect(page).toHaveURL('https://example.com/profile');
await expect(page).toHaveURL('/users/profile');

// ✅ Pattern matching
await expect(page).toHaveURL(/\/users\/\d+/);
await expect(page).toHaveURL(/profile$/);

// ✅ After navigation
await page.getByRole('link', { name: 'Settings' }).click();
await expect(page).toHaveURL('/settings');
```

### `toHaveTitle()`

Validates page title.

````typescript
```typescript
// ✅ Exact title matching
await expect(page).toHaveTitle('User Profile - MyApp');
await expect(page).toHaveTitle('Login');

// ✅ Pattern matching
await expect(page).toHaveTitle(/Profile/);
await expect(page).toHaveTitle(/MyApp$/);
````

// ✅ After page changes
await page.getByRole('link', { name: 'Profile' }).click();
await expect(page).toHaveTitle('User Profile - MyApp');

````

## Collection Assertions

### `toHaveCount()`

Validates the number of elements matching a locator.

```typescript
// ✅ List item counts
await expect(page.getByTestId('user-row')).toHaveCount(10);
await expect(page.getByRole('listitem')).toHaveCount(5);
await expect(page.getByText(/Product \d+/)).toHaveCount(15);

// ✅ Dynamic count validation
await page.getByRole('button', { name: 'Add Item' }).click();
await expect(page.getByTestId('cart-item')).toHaveCount(3);

// ✅ Empty state validation
await expect(page.getByTestId('result-item')).toHaveCount(0);
await expect(page.getByText('No results found')).toBeVisible();
````

### Collection Content Validation

```typescript
// ✅ Validating all items in a collection
await expect(page.getByTestId('status-badge')).toHaveText(['Active', 'Pending', 'Inactive']);

// ✅ Validating collection properties
const menuItems = page.getByRole('menuitem');
await expect(menuItems).toHaveCount(4);
await expect(menuItems.nth(0)).toHaveText('Home');
await expect(menuItems.nth(1)).toHaveText('Profile');
await expect(menuItems.nth(2)).toHaveText('Settings');
await expect(menuItems.nth(3)).toHaveText('Logout');
```

## Screenshot Assertions

### `toHaveScreenshot()`

Validates visual appearance through screenshot comparison.

```typescript
// ✅ Full page screenshots
await expect(page).toHaveScreenshot('profile-page.png');

// ✅ Element screenshots
await expect(page.getByTestId('chart')).toHaveScreenshot('sales-chart.png');

// ✅ With configuration options
await expect(page.getByTestId('dynamic-content')).toHaveScreenshot('content.png', {
  mask: [page.getByTestId('timestamp')], // Mask dynamic content
  threshold: 0.2, // Allow 20% difference
});

// ✅ Mobile responsive screenshots
await page.setViewportSize({ width: 375, height: 667 });
await expect(page).toHaveScreenshot('mobile-profile.png');
```

## Custom Assertion Patterns

### Combining Multiple Assertions

```typescript
// ✅ Comprehensive element validation
async function validateUserCard(page: Page, userData: UserData) {
  const userCard = page.getByTestId('user-card');

  // Structural validation
  await expect(userCard).toBeVisible();
  await expect(userCard).toHaveAttribute('data-user-id', userData.id);

  // Content validation
  await expect(userCard.getByRole('heading')).toHaveText(userData.name);
  await expect(userCard.getByText(userData.email)).toBeVisible();
  await expect(userCard.getByText(`Role: ${userData.role}`)).toBeVisible();

  // State validation
  if (userData.isActive) {
    await expect(userCard.getByTestId('status-badge')).toHaveText('Active');
    await expect(userCard.getByTestId('status-badge')).toHaveClass('status-active');
  } else {
    await expect(userCard.getByTestId('status-badge')).toHaveText('Inactive');
    await expect(userCard.getByTestId('status-badge')).toHaveClass('status-inactive');
  }
}
```

### Conditional Assertions

```typescript
// ✅ Feature flag dependent assertions
async function validateProfile(page: Page, features: FeatureFlags) {
  // Base validation
  await expect(page.getByRole('heading', { name: 'Profile' })).toBeVisible();
```

### Error State Assertions

```typescript
// ✅ Comprehensive error validation
async function validateErrorState(page: Page, expectedError: ErrorInfo) {
  // Error visibility
  await expect(page.getByRole('alert')).toBeVisible();

  // Error content
  await expect(page.getByRole('alert')).toContainText(expectedError.message);
  await expect(page.getByTestId('error-code')).toHaveText(expectedError.code);

  // Error styling
  await expect(page.getByRole('alert')).toHaveClass('alert-error');
  await expect(page.getByRole('alert')).toHaveCSS('color', 'rgb(220, 53, 69)');

  // Related UI state
  await expect(page.getByRole('button', { name: 'Submit' })).toBeEnabled(); // Still can retry
  await expect(page.getByRole('button', { name: 'Reset' })).toBeVisible(); // Reset option available
}
```

## Assertion Timing Configuration

### Global Timeout Configuration

```typescript
// In playwright.config.ts
export default defineConfig({
  expect: {
    timeout: 10000, // Global assertion timeout
  },
});
```

### Per-Assertion Timeout

```typescript
// ✅ Specific timeout for slow operations
await expect(page.getByText('Report generated')).toBeVisible({ timeout: 30000 });

// ✅ Shorter timeout for fast operations
await expect(page.getByText('Validation error')).toBeVisible({ timeout: 2000 });
```

### Retry Configuration

```typescript
// ✅ Custom retry configuration
await expect(page.getByTestId('flaky-element')).toBeVisible({
  timeout: 15000,
  intervals: [1000, 2000, 5000], // Custom retry intervals
});
```

## Best Practices Summary

### ✅ Do Use

- Semantic locators in assertions (`getByRole`, `getByLabel`)
- Specific assertion methods for different validation types
- Custom timeouts only when necessary with justification
- Meaningful error messages for debugging
- Combined assertions for comprehensive validation

### ❌ Don't Use

- Generic selectors in assertions
- Manual waits before assertions
- Overly broad assertions that can cause false positives
- Assertions without clear business context
- Hardcoded values that should be parameterized

This reference guide provides the foundation for choosing appropriate assertion methods based on your specific validation needs. Combine these patterns with the best practices outlined in other documentation files for optimal test reliability and maintainability.
