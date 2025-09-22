# Method Design

**Description**: Comprehensive guide to designing effective page object methods, including naming conventions, categorization, TypeScript integration, and common patterns for complex workflows. Emphasizes clear, maintainable, and reusable method design.

## Table of Contents

- [Naming Conventions](#naming-conventions)
- [Method Categories](#method-categories)
- [TypeScript Integration](#typescript-integration)
- [Complex Business Logic](#complex-business-logic)
- [Common Patterns](#common-patterns)
- [Error Handling](#error-handling)
- [Best Practices](#best-practices)

## Naming Conventions

Use consistent, descriptive method names that clearly indicate their purpose:

```typescript
// ✅ Good - Consistent method naming patterns

// Navigation methods
async goto(): Promise<void>                    // Navigation to page
async gotoSpecificSection(): Promise<void>     // Navigation to page section
async gotoUserProfile(userId: string): Promise<void>

// Form filling methods
async fillUsername(username: string): Promise<void>
async fillEmail(email: string): Promise<void>
async fillPassword(password: string): Promise<void>

// Click actions with specific targets
async clickSubmitButton(): Promise<void>
async clickSaveButton(): Promise<void>
async clickCancelLink(): Promise<void>

// Selection and interaction methods
async selectFromDropdown(value: string): Promise<void>
async checkCheckbox(): Promise<void>
async uncheckCheckbox(): Promise<void>

// Getter methods for data retrieval
async getErrorMessage(): Promise<string | null>
async getUserDisplayName(): Promise<string>
async getResultCount(): Promise<number>

// State checking methods
async isButtonEnabled(): Promise<boolean>
async isFormVisible(): Promise<boolean>
async isLoading(): Promise<boolean>

// Wait methods
async waitForPageLoad(): Promise<void>
async waitForErrorToAppear(): Promise<void>
```

### ❌ Avoid These Naming Patterns

```typescript
// ❌ Bad - Unclear or inconsistent naming
async doLogin(): Promise<void>                 // Unclear action
async submit(): Promise<void>                  // Too generic
async click(): Promise<void>                   // Doesn't specify what to click
async check(): Promise<boolean>                // Ambiguous return type
async get(): Promise<string>                   // Too generic
```

## Method Categories

### 1. Navigation Methods

```typescript
// Basic navigation
async goto(): Promise<void> {
  await this.page.goto('/dashboard');
  await this.page.waitForLoadState('domcontentloaded');
}

// Navigation with parameters
async gotoUserProfile(userId: string): Promise<void> {
  await this.page.goto(`/users/${userId}`);
  await this.page.waitForLoadState('networkidle');
}

// Navigation with validation
async gotoWithValidation(): Promise<string> {
  await this.page.goto('/dashboard');
  await this.page.waitForLoadState('domcontentloaded');
  return this.page.url(); // Return URL for test validation
}
```

### 2. Action Methods

```typescript
// Simple actions
async clickSubmitButton(): Promise<void> {
  await this.submitButton.click();
}

// Actions with waiting
async clickSubmitAndWait(): Promise<void> {
  await this.submitButton.click();
  await this.page.waitForLoadState('networkidle');
}

// Actions with validation
async clickSubmitWithValidation(): Promise<string> {
  await this.submitButton.click();
  await this.page.waitForLoadState('networkidle');
  return this.page.url(); // Return for test to validate navigation
}
```

### 3. Form Interaction Methods

```typescript
// Individual field methods
async fillUsername(username: string): Promise<void> {
  await this.usernameField.fill(username);
}

async fillPassword(password: string): Promise<void> {
  await this.passwordField.fill(password);
}

// Composite form methods
async fillLoginForm(username: string, password: string): Promise<void> {
  await this.fillUsername(username);
  await this.fillPassword(password);
}

// Complete workflow methods
async login(username: string, password: string): Promise<void> {
  await this.fillLoginForm(username, password);
  await this.clickSubmitButton();
  await this.page.waitForLoadState('networkidle');
}
```

### 4. Data Retrieval Methods

```typescript
// Simple text retrieval
async getErrorMessage(): Promise<string | null> {
  return await this.errorMessage.textContent();
}

async getUserDisplayName(): Promise<string> {
  const text = await this.userNameElement.textContent();
  return text?.trim() || '';
}

// Parsed data retrieval
async getResultCount(): Promise<number> {
  const countText = await this.resultCountElement.textContent();
  const match = countText?.match(/(\d+) results/);
  return match ? parseInt(match[1], 10) : 0;
}

// List data retrieval
async getSearchResults(): Promise<string[]> {
  const results = await this.searchResultsList.allTextContents();
  return results.filter((result) => result.trim().length > 0);
}

// Complex data extraction
async extractIDFromURL(url: string): Promise<string> {
  if (url) {
    const parts = url.split('/');
    return parts[parts.length - 2] || 'No ID found';
  }
  return 'No URL found';
}
```

### 5. State Checking Methods

```typescript
// Visibility checks
async isLoginFormVisible(): Promise<boolean> {
  return await this.loginForm.isVisible();
}

// Enabled/disabled state
async isSubmitButtonEnabled(): Promise<boolean> {
  return await this.submitButton.isEnabled();
}

// Content validation
async hasErrorMessage(): Promise<boolean> {
  return await this.errorMessage.isVisible();
}

// Complex state checks
async isPageFullyLoaded(): Promise<boolean> {
  const isVisible = await this.mainContent.isVisible();
  const isLoadingHidden = await this.loadingIndicator.isHidden();
  return isVisible && isLoadingHidden;
}
```

### 6. Wait Utility Methods

```typescript
// Basic wait methods
async waitForPageLoad(): Promise<void> {
  await this.page.waitForLoadState('domcontentloaded');
}

// Element-specific waits
async waitForErrorToAppear(): Promise<void> {
  await this.errorMessage.waitFor({ state: 'visible' });
}

async waitForLoadingToComplete(): Promise<void> {
  await this.loadingIndicator.waitFor({ state: 'hidden' });
}

// Custom wait conditions
async waitForSearchResults(): Promise<void> {
  await this.page.waitForFunction(() => {
    const resultsContainer = document.querySelector('[data-testid="search-results"]');
    return resultsContainer && resultsContainer.children.length > 0;
  });
}
```

## TypeScript Integration

### Proper Type Definitions

```typescript
// Define interfaces for complex data structures
interface LoginCredentials {
  username: string;
  password: string;
}

interface SearchFilters {
  dateRange?: {
    from: Date;
    to: Date;
  };
  category?: string;
  status?: "active" | "inactive" | "pending";
}

interface UserProfile {
  id: string;
  name: string;
  email: string;
  role: string;
}

export class SearchPage {
  readonly page: Page;

  // Typed method parameters
  async performSearch(query: string, filters?: SearchFilters): Promise<void> {
    await this.searchField.fill(query);

    if (filters?.dateRange) {
      await this.setDateRange(filters.dateRange.from, filters.dateRange.to);
    }

    if (filters?.category) {
      await this.selectCategory(filters.category);
    }

    await this.searchButton.click();
  }

  // Explicit return types
  async getUserProfile(): Promise<UserProfile> {
    const name = (await this.userNameElement.textContent()) || "";
    const email = (await this.userEmailElement.textContent()) || "";
    const role = (await this.userRoleElement.textContent()) || "";
    const id = await this.extractIDFromURL(this.page.url());

    return { id, name, email, role };
  }
}
```

### Optional Parameters and Defaults

```typescript
// Methods with optional parameters
async createUser(
  name: string,
  email: string,
  options: {
    role?: 'admin' | 'user' | 'guest';
    sendEmail?: boolean;
    department?: string;
  } = {}
): Promise<void> {
  await this.fillUserName(name);
  await this.fillUserEmail(email);

  if (options.role) {
    await this.selectRole(options.role);
  }

  if (options.sendEmail !== false) { // Default to true
    await this.checkSendEmailCheckbox();
  }

  if (options.department) {
    await this.selectDepartment(options.department);
  }

  await this.clickCreateButton();
}
```

## Complex Business Logic

### Multi-step Workflows

```typescript
// Encapsulate complex business processes
async createMonitorList(
  name: string,
  options: {
    negativeNews?: boolean;
    financialNews?: boolean;
    watchlist?: boolean;
    peps?: boolean;
  } = {}
): Promise<void> {
  // Add timestamp to ensure uniqueness
  await this.fillListName(name + ' ' + Date.now());

  // Configure options based on parameters
  if (options.negativeNews) {
    await this.negativeNewsCheckbox.check();
  }
  if (options.financialNews) {
    await this.financialNewsCheckbox.check();
  }
  if (options.watchlist) {
    await this.watchlistCheckbox.check();
  }
  if (options.peps) {
    await this.pepsCheckbox.check();
  }

  await this.clickCreateListButton();
  await this.page.waitForLoadState('networkidle');
}

// Modal handling workflows
async createReportWithModal(): Promise<void> {
  await this.clickOnCreateReportBtn();
  await this.modalWindow.waitFor({ state: 'visible' });
  await this.selectAuditOption();
  await this.clickOnCreateReportModalBtn();
  await this.modalWindow.waitFor({ state: 'hidden' });
}
```

### Pagination Handling

```typescript
// Navigate through paginated results
async navigateToPage(pageNumber: number): Promise<void> {
  const pageButton = this.page.getByRole('link', {
    name: pageNumber.toString(),
    exact: true
  });
  await pageButton.click();
  await this.page.waitForLoadState('networkidle');
}

// Get total page count
async getTotalPageCount(): Promise<number> {
  const paginationList = this.page.locator('div.center ul.pagination');
  const itemCount = await paginationList.locator('li').count();
  return itemCount - 2; // Subtract prev/next buttons
}

// Iterate through all pages
async getAllResultsFromAllPages(): Promise<string[]> {
  const allResults: string[] = [];
  const totalPages = await this.getTotalPageCount();

  for (let page = 1; page <= totalPages; page++) {
    await this.navigateToPage(page);
    const pageResults = await this.getPageResults();
    allResults.push(...pageResults);
  }

  return allResults;
}
```

## Common Patterns

### File Upload Pattern

```typescript
// Basic file upload
async uploadFile(filePath: string): Promise<void> {
  await this.fileInput.setInputFiles(filePath);
  await this.uploadSuccessMessage.waitFor({ state: 'visible' });
}

// File upload with validation
async uploadFileWithValidation(
  filePath: string,
  expectedFileName: string
): Promise<string> {
  await this.uploadFile(filePath);
  const uploadedFileName = await this.uploadedFileName.textContent() || '';

  if (!uploadedFileName.includes(expectedFileName)) {
    throw new Error(`Expected file ${expectedFileName}, but got ${uploadedFileName}`);
  }

  return uploadedFileName;
}

// Multiple file upload
async uploadMultipleFiles(filePaths: string[]): Promise<void> {
  await this.fileInput.setInputFiles(filePaths);

  for (let i = 0; i < filePaths.length; i++) {
    await this.page.locator(`[data-testid="uploaded-file-${i}"]`)
      .waitFor({ state: 'visible' });
  }
}
```

### Download Pattern

```typescript
// File download handling
async downloadReport(): Promise<string> {
  const [download] = await Promise.all([
    this.page.waitForEvent('download'),
    this.downloadButton.click()
  ]);

  const fileName = download.suggestedFilename();
  const downloadPath = `./downloads/${fileName}`;
  await download.saveAs(downloadPath);

  return fileName;
}

// Download with format selection
async downloadReportInFormat(format: 'pdf' | 'csv' | 'xlsx'): Promise<string> {
  await this.selectDownloadFormat(format);
  return await this.downloadReport();
}
```

### Search and Filter Pattern

```typescript
// Complex search with multiple filters
async performAdvancedSearch(criteria: {
  query: string;
  dateRange?: { from: Date; to: Date };
  category?: string;
  sortBy?: 'date' | 'relevance' | 'name';
}): Promise<void> {
  await this.searchField.fill(criteria.query);

  if (criteria.dateRange) {
    await this.setDateFilter(criteria.dateRange.from, criteria.dateRange.to);
  }

  if (criteria.category) {
    await this.selectCategory(criteria.category);
  }

  if (criteria.sortBy) {
    await this.selectSortOption(criteria.sortBy);
  }

  await this.clickSearchButton();
  await this.waitForSearchResults();
}
```

## Error Handling

### Graceful Error Handling

```typescript
// Methods that handle expected errors
async getErrorMessageIfExists(): Promise<string | null> {
  try {
    await this.errorMessage.waitFor({ state: 'visible', timeout: 1000 });
    return await this.errorMessage.textContent();
  } catch {
    return null; // No error message found
  }
}

// Retry patterns for flaky elements
async clickWithRetry(locator: Locator, maxAttempts = 3): Promise<void> {
  for (let attempt = 1; attempt <= maxAttempts; attempt++) {
    try {
      await locator.click({ timeout: 5000 });
      return; // Success, exit early
    } catch (error) {
      if (attempt === maxAttempts) {
        throw error; // Last attempt failed
      }
      await this.page.waitForTimeout(1000); // Wait before retry
    }
  }
}
```

## Best Practices

1. **Single Responsibility**: Each method should do one thing well
2. **Descriptive Names**: Method names should clearly indicate their purpose
3. **Consistent Return Types**: Use consistent types for similar operations
4. **Parameter Validation**: Validate inputs when necessary
5. **Error Handling**: Handle expected failures gracefully
6. **Documentation**: Comment complex business logic
7. **Type Safety**: Use TypeScript types for better maintainability
8. **Async/Await**: Always use async/await for Playwright operations
9. **Wait Strategies**: Include appropriate waits for stability
10. **Reusability**: Design methods to be reusable across tests

### Example Test Usage

```typescript
// Clean, readable test code using well-designed page objects
test("user can create and configure monitor list", async ({ page }) => {
  const addMonitorListPage = new AddMonitorListPage(page);

  await addMonitorListPage.goto();
  await addMonitorListPage.createMonitorList("Test List", {
    negativeNews: true,
    watchlist: true,
    financialNews: false,
  });

  // Verify the list was created
  const confirmationMessage = await addMonitorListPage.getSuccessMessage();
  expect(confirmationMessage).toContain("successfully created");
});
```
