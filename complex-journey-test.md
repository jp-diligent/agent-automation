# Complex Journey Test

```gherkin
Feature: Issue Management Complex Journey
  As multiple stakeholders responsible for managing issues and actions
  I want to validate the complete lifecycle of a complex journey
  So that the system enforces the correct statuses, permissions, and transitions for every role

  # --- Issue lifecycle ---
  Scenario: End-to-end complex journey from issue creation to closure

    # Admin creates subtype
    When I log in as "jpablosanchez+aut_sysadmin@diligent.com" (Admin)
    And Admin navigates to /settings/issue-types
    And adds a subtype
    And assigns "jpablosanchez+aut_categorymanager@diligent.com" as the Type Manager
    Then the subtype is created

    # Reporter creates the issue for a sub-type
    When I log in as "jpablosanchez+aut_issuereporter@diligent.com" (Reporter)
    And the Reporter creates a new Issue
    Then the Issue status should be "Open"

    # Type Manager saves without assignment
    When I log in as "jpablosanchez+aut_categorymanager@diligent.com" (Type Manager)
    And the Type Manager opens Manage Issue
    And adds Action 1
    And saves without assignment
    Then Action 1 should have status "DRAFT"
    And the Issue remains "Open"

    # Type Manager assigns the Issue
    When the Type Manager assigns the Issue
    Then the Issue status should be "In progress" and not initiated

    # Issue Owner saves without initiation
    When I log in as "jpablosanchez+aut_issueowner@diligent.com" (Issue Owner)
    And the Issue Owner edits issue
    And adds Action 2
    And sets "Automation ActionOwner1" as the owner of Action 1
    And sets "Automation ActionOwner2" as the owner of Action 2
    And sets "Automation ActionApprover2" as the approver of Action 2
    And saves without initiation
    Then the Issue status should be "In Progress" and not inititated
    And Action 1 should have status "DRAFT"
    And Action 2 should have status "DRAFT"
    
    # Issue Owner initiates the Issue
    When the Issue Owner initiates the Issue
    Then Action 1 should have status "OPEN"
    And Action 2 should have status "PENDING"

    # Action 1 Owner processes Action 1
    When I log in as "jpablosanchez+aut_actionowner1@diligent.com" (Action 1 Owner)
    And the Action 1 Owner starts Action 1
    And the Action 1 Owner marks Action 1 as Done
    Then Action 1 status should be "COMPLETED"
    And Action 2 status should change to "OPEN"

    # Action 2 Owner processes Action 2
    When I log in as "jpablosanchez+aut_actionowner2@diligent.com" (Action 2 Owner)
    And the Action 2 Owner starts Action 2
    And the Action 2 Owner marks Action 2 as Done
    Then Action 2 status should be "TO BE APPROVED"

    # Action 2 Approver review cycle
    When I log in as "jpablosanchez+aut_actionapprover2@diligent.com" (Action 2 Approver)
    And the Action 2 Approver declines Action 2
    Then Action 2 status should be "IN PROGRESS"

    When I log in as "jpablosanchez+aut_actionowner2@diligent.com" (Action 2 Owner)
    And the Action 2 Owner marks Action 2 as Done again

    When I log in as "jpablosanchez+aut_actionapprover2@diligent.com"" (Action 2 Approver)
    And the Action 2 Approver approves Action 2
    Then Action 1 and Action 2 should both be "COMPLETED"
    And the Issue status should be "TO BE APPROVED"

    # Issue Approver final decision
    When I log in as "jpablosanchez+aut_issueapprover@diligent.com" (Issue Approver)
    And the Issue Approver approves the Issue
    Then the Issue status should be "CLOSED"
```

