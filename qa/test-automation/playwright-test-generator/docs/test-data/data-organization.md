# 🗂️ Data Organization Strategies

## 🎯 Overview

Proper data organization ensures maintainability, discoverability, and scalability of test data. This guide covers strategies for structuring test data in large automation suites.

## 🏗️ Organizational Patterns

### 1. Domain-Based Organization

**Strategy**: Group data by business domains or application features.

**Structure**:

```typescript
export class TestData {
  // Authentication domain
  readonly authentication = {
    users: {
      adminUser: 'admin@company.com',
      regularUser: 'user@company.com',
      viewerUser: 'viewer@company.com',
    },
    credentials: {
      validPassword: 'ValidPass123!',
      invalidPassword: 'invalid',
    },
    tokens: {
      validToken: 'valid-jwt-token',
      expiredToken: 'expired-jwt-token',
      invalidToken: 'invalid-token',
    },
  };

  // Form data domain
  readonly forms = {
    inputs: {
      validText: 'Valid Text Input',
      invalidText: 'Invalid@Text#Input',
      longText: 'Very long text input that exceeds normal limits',
    },
    dropdowns: {
      option1: 'First Option',
      option2: 'Second Option',
      option3: 'Third Option',
    },
    validation: {
      requiredField: 'This field is required',
      invalidFormat: 'Please enter a valid format',
      tooShort: 'Input is too short',
    },
  };

  // Navigation domain
  readonly navigation = {
    pages: {
      homePage: '/home',
      profilePage: '/profile',
      settingsPage: '/settings',
    },
    buttons: {
      save: 'Save',
      cancel: 'Cancel',
      delete: 'Delete',
    },
  };
}
```

**Usage Benefits**:

- Clear domain boundaries
- Easy to locate relevant data
- Supports domain expert collaboration
- Reduces naming conflicts

### 2. Layer-Based Organization

**Strategy**: Organize data by application layers (UI, API, Database).

**Structure**:

```typescript
export class TestData {
  // UI Layer Data
  readonly ui = {
    buttons: {
      submit: 'Submit',
      cancel: 'Cancel',
      delete: 'Delete',
    },
    labels: {
      required: '*',
      optional: '(optional)',
    },
    messages: {
      success: 'Operation completed successfully',
      error: 'An error occurred',
    },
  };

  // API Layer Data
  readonly api = {
    endpoints: {
      users: '/api/v1/users',
      monitorLists: '/api/v1/monitor-lists',
      search: '/api/v1/search',
    },
    headers: {
      contentType: 'application/json',
      authorization: 'Bearer {token}',
    },
    statusCodes: {
      success: 200,
      created: 201,
      badRequest: 400,
      unauthorized: 401,
    },
  };

  // Database Layer Data
  readonly database = {
    tables: {
      users: 'users',
      monitorLists: 'monitor_lists',
      searchResults: 'search_results',
    },
    queries: {
      findUserById: 'SELECT * FROM users WHERE id = ?',
      deleteTestData: 'DELETE FROM monitor_lists WHERE name LIKE "00 Test%"',
    },
  };
}
```

### 3. Test-Type Organization

**Strategy**: Group data by test type and purpose.

**Structure**:

```typescript
export class TestData {
  // Positive test data
  readonly positive = {
    validInputs: {
      email: 'valid@example.com',
      password: 'ValidPass123!',
      name: 'John Doe',
    },
    successScenarios: {
      loginSuccess: {
        email: 'user@example.com',
        password: 'password123',
        expectedRedirect: '/dashboard',
      },
    },
  };

  // Negative test data
  readonly negative = {
    invalidInputs: {
      emptyEmail: '',
      invalidEmail: 'not-an-email',
      shortPassword: '123',
      longName: 'a'.repeat(101),
    },
    errorScenarios: {
      loginFailure: {
        email: 'user@example.com',
        password: 'wrongpassword',
        expectedError: 'Invalid credentials',
      },
    },
  };

  // Boundary test data
  readonly boundary = {
    minValues: {
      passwordLength: 8,
      nameLength: 1,
      searchTermLength: 2,
    },
    maxValues: {
      passwordLength: 50,
      nameLength: 100,
      searchTermLength: 1000,
    },
  };

  // Performance test data
  readonly performance = {
    largeSets: {
      userList: Array.from({ length: 1000 }, (_, i) => `user${i}@example.com`),
      searchTerms: Array.from({ length: 100 }, (_, i) => `search term ${i}`),
    },
  };
}
```

## 📁 File Organization Strategies

### Single File Approach (Current)

**Best for**: Small to medium projects
**File**: `test.data.ts`

**Pros**:

- Simple to maintain
- Easy to import
- No circular dependencies

**Cons**:

- Can become large
- Merge conflicts in teams
- Less modular

### Multi-File Approach

**Best for**: Large projects with multiple domains

**Structure**:

```
test_data/
├── index.ts                 # Main export file
├── domains/
│   ├── authentication.data.ts
│   ├── search.data.ts
│   ├── monitor-lists.data.ts
│   └── admin.data.ts
├── shared/
│   ├── common.data.ts
│   ├── constants.data.ts
│   └── generators.data.ts
└── environments/
    ├── dev.data.ts
    ├── staging.data.ts
    └── prod.data.ts
```

**Implementation**:

```typescript
// domains/authentication.data.ts
export class AuthenticationData {
  readonly users = {
    admin: 'admin@example.com',
    user: 'user@example.com',
  };

  readonly passwords = {
    valid: 'ValidPass123!',
    invalid: 'invalid',
  };
}

// domains/search.data.ts
export class SearchData {
  readonly terms = {
    barackObama: 'Barack Obama',
    billGates: 'Bill Gates',
  };

  readonly periods = {
    lastWeek: 'Last Week',
    lastMonth: 'Last Month',
  };
}

// index.ts - Main aggregator
import { AuthenticationData } from './domains/authentication.data';
import { SearchData } from './domains/search.data';
import { CommonData } from './shared/common.data';

export class TestData extends CommonData {
  readonly auth = new AuthenticationData();
  readonly search = new SearchData();

  constructor() {
    super();
  }
}
```

**Usage**:

```typescript
import { TestData } from '../test_data';

const testData = new TestData();
await app.loginPage.login(testData.auth.users.admin, testData.auth.passwords.valid);
await app.searchPage.performSearch(testData.search.terms.barackObama);
```

## 🔗 Relationship Management

### Data Dependencies

**Strategy**: Manage relationships between related data elements.

```typescript
export class TestData {
  readonly users = {
    admin: {
      id: 1,
      email: 'admin@example.com',
      role: 'admin',
      monitorLists: ['admin-list-1', 'admin-list-2'],
    },
    user: {
      id: 2,
      email: 'user@example.com',
      role: 'user',
      monitorLists: ['user-list-1'],
    },
  };

  readonly monitorLists = {
    'admin-list-1': {
      id: 101,
      name: 'Admin Test List 1',
      ownerId: 1, // References users.admin.id
      category: 'Adverse Media',
    },
    'user-list-1': {
      id: 102,
      name: 'User Test List 1',
      ownerId: 2, // References users.user.id
      category: 'Watchlists',
    },
  };

  // Helper method to get user's lists
  getUserLists(userId: number) {
    return Object.values(this.monitorLists).filter((list) => list.ownerId === userId);
  }

  // Helper method to get list owner
  getListOwner(listId: number) {
    const list = Object.values(this.monitorLists).find((l) => l.id === listId);
    return Object.values(this.users).find((user) => user.id === list?.ownerId);
  }
}
```

### Data Hierarchies

**Strategy**: Represent hierarchical relationships in data.

```typescript
export class TestData {
  readonly organizationHierarchy = {
    company: {
      name: 'Test Corporation',
      id: 'corp-001',
      departments: {
        engineering: {
          name: 'Engineering',
          id: 'dept-eng',
          teams: {
            frontend: {
              name: 'Frontend Team',
              id: 'team-fe',
              members: ['john.doe', 'jane.smith'],
            },
            backend: {
              name: 'Backend Team',
              id: 'team-be',
              members: ['mike.johnson', 'sarah.wilson'],
            },
          },
        },
        sales: {
          name: 'Sales',
          id: 'dept-sales',
          teams: {
            inside: {
              name: 'Inside Sales',
              id: 'team-inside',
              members: ['tom.brown', 'lisa.davis'],
            },
          },
        },
      },
    },
  };

  // Helper methods for hierarchy navigation
  getAllTeams() {
    const teams = [];
    const departments = this.organizationHierarchy.company.departments;

    Object.values(departments).forEach((dept) => {
      Object.values(dept.teams).forEach((team) => {
        teams.push(team);
      });
    });

    return teams;
  }

  getAllMembers() {
    return this.getAllTeams().flatMap((team) => team.members);
  }

  findTeamByMember(memberName: string) {
    return this.getAllTeams().find((team) => team.members.includes(memberName));
  }
}
```

## 🎨 Naming Conventions

### Consistent Naming Strategy

```typescript
export class TestData {
  // Use descriptive, domain-specific names
  readonly searchTerms = {
    // Not just 'terms'
    politicalFigures: {
      barackObama: 'Barack Obama',
      donaldTrump: 'Donald Trump',
    },
    businessLeaders: {
      billGates: 'Bill Gates',
      elonMusk: 'Elon Musk',
    },
  };

  // Use verb prefixes for actions
  readonly userActions = {
    createUser: 'Create New User',
    updateProfile: 'Update Profile',
    deleteAccount: 'Delete Account',
  };

  // Use state suffixes for conditions
  readonly accountStates = {
    userActive: 'active',
    userSuspended: 'suspended',
    userPending: 'pending',
  };

  // Use type suffixes for data types
  readonly validationMessages = {
    emailInvalidFormat: 'Please enter a valid email address',
    passwordTooShort: 'Password must be at least 8 characters',
    fieldRequired: 'This field is required',
  };
}
```

### Naming Patterns

| Pattern              | Example           | Use Case             |
| -------------------- | ----------------- | -------------------- |
| `{domain}{Type}`     | `userCredentials` | Domain-specific data |
| `{action}{Object}`   | `createUser`      | Action-related data  |
| `{object}{State}`    | `accountActive`   | State descriptions   |
| `{validation}{Rule}` | `emailRequired`   | Validation scenarios |
| `{test}{Scenario}`   | `loginSuccess`    | Test scenarios       |

## 🔧 Data Access Patterns

### Centralized Access

```typescript
export class TestDataAccessor {
  private static instance: TestData;

  static getInstance(): TestData {
    if (!TestDataAccessor.instance) {
      TestDataAccessor.instance = new TestData();
    }
    return TestDataAccessor.instance;
  }

  // Convenient access methods
  static getUser(type: 'admin' | 'user' | 'viewer') {
    return this.getInstance().users[type];
  }

  static getSearchTerm(category: string, term: string) {
    return this.getInstance().search.terms[category]?.[term];
  }

  static generateUniqueData(prefix: string) {
    return `${prefix}_${this.getInstance().generateRandomNumber()}`;
  }
}

// Usage
const adminUser = TestDataAccessor.getUser('admin');
const searchTerm = TestDataAccessor.getSearchTerm('people', 'barackObama');
```

### Context-Aware Access

```typescript
export class ContextualTestData extends TestData {
  private currentContext: string = 'default';

  setContext(context: string) {
    this.currentContext = context;
    return this;
  }

  getContextualData(key: string) {
    const contextData = {
      login: this.authentication,
      search: this.search,
      admin: this.admin,
      default: this,
    };

    return contextData[this.currentContext] || contextData['default'];
  }
}

// Usage
const testData = new ContextualTestData();
const loginData = testData.setContext('login').getContextualData('users');
```

## 🚨 Organization Best Practices

### ✅ **DO**

- **Group related data** logically by domain or feature
- **Use consistent naming** conventions throughout
- **Implement helper methods** for complex data relationships
- **Document data dependencies** and relationships
- **Separate static and dynamic** data clearly
- **Version control data changes** carefully

### 🚫 **DON'T**

- **Mix different domains** in the same data group
- **Use unclear abbreviations** in names
- **Create circular dependencies** between data files
- **Ignore data relationships** and constraints
- **Duplicate data** across multiple locations
- **Use magic numbers** without explanatory constants

## 📊 Organization Decision Matrix

| Project Size | Team Size | Domains | Recommended Approach              |
| ------------ | --------- | ------- | --------------------------------- |
| Small        | 1-3       | 1-2     | Single file with domain grouping  |
| Medium       | 4-8       | 3-5     | Multi-file with domain separation |
| Large        | 9+        | 6+      | Multi-file with team ownership    |

## 🔄 Migration Strategies

### From Single File to Multi-File

1. **Identify domains** in current data structure
2. **Create domain-specific files** with extracted data
3. **Update imports** in test files gradually
4. **Maintain backward compatibility** during transition
5. **Remove old references** once migration is complete

### Example Migration:

```typescript
// Before (single file)
export class TestData {
  readonly terms = {
    /* search data */
  };
  readonly users = {
    /* auth data */
  };
  readonly lists = {
    /* monitor list data */
  };
}

// After (multi-file)
// search.data.ts
export class SearchData {
  readonly terms = {
    /* moved from TestData */
  };
}

// auth.data.ts
export class AuthData {
  readonly users = {
    /* moved from TestData */
  };
}

// index.ts
export class TestData {
  readonly search = new SearchData();
  readonly auth = new AuthData();
  // ... other domains
}
```

---

**Next**: Explore the [Quick References](../quick-references/test-data-checklist.md) for practical implementation checklists and examples.
