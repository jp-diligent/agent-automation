# Common Examples

**Description**: Ready-to-use code examples for common Page Object Model patterns and scenarios. Copy-paste templates for typical page objects, components, and testing patterns.

## Table of Contents

- [Basic Page Object Template](#basic-page-object-template)
- [Login Page Examples](#login-page-examples)
- [Form Handling Patterns](#form-handling-patterns)
- [Search and Filter Patterns](#search-and-filter-patterns)
- [Data Table Interactions](#data-table-interactions)
- [Modal and Dialog Handling](#modal-and-dialog-handling)
- [File Upload/Download](#file-uploaddownload)
- [Navigation Patterns](#navigation-patterns)
- [Error Handling Examples](#error-handling-examples)
- [Component Patterns](#component-patterns)

## Basic Page Object Template

```typescript
// pages/example.page.ts
import { type Page, type Locator } from "@playwright/test";

export class ExamplePage {
  readonly page: Page;

  // Group locators by functionality
  // Navigation elements
  readonly homeLink: Locator;
  readonly menuButton: Locator;

  // Form elements
  readonly inputField: Locator;
  readonly submitButton: Locator;
  readonly cancelButton: Locator;

  // Content elements
  readonly mainContent: Locator;
  readonly errorMessage: Locator;
  readonly successMessage: Locator;

  constructor(page: Page) {
    this.page = page;

    // Use semantic locators first
    this.homeLink = page.getByRole("link", { name: "Home" });
    this.submitButton = page.getByRole("button", { name: "Submit" });
    this.cancelButton = page.getByRole("button", { name: "Cancel" });

    // Use test IDs when semantic locators aren't available
    this.menuButton = page.getByTestId("menu-toggle");
    this.mainContent = page.getByTestId("main-content");

    // Use labels for form fields
    this.inputField = page.getByLabel("Input Field");

    // Use specific selectors for unique elements
    this.errorMessage = page.locator(".error-message");
    this.successMessage = page.locator(".success-message");
  }

  // Navigation methods
  async goto(): Promise<void> {
    await this.page.goto("/example");
    await this.page.waitForLoadState("domcontentloaded");
  }

  // Action methods
  async fillInput(value: string): Promise<void> {
    await this.inputField.fill(value);
  }

  async submitForm(): Promise<void> {
    await this.submitButton.click();
  }

  async cancelForm(): Promise<void> {
    await this.cancelButton.click();
  }

  // Utility methods
  async getErrorMessage(): Promise<string | null> {
    try {
      await this.errorMessage.waitFor({ state: "visible", timeout: 2000 });
      return await this.errorMessage.textContent();
    } catch {
      return null;
    }
  }

  async isFormValid(): Promise<boolean> {
    return await this.submitButton.isEnabled();
  }

  // Wait methods
  async waitForPageLoad(): Promise<void> {
    await this.mainContent.waitFor({ state: "visible" });
  }
}
```

## Login Page Examples

### Simple Login Page

```typescript
// pages/auth/login.page.ts
import { type Page, type Locator } from "@playwright/test";

export interface LoginCredentials {
  username: string;
  password: string;
}

export class LoginPage {
  readonly page: Page;
  readonly usernameField: Locator;
  readonly passwordField: Locator;
  readonly loginButton: Locator;
  readonly errorMessage: Locator;
  readonly forgotPasswordLink: Locator;
  readonly rememberMeCheckbox: Locator;

  constructor(page: Page) {
    this.page = page;
    this.usernameField = page.getByLabel("Username");
    this.passwordField = page.getByLabel("Password");
    this.loginButton = page.getByRole("button", { name: "Sign In" });
    this.errorMessage = page.locator(".error-message");
    this.forgotPasswordLink = page.getByRole("link", {
      name: "Forgot Password?",
    });
    this.rememberMeCheckbox = page.getByLabel("Remember me");
  }

  async goto(): Promise<void> {
    await this.page.goto("/login");
    await this.page.waitForLoadState("domcontentloaded");
  }

  async login(credentials: LoginCredentials): Promise<string> {
    await this.usernameField.fill(credentials.username);
    await this.passwordField.fill(credentials.password);
    await this.loginButton.click();
    await this.page.waitForLoadState("networkidle");
    return this.page.url();
  }

  async loginWithRememberMe(credentials: LoginCredentials): Promise<string> {
    await this.rememberMeCheckbox.check();
    return await this.login(credentials);
  }

  async getErrorMessage(): Promise<string | null> {
    try {
      await this.errorMessage.waitFor({ state: "visible", timeout: 3000 });
      return await this.errorMessage.textContent();
    } catch {
      return null;
    }
  }

  async clickForgotPassword(): Promise<void> {
    await this.forgotPasswordLink.click();
  }

  async isLoginFormVisible(): Promise<boolean> {
    return (
      (await this.usernameField.isVisible()) &&
      (await this.passwordField.isVisible())
    );
  }
}
```

### Advanced Login with Multi-Factor Authentication

```typescript
// pages/auth/mfa-login.page.ts
import { LoginPage, type LoginCredentials } from "./login.page";

export class MFALoginPage extends LoginPage {
  readonly mfaCodeField: Locator;
  readonly verifyMFAButton: Locator;
  readonly resendCodeButton: Locator;
  readonly mfaErrorMessage: Locator;

  constructor(page: Page) {
    super(page);
    this.mfaCodeField = page.getByLabel("Verification Code");
    this.verifyMFAButton = page.getByRole("button", { name: "Verify" });
    this.resendCodeButton = page.getByRole("button", { name: "Resend Code" });
    this.mfaErrorMessage = page.locator(".mfa-error");
  }

  async loginWithMFA(
    credentials: LoginCredentials,
    mfaCode: string
  ): Promise<string> {
    // First step: regular login
    await this.login(credentials);

    // Second step: MFA verification
    await this.mfaCodeField.waitFor({ state: "visible" });
    await this.mfaCodeField.fill(mfaCode);
    await this.verifyMFAButton.click();
    await this.page.waitForLoadState("networkidle");

    return this.page.url();
  }

  async resendMFACode(): Promise<void> {
    await this.resendCodeButton.click();
  }

  async getMFAErrorMessage(): Promise<string | null> {
    try {
      await this.mfaErrorMessage.waitFor({ state: "visible", timeout: 2000 });
      return await this.mfaErrorMessage.textContent();
    } catch {
      return null;
    }
  }
}
```

## Form Handling Patterns

### Generic Form Handler

```typescript
// pages/forms/base-form.page.ts
export interface FormField {
  label: string;
  value: string;
  type?: "text" | "email" | "password" | "select" | "checkbox";
}

export class BaseFormPage {
  readonly page: Page;
  readonly form: Locator;
  readonly submitButton: Locator;
  readonly cancelButton: Locator;
  readonly validationErrors: Locator;

  constructor(page: Page, formSelector: string = "form") {
    this.page = page;
    this.form = page.locator(formSelector);
    this.submitButton = this.form.getByRole("button", {
      name: /submit|save|create/i,
    });
    this.cancelButton = this.form.getByRole("button", {
      name: /cancel|close/i,
    });
    this.validationErrors = this.form.locator(".error, .invalid-feedback");
  }

  async fillForm(fields: FormField[]): Promise<void> {
    for (const field of fields) {
      await this.fillField(field);
    }
  }

  async fillField(field: FormField): Promise<void> {
    const locator = this.form.getByLabel(field.label);

    switch (field.type) {
      case "checkbox":
        if (field.value === "true") {
          await locator.check();
        } else {
          await locator.uncheck();
        }
        break;
      case "select":
        await locator.selectOption(field.value);
        break;
      default:
        await locator.fill(field.value);
    }
  }

  async submitForm(): Promise<void> {
    await this.submitButton.click();
  }

  async cancelForm(): Promise<void> {
    await this.cancelButton.click();
  }

  async getValidationErrors(): Promise<string[]> {
    const errors = await this.validationErrors.allTextContents();
    return errors.filter((error) => error.trim().length > 0);
  }

  async isFormValid(): Promise<boolean> {
    return await this.submitButton.isEnabled();
  }
}
```

### User Registration Form

```typescript
// pages/forms/user-registration.page.ts
export interface UserRegistrationData {
  firstName: string;
  lastName: string;
  email: string;
  password: string;
  confirmPassword: string;
  agreeToTerms: boolean;
  newsletter?: boolean;
}

export class UserRegistrationPage {
  readonly page: Page;
  readonly firstNameField: Locator;
  readonly lastNameField: Locator;
  readonly emailField: Locator;
  readonly passwordField: Locator;
  readonly confirmPasswordField: Locator;
  readonly agreeToTermsCheckbox: Locator;
  readonly newsletterCheckbox: Locator;
  readonly submitButton: Locator;
  readonly successMessage: Locator;

  constructor(page: Page) {
    this.page = page;
    this.firstNameField = page.getByLabel("First Name");
    this.lastNameField = page.getByLabel("Last Name");
    this.emailField = page.getByLabel("Email Address");
    this.passwordField = page.getByLabel("Password");
    this.confirmPasswordField = page.getByLabel("Confirm Password");
    this.agreeToTermsCheckbox = page.getByLabel(
      "I agree to the Terms of Service"
    );
    this.newsletterCheckbox = page.getByLabel("Subscribe to newsletter");
    this.submitButton = page.getByRole("button", { name: "Create Account" });
    this.successMessage = page.locator(".success-message");
  }

  async goto(): Promise<void> {
    await this.page.goto("/register");
    await this.page.waitForLoadState("domcontentloaded");
  }

  async registerUser(userData: UserRegistrationData): Promise<void> {
    await this.firstNameField.fill(userData.firstName);
    await this.lastNameField.fill(userData.lastName);
    await this.emailField.fill(userData.email);
    await this.passwordField.fill(userData.password);
    await this.confirmPasswordField.fill(userData.confirmPassword);

    if (userData.agreeToTerms) {
      await this.agreeToTermsCheckbox.check();
    }

    if (userData.newsletter) {
      await this.newsletterCheckbox.check();
    }

    await this.submitButton.click();
  }

  async getSuccessMessage(): Promise<string | null> {
    try {
      await this.successMessage.waitFor({ state: "visible", timeout: 5000 });
      return await this.successMessage.textContent();
    } catch {
      return null;
    }
  }
}
```

## Search and Filter Patterns

```typescript
// pages/search/search.page.ts
export interface SearchFilters {
  category?: string;
  dateRange?: {
    from: Date;
    to: Date;
  };
  priceRange?: {
    min: number;
    max: number;
  };
  sortBy?: "relevance" | "date" | "price" | "name";
}

export class SearchPage {
  readonly page: Page;
  readonly searchField: Locator;
  readonly searchButton: Locator;
  readonly categoryFilter: Locator;
  readonly dateFromField: Locator;
  readonly dateToField: Locator;
  readonly priceMinField: Locator;
  readonly priceMaxField: Locator;
  readonly sortDropdown: Locator;
  readonly results: Locator;
  readonly resultCount: Locator;
  readonly clearFiltersButton: Locator;

  constructor(page: Page) {
    this.page = page;
    this.searchField = page.getByPlaceholder("Enter search term");
    this.searchButton = page.getByRole("button", { name: "Search" });
    this.categoryFilter = page.getByLabel("Category");
    this.dateFromField = page.getByLabel("From Date");
    this.dateToField = page.getByLabel("To Date");
    this.priceMinField = page.getByLabel("Min Price");
    this.priceMaxField = page.getByLabel("Max Price");
    this.sortDropdown = page.getByLabel("Sort by");
    this.results = page.getByTestId("search-results");
    this.resultCount = page.getByTestId("result-count");
    this.clearFiltersButton = page.getByRole("button", {
      name: "Clear Filters",
    });
  }

  async goto(): Promise<void> {
    await this.page.goto("/search");
    await this.page.waitForLoadState("domcontentloaded");
  }

  async performSearch(query: string): Promise<void> {
    await this.searchField.fill(query);
    await this.searchButton.click();
    await this.waitForSearchResults();
  }

  async applyFilters(filters: SearchFilters): Promise<void> {
    if (filters.category) {
      await this.categoryFilter.selectOption(filters.category);
    }

    if (filters.dateRange) {
      await this.dateFromField.fill(
        filters.dateRange.from.toISOString().split("T")[0]
      );
      await this.dateToField.fill(
        filters.dateRange.to.toISOString().split("T")[0]
      );
    }

    if (filters.priceRange) {
      await this.priceMinField.fill(filters.priceRange.min.toString());
      await this.priceMaxField.fill(filters.priceRange.max.toString());
    }

    if (filters.sortBy) {
      await this.sortDropdown.selectOption(filters.sortBy);
    }

    await this.waitForSearchResults();
  }

  async performAdvancedSearch(
    query: string,
    filters: SearchFilters
  ): Promise<void> {
    await this.performSearch(query);
    await this.applyFilters(filters);
  }

  async clearAllFilters(): Promise<void> {
    await this.clearFiltersButton.click();
    await this.waitForSearchResults();
  }

  async getResultCount(): Promise<number> {
    const countText = await this.resultCount.textContent();
    const match = countText?.match(/(\d+) results?/);
    return match ? parseInt(match[1], 10) : 0;
  }

  async getSearchResults(): Promise<string[]> {
    return await this.results.locator(".result-item .title").allTextContents();
  }

  private async waitForSearchResults(): Promise<void> {
    await this.page.waitForLoadState("networkidle");
    await this.results
      .first()
      .waitFor({ state: "visible" })
      .catch(() => {
        // No results might be valid
      });
  }
}
```

## Data Table Interactions

```typescript
// components/data-table.component.ts
export interface TableRow {
  [key: string]: string;
}

export class DataTableComponent {
  readonly container: Locator;
  readonly headers: Locator;
  readonly rows: Locator;
  readonly pagination: Locator;
  readonly sortButtons: Locator;

  constructor(
    page: Page,
    tableSelector: string = '[data-testid="data-table"]'
  ) {
    this.container = page.locator(tableSelector);
    this.headers = this.container.locator("thead th");
    this.rows = this.container.locator("tbody tr");
    this.pagination = page.locator(".pagination");
    this.sortButtons = this.headers.locator(".sort-button");
  }

  async getRowCount(): Promise<number> {
    return await this.rows.count();
  }

  async getColumnHeaders(): Promise<string[]> {
    return await this.headers.allTextContents();
  }

  async getRowByIndex(index: number): Promise<Locator> {
    return this.rows.nth(index);
  }

  async getRowByText(text: string): Promise<Locator> {
    return this.rows.filter({ hasText: text });
  }

  async getCellValue(rowIndex: number, columnIndex: number): Promise<string> {
    const row = this.rows.nth(rowIndex);
    const cell = row.locator("td").nth(columnIndex);
    return (await cell.textContent()) || "";
  }

  async getRowData(rowIndex: number): Promise<TableRow> {
    const headers = await this.getColumnHeaders();
    const row = this.rows.nth(rowIndex);
    const cells = await row.locator("td").allTextContents();

    const rowData: TableRow = {};
    headers.forEach((header, index) => {
      rowData[header] = cells[index] || "";
    });

    return rowData;
  }

  async sortByColumn(columnName: string): Promise<void> {
    const headerIndex = (await this.getColumnHeaders()).indexOf(columnName);
    if (headerIndex >= 0) {
      await this.headers.nth(headerIndex).click();
    }
  }

  async searchInTable(searchTerm: string): Promise<Locator[]> {
    const matchingRows: Locator[] = [];
    const rowCount = await this.getRowCount();

    for (let i = 0; i < rowCount; i++) {
      const row = this.rows.nth(i);
      const rowText = await row.textContent();
      if (rowText?.includes(searchTerm)) {
        matchingRows.push(row);
      }
    }

    return matchingRows;
  }

  async clickRowAction(rowIndex: number, actionName: string): Promise<void> {
    const row = this.rows.nth(rowIndex);
    const actionButton = row.getByRole("button", { name: actionName });
    await actionButton.click();
  }

  // Pagination methods
  async navigateToPage(pageNumber: number): Promise<void> {
    const pageButton = this.pagination.getByRole("button", {
      name: pageNumber.toString(),
    });
    await pageButton.click();
    await this.container.waitFor({ state: "stable" });
  }

  async getTotalPages(): Promise<number> {
    const pageButtons = this.pagination.locator("button[data-page]");
    const pageNumbers = await pageButtons.allTextContents();
    return Math.max(
      ...pageNumbers.map((p) => parseInt(p, 10)).filter((n) => !isNaN(n))
    );
  }
}
```

## Modal and Dialog Handling

```typescript
// components/modal.component.ts
export class ModalComponent {
  readonly container: Locator;
  readonly title: Locator;
  readonly content: Locator;
  readonly closeButton: Locator;
  readonly confirmButton: Locator;
  readonly cancelButton: Locator;
  readonly backdrop: Locator;

  constructor(page: Page, modalSelector: string = ".modal") {
    this.container = page.locator(modalSelector);
    this.title = this.container.locator(".modal-title");
    this.content = this.container.locator(".modal-body");
    this.closeButton = this.container.getByRole("button", { name: "Close" });
    this.confirmButton = this.container.getByRole("button", {
      name: /confirm|ok|yes|delete|save/i,
    });
    this.cancelButton = this.container.getByRole("button", {
      name: /cancel|no|dismiss/i,
    });
    this.backdrop = page.locator(".modal-backdrop");
  }

  async waitForModal(): Promise<void> {
    await this.container.waitFor({ state: "visible" });
  }

  async getTitle(): Promise<string> {
    return (await this.title.textContent()) || "";
  }

  async getContent(): Promise<string> {
    return (await this.content.textContent()) || "";
  }

  async confirm(): Promise<void> {
    await this.confirmButton.click();
    await this.container.waitFor({ state: "hidden" });
  }

  async cancel(): Promise<void> {
    await this.cancelButton.click();
    await this.container.waitFor({ state: "hidden" });
  }

  async close(): Promise<void> {
    await this.closeButton.click();
    await this.container.waitFor({ state: "hidden" });
  }

  async dismissByBackdrop(): Promise<void> {
    await this.backdrop.click();
    await this.container.waitFor({ state: "hidden" });
  }

  async isVisible(): Promise<boolean> {
    return await this.container.isVisible();
  }
}

// Usage in page objects
export class UserManagementPage {
  readonly deleteModal: ModalComponent;
  readonly editModal: ModalComponent;

  constructor(page: Page) {
    this.page = page;
    this.deleteModal = new ModalComponent(page, "#delete-confirmation-modal");
    this.editModal = new ModalComponent(page, "#edit-user-modal");
  }

  async deleteUser(userName: string): Promise<boolean> {
    const userRow = await this.getUserRowByName(userName);
    const deleteButton = userRow.getByRole("button", { name: "Delete" });

    await deleteButton.click();
    await this.deleteModal.waitForModal();

    const modalTitle = await this.deleteModal.getTitle();
    if (modalTitle.includes("Delete User")) {
      await this.deleteModal.confirm();
      return true;
    }

    return false;
  }

  async editUser(userName: string, newData: UserData): Promise<void> {
    const userRow = await this.getUserRowByName(userName);
    const editButton = userRow.getByRole("button", { name: "Edit" });

    await editButton.click();
    await this.editModal.waitForModal();

    // Fill form in modal
    await this.fillUserEditForm(newData);
    await this.editModal.confirm();
  }
}
```

## File Upload/Download

```typescript
// pages/upload.page.ts
export class UploadPage {
  readonly page: Page;
  readonly fileInput: Locator;
  readonly uploadButton: Locator;
  readonly downloadButton: Locator;
  readonly uploadProgress: Locator;
  readonly uploadSuccessMessage: Locator;
  readonly fileList: Locator;

  constructor(page: Page) {
    this.page = page;
    this.fileInput = page.locator('input[type="file"]');
    this.uploadButton = page.getByRole("button", { name: "Upload" });
    this.downloadButton = page.getByRole("button", { name: "Download" });
    this.uploadProgress = page.locator(".upload-progress");
    this.uploadSuccessMessage = page.locator(".upload-success");
    this.fileList = page.getByTestId("file-list");
  }

  async uploadFile(filePath: string): Promise<string> {
    await this.fileInput.setInputFiles(filePath);
    await this.uploadButton.click();

    // Wait for upload to complete
    await this.uploadSuccessMessage.waitFor({
      state: "visible",
      timeout: 30000,
    });

    return (await this.uploadSuccessMessage.textContent()) || "";
  }

  async uploadMultipleFiles(filePaths: string[]): Promise<string[]> {
    await this.fileInput.setInputFiles(filePaths);
    await this.uploadButton.click();

    const results: string[] = [];
    for (let i = 0; i < filePaths.length; i++) {
      const successMessage = this.page.locator(`.upload-success-${i}`);
      await successMessage.waitFor({ state: "visible", timeout: 30000 });
      results.push((await successMessage.textContent()) || "");
    }

    return results;
  }

  async downloadFile(fileName: string): Promise<string> {
    const [download] = await Promise.all([
      this.page.waitForEvent("download"),
      this.fileList.getByText(fileName).click(),
    ]);

    const downloadedFileName = download.suggestedFilename();
    const downloadPath = `./downloads/${downloadedFileName}`;
    await download.saveAs(downloadPath);

    return downloadedFileName;
  }

  async downloadReport(
    format: "pdf" | "csv" | "xlsx" = "pdf"
  ): Promise<string> {
    // Select format first
    const formatDropdown = this.page.getByLabel("Export Format");
    await formatDropdown.selectOption(format);

    const [download] = await Promise.all([
      this.page.waitForEvent("download"),
      this.downloadButton.click(),
    ]);

    const fileName = download.suggestedFilename();
    await download.saveAs(`./downloads/${fileName}`);

    return fileName;
  }

  async getUploadProgress(): Promise<number> {
    const progressText = await this.uploadProgress.textContent();
    const match = progressText?.match(/(\d+)%/);
    return match ? parseInt(match[1], 10) : 0;
  }

  async getUploadedFileNames(): Promise<string[]> {
    return await this.fileList.locator(".file-name").allTextContents();
  }
}
```

This collection provides ready-to-use templates and patterns for the most common Page Object Model scenarios. Copy and adapt these examples to fit your specific application needs.
