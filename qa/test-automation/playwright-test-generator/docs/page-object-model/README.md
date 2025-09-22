# Page Object Model Design Guide

> **Overview**: This is the main entry point for understanding how to implement the Page Object Model (POM) pattern in our Playwright test automation suite. The documentation is organized into focused guides for easy navigation and learning.

## 🎯 What is Page Object Model?

The Page Object Model is a design pattern that creates an abstraction layer between your tests and the web pages. Instead of directly interacting with page elements in tests, you create "page objects" that encapsulate the page structure and provide high-level methods for interacting with the page.

**Key Benefits:**

- **Maintainability**: Centralized element definitions and page interactions
- **Reusability**: Page objects can be used across multiple test files
- **Readability**: Test code focuses on business logic, not implementation details
- **Scalability**: Easy to extend and modify as application grows

## 🚀 Quick Start

### 1. Choose Your Architecture Pattern

We support two main patterns:

**Standard Pattern** (Traditional Playwright approach):

```typescript
test('login flow', async ({ page }) => {
  const loginPage = new LoginPage(page);
  await loginPage.goto();
  await loginPage.login('user@example.com', 'password');
});
```

**App Fixture Pattern** (Recommended for large test suites):

```typescript
test('login flow', async ({ app }) => {
  await app.loginPage.goto();
  await app.loginPage.login('user@example.com', 'password');
});
```

### 2. Follow Our Conventions

- Use semantic locators first: `getByRole()`, `getByLabel()`, `getByText()`
- Group methods by purpose: navigation, actions, utilities
- Return data for tests to assert (no assertions in page objects)
- Use consistent naming: `goto()`, `clickOn*()`, `fill*()`, `get*()`, `is*()`

## 📚 Documentation Structure

### Core Concepts

- **[Architecture Patterns](./architecture-patterns.md)** - Standard vs App Fixture patterns, when to use each
- **[Class Design](./class-design.md)** - Structure, locator organization, TypeScript integration
- **[Locator Strategies](./locator-strategies.md)** - Priority order, semantic locators, advanced patterns

### Implementation Details

- **[Method Design](./method-design.md)** - Naming conventions, categorization, TypeScript integration, and common patterns
- **[Advanced Patterns](./advanced-patterns.md)** - App fixtures, components, error handling, and maintenance strategies

### Best Practices & Guidelines

- **[Best Practices](./best-practices.md)** - Design principles, performance optimization, and team collaboration
- **[Anti-Patterns](./anti-patterns.md)** - Navigation, assertion, locator, and architectural mistakes to avoid
- **[Method Design](./method-design.md)** - Naming conventions, categories, complex business logic

### Advanced Topics

- **[Advanced Patterns](./advanced-patterns.md)** - App fixture implementation, components, error handling
- **[Best Practices](./best-practices.md)** - Organization, waiting strategies, performance optimization

### Guidelines & Reference

- **[Anti-Patterns](./anti-patterns.md)** - Common mistakes and what to avoid
- **[Quick Reference Checklist](../quick-references/pom-checklist.md)** - Validation checklist for page objects
- **[Common Examples](../quick-references/common-examples.md)** - Copy-paste code examples

## 🎯 Architecture Decision Guide

**Use Standard Pattern when:**

- Small to medium test suite (< 50 tests)
- Simple page interactions
- Team prefers explicit instantiation

**Use App Fixture Pattern when:**

- Large test suite (50+ tests)
- Many page objects (10+ pages)
- Complex cross-page workflows
- Want centralized page object management

## 🔧 Getting Started Template

```typescript
// Basic page object structure
import { type Locator, type Page } from '@playwright/test';

export class YourPage {
  readonly page: Page;

  // Group locators by purpose
  readonly submitButton: Locator;
  readonly errorMessage: Locator;

  constructor(page: Page) {
    this.page = page;
    // Prefer semantic locators
    this.submitButton = page.getByRole('button', { name: 'Submit' });
    this.errorMessage = page.locator('#error-message');
  }

  // Navigation methods
  async goto(): Promise<void> {
    await this.page.goto('/your-page');
    await this.page.waitForLoadState('domcontentloaded');
  }

  // Action methods
  async clickSubmit(): Promise<void> {
    await this.submitButton.click();
  }

  // Utility methods - return data for tests to assert
  async getErrorMessage(): Promise<string | null> {
    return await this.errorMessage.textContent();
  }
}
```

## 🚨 Critical Rules

1. **Never use hardcoded URLs in tests** - always use page object navigation methods
2. **No assertions in page objects** - return data for tests to assert
3. **Use semantic locators first** - `getByRole()` over CSS selectors
4. **Every action wrapped in `test.step()`** in test files
5. **Include proper wait strategies** in navigation methods

## 📖 Next Steps

1. Start with **[Architecture Patterns](./architecture-patterns.md)** to choose your approach
2. Read **[Class Design](./class-design.md)** for structure guidelines
3. Review **[Locator Strategies](./locator-strategies.md)** for element selection
4. Check **[Anti-Patterns](./anti-patterns.md)** to avoid common mistakes
5. Use **[Quick Reference Checklist](../quick-references/pom-checklist.md)** for validation

---

_This modular documentation structure allows you to focus on specific aspects of Page Object Model implementation while maintaining comprehensive coverage of the pattern._
