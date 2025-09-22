# POM Checklist

**Description**: Quick reference checklist for Page Object Model implementation, covering all essential aspects from design to maintenance. Use this during code reviews and when creating new page objects.

## Page Object Design ✅

### Class Structure

- [ ] **Single Responsibility**: Each page object represents one logical page/component
- [ ] **Readonly Properties**: All locators and page references are readonly
- [ ] **TypeScript Typing**: Proper types for all parameters and return values
- [ ] **Constructor Pattern**: Clean initialization with page parameter
- [ ] **Immutability**: No state stored in page objects (except cached data with invalidation)

### Naming Conventions

- [ ] **File Names**: Consistent `.page.ts` suffix (e.g., `login.page.ts`)
- [ ] **Class Names**: PascalCase with "Page" suffix (e.g., `LoginPage`)
- [ ] **Method Names**: Clear, descriptive action verbs
  - Navigation: `goto()`, `gotoUserProfile()`
  - Actions: `clickSubmitButton()`, `clickSaveButton()`
  - Form: `fillUsername()`, `fillEmail()`
  - Data: `getUserName()`, `getErrorMessage()`
  - State: `isButtonEnabled()`, `isFormVisible()`

## Locator Strategy ✅

### Priority Order (Most to Least Preferred)

- [ ] **Semantic Locators**: `getByRole()`, `getByLabel()`, `getByText()`
- [ ] **Test IDs**: `getByTestId()` for complex elements
- [ ] **Unique IDs**: `#element-id` for stable identifiers
- [ ] **CSS Selectors**: Only when other options aren't available

### Locator Quality Checks

- [ ] **Semantic First**: Prefer ARIA roles and labels
- [ ] **Stable Selectors**: Avoid nth-child, position-dependent selectors
- [ ] **Filter Patterns**: Use `.filter()` for precision when needed
- [ ] **Exact Matching**: Use `{ exact: true }` for text-based locators when ambiguous
- [ ] **No XPath**: Avoid XPath selectors for maintainability

### Organization

- [ ] **Functional Grouping**: Group locators by their purpose (navigation, forms, content, etc.)
- [ ] **Logical Order**: Arrange locators in the order they appear on the page
- [ ] **Clear Comments**: Document complex locator strategies

## Method Design ✅

### Method Categories

- [ ] **Navigation Methods**: `goto()`, `gotoSection()`
- [ ] **Action Methods**: `click*()`, `fill*()`, `select*()`
- [ ] **Utility Methods**: `get*()`, `is*()`, `wait*()`
- [ ] **Complex Workflows**: Business logic methods that combine simple actions

### Method Quality

- [ ] **Single Purpose**: Each method does one thing well
- [ ] **Return Values**: Methods return data for tests to validate
- [ ] **No Assertions**: No `expect()` statements in page objects
- [ ] **Error Handling**: Graceful handling of expected failures
- [ ] **Async/Await**: Proper async patterns throughout

### Parameters and Types

- [ ] **Typed Parameters**: Use interfaces for complex data
- [ ] **Optional Parameters**: Default values for optional parameters
- [ ] **Explicit Returns**: Clear return types (Promise<string>, Promise<boolean>, etc.)

## Architecture Patterns ✅

### Pattern Selection

- [ ] **Standard vs App Fixture**: Choose appropriate pattern for project size
- [ ] **Component Reuse**: Extract reusable UI components
- [ ] **Base Classes**: Use sparingly, prefer composition
- [ ] **Constants Management**: Centralize URLs, selectors, test data

### File Organization

- [ ] **Domain Grouping**: Organize by business functionality
- [ ] **Consistent Structure**: Follow established directory patterns
- [ ] **Clear Dependencies**: Minimize coupling between page objects

## Performance ✅

### Waiting Strategies

- [ ] **No Hardcoded Waits**: Never use `waitForTimeout()`
- [ ] **Smart Waiting**: Leverage Playwright's auto-waiting
- [ ] **Specific Conditions**: Use `waitFor()` with specific states
- [ ] **Load States**: Use appropriate `waitForLoadState()` options

### Efficiency

- [ ] **Locator Reuse**: Store and reuse locators instead of recreating
- [ ] **Lazy Initialization**: Initialize expensive operations only when needed
- [ ] **Caching Strategy**: Cache expensive computations with proper invalidation

## Code Quality ✅

### Best Practices

- [ ] **No Hardcoded Values**: Parameterize all inputs
- [ ] **Error Recovery**: Implement retry patterns for flaky operations
- [ ] **Documentation**: Comment complex business logic
- [ ] **Consistent Formatting**: Follow team code style guidelines

### Anti-Patterns Avoided

- [ ] **No Direct Navigation**: Always use page object methods instead of `page.goto()`
- [ ] **No Test Logic**: Keep test orchestration out of page objects
- [ ] **No Data Storage**: Don't store test data in page objects
- [ ] **No Deep Inheritance**: Avoid complex inheritance chains

## Testing Integration ✅

### Test Framework Integration

- [ ] **Clean Test Code**: Tests use page objects naturally
- [ ] **Data Separation**: Test data managed externally
- [ ] **Assertion Placement**: All assertions in test files, not page objects
- [ ] **Error Handling**: Tests handle and verify page object return values

### Maintainability

- [ ] **Health Monitoring**: Regular validation of locator health
- [ ] **Migration Strategy**: Clear path for updating page objects
- [ ] **Documentation**: Keep documentation up to date
- [ ] **Code Reviews**: Regular review of page object changes

## Project-Specific Checklist ✅

### Environment Configuration

- [ ] **URL Management**: Environment-specific URL handling
- [ ] **Test Data**: Environment-appropriate test data
- [ ] **Configuration**: Proper config management for different environments

### Team Standards

- [ ] **Code Review Guidelines**: Established review criteria
- [ ] **Naming Conventions**: Team-agreed naming patterns
- [ ] **Architecture Decisions**: Documented pattern choices
- [ ] **Training Materials**: Onboarding documentation for new team members

## Accessibility ✅

### Inclusive Design

- [ ] **Semantic Locators**: Preference for ARIA-compliant selectors
- [ ] **Screen Reader Compatibility**: Locators work with assistive technology
- [ ] **Keyboard Navigation**: Support for keyboard-only testing
- [ ] **Accessibility Validation**: Methods to check accessibility compliance

## Security Considerations ✅

### Data Handling

- [ ] **Sensitive Data**: No hardcoded credentials or sensitive information
- [ ] **Parameter Validation**: Input validation for security-sensitive operations
- [ ] **Log Sanitization**: Ensure sensitive data doesn't appear in logs

## Documentation ✅

### Code Documentation

- [ ] **Method Documentation**: JSDoc comments for complex methods
- [ ] **Business Logic**: Comments explaining business rules
- [ ] **Architecture Decisions**: Document why specific patterns were chosen
- [ ] **Migration Guides**: Instructions for updating existing code

### Team Documentation

- [ ] **README Files**: Clear setup and usage instructions
- [ ] **Examples**: Working examples for common patterns
- [ ] **Troubleshooting**: Common issues and solutions
- [ ] **Style Guide**: Coding standards and conventions

---

## Quick Validation Commands

```bash
# Check TypeScript compilation
npx tsc --noEmit

# Run linting
npx eslint pages/**/*.ts

# Check for unused locators
# (Custom script to analyze locator usage)

# Validate page object health
# (Custom script to check locator availability)
```

## Code Review Checklist

When reviewing page object changes:

1. **Does it follow the established patterns?**
2. **Are locators using the priority order correctly?**
3. **Do method names clearly indicate their purpose?**
4. **Are there any hardcoded values that should be parameterized?**
5. **Is error handling appropriate for the use case?**
6. **Are TypeScript types properly defined?**
7. **Does it maintain backward compatibility or provide migration path?**
8. **Is the documentation updated if needed?**

Use this checklist during development, code reviews, and maintenance to ensure consistent, high-quality Page Object Model implementation.
