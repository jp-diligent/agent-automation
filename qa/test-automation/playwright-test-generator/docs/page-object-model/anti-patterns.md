# Anti-Patterns

**Description**: Common mistakes and anti-patterns to avoid when implementing Page Object Model, with clear examples of what not to do and better alternatives. Essential reference for maintaining clean, maintainable test automation code.

## Table of Contents

- [Navigation Anti-Patterns](#navigation-anti-patterns)
- [Data Management Anti-Patterns](#data-management-anti-patterns)
- [Assertion Anti-Patterns](#assertion-anti-patterns)
- [Locator Anti-Patterns](#locator-anti-patterns)
- [Method Design Anti-Patterns](#method-design-anti-patterns)
- [Architecture Anti-Patterns](#architecture-anti-patterns)
- [Performance Anti-Patterns](#performance-anti-patterns)
- [Testing Anti-Patterns](#testing-anti-patterns)

## Navigation Anti-Patterns

### ❌ Hardcoded URLs in Tests

**Problem**: Direct navigation bypasses page object encapsulation and makes tests brittle.

```typescript
// ❌ BAD - Hardcoded URLs directly in test code
test('should navigate to dashboard', async ({ page }) => {
  await page.goto('https://example.com/dashboard'); // Brittle and hard to maintain
  await page.goto('/monitor/manage_lists'); // Bypasses page object

  // Test continues...
});

// ❌ BAD - Bypassing page object navigation methods
test('should access user profile', async ({ app }) => {
  await app.page.goto('https://example.com/users/123'); // Don't access page directly
  await app.userPage.fillUserForm(); // Inconsistent navigation
});
```

**✅ Better Approach**: Always use page object navigation methods

```typescript
// ✅ GOOD - Use page object navigation methods
test('should navigate to dashboard', async ({ app }) => {
  await app.dashboardPage.goto(); // Centralized navigation logic
  await app.monitorPage.goToManageLists(); // Consistent navigation pattern
});

// ✅ GOOD - Create navigation methods in page objects
export class DashboardPage {
  async goto(): Promise<void> {
    await this.page.goto('/dashboard');
    await this.page.waitForLoadState('domcontentloaded');
  }

  async gotoWithFilter(filter: string): Promise<void> {
    await this.page.goto(`/dashboard?filter=${encodeURIComponent(filter)}`);
    await this.page.waitForLoadState('domcontentloaded');
  }
}

export class UserProfilePage {
  async gotoUserProfile(userId: string): Promise<void> {
    await this.page.goto(`/users/${userId}`);
    await this.page.waitForLoadState('domcontentloaded');
  }
}
```

**Why This Matters**:

- Centralizes URL management in page objects
- Makes environment changes easier to handle
- Provides consistent waiting strategies
- Enables URL validation and error handling

## Data Management Anti-Patterns

### ❌ Storing Test Data in Page Objects

**Problem**: Page objects become data containers instead of interface abstractions.

```typescript
// ❌ BAD - Page object storing test data
export class LoginPage {
  // Don't store test data in page objects
  private readonly testUsers = {
    admin: { username: 'admin', password: 'pass123' },
    user: { username: 'user', password: 'userpass' },
  };

  private currentUser: string = ''; // Stateful page objects are problematic

  async loginAsAdmin(): Promise<void> {
    const admin = this.testUsers.admin; // Coupling data with page logic
    await this.login(admin.username, admin.password);
    this.currentUser = 'admin'; // Storing state
  }
}

// ❌ BAD - Hardcoded values in methods
export class SearchPage {
  async performDefaultSearch(): Promise<void> {
    await this.searchField.fill('John Doe'); // Hardcoded search term
    await this.categoryDropdown.selectOption('Individual'); // Hardcoded category
    await this.dateFromField.fill('2023-01-01'); // Hardcoded date
  }
}
```

**✅ Better Approach**: Pass data as parameters, use external data sources

```typescript
// ✅ GOOD - Accept data as parameters
export class LoginPage {
  async login(username: string, password: string): Promise<void> {
    await this.usernameField.fill(username);
    await this.passwordField.fill(password);
    await this.loginButton.click();
  }

  async loginWithCredentials(credentials: LoginCredentials): Promise<void> {
    await this.login(credentials.username, credentials.password);
  }
}

// ✅ GOOD - Flexible, parameterized methods
export class SearchPage {
  async performSearch(criteria: SearchCriteria): Promise<void> {
    await this.searchField.fill(criteria.query);

    if (criteria.category) {
      await this.categoryDropdown.selectOption(criteria.category);
    }

    if (criteria.dateRange) {
      await this.setDateRange(criteria.dateRange.from, criteria.dateRange.to);
    }

    await this.searchButton.click();
  }
}

// ✅ GOOD - External test data management
// test-data/users.data.ts
export const TEST_USERS = {
  ADMIN: { username: 'admin@example.com', password: 'AdminPass123!' },
  REGULAR_USER: { username: 'user@example.com', password: 'UserPass123!' },
} as const;

// In tests
test('admin login flow', async ({ app }) => {
  await app.loginPage.goto();
  await app.loginPage.loginWithCredentials(TEST_USERS.ADMIN);
});
```

## Assertion Anti-Patterns

### ❌ Assertions in Page Objects

**Problem**: Mixing test logic with page abstraction violates separation of concerns.

```typescript
// ❌ BAD - Assertions in page objects
export class LoginPage {
  async login(username: string, password: string): Promise<void> {
    await this.usernameField.fill(username);
    await this.passwordField.fill(password);
    await this.loginButton.click();

    // Don't assert in page objects
    await expect(this.page).toHaveURL('/dashboard');
    await expect(this.welcomeMessage).toBeVisible();
    await expect(this.errorMessage).toBeHidden();
  }

  async uploadFile(filePath: string): Promise<void> {
    await this.fileInput.setInputFiles(filePath);

    // Assertions don't belong here
    await expect(this.uploadSuccessMessage).toBeVisible();
    await expect(this.uploadSuccessMessage).toContainText('Upload successful');
  }

  async validateUserIsLoggedIn(): Promise<void> {
    // This method name suggests validation but does assertions
    await expect(this.userMenu).toBeVisible();
    await expect(this.logoutButton).toBeVisible();
  }
}
```

**✅ Better Approach**: Return data for tests to assert

```typescript
// ✅ GOOD - Return data, let tests handle assertions
export class LoginPage {
  async login(username: string, password: string): Promise<string> {
    await this.usernameField.fill(username);
    await this.passwordField.fill(password);
    await this.loginButton.click();
    await this.page.waitForLoadState('networkidle');

    return this.page.url(); // Return URL for test to validate
  }

  async uploadFile(filePath: string): Promise<{
    success: boolean;
    message: string;
    fileName?: string;
  }> {
    await this.fileInput.setInputFiles(filePath);

    try {
      await this.uploadSuccessMessage.waitFor({ state: 'visible', timeout: 10000 });
      const message = (await this.uploadSuccessMessage.textContent()) || '';
      const fileName = (await this.uploadedFileName.textContent()) || '';

      return { success: true, message, fileName };
    } catch {
      const errorMessage = await this.getErrorMessage();
      return { success: false, message: errorMessage || 'Upload failed' };
    }
  }

  async getUserMenuState(): Promise<{
    isVisible: boolean;
    userName?: string;
    hasLogoutOption: boolean;
  }> {
    const isVisible = await this.userMenu.isVisible();
    const userName = isVisible ? await this.userNameElement.textContent() : undefined;
    const hasLogoutOption = isVisible ? await this.logoutButton.isVisible() : false;

    return { isVisible, userName, hasLogoutOption };
  }
}

// ✅ GOOD - Assertions in tests where they belong
test('user login flow', async ({ app }) => {
  const loginPage = app.loginPage;

  await loginPage.goto();
  const currentUrl = await loginPage.login('user@example.com', 'password');

  // Assertions belong in tests
  expect(currentUrl).toBe('/dashboard');

  const userMenuState = await loginPage.getUserMenuState();
  expect(userMenuState.isVisible).toBe(true);
  expect(userMenuState.userName).toContain('user@example.com');
});
```

## Locator Anti-Patterns

### ❌ Fragile Complex Selectors

**Problem**: Complex, brittle selectors that break with minor UI changes.

```typescript
// ❌ BAD - Fragile, complex selectors
export class FragilePage {
  // Overly specific nth-child selectors
  readonly fragileButton = this.page.locator(
    'div:nth-child(3) > .panel > .body > table > tbody > tr:first-child > td:nth-child(2) > button',
  );

  // Deep CSS selector chains
  readonly nestedElement = this.page.locator('.container .sidebar .menu .submenu .item .link');

  // Position-dependent selectors
  readonly unreliableElement = this.page.locator('table tr:first-child td:last-child a');

  // XPath selectors (harder to maintain)
  readonly xpathElement = this.page.locator('//div[@class="content"]//span[contains(text(), "Error")]');

  // Text selectors without exact matching
  readonly ambiguousButton = this.page.getByText('Save'); // Could match "Save Draft", "Save As", etc.
}
```

**✅ Better Approach**: Use semantic, stable locators

```typescript
// ✅ GOOD - Semantic, stable locators
export class RobustPage {
  // Semantic locators (most preferred)
  readonly saveButton = this.page.getByRole('button', { name: 'Save Changes', exact: true });
  readonly emailField = this.page.getByLabel('Email Address');
  readonly navigation = this.page.getByRole('navigation');

  // Test IDs when semantic options aren't available
  readonly complexWidget = this.page.getByTestId('user-dashboard-widget');
  readonly dataTable = this.page.getByTestId('search-results-table');

  // Unique IDs for specific elements
  readonly searchInput = this.page.locator('#search-input');
  readonly errorContainer = this.page.locator('#error-container');

  // Filter patterns for precision
  readonly activeUserRow = this.page.locator('[data-testid="user-row"]').filter({ hasText: 'Active' });

  readonly specificModal = this.page.locator('.modal').filter({ has: this.page.getByText('Confirm Delete') });
}
```

### ❌ Mutable Locators

**Problem**: Locators that can be accidentally modified after creation.

```typescript
// ❌ BAD - Mutable properties
export class MutablePage {
  page: Page; // Not readonly - can be reassigned
  searchField: Locator; // Not readonly - can be reassigned

  constructor(page: Page) {
    this.page = page;
    this.searchField = page.getByPlaceholder('Search');
  }

  async someMethod(): Promise<void> {
    // Accidentally reassigning locator
    this.searchField = this.page.locator('#different-search'); // Oops!
  }
}
```

**✅ Better Approach**: Use readonly properties

```typescript
// ✅ GOOD - Immutable properties
export class ImmutablePage {
  readonly page: Page;
  readonly searchField: Locator;
  readonly searchButton: Locator;

  constructor(page: Page) {
    this.page = page;
    this.searchField = page.getByPlaceholder('Search');
    this.searchButton = page.getByRole('button', { name: 'Search' });
  }

  // Properties cannot be accidentally reassigned
}
```

## Method Design Anti-Patterns

### ❌ God Methods

**Problem**: Methods that do too many things, violating single responsibility principle.

```typescript
// ❌ BAD - Method doing too many things
export class OverComplexPage {
  async doEverything(userData: any): Promise<void> {
    // Navigation
    await this.page.goto('/users');
    await this.page.waitForLoadState('domcontentloaded');

    // User creation
    await this.addUserButton.click();
    await this.userNameField.fill(userData.name);
    await this.emailField.fill(userData.email);
    await this.roleDropdown.selectOption(userData.role);

    // Form validation
    if (await this.nameError.isVisible()) {
      throw new Error('Name validation failed');
    }

    // Submission
    await this.submitButton.click();
    await this.successMessage.waitFor({ state: 'visible' });

    // Navigation to user list
    await this.backButton.click();

    // Search for created user
    await this.searchField.fill(userData.email);
    await this.searchButton.click();

    // Validation
    await expect(this.searchResults.first()).toContainText(userData.name);
  }
}
```

**✅ Better Approach**: Break into smaller, focused methods

```typescript
// ✅ GOOD - Focused, single-responsibility methods
export class WellStructuredPage {
  async goto(): Promise<void> {
    await this.page.goto('/users');
    await this.page.waitForLoadState('domcontentloaded');
  }

  async openAddUserForm(): Promise<void> {
    await this.addUserButton.click();
    await this.userForm.waitFor({ state: 'visible' });
  }

  async fillUserForm(userData: UserFormData): Promise<void> {
    await this.userNameField.fill(userData.name);
    await this.emailField.fill(userData.email);
    await this.roleDropdown.selectOption(userData.role);
  }

  async submitUserForm(): Promise<void> {
    await this.submitButton.click();
  }

  async waitForUserCreationSuccess(): Promise<void> {
    await this.successMessage.waitFor({ state: 'visible' });
  }

  async getFormValidationErrors(): Promise<string[]> {
    const errors: string[] = [];

    if (await this.nameError.isVisible()) {
      errors.push((await this.nameError.textContent()) || '');
    }

    if (await this.emailError.isVisible()) {
      errors.push((await this.emailError.textContent()) || '');
    }

    return errors;
  }

  async searchForUser(email: string): Promise<void> {
    await this.searchField.fill(email);
    await this.searchButton.click();
  }

  async getSearchResults(): Promise<string[]> {
    return await this.searchResults.allTextContents();
  }

  // Composite method that orchestrates smaller methods
  async createUser(userData: UserFormData): Promise<void> {
    await this.openAddUserForm();
    await this.fillUserForm(userData);
    await this.submitUserForm();
    await this.waitForUserCreationSuccess();
  }
}
```

### ❌ Inconsistent Method Naming

**Problem**: Methods with unclear, inconsistent names that don't indicate their purpose.

```typescript
// ❌ BAD - Inconsistent, unclear naming
export class PoorlyNamedPage {
  async go(): Promise<void> {} // Go where?
  async doLogin(): Promise<void> {} // "do" prefix is unclear
  async submit(): Promise<void> {} // Submit what?
  async check(): Promise<boolean> {} // Check what? Return type unclear
  async get(): Promise<string> {} // Get what?
  async clickOn(): Promise<void> {} // Click on what?
  async handle(): Promise<void> {} // Handle what?
  async process(): Promise<void> {} // Process what?
}
```

**✅ Better Approach**: Clear, consistent naming patterns

```typescript
// ✅ GOOD - Clear, consistent method names
export class ClearlyNamedPage {
  // Navigation methods
  async goto(): Promise<void> {}
  async gotoUserProfile(userId: string): Promise<void> {}

  // Form filling methods
  async fillUsername(username: string): Promise<void> {}
  async fillPassword(password: string): Promise<void> {}

  // Click actions with specific targets
  async clickSubmitButton(): Promise<void> {}
  async clickCancelButton(): Promise<void> {}
  async clickSaveAndContinueButton(): Promise<void> {}

  // State checking methods with boolean returns
  async isFormValid(): Promise<boolean> {}
  async isSubmitButtonEnabled(): Promise<boolean> {}
  async isErrorMessageVisible(): Promise<boolean> {}

  // Data retrieval methods
  async getErrorMessage(): Promise<string | null> {}
  async getUserDisplayName(): Promise<string> {}
  async getValidationErrors(): Promise<string[]> {}

  // Wait methods
  async waitForPageLoad(): Promise<void> {}
  async waitForFormSubmission(): Promise<void> {}
}
```

## Architecture Anti-Patterns

### ❌ Deep Inheritance Chains

**Problem**: Complex inheritance hierarchies that become hard to maintain.

```typescript
// ❌ BAD - Too many inheritance levels
abstract class BaseBasePage {
  abstract page: Page;
}

abstract class BasePage extends BaseBasePage {
  abstract goto(): Promise<void>;
}

abstract class FormBasePage extends BasePage {
  abstract submitForm(): Promise<void>;
}

abstract class UserFormBasePage extends FormBasePage {
  abstract fillUserData(): Promise<void>;
}

class LoginPage extends UserFormBasePage {
  // Too deep inheritance
  // Implementation
}
```

**✅ Better Approach**: Favor composition over inheritance

```typescript
// ✅ GOOD - Composition-based design
interface NavigationUtils {
  goto(path: string): Promise<void>;
  waitForPageLoad(): Promise<void>;
}

interface FormUtils {
  fillField(locator: Locator, value: string): Promise<void>;
  submitForm(submitButton: Locator): Promise<void>;
  getValidationErrors(): Promise<string[]>;
}

class NavigationHelper implements NavigationUtils {
  constructor(private page: Page) {}

  async goto(path: string): Promise<void> {
    await this.page.goto(path);
    await this.waitForPageLoad();
  }

  async waitForPageLoad(): Promise<void> {
    await this.page.waitForLoadState('domcontentloaded');
  }
}

class FormHelper implements FormUtils {
  constructor(private page: Page) {}

  async fillField(locator: Locator, value: string): Promise<void> {
    await locator.fill(value);
  }

  async submitForm(submitButton: Locator): Promise<void> {
    await submitButton.click();
  }

  async getValidationErrors(): Promise<string[]> {
    // Implementation
    return [];
  }
}

// Use composition instead of inheritance
export class LoginPage {
  readonly page: Page;
  readonly usernameField: Locator;
  readonly passwordField: Locator;
  readonly submitButton: Locator;

  private navigationHelper: NavigationHelper;
  private formHelper: FormHelper;

  constructor(page: Page) {
    this.page = page;
    this.navigationHelper = new NavigationHelper(page);
    this.formHelper = new FormHelper(page);

    this.usernameField = page.getByLabel('Username');
    this.passwordField = page.getByLabel('Password');
    this.submitButton = page.getByRole('button', { name: 'Sign In' });
  }

  async goto(): Promise<void> {
    await this.navigationHelper.goto('/login');
  }

  async login(username: string, password: string): Promise<void> {
    await this.formHelper.fillField(this.usernameField, username);
    await this.formHelper.fillField(this.passwordField, password);
    await this.formHelper.submitForm(this.submitButton);
  }
}
```

## Performance Anti-Patterns

### ❌ Hardcoded Waits and Timeouts

**Problem**: Using arbitrary waits instead of Playwright's smart waiting.

```typescript
// ❌ BAD - Hardcoded timeouts and explicit waits
export class SlowPage {
  async performAction(): Promise<void> {
    await this.button.click();
    await this.page.waitForTimeout(5000); // Never use this!
  }

  async waitForElement(): Promise<void> {
    await this.page.waitForTimeout(2000); // Arbitrary wait
    await this.element.click();
  }

  async retryAction(): Promise<void> {
    try {
      await this.unreliableButton.click({ timeout: 1000 });
    } catch {
      await this.page.waitForTimeout(3000); // Bad retry pattern
      await this.unreliableButton.click();
    }
  }

  async loadPageSlowly(): Promise<void> {
    await this.page.goto('/dashboard');
    await this.page.waitForTimeout(10000); // Excessive wait
  }
}
```

**✅ Better Approach**: Use Playwright's built-in smart waiting

```typescript
// ✅ GOOD - Smart waiting and proper retry patterns
export class FastPage {
  async performAction(): Promise<void> {
    await this.button.click(); // Auto-waits for actionability
    await this.page.waitForLoadState('networkidle'); // Wait for actual network activity
  }

  async waitForElement(): Promise<void> {
    await this.element.waitFor({ state: 'visible' }); // Wait for specific condition
    await this.element.click();
  }

  async retryAction(): Promise<void> {
    // Let Playwright handle retries automatically
    await this.unreliableButton.click({
      timeout: 10000, // Reasonable timeout for auto-retry
    });
  }

  async loadPage(): Promise<void> {
    await this.page.goto('/dashboard');
    await this.page.waitForLoadState('domcontentloaded'); // Wait for DOM
    await this.loadingIndicator.waitFor({ state: 'hidden' }); // Wait for loading to finish
  }

  async waitForSearchResults(): Promise<void> {
    // Wait for specific condition instead of arbitrary time
    await this.page.waitForFunction(() => {
      const results = document.querySelectorAll('[data-testid="search-result"]');
      return results.length > 0;
    });
  }
}
```

### ❌ Inefficient Locator Creation

**Problem**: Creating new locators repeatedly instead of reusing them.

```typescript
// ❌ BAD - Creating locators repeatedly
export class InefficientPage {
  async getUserCount(): Promise<number> {
    return await this.page.locator('[data-testid="user-card"]').count();
  }

  async getUserNames(): Promise<string[]> {
    return await this.page.locator('[data-testid="user-card"] .name').allTextContents();
  }

  async clickFirstUser(): Promise<void> {
    await this.page.locator('[data-testid="user-card"]').first().click();
  }
}
```

**✅ Better Approach**: Reuse locators for efficiency

```typescript
// ✅ GOOD - Reuse locators
export class EfficientPage {
  readonly page: Page;
  readonly userCards: Locator;
  readonly userNames: Locator;

  constructor(page: Page) {
    this.page = page;
    this.userCards = page.locator('[data-testid="user-card"]');
    this.userNames = this.userCards.locator('.name');
  }

  async getUserCount(): Promise<number> {
    return await this.userCards.count();
  }

  async getUserNames(): Promise<string[]> {
    return await this.userNames.allTextContents();
  }

  async clickFirstUser(): Promise<void> {
    await this.userCards.first().click();
  }
}
```

## Testing Anti-Patterns

### ❌ Page Objects with Test Logic

**Problem**: Mixing test orchestration with page abstraction.

```typescript
// ❌ BAD - Test logic in page objects
export class TestLogicInPageObject {
  async testCompleteUserFlow(): Promise<void> {
    // This is test logic, not page object logic
    await this.goto();
    await this.login('user', 'pass');

    // Test assertions don't belong here
    await expect(this.welcomeMessage).toBeVisible();

    await this.navigateToProfile();
    await this.updateProfile('New Name');

    // More test assertions
    await expect(this.successMessage).toContainText('Profile updated');

    await this.logout();
    await expect(this.loginForm).toBeVisible();
  }
}
```

**✅ Better Approach**: Keep page objects focused on page interaction

```typescript
// ✅ GOOD - Page objects handle page interaction only
export class ProperPageObject {
  async goto(): Promise<void> {
    await this.page.goto('/login');
  }

  async login(username: string, password: string): Promise<string> {
    await this.usernameField.fill(username);
    await this.passwordField.fill(password);
    await this.loginButton.click();
    return this.page.url(); // Return data for test to validate
  }

  async getWelcomeMessage(): Promise<string> {
    return (await this.welcomeMessage.textContent()) || '';
  }

  async updateProfile(name: string): Promise<void> {
    await this.nameField.fill(name);
    await this.saveButton.click();
  }

  async getSuccessMessage(): Promise<string> {
    return (await this.successMessage.textContent()) || '';
  }
}

// Test logic belongs in test files
test('complete user flow', async ({ page }) => {
  const userPage = new ProperPageObject(page);

  await userPage.goto();
  const currentUrl = await userPage.login('user', 'pass');
  expect(currentUrl).toContain('/dashboard');

  const welcomeMessage = await userPage.getWelcomeMessage();
  expect(welcomeMessage).toContain('Welcome');

  await userPage.updateProfile('New Name');
  const successMessage = await userPage.getSuccessMessage();
  expect(successMessage).toContain('Profile updated');
});
```

Following these anti-pattern guidelines helps maintain clean, maintainable, and reliable Page Object Model implementations that scale well with your test suite.
