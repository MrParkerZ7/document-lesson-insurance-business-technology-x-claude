# Diagrams

This folder contains shared architectural and process diagrams used across lessons.

## Naming Convention

Diagrams follow the pattern: `[lesson].[sub-lesson]-descriptive-name.drawio`

Examples:
- `04.1-microservices-architecture.drawio`
- `05.2-policy-entity-diagram.drawio`
- `03.0-insurance-processes-overview.drawio`

## File Requirements

- All diagrams use **DrawIO** format (`.drawio` XML files)
- Each diagram must have a corresponding **PNG export** for viewing
- When modifying diagrams, update both the `.drawio` source and regenerate the `.png` export

## Styling Standards

- Enable shadows on shapes (`shadow=1`)
- Use curved arrows where appropriate (`curved=1`)
- Add flow animation to directional arrows (`flowAnimation=1`)
- Use consistent color schemes:
  - **Blue**: Services/APIs
  - **Green**: Data/Storage
  - **Orange**: External integrations
  - **Purple**: Security/Auth
  - **Gray**: Infrastructure

## Diagram Categories

| Category | Description |
|----------|-------------|
| Architecture | System and component diagrams |
| Process | Business process flows |
| Data | Entity relationships, schemas |
| Integration | API and event flows |
| DevOps | CI/CD and infrastructure |
