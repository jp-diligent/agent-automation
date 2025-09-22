# 📊 Test Data Management Guide

> **Overview**: This guide covers test data management strategies for Playwright test automation, focusing on centralized data classes, data isolation patterns, and maintainable test data practices.

## 🎯 What is Test Data Management?

Test data management is the practice of organizing, generating, and maintaining all data used in test automation. Proper test data management ensures tests are reliable, maintainable, and provide consistent results across different environments.

**Key Benefits:**

- **Consistency**: Standardized data across all tests
- **Maintainability**: Centralized data definitions reduce duplication
- **Reusability**: Data can be shared across multiple test files
- **Isolation**: Tests don't interfere with each other's data
- **Flexibility**: Easy to modify data for different test scenarios

## 🗂️ Test Data Architecture

### Directory Structure

```
test_data/
├── test.data.ts              # Central test data class
├── documentation/            # Test data documentation
├── parsed_cases/            # Parsed test cases from XML
├── files/                   # Test files for upload/download
│   ├── valid_sample.csv
│   ├── test_image.jpg
│   └── invalid_file.csv
└── environments/            # Environment-specific data
```

## 🏗️ Core Patterns

### 1. Central Test Data Class

**Primary Pattern**: Use `test.data.ts` for all shared test data:

```typescript
export class TestData {
  readonly users: {
    admin: string;
    user: string;
    viewer: string;
  };

  readonly forms: {
    validInput: string;
    invalidInput: string;
    requiredField: string;
  };

  constructor() {
    this.users = {
      admin: 'admin@company.com',
      user: 'user@company.com',
      viewer: 'viewer@company.com',
    };

    this.forms = {
      validInput: 'Valid Test Input',
      invalidInput: 'Invalid Input',
      requiredField: 'Required Field Value',
    };
  }

  generateRandomNumber() {
    return Math.random().toString(36).substring(2);
  }

  generateDate() {
    return new Date().toISOString().substring(0, 19).replace('T', ' ');
  }

  generateUniqueId(prefix: string = 'test') {
    return `${prefix}_${this.generateRandomNumber()}_${Date.now()}`;
  }
}
```

### 2. Data Categories

Organize data into logical categories:

- **Static Data**: Fixed values that don't change (user names, test strings)
- **Dynamic Data**: Generated values (timestamps, random IDs)
- **Environment Data**: Environment-specific values (URLs, configurations)
- **Test Files**: Files for upload/download testing

## 📖 Detailed Guides

### Core Concepts

- [Data Types & Patterns](data-types-patterns.md) 📋
- [Data Organization](data-organization.md) 🗂️
- [Best Practices](best-practices.md) ⭐

### Quick References

- [Data Checklist](../quick-references/test-data-checklist.md) ✅
- [Common Examples](../quick-references/test-data-examples.md) 📝

## 🚀 Quick Start Examples

### Basic Usage in Tests

```typescript
import { expect, test } from 'fixtures/fixtures';
import { TestData } from '../test_data/test.data';

const testData = new TestData();

test.describe('Application Tests', () => {
  test('should perform basic functionality', async ({ app }) => {
    await test.step('Navigate to page', async () => {
      await app.homePage.goto();
    });

    await test.step('Fill form with test data', async () => {
      await app.formPage.fillField(testData.forms.validInput);
    });

    await test.step('Verify results', async () => {
      await expect(app.resultsPage.container).toBeVisible();
    });
  });
});
```

### Dynamic Data Generation

```typescript
test('should create unique record', async ({ app }) => {
  const uniqueName = `Test Record ${testData.generateRandomNumber()}`;
  const timestamp = testData.generateDate();

  await test.step('Create new record with unique data', async () => {
    await app.recordPage.createRecord(uniqueName, timestamp);
  });
});
```

### File Testing

```typescript
test('should upload file', async ({ app }) => {
  await test.step('Upload sample file', async () => {
    const filePath = path.join(__dirname, '../test_data/valid_sample.csv');
    await app.uploadPage.uploadFile(filePath);
  });
});
```

## 🔑 Key Principles

### 1. Data Isolation

- Each test should use independent data
- Avoid shared mutable state between tests
- Use unique identifiers for test data

### 2. Data Lifecycle

- **Setup**: Generate/prepare data before test
- **Execution**: Use data during test
- **Cleanup**: Remove/reset data after test

### 3. Environment Awareness

- Support multiple environments (dev, staging, prod)
- Use environment-specific configurations
- Avoid hardcoded environment values

### 4. Security Considerations

- Never commit sensitive data to repository
- Use environment variables for secrets
- Implement data masking for logs

## 🚨 Critical Requirements

### ✅ **MUST DO**

- **Use TestData class** for all shared test data
- **Implement data isolation** between tests
- **Use descriptive names** for data properties
- **Generate unique identifiers** for test records
- **Document data purposes** with comments
- **Validate data formats** before use

### 🚫 **NEVER DO**

- **Hardcode data** directly in test files
- **Share mutable data** between tests
- **Commit sensitive data** to repository
- **Use production data** in tests
- **Ignore data cleanup** after tests
- **Mix test data** with application code

## 📊 Data Quality Standards

### Naming Conventions

- Use descriptive, meaningful names
- Follow camelCase for properties
- Use consistent naming patterns
- Include data type hints in names

### Documentation Requirements

- Document data purpose and usage
- Include examples in comments
- Specify data dependencies
- Note any constraints or validations

### Validation Standards

- Implement data format validation
- Check data completeness
- Verify data consistency
- Test edge cases and boundaries

## 🔗 Integration Points

### With Page Objects

```typescript
// ✅ CORRECT: Pass data to page object methods
await app.formPage.fillUserDetails(testData.users.defaultUser);

// ❌ WRONG: Access data directly in page objects
// Page objects should not import TestData directly
```

### With API Tests

```typescript
// ✅ CORRECT: Use consistent data across UI and API tests
const apiResponse = await api.createUser(testData.users.defaultUser);
await app.userPage.verifyUserCreated(testData.users.defaultUser);
```

### With Test Fixtures

```typescript
// ✅ CORRECT: Prepare test data in fixtures
test.beforeEach(async ({ page }) => {
  const testUser = testData.users.generateTestUser();
  await api.setupTestUser(testUser);
});
```

---

**Next Steps**: Start with [Data Types & Patterns](data-types-patterns.md) to understand the fundamental concepts, then explore specific guides based on your needs.
