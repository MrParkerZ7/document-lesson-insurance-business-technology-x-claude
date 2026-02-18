# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an educational curriculum repository for **IT Developers** and **Product Owners** working in the insurance industry. It contains comprehensive lessons covering insurance business fundamentals, technology systems, architecture patterns, and best practices. This is NOT a software application - it's a knowledge base with markdown documentation and visual diagrams.

## Repository Structure

```
document-lesson-insurance-business-technology-x-claude/
├── README.md                           # Main curriculum document (comprehensive guide)
├── CLAUDE.md                           # Claude Code guidance file
├── lessons/                            # Individual lesson modules
│   ├── 01-insurance-fundamentals/
│   │   ├── README.md                   # Lesson content
│   │   ├── 01.1-diagram-name.drawio    # DrawIO diagram source
│   │   └── 01.1-diagram-name.png       # PNG export of diagram
│   ├── 02-insurance-products/
│   ├── 03-core-processes/
│   ├── 04-system-architecture/
│   ├── 05-data-models/
│   ├── 06-api-design/
│   ├── 07-security-compliance/
│   ├── 08-development-practices/
│   ├── 09-testing-strategies/
│   ├── 10-devops-deployment/
│   └── 11-product-owner-guide/
├── diagrams/                           # Shared architectural diagrams
├── examples/                           # Code examples and templates
│   ├── api-contracts/
│   ├── database-schemas/
│   └── event-schemas/
└── resources/                          # Additional learning resources
```

## Lesson Topics

- **01**: Insurance Business Fundamentals (value chain, stakeholders, business model)
- **02**: Insurance Product Types (life, health, motor, property, liability)
- **03**: Core Insurance Processes (underwriting, policy admin, claims, billing)
- **04**: System Architecture (microservices, API gateway, event-driven)
- **05**: Data Models & Database Design (entities, schemas, state machines)
- **06**: API Design & Integration (REST, events, external integrations)
- **07**: Security & Compliance (GDPR, regulatory, data protection)
- **08**: Development Best Practices (patterns, conventions, error handling)
- **09**: Testing Strategies (unit, integration, E2E, test data)
- **10**: DevOps & Deployment (CI/CD, Kubernetes, monitoring)
- **11**: Product Owner Guide (domain knowledge, user stories, KPIs)

## Target Audience

### IT Developers
- Backend/frontend developers building insurance platforms
- API designers and integration specialists
- DevOps engineers working on insurance systems
- QA engineers testing insurance applications

### Product Owners
- POs managing insurance product development
- Business analysts in insurance domain
- Technical product managers
- Scrum masters working with insurance teams

## Working with Diagrams

### Naming Convention
- Diagrams follow the pattern: `[lesson].[sub-lesson]-descriptive-name.drawio`
- Examples: `04.1-microservices-architecture.drawio`, `05.2-policy-entity-diagram.drawio`
- Lesson overview diagrams use `.0` suffix: `03.0-insurance-processes-overview.drawio`

### File Requirements
- All diagrams use DrawIO format (`.drawio` XML files)
- Each diagram must have a corresponding PNG export for viewing
- When modifying diagrams, update both the `.drawio` source and regenerate the `.png` export

### Styling Standards
- Enable shadows on shapes (`shadow=1`)
- Use curved arrows where appropriate (`curved=1`)
- Add flow animation to directional arrows (`flowAnimation=1`)
- Use consistent color schemes:
  - Blue: Services/APIs
  - Green: Data/Storage
  - Orange: External integrations
  - Purple: Security/Auth
  - Gray: Infrastructure
- Include title and descriptive labels in diagrams

## Content Conventions

### Documentation Structure
- Each lesson README follows consistent structure:
  - Overview and learning objectives
  - Detailed content with diagrams
  - Code examples (where applicable)
  - Key takeaways
  - Further reading

### Insurance Domain Terminology
- Use standard ACORD terminology where applicable
- Include glossary references for technical insurance terms
- Explain acronyms on first use (FNOL, TPA, MGA, etc.)

### Code Examples
- Use realistic insurance domain examples
- Include schema definitions in SQL format
- API examples in OpenAPI/JSON format
- Event schemas in JSON with clear documentation

## Key Insurance Concepts Covered

### Business Processes
- Policy Lifecycle: Quote → Proposal → Issuance → Endorsement → Renewal → Expiry
- Claims Flow: FNOL → Triage → Assessment → Decision → Settlement
- Underwriting: Risk assessment, rating, acceptance criteria

### Technical Components
- Core Systems: Product, Quote, Policy, Claims, Billing, Customer
- Integration Points: Payment gateways, KYC/AML, reinsurance
- Data Patterns: Event sourcing, CQRS, state machines

### Regulatory Compliance
- Data Protection: GDPR, data masking, consent management
- Financial Reporting: IFRS 17, Solvency II
- Industry Standards: PCI-DSS, HIPAA (for health)

## Git Workflow

Commits in this repository include Claude as co-author:
```
Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>
```

## Commands

### Documentation
- No build process required - all content is markdown
- Preview markdown files directly in IDE or GitHub

### Diagram Export
- Open `.drawio` files in Draw.io (desktop or web)
- Export as PNG with transparent background
- Match filename with `.drawio` source file

## Contributing Guidelines

1. Follow existing folder structure and naming conventions
2. Include diagrams for complex concepts
3. Use consistent terminology from glossary
4. Add practical examples relevant to insurance domain
5. Update table of contents when adding new sections
6. Ensure all links between documents are valid
