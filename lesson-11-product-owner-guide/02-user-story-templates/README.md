# 11.2 User Story Templates

## Overview

This sub-lesson provides practical user story templates for common insurance features. Well-written user stories with clear acceptance criteria are essential for successful sprint planning and delivery.

---

## User Story Structure

A complete user story includes:

```
+----------------------------------------------------------+
|  USER STORY COMPONENTS                                     |
+----------------------------------------------------------+
|                                                            |
|  1. TITLE: Brief, descriptive name                         |
|                                                            |
|  2. USER STORY FORMAT:                                     |
|     AS A [user type]                                       |
|     I WANT TO [action/goal]                                |
|     SO THAT [benefit/value]                                |
|                                                            |
|  3. ACCEPTANCE CRITERIA: GIVEN/WHEN/THEN format            |
|                                                            |
|  4. TECHNICAL NOTES: Implementation guidance               |
|                                                            |
|  5. DEFINITION OF DONE: Completion checklist               |
|                                                            |
+----------------------------------------------------------+
```

---

## Quote Generation Story

```markdown
## User Story: Online Quote Generation

**AS A** potential customer
**I WANT TO** get an instant quote for motor insurance online
**SO THAT** I can understand the cost before purchasing

### Acceptance Criteria

**GIVEN** I am on the quote page
**WHEN** I enter my vehicle details (make, model, year, registration)
**AND** I enter my personal details (name, DOB, license info)
**AND** I select coverage options
**THEN** I should see a premium breakdown within 3 seconds

**GIVEN** I have completed a quote
**WHEN** I view the results
**THEN** I should see:
- Base premium amount
- All applicable discounts with explanations
- All applicable loadings with explanations
- Total annual premium
- Monthly payment option

**GIVEN** I have received a quote
**WHEN** I want to save it for later
**THEN** I should be able to:
- Save the quote to my account (if logged in)
- Receive the quote via email (if guest)
- Access the saved quote for 30 days

### Technical Notes
- Integrate with vehicle database for auto-fill
- Use rating engine for premium calculation
- Store quote in MongoDB for flexibility
- Generate unique quote reference number
- Track quote conversion funnel

### Definition of Done
- [ ] Unit tests pass (>90% coverage)
- [ ] Integration tests pass
- [ ] API documentation updated
- [ ] Performance: < 3 second response time
- [ ] Accessibility: WCAG 2.1 AA compliant
- [ ] Analytics events implemented
```

---

## Claim FNOL Story

```markdown
## User Story: Online Claim Registration (FNOL)

**AS A** policyholder
**I WANT TO** report a claim online
**SO THAT** I can initiate the claims process immediately after an incident

### Acceptance Criteria

**GIVEN** I am logged in to my account
**AND** I have an active policy
**WHEN** I click "Report a Claim"
**THEN** I should see a claim form with my policy details pre-filled

**GIVEN** I am completing the claim form
**WHEN** I enter the incident details
**THEN** I should be able to:
- Select the date and time of incident
- Mark the location on a map or enter address
- Describe what happened (free text)
- Upload photos (up to 10, max 10MB each)
- Add witness information (optional)

**GIVEN** I have submitted the claim
**WHEN** the submission is successful
**THEN** I should:
- See a confirmation screen with claim number
- Receive a confirmation email immediately
- See the claim in my "My Claims" section
- See next steps and required documents

**GIVEN** I have an open claim
**WHEN** I view the claim status
**THEN** I should see:
- Current status with clear explanation
- Timeline of all events
- Assigned adjuster contact information
- List of required/submitted documents
- Estimated timeline for resolution

### Non-Functional Requirements
- Form submission: < 5 seconds
- File uploads: Support offline with sync
- Mobile responsive design
- Work on low bandwidth connections

### Definition of Done
- [ ] Form validation complete
- [ ] File upload tested with large files
- [ ] Email notification configured
- [ ] Fraud indicators captured
- [ ] Claims dashboard updated
- [ ] Mobile testing complete
```

---

## Policy Endorsement Story

```markdown
## User Story: Self-Service Policy Endorsement

**AS A** policyholder
**I WANT TO** make changes to my policy online
**SO THAT** I can update my coverage without calling customer service

### Acceptance Criteria

**GIVEN** I am viewing my active policy
**WHEN** I click "Make Changes"
**THEN** I should see available endorsement options:
- Change sum insured
- Add/remove coverage
- Update beneficiary
- Change payment method
- Update contact information

**GIVEN** I want to change my sum insured
**WHEN** I enter a new amount
**THEN** I should see:
- Premium adjustment (refund or additional)
- Effective date options
- Clear explanation of coverage change

**GIVEN** I have confirmed an endorsement
**WHEN** payment is required
**THEN** I should:
- See secure payment options
- Receive confirmation upon success
- Get updated policy documents via email

### Business Rules
- Endorsements effective same day if before 4 PM
- Sum insured increase may require underwriting approval
- Some changes require waiting period
- Maximum 2 sum insured changes per year

### Definition of Done
- [ ] All endorsement types implemented
- [ ] Premium recalculation accurate
- [ ] Document regeneration working
- [ ] Audit trail complete
- [ ] Underwriting rules applied
```

---

## Additional Story Templates

### Policy Renewal Story

```markdown
## User Story: Online Policy Renewal

**AS A** policyholder with an expiring policy
**I WANT TO** renew my policy online
**SO THAT** I can maintain continuous coverage

### Acceptance Criteria

**GIVEN** my policy is within 30 days of expiry
**WHEN** I log in to my account
**THEN** I should see a renewal reminder prominently displayed

**GIVEN** I am reviewing my renewal
**WHEN** I view the renewal offer
**THEN** I should see:
- Previous year's premium
- New premium with explanation of changes
- Coverage comparison
- Option to adjust coverage

**GIVEN** I accept the renewal
**WHEN** payment is successful
**THEN** I should receive:
- Immediate confirmation
- New policy documents via email
- Updated policy in my account
```

### Document Upload Story

```markdown
## User Story: Supporting Document Upload

**AS A** policyholder or claimant
**I WANT TO** upload supporting documents online
**SO THAT** I can complete my application without mailing documents

### Acceptance Criteria

**GIVEN** I have a pending document request
**WHEN** I access the document upload section
**THEN** I should see:
- List of required documents
- Clear instructions for each
- Accepted file formats

**GIVEN** I am uploading a document
**WHEN** the upload completes
**THEN** I should:
- See a preview of the uploaded file
- Be able to replace if needed
- Receive confirmation when all documents submitted
```

---

## Story Writing Best Practices

### Do's

| Practice | Example |
|----------|---------|
| Focus on user value | "SO THAT I can save time" |
| Be specific in criteria | "within 3 seconds" not "quickly" |
| Include error scenarios | "GIVEN the payment fails..." |
| Reference business rules | "Maximum 2 changes per year" |

### Don'ts

| Avoid | Better Approach |
|-------|-----------------|
| Technical implementation details | Keep in technical notes |
| Vague acceptance criteria | Use GIVEN/WHEN/THEN |
| Multiple features in one story | Split into smaller stories |
| Assuming context | Make criteria explicit |

---

## Key Takeaways

1. User stories should follow a consistent format
2. Acceptance criteria define when a story is complete
3. Technical notes guide implementation without dictating
4. Definition of Done ensures quality standards
5. Business rules capture domain-specific requirements

---

[Previous: PO Knowledge Areas](../01-po-knowledge-areas/README.md) | [Back to Lesson 11](../README.md) | [Next: Feature Prioritization](../03-feature-prioritization/README.md)
