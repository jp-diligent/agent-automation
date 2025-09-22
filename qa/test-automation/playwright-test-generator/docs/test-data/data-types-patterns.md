# 📋 Data Types & Patterns

## 🎯 Overview

Understanding different data types and their usage patterns is crucial for effective test data management. This guide covers the various types of test data and when to use each pattern.

## 📊 Data Type Categories

### 1. Static Data (Immutable)

**Definition**: Fixed values that never change during test execution.

**Examples**:

- User credentials
- Form input values
- Static configuration values
- Test strings and labels

**Implementation**:

```typescript
export class TestData {
  readonly users: {
    admin: string;
    regularUser: string;
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
      regularUser: 'user@company.com',
      viewer: 'viewer@company.com',
    };

    this.forms = {
      validInput: 'Valid Test Input',
      invalidInput: 'Invalid Test Input',
      requiredField: 'Required Field Value',
    };
  }
}
```

**Usage in Tests**:

```typescript
const testData = new TestData();

test('login functionality', async ({ app }) => {
  await app.loginPage.login(testData.users.admin, 'password123');
  await app.formPage.fillInput(testData.forms.validInput);
});
```

### 2. Dynamic Data (Generated)

**Definition**: Values generated at runtime, often unique for each test execution.

**Examples**:

- Timestamps
- Random identifiers
- Unique names
- Sequential numbers

**Implementation**:

```typescript
export class TestData {
  generateRandomNumber(): string {
    return Math.random().toString(36).substring(2);
  }

  generateDate(): string {
    return new Date().toISOString().substring(0, 19).replace('T', ' ');
  }

  generateUniqueEmail(): string {
    return `test.user.${this.generateRandomNumber()}@example.com`;
  }

  generateTestListName(): string {
    return `Test List ${this.generateRandomNumber()}`;
  }

  generateUserName(): string {
    const names = ['John', 'Jane', 'Mike', 'Sarah'];
    const surnames = ['Doe', 'Smith', 'Johnson', 'Brown'];
    const name = names[Math.floor(Math.random() * names.length)];
    const surname = surnames[Math.floor(Math.random() * surnames.length)];
    return `${name} ${surname} ${this.generateRandomNumber()}`;
  }
}
```

**Usage in Tests**:

```typescript
test('create new record', async ({ app }) => {
  const uniqueName = testData.generateUniqueRecordName();
  const timestamp = testData.generateDate();

  await app.recordPage.createRecord(uniqueName, timestamp);
});
```

### 3. Parameterized Data (Test Variants)

**Definition**: Structured data sets for testing multiple scenarios with different inputs.

**Examples**:

- User roles and permissions
- Different input combinations
- Boundary value test cases
- Error condition scenarios

**Implementation**:

```typescript
export class TestData {
  readonly userTypes = [
    {
      role: 'admin',
      email: 'admin@company.com',
      permissions: ['create', 'read', 'update', 'delete'],
      expectedFeatures: ['Dashboard', 'Users', 'Reports', 'Settings'],
    },
    {
      role: 'user',
      email: 'user@company.com',
      permissions: ['read', 'create'],
      expectedFeatures: ['Dashboard', 'Reports'],
    },
    {
      role: 'viewer',
      email: 'viewer@company.com',
      permissions: ['read'],
      expectedFeatures: ['Dashboard'],
    },
  ];

  readonly inputValidationCases = [
    {
      input: '',
      expectedError: 'Field is required',
      shouldFail: true,
    },
    {
      input: 'a',
      expectedError: 'Input must be at least 2 characters',
      shouldFail: true,
    },
    {
      input: 'Valid Input',
      expectedError: null,
      shouldFail: false,
    },
  ];
}
```

**Usage in Tests**:

```typescript
// Test multiple user roles
testData.userTypes.forEach((userType) => {
  test(`features for ${userType.role}`, async ({ app }) => {
    await app.loginPage.loginAs(userType.email);
    await app.navigationPage.verifyFeatures(userType.expectedFeatures);
  });
});

// Test validation scenarios
testData.inputValidationCases.forEach((testCase) => {
  test(`input validation: ${testCase.input || 'empty'}`, async ({ app }) => {
    await app.formPage.enterInput(testCase.input);

    if (testCase.shouldFail) {
      await expect(app.formPage.errorMessage).toContainText(testCase.expectedError);
    } else {
      await expect(app.formPage.successMessage).toBeVisible();
    }
  });
});
```

### 4. File Data (External Resources)

**Definition**: External files used for upload, download, or processing tests.

**Examples**:

- CSV files for bulk operations
- Images for upload testing
- Invalid files for error testing
- Sample documents

**Implementation**:

```typescript
export class TestData {
  readonly files = {
    validCsv: 'upload_sample.csv',
    invalidCsv: 'invalid_file.csv',
    testImage: 'test_image.jpg',
    largeFile: 'large_sample.pdf',
    emptyFile: 'empty_file.txt',
  };

  getFilePath(fileName: string): string {
    return path.join(__dirname, '../test_data', fileName);
  }

  async getFileContent(fileName: string): Promise<string> {
    return fs.readFileSync(this.getFilePath(fileName), 'utf-8');
  }

  getFileSize(fileName: string): number {
    const stats = fs.statSync(this.getFilePath(fileName));
    return stats.size;
  }
}
```

**Usage in Tests**:

```typescript
test('CSV upload functionality', async ({ app }) => {
  const csvFile = testData.getFilePath(testData.files.validCsv);

  await app.uploadPage.uploadFile(csvFile);
  await expect(app.uploadPage.successMessage).toBeVisible();
});

test('invalid file handling', async ({ app }) => {
  const invalidFile = testData.getFilePath(testData.files.invalidCsv);

  await app.uploadPage.uploadFile(invalidFile);
  await expect(app.uploadPage.errorMessage).toContainText('Invalid file format');
});
```

### 5. Configuration Data (Environment-Specific)

**Definition**: Data that varies between different environments or configurations.

**Examples**:

- API endpoints
- Database connection strings
- Feature flags
- Environment-specific settings

**Implementation**:

```typescript
export class TestData {
  readonly environments = {
    dev: {
      baseUrl: 'https://dev.example.com',
      apiUrl: 'https://api-dev.example.com',
      features: {
        newSearch: true,
        advancedFilters: false,
      },
    },
    staging: {
      baseUrl: 'https://staging.example.com',
      apiUrl: 'https://api-staging.example.com',
      features: {
        newSearch: true,
        advancedFilters: true,
      },
    },
  };

  getCurrentEnvironment() {
    const env = process.env.TEST_ENV || 'dev';
    return this.environments[env as keyof typeof this.environments];
  }
}
```

## 🔄 Data Pattern Selection Guide

### When to Use Static Data

- ✅ Fixed test inputs that never change
- ✅ Reference data for comparisons
- ✅ Dropdown option values
- ✅ Error messages for validation

### When to Use Dynamic Data

- ✅ Creating unique test records
- ✅ Avoiding data conflicts between tests
- ✅ Testing with current timestamps
- ✅ Generating realistic test scenarios

### When to Use Parameterized Data

- ✅ Testing multiple user roles
- ✅ Boundary value testing
- ✅ Cross-browser compatibility tests
- ✅ Data-driven test scenarios

### When to Use File Data

- ✅ Upload/download functionality
- ✅ Bulk data processing
- ✅ File format validation
- ✅ Performance testing with large files

### When to Use Configuration Data

- ✅ Multi-environment testing
- ✅ Feature flag testing
- ✅ API endpoint management
- ✅ Environment-specific behaviors

## 🏗️ Advanced Patterns

### Builder Pattern for Complex Data

```typescript
export class UserDataBuilder {
  private user: any = {};

  withName(name: string): UserDataBuilder {
    this.user.name = name;
    return this;
  }

  withEmail(email: string): UserDataBuilder {
    this.user.email = email;
    return this;
  }

  withRole(role: string): UserDataBuilder {
    this.user.role = role;
    return this;
  }

  build(): any {
    return { ...this.user };
  }
}

// Usage
const adminUser = new UserDataBuilder().withName('Admin User').withEmail('admin@example.com').withRole('admin').build();
```

### Factory Pattern for Data Generation

```typescript
export class TestDataFactory {
  static createUser(type: 'admin' | 'user' | 'viewer'): any {
    const baseUser = {
      id: Math.floor(Math.random() * 10000),
      createdAt: new Date().toISOString(),
    };

    switch (type) {
      case 'admin':
        return {
          ...baseUser,
          name: 'Admin User',
          email: 'admin@example.com',
          permissions: ['all'],
        };
      case 'user':
        return {
          ...baseUser,
          name: 'Regular User',
          email: 'user@example.com',
          permissions: ['read', 'write'],
        };
      case 'viewer':
        return {
          ...baseUser,
          name: 'Viewer User',
          email: 'viewer@example.com',
          permissions: ['read'],
        };
    }
  }
}
```

## 🚨 Anti-Patterns to Avoid

### ❌ Hardcoded Data in Tests

```typescript
// ❌ WRONG
test('search test', async ({ app }) => {
  await app.searchPage.performSearch('Barack Obama'); // Hardcoded
});

// ✅ CORRECT
test('search test', async ({ app }) => {
  await app.searchPage.performSearch(testData.terms.barackObama);
});
```

### ❌ Mutable Shared Data

```typescript
// ❌ WRONG
let sharedCounter = 0;
test('test 1', async ({ app }) => {
  sharedCounter++; // Tests affect each other
});

// ✅ CORRECT
test('test 1', async ({ app }) => {
  const uniqueId = testData.generateRandomNumber();
});
```

### ❌ Environment-Specific Hardcoding

```typescript
// ❌ WRONG
const baseUrl = 'https://dev.example.com'; // Hardcoded environment

// ✅ CORRECT
const baseUrl = testData.getCurrentEnvironment().baseUrl;
```

---

**Next**: Learn about [Data Organization](data-organization.md) strategies for structuring your test data effectively.
