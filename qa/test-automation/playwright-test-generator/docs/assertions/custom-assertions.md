# Custom Assertions

**Description**: Guide to creating reusable custom assertion methods that encapsulate complex validation logic and provide domain-specific testing capabilities. This document covers patterns for building maintainable assertion libraries.

## Table of Contents

- [Custom Assertion Patterns](#custom-assertion-patterns)
- [Business Logic Assertions](#business-logic-assertions)
- [Reusable Assertion Helpers](#reusable-assertion-helpers)
- [Type-Safe Custom Assertions](#type-safe-custom-assertions)
- [Assertion Composition](#assertion-composition)
- [Test Utilities Integration](#test-utilities-integration)
- [Domain-Specific Assertions](#domain-specific-assertions)
- [Assertion Libraries](#assertion-libraries)

## Custom Assertion Patterns

### Basic Custom Assertion Structure

```typescript
// ✅ Simple custom assertion helper
export class CustomAssertions {
  static async assertUserCard(
    page: Page,
    userData: {
      name: string;
      email: string;
      role: string;
      isActive: boolean;
    },
  ): Promise<void> {
    const userCard = page.getByTestId('user-card');

    // Structural validation
    await expect(userCard, 'User card should be visible').toBeVisible();

    // Content validation
    await expect(userCard.getByRole('heading'), 'User name should be displayed prominently').toHaveText(userData.name);

    await expect(userCard.getByText(userData.email), 'User email should be visible in the card').toBeVisible();

    await expect(userCard.getByText(`Role: ${userData.role}`), 'User role should be clearly indicated').toBeVisible();

    // Status-dependent validation
    const statusBadge = userCard.getByTestId('status-badge');
    if (userData.isActive) {
      await expect(statusBadge).toHaveText('Active');
      await expect(statusBadge).toHaveClass('status-active');
    } else {
      await expect(statusBadge).toHaveText('Inactive');
      await expect(statusBadge).toHaveClass('status-inactive');
    }
  }
}

// Usage in tests
test('user profile display', async ({ page }) => {
  const profilePage = new ProfilePage(page);
  await profilePage.goto();

  await CustomAssertions.assertUserCard(page, {
    name: 'John Doe',
    email: 'john.doe@example.com',
    role: 'Administrator',
    isActive: true,
  });
});
```

### Fluent Assertion Interface

```typescript
// ✅ Fluent interface for complex assertions
export class FluentAssertions {
  constructor(private page: Page) {}

  element(selector: string) {
    return new ElementAssertion(this.page, selector);
  }

  form(formName: string) {
    return new FormAssertion(this.page, formName);
  }

  table(tableName: string) {
    return new TableAssertion(this.page, tableName);
  }
}

class ElementAssertion {
  constructor(
    private page: Page,
    private selector: string,
  ) {}

  async shouldBeVisible(): Promise<this> {
    await expect(this.page.locator(this.selector)).toBeVisible();
    return this;
  }

  async shouldHaveText(text: string): Promise<this> {
    await expect(this.page.locator(this.selector)).toHaveText(text);
    return this;
  }

  async shouldHaveClass(className: string): Promise<this> {
    await expect(this.page.locator(this.selector)).toHaveClass(className);
    return this;
  }

  async shouldBeEnabled(): Promise<this> {
    await expect(this.page.locator(this.selector)).toBeEnabled();
    return this;
  }
}

class FormAssertion {
  constructor(
    private page: Page,
    private formSelector: string,
  ) {}

  async shouldBeValid(): Promise<this> {
    const form = this.page.locator(this.formSelector);
    await expect(form).toBeVisible();
    await expect(form.getByText(/error/i)).not.toBeVisible();
    await expect(form.getByRole('button', { name: /submit/i })).toBeEnabled();
    return this;
  }

  async shouldHaveError(fieldName: string, errorMessage: string): Promise<this> {
    const form = this.page.locator(this.formSelector);
    const field = form.getByLabel(fieldName);
    const error = form.locator(`[data-error-for="${fieldName}"]`);

    await expect(field).toHaveClass(/error/);
    await expect(error).toHaveText(errorMessage);
    return this;
  }

  async fieldShouldHaveValue(fieldName: string, value: string): Promise<this> {
    const field = this.page.locator(this.formSelector).getByLabel(fieldName);
    await expect(field).toHaveValue(value);
    return this;
  }
}

// Usage with fluent interface
test('fluent assertions example', async ({ page }) => {
  const assert = new FluentAssertions(page);

  await page.goto('/contact');

  // Fluent chaining
  await assert.element('[data-testid="contact-form"]').shouldBeVisible().shouldHaveClass('form-container');

  await assert
    .form('[data-testid="contact-form"]')
    .fieldShouldHaveValue('Name', '')
    .fieldShouldHaveValue('Email', '')
    .shouldBeValid();
});
```

## Business Logic Assertions

### Domain-Specific Validation

```typescript
// ✅ E-commerce specific assertions
export class EcommerceAssertions {
  static async assertShoppingCart(
    page: Page,
    expectedCart: {
      items: Array<{
        name: string;
        quantity: number;
        price: number;
      }>;
      subtotal: number;
      tax: number;
      total: number;
      discountCode?: string;
      discountAmount?: number;
    },
  ): Promise<void> {
    const cart = page.getByTestId('shopping-cart');

    await expect(cart, 'Shopping cart should be visible').toBeVisible();

    // Validate items
    for (let i = 0; i < expectedCart.items.length; i++) {
      const item = expectedCart.items[i];
      const cartItem = cart.getByTestId(`cart-item-${i}`);

      await expect(cartItem.getByTestId('item-name')).toHaveText(item.name);
      await expect(cartItem.getByTestId('item-quantity')).toHaveValue(item.quantity.toString());
      await expect(cartItem.getByTestId('item-price')).toHaveText(`$${item.price.toFixed(2)}`);

      const itemTotal = item.quantity * item.price;
      await expect(cartItem.getByTestId('item-total')).toHaveText(`$${itemTotal.toFixed(2)}`);
    }

    // Validate totals
    await expect(cart.getByTestId('subtotal')).toHaveText(`$${expectedCart.subtotal.toFixed(2)}`);
    await expect(cart.getByTestId('tax')).toHaveText(`$${expectedCart.tax.toFixed(2)}`);
    await expect(cart.getByTestId('total')).toHaveText(`$${expectedCart.total.toFixed(2)}`);

    // Validate discount if present
    if (expectedCart.discountCode && expectedCart.discountAmount) {
      await expect(cart.getByTestId('discount-code')).toHaveText(expectedCart.discountCode);
      await expect(cart.getByTestId('discount-amount')).toHaveText(`-$${expectedCart.discountAmount.toFixed(2)}`);
    }
  }

  static async assertOrderConfirmation(
    page: Page,
    orderData: {
      orderNumber: string;
      customerEmail: string;
      items: Array<{ name: string; quantity: number }>;
      total: number;
      estimatedDelivery: string;
    },
  ): Promise<void> {
    const confirmation = page.getByTestId('order-confirmation');

    await expect(confirmation).toBeVisible();

    // Order details
    await expect(confirmation.getByTestId('order-number')).toHaveText(orderData.orderNumber);
    await expect(confirmation.getByText(orderData.customerEmail)).toBeVisible();
    await expect(confirmation.getByTestId('order-total')).toHaveText(`$${orderData.total.toFixed(2)}`);
    await expect(confirmation.getByTestId('estimated-delivery')).toHaveText(orderData.estimatedDelivery);

    // Validate order items
    for (const item of orderData.items) {
      await expect(confirmation.getByText(item.name)).toBeVisible();
      await expect(confirmation.getByText(`Qty: ${item.quantity}`)).toBeVisible();
    }

    // Validate order number format
    expect(orderData.orderNumber).toMatch(/^ORD-\d{8}$/);
  }
}
```

### Financial Calculation Assertions

```typescript
// ✅ Financial/accounting specific assertions
export class FinancialAssertions {
  static async assertInvoiceCalculations(
    page: Page,
    expectedInvoice: {
      lineItems: Array<{
        description: string;
        quantity: number;
        rate: number;
        amount: number;
      }>;
      subtotal: number;
      discountPercent?: number;
      discountAmount?: number;
      taxPercent: number;
      taxAmount: number;
      total: number;
    },
  ): Promise<void> {
    const invoice = page.getByTestId('invoice');

    // Validate line items
    await expect(invoice.getByTestId('line-item')).toHaveCount(expectedInvoice.lineItems.length);

    for (let i = 0; i < expectedInvoice.lineItems.length; i++) {
      const item = expectedInvoice.lineItems[i];
      const lineItem = invoice.getByTestId(`line-item-${i}`);

      await expect(lineItem.getByTestId('description')).toHaveText(item.description);
      await expect(lineItem.getByTestId('quantity')).toHaveText(item.quantity.toString());
      await expect(lineItem.getByTestId('rate')).toHaveText(`$${item.rate.toFixed(2)}`);
      await expect(lineItem.getByTestId('amount')).toHaveText(`$${item.amount.toFixed(2)}`);

      // Validate calculation
      const calculatedAmount = item.quantity * item.rate;
      expect(item.amount).toBeCloseTo(calculatedAmount, 2);
    }

    // Validate totals with precision
    await expect(invoice.getByTestId('subtotal')).toHaveText(`$${expectedInvoice.subtotal.toFixed(2)}`);

    if (expectedInvoice.discountPercent && expectedInvoice.discountAmount) {
      await expect(invoice.getByTestId('discount-percent')).toHaveText(`${expectedInvoice.discountPercent}%`);
      await expect(invoice.getByTestId('discount-amount')).toHaveText(`$${expectedInvoice.discountAmount.toFixed(2)}`);

      // Validate discount calculation
      const calculatedDiscount = expectedInvoice.subtotal * (expectedInvoice.discountPercent / 100);
      expect(expectedInvoice.discountAmount).toBeCloseTo(calculatedDiscount, 2);
    }

    await expect(invoice.getByTestId('tax-percent')).toHaveText(`${expectedInvoice.taxPercent}%`);
    await expect(invoice.getByTestId('tax-amount')).toHaveText(`$${expectedInvoice.taxAmount.toFixed(2)}`);
    await expect(invoice.getByTestId('total')).toHaveText(`$${expectedInvoice.total.toFixed(2)}`);

    // Validate final calculation
    const taxableAmount = expectedInvoice.discountAmount
      ? expectedInvoice.subtotal - expectedInvoice.discountAmount
      : expectedInvoice.subtotal;
    const calculatedTax = taxableAmount * (expectedInvoice.taxPercent / 100);
    const calculatedTotal = taxableAmount + calculatedTax;

    expect(expectedInvoice.taxAmount).toBeCloseTo(calculatedTax, 2);
    expect(expectedInvoice.total).toBeCloseTo(calculatedTotal, 2);
  }
}
```

## Reusable Assertion Helpers

### Generic Helper Functions

```typescript
// ✅ Reusable assertion utilities
export class AssertionHelpers {
  static async assertElementExists(page: Page, selector: string, description: string): Promise<Locator> {
    const element = page.locator(selector);
    await expect(element, `${description} should exist`).toBeVisible();
    return element;
  }

  static async assertElementCount(
    page: Page,
    selector: string,
    expectedCount: number,
    description: string,
  ): Promise<void> {
    await expect(page.locator(selector), `Should have exactly ${expectedCount} ${description}`).toHaveCount(
      expectedCount,
    );
  }

  static async assertElementsHaveText(elements: Locator, expectedTexts: string[]): Promise<void> {
    await expect(elements).toHaveCount(expectedTexts.length);

    for (let i = 0; i < expectedTexts.length; i++) {
      await expect(elements.nth(i)).toHaveText(expectedTexts[i]);
    }
  }

  static async assertFormField(
    page: Page,
    fieldLabel: string,
    expectedValue: string,
    shouldBeEnabled: boolean = true,
  ): Promise<void> {
    const field = page.getByLabel(fieldLabel);

    await expect(field, `${fieldLabel} field should be visible`).toBeVisible();
    await expect(field, `${fieldLabel} should have correct value`).toHaveValue(expectedValue);

    if (shouldBeEnabled) {
      await expect(field, `${fieldLabel} should be enabled`).toBeEnabled();
    } else {
      await expect(field, `${fieldLabel} should be disabled`).toBeDisabled();
    }
  }

  static async assertUrlPattern(page: Page, pattern: RegExp, description: string): Promise<void> {
    await expect(page, description).toHaveURL(pattern);
  }

  static async assertNoErrors(page: Page): Promise<void> {
    // Check for common error indicators
    await expect(page.getByRole('alert')).not.toBeVisible();
    await expect(page.getByText(/error/i)).not.toBeVisible();
    await expect(page.locator('.error, .alert-error')).not.toBeVisible();
  }
}

// Usage in tests
test('helper functions example', async ({ page }) => {
  await page.goto('/user-profile');

  // Use helpers for common patterns
  await AssertionHelpers.assertElementExists(page, '[data-testid="profile-card"]', 'Profile card');
  await AssertionHelpers.assertElementCount(page, '.skill-tag', 5, 'skill tags');

  await AssertionHelpers.assertFormField(page, 'First Name', 'John', true);
  await AssertionHelpers.assertFormField(page, 'Last Name', 'Doe', true);
  await AssertionHelpers.assertFormField(page, 'Employee ID', 'EMP001', false);

  await AssertionHelpers.assertUrlPattern(page, /\/user-profile\/\d+/, 'Profile URL should include user ID');
  await AssertionHelpers.assertNoErrors(page);
});
```

### State Validation Helpers

```typescript
// ✅ Application state validation helpers
export class StateAssertions {
  static async assertLoadingState(page: Page, isLoading: boolean, context: string = ''): Promise<void> {
    const loadingIndicator = page.getByTestId('loading-spinner');
    const loadingMessage = page.getByTestId('loading-message');

    if (isLoading) {
      await expect(
        loadingIndicator,
        `Loading indicator should be visible${context ? ` ${context}` : ''}`,
      ).toBeVisible();
      await expect(loadingMessage, `Loading message should be visible${context ? ` ${context}` : ''}`).toBeVisible();
    } else {
      await expect(
        loadingIndicator,
        `Loading indicator should not be visible${context ? ` ${context}` : ''}`,
      ).not.toBeVisible();
      await expect(
        loadingMessage,
        `Loading message should not be visible${context ? ` ${context}` : ''}`,
      ).not.toBeVisible();
    }
  }

  static async assertEmptyState(page: Page, containerSelector: string, emptyMessage: string): Promise<void> {
    const container = page.locator(containerSelector);

    await expect(container).toBeVisible();
    await expect(container.getByText(emptyMessage)).toBeVisible();
    await expect(container.locator('[data-testid*="item"]')).toHaveCount(0);
  }

  static async assertErrorState(
    page: Page,
    expectedError: {
      type: 'validation' | 'network' | 'permission' | 'general';
      message: string;
      recoverable: boolean;
    },
  ): Promise<void> {
    const errorContainer = page.getByRole('alert');

    await expect(errorContainer).toBeVisible();
    await expect(errorContainer).toContainText(expectedError.message);

    // Check error type styling
    const expectedClass = `alert-${expectedError.type}`;
    await expect(errorContainer).toHaveClass(new RegExp(expectedClass));

    // Check recovery options
    if (expectedError.recoverable) {
      await expect(page.getByRole('button', { name: /retry|try again/i })).toBeVisible();
    } else {
      await expect(page.getByRole('button', { name: /contact support/i })).toBeVisible();
    }
  }
}
```

## Type-Safe Custom Assertions

### TypeScript Integration

```typescript
// ✅ Type-safe custom assertions
interface UserProfile {
  id: string;
  name: string;
  email: string;
  role: 'admin' | 'user' | 'manager';
  isActive: boolean;
  lastLogin?: Date;
}

interface TableValidationOptions {
  sortable?: boolean;
  filterable?: boolean;
  paginated?: boolean;
  selectable?: boolean;
}

export class TypedAssertions {
  static async assertUserProfile(page: Page, expectedProfile: UserProfile): Promise<void> {
    const profileCard = page.getByTestId('user-profile');

    await expect(profileCard).toBeVisible();

    // Type-safe field validation
    await expect(profileCard.getByTestId('user-id')).toHaveText(expectedProfile.id);
    await expect(profileCard.getByTestId('user-name')).toHaveText(expectedProfile.name);
    await expect(profileCard.getByTestId('user-email')).toHaveText(expectedProfile.email);
    await expect(profileCard.getByTestId('user-role')).toHaveText(expectedProfile.role);

    // Status validation
    const statusBadge = profileCard.getByTestId('status-badge');
    if (expectedProfile.isActive) {
      await expect(statusBadge).toHaveText('Active');
      await expect(statusBadge).toHaveClass('status-active');
    } else {
      await expect(statusBadge).toHaveText('Inactive');
      await expect(statusBadge).toHaveClass('status-inactive');
    }

    // Optional last login validation
    if (expectedProfile.lastLogin) {
      const lastLoginText = expectedProfile.lastLogin.toLocaleDateString();
      await expect(profileCard.getByTestId('last-login')).toContainText(lastLoginText);
    }
  }

  static async assertTable<T extends Record<string, any>>(
    page: Page,
    tableSelector: string,
    expectedData: T[],
    columnMapping: Record<keyof T, string>,
    options: TableValidationOptions = {},
  ): Promise<void> {
    const table = page.locator(tableSelector);
    await expect(table).toBeVisible();

    // Validate table features
    if (options.sortable) {
      const headers = table.locator('th[data-sortable="true"]');
      await expect(headers).toHaveCount.greaterThan(0);
    }

    if (options.filterable) {
      await expect(table.locator('[data-testid="table-filter"]')).toBeVisible();
    }

    if (options.paginated) {
      await expect(page.locator('[data-testid="pagination"]')).toBeVisible();
    }

    // Validate data rows
    const rows = table.locator('tbody tr');
    await expect(rows).toHaveCount(expectedData.length);

    for (let i = 0; i < expectedData.length; i++) {
      const row = rows.nth(i);
      const rowData = expectedData[i];

      for (const [key, selector] of Object.entries(columnMapping)) {
        const cell = row.locator(selector);
        const expectedValue = String(rowData[key]);
        await expect(cell).toHaveText(expectedValue);
      }

      if (options.selectable) {
        await expect(row.locator('input[type="checkbox"]')).toBeVisible();
      }
    }
  }
}

// Usage with type safety
test('typed assertions example', async ({ page }) => {
  const userProfile: UserProfile = {
    id: 'USR-12345',
    name: 'John Doe',
    email: 'john.doe@example.com',
    role: 'admin',
    isActive: true,
    lastLogin: new Date('2024-01-15'),
  };

  await page.goto('/admin/users/USR-12345');
  await TypedAssertions.assertUserProfile(page, userProfile);

  // Type-safe table validation
  const userData = [
    { name: 'John Doe', email: 'john@example.com', role: 'admin' },
    { name: 'Jane Smith', email: 'jane@example.com', role: 'user' },
  ];

  await TypedAssertions.assertTable(
    page,
    '[data-testid="users-table"]',
    userData,
    {
      name: '[data-column="name"]',
      email: '[data-column="email"]',
      role: '[data-column="role"]',
    },
    { sortable: true, filterable: true, selectable: true },
  );
});
```

## Assertion Composition

### Composable Assertion Patterns

```typescript
// ✅ Composable assertion building blocks
export class ComposableAssertions {
  private assertions: Array<() => Promise<void>> = [];

  constructor(private page: Page) {}

  element(selector: string) {
    const element = this.page.locator(selector);
    return {
      shouldBeVisible: () => {
        this.assertions.push(() => expect(element).toBeVisible());
        return this;
      },
      shouldHaveText: (text: string) => {
        this.assertions.push(() => expect(element).toHaveText(text));
        return this;
      },
      shouldBeEnabled: () => {
        this.assertions.push(() => expect(element).toBeEnabled());
        return this;
      },
    };
  }

  url() {
    return {
      shouldMatch: (pattern: RegExp) => {
        this.assertions.push(() => expect(this.page).toHaveURL(pattern));
        return this;
      },
      shouldBe: (url: string) => {
        this.assertions.push(() => expect(this.page).toHaveURL(url));
        return this;
      },
    };
  }

  async run(): Promise<void> {
    for (const assertion of this.assertions) {
      await assertion();
    }
    this.assertions = []; // Clear for reuse
  }
}

// Usage
test('composable assertions example', async ({ page }) => {
  const assert = new ComposableAssertions(page);

  await page.goto('/dashboard');

  // Build up assertions
  assert
    .url()
    .shouldMatch(/\/dashboard/)
    .element('[data-testid="header"]')
    .shouldBeVisible()
    .element('[data-testid="welcome-message"]')
    .shouldHaveText('Welcome!')
    .element('[data-testid="logout-button"]')
    .shouldBeEnabled();

  // Execute all at once
  await assert.run();
});
```

## Test Utilities Integration

### Page Object Integration

```typescript
// ✅ Integration with page object pattern
export class PageObjectAssertions {
  static async assertPageObjectState<T extends Record<string, any>>(
    pageObject: T,
    expectedState: Partial<Record<keyof T, any>>,
  ): Promise<void> {
    for (const [property, expectedValue] of Object.entries(expectedState)) {
      const actualValue = pageObject[property];

      if (actualValue && typeof actualValue.textContent === 'function') {
        // It's a locator
        await expect(actualValue).toHaveText(expectedValue);
      } else if (actualValue && typeof actualValue.isVisible === 'function') {
        // It's a locator - check visibility
        if (expectedValue === true) {
          await expect(actualValue).toBeVisible();
        } else if (expectedValue === false) {
          await expect(actualValue).not.toBeVisible();
        }
      }
    }
  }
}

// Usage with page objects
test('page object assertions', async ({ page }) => {
  const loginPage = new LoginPage(page);
  await loginPage.goto();

  // Assert page object state
  await PageObjectAssertions.assertPageObjectState(loginPage, {
    usernameField: true, // Should be visible
    passwordField: true, // Should be visible
    submitButton: true, // Should be visible
    errorMessage: false, // Should not be visible
  });
});
```

This comprehensive guide provides patterns for creating maintainable, reusable custom assertions that encapsulate complex validation logic and integrate well with your existing test architecture.
