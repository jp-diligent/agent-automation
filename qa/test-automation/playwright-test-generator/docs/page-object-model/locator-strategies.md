# Locator Strategies

**Description**: Comprehensive guide to Playwright locator selection strategies, with priority order from most preferred semantic locators to least preferred complex selectors. Includes advanced patterns and real-world examples.

## Table of Contents

- [Priority Order](#priority-order)
- [Semantic Locators](#semantic-locators)
- [Test ID Strategy](#test-id-strategy)
- [CSS Selectors](#css-selectors)
- [Advanced Patterns](#advanced-patterns)
- [Locator Organization](#locator-organization)
- [Common Anti-Patterns](#common-anti-patterns)

## Priority Order

Use this hierarchy when selecting locators (most to least preferred):

1. **Semantic Locators** (getByRole, getByLabel, getByText)
2. **Test ID Attributes** (getByTestId)
3. **Unique ID Selectors** (#id)
4. **Complex CSS Selectors** (last resort)

## Semantic Locators

### getByRole (Highest Priority)

```typescript
// ✅ Excellent - Use ARIA roles when possible
readonly submitButton = page.getByRole('button', { name: 'Submit' });
readonly emailField = page.getByRole('textbox', { name: 'Email Address' });
readonly header = page.getByRole('heading', { name: 'Welcome' });
readonly navigation = page.getByRole('navigation');
readonly searchLink = page.getByRole('link', { name: 'Search' });

// With additional options
readonly submitButton = page.getByRole('button', { name: 'Submit', exact: true });
readonly disabledButton = page.getByRole('button', { name: 'Save', disabled: true });
```

### getByLabel

```typescript
// ✅ Very Good - Form elements with labels
readonly emailField = page.getByLabel('Email Address');
readonly passwordField = page.getByLabel('Password');
readonly rememberCheckbox = page.getByLabel('Remember me');

// With exact matching
readonly phoneField = page.getByLabel('Phone', { exact: true });
```

### getByText

```typescript
// ✅ Good - For unique text content
readonly saveButton = page.getByText('Save Changes', { exact: true });
readonly cancelLink = page.getByText('Cancel', { exact: true });
readonly welcomeMessage = page.getByText('Welcome to the application');

// Partial text matching
readonly notificationMessage = page.getByText('Successfully saved');
```

## Test ID Strategy

### Implementation

```typescript
// ✅ Very Good - When semantic locators aren't available
readonly userDropdown = page.getByTestId('user-dropdown');
readonly navigationMenu = page.getByTestId('nav-menu');
readonly dataTable = page.getByTestId('search-results-table');
```

### Naming Convention

Use kebab-case for consistency:

- `data-testid="user-profile-menu"`
- `data-testid="search-results-container"`
- `data-testid="error-notification"`

## CSS Selectors

### Unique IDs

```typescript
// ✅ Good - When elements have stable IDs
readonly searchInput = page.locator('#search-input');
readonly errorContainer = page.locator('#error-container');
readonly mainContent = page.locator('#main-content');
```

### Class-based Selectors

```typescript
// ✅ Acceptable - For stable, semantic classes
readonly errorMessage = page.locator('.error-message');
readonly loadingSpinner = page.locator('.loading-indicator');

// ⚠️ Use with caution - Multiple classes
readonly primaryButton = page.locator('.btn.btn-primary');
```

## Advanced Patterns

### Filtering

```typescript
// Filter by text content
readonly activeUsers = page.locator('[data-testid="user-row"]').filter({ hasText: 'Active' });
readonly specificModal = page.locator('.modal').filter({ has: page.getByText('Confirm Delete') });

// Filter by child elements
readonly userCardWithEmail = page
  .locator('[data-testid="user-card"]')
  .filter({ has: page.getByText('john@example.com') });
```

### Chaining

```typescript
// Combine multiple locator strategies
readonly userActions = page
  .locator('[data-testid="user-card"]')
  .filter({ hasText: 'John Doe' })
  .getByRole('button', { name: 'Actions' });

// Navigate through DOM hierarchy
readonly tableCell = page
  .getByRole('table')
  .getByRole('row', { name: 'John Doe' })
  .getByRole('cell')
  .nth(2);
```

### Indexed Selection

```typescript
// Use nth() for indexed elements
readonly secondMenuItem = page.getByRole('menuitem').nth(1);
readonly lastNotification = page.locator('.notification').last();
readonly firstTableRow = page.locator('table tbody tr').first();

// For dynamic lists
readonly dynamicItem = page.locator('[data-testid="list-item"]').nth(index);
```

## Locator Organization

### Functional Grouping

```typescript
export class HomePage {
  readonly page: Page;

  // Navigation elements
  readonly logo: Locator;
  readonly searchLink: Locator;
  readonly catalogLink: Locator;
  readonly userMenu: Locator;
  readonly logoutLink: Locator;

  // Search elements
  readonly searchField: Locator;
  readonly searchButton: Locator;
  readonly searchFilters: Locator;

  // Content panels
  readonly latestSearchesPanel: Locator;
  readonly savedReportsPanel: Locator;
  readonly catalogPanel: Locator;

  // Status indicators
  readonly loadingIndicator: Locator;
  readonly errorNotification: Locator;
  readonly successNotification: Locator;

  constructor(page: Page) {
    this.page = page;

    // Navigation - semantic locators first
    this.logo = page.getByRole('link', { name: 'Company Logo' });
    this.searchLink = page.getByRole('link', { name: 'Search' });
    this.userMenu = page.getByTestId('user-menu');

    // Search - mixed strategies based on availability
    this.searchField = page.getByLabel('Search term');
    this.searchButton = page.getByRole('button', { name: 'Search' });
    this.searchFilters = page.locator('#search-filters');

    // Content - filter patterns for precision
    this.latestSearchesPanel = page.locator('.panel').filter({ hasText: 'Latest Searches' });
    this.savedReportsPanel = page.locator('.panel').filter({ hasText: 'Saved Reports' });
  }
}
```

## Common Anti-Patterns

### ❌ Avoid These Patterns

```typescript
// ❌ Fragile nth-child selectors
readonly fragileElement = page.locator('div:nth-child(3) > .panel > .body > table');

// ❌ Deep CSS selector chains
readonly nestedElement = page.locator('.container .sidebar .menu .item .link');

// ❌ Brittle position-based selectors
readonly unreliableElement = page.locator('tr:first-child td:nth-child(2) a');

// ❌ Text selectors without exact matching for ambiguous text
readonly ambiguousButton = page.getByText('Save'); // Could match "Save Draft", "Save As", etc.

// ❌ XPath selectors (harder to maintain)
readonly xpathLocator = page.locator('//div[@class="content"]//span[contains(text(), "Error")]');
```

### ✅ Better Alternatives

```typescript
// ✅ Use semantic locators instead
readonly saveButton = page.getByRole('button', { name: 'Save Changes', exact: true });

// ✅ Use test IDs for complex elements
readonly complexElement = page.getByTestId('complex-widget-container');

// ✅ Use filter chains for precision
readonly specificRow = page
  .getByRole('row')
  .filter({ hasText: 'John Doe' });

// ✅ Use stable attributes
readonly stableElement = page.locator('[data-cy="user-profile"]');
```

## Best Practices Summary

1. **Start with semantic locators** - they're the most resilient to UI changes
2. **Use test IDs** when semantic options aren't available
3. **Avoid deep CSS selector chains** - they're brittle and hard to maintain
4. **Group locators logically** in your page objects
5. **Use exact text matching** when text content might be ambiguous
6. **Leverage filtering and chaining** for complex element selection
7. **Document complex locator strategies** with comments explaining why they're necessary

## Migration Strategy

When refactoring existing locators:

1. **Audit current locators** - identify fragile selectors
2. **Prioritize high-traffic pages** - start with most-used page objects
3. **Work with developers** to add test IDs where semantic locators aren't sufficient
4. **Test thoroughly** after locator changes
5. **Update gradually** - don't change everything at once
