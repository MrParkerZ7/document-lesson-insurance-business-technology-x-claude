# Lesson 11: Product Owner Guide

## Learning Objectives

By the end of this lesson, you will be able to:
- Understand the insurance domain from a PO perspective
- Write effective user stories for insurance features
- Define acceptance criteria and KPIs
- Prioritize features using insurance-specific criteria

---

## 11.1 PO Knowledge Areas

### Domain Knowledge Requirements

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       PO KNOWLEDGE AREAS                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  BUSINESS DOMAIN                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │  - Insurance products and coverages                                  │    │
│  │  - Underwriting principles and risk assessment                      │    │
│  │  - Claims processing and settlement                                  │    │
│  │  - Regulatory requirements (local and international)                │    │
│  │  - Distribution channels (agents, brokers, digital)                 │    │
│  │  - Reinsurance basics                                                │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  TECHNICAL UNDERSTANDING                                                     │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │  - API-first architecture concepts                                   │    │
│  │  - Microservices and event-driven patterns                          │    │
│  │  - Data models and entity relationships                              │    │
│  │  - Integration patterns (sync, async, batch)                        │    │
│  │  - Security and compliance requirements                              │    │
│  │  - Performance and scalability concepts                              │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  USER EXPERIENCE                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │  - Customer journey mapping                                          │    │
│  │  - Agent/broker workflows                                            │    │
│  │  - Operations user needs (underwriters, claims handlers)            │    │
│  │  - Self-service capabilities                                         │    │
│  │  - Mobile-first design principles                                    │    │
│  │  - Accessibility requirements                                        │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  STAKEHOLDER MANAGEMENT                                                      │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │  - Business stakeholder alignment                                    │    │
│  │  - Compliance and legal coordination                                │    │
│  │  - IT and development team collaboration                            │    │
│  │  - Vendor and partner relationships                                 │    │
│  │  - Customer feedback integration                                     │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 11.2 User Story Templates

### Quote Generation Story

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

### Claim FNOL Story

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

### Policy Endorsement Story

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

## 11.3 Feature Prioritization

### Prioritization Matrix

| Factor | Weight | Description |
|--------|--------|-------------|
| **Business Value** | 30% | Revenue impact, customer satisfaction, market differentiation |
| **Regulatory Need** | 25% | Compliance requirements, legal deadlines |
| **Technical Risk** | 20% | Complexity, dependencies, unknowns |
| **Customer Impact** | 15% | User experience improvement, customer complaints |
| **Operational Efficiency** | 10% | Internal productivity gains, cost reduction |

### Priority Scoring Template

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       FEATURE PRIORITY SCORING                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  FEATURE: Online Claims Status Tracking                                      │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │  CRITERIA              SCORE (1-5)    WEIGHT    WEIGHTED SCORE      │    │
│  │  ─────────────────────────────────────────────────────────────────  │    │
│  │  Business Value            4           30%          1.2             │    │
│  │  Regulatory Need           2           25%          0.5             │    │
│  │  Technical Risk (inv)      4           20%          0.8             │    │
│  │  Customer Impact           5           15%          0.75            │    │
│  │  Operational Efficiency    3           10%          0.3             │    │
│  │  ─────────────────────────────────────────────────────────────────  │    │
│  │  TOTAL SCORE                                        3.55/5.0        │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  RECOMMENDATION: High priority - implement in next sprint                   │
│                                                                              │
│  JUSTIFICATION:                                                              │
│  - High customer impact (reduces call center volume by ~30%)                │
│  - Medium technical complexity (uses existing claim API)                    │
│  - No regulatory deadline                                                   │
│  - Aligns with digital self-service strategy                                │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### MoSCoW for Insurance Features

| Must Have | Should Have | Could Have | Won't Have |
|-----------|-------------|------------|------------|
| Quote generation | Quote comparison | AI chatbot | Blockchain claims |
| Policy issuance | Online endorsements | Voice assistant | Peer-to-peer insurance |
| Claims FNOL | Claims tracking | Gamification | Crypto payments |
| Payment processing | Auto-renewal | Social sharing | NFT policies |
| Regulatory reports | Customer dashboard | AR damage assessment | |

---

## 11.4 Key Performance Indicators (KPIs)

### Sales & Distribution KPIs

| Metric | Formula | Target | Frequency |
|--------|---------|--------|-----------|
| **Quote to Policy Conversion** | Policies / Quotes × 100 | > 25% | Daily |
| **Average Quote Time** | Sum(Quote Duration) / Count | < 2 min | Daily |
| **Digital Sales %** | Digital Policies / Total × 100 | > 40% | Monthly |
| **Agent Productivity** | Policies per Agent | > 20/month | Monthly |
| **Quote Abandonment Rate** | Abandoned / Started × 100 | < 30% | Weekly |

### Policy Operations KPIs

| Metric | Formula | Target | Frequency |
|--------|---------|--------|-----------|
| **Policy Issuance SLA** | Policies issued within SLA / Total | > 95% | Daily |
| **Endorsement Turnaround** | Avg time to process endorsement | < 4 hours | Daily |
| **Renewal Rate** | Renewed / Expiring × 100 | > 85% | Monthly |
| **Auto-Renewal Success** | Successful / Attempted × 100 | > 90% | Monthly |
| **Policy Error Rate** | Policies with errors / Total | < 0.5% | Weekly |

### Claims KPIs

| Metric | Formula | Target | Frequency |
|--------|---------|--------|-----------|
| **Claims Cycle Time** | FNOL to Settlement Duration | < 5 days | Weekly |
| **First Contact Resolution** | Resolved first contact / Total | > 70% | Weekly |
| **Claims Approval Rate** | Approved / Total × 100 | > 85% | Monthly |
| **STP Rate** | Auto-processed / Total × 100 | > 50% | Weekly |
| **Fraud Detection Rate** | Fraud caught / Total fraud | > 80% | Monthly |
| **Customer Satisfaction (Claims)** | CSAT Score | > 4.2/5 | Monthly |

### Technical KPIs

| Metric | Formula | Target | Frequency |
|--------|---------|--------|-----------|
| **System Uptime** | Uptime / Total Time × 100 | > 99.9% | Daily |
| **API Response Time** | p95 latency | < 200ms | Hourly |
| **Error Rate** | Errors / Requests × 100 | < 0.1% | Hourly |
| **Deployment Frequency** | Deployments per week | > 2 | Weekly |
| **Lead Time for Changes** | Commit to production | < 1 day | Weekly |

---

## 11.5 Roadmap Planning

### Quarterly Roadmap Template

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       QUARTERLY ROADMAP                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  Q1 2024: FOUNDATION                                                         │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │  Theme: Core Platform Stability                                      │    │
│  │                                                                      │    │
│  │  EPICS:                                                              │    │
│  │  ├── [P1] Quote Engine 2.0 (Performance Optimization)              │    │
│  │  ├── [P1] Policy Issuance API Modernization                        │    │
│  │  ├── [P2] Claims FNOL Mobile App                                   │    │
│  │  └── [P2] Customer Portal Redesign                                 │    │
│  │                                                                      │    │
│  │  SUCCESS METRICS:                                                    │    │
│  │  - Quote response time < 2 seconds                                   │    │
│  │  - Mobile FNOL adoption > 30%                                        │    │
│  │  - NPS improvement +5 points                                         │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  Q2 2024: SELF-SERVICE                                                       │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │  Theme: Customer Self-Service Expansion                              │    │
│  │                                                                      │    │
│  │  EPICS:                                                              │    │
│  │  ├── [P1] Online Policy Endorsements                                │    │
│  │  ├── [P1] Claims Status Tracking                                    │    │
│  │  ├── [P2] Payment Method Management                                 │    │
│  │  └── [P3] Document Download Center                                  │    │
│  │                                                                      │    │
│  │  SUCCESS METRICS:                                                    │    │
│  │  - Self-service endorsement > 40%                                    │    │
│  │  - Call center volume -20%                                           │    │
│  │  - Customer satisfaction > 4.0                                       │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  Q3 2024: AUTOMATION                                                         │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │  Theme: Intelligent Automation                                       │    │
│  │                                                                      │    │
│  │  EPICS:                                                              │    │
│  │  ├── [P1] Auto-Underwriting Enhancement                             │    │
│  │  ├── [P1] Claims Straight-Through Processing                        │    │
│  │  ├── [P2] Chatbot for Customer Queries                              │    │
│  │  └── [P2] Fraud Detection ML Model                                  │    │
│  │                                                                      │    │
│  │  SUCCESS METRICS:                                                    │    │
│  │  - STP rate > 60%                                                    │    │
│  │  - Fraud detection +15%                                              │    │
│  │  - Processing cost -25%                                              │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  Q4 2024: GROWTH                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │  Theme: New Product Launch & Partnerships                            │    │
│  │                                                                      │    │
│  │  EPICS:                                                              │    │
│  │  ├── [P1] Cyber Insurance Product                                   │    │
│  │  ├── [P1] Partner API Program                                       │    │
│  │  ├── [P2] Embedded Insurance Integrations                           │    │
│  │  └── [P3] Usage-Based Insurance Pilot                               │    │
│  │                                                                      │    │
│  │  SUCCESS METRICS:                                                    │    │
│  │  - New product GWP $5M                                               │    │
│  │  - Partner integrations > 10                                         │    │
│  │  - API transactions +50%                                             │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 11.6 Stakeholder Communication

### Sprint Review Agenda

```markdown
## Sprint Review - Sprint 24

### Attendees
- Product Owner
- Development Team
- Business Stakeholders
- UX Designer

### Agenda (60 minutes)

1. **Sprint Summary** (5 min)
   - Sprint goal achievement
   - Key metrics

2. **Demo: Completed Features** (30 min)
   - Online Quote Comparison Tool
   - Claims Photo Upload Enhancement
   - Policy Renewal Reminder Flow

3. **Technical Highlights** (10 min)
   - Performance improvements
   - Technical debt addressed

4. **Metrics Review** (10 min)
   - Conversion rate changes
   - Customer feedback
   - System performance

5. **Upcoming Sprint Preview** (5 min)
   - Next sprint priorities
   - Dependencies and risks

### Success Criteria
- Stakeholder feedback captured
- Priority adjustments documented
- Action items assigned
```

### Release Notes Template

```markdown
## Release Notes - v2.5.0

**Release Date:** January 15, 2024

### New Features

#### Online Policy Endorsements
Customers can now make policy changes directly through the customer portal:
- Change sum insured
- Add or remove coverage options
- Update beneficiary information

**Business Impact:** Expected 40% reduction in call center endorsement requests

#### Enhanced Claims Tracking
New real-time claims status tracking with:
- Visual timeline of claim progress
- Document upload status
- Estimated resolution date

**Business Impact:** Improved claims NPS by +8 points in pilot

### Improvements
- Quote generation time reduced by 35%
- Mobile app performance optimization
- Search functionality in policy list

### Bug Fixes
- Fixed premium calculation for multi-vehicle discounts
- Resolved document download issues on Safari
- Corrected renewal date display in customer dashboard

### Known Issues
- PDF generation may be slow for large policies (fix planned for v2.5.1)

### Migration Notes
- No database migrations required
- API backwards compatible
- No action required from operations
```

---

## Key Takeaways

1. **Domain knowledge** is essential for effective PO work in insurance
2. **User stories** should include clear acceptance criteria
3. **Prioritization** balances business value with risk and effort
4. **KPIs** measure success across sales, operations, and technology
5. **Stakeholder communication** keeps everyone aligned

---

## Exercises

1. Write user stories for policy renewal reminder feature
2. Create a priority scoring for 5 proposed features
3. Design a KPI dashboard for claims operations

---

[← Previous Lesson](../10-devops-deployment/README.md) | [Back to Main Curriculum →](../../README.md)
