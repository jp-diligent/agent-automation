# Assertion Patterns & Strategies

**Description**: Advanced patterns and strategies for implementing effective assertions in Playwright tests. This guide covers sophisticated assertion techniques, workflow validation patterns, and strategies for handling complex UI scenarios.

## Table of Contents

- [Workflow Validation Patterns](#workflow-validation-patterns)
- [State Transition Assertions](#state-transition-assertions)
- [Data Validation Strategies](#data-validation-strategies)
- [Error Handling Patterns](#error-handling-patterns)
- [Progressive Validation](#progressive-validation)
- [Conditional Assertion Patterns](#conditional-assertion-patterns)
- [Performance-Aware Assertions](#performance-aware-assertions)
- [Complex UI Validation](#complex-ui-validation)

## Workflow Validation Patterns

### Complete User Journey Validation

Validate entire user workflows with comprehensive state checking:

```typescript
test('complete order workflow', async ({ page }) => {
  const productPage = new ProductPage(page);
  const cartPage = new CartPage(page);
  const checkoutPage = new CheckoutPage(page);
  const confirmationPage = new ConfirmationPage(page);

  await test.step('Product selection and cart addition', async () => {
    await productPage.goto('widget-pro');

    // Validate product page state
    await expect(page).toHaveURL(/\/products\/widget-pro/);
    await expect(productPage.productTitle).toHaveText('Widget Pro');
    await expect(productPage.price).toHaveText('$99.99');
    await expect(productPage.addToCartButton).toBeEnabled();

    await productPage.addToCart(2);

    // Validate immediate feedback
    await expect(page.getByText('Added to cart')).toBeVisible();
    await expect(productPage.cartCounter).toHaveText('2');
  });

  await test.step('Cart review and validation', async () => {
    await cartPage.goto();

    // Validate cart state
    await expect(page).toHaveURL('/cart');
    await expect(cartPage.itemCount).toHaveText('2 items');
    await expect(cartPage.itemName).toHaveText('Widget Pro');
    await expect(cartPage.itemQuantity).toHaveValue('2');
    await expect(cartPage.itemPrice).toHaveText('$99.99');
    await expect(cartPage.subtotal).toHaveText('$199.98');
    await expect(cartPage.checkoutButton).toBeEnabled();
  });

  await test.step('Checkout process', async () => {
    await cartPage.proceedToCheckout();

    // Validate checkout initialization
    await expect(page).toHaveURL('/checkout');
    await expect(checkoutPage.orderSummary).toBeVisible();
    await expect(checkoutPage.totalAmount).toHaveText('$199.98');

    // Fill and validate checkout form
    await checkoutPage.fillShippingInfo({
      firstName: 'John',
      lastName: 'Doe',
      email: 'john@example.com',
      address: '123 Main St',
      city: 'Boston',
      zipCode: '02101',
    });

    // Validate form completion enables submission
    await expect(checkoutPage.submitButton).toBeEnabled();
    await expect(checkoutPage.shippingCost).toHaveText('$9.99');
    await expect(checkoutPage.finalTotal).toHaveText('$209.97');
  });

  await test.step('Order completion and confirmation', async () => {
    await checkoutPage.submitOrder();

    // Validate successful completion
    await expect(page).toHaveURL(/\/confirmation\/[A-Z0-9]+/);
    await expect(confirmationPage.successMessage).toBeVisible();
    await expect(confirmationPage.orderNumber).toBeVisible();
    await expect(confirmationPage.orderTotal).toHaveText('$209.97');
    await expect(confirmationPage.shippingInfo).toContainText('123 Main St');
    await expect(confirmationPage.estimatedDelivery).toBeVisible();

    // Validate order number format
    const orderNumber = await confirmationPage.orderNumber.textContent();
    expect(orderNumber).toMatch(/^ORD-\d{8}$/);
  });
});
```

### Multi-Step Form Validation

Progressive validation of complex forms:

```typescript
test('user registration form workflow', async ({ page }) => {
  const registrationPage = new RegistrationPage(page);

  await test.step('Form initialization', async () => {
    await registrationPage.goto();

    // Validate initial form state
    await expect(registrationPage.form).toBeVisible();
    await expect(registrationPage.nextButton).toBeDisabled();
    await expect(registrationPage.progressIndicator).toHaveText('Step 1 of 3');
  });

  await test.step('Personal information step', async () => {
    await registrationPage.fillPersonalInfo({
      firstName: 'John',
      lastName: 'Doe',
      dateOfBirth: '1990-01-01',
    });

    // Validate step completion
    await expect(registrationPage.nextButton).toBeEnabled();
    await expect(registrationPage.firstNameField).toHaveValue('John');
    await expect(registrationPage.lastNameField).toHaveValue('Doe');

    await registrationPage.clickNext();

    // Validate progression
    await expect(registrationPage.progressIndicator).toHaveText('Step 2 of 3');
    await expect(registrationPage.personalInfoSection).toHaveClass(/completed/);
  });

  await test.step('Account information step', async () => {
    await registrationPage.fillAccountInfo({
      username: 'johndoe2024',
      email: 'john.doe@example.com',
      password: 'SecurePass123!',
    });

    // Validate real-time feedback
    await expect(registrationPage.usernameAvailability).toHaveText('✓ Available');
    await expect(registrationPage.emailValidation).toHaveText('✓ Valid email');
    await expect(registrationPage.passwordStrength).toHaveText('Strong');
    await expect(registrationPage.nextButton).toBeEnabled();

    await registrationPage.clickNext();

    // Validate progression
    await expect(registrationPage.progressIndicator).toHaveText('Step 3 of 3');
    await expect(registrationPage.accountInfoSection).toHaveClass(/completed/);
  });

  await test.step('Preferences and completion', async () => {
    await registrationPage.selectPreferences({
      newsletter: true,
      notifications: false,
      theme: 'dark',
    });

    // Validate final step
    await expect(registrationPage.submitButton).toBeEnabled();
    await expect(registrationPage.termsCheckbox).not.toBeChecked();

    await registrationPage.acceptTerms();

    // Validate terms acceptance
    await expect(registrationPage.termsCheckbox).toBeChecked();
    await expect(registrationPage.submitButton).toBeEnabled();

    await registrationPage.submit();

    // Validate successful registration
    await expect(page).toHaveURL('/welcome');
    await expect(page.getByText('Registration successful')).toBeVisible();
    await expect(page.getByText('Welcome, John Doe!')).toBeVisible();
  });
});
```

## State Transition Assertions

### UI State Machine Validation

Validate complex UI state transitions:

```typescript
test('document editor state transitions', async ({ page }) => {
  const editorPage = new DocumentEditorPage(page);

  await test.step('Initialize editor in read-only state', async () => {
    await editorPage.goto('document-123');

    // Validate initial read-only state
    await expect(editorPage.content).toBeVisible();
    await expect(editorPage.content).not.toBeEditable();
    await expect(editorPage.editButton).toBeVisible();
    await expect(editorPage.editButton).toBeEnabled();
    await expect(editorPage.saveButton).not.toBeVisible();
    await expect(editorPage.statusBadge).toHaveText('Read-only');
  });

  await test.step('Transition to edit mode', async () => {
    await editorPage.enterEditMode();

    // Validate edit mode state
    await expect(editorPage.content).toBeEditable();
    await expect(editorPage.editButton).not.toBeVisible();
    await expect(editorPage.saveButton).toBeVisible();
    await expect(editorPage.saveButton).toBeDisabled(); // No changes yet
    await expect(editorPage.cancelButton).toBeVisible();
    await expect(editorPage.statusBadge).toHaveText('Editing');
    await expect(editorPage.autoSaveIndicator).toBeVisible();
  });

  await test.step('Make changes and validate dirty state', async () => {
    await editorPage.typeContent('Updated content here');

    // Validate dirty state
    await expect(editorPage.saveButton).toBeEnabled();
    await expect(editorPage.statusBadge).toHaveText('Unsaved changes');
    await expect(editorPage.autoSaveIndicator).toHaveText('Saving...');

    // Wait for auto-save
    await expect(editorPage.autoSaveIndicator).toHaveText('Auto-saved');
    await expect(editorPage.statusBadge).toHaveText('Editing');
  });

  await test.step('Save and return to read-only', async () => {
    await editorPage.save();

    // Validate save transition
    await expect(page.getByText('Document saved')).toBeVisible();
    await expect(editorPage.content).not.toBeEditable();
    await expect(editorPage.editButton).toBeVisible();
    await expect(editorPage.saveButton).not.toBeVisible();
    await expect(editorPage.statusBadge).toHaveText('Saved');

    // Validate content persistence
    await expect(editorPage.content).toContainText('Updated content here');
  });
});
```

### Loading State Validation

Progressive loading state validation:

```typescript
test('data loading state progression', async ({ page }) => {
  const dashboardPage = new DashboardPage(page);

  await test.step('Initial loading state', async () => {
    await dashboardPage.goto();

    // Validate loading indicators
    await expect(dashboardPage.loadingSpinner).toBeVisible();
    await expect(dashboardPage.loadingMessage).toHaveText('Loading dashboard...');
    await expect(dashboardPage.dataContainer).not.toBeVisible();
    await expect(dashboardPage.errorMessage).not.toBeVisible();
  });

  await test.step('Progressive data loading', async () => {
    // First data chunk loads
    await expect(dashboardPage.userStats).toBeVisible();
    await expect(dashboardPage.loadingMessage).toHaveText('Loading charts...');
    await expect(dashboardPage.chartContainer).not.toBeVisible();

    // Second data chunk loads
    await expect(dashboardPage.chartContainer).toBeVisible();
    await expect(dashboardPage.loadingMessage).toHaveText('Loading recent activity...');
    await expect(dashboardPage.activityFeed).not.toBeVisible();

    // Final data chunk loads
    await expect(dashboardPage.activityFeed).toBeVisible();
    await expect(dashboardPage.loadingSpinner).not.toBeVisible();
    await expect(dashboardPage.loadingMessage).not.toBeVisible();
  });

  await test.step('Validate complete loaded state', async () => {
    // All components visible and functional
    await expect(dashboardPage.userStats).toBeVisible();
    await expect(dashboardPage.chartContainer).toBeVisible();
    await expect(dashboardPage.activityFeed).toBeVisible();

    // Interactive elements enabled
    await expect(dashboardPage.refreshButton).toBeEnabled();
    await expect(dashboardPage.filterDropdown).toBeEnabled();

    // Data integrity
    await expect(dashboardPage.userCount).toMatch(/^\d+$/);
    await expect(dashboardPage.chartTitle).toBeVisible();
    await expect(dashboardPage.activityItems).toHaveCount(10);
  });
});
```

## Data Validation Strategies

### Table Data Validation

Comprehensive table validation patterns:

```typescript
test('user management table validation', async ({ page }) => {
  const userManagementPage = new UserManagementPage(page);

  await test.step('Table structure validation', async () => {
    await userManagementPage.goto();

    // Validate table presence and structure
    await expect(userManagementPage.userTable).toBeVisible();
    await expect(userManagementPage.tableHeaders).toHaveText([
      'Name',
      'Email',
      'Role',
      'Status',
      'Last Login',
      'Actions',
    ]);
    await expect(userManagementPage.userRows).toHaveCount(25); // Page size
  });

  await test.step('Data integrity validation', async () => {
    const firstRow = userManagementPage.userRows.first();

    // Validate row structure
    await expect(firstRow.getByTestId('user-name')).toBeVisible();
    await expect(firstRow.getByTestId('user-email')).toBeVisible();
    await expect(firstRow.getByTestId('user-role')).toBeVisible();
    await expect(firstRow.getByTestId('user-status')).toBeVisible();

    // Validate data formats
    await expect(firstRow.getByTestId('user-email')).toHaveText(/@.+\..+/);
    await expect(firstRow.getByTestId('last-login')).toHaveText(/\d{4}-\d{2}-\d{2}/);

    // Validate actionable elements
    await expect(firstRow.getByRole('button', { name: 'Edit' })).toBeEnabled();
    await expect(firstRow.getByRole('button', { name: 'Delete' })).toBeEnabled();
  });

  await test.step('Sorting validation', async () => {
    // Sort by name
    await userManagementPage.sortByColumn('Name');
    await expect(userManagementPage.sortIndicator('Name')).toHaveText('↑');

    // Validate sort order
    const userNames = await userManagementPage.getAllUserNames();
    const sortedNames = [...userNames].sort();
    expect(userNames).toEqual(sortedNames);

    // Reverse sort
    await userManagementPage.sortByColumn('Name');
    await expect(userManagementPage.sortIndicator('Name')).toHaveText('↓');

    const reversedNames = await userManagementPage.getAllUserNames();
    const reverseSortedNames = [...userNames].sort().reverse();
    expect(reversedNames).toEqual(reverseSortedNames);
  });

  await test.step('Filtering validation', async () => {
    // Apply role filter
    await userManagementPage.filterByRole('Admin');

    // Validate filtered results
    await expect(userManagementPage.userRows).toHaveCount(3);
    const visibleRoles = await userManagementPage.getAllUserRoles();
    expect(visibleRoles.every((role) => role === 'Admin')).toBe(true);

    // Clear filter
    await userManagementPage.clearFilters();
    await expect(userManagementPage.userRows).toHaveCount(25);
  });
});
```

### Form Data Validation

Complex form validation patterns:

```typescript
test('invoice creation form validation', async ({ page }) => {
  const invoicePage = new InvoiceCreationPage(page);

  await test.step('Dynamic line item validation', async () => {
    await invoicePage.goto();

    // Initial state
    await expect(invoicePage.lineItems).toHaveCount(1);
    await expect(invoicePage.subtotal).toHaveText('$0.00');

    // Add first line item
    await invoicePage.addLineItem({
      description: 'Web Development',
      quantity: 40,
      rate: 75.0,
    });

    // Validate calculations
    await expect(invoicePage.lineItemTotal(0)).toHaveText('$3,000.00');
    await expect(invoicePage.subtotal).toHaveText('$3,000.00');
    await expect(invoicePage.tax).toHaveText('$300.00'); // 10% tax
    await expect(invoicePage.total).toHaveText('$3,300.00');
  });

  await test.step('Multiple line items validation', async () => {
    // Add second line item
    await invoicePage.addLineItem({
      description: 'Consulting',
      quantity: 10,
      rate: 150.0,
    });

    // Validate updated calculations
    await expect(invoicePage.lineItems).toHaveCount(2);
    await expect(invoicePage.lineItemTotal(1)).toHaveText('$1,500.00');
    await expect(invoicePage.subtotal).toHaveText('$4,500.00');
    await expect(invoicePage.tax).toHaveText('$450.00');
    await expect(invoicePage.total).toHaveText('$4,950.00');
  });

  await test.step('Discount application validation', async () => {
    await invoicePage.applyDiscount(10); // 10% discount

    // Validate discount calculations
    await expect(invoicePage.subtotal).toHaveText('$4,500.00');
    await expect(invoicePage.discount).toHaveText('-$450.00');
    await expect(invoicePage.discountedSubtotal).toHaveText('$4,050.00');
    await expect(invoicePage.tax).toHaveText('$405.00'); // Tax on discounted amount
    await expect(invoicePage.total).toHaveText('$4,455.00');
  });
});
```

## Error Handling Patterns

### Validation Error Patterns

Comprehensive form validation error handling:

```typescript
test('registration form validation errors', async ({ page }) => {
  const registrationPage = new RegistrationPage(page);

  await test.step('Required field validation', async () => {
    await registrationPage.goto();
    await registrationPage.attemptSubmission();

    // Validate required field errors
    await expect(registrationPage.emailError).toHaveText('Email is required');
    await expect(registrationPage.passwordError).toHaveText('Password is required');
    await expect(registrationPage.firstNameError).toHaveText('First name is required');

    // Validate error styling
    await expect(registrationPage.emailField).toHaveClass(/error/);
    await expect(registrationPage.passwordField).toHaveClass(/error/);
    await expect(registrationPage.submitButton).toBeDisabled();
  });

  await test.step('Format validation errors', async () => {
    await registrationPage.fillEmail('invalid-email');
    await registrationPage.fillPassword('weak');

    // Validate format errors
    await expect(registrationPage.emailError).toHaveText('Please enter a valid email address');
    await expect(registrationPage.passwordError).toHaveText('Password must be at least 8 characters');

    // Validate real-time validation
    await registrationPage.fillEmail('valid@example.com');
    await expect(registrationPage.emailError).not.toBeVisible();
    await expect(registrationPage.emailField).not.toHaveClass(/error/);
  });

  await test.step('Server-side validation errors', async () => {
    await registrationPage.fillValidForm({
      email: 'existing@example.com', // Existing email
      password: 'ValidPass123!',
      firstName: 'John',
      lastName: 'Doe',
    });

    await registrationPage.submit();

    // Validate server error handling
    await expect(page.getByRole('alert')).toBeVisible();
    await expect(page.getByRole('alert')).toHaveText('Email address is already registered');
    await expect(registrationPage.emailField).toHaveClass(/error/);
    await expect(registrationPage.submitButton).toBeEnabled(); // Can retry
  });
});
```

### Network Error Handling

Network and connectivity error patterns:

```typescript
test('network error handling', async ({ page, context }) => {
  const dashboardPage = new DashboardPage(page);

  await test.step('Offline state handling', async () => {
    await dashboardPage.goto();

    // Simulate network failure
    await context.setOffline(true);
    await dashboardPage.refreshData();

    // Validate offline error state
    await expect(page.getByRole('alert')).toBeVisible();
    await expect(page.getByText('Connection lost')).toBeVisible();
    await expect(dashboardPage.retryButton).toBeVisible();
    await expect(dashboardPage.dataContainer).toHaveClass(/offline/);
  });

  await test.step('Recovery from offline state', async () => {
    // Restore network
    await context.setOffline(false);
    await dashboardPage.retry();

    // Validate recovery
    await expect(page.getByText('Connection restored')).toBeVisible();
    await expect(dashboardPage.dataContainer).not.toHaveClass(/offline/);
    await expect(dashboardPage.retryButton).not.toBeVisible();
    await expect(dashboardPage.data).toBeVisible();
  });
});
```

## Progressive Validation

### Step-by-Step Validation

Progressive validation for complex interactions:

```typescript
test('file upload with progress validation', async ({ page }) => {
  const uploadPage = new FileUploadPage(page);

  await test.step('File selection validation', async () => {
    await uploadPage.goto();

    // Initial state
    await expect(uploadPage.dropZone).toBeVisible();
    await expect(uploadPage.uploadButton).toBeDisabled();
    await expect(uploadPage.progressBar).not.toBeVisible();

    // Select file
    await uploadPage.selectFile('large-document.pdf');

    // Validate file selection
    await expect(uploadPage.selectedFileName).toHaveText('large-document.pdf');
    await expect(uploadPage.fileSize).toHaveText('2.5 MB');
    await expect(uploadPage.uploadButton).toBeEnabled();
  });

  await test.step('Upload progress validation', async () => {
    await uploadPage.startUpload();

    // Validate upload initialization
    await expect(uploadPage.progressBar).toBeVisible();
    await expect(uploadPage.progressPercent).toHaveText('0%');
    await expect(uploadPage.uploadButton).toBeDisabled();
    await expect(uploadPage.cancelButton).toBeVisible();

    // Validate progress updates
    await expect(uploadPage.progressPercent).toHaveText('25%', { timeout: 5000 });
    await expect(uploadPage.progressBar).toHaveCSS('width', '25%');

    await expect(uploadPage.progressPercent).toHaveText('50%', { timeout: 5000 });
    await expect(uploadPage.progressBar).toHaveCSS('width', '50%');

    await expect(uploadPage.progressPercent).toHaveText('100%', { timeout: 10000 });
  });

  await test.step('Upload completion validation', async () => {
    // Validate completion state
    await expect(page.getByText('Upload completed successfully')).toBeVisible();
    await expect(uploadPage.progressBar).not.toBeVisible();
    await expect(uploadPage.downloadLink).toBeVisible();
    await expect(uploadPage.uploadButton).toBeEnabled(); // Ready for next upload

    // Validate file processing
    await expect(uploadPage.thumbnailPreview).toBeVisible();
    await expect(uploadPage.fileInfo).toContainText('large-document.pdf');
    await expect(uploadPage.processingStatus).toHaveText('Processing complete');
  });
});
```

## Conditional Assertion Patterns

### Feature Flag Dependent Validation

```typescript
test('dashboard with feature flags', async ({ page }) => {
  const dashboardPage = new DashboardPage(page);
  const features = await getFeatureFlags();

  await dashboardPage.goto();

  // Always present elements
  await expect(dashboardPage.header).toBeVisible();
  await expect(dashboardPage.navigation).toBeVisible();

  // Conditional feature validation
  if (features.advancedAnalytics) {
    await expect(dashboardPage.analyticsPanel).toBeVisible();
    await expect(dashboardPage.chartContainer).toBeVisible();
    await expect(dashboardPage.metricsGrid).toHaveCount(6);
  } else {
    await expect(dashboardPage.basicStatsPanel).toBeVisible();
    await expect(dashboardPage.metricsGrid).toHaveCount(3);
  }

  if (features.realTimeUpdates) {
    await expect(dashboardPage.liveIndicator).toBeVisible();
    await expect(dashboardPage.refreshInterval).toHaveText('Live');
  } else {
    await expect(dashboardPage.lastUpdated).toBeVisible();
    await expect(dashboardPage.refreshButton).toBeVisible();
  }

  if (features.exportOptions) {
    await expect(dashboardPage.exportButton).toBeVisible();
    await expect(dashboardPage.exportFormats).toHaveCount(3);
  }
});
```

### User Role Based Validation

```typescript
test('admin dashboard access validation', async ({ page }) => {
  const dashboardPage = new DashboardPage(page);
  const userRole = await getCurrentUserRole();

  await dashboardPage.goto();

  // Common elements for all users
  await expect(dashboardPage.userProfile).toBeVisible();
  await expect(dashboardPage.mainContent).toBeVisible();

  // Role-specific validation
  switch (userRole) {
    case 'admin':
      await expect(dashboardPage.adminPanel).toBeVisible();
      await expect(dashboardPage.userManagementLink).toBeVisible();
      await expect(dashboardPage.systemSettingsLink).toBeVisible();
      await expect(dashboardPage.auditLogLink).toBeVisible();
      break;

    case 'manager':
      await expect(dashboardPage.adminPanel).not.toBeVisible();
      await expect(dashboardPage.teamManagementLink).toBeVisible();
      await expect(dashboardPage.reportsLink).toBeVisible();
      break;

    case 'user':
      await expect(dashboardPage.adminPanel).not.toBeVisible();
      await expect(dashboardPage.userManagementLink).not.toBeVisible();
      await expect(dashboardPage.personalSettingsLink).toBeVisible();
      break;
  }
});
```

## Performance-Aware Assertions

### Optimized Assertion Patterns

```typescript
test('performance optimized validation', async ({ page }) => {
  const searchPage = new SearchPage(page);

  await test.step('Efficient batch validation', async () => {
    await searchPage.performSearch('playwright testing');

    // Store locators for reuse
    const resultsList = page.getByTestId('search-results');
    const firstResult = resultsList.getByTestId('result-item').first();

    // Batch related assertions
    await expect(resultsList).toBeVisible();
    await expect(resultsList.getByTestId('result-item')).toHaveCount(10);

    // Validate first result structure efficiently
    await expect(firstResult).toBeVisible();
    await expect(firstResult.getByRole('heading')).toBeVisible();
    await expect(firstResult.getByTestId('snippet')).toBeVisible();
    await expect(firstResult.getByTestId('url')).toBeVisible();
  });

  await test.step('Parallel validation where safe', async () => {
    // These can be validated in parallel as they don't depend on each other
    await Promise.all([
      expect(searchPage.searchStats).toHaveText(/Found \d+ results/),
      expect(searchPage.sortOptions).toBeVisible(),
      expect(searchPage.filterPanel).toBeVisible(),
      expect(searchPage.paginationControls).toBeVisible(),
    ]);
  });
});
```

This comprehensive guide provides advanced patterns for implementing sophisticated assertion strategies in your Playwright tests. These patterns ensure thorough validation while maintaining test performance and reliability.
