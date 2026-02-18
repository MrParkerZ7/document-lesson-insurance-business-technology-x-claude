# 11.6 Stakeholder Communication

## Overview

Effective stakeholder communication is critical for Product Owner success. This sub-lesson covers templates and best practices for sprint reviews, release notes, and ongoing stakeholder engagement in insurance technology projects.

---

## Sprint Review Agenda

The sprint review is a key opportunity to demonstrate progress and gather feedback:

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

### Sprint Review Best Practices

| Do | Don't |
|----|-------|
| Demo working software | Show mockups or prototypes |
| Focus on business value | Get into technical details |
| Invite feedback | Present as final |
| Capture action items | Ignore concerns |
| Keep to time | Let discussions drag |

---

## Release Notes Template

Release notes communicate changes to stakeholders and users:

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

### Release Notes Distribution

| Audience | Format | Content Focus |
|----------|--------|---------------|
| **Executive** | Summary email | Business impact, KPIs |
| **Operations** | Detailed document | Process changes, training |
| **Technical** | Full release notes | APIs, migrations, known issues |
| **Customers** | In-app/email | New features, benefits |

---

## Stakeholder Communication Matrix

### Communication Frequency

| Stakeholder | Daily | Weekly | Sprint | Monthly | Quarterly |
|-------------|-------|--------|--------|---------|-----------|
| Dev Team | Standup | - | Review | - | - |
| Business Sponsor | - | Status | Review | Steering | Strategy |
| Compliance | - | - | As needed | Review | Audit |
| Operations | - | Sync | Review | Training | - |
| Customers | - | - | - | Newsletter | Survey |

### Communication Channels

| Channel | Best For | Frequency |
|---------|----------|-----------|
| **Daily Standup** | Team coordination | Daily |
| **Status Email** | Progress updates | Weekly |
| **Sprint Review** | Demo and feedback | Bi-weekly |
| **Steering Committee** | Strategic decisions | Monthly |
| **Roadmap Presentation** | Annual planning | Quarterly |
| **One-on-One** | Sensitive discussions | As needed |

---

## Status Report Template

Weekly status updates keep stakeholders informed:

```markdown
## Weekly Status Report - Week 3

### Summary
Sprint 24 is on track. Online endorsements feature completed ahead of schedule.
Claims tracking in final testing.

### Completed This Week
- [Done] Online endorsements - all types implemented
- [Done] Premium recalculation integration
- [Done] Document regeneration

### In Progress
- [80%] Claims tracking - visual timeline
- [60%] Mobile performance optimization

### Blockers/Risks
- [RISK] Third-party claims API response times degrading
  - Mitigation: Implementing caching layer

### Upcoming
- Claims tracking release (next Tuesday)
- Begin payment method management

### Metrics
| Metric | Target | Actual | Status |
|--------|--------|--------|--------|
| Sprint velocity | 45 | 47 | GREEN |
| Defect rate | <5% | 3% | GREEN |
| Test coverage | >80% | 82% | GREEN |

### Decisions Needed
- Approve marketing copy for claims tracking launch
```

---

## Meeting Facilitation

### Sprint Planning Meeting

**Duration:** 2-4 hours (depending on sprint length)

| Phase | Duration | Activity |
|-------|----------|----------|
| **Review Goals** | 15 min | Confirm sprint objective |
| **Backlog Presentation** | 30 min | PO presents prioritized items |
| **Estimation** | 60 min | Team estimates stories |
| **Commitment** | 30 min | Team commits to sprint scope |
| **Task Breakdown** | 45 min | Break stories into tasks |

### Stakeholder Workshop

**Purpose:** Gather requirements for new initiative

| Phase | Duration | Activity |
|-------|----------|----------|
| **Context Setting** | 10 min | Problem statement, goals |
| **Current State** | 15 min | Pain points, existing process |
| **Ideation** | 30 min | Brainstorm solutions |
| **Prioritization** | 20 min | Vote on ideas |
| **Next Steps** | 5 min | Action items, timeline |

---

## Difficult Conversations

### Handling Scope Changes

When stakeholders request changes mid-sprint:

1. **Acknowledge** the request
2. **Assess** impact on current commitments
3. **Present** trade-off options
4. **Document** the decision
5. **Communicate** changes to team

### Managing Expectations

| Situation | Approach |
|-----------|----------|
| **Feature delay** | Communicate early, provide alternatives |
| **Budget constraint** | Prioritize, defer nice-to-haves |
| **Quality concern** | Present data, recommend timeline adjustment |
| **Stakeholder conflict** | Facilitate discussion, escalate if needed |

### Saying No Professionally

```
"I understand this feature is important to you. Here's how we can address it:

Option A: Include it in the current sprint, which means [X] will be deferred
Option B: Plan it for the next sprint after [current priority]
Option C: Deliver a simplified version now, enhance later

Which approach works best for your business needs?"
```

---

## Feedback Collection

### Sprint Retrospective Questions

For gathering team feedback:

1. What went well this sprint?
2. What could be improved?
3. What will we commit to improving?

### Stakeholder Feedback Survey

| Question | Scale |
|----------|-------|
| Communication clarity | 1-5 |
| Meeting effectiveness | 1-5 |
| Roadmap visibility | 1-5 |
| Responsiveness | 1-5 |
| Overall satisfaction | 1-5 |

**Open Questions:**
- What should we start doing?
- What should we stop doing?
- What should we continue doing?

---

## Documentation Standards

### What to Document

| Type | When | Where |
|------|------|-------|
| **Decisions** | All significant decisions | Confluence/Wiki |
| **Requirements** | New features | User stories, PRDs |
| **Meeting Notes** | All stakeholder meetings | Shared drive |
| **Release Notes** | Each release | Release log |
| **Retrospectives** | Each sprint | Team space |

### Decision Log Template

```markdown
## Decision Log Entry

**Date:** 2024-01-15
**Decision:** Use third-party rating engine instead of building custom

**Context:**
Need rating engine for new cyber insurance product. Options were build vs. buy.

**Decision Rationale:**
- Faster time to market (3 months vs 9 months)
- Lower initial cost ($50K vs $200K build)
- Established market solution with good reviews

**Stakeholders Involved:**
- CTO (approved)
- Finance Director (approved)
- Product Owner (recommended)

**Impact:**
- Dependency on vendor for roadmap
- Integration effort required
- Ongoing license cost

**Review Date:** 2024-07-15 (6 months post-implementation)
```

---

## Key Takeaways

1. Regular, structured communication keeps stakeholders aligned
2. Sprint reviews should focus on demos and feedback, not status
3. Release notes should be tailored for different audiences
4. Document decisions and rationale for future reference
5. Handle difficult conversations with options and trade-offs

---

[Previous: Roadmap Planning](../05-roadmap-planning/README.md) | [Back to Lesson 11](../README.md)
