# 📝 Test Data Examples & Common Patterns

## 🎯 Quick Reference Examples

### Basic TestData Class Structure

```typescript
import * as fs from 'fs';
import * as path from 'path';

export class TestData {
  // Static data - Fixed values for consistent testing
  readonly staticData = {
    // User roles and permissions
    users: {
      admin: {
        email: 'admin@company.com',
        role: 'admin',
        permissions: ['create', 'read', 'update', 'delete', 'admin'],
      },
      user: {
        email: 'user@company.com',
        role: 'user',
        permissions: ['create', 'read', 'update'],
      },
      viewer: {
        email: 'viewer@company.com',
        role: 'viewer',
        permissions: ['read'],
      },
    },

    // Form inputs categorized by type
    forms: {
      inputs: {
        validText: 'Valid Text Input',
        invalidText: 'Invalid Input',
        longText: 'Very long text that exceeds normal character limits for testing',
      },
      dropdowns: {
        firstOption: 'First Option',
        secondOption: 'Second Option',
        thirdOption: 'Third Option',
      },
    },

    // UI elements and messages
    ui: {
      buttons: {
        submit: 'Submit',
        cancel: 'Cancel',
        delete: 'Delete',
        edit: 'Edit',
      },
      messages: {
        success: 'Operation completed successfully',
        error: 'An error occurred',
        validation: 'Please check your input',
      },
    },
  };

  // Dynamic data generation methods
  generateRandomNumber(): string {
    return Math.random().toString(36).substring(2);
  }

  generateTimestamp(): string {
    return new Date().toISOString();
  }

  generateDate(): string {
    return new Date().toISOString().substring(0, 19).replace('T', ' ');
  }

  generateUniqueEmail(): string {
    return `test.user.${this.generateRandomNumber()}@example.com`;
  }

  generateUniqueRecordName(): string {
    return `Test Record ${this.generateRandomNumber()}`;
  }
}
```

### Usage in Test Files

```typescript
import { expect, test } from 'fixtures/fixtures';
import { TestData } from '../test_data/test.data';

const testData = new TestData();

test.describe('Application Functionality', () => {
  test('should perform basic form operations', async ({ app }) => {
    await test.step('Navigate to form page', async () => {
      await app.formPage.goto();
    });

    await test.step('Fill form with valid data', async () => {
      await app.formPage.fillInput(testData.staticData.forms.inputs.validText);
      await app.formPage.selectOption(testData.staticData.forms.dropdowns.firstOption);
    });

    await test.step('Verify form submission', async () => {
      await expect(app.formPage.successMessage).toContainText(testData.staticData.ui.messages.success);
    });
  });

  test('should create unique record', async ({ app }) => {
    const uniqueName = testData.generateUniqueRecordName();
    const timestamp = testData.generateTimestamp();

    await test.step('Create new record with unique data', async () => {
      await app.recordPage.goto();
      await app.recordPage.createRecord(uniqueName, timestamp);
    });

    await test.step('Verify record creation', async () => {
      await expect(app.recordPage.successMessage).toContainText(testData.staticData.ui.messages.success);
      await expect(app.recordPage.recordContainer).toContainText(uniqueName);
    });
  });
});
```

## 🔄 Common Patterns

### 1. User Data Generation Pattern

```typescript
export class TestData {
  // User generation with role-based data
  generateUserWithRole(role: 'admin' | 'user' | 'viewer'): UserData {
    const person = this.generatePersonName();
    const baseEmail = `${person.first.toLowerCase()}.${person.last.toLowerCase()}`;

    const roleConfigs = {
      admin: {
        email: `admin.${baseEmail}@company.com`,
        permissions: ['create', 'read', 'update', 'delete', 'admin'],
        department: 'IT',
      },
      user: {
        email: `${baseEmail}@company.com`,
        permissions: ['create', 'read', 'update'],
        department: 'Compliance',
      },
      viewer: {
        email: `view.${baseEmail}@company.com`,
        permissions: ['read'],
        department: 'Audit',
      },
    };

    const config = roleConfigs[role];

    return {
      id: this.generateUUID(),
      name: person.full,
      email: config.email,
      role: role,
      permissions: config.permissions,
      department: config.department,
      createdAt: this.generateTimestamp(),
    };
  }

  // Helper for generating realistic names
  generatePersonName(): { first: string; last: string; full: string } {
    const firstNames = ['John', 'Jane', 'Michael', 'Sarah', 'David', 'Emma'];
    const lastNames = ['Smith', 'Johnson', 'Williams', 'Brown', 'Jones', 'Garcia'];

    const first = firstNames[Math.floor(Math.random() * firstNames.length)];
    const last = lastNames[Math.floor(Math.random() * lastNames.length)];

    return { first, last, full: `${first} ${last}` };
  }
}

// Usage
test('user creation with different roles', async ({ app }) => {
  const adminUser = testData.generateUserWithRole('admin');
  const regularUser = testData.generateUserWithRole('user');

  await app.userPage.createUser(adminUser);
  await app.userPage.createUser(regularUser);
});
```

### 2. Parameterized Test Data Pattern

```typescript
export class TestData {
  // Test scenarios for data-driven testing
  readonly loginScenarios = [
    {
      name: 'valid admin login',
      email: 'admin@company.com',
      password: 'AdminPass123!',
      expectedResult: 'success',
      expectedRedirect: '/dashboard',
    },
    {
      name: 'invalid password',
      email: 'admin@company.com',
      password: 'wrongpassword',
      expectedResult: 'error',
      expectedMessage: 'Invalid credentials',
    },
    {
      name: 'invalid email format',
      email: 'invalid-email',
      password: 'AdminPass123!',
      expectedResult: 'error',
      expectedMessage: 'Please enter a valid email address',
    },
  ];

  readonly searchValidationCases = [
    {
      input: '',
      expectedError: 'Search term is required',
      shouldFail: true,
    },
    {
      input: 'a',
      expectedError: 'Search term must be at least 2 characters',
      shouldFail: true,
    },
    {
      input: 'Valid Search Term',
      expectedError: null,
      shouldFail: false,
    },
  ];
}

// Usage in parameterized tests
testData.loginScenarios.forEach((scenario) => {
  test(`login: ${scenario.name}`, async ({ app }) => {
    await test.step('Attempt login', async () => {
      await app.loginPage.login(scenario.email, scenario.password);
    });

    if (scenario.expectedResult === 'success') {
      await test.step('Verify successful login', async () => {
        await expect(app.page).toHaveURL(scenario.expectedRedirect);
      });
    } else {
      await test.step('Verify login error', async () => {
        await expect(app.loginPage.errorMessage).toContainText(scenario.expectedMessage);
      });
    }
  });
});
```

### 3. File Upload/Download Pattern

```typescript
export class TestData {
  // File management for upload tests
  readonly testFiles = {
    validCsv: 'upload_sample.csv',
    invalidCsv: 'invalid_file.csv',
    testImage: 'test_image.jpg',
    largeFile: 'large_sample.pdf',
  };

  getFilePath(fileName: string): string {
    return path.join(__dirname, '../test_data', fileName);
  }

  // Generate CSV content for testing
  generateCSVContent(rows: any[], headers?: string[]): string {
    let csvContent = '';

    if (headers) {
      csvContent += headers.join(',') + '\n';
    } else if (rows.length > 0) {
      csvContent += Object.keys(rows[0]).join(',') + '\n';
    }

    rows.forEach((row) => {
      const values = Object.values(row).map((value) => {
        if (typeof value === 'string' && value.includes(',')) {
          return `"${value}"`;
        }
        return value;
      });
      csvContent += values.join(',') + '\n';
    });

    return csvContent;
  }

  // Generate record CSV for bulk upload
  generateRecordCSV(count: number = 10): string {
    const data = [];
    for (let i = 0; i < count; i++) {
      data.push({
        name: `Test Record ${this.generateRandomNumber()}`,
        category: 'Standard',
        description: `Auto-generated test record ${i + 1}`,
        status: 'active',
      });
    }
    return this.generateCSVContent(data);
  }
}

// Usage in upload tests
test('bulk upload monitor lists', async ({ app }) => {
  const csvContent = testData.generateMonitorListCSV(5);
  const tempFilePath = path.join(__dirname, '../test_data/temp_upload.csv');

  await test.step('Create temporary CSV file', async () => {
    fs.writeFileSync(tempFilePath, csvContent);
  });

  await test.step('Upload CSV file', async () => {
    await app.uploadPage.uploadFile(tempFilePath);
  });

  await test.step('Verify upload success', async () => {
    await expect(app.uploadPage.successMessage).toBeVisible();
  });

  await test.step('Cleanup temporary file', async () => {
    if (fs.existsSync(tempFilePath)) {
      fs.unlinkSync(tempFilePath);
    }
  });
});
```

### 4. Environment-Specific Data Pattern

```typescript
export class TestData {
  private readonly environment = process.env.TEST_ENV || 'dev';

  // Environment configurations
  getEnvironmentConfig() {
    const configs = {
      dev: {
        baseUrl: 'https://dev.example.com',
        apiUrl: 'https://api-dev.example.com',
        testUserSuffix: 'dev.test',
        features: {
          featureA: true,
          featureB: false,
        },
      },
      staging: {
        baseUrl: 'https://staging.example.com',
        apiUrl: 'https://api-staging.example.com',
        testUserSuffix: 'staging.test',
        features: {
          featureA: true,
          featureB: true,
        },
      },
      prod: {
        baseUrl: 'https://example.com',
        apiUrl: 'https://api.example.com',
        testUserSuffix: 'prod.test',
        features: {
          featureA: true,
          featureB: true,
        },
      },
    };

    return configs[this.environment as keyof typeof configs] || configs.dev;
  }

  // Generate environment-aware test data
  generateEnvironmentTestUser(): UserData {
    const config = this.getEnvironmentConfig();
    const person = this.generatePersonName();

    return {
      id: this.generateUUID(),
      name: person.full,
      email: `${person.first.toLowerCase()}.${person.last.toLowerCase()}@${config.testUserSuffix}`,
      environment: this.environment,
      baseUrl: config.baseUrl,
    };
  }

  // Feature flag aware data
  getDataForFeature(feature: string): any {
    const config = this.getEnvironmentConfig();

    if (!config.features[feature as keyof typeof config.features]) {
      throw new Error(`Feature ${feature} is not enabled in ${this.environment} environment`);
    }

    // Return feature-specific test data
    const featureData = {
      featureA: {
        methods: ['basic', 'advanced', 'fuzzy'],
        filters: ['date', 'category', 'location'],
      },
      featureB: {
        filterTypes: ['boolean', 'range', 'multi-select'],
        operators: ['AND', 'OR', 'NOT'],
      },
    };

    return featureData[feature as keyof typeof featureData] || {};
  }
}

// Usage with environment awareness
test('feature testing with environment-specific data', async ({ app }) => {
  const config = testData.getEnvironmentConfig();

  await test.step('Navigate to application', async () => {
    await app.page.goto(config.baseUrl);
  });

  if (config.features.featureB) {
    await test.step('Use advanced feature', async () => {
      const filterData = testData.getDataForFeature('featureB');
      await app.featurePage.enableAdvancedOptions();
      await app.featurePage.setFilterType(filterData.filterTypes[0]);
    });
  }
});
```

### 5. Data Builder Pattern

```typescript
export class RecordBuilder {
  private data: Partial<RecordData> = {};

  withName(name: string): RecordBuilder {
    this.data.name = name;
    return this;
  }

  withCategory(category: string): RecordBuilder {
    this.data.category = category;
    return this;
  }

  withDescription(description: string): RecordBuilder {
    this.data.description = description;
    return this;
  }

  withTags(tags: string[]): RecordBuilder {
    this.data.tags = tags;
    return this;
  }

  withRandomData(): RecordBuilder {
    const testData = new TestData();
    this.data.name = testData.generateUniqueRecordName();
    this.data.category = 'Standard';
    this.data.description = `Auto-generated record ${testData.generateRandomNumber()}`;
    this.data.tags = ['test', 'automated'];
    return this;
  }

  build(): RecordData {
    if (!this.data.name) {
      throw new Error('Record name is required');
    }

    return {
      id: new TestData().generateUUID(),
      name: this.data.name,
      category: this.data.category || 'General',
      description: this.data.description || '',
      tags: this.data.tags || [],
      createdAt: new TestData().generateTimestamp(),
      status: 'active',
    } as RecordData;
  }
}

// Usage with builder pattern
test('create record with builder', async ({ app }) => {
  const record = new RecordBuilder()
    .withName('Important Record')
    .withCategory('Standard')
    .withDescription('Critical data requiring immediate attention')
    .withTags(['critical', 'high-priority', 'immediate'])
    .build();

  await test.step('Create record', async () => {
    await app.recordPage.createRecordWithData(record);
  });

  await test.step('Verify record creation', async () => {
    await expect(app.recordPage.recordContainer).toContainText(record.name);
  });
});
```

### 6. Data Factory Pattern

```typescript
export class TestDataFactory {
  // Create different types of users
  static createUser(type: UserType): UserData {
    const testData = new TestData();
    const person = testData.generatePersonName();

    const baseUser = {
      id: testData.generateUUID(),
      name: person.full,
      createdAt: testData.generateTimestamp(),
    };

    switch (type) {
      case 'admin':
        return {
          ...baseUser,
          email: `admin.${person.first.toLowerCase()}@company.com`,
          role: 'admin',
          permissions: ['all'],
          department: 'IT',
        };

      case 'standard_user':
        return {
          ...baseUser,
          email: `user.${person.first.toLowerCase()}@company.com`,
          role: 'user',
          permissions: ['read', 'write'],
          department: 'Operations',
        };

      case 'auditor':
        return {
          ...baseUser,
          email: `audit.${person.first.toLowerCase()}@company.com`,
          role: 'viewer',
          permissions: ['read', 'audit'],
          department: 'Audit',
        };

      default:
        throw new Error(`Unknown user type: ${type}`);
    }
  }

  // Create different types of records
  static createRecord(type: RecordType): RecordData {
    const testData = new TestData();

    const baseRecord = {
      id: testData.generateUUID(),
      createdAt: testData.generateTimestamp(),
      status: 'active',
    };

    switch (type) {
      case 'high_priority':
        return {
          ...baseRecord,
          name: `High Priority Record ${testData.generateRandomNumber()}`,
          category: 'Important',
          priority: 'High',
          alertThreshold: 95,
          tags: ['high-priority', 'critical'],
        };

      case 'standard':
        return {
          ...baseRecord,
          name: `Standard Record ${testData.generateRandomNumber()}`,
          category: 'Standard',
          priority: 'Medium',
          alertThreshold: 80,
          tags: ['standard', 'routine'],
        };

      case 'low_priority':
        return {
          ...baseRecord,
          name: `Low Priority Record ${testData.generateRandomNumber()}`,
          category: 'General',
          priority: 'Low',
          alertThreshold: 60,
          tags: ['low-priority', 'general'],
        };

      default:
        throw new Error(`Unknown record type: ${type}`);
    }
  }
}

// Usage with factory pattern
test('create different types of records', async ({ app }) => {
  const highPriorityRecord = TestDataFactory.createRecord('high_priority');
  const standardRecord = TestDataFactory.createRecord('standard');

  await test.step('Create high priority record', async () => {
    await app.recordPage.createRecordWithData(highPriorityRecord);
  });

  await test.step('Create standard record', async () => {
    await app.recordPage.createRecordWithData(standardRecord);
  });

  await test.step('Verify both records created', async () => {
    await expect(app.recordPage.recordContainer).toContainText(highPriorityRecord.name);
    await expect(app.recordPage.recordContainer).toContainText(standardRecord.name);
  });
});
```

## 🎭 Integration with Page Objects

### Passing Data to Page Object Methods

```typescript
// ✅ CORRECT: Pass data as parameters to page object methods
export class RecordPage {
  async createRecordWithBasicInfo(name: string, category: string): Promise<void> {
    await this.nameInput.fill(name);
    await this.categoryDropdown.selectOption(category);
    await this.createButton.click();
  }

  async createRecordWithFullData(recordData: RecordData): Promise<void> {
    await this.nameInput.fill(recordData.name);
    await this.categoryDropdown.selectOption(recordData.category);
    await this.descriptionField.fill(recordData.description);

    if (recordData.tags) {
      for (const tag of recordData.tags) {
        await this.addTag(tag);
      }
    }

    await this.createButton.click();
  }
}

// Usage in tests
test('create record with page object', async ({ app }) => {
  const recordData = testData.generateRecordData();

  await test.step('Navigate to record page', async () => {
    await app.recordPage.goto();
  });

  await test.step('Create record with full data', async () => {
    await app.recordPage.createRecordWithFullData(recordData);
  });

  await test.step('Verify record creation', async () => {
    await expect(app.recordPage.successMessage).toBeVisible();
  });
});
```

---

**This reference guide provides practical, copy-paste examples for the most common test data patterns. Use these as starting points and adapt them to your specific testing needs.**
