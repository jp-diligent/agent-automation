# Assertion Checklist

**Description**: Quick validation checklist for writing effective Playwright assertions. Use this during development and code reviews to ensure assertion quality and consistency.

## Pre-Assertion Checklist ✅

### Element Selection

- [ ] **Semantic Locators**: Using `getByRole()`, `getByLabel()`, `getByText()` when possible?
- [ ] **Test IDs**: Using `getByTestId()` for complex elements that lack semantic meaning?
- [ ] **Stable Selectors**: Avoiding position-dependent selectors (`nth-child`, etc.)?
- [ ] **Context-Specific**: Using container-based selectors to limit scope?
- [ ] **No XPath**: Avoiding XPath selectors for better maintainability?

### Assertion Type Selection

- [ ] **Appropriate Method**: Using the most specific assertion method available?
- [ ] **Visibility vs Existence**: Using `toBeVisible()` vs `toHaveCount()` appropriately?
- [ ] **Text Matching**: Using `toHaveText()` for exact match vs `toContainText()` for partial?
- [ ] **State Validation**: Using `toBeEnabled()`, `toBeChecked()`, etc. for interactive elements?

### Timing and Performance

- [ ] **Auto-Waiting**: Leveraging Playwright's built-in waiting instead of manual delays?
- [ ] **Appropriate Timeouts**: Using custom timeouts only when justified?
- [ ] **Locator Reuse**: Storing locators in variables when used multiple times?
- [ ] **Efficient Ordering**: Running quick assertions before expensive ones?

## Assertion Quality Checklist ✅

### Clarity and Intent

- [ ] **Clear Purpose**: Does the assertion clearly communicate what's being validated?
- [ ] **Descriptive Messages**: Are error messages helpful for debugging?
- [ ] **Business Context**: Do assertions validate user-facing behavior, not implementation?
- [ ] **Comprehensive Coverage**: Are both positive and negative scenarios covered?

### Reliability and Robustness

- [ ] **No Race Conditions**: Are assertions free from timing-related issues?
- [ ] **Stable Elements**: Are targeted elements unlikely to change frequently?
- [ ] **Error Handling**: Are edge cases and error states properly validated?
- [ ] **Environment Independence**: Will assertions work across different environments?

### Structure and Organization

- [ ] **Logical Grouping**: Are related assertions grouped together using test steps?
- [ ] **Single Responsibility**: Does each assertion validate one specific thing?
- [ ] **Proper Hierarchy**: Are assertions ordered from general to specific?
- [ ] **No Implementation Details**: Are assertions focused on user experience?

## Code Review Checklist ✅

### When Reviewing Assertion Code

#### Locator Quality

- [ ] **Semantic Priority**: Are semantic locators used where possible?
- [ ] **Selector Stability**: Will selectors survive UI changes?
- [ ] **Performance Impact**: Are locators created efficiently?
- [ ] **Maintenance Burden**: Are selectors easy to understand and update?

#### Assertion Logic

- [ ] **Correct Methods**: Are assertion methods appropriate for what's being tested?
- [ ] **Complete Validation**: Are all necessary aspects of the feature validated?
- [ ] **Timeout Usage**: Are custom timeouts justified and appropriate?
- [ ] **Error Messages**: Will failures provide actionable debugging information?

#### Test Structure

- [ ] **Clear Steps**: Are complex workflows broken into clear test steps?
- [ ] **Focused Tests**: Does each test have a single, clear purpose?
- [ ] **Reusable Patterns**: Are common assertion patterns extracted into helpers?
- [ ] **Documentation**: Are complex business rules explained in comments?

## Common Anti-Pattern Checks ❌

### What to Look For and Fix

#### Timing Issues

- [ ] **No Manual Waits**: Are there `waitForTimeout()` calls before assertions?
- [ ] **No Polling Loops**: Are there manual retry loops instead of built-in waiting?
- [ ] **Appropriate Timeouts**: Are timeout values reasonable and documented?

#### Locator Problems

- [ ] **No XPath**: Are XPath selectors being used unnecessarily?
- [ ] **No Brittle Selectors**: Are position-dependent or implementation-specific selectors used?
- [ ] **No Generic Selectors**: Are overly broad selectors like `div` or `button` used?

#### Assertion Structure

- [ ] **No Assertions in Page Objects**: Are assertions kept in test files?
- [ ] **No Vague Messages**: Are assertion error messages specific and helpful?
- [ ] **No Implementation Testing**: Are internal details being tested instead of user behavior?

## Performance Optimization Checklist ⚡

### Efficiency Considerations

- [ ] **Locator Reuse**: Are commonly used locators stored in variables?
- [ ] **Parallel Safety**: Can independent assertions run in parallel?
- [ ] **Resource Management**: Are expensive operations (screenshots, etc.) used judiciously?
- [ ] **Batch Operations**: Are related assertions grouped for efficiency?

### Scalability

- [ ] **Large Dataset Handling**: Are large lists/tables validated efficiently?
- [ ] **Memory Usage**: Are locators and resources cleaned up appropriately?
- [ ] **Timeout Scaling**: Do timeouts scale appropriately with data size?

## Test Organization Checklist 📋

### File and Test Structure

- [ ] **Clear Test Names**: Do test names clearly describe what's being validated?
- [ ] **Logical Grouping**: Are related tests grouped in describe blocks?
- [ ] **Step Organization**: Are complex workflows broken into logical steps?
- [ ] **Setup and Teardown**: Are test preconditions and cleanup handled properly?

### Data Management

- [ ] **Parameterized Data**: Are hardcoded values extracted into variables/constants?
- [ ] **Test Data Separation**: Is test data managed separately from test logic?
- [ ] **Environment Handling**: Are environment-specific values handled appropriately?

## Debugging and Maintenance Checklist 🔧

### Debugging Support

- [ ] **Error Context**: Will assertion failures provide enough context for debugging?
- [ ] **Screenshot Strategy**: Are screenshots captured for complex failures?
- [ ] **Logging**: Is appropriate logging included for complex workflows?
- [ ] **State Capture**: Is relevant application state captured on failures?

### Maintainability

- [ ] **Documentation**: Are complex business rules documented?
- [ ] **Update Strategy**: Will changes to the UI require minimal test updates?
- [ ] **Pattern Consistency**: Are assertion patterns consistent across the test suite?
- [ ] **Refactoring Safety**: Can common patterns be safely extracted and updated?

## Quick Validation Commands

```bash
# Type checking
npx tsc --noEmit

# Linting
npx eslint tests/**/*.ts

# Run specific test file
npx playwright test login.spec.ts

# Run tests with debug info
npx playwright test --debug

# Generate test report
npx playwright test --reporter=html
```

## Checklist Usage Guide

### During Development

1. **Pre-Writing**: Review element selection and assertion type checklists
2. **While Writing**: Check assertion quality and structure items
3. **Post-Writing**: Validate with anti-pattern and performance checklists

### During Code Review

1. **First Pass**: Focus on locator quality and assertion logic
2. **Second Pass**: Check test structure and organization
3. **Final Pass**: Verify debugging support and maintainability

### During Maintenance

1. **Regular Audits**: Review performance and scalability items
2. **After UI Changes**: Validate locator stability
3. **Before Releases**: Ensure comprehensive coverage and error handling

## Team Standards Template

Customize this checklist for your team by adding project-specific items:

```markdown
## Project-Specific Checklist ✅

### [Your Project] Standards

- [ ] **Custom Item 1**: [Description]
- [ ] **Custom Item 2**: [Description]
- [ ] **Custom Item 3**: [Description]

### [Your Domain] Validation

- [ ] **Business Rule 1**: [Validation requirement]
- [ ] **Business Rule 2**: [Validation requirement]
- [ ] **Business Rule 3**: [Validation requirement]
```

Use this checklist consistently to maintain high-quality assertions across your Playwright test suite.
