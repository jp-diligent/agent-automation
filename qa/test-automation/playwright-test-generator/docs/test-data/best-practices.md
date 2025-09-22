# ⭐ Test Data Best Practices

## 🎯 Overview

This guide consolidates the most important best practices for test data management, providing clear guidelines for creating maintainable, reliable, and efficient test data strategies.

## 🏆 Core Principles

### 1. Data Isolation and Independence

**Principle**: Each test should be completely independent with its own data.

```typescript
// ✅ CORRECT: Each test uses unique data
test('create new record', async ({ app }) => {
  const uniqueName = `Test Record ${testData.generateRandomNumber()}`;
  await app.recordPage.createRecord(uniqueName);
});

test('edit existing record', async ({ app }) => {
  const anotherUniqueName = `Edit Test ${testData.generateRandomNumber()}`;
  await app.recordPage.createRecord(anotherUniqueName);
  await app.recordPage.editRecord(anotherUniqueName, 'New Name');
});

// ❌ WRONG: Tests sharing mutable data
let sharedRecordName = 'Shared Test Record';
test('create new record', async ({ app }) => {
  await app.recordPage.createRecord(sharedRecordName); // First test creates
});

test('edit existing record', async ({ app }) => {
  sharedRecordName = 'Modified Name'; // Second test modifies shared data
  await app.recordPage.editRecord(sharedRecordName, 'New Name');
});
```

### 2. Centralized Data Management

**Principle**: Maintain all test data in a centralized, well-organized structure.

```typescript
// ✅ CORRECT: Centralized test data class
export class TestData {
  readonly forms = {
    inputs: {
      validText: 'Valid Input Text',
      invalidText: 'Invalid Input',
    },
    dropdowns: {
      firstOption: 'First Option',
      secondOption: 'Second Option',
    },
  };

  generateUniqueRecordName(): string {
    return `Test Record ${this.generateRandomNumber()}`;
  }
}

// Usage in tests
const testData = new TestData();
await app.formPage.fillInput(testData.forms.inputs.validText);

// ❌ WRONG: Scattered hardcoded data
test('form test 1', async ({ app }) => {
  await app.formPage.fillInput('Valid Input Text'); // Hardcoded
});

test('form test 2', async ({ app }) => {
  await app.formPage.fillInput('Another Input'); // Hardcoded elsewhere
});
```

### 3. Descriptive and Meaningful Data

**Principle**: Use data that clearly indicates its purpose and usage context.

```typescript
// ✅ CORRECT: Descriptive data with clear purpose
export class TestData {
  readonly users = {
    adminWithFullPermissions: {
      email: 'admin.full@company.com',
      role: 'admin',
      permissions: ['create', 'read', 'update', 'delete', 'admin'],
    },
    regularUserWithLimitedAccess: {
      email: 'user.limited@company.com',
      role: 'user',
      permissions: ['read', 'create'],
    },
    viewerWithReadOnlyAccess: {
      email: 'viewer.readonly@company.com',
      role: 'viewer',
      permissions: ['read'],
    },
  };

  readonly testScenarios = {
    validLoginCredentials: {
      email: 'valid.user@company.com',
      password: 'ValidPassword123!',
      expectedOutcome: 'successful_login',
    },
    invalidLoginCredentials: {
      email: 'invalid.user@company.com',
      password: 'WrongPassword',
      expectedOutcome: 'login_failure',
    },
  };
}

// ❌ WRONG: Unclear, generic data
export class TestData {
  readonly users = {
    user1: { email: 'test1@test.com' }, // What type of user?
    user2: { email: 'test2@test.com' }, // What's their role?
    user3: { email: 'test3@test.com' }, // What permissions?
  };

  readonly data = {
    string1: 'some value', // What is this for?
    string2: 'other value', // When should I use this?
    number1: 123, // What does this represent?
  };
}
```

## 🔧 Implementation Best Practices

### 1. Data Generation Strategies

**Dynamic vs Static Data**:

```typescript
export class TestData {
  // ✅ Static data for stable references
  readonly staticData = {
    searchTerms: {
      celebrityPolitician: 'Barack Obama',
      techBillionaire: 'Bill Gates',
      businessMagnate: 'Elon Musk',
    },
    categories: {
      adverseMedia: 'Adverse Media',
      watchlists: 'Watchlists',
      pep: 'PEP',
    },
  };

  // ✅ Dynamic data for unique test instances
  generateUniqueUserData(): UserData {
    return {
      id: this.generateUUID(),
      name: this.generatePersonName().full,
      email: this.generateEmail(),
      createdAt: this.generateTimestamp(),
      listName: `Test List ${this.generateRandomNumber()}`,
    };
  }

  // ✅ Template-based generation for consistent structure
  generateMonitorListFromTemplate(type: 'high_risk' | 'standard' | 'low_priority'): MonitorList {
    const templates = {
      high_risk: {
        namePrefix: 'Critical Alert List',
        category: 'Adverse Media',
        priority: 'High',
        alertThreshold: 95,
      },
      standard: {
        namePrefix: 'Standard Monitor List',
        category: 'Watchlists',
        priority: 'Medium',
        alertThreshold: 80,
      },
      low_priority: {
        namePrefix: 'Review List',
        category: 'PEP',
        priority: 'Low',
        alertThreshold: 60,
      },
    };

    const template = templates[type];
    return {
      name: `${template.namePrefix} ${this.generateRandomNumber()}`,
      category: template.category,
      priority: template.priority,
      alertThreshold: template.alertThreshold,
      createdDate: this.generateTimestamp(),
    };
  }
}
```

### 2. Data Validation and Consistency

**Implement validation for generated data**:

```typescript
export class TestData {
  // ✅ Data validation methods
  validateEmailFormat(email: string): boolean {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return emailRegex.test(email);
  }

  validatePasswordStrength(password: string): { valid: boolean; errors: string[] } {
    const errors: string[] = [];

    if (password.length < 8) errors.push('Password must be at least 8 characters');
    if (!/[A-Z]/.test(password)) errors.push('Password must contain uppercase letter');
    if (!/[a-z]/.test(password)) errors.push('Password must contain lowercase letter');
    if (!/\d/.test(password)) errors.push('Password must contain number');
    if (!/[!@#$%^&*()_+\-=\[\]{}|;:,.<>?]/.test(password)) errors.push('Password must contain special character');

    return { valid: errors.length === 0, errors };
  }

  // ✅ Generate valid data with validation
  generateValidPassword(): string {
    let password: string;
    let attempts = 0;
    const maxAttempts = 10;

    do {
      password = this.generateRandomPassword();
      attempts++;
    } while (!this.validatePasswordStrength(password).valid && attempts < maxAttempts);

    if (attempts >= maxAttempts) {
      throw new Error('Could not generate valid password after maximum attempts');
    }

    return password;
  }

  // ✅ Consistent data relationships
  generateUserWithConsistentData(): UserData {
    const person = this.generatePersonName();
    const email = this.generateEmailFromName(person);
    const userData = {
      id: this.generateUUID(),
      firstName: person.first,
      lastName: person.last,
      fullName: person.full,
      email: email,
      username: email.split('@')[0],
      password: this.generateValidPassword(),
      createdAt: this.generateTimestamp(),
    };

    // Validate consistency
    if (!this.validateEmailFormat(userData.email)) {
      throw new Error('Generated email is invalid');
    }

    if (userData.fullName !== `${userData.firstName} ${userData.lastName}`) {
      throw new Error('Name fields are inconsistent');
    }

    return userData;
  }
}
```

### 3. Environment-Aware Data Management

**Handle different environments properly**:

```typescript
export class TestData {
  private readonly environment: string;

  constructor() {
    this.environment = process.env.TEST_ENV || 'dev';
  }

  // ✅ Environment-specific data
  getEnvironmentConfig() {
    const configs = {
      dev: {
        baseUrl: 'https://dev.example.com',
        apiUrl: 'https://api-dev.example.com',
        dbPrefix: 'dev_',
        features: {
          newSearchEnabled: true,
          advancedFiltersEnabled: false,
        },
      },
      staging: {
        baseUrl: 'https://staging.example.com',
        apiUrl: 'https://api-staging.example.com',
        dbPrefix: 'staging_',
        features: {
          newSearchEnabled: true,
          advancedFiltersEnabled: true,
        },
      },
      prod: {
        baseUrl: 'https://example.com',
        apiUrl: 'https://api.example.com',
        dbPrefix: '',
        features: {
          newSearchEnabled: true,
          advancedFiltersEnabled: true,
        },
      },
    };

    return configs[this.environment as keyof typeof configs] || configs.dev;
  }

  // ✅ Environment-aware test data
  generateTestDataForEnvironment() {
    const config = this.getEnvironmentConfig();

    return {
      testUserPrefix: `${config.dbPrefix}test_user_`,
      testListPrefix: `${config.dbPrefix}test_list_`,
      enabledFeatures: config.features,
    };
  }

  // ✅ Feature flag aware data
  getDataForFeature(feature: string) {
    const config = this.getEnvironmentConfig();

    if (!config.features[feature as keyof typeof config.features]) {
      throw new Error(`Feature ${feature} is not enabled in ${this.environment} environment`);
    }

    // Return feature-specific test data
    return this.getFeatureTestData(feature);
  }
}
```

## 📋 Data Organization Best Practices

### 1. Logical Grouping and Hierarchy

```typescript
// ✅ CORRECT: Well-organized hierarchical structure
export class TestData {
  readonly authentication = {
    validCredentials: {
      admin: { email: 'admin@company.com', password: 'AdminPass123!' },
      user: { email: 'user@company.com', password: 'UserPass123!' },
      viewer: { email: 'viewer@company.com', password: 'ViewPass123!' },
    },
    invalidCredentials: {
      wrongPassword: { email: 'user@company.com', password: 'WrongPass' },
      wrongEmail: { email: 'wrong@company.com', password: 'UserPass123!' },
      emptyFields: { email: '', password: '' },
    },
    securityPolicies: {
      passwordRequirements: {
        minLength: 8,
        requireUppercase: true,
        requireNumbers: true,
        requireSpecialChars: true,
      },
      lockoutPolicy: {
        maxAttempts: 3,
        lockoutDuration: '15 minutes',
      },
    },
  };

  readonly search = {
    queries: {
      people: {
        politicians: ['Barack Obama', 'Donald Trump', 'Joe Biden'],
        businessLeaders: ['Bill Gates', 'Elon Musk', 'Jeff Bezos'],
        celebrities: ['Tom Hanks', 'Oprah Winfrey', 'Leonardo DiCaprio'],
      },
      entities: {
        corporations: ['Microsoft Corporation', 'Apple Inc.', 'Google LLC'],
        nonprofits: ['Red Cross', 'United Nations', 'Doctors Without Borders'],
        governments: ['United States Government', 'European Union', 'United Nations'],
      },
    },
    filters: {
      timeRanges: {
        recent: 'Last 30 Days',
        quarterly: 'Last 3 Months',
        annual: 'Last Year',
        historical: 'All Time',
      },
      categories: {
        compliance: ['PEP', 'Sanctions', 'Watchlists'],
        media: ['Adverse Media', 'Financial News', 'General News'],
        geographic: ['North America', 'Europe', 'Asia Pacific'],
      },
    },
  };
}
```

### 2. Naming Conventions

```typescript
// ✅ CORRECT: Consistent, descriptive naming
export class TestData {
  // Use clear prefixes for different data types
  readonly validationMessages = {
    emailRequired: 'Email address is required',
    emailInvalidFormat: 'Please enter a valid email address',
    passwordTooShort: 'Password must be at least 8 characters',
    passwordMissingNumber: 'Password must contain at least one number',
  };

  readonly buttonLabels = {
    submitForm: 'Submit',
    cancelAction: 'Cancel',
    deleteItem: 'Delete',
    editItem: 'Edit',
  };

  readonly pageUrls = {
    loginPage: '/auth/login',
    dashboardPage: '/dashboard',
    searchPage: '/search',
    profilePage: '/profile',
  };

  // Use descriptive method names
  generateUniqueMonitorListName(): string {
    /* */
  }
  generateValidSearchTerm(): string {
    /* */
  }
  generateRandomUserEmail(): string {
    /* */
  }
  generateCurrentTimestamp(): string {
    /* */
  }

  // Use clear parameter names
  createUserWithRole(role: 'admin' | 'user' | 'viewer'): UserData {
    /* */
  }
  generateDateInRange(startDate: string, endDate: string): string {
    /* */
  }
  createListWithCategory(category: 'PEP' | 'Watchlist' | 'Adverse Media'): ListData {
    /* */
  }
}
```

## 🚨 Common Anti-Patterns to Avoid

### 1. Data Coupling and Dependencies

```typescript
// ❌ WRONG: Tests dependent on execution order
let globalCounter = 0;
const sharedList = 'Global Test List';

test('create list', async ({ app }) => {
  globalCounter++;
  await app.monitorPage.createList(`${sharedList} ${globalCounter}`);
});

test('edit list', async ({ app }) => {
  // This test depends on the previous test running first
  await app.monitorPage.editList(`${sharedList} ${globalCounter}`, 'New Name');
});

// ✅ CORRECT: Independent tests
test('create list', async ({ app }) => {
  const uniqueListName = testData.generateUniqueListName();
  await app.monitorPage.createList(uniqueListName);
});

test('edit list', async ({ app }) => {
  const uniqueListName = testData.generateUniqueListName();
  await app.monitorPage.createList(uniqueListName);
  await app.monitorPage.editList(uniqueListName, 'New Name');
});
```

### 2. Hardcoded Values

```typescript
// ❌ WRONG: Hardcoded magic numbers and strings
test('create user profile', async ({ app }) => {
  await app.profilePage.fillName('John Doe'); // Hardcoded name
  await app.profilePage.fillEmail('john@test.com'); // Hardcoded email
  await app.profilePage.fillAge(25); // Magic number
  await app.profilePage.selectRole('admin'); // Hardcoded role
});

// ✅ CORRECT: Centralized, meaningful data
test('create user profile', async ({ app }) => {
  const userData = testData.generateValidUserData();
  await app.profilePage.fillName(userData.fullName);
  await app.profilePage.fillEmail(userData.email);
  await app.profilePage.fillAge(userData.age);
  await app.profilePage.selectRole(userData.role);
});
```

### 3. Poor Error Handling

```typescript
// ❌ WRONG: Silent failures in data generation
generateEmailAddress(): string {
  try {
    return this.generateEmail();
  } catch (error) {
    return 'fallback@test.com'; // Silent failure
  }
}

// ✅ CORRECT: Proper error handling with context
generateEmailAddress(): string {
  try {
    const email = this.generateEmail();
    if (!this.validateEmailFormat(email)) {
      throw new Error(`Generated invalid email format: ${email}`);
    }
    return email;
  } catch (error) {
    throw new Error(`Failed to generate valid email address: ${error.message}`);
  }
}
```

### 4. Inconsistent Data Types

```typescript
// ❌ WRONG: Inconsistent types and formats
export class TestData {
  readonly dates = {
    startDate: '2023-01-01', // String format
    endDate: new Date('2023-12-31'), // Date object
    timestamp: 1672531200000, // Unix timestamp
  };

  readonly ids = {
    userId: 123, // Number
    listId: 'list-456', // String
    sessionId: 'session_789', // Different string format
  };
}

// ✅ CORRECT: Consistent types and formats
export class TestData {
  readonly dates = {
    startDate: '2023-01-01',
    endDate: '2023-12-31',
    currentDate: '2023-06-15',
  };

  readonly ids = {
    userId: 'user-123',
    listId: 'list-456',
    sessionId: 'session-789',
  };

  // Provide conversion methods when needed
  convertToDate(dateString: string): Date {
    return new Date(dateString);
  }

  convertToTimestamp(dateString: string): number {
    return new Date(dateString).getTime();
  }
}
```

## 🔄 Data Lifecycle Management

### 1. Test Setup and Teardown

```typescript
export class TestDataLifecycle {
  private createdResources: string[] = [];

  // ✅ Track created resources for cleanup
  async createTestResource(type: string, data: any): Promise<string> {
    const resourceId = await this.createResource(type, data);
    this.createdResources.push(resourceId);
    return resourceId;
  }

  // ✅ Clean up all created resources
  async cleanupTestResources(): Promise<void> {
    const cleanupPromises = this.createdResources.map((id) =>
      this.deleteResource(id).catch((error) => console.warn(`Failed to cleanup resource ${id}:`, error)),
    );

    await Promise.all(cleanupPromises);
    this.createdResources = [];
  }

  // ✅ Isolate test data by test run
  generateTestRunId(): string {
    return `test_run_${Date.now()}_${Math.random().toString(36).substring(2)}`;
  }
}

// Usage in tests
test.describe('Monitor List Tests', () => {
  const dataLifecycle = new TestDataLifecycle();
  const testRunId = dataLifecycle.generateTestRunId();

  test.afterEach(async () => {
    await dataLifecycle.cleanupTestResources();
  });

  test('create monitor list', async ({ app }) => {
    const listData = testData.generateMonitorListData();
    listData.name = `${testRunId}_${listData.name}`;

    const listId = await dataLifecycle.createTestResource('monitor_list', listData);
    await app.monitorPage.verifyListExists(listId);
  });
});
```

### 2. Data Versioning and Migration

```typescript
// ✅ Handle data format changes gracefully
export class TestDataVersionManager {
  private readonly currentVersion = '2.1.0';

  migrateTestData(data: any, fromVersion: string): any {
    if (this.compareVersions(fromVersion, '2.0.0') < 0) {
      data = this.migrateFrom1xTo2x(data);
    }

    if (this.compareVersions(fromVersion, '2.1.0') < 0) {
      data = this.migrateFrom20To21(data);
    }

    return data;
  }

  private migrateFrom1xTo2x(data: any): any {
    // Handle breaking changes from v1.x to v2.x
    if (data.userRoles) {
      data.users = data.userRoles.map((role: any) => ({
        role: role,
        permissions: this.mapRoleToPermissions(role),
      }));
      delete data.userRoles;
    }
    return data;
  }

  private migrateFrom20To21(data: any): any {
    // Handle changes from v2.0 to v2.1
    if (data.searchTerms && !Array.isArray(data.searchTerms)) {
      data.searchTerms = Object.values(data.searchTerms);
    }
    return data;
  }
}
```

## 📊 Performance Best Practices

### 1. Lazy Loading and Caching

```typescript
export class TestData {
  private _cachedLargeDataset: any[] | null = null;
  private _cachedUserProfiles: Map<string, UserProfile> = new Map();

  // ✅ Lazy load expensive data
  getLargeDataset(): any[] {
    if (!this._cachedLargeDataset) {
      console.log('Generating large dataset (one-time operation)...');
      this._cachedLargeDataset = this.generateLargeDataset();
    }
    return this._cachedLargeDataset;
  }

  // ✅ Cache generated user profiles
  getUserProfile(role: string): UserProfile {
    if (!this._cachedUserProfiles.has(role)) {
      this._cachedUserProfiles.set(role, this.generateUserProfile(role));
    }
    return this._cachedUserProfiles.get(role)!;
  }

  // ✅ Clear caches when needed
  clearCaches(): void {
    this._cachedLargeDataset = null;
    this._cachedUserProfiles.clear();
  }
}
```

### 2. Batch Operations

```typescript
export class TestData {
  // ✅ Generate data in batches for better performance
  generateUsersBatch(count: number = 100): UserData[] {
    console.log(`Generating ${count} users in batch...`);
    return Array.from({ length: count }, () => this.generateUserData());
  }

  // ✅ Prepare test data efficiently
  prepareTestSuite(): TestSuiteData {
    const startTime = Date.now();

    const suiteData = {
      users: this.generateUsersBatch(50),
      lists: this.generateListsBatch(20),
      searchTerms: this.generateSearchTermsBatch(100),
    };

    const duration = Date.now() - startTime;
    console.log(`Test suite data prepared in ${duration}ms`);

    return suiteData;
  }
}
```

## 🎯 Quality Assurance Checklist

### Pre-Test Validation

- [ ] All required test data is available and valid
- [ ] Data relationships are consistent
- [ ] Environment-specific data is loaded correctly
- [ ] File dependencies are resolved
- [ ] Data generators are working properly

### During Test Execution

- [ ] Unique identifiers are generated for each test
- [ ] Data isolation is maintained between tests
- [ ] Error handling works correctly for invalid data
- [ ] Performance is acceptable for large datasets
- [ ] Memory usage is reasonable

### Post-Test Cleanup

- [ ] All generated test data is cleaned up
- [ ] Temporary files are removed
- [ ] Database records are deleted
- [ ] Cache is cleared appropriately
- [ ] Resource cleanup completed successfully

---

**Next**: Review [Common Patterns](common-patterns.md) for additional implementation examples and advanced techniques.
