# 🎭 Zephyr XML to Playwright Test Generation Instructions

## 🎯 Overview

These instructions define the process for parsing Zephyr XML test case exports and enriching them for Playwright test generation using MCP tools for live execution.

## 📋 Zephyr XML Structure Analysis

### Key XML Elements to Parse

```xml
<testCase id="..." key="...">
    <name><![CDATA[Test Case Name]]></name>
    <objective><![CDATA[Test objective description]]></objective>
    <precondition><![CDATA[Prerequisites for the test]]></precondition>
    <testScript type="steps">
        <steps>
            <step index="0">
                <description><![CDATA[Step description with HTML formatting]]></description>
                <expectedResult><![CDATA[Expected outcome with HTML formatting]]></expectedResult>
                <testData><![CDATA[Test data or element identifier]]></testData>
            </step>
        </steps>
    </testScript>
</testCase>
```

## 🎭 Step 1: XML Parsing and Enrichment Process

### 1.1 Extract Test Case Metadata

- **Test Name**: Extract from `<name>` element (strip CDATA)
- **Objective**: Extract from `<objective>` element (strip CDATA and HTML)
- **Preconditions**: Extract from `<precondition>` element (strip CDATA and HTML)
- **Test ID**: Extract from `key` attribute for reference

### 1.2 Parse Test Steps

For each `<step>` element:

1. **Index**: Step order from `index` attribute
2. **Description**: Strip HTML tags and CDATA from `<description>`
3. **Expected Result**: Strip HTML tags and CDATA from `<expectedResult>`
4. **Test Data**: Extract from `<testData>` - this often contains:
   - URLs for navigation
   - Button/link text for clicking
   - Field names for form interactions
   - Checkbox/option labels
   - Expected text for validation

### 1.3 Enrich Steps for MCP Execution

Transform each parsed step into actionable MCP commands:

#### Step Type Detection Patterns:

```typescript
// Navigation steps
if (/https?:\/\/\S+/i.test(testData) || description.includes("Open")) {
  action = "mcp_playwright_browser_navigate";
  parameter = extractUrl(testData);
}

// Click actions
if (description.includes("Click") || description.includes("click")) {
  action = "mcp_playwright_browser_click";
  elementText = testData; // Button/link text
}

// Text input
if (description.includes("Enter") || description.includes("Fill")) {
  action = "mcp_playwright_browser_type";
  text = testData; // Text to enter
}

// Checkbox/Selection
if (description.includes("Check") || description.includes("Select")) {
  action = "mcp_playwright_browser_click";
  elementText = testData; // Checkbox label
}
```

## 🎭 Step 2: MCP Execution Workflow

### 2.1 Pre-Execution Setup

1. **Start with navigation**: Always begin with `mcp_playwright_browser_navigate` to the application URL
2. **Authentication handling**:
   - For MCP: Wait for manual login (cannot use stored auth)
   - For final test: Use Playwright's global authentication
3. **Initial snapshot**: Take `mcp_playwright_browser_snapshot` after navigation

### 2.2 Step-by-Step MCP Execution

For each enriched step:

```typescript
// Pattern for each step execution:
1. Take snapshot: `mcp_playwright_browser_snapshot()`
2. Identify elements from snapshot
3. Execute action with correct element reference
4. **IMMEDIATELY UPDATE MARKDOWN FILE**:
   - Mark step as completed: `- [x]`
   - Update execution status: `✅ Success` or `❌ Failed`
   - Record observed behavior: What actually happened
   - **CAPTURE DISCOVERED ELEMENTS**: Exact selectors, refs, and locator strategies
   - Add implementation notes: Page object methods identified
5. Validate expected result through subsequent snapshot
6. **NEVER proceed to next step without updating current step details**
```

### 🚨 CRITICAL: Real-Time Markdown Updates

**After each MCP command execution, you MUST:**

1. **Update step checkbox**: `- [ ]` → `- [x]`
2. **Update execution status**: `⏳ Pending` → `✅ Success` or `❌ Failed`
3. **Record observed behavior**: Describe what actually happened
4. **Capture discovered elements**:
   - Element refs from MCP output (e.g., `ref=e28`)
   - Selectors used (e.g., `getByRole('link', { name: 'Monitor' })`)
   - Element types and properties
5. **Add implementation notes**: Required page object methods, navigation patterns
6. **Update notes**: Any important observations or deviations

**Why This Is Critical:**

- **Context Window Management**: If LLM context is exceeded, the markdown file becomes the primary source of truth
- **Element Reference Preservation**: All discovered selectors are preserved for code generation
- **Progress Continuity**: Another LLM session can pick up exactly where the previous one left off
- **Quality Assurance**: Real-time updates ensure no details are lost during execution

### 2.3 Element Detection Strategy

From each snapshot, detect elements using:

- **Button text**: Match against step's `testData`
- **Link text**: Look for navigation elements
- **Form fields**: Identify input fields by labels
- **Checkboxes**: Find by associated text/labels

## 🎭 Step 3: Test Code Generation Rules

### 3.1 Authentication Requirements Analysis

```typescript
// Determine auth level from test steps:
const requiresAdminAuth = steps.some(
  (step) =>
    step.description.includes("admin") ||
    step.testData.includes("admin") ||
    step.expectedResult.includes("admin")
);

// Only add admin auth if explicitly required:
if (requiresAdminAuth) {
  testCode += "test.use({ storageState: 'tests/.auth/admin.json' });\n";
}
```

### 3.2 Page Object Method Mapping

Based on MCP execution results, map actions to existing page object methods:

```typescript
// Check existing page objects for methods:
const pageObjectMethods = {
  "Navigate to Monitor": "app.dashboardPage.navigateToMonitor()",
  "Click Manage Lists": "app.monitorPage.clickManageListsTab()",
  "Add new list": "app.monitorPage.clickAddNewList()",
  "Fill list name": "app.addMonitorListPage.fillListName(text)",
  "Check Negative News": "app.addMonitorListPage.selectNegativeNews()",
  "Create list": "app.addMonitorListPage.clickCreateList()",
};
```

### 3.3 Test Structure Generation

```typescript
// Generated test structure:
import { expect, test } from "fixtures/fixtures";

// Only if admin required from analysis
// test.use({ storageState: 'tests/.auth/admin.json' });

test.describe("${testCaseName}", () => {
  test("should ${cleanObjective}", async ({ app }) => {
    // Step 1: Navigation (use page object method)
    await test.step("Navigate to application", async () => {
      await app.dashboardPage.goto(); // Never hardcode URLs
    });

    // No manual login - global auth handles this

    // Generate steps based on MCP execution results
    await test.step("${stepDescription}", async () => {
      // Use page object methods discovered from MCP execution
      // Include appropriate assertions based on expectedResult
    });
  });
});
```

## 📋 Implementation Checklist

### ⌚️ Style Guide Compliance

- [ ] Every action wrapped in `test.step()` with descriptive names
- [ ] No plain comments (`// Step 1:`) - use `test.step()` instead
- [ ] Follow established test structure patterns

### 💉 Page Object Model Compliance

- [ ] **NO direct `app.page.locator()` calls in test steps**
- [ ] **NO direct `app.page.getByRole()` calls in test steps**
- [ ] **ALL actions through page object methods**: `app.pageName.methodName()`
- [ ] Check existing page object files for available methods
- [ ] Create new page object methods if none exist (never bypass POM)
- [ ] Use existing navigation methods instead of hardcoded URLs

### 🎯 Assertion Compliance

- [ ] Use semantic locators from MCP execution results
- [ ] Group related assertions logically
- [ ] Include descriptive error messages
- [ ] Avoid manual waits before assertions

### 🔐 Authentication Analysis

- [ ] Only use admin auth when test explicitly requires admin features
- [ ] Default to regular user authentication for standard flows
- [ ] No manual login steps in final test code

### 🚫 Anti-Pattern Avoidance

- [ ] No hardcoded timeouts
- [ ] No try-catch blocks for flow control
- [ ] No conditional logic in test steps
- [ ] No console.log for validation
- [ ] No error recovery logic

## 🔄 Workflow Summary

1. **Parse XML**: Extract test metadata and steps
2. **Enrich Steps**: Convert to MCP-actionable commands
3. **Live Execute**: Run each step via MCP tools
4. **Capture Selectors**: Record real element references from execution
5. **Map to Page Objects**: Use existing methods or identify needed ones
6. **Generate Test**: Create compliant Playwright test based on execution history
7. **Validate**: Run generated test and iterate until passing

## 📝 Example Parsing Output

For the "Create monitor list" XML, generate a Markdown file:

```markdown
# 📋 TC-456: Create Monitor List

## 📊 Test Case Metadata

- **Test Case ID**: TC-456
- **Priority**: High
- **Parsed Date**: 2025-08-19 10:30:00
- **Original XML File**: `create-monitor-list.xml`
- **Status**: 🟡 In Progress

## 🎯 Test Objective

Create a monitor list on the UI and validate that the list is created

## 🔧 Preconditions

Already existing manager user with all the benefits

## 📋 Requirements Analysis

- **Authentication Level**: user (no admin-specific actions detected)
- **Required Page Objects**: `dashboardPage`, `monitorPage`, `addMonitorListPage`
- **Test Data Dependencies**: monitor-list-data

## 🎭 MCP Execution Progress

### Pre-Execution Setup

- [ ] Browser environment ready
- [ ] Authentication strategy confirmed
- [ ] Initial navigation planned

### Test Steps Execution

#### Step 0: Open the given page

- [ ] **MCP Command**: `mcp_playwright_browser_navigate`
- [ ] **Expected Result**: The page should be opened
- [ ] **Execution Status**: ⏳ Pending
- **Test Data**: https://search.transparint.com/
- **Observed Behavior**: [UPDATE AFTER MCP EXECUTION]
- **Discovered Elements**: [UPDATE WITH EXACT SELECTORS FROM MCP OUTPUT]
- **Page Object Method**: `app.dashboardPage.goto()`
- **Notes**: [UPDATE WITH IMPLEMENTATION DETAILS]

#### Step 3: Click on the given link

- [ ] **MCP Command**: `mcp_playwright_browser_click`
- [ ] **Expected Result**: The user is redirected to the monitor page
- [ ] **Execution Status**: ⏳ Pending
- **Test Data**: Monitor
- **Observed Behavior**: [UPDATE AFTER MCP EXECUTION]
- **Discovered Elements**: [UPDATE WITH EXACT SELECTORS FROM MCP OUTPUT]
- **Page Object Method**: `app.dashboardPage.navigateToMonitor()`
- **Notes**: [UPDATE WITH IMPLEMENTATION DETAILS]

## 🏗️ Generated Test Information

- **Test File**: `create-monitor-list.spec.ts`
- **Location**: `/tests/monitor/create-monitor-list.spec.ts`
- **Generation Status**: ⏳ Pending
```

This enriched Markdown structure provides everything needed for both MCP execution and final test generation while ensuring compliance with all established patterns and guidelines. The checkbox format allows for easy progress tracking and visual status updates.
