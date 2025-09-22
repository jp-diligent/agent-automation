# Common Assertion Examples

**Description**: Collection of frequently used assertion patterns and examples for common UI testing scenarios. This serves as a quick reference for implementing assertions in typical web application testing situations.

## Table of Contents

- [Basic Element Assertions](#basic-element-assertions)
- [Form Validation Assertions](#form-validation-assertions)
- [Navigation and URL Assertions](#navigation-and-url-assertions)
- [Table and List Assertions](#table-and-list-assertions)
- [Modal and Dialog Assertions](#modal-and-dialog-assertions)
- [Loading State Assertions](#loading-state-assertions)
- [Error State Assertions](#error-state-assertions)
- [User Interface State Assertions](#user-interface-state-assertions)

## Basic Element Assertions

### Element Visibility and Presence

```typescript
// ✅ Check if element is visible
await expect(page.getByRole('button', { name: 'Submit' })).toBeVisible();
await expect(page.getByText('Welcome to Profile')).toBeVisible();
await expect(page.getByTestId('user-profile')).toBeVisible();

// ✅ Check if element is hidden/not visible
await expect(page.getByText('Loading...')).not.toBeVisible();
await expect(page.getByTestId('error-banner')).not.toBeVisible();

// ✅ Check element count
await expect(page.getByTestId('menu-item')).toHaveCount(5);
await expect(page.getByRole('listitem')).toHaveCount(10);
await expect(page.locator('.notification')).toHaveCount(0); // No notifications
```

### Text Content Validation

```typescript
// ✅ Exact text matching
await expect(page.getByRole('heading')).toHaveText('User Profile');
await expect(page.getByTestId('welcome-message')).toHaveText('Welcome, John Doe!');

// ✅ Partial text matching
await expect(page.getByTestId('status-message')).toContainText('successfully');
await expect(page.getByRole('alert')).toContainText('error');

// ✅ Text with regular expressions
await expect(page.getByTestId('timestamp')).toHaveText(/\d{4}-\d{2}-\d{2}/);
await expect(page.getByTestId('phone-number')).toHaveText(/^\(\d{3}\) \d{3}-\d{4}$/);

// ✅ Multiple elements with expected texts
await expect(page.getByTestId('tab')).toHaveText(['Home', 'Profile', 'Settings']);
```

### Attribute and Property Validation

```typescript
// ✅ Check attributes
await expect(page.getByRole('link')).toHaveAttribute('href', '/profile');
await expect(page.getByTestId('external-link')).toHaveAttribute('target', '_blank');
await expect(page.getByLabel('Required Field')).toHaveAttribute('required');

// ✅ Check CSS classes
await expect(page.getByRole('button')).toHaveClass('btn-primary');
await expect(page.getByTestId('status-indicator')).toHaveClass(['status', 'active']);

// ✅ Check CSS properties
await expect(page.getByTestId('error-text')).toHaveCSS('color', 'rgb(220, 53, 69)');
await expect(page.getByTestId('sidebar')).toHaveCSS('width', '300px');
```

## Form Validation Assertions

### Input Field Validation

```typescript
// ✅ Input values
await expect(page.getByLabel('Email')).toHaveValue('user@example.com');
await expect(page.getByLabel('First Name')).toHaveValue('John');
await expect(page.getByLabel('Age')).toHaveValue('25');

// ✅ Input states
await expect(page.getByLabel('Username')).toBeEnabled();
await expect(page.getByLabel('Employee ID')).toBeDisabled();
await expect(page.getByLabel('Email')).toBeEditable();
await expect(page.getByLabel('Read Only Field')).not.toBeEditable();

// ✅ Placeholder text
await expect(page.getByLabel('Search')).toHaveAttribute('placeholder', 'Enter search terms...');

// ✅ Input validation states
await expect(page.getByLabel('Email')).not.toHaveClass('error');
await expect(page.getByTestId('email-error')).not.toBeVisible();
```

### Checkbox and Radio Button Validation

```typescript
// ✅ Checkbox states
await expect(page.getByRole('checkbox', { name: 'I agree to terms' })).toBeChecked();
await expect(page.getByRole('checkbox', { name: 'Newsletter' })).not.toBeChecked();

// ✅ Radio button selection
await expect(page.getByRole('radio', { name: 'Premium Plan' })).toBeChecked();
await expect(page.getByRole('radio', { name: 'Basic Plan' })).not.toBeChecked();

// ✅ Multiple checkboxes
const permissions = ['Read', 'Write', 'Delete'];
for (const permission of permissions) {
  await expect(page.getByRole('checkbox', { name: permission })).toBeChecked();
}
```

### Select Dropdown Validation

```typescript
// ✅ Select element value
await expect(page.getByLabel('Country')).toHaveValue('US');
await expect(page.getByLabel('Language')).toHaveValue('en');

// ✅ Select options validation
const countrySelect = page.getByLabel('Country');
await expect(countrySelect.getByRole('option')).toHaveCount(195);
await expect(countrySelect.getByRole('option', { name: 'United States' })).toBeVisible();
```

### Form Submission States

```typescript
// ✅ Submit button states
await expect(page.getByRole('button', { name: 'Submit' })).toBeEnabled();
await expect(page.getByRole('button', { name: 'Submit' })).toBeDisabled();

// ✅ Form validation messages
await expect(page.getByText('Please fill in all required fields')).toBeVisible();
await expect(page.getByTestId('form-success')).toHaveText('Form submitted successfully');

// ✅ Field-specific error messages
await expect(page.getByTestId('email-error')).toHaveText('Please enter a valid email address');
await expect(page.getByTestId('password-error')).toHaveText('Password must be at least 8 characters');
```

## Navigation and URL Assertions

### URL Validation

```typescript
// ✅ Exact URL matching
await expect(page).toHaveURL('https://example.com/profile');
await expect(page).toHaveURL('/users/profile');

// ✅ URL pattern matching
await expect(page).toHaveURL(/\/users\/\d+/);
await expect(page).toHaveURL(/profile$/);
await expect(page).toHaveURL(/^https:\/\/.*\.example\.com/);

// ✅ URL contains specific parts
await expect(page).toHaveURL(/\/search\?q=playwright/);
```

### Page Title Validation

```typescript
// ✅ Page title assertions
await expect(page).toHaveTitle('User Profile - MyApp');
await expect(page).toHaveTitle(/Profile/);
await expect(page).toHaveTitle(/MyApp$/);
```

### Navigation Menu Validation

```typescript
// ✅ Navigation structure
await expect(page.getByRole('navigation')).toBeVisible();
await expect(page.getByRole('menuitem')).toHaveCount(5);

// ✅ Menu items and links
await expect(page.getByRole('link', { name: 'Home' })).toBeVisible();
await expect(page.getByRole('link', { name: 'Profile' })).toHaveAttribute('href', '/profile');

// ✅ Active menu item
await expect(page.getByRole('link', { name: 'Dashboard' })).toHaveClass('active');
```

## Table and List Assertions

### Table Structure Validation

```typescript
// ✅ Table presence and structure
await expect(page.getByRole('table')).toBeVisible();
await expect(page.getByRole('columnheader')).toHaveCount(5);
await expect(page.getByRole('row')).toHaveCount(11); // 1 header + 10 data rows

// ✅ Table headers
await expect(page.getByRole('columnheader')).toHaveText(['Name', 'Email', 'Role', 'Status', 'Actions']);

// ✅ Specific table content
await expect(page.getByRole('cell', { name: 'John Doe' })).toBeVisible();
await expect(page.getByRole('row').filter({ hasText: 'john@example.com' })).toBeVisible();
```

### List Item Validation

```typescript
// ✅ List structure
await expect(page.getByRole('list')).toBeVisible();
await expect(page.getByRole('listitem')).toHaveCount(8);

// ✅ Specific list items
await expect(page.getByRole('listitem').first()).toHaveText('First item');
await expect(page.getByRole('listitem').last()).toHaveText('Last item');
await expect(page.getByRole('listitem').nth(2)).toContainText('Third item');
```

### Data Grid Validation

```typescript
// ✅ Grid with sorting and filtering
const dataGrid = page.getByTestId('data-grid');
await expect(dataGrid).toBeVisible();

// Sort indicator
await expect(dataGrid.getByTestId('sort-indicator-name')).toHaveText('↑');

// Filter status
await expect(dataGrid.getByTestId('active-filters')).toHaveText('2 filters active');

// Pagination
await expect(page.getByTestId('pagination-info')).toHaveText('Showing 1-10 of 150');
```

## Modal and Dialog Assertions

### Modal Dialog Validation

```typescript
// ✅ Modal presence and accessibility
await expect(page.getByRole('dialog')).toBeVisible();
await expect(page.getByRole('dialog')).toHaveAttribute('aria-modal', 'true');

// ✅ Modal content
await expect(page.getByRole('dialog').getByRole('heading')).toHaveText('Confirm Action');
await expect(page.getByRole('dialog').getByText('Are you sure?')).toBeVisible();

// ✅ Modal buttons
await expect(page.getByRole('dialog').getByRole('button', { name: 'Confirm' })).toBeVisible();
await expect(page.getByRole('dialog').getByRole('button', { name: 'Cancel' })).toBeVisible();

// ✅ Modal backdrop
await expect(page.locator('.modal-backdrop')).toBeVisible();
```

### Confirmation Dialogs

```typescript
// ✅ Alert dialogs
await expect(page.getByRole('alertdialog')).toBeVisible();
await expect(page.getByRole('alertdialog').getByText('Delete User')).toBeVisible();
await expect(page.getByRole('alertdialog').getByText(/cannot be undone/)).toBeVisible();

// ✅ Dialog actions
await expect(page.getByRole('button', { name: 'Delete' })).toHaveClass('btn-danger');
await expect(page.getByRole('button', { name: 'Cancel' })).toBeEnabled();
```

### Tooltip and Popover Validation

```typescript
// ✅ Tooltip presence
await page.getByTestId('help-icon').hover();
await expect(page.getByRole('tooltip')).toBeVisible();
await expect(page.getByRole('tooltip')).toHaveText('Click for more information');

// ✅ Popover content
await page.getByRole('button', { name: 'Options' }).click();
await expect(page.getByRole('menu')).toBeVisible();
await expect(page.getByRole('menuitem')).toHaveCount(4);
```

## Loading State Assertions

### Loading Indicators

```typescript
// ✅ Loading spinner visibility
await expect(page.getByTestId('loading-spinner')).toBeVisible();
await expect(page.getByTestId('loading-spinner')).not.toBeVisible();

// ✅ Loading messages
await expect(page.getByText('Loading data...')).toBeVisible();
await expect(page.getByText('Please wait')).toBeVisible();

// ✅ Progress bars
await expect(page.getByRole('progressbar')).toBeVisible();
await expect(page.getByRole('progressbar')).toHaveAttribute('value', '50');
```

### Skeleton Loading States

```typescript
// ✅ Skeleton elements
await expect(page.getByTestId('skeleton-card')).toBeVisible();
await expect(page.getByTestId('skeleton-card')).toHaveClass('skeleton-loading');

// ✅ Transition from skeleton to content
await expect(page.getByTestId('skeleton-card')).not.toBeVisible();
await expect(page.getByTestId('actual-card')).toBeVisible();
```

### Async Content Loading

```typescript
// ✅ Sequential loading validation
await expect(page.getByText('Loading...')).toBeVisible();
await expect(page.getByText('Data loaded')).toBeVisible();
await expect(page.getByText('Loading...')).not.toBeVisible();

// ✅ Content availability after loading
await expect(page.getByTestId('data-container')).toBeVisible();
await expect(page.getByTestId('data-item')).toHaveCount.greaterThan(0);
```

## Error State Assertions

### Error Messages and Alerts

```typescript
// ✅ Error visibility and content
await expect(page.getByRole('alert')).toBeVisible();
await expect(page.getByRole('alert')).toHaveText('Operation failed');
await expect(page.getByRole('alert')).toHaveClass('alert-error');

// ✅ Field-specific errors
await expect(page.getByTestId('username-error')).toBeVisible();
await expect(page.getByTestId('username-error')).toHaveText('Username is required');

// ✅ Multiple error scenarios
await expect(page.getByTestId('error-list').getByRole('listitem')).toHaveCount(3);
await expect(page.getByTestId('error-list')).toContainText('Email is invalid');
```

### Network Error States

```typescript
// ✅ Connection errors
await expect(page.getByText('Connection lost')).toBeVisible();
await expect(page.getByText('Unable to connect to server')).toBeVisible();
await expect(page.getByRole('button', { name: 'Retry' })).toBeVisible();

// ✅ Timeout errors
await expect(page.getByText('Request timed out')).toBeVisible();
await expect(page.getByText('Please try again later')).toBeVisible();
```

### Validation Error States

```typescript
// ✅ Form validation errors
await expect(page.getByText('Please correct the following errors:')).toBeVisible();
await expect(page.locator('.field-error')).toHaveCount(2);

// ✅ Server validation errors
await expect(page.getByText('Email address already exists')).toBeVisible();
await expect(page.getByText('Invalid username or password')).toBeVisible();
```

## User Interface State Assertions

### Button States

```typescript
// ✅ Button interactions
await expect(page.getByRole('button', { name: 'Save' })).toBeEnabled();
await expect(page.getByRole('button', { name: 'Save' })).toBeDisabled();
await expect(page.getByRole('button', { name: 'Delete' })).toHaveClass('btn-danger');

// ✅ Toggle buttons
await expect(page.getByRole('button', { name: 'Toggle Sidebar' })).toHaveAttribute('aria-pressed', 'false');
await expect(page.getByRole('button', { name: 'Toggle Theme' })).toHaveAttribute('aria-pressed', 'true');
```

### Focus and Accessibility States

```typescript
// ✅ Focus management
await expect(page.getByLabel('Search')).toBeFocused();
await expect(page.getByRole('button', { name: 'Next' })).toBeFocused();

// ✅ ARIA states
await expect(page.getByRole('button', { name: 'Menu' })).toHaveAttribute('aria-expanded', 'false');
await expect(page.getByRole('tabpanel')).toHaveAttribute('aria-hidden', 'false');
```

### Responsive States

```typescript
// ✅ Mobile/desktop visibility
await page.setViewportSize({ width: 768, height: 1024 });
await expect(page.getByTestId('mobile-menu')).toBeVisible();
await expect(page.getByTestId('desktop-nav')).not.toBeVisible();

await page.setViewportSize({ width: 1200, height: 800 });
await expect(page.getByTestId('mobile-menu')).not.toBeVisible();
await expect(page.getByTestId('desktop-nav')).toBeVisible();
```

### Theme and Appearance States

```typescript
// ✅ Theme validation
await expect(page.locator('html')).toHaveAttribute('data-theme', 'dark');
await expect(page.getByTestId('theme-toggle')).toHaveText('Light Mode');

// ✅ Color scheme
await expect(page.locator('body')).toHaveCSS('background-color', 'rgb(33, 37, 41)');
```

## Complex Workflow Examples

### Multi-Step Form Validation

```typescript
test('complete user registration workflow', async ({ page }) => {
  await page.goto('/register');

  // Step 1: Personal Information
  await expect(page.getByText('Step 1 of 3')).toBeVisible();
  await expect(page.getByRole('button', { name: 'Next' })).toBeDisabled();

  await page.getByLabel('First Name').fill('John');
  await page.getByLabel('Last Name').fill('Doe');
  await expect(page.getByRole('button', { name: 'Next' })).toBeEnabled();

  // Step 2: Account Details
  await page.getByRole('button', { name: 'Next' }).click();
  await expect(page.getByText('Step 2 of 3')).toBeVisible();

  await page.getByLabel('Email').fill('john@example.com');
  await page.getByLabel('Password').fill('SecurePass123!');
  await expect(page.getByText('Password strength: Strong')).toBeVisible();

  // Step 3: Confirmation
  await page.getByRole('button', { name: 'Next' }).click();
  await expect(page.getByText('Step 3 of 3')).toBeVisible();
  await expect(page.getByText('John Doe')).toBeVisible();
  await expect(page.getByText('john@example.com')).toBeVisible();

  await page.getByRole('button', { name: 'Complete Registration' }).click();
  await expect(page.getByText('Registration successful')).toBeVisible();
});
```

### E-commerce Cart Validation

```typescript
test('shopping cart operations', async ({ page }) => {
  await page.goto('/products');

  // Add items to cart
  await page.getByTestId('product-1').getByRole('button', { name: 'Add to Cart' }).click();
  await expect(page.getByTestId('cart-counter')).toHaveText('1');

  // View cart
  await page.getByTestId('cart-icon').click();
  await expect(page.getByTestId('cart-drawer')).toBeVisible();
  await expect(page.getByTestId('cart-item')).toHaveCount(1);
  await expect(page.getByTestId('cart-total')).toHaveText('$29.99');

  // Update quantity
  await page.getByTestId('quantity-input').fill('2');
  await expect(page.getByTestId('cart-total')).toHaveText('$59.98');

  // Proceed to checkout
  await page.getByRole('button', { name: 'Checkout' }).click();
  await expect(page).toHaveURL('/checkout');
  await expect(page.getByText('Order Summary')).toBeVisible();
});
```

This comprehensive reference provides ready-to-use assertion patterns for common web application testing scenarios. Copy and adapt these examples for your specific testing needs.
