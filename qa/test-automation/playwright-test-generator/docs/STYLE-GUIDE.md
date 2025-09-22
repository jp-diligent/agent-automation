# ⌚️ Playwright Test Style Guide

## 🎯 Overview

This style guide defines the coding standards, patterns, and best practices for writing maintainable and consistent Playwright tests in this project. All test code MUST strictly adhere to these guidelines.

## 📁 File Structure & Organization

### Test File Naming

- **Test files**: Use `.spec.ts` extension
- **Naming convention**: `featureName.spec.ts` (camelCase)
- **Examples**: `userProfile.spec.ts`, `productCatalog.spec.ts`, `checkout.spec.ts`

### Directory Structure

```
tests/
├── authentication/     # Authentication tests
├── profile/           # User profile tests
├── catalog/           # Product catalog tests
├── orders/            # Order management tests
├── admin/             # Administrative tests
├── api/               # API tests
└── [feature]/         # Feature-specific tests
```

### Page Object Structure

```
pages/
├── app.page.ts        # Main application page object
├── authentication/    # Authentication page objects
├── profile/           # User profile page objects
├── catalog/           # Product catalog page objects
├── orders/            # Order management page objects
└── [feature]/         # Feature-specific page objects
```

## 🏗️ Test Structure Requirements

### Import Statement

```typescript
import { expect, test } from 'fixtures/fixtures';
```

### Authentication Usage

```typescript
// ❌ WRONG: Don't use admin auth unless explicitly required
test.use({ storageState: 'path/to/admin-auth.json' });

// ✅ CORRECT: Only use when admin privileges are needed
// Use admin auth only for admin-specific features
```

### Test Organization

```typescript
test.describe('Feature Name', () => {
  test.beforeEach(async ({ app }) => {
    // Setup steps using test.step()
  });

  test('should perform specific action', { tag: ['@prod'] }, async ({ app }) => {
    // Test implementation
  });
});
```

## 🎭 Test Step Structure - MANDATORY

### ✅ REQUIRED: Use test.step() for ALL Actions

Every test action MUST be wrapped in `test.step()` with descriptive names:

```typescript
test('should verify profile page functionality', async ({ app }) => {
  await test.step('Navigate to user profile', async () => {
    await app.homePage.goto();
    await app.homePage.clickOnProfileLink();
  });

  await test.step('Validate profile page elements are visible', async () => {
    await expect(app.profilePage.profileTitle).toBeVisible();
    await expect(app.profilePage.settingsSection).toBeVisible();
  });

  await test.step('Update profile information', async () => {
    await app.profilePage.clickOnEditButton();
    expect(app.page.url()).toContain('profile/edit');
  });
});
```

### 🚫 FORBIDDEN: Plain Comments for Steps

```typescript
// ❌ WRONG: Don't use plain comments for test steps
// Step 1: Navigate to profile page
await app.homePage.goto();

// Step 2: Click on profile link
await app.homePage.clickOnProfileLink();

// ✅ CORRECT: Use test.step() instead
await test.step('Navigate to profile page', async () => {
  await app.homePage.goto();
  await app.homePage.clickOnProfileLink();
});
```

### Step Naming Conventions

- **Descriptive**: Clearly describe what the step accomplishes
- **Action-oriented**: Start with verbs (Navigate, Validate, Click, Fill, etc.)
- **Specific**: Avoid generic names like "Step 1" or "Do something"

**Examples of Good Step Names:**

```typescript
await test.step('Navigate to product catalog', async () => { ... });
await test.step('Create new user account with valid data', async () => { ... });
await test.step('Validate successful account creation', async () => { ... });
await test.step('Verify error message for invalid input', async () => { ... });
```

## 💉 Page Object Model Requirements

### Class Structure

```typescript
import { type Locator, type Page } from '@playwright/test';

export class FeaturePage {
  readonly page: Page;
  readonly elementName: Locator;
  readonly anotherElement: Locator;

  constructor(page: Page) {
    this.page = page;
    this.elementName = page.getByRole('button', { name: 'Element Name' });
    this.anotherElement = page.locator('#element-id');
  }

  async performAction() {
    await this.elementName.click();
  }

  async getElementText() {
    return await this.elementName.textContent();
  }
}
```

### Method Design Patterns

- **Action methods**: Return `Promise<void>` for actions
- **Getter methods**: Return values for data extraction
- **Validation methods**: Can return boolean or be used with assertions

### Navigation Methods

```typescript
// ✅ CORRECT: Navigation methods in page objects
async goto() {
  await this.page.goto('/profile');
  await this.page.waitForLoadState('networkidle');
}

// ❌ WRONG: Hardcoded URLs in tests
await page.goto('https://example.com/profile');
```

## 🎯 Assertion Patterns

### Visibility Assertions

```typescript
// ✅ CORRECT: Use semantic assertions
await expect(app.profilePage.profileTitle).toBeVisible();
await expect(app.catalogPage.productList).toBeVisible();
```

### Value Assertions

```typescript
// ✅ CORRECT: Assert on expected values
await expect(app.profilePage.firstName).toHaveValue(expectedValue);
await expect(app.catalogPage.selectedCategory).toHaveValue('electronics');
```

### URL Assertions

```typescript
// ✅ CORRECT: Use expect with URL validation
expect(app.page.url()).toContain('profile/edit');
expect(app.page.url()).toContain('catalog');
```

### Grouped Assertions

```typescript
await test.step('Validate all required elements are present', async () => {
  await expect(app.profilePage.profileTitle).toBeVisible();
  await expect(app.profilePage.personalInfoSection).toBeVisible();
  await expect(app.profilePage.settingsSection).toBeVisible();
  await expect(app.profilePage.preferencesSection).toBeVisible();
});
```

## � Comment Guidelines - MINIMIZE UNNECESSARY COMMENTS

### ✅ When to Use Comments

Comments should be **rare** and only used for:

1. **Complex Business Logic**: When the code implements non-obvious business rules
2. **Workarounds**: When dealing with known browser/application limitations
3. **Technical Explanations**: When using advanced Playwright features
4. **Temporary Notes**: TODO comments for future improvements

```typescript
// ✅ CORRECT: Comment for complex business logic
await test.step('Validate subscription tier access', async () => {
  // Premium users get 50 searches, Basic users get 10
  const expectedCount = user.tier === 'premium' ? 50 : 10;
  await expect(app.dashboardPage.searchCount).toHaveText(expectedCount.toString());
});

// ✅ CORRECT: Comment for workaround
await test.step('Handle browser-specific date picker', async () => {
  // Chrome requires clicking twice to open date picker
  if (app.page.context().browser()?.browserType().name() === 'chromium') {
    await app.formPage.dateField.click();
  }
  await app.formPage.selectDate('2025-08-19');
});
```

### �🚫 When NOT to Use Comments

**Avoid comments for:**

1. **Self-explanatory test steps** - `test.step()` names should be descriptive enough
2. **Obvious actions** - clicking buttons, filling forms, navigation
3. **Restating what the code does** - redundant explanations
4. **Step numbering** - avoid "Step 1:", "Step 2:" comments

```typescript
// ❌ WRONG: Unnecessary comments for obvious actions
await test.step('Navigate to profile page', async () => {
  // Navigate to home page first
  await app.homePage.goto();
  // Click on the profile link
  await app.homePage.clickOnProfileLink();
  // Verify we're on the profile page
  expect(app.page.url()).toContain('profile');
});

// ✅ CORRECT: Self-explanatory test steps without comments
await test.step('Navigate to profile page', async () => {
  await app.homePage.goto();
  await app.homePage.clickOnProfileLink();
  expect(app.page.url()).toContain('profile');
});

// ❌ WRONG: Redundant step numbering comments
// Step 1: Login to application
await test.step('Login to application', async () => {
  await app.loginPage.login(user.email, user.password);
});

// Step 2: Navigate to settings
await test.step('Navigate to settings', async () => {
  await app.dashboardPage.clickSettingsLink();
});

// ✅ CORRECT: No unnecessary step comments
await test.step('Login to application', async () => {
  await app.loginPage.login(user.email, user.password);
});

await test.step('Navigate to settings', async () => {
  await app.dashboardPage.clickSettingsLink();
});
```

### 📝 Comment Best Practices

1. **Write self-documenting code first** - prefer descriptive method and variable names
2. **Keep comments concise** - one line when possible
3. **Explain WHY, not WHAT** - the code shows what, comments explain why
4. **Update comments with code changes** - outdated comments are worse than no comments
5. **Remove TODO comments** before merging to main branch

```typescript
// ✅ CORRECT: Concise, explains WHY
await test.step('Create account with unique email', async () => {
  // Generate unique email to avoid conflicts with existing test data
  const uniqueEmail = `test-${Date.now()}@example.com`;
  await app.signupPage.fillEmail(uniqueEmail);
  await app.signupPage.clickCreateAccount();
});

// ❌ WRONG: Verbose, explains WHAT the code already shows
await test.step('Create account with unique email', async () => {
  // Generate a unique email address using current timestamp
  // This will create an email like test-1692345678901@example.com
  // We do this to make sure we don't conflict with existing users
  const uniqueEmail = `test-${Date.now()}@example.com`;
  // Fill the email field with the generated unique email
  await app.signupPage.fillEmail(uniqueEmail);
  // Click the create account button to submit the form
  await app.signupPage.clickCreateAccount();
});
```

### ❌ No Direct Page Interactions in Tests

```typescript
// ❌ WRONG: Direct locators in test steps
await app.page.getByRole('button', { name: 'Submit' }).click();
await app.page.locator('#input-field').fill('value');

// ✅ CORRECT: Use page object methods
await app.formPage.clickSubmitButton();
await app.formPage.fillInputField('value');
```

### ❌ No Hardcoded Timeouts

```typescript
// ❌ WRONG: Explicit timeouts
await page.waitForTimeout(5000);
await expect(element).toBeVisible({ timeout: 10000 });

// ✅ CORRECT: Rely on Playwright's auto-waiting
await expect(element).toBeVisible();
```

### ❌ No Try-Catch in Tests

```typescript
// ❌ WRONG: Error handling in tests
try {
  await element.click();
} catch (error) {
  await fallbackElement.click();
}

// ✅ CORRECT: Let tests fail cleanly
await element.click();
```

### ❌ No Conditional Logic

```typescript
// ❌ WRONG: Conditional test logic
if (await element.isVisible()) {
  await element.click();
} else {
  await alternativeElement.click();
}

// ✅ CORRECT: Deterministic test flow
await element.click();
```

### ❌ No Console.log for Validation

```typescript
// ❌ WRONG: Using console.log instead of assertions
const text = await element.textContent();
console.log('Element text:', text);

// ✅ CORRECT: Use proper assertions
await expect(element).toHaveText('expected text');
```

## 🏷️ Test Tags

### Tag Usage

```typescript
test('test description', { tag: ['@prod'] }, async ({ app }) => { ... });
test('test description', { tag: ['@prod', '@stage'] }, async ({ app }) => { ... });
```

### Available Tags

- `@prod`: Tests that run in production environment
- `@stage`: Tests that run in staging environment
- `@smoke`: Quick smoke tests
- `@regression`: Full regression test suite

## 📊 Test Data Management

### Using Test Data

```typescript
// ✅ CORRECT: Access test data through app fixture
await app.formPage.fillDescription(app.testData.product.description);
await app.formPage.fillPrice(app.testData.product.price);
```

### Test Data Structure

Test data should be centralized in `test_data/test.data.ts` and accessed via the app fixture.

## 🔧 Fixture Usage

### App Fixture Pattern

```typescript
test('test name', async ({ app }) => {
  // ✅ CORRECT: All interactions through app fixture
  await app.homePage.goto();
  await app.profilePage.clickOnEditButton();

  // ✅ CORRECT: Access page instance when needed
  expect(app.page.url()).toContain('expected-path');
});
```

## 📋 Code Quality Checklist

Before submitting any test code, ensure:

- [ ] **⌚️ All actions are wrapped in `test.step()` with descriptive names**
- [ ] **💉 No direct `app.page.locator()` or `app.page.getByRole()` calls in test steps**
- [ ] **💉 All UI interactions use page object methods**
- [ ] **🎯 All assertions use appropriate Playwright matchers**
- [ ] **🚫 No hardcoded timeouts, try-catch blocks, or conditional logic**
- [ ] **🔐 Admin authentication only used when explicitly required**
- [ ] **📁 File is in correct directory with proper naming**
- [ ] **🏷️ Appropriate test tags are applied**
- [ ] **📊 Test data accessed through app fixture**

## 🎨 Example Complete Test

```typescript
import { expect, test } from 'fixtures/fixtures';

test.describe('User Profile Management', () => {
  test.beforeEach(async ({ app }) => {
    await test.step('Navigate to home page and access profile', async () => {
      await app.homePage.goto();
      await expect(app.homePage.logo).toBeVisible();
      await app.homePage.clickOnProfileLink();
      expect(app.page.url()).toContain('profile');
    });
  });

  test('should display all profile sections correctly', { tag: ['@prod'] }, async ({ app }) => {
    await test.step('Validate profile page structure', async () => {
      await expect(app.profilePage.profileTitle).toBeVisible();
      await expect(app.profilePage.personalInfoSection).toBeVisible();
      await expect(app.profilePage.settingsSection).toBeVisible();
      await expect(app.profilePage.preferencesSection).toBeVisible();
    });

    await test.step('Verify user guide link functionality', async () => {
      const url = await app.profilePage.getUserGuideUrl();
      expect(url).toContain('user-guide');
    });
  });

  test('should navigate to profile edit successfully', { tag: ['@prod', '@stage'] }, async ({ app }) => {
    await test.step('Open profile edit page', async () => {
      await app.profilePage.clickOnEditButton();
      expect(app.page.url()).toContain('profile/edit');
    });

    await test.step('Validate edit page elements', async () => {
      await expect(app.profileEditPage.editTitle).toBeVisible();
      await expect(app.profileEditPage.saveButton).toBeVisible();
    });
  });
});
```

## 🚨 Enforcement

- **Code reviews MUST verify compliance** with this style guide
- **Tests that violate these patterns WILL BE REJECTED**
- **Use the emojis (⌚️💉🎯🚫🔐📁🏷️📊) when referencing sections** in code reviews and documentation
- **This style guide is the source of truth** for all Playwright test development

---

**Remember**: Consistency is key to maintainable test automation. Follow these patterns religiously to ensure our test suite remains robust, readable, and reliable.
