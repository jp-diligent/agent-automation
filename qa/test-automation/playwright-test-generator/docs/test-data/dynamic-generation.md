# ⚡ Dynamic Generation Techniques

## 🎯 Overview

Dynamic data generation is crucial for creating unique, non-conflicting test data that ensures test isolation and reliability. This guide covers techniques for generating realistic and useful test data at runtime.

## 🔧 Core Generation Methods

### 1. Random Identifiers

**Purpose**: Generate unique identifiers to avoid test data conflicts.

```typescript
export class TestData {
  // Basic random string generation
  generateRandomNumber(): string {
    return Math.random().toString(36).substring(2);
  }

  // Enhanced random string with custom length
  generateRandomString(length: number = 10): string {
    const characters =
      "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789";
    let result = "";
    for (let i = 0; i < length; i++) {
      result += characters.charAt(
        Math.floor(Math.random() * characters.length)
      );
    }
    return result;
  }

  // UUID-like identifier generation
  generateUUID(): string {
    return "xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx".replace(
      /[xy]/g,
      function (c) {
        const r = (Math.random() * 16) | 0;
        const v = c === "x" ? r : (r & 0x3) | 0x8;
        return v.toString(16);
      }
    );
  }

  // Sequential ID with prefix
  private static counter: number = 0;
  generateSequentialId(prefix: string = "test"): string {
    return `${prefix}_${++TestData.counter}_${Date.now()}`;
  }

  // Fisher-Yates shuffle algorithm for proper randomization
  private shuffleArray<T>(array: T[]): T[] {
    const shuffled = [...array]; // Create a copy to avoid mutating original
    for (let i = shuffled.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1));
      [shuffled[i], shuffled[j]] = [shuffled[j], shuffled[i]];
    }
    return shuffled;
  }
}
```

**Usage Examples**:

```typescript
test("create unique monitor list", async ({ app }) => {
  const uniqueListName = `Test List ${testData.generateRandomNumber()}`;
  const uniqueId = testData.generateSequentialId("list");

  await app.monitorPage.createList(uniqueListName, uniqueId);
});
```

### 2. Timestamp Generation

**Purpose**: Generate time-based data for testing date/time functionality.

```typescript
export class TestData {
  // Current timestamp in various formats
  generateTimestamp(): string {
    return new Date().toISOString();
  }

  generateDate(): string {
    return new Date().toISOString().substring(0, 19).replace("T", " ");
  }

  generateDateOnly(): string {
    return new Date().toISOString().substring(0, 10);
  }

  generateTimeOnly(): string {
    return new Date().toTimeString().substring(0, 8);
  }

  // Relative date generation
  generatePastDate(daysAgo: number): string {
    const date = new Date();
    date.setDate(date.getDate() - daysAgo);
    return date.toISOString().substring(0, 10);
  }

  generateFutureDate(daysFromNow: number): string {
    const date = new Date();
    date.setDate(date.getDate() + daysFromNow);
    return date.toISOString().substring(0, 10);
  }

  // Custom date range generation
  generateDateInRange(startDate: string, endDate: string): string {
    const start = new Date(startDate);
    const end = new Date(endDate);
    const randomTime =
      start.getTime() + Math.random() * (end.getTime() - start.getTime());
    return new Date(randomTime).toISOString().substring(0, 10);
  }

  // Business day generation (excluding weekends)
  generateBusinessDate(): string {
    const date = new Date();
    const dayOfWeek = date.getDay();

    // If weekend, move to Monday
    if (dayOfWeek === 0) {
      // Sunday
      date.setDate(date.getDate() + 1);
    } else if (dayOfWeek === 6) {
      // Saturday
      date.setDate(date.getDate() + 2);
    }

    return date.toISOString().substring(0, 10);
  }
}
```

**Usage Examples**:

```typescript
test("search with date range", async ({ app }) => {
  const startDate = testData.generatePastDate(30);
  const endDate = testData.generateDate();

  await app.searchPage.setDateRange(startDate, endDate);
});
```

### 3. Realistic Data Generation

**Purpose**: Generate realistic-looking test data that mimics real user input.

```typescript
export class TestData {
  // Name generation
  private readonly firstNames = [
    "John",
    "Jane",
    "Michael",
    "Sarah",
    "David",
    "Emma",
    "Chris",
    "Lisa",
  ];
  private readonly lastNames = [
    "Smith",
    "Johnson",
    "Williams",
    "Brown",
    "Jones",
    "Garcia",
    "Miller",
    "Davis",
  ];

  generatePersonName(): { first: string; last: string; full: string } {
    const first =
      this.firstNames[Math.floor(Math.random() * this.firstNames.length)];
    const last =
      this.lastNames[Math.floor(Math.random() * this.lastNames.length)];
    return {
      first,
      last,
      full: `${first} ${last}`,
    };
  }

  generateCompanyName(): string {
    const prefixes = [
      "Global",
      "International",
      "United",
      "Advanced",
      "Premier",
    ];
    const suffixes = ["Corp", "Inc", "LLC", "Group", "Solutions", "Services"];
    const middle = ["Tech", "Systems", "Industries", "Enterprises", "Holdings"];

    const prefix = prefixes[Math.floor(Math.random() * prefixes.length)];
    const mid = middle[Math.floor(Math.random() * middle.length)];
    const suffix = suffixes[Math.floor(Math.random() * suffixes.length)];

    return `${prefix} ${mid} ${suffix}`;
  }

  // Email generation
  generateEmail(name?: string): string {
    if (!name) {
      const person = this.generatePersonName();
      name = `${person.first.toLowerCase()}.${person.last.toLowerCase()}`;
    }

    const domains = ["example.com", "test.org", "sample.net", "demo.io"];
    const domain = domains[Math.floor(Math.random() * domains.length)];
    const uniqueSuffix = this.generateRandomNumber();

    return `${name}.${uniqueSuffix}@${domain}`;
  }

  // Address generation
  generateAddress(): {
    street: string;
    city: string;
    state: string;
    zipCode: string;
    country: string;
  } {
    const streetNumbers = Math.floor(Math.random() * 9999) + 1;
    const streetNames = [
      "Main St",
      "Oak Ave",
      "First St",
      "Second Ave",
      "Park Rd",
      "Elm St",
    ];
    const cities = [
      "Springfield",
      "Riverside",
      "Franklin",
      "Georgetown",
      "Clinton",
    ];
    const states = ["CA", "NY", "TX", "FL", "IL", "PA", "OH", "GA", "NC", "MI"];

    return {
      street: `${streetNumbers} ${
        streetNames[Math.floor(Math.random() * streetNames.length)]
      }`,
      city: cities[Math.floor(Math.random() * cities.length)],
      state: states[Math.floor(Math.random() * states.length)],
      zipCode: Math.floor(Math.random() * 90000 + 10000).toString(),
      country: "USA",
    };
  }

  // Phone number generation
  generatePhoneNumber(): string {
    const areaCode = Math.floor(Math.random() * 800) + 200;
    const exchange = Math.floor(Math.random() * 800) + 200;
    const number = Math.floor(Math.random() * 9000) + 1000;

    return `(${areaCode}) ${exchange}-${number}`;
  }
}
```

**Usage Examples**:

```typescript
test("create user profile", async ({ app }) => {
  const person = testData.generatePersonName();
  const email = testData.generateEmail(person.first.toLowerCase());
  const address = testData.generateAddress();

  await app.profilePage.fillUserDetails({
    name: person.full,
    email: email,
    address: address.street,
    city: address.city,
  });
});
```

### 4. Domain-Specific Generation

**Purpose**: Generate data specific to your application domain.

```typescript
export class TestData {
  // Monitor list generation
  generateMonitorListData(): {
    name: string;
    description: string;
    category: string;
    tags: string[];
  } {
    const categories = [
      "Adverse Media",
      "Watchlists",
      "PEP",
      "Financial Media",
    ];
    const adjectives = [
      "Critical",
      "Important",
      "Standard",
      "Review",
      "Priority",
    ];
    const subjects = ["Entities", "Individuals", "Organizations", "Companies"];

    const adjective = adjectives[Math.floor(Math.random() * adjectives.length)];
    const subject = subjects[Math.floor(Math.random() * subjects.length)];
    const uniqueId = this.generateRandomNumber();

    return {
      name: `${adjective} ${subject} List ${uniqueId}`,
      description: `Auto-generated test list for ${subject.toLowerCase()} monitoring`,
      category: categories[Math.floor(Math.random() * categories.length)],
      tags: this.generateTags(2, 4),
    };
  }

  // Search term generation
  generateSearchTerms(count: number = 5): string[] {
    const termTypes = [
      () => this.generatePersonName().full,
      () => this.generateCompanyName(),
      () => `${this.generatePersonName().last} Foundation`,
      () => `${this.generateCompanyName()} Holdings`,
    ];

    const terms = [];
    for (let i = 0; i < count; i++) {
      const generator = termTypes[Math.floor(Math.random() * termTypes.length)];
      terms.push(generator());
    }

    return terms;
  }

  // Tag generation
  private generateTags(min: number, max: number): string[] {
    const availableTags = [
      "high-risk",
      "medium-risk",
      "low-risk",
      "urgent",
      "review",
      "compliance",
      "kyc",
      "aml",
      "sanctions",
      "pep",
      "watchlist",
    ];

    const count = Math.floor(Math.random() * (max - min + 1)) + min;
    const shuffled = availableTags.sort(() => 0.5 - Math.random());
    return shuffled.slice(0, count);
  }

  // Report data generation
  generateReportData(): {
    title: string;
    period: string;
    metrics: { [key: string]: number };
    status: string;
  } {
    const titles = [
      "Monthly Compliance Review",
      "Risk Assessment Summary",
      "Alert Processing Report",
      "KYC Review Analysis",
    ];

    return {
      title: titles[Math.floor(Math.random() * titles.length)],
      period: this.generateDateInRange(
        this.generatePastDate(30),
        this.generateDate()
      ),
      metrics: {
        totalAlerts: Math.floor(Math.random() * 1000) + 100,
        processedAlerts: Math.floor(Math.random() * 800) + 50,
        pendingReview: Math.floor(Math.random() * 200) + 10,
        falsePositives: Math.floor(Math.random() * 100) + 5,
      },
      status: ["Draft", "In Review", "Approved", "Published"][
        Math.floor(Math.random() * 4)
      ],
    };
  }
}
```

## 🎲 Advanced Generation Patterns

### 1. Weighted Random Selection

**Purpose**: Generate data with realistic distribution patterns.

```typescript
export class TestData {
  // Weighted selection helper
  private selectWeighted<T>(items: { value: T; weight: number }[]): T {
    const totalWeight = items.reduce((sum, item) => sum + item.weight, 0);
    let random = Math.random() * totalWeight;

    for (const item of items) {
      random -= item.weight;
      if (random <= 0) {
        return item.value;
      }
    }

    return items[items.length - 1].value;
  }

  // Generate user type with realistic distribution
  generateUserType(): string {
    return this.selectWeighted([
      { value: "viewer", weight: 60 }, // 60% viewers
      { value: "user", weight: 30 }, // 30% regular users
      { value: "admin", weight: 10 }, // 10% admins
    ]);
  }

  // Generate risk level with business logic
  generateRiskLevel(): string {
    return this.selectWeighted([
      { value: "low", weight: 50 },
      { value: "medium", weight: 35 },
      { value: "high", weight: 12 },
      { value: "critical", weight: 3 },
    ]);
  }
}
```

### 2. Contextual Generation

**Purpose**: Generate data that makes sense in context.

```typescript
export class TestData {
  // Generate contextually related data
  generateUserWithRole(role: "admin" | "user" | "viewer"): any {
    const person = this.generatePersonName();
    const baseEmail = `${person.first.toLowerCase()}.${person.last.toLowerCase()}`;

    const roleConfigs = {
      admin: {
        email: `admin.${baseEmail}@company.com`,
        permissions: ["create", "read", "update", "delete", "admin"],
        department: "IT",
        title: "System Administrator",
      },
      user: {
        email: `${baseEmail}@company.com`,
        permissions: ["create", "read", "update"],
        department: this.selectWeighted([
          { value: "Compliance", weight: 40 },
          { value: "Risk", weight: 30 },
          { value: "Operations", weight: 30 },
        ]),
        title: "Analyst",
      },
      viewer: {
        email: `view.${baseEmail}@company.com`,
        permissions: ["read"],
        department: "Audit",
        title: "Auditor",
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
      title: config.title,
      createdAt: this.generateTimestamp(),
      lastLogin: this.generatePastDate(Math.floor(Math.random() * 30)),
    };
  }

  // Generate related dataset
  generateMonitorListWithTerms(termCount: number = 5): {
    list: any;
    terms: any[];
  } {
    const list = this.generateMonitorListData();
    const terms = [];

    for (let i = 0; i < termCount; i++) {
      terms.push({
        id: this.generateUUID(),
        listId: list.name, // Reference to parent list
        term: this.generateSearchTerms(1)[0],
        addedDate: this.generatePastDate(Math.floor(Math.random() * 90)),
        status: this.selectWeighted([
          { value: "active", weight: 80 },
          { value: "inactive", weight: 15 },
          { value: "pending", weight: 5 },
        ]),
      });
    }

    return { list, terms };
  }
}
```

### 3. Constraint-Based Generation

**Purpose**: Generate data that satisfies specific business rules.

```typescript
export class TestData {
  // Generate data with constraints
  generateValidPassword(): string {
    const length = Math.floor(Math.random() * 8) + 8; // 8-16 characters
    const uppercase = "ABCDEFGHIJKLMNOPQRSTUVWXYZ";
    const lowercase = "abcdefghijklmnopqrstuvwxyz";
    const numbers = "0123456789";
    const symbols = "!@#$%^&*()_+-=[]{}|;:,.<>?";

    let password = "";

    // Ensure at least one of each required type
    password += uppercase[Math.floor(Math.random() * uppercase.length)];
    password += lowercase[Math.floor(Math.random() * lowercase.length)];
    password += numbers[Math.floor(Math.random() * numbers.length)];
    password += symbols[Math.floor(Math.random() * symbols.length)];

    // Fill remaining length
    const allChars = uppercase + lowercase + numbers + symbols;
    for (let i = password.length; i < length; i++) {
      password += allChars[Math.floor(Math.random() * allChars.length)];
    }

    // Shuffle the password using Fisher-Yates
    return this.shuffleArray(password.split("")).join("");
  }

  // Generate email with domain constraints
  generateCorporateEmail(domain: string = "company.com"): string {
    const person = this.generatePersonName();
    const formats = [
      `${person.first.toLowerCase()}.${person.last.toLowerCase()}`,
      `${person.first.toLowerCase()}${person.last.toLowerCase()}`,
      `${person.first.toLowerCase().charAt(0)}${person.last.toLowerCase()}`,
      `${person.first.toLowerCase()}.${person.last.toLowerCase().charAt(0)}`,
    ];

    const format = formats[Math.floor(Math.random() * formats.length)];
    const uniqueSuffix = this.generateRandomNumber();

    return `${format}${uniqueSuffix}@${domain}`;
  }

  // Generate date within business constraints
  generateValidDateRange(): { start: string; end: string } {
    const endDate = new Date();
    const maxDaysBack = 365; // Maximum 1 year back
    const minDaysBack = 1; // Minimum 1 day back

    const daysBack =
      Math.floor(Math.random() * (maxDaysBack - minDaysBack)) + minDaysBack;
    const startDate = new Date(endDate);
    startDate.setDate(startDate.getDate() - daysBack);

    // Ensure start is before end
    const rangeDays = Math.floor(Math.random() * Math.min(daysBack, 90)) + 1; // Max 90 day range
    const actualEndDate = new Date(startDate);
    actualEndDate.setDate(actualEndDate.getDate() + rangeDays);

    return {
      start: startDate.toISOString().substring(0, 10),
      end: actualEndDate.toISOString().substring(0, 10),
    };
  }
}
```

## 🔄 Generation Strategies

### 1. Seed-Based Generation

**Purpose**: Reproducible random data for debugging.

```typescript
export class TestData {
  private seed: number = 12345;

  setSeed(seed: number): void {
    this.seed = seed;
  }

  // Seeded random number generator
  private seededRandom(): number {
    const x = Math.sin(this.seed++) * 10000;
    return x - Math.floor(x);
  }

  generateSeededString(length: number = 10): string {
    const characters =
      "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789";
    let result = "";
    for (let i = 0; i < length; i++) {
      result += characters.charAt(
        Math.floor(this.seededRandom() * characters.length)
      );
    }
    return result;
  }
}

// Usage for reproducible tests
test("reproducible data test", async ({ app }) => {
  testData.setSeed(42); // Same seed = same data
  const name1 = testData.generateSeededString();

  testData.setSeed(42); // Reset to same seed
  const name2 = testData.generateSeededString();

  expect(name1).toBe(name2); // Will be identical
});
```

### 2. Template-Based Generation

**Purpose**: Generate data using predefined templates.

```typescript
export class TestData {
  private readonly templates = {
    person: {
      individual: "{firstName} {lastName}",
      formal: "{title} {firstName} {lastName}",
      business: "{firstName} {lastName}, {title}",
    },
    company: {
      corp: "{adjective} {industry} Corporation",
      llc: "{adjective} {industry} LLC",
      inc: "{adjective} {industry} Inc.",
    },
  };

  generateFromTemplate(
    template: string,
    variables: { [key: string]: string[] }
  ): string {
    let result = template;

    Object.keys(variables).forEach((key) => {
      const values = variables[key];
      const value = values[Math.floor(Math.random() * values.length)];
      result = result.replace(`{${key}}`, value);
    });

    return result;
  }

  generatePersonFromTemplate(
    type: "individual" | "formal" | "business" = "individual"
  ): string {
    return this.generateFromTemplate(this.templates.person[type], {
      firstName: ["John", "Jane", "Michael", "Sarah"],
      lastName: ["Smith", "Johnson", "Williams", "Brown"],
      title: ["Dr.", "Mr.", "Ms.", "Prof."],
    });
  }
}
```

## 🚨 Generation Best Practices

### ✅ **DO**

- **Use unique identifiers** to prevent test conflicts
- **Generate realistic data** that matches production patterns
- **Implement data constraints** based on business rules
- **Provide seed options** for reproducible tests
- **Cache expensive generations** when appropriate
- **Validate generated data** before use

### 🚫 **DON'T**

- **Generate data that violates** business constraints
- **Create dependencies** between generated datasets
- **Use truly random data** for critical validations
- **Generate sensitive information** in logs
- **Ignore performance implications** of complex generation
- **Forget to clean up** generated data after tests

## 📊 Performance Considerations

### Lazy Generation

```typescript
export class TestData {
  private _cachedUsers: any[] | null = null;

  get generatedUsers(): any[] {
    if (!this._cachedUsers) {
      this._cachedUsers = Array.from({ length: 100 }, () =>
        this.generateUserWithRole(this.generateUserType())
      );
    }
    return this._cachedUsers;
  }

  clearCache(): void {
    this._cachedUsers = null;
  }
}
```

### Batch Generation

```typescript
export class TestData {
  generateBatch<T>(generator: () => T, count: number): T[] {
    return Array.from({ length: count }, generator);
  }

  // Usage
  users = testData.generateBatch(
    () => testData.generateUserWithRole("user"),
    50
  );
  lists = testData.generateBatch(() => testData.generateMonitorListData(), 20);
}
```

---

**Next**: Learn about [File Management](file-management.md) for handling test files and external data sources.
