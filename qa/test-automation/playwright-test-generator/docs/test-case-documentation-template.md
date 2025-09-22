# 📋 Test Case Documentation Template

## 🎯 Purpose

This template defines the comprehensive Markdown structure for parsed test cases in the `test_data/parsed_cases/` directory. This ensures consistent, detailed documentation with visual progress tracking that preserves knowledge across LLM sessions and supports team collaboration.

## 📝 Template Structure

### Required Basic Structure (Minimum)

```markdown
# 📋 TC-XXXX: Test Case Name

## 📊 Test Case Metadata

- **Test Case ID**: TC-XXXX
- **Priority**: High|Normal|Low
- **Parsed Date**: YYYY-MM-DD HH:mm:ss
- **Original XML File**: `source-file.xml`
- **Status**: 🟡 In Progress | ✅ Completed | ❌ Failed

## 🎯 Test Objective

What the test validates

## 🔧 Preconditions

Required setup before test execution

## 📋 Requirements Analysis

- **Authentication Level**: admin|user|none
- **Required Page Objects**: `pageName1`, `pageName2`
- **Test Data Dependencies**: data-type1, data-type2

## 🎭 MCP Execution Progress

### Pre-Execution Setup

- [ ] Browser environment ready
- [ ] Authentication strategy confirmed
- [ ] Initial navigation planned

### Test Steps Execution

#### Step 0: Step Description

- [ ] **MCP Command**: `mcp_command_used`
- [ ] **Expected Result**: Expected outcome description
- [ ] **Execution Status**: ⏳ Pending | ✅ Success | ❌ Failed
- **Observed Behavior**: What actually happened during execution
- **Discovered Elements**: Key selectors or elements found
- **Notes**: Any important observations

## 🔍 Execution Results

- **Overall Status**: ⏳ Pending | ✅ Completed | ❌ Failed
- **Success Rate**: X/Y steps completed
- **Critical Findings**:
  - Finding 1
  - Finding 2

## 🏗️ Generated Test Information

- **Test File**: `test-name.spec.ts`
- **Location**: `/path/to/test`
- **Generation Status**: ⏳ Pending | ✅ Completed | ❌ Failed
- **Last Updated**: YYYY-MM-DD HH:mm:ss
```

### Enhanced Structure (For Complex Cases)

When a test case involves complex interactions, form validation, or significant findings, include these additional sections:

```markdown
# 📋 TC-XXXX: Complex Test Case Name

## 📊 Test Case Metadata

[Same as basic structure]

## 🎯 Test Objective

[Same as basic structure]

## 🔧 Preconditions

[Same as basic structure]

## 📋 Requirements Analysis

[Same as basic structure]

## 🎭 MCP Execution Progress

[Same as basic structure with expanded step tracking]

## 🔍 Detailed Execution Log

### Environment Details

- **Target URL**: https://example.com/
- **Execution Date**: YYYY-MM-DD
- **Browser Mode**: headed|headless

### Page State Snapshots

#### After Step X: Page Title

- **URL**: Current page URL
- **Key Elements Discovered**:
  - `elementName`: selector strategy used
  - `validationElement`: assertion approach
- **Important Observations**: Critical findings about page behavior

### Technical Discoveries

#### Element Stability & Interactions

- **Page Load Behavior**: Loading patterns observed
- **Element Readiness**: Interaction timing requirements
- **Navigation Patterns**: How page transitions work
- **Error Scenarios**: Any error conditions encountered

#### Discovered Selectors

- **Primary Elements**:
  - Form fields: `#form_name`, `getByLabel('Email')`
  - Buttons: `getByRole('button', { name: 'Submit' })`
  - Navigation: `getByText('Next Page')`

#### Validation Strategies

- **Success Indicators**:
  - URL contains: `/confirmation`
  - Success message: `expect(page.getByText('Success')).toBeVisible()`
  - Element states: Form fields cleared, buttons enabled

## 💼 Business Logic Insights

- **Key Business Rule 1**: Description of discovered rule
- **Key Business Rule 2**: Another important finding
- **Validation Requirements**: Complex validation scenarios found

## 🔮 Future Test Considerations

- **Test Expansion Opportunities**:
  - [ ] Error scenario testing
  - [ ] Edge case validation
  - [ ] Performance testing
- **Data Cleanup Requirements**:
  - [ ] Remove test records after execution
  - [ ] Reset application state
- **Page Object Improvements Needed**:
  - [ ] Add missing navigation methods
  - [ ] Update element locators
  - [ ] Create new helper methods

## 📈 Execution Summary

- **Total MCP Commands**: X commands executed
- **Success Rate**: X% (Y/Z successful)
- **Critical Findings**:
  - ✅ Finding 1: Successful discovery
  - ⚠️ Finding 2: Important consideration
  - ❌ Finding 3: Issue requiring attention
- **Lessons Learned**:
  - Lesson 1: Key insight about application behavior
  - Lesson 2: Important technical consideration
- **Recommended Improvements**:
  - Improvement 1: Suggestion for better testing
  - Improvement 2: Process enhancement idea

## 🏗️ Generated Test Information

[Same as basic structure with additional details]

### Page Object Updates Required

- [ ] **File**: `/pages/forms/formPage.ts`
  - [ ] Added validation message locators
  - [ ] Updated submit method with proper waits
- [ ] **File**: `/pages/navigation/navPage.ts`
  - [ ] Created new navigation method
  - [ ] Added proper wait strategies
```

## 🎯 When to Use Enhanced Structure

### Use Enhanced Documentation When:

- **Complex Forms**: Multi-step forms with many fields and validation rules
- **Critical Business Flows**: Core application functionality that many tests will depend on
- **New Feature Areas**: First test in a new section of the application
- **Page Object Updates Needed**: When MCP execution reveals page object issues
- **Unusual Application Behavior**: Non-standard navigation or interaction patterns
- **Rich Validation Requirements**: Multiple success criteria or complex assertions

### Use Basic Structure When:

- **Simple CRUD Operations**: Standard create, read, update, delete flows
- **Straightforward Navigation**: Basic page-to-page movement
- **Repetitive Test Patterns**: Similar to existing documented test cases
- **Quick Validation Tests**: Simple checks without complex setup

## 📋 Documentation Guidelines

### Essential Information (Always Include)

1. **MCP Execution Results**: What actually happened during live testing
2. **Key Findings**: Selectors, validation points, and critical observations
3. **Page Object Requirements**: What methods exist vs what's needed
4. **Success Criteria**: How to validate the test passed

### Detailed Information (Include When Valuable)

1. **Page States**: Full snapshots at key steps for complex flows
2. **Form Default Values**: Important for comprehensive test coverage
3. **Business Logic Insights**: Rules discovered that affect test design
4. **Technical Details**: Performance, errors, console messages
5. **Future Considerations**: Test expansion opportunities and data cleanup needs

## 🔧 Maintenance Guidelines

### File Naming Convention

```
{testCaseId}-{sanitized-name}.md
```

Example: `TC-306-create-user-account.md`

### Update Triggers

- When test fails and requires debugging
- When page objects are updated based on findings
- When business logic changes are discovered
- When new test variations are identified

### Version Control

- Commit parsed cases with their generated tests
- Include meaningful commit messages about findings
- Tag major application behavior discoveries

## 🎨 Usage Examples

### Basic Documentation Example

```markdown
# 📋 TC-123: Login with Valid Credentials

## 📊 Test Case Metadata

- **Test Case ID**: TC-123
- **Priority**: High
- **Parsed Date**: 2025-08-01 10:30:00
- **Original XML File**: `login-test.xml`
- **Status**: ✅ Completed

## 🎯 Test Objective

Verify user can log in with correct username and password

## 🔧 Preconditions

Valid user account exists in the system

## 📋 Requirements Analysis

- **Authentication Level**: user
- **Required Page Objects**: `loginPage`, `dashboardPage`
- **Test Data Dependencies**: valid-credentials

## 🎭 MCP Execution Progress

### Pre-Execution Setup

- [x] Browser environment ready
- [x] Authentication strategy confirmed
- [x] Initial navigation planned

### Test Steps Execution

#### Step 0: Navigate to login page

- [x] **MCP Command**: `mcp_playwright_browser_navigate`
- [x] **Expected Result**: Login form should be displayed
- [x] **Execution Status**: ✅ Success
- **Observed Behavior**: Login form displayed correctly with username and password fields
- **Discovered Elements**:
  - Username field: `#username`
  - Password field: `#password`
  - Login button: `getByRole('button', { name: 'Login' })`

## 🔍 Execution Results

- **Overall Status**: ✅ Completed
- **Success Rate**: 3/3 steps completed
- **Critical Findings**:
  - Standard authentication flow works as expected
  - No additional validation required

## 🏗️ Generated Test Information

- **Test File**: `login-valid-credentials.spec.ts`
- **Location**: `/tests/auth/login-valid-credentials.spec.ts`
- **Generation Status**: ✅ Completed
- **Last Updated**: 2025-08-01 11:00:00
```

### Enhanced Documentation Example

For complex cases requiring detailed documentation:

```markdown
# 📋 TC-999: Complex Form Submission with Validation

## 📊 Test Case Metadata

- **Test Case ID**: TC-999
- **Priority**: High
- **Parsed Date**: 2025-08-01 10:30:00
- **Original XML File**: `complex-form-test.xml`
- **Status**: ✅ Completed

## 🎯 Test Objective

Verify multi-step form handles validation and submission correctly

## 🔧 Preconditions

User with appropriate permissions logged in

## 📋 Requirements Analysis

- **Authentication Level**: user
- **Required Page Objects**: `homePage`, `formPage`, `confirmationPage`
- **Test Data Dependencies**: form-data.json, validation-rules

## 🎭 MCP Execution Progress

### Pre-Execution Setup

- [x] Browser environment ready
- [x] Authentication strategy confirmed
- [x] Initial navigation planned

### Test Steps Execution

#### Step 0: Navigate to form page

- [x] **MCP Command**: `mcp_playwright_browser_navigate`
- [x] **Expected Result**: Form should be displayed with multiple sections
- [x] **Execution Status**: ✅ Success
- **Observed Behavior**: Form displayed with multiple sections and validation
- **Discovered Elements**:
  - Main form: `#main-form`
  - Required fields with asterisks
  - Real-time validation messages

#### Step 1: Fill required fields

- [x] **MCP Command**: `mcp_playwright_browser_type`
- [x] **Expected Result**: Fields should accept input with validation
- [x] **Execution Status**: ✅ Success
- **Observed Behavior**: Real-time validation triggered on field blur
- **Discovered Elements**:
  - Name field: `#form_name`
  - Email field: `getByLabel('Email Address')`
  - Validation messages: `.error-message`

#### Step 2: Submit form

- [x] **MCP Command**: `mcp_playwright_browser_click`
- [x] **Expected Result**: Form should submit and redirect to confirmation
- [x] **Execution Status**: ✅ Success
- **Observed Behavior**: Successful submission with redirect to confirmation page
- **Discovered Elements**:
  - Submit button: `getByRole('button', { name: 'Submit Form' })`
  - Success banner: `.success-notification`

## 🔍 Detailed Execution Log

### Environment Details

- **Target URL**: https://example.com/forms/complex
- **Execution Date**: 2025-08-01
- **Browser Mode**: headed

### Page State Snapshots

#### After Step 0: Complex Form Page

- **URL**: https://example.com/forms/complex
- **Key Elements Discovered**:
  - `#main-form`: Primary form container
  - `.form-section`: Multiple form sections
  - `.required-field`: Fields marked as required
- **Important Observations**: Form has client-side validation with real-time feedback

#### After Step 2: Confirmation Page

- **URL**: https://example.com/forms/confirmation
- **Key Elements Discovered**:
  - `.success-notification`: Confirmation message
  - `#confirmation-id`: Unique submission identifier
- **Important Observations**: Successful submission generates unique ID

### Technical Discoveries

#### Element Stability & Interactions

- **Page Load Behavior**: Form loads with progressive enhancement
- **Element Readiness**: Fields become interactive after page load state
- **Navigation Patterns**: Form submission uses POST with redirect
- **Error Scenarios**: Client-side validation prevents invalid submissions

#### Discovered Selectors

- **Primary Elements**:
  - Form fields: `#form_name`, `getByLabel('Email Address')`
  - Buttons: `getByRole('button', { name: 'Submit Form' })`
  - Validation: `.error-message`, `.success-notification`

#### Validation Strategies

- **Success Indicators**:
  - URL contains: `/confirmation`
  - Success message: `expect(page.getByText('Form submitted successfully')).toBeVisible()`
  - Element states: Form cleared, confirmation ID displayed

## 💼 Business Logic Insights

- **Validation Rules**: Real-time validation on field blur events
- **Submission Process**: Two-step validation (client + server side)
- **Success Confirmation**: Unique submission ID generated for tracking

## 🔮 Future Test Considerations

- **Test Expansion Opportunities**:
  - [x] Happy path validation completed
  - [ ] Error scenario testing (invalid data)
  - [ ] Edge case validation (boundary values)
  - [ ] Performance testing (large datasets)
- **Data Cleanup Requirements**:
  - [ ] Remove test submissions after execution
  - [ ] Reset form state for next test
- **Page Object Improvements Needed**:
  - [x] Added form interaction methods
  - [ ] Create validation helper methods
  - [ ] Add error state checking methods

## 📈 Execution Summary

- **Total MCP Commands**: 5 commands executed
- **Success Rate**: 100% (5/5 successful)
- **Critical Findings**:
  - ✅ Real-time validation works correctly
  - ✅ Form submission process reliable
  - ⚠️ Requires cleanup of test submissions
- **Lessons Learned**:
  - Form validation timing is crucial for reliable tests
  - Success confirmation requires checking both URL and message
- **Recommended Improvements**:
  - Add helper methods for form validation checking
  - Implement cleanup process for test data

## 🏗️ Generated Test Information

- **Test File**: `complex-form-submission.spec.ts`
- **Location**: `/tests/forms/complex-form-submission.spec.ts`
- **Generation Status**: ✅ Completed
- **Last Updated**: 2025-08-01 11:00:00

### Page Object Updates Required

- [x] **File**: `/pages/forms/formPage.ts`
  - [x] Added validation message locators
  - [x] Updated submit method with proper waits
  - [x] Created form field interaction methods
- [ ] **File**: `/pages/forms/confirmationPage.ts`
  - [ ] Created new page object for confirmation
  - [ ] Added confirmation ID extraction method
```

## 🚀 Benefits

1. **Visual Progress Tracking**: Checkboxes provide clear visual indication of completion status
2. **Context Preservation**: Maintains knowledge when LLM sessions exceed limits
3. **Team Collaboration**: Easy-to-read format for team knowledge sharing
4. **Test Maintenance**: Quick debugging when tests fail with step-by-step breakdown
5. **Coverage Planning**: Clear identification of test expansion opportunities
6. **Page Object Evolution**: Trackable improvements to page object implementations
7. **Real-time Updates**: Easy to update progress as work proceeds
8. **Version Control Friendly**: Markdown diffs are readable in pull requests
