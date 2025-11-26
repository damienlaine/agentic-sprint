# Project Goals

> High-level product objectives that guide sprint planning and architectural decisions.

## Vision

[One sentence describing what this product aims to achieve]

Example: "Build a collaborative task management platform that helps remote teams stay aligned and productive."

## Target Users

- **Primary:** [Who is the main user?]
- **Secondary:** [Other user types?]

Example:
- **Primary:** Remote team managers who need visibility into team progress
- **Secondary:** Individual contributors tracking their own tasks

## Core Problems Solved

1. [Problem 1]
2. [Problem 2]
3. [Problem 3]

Example:
1. Teams lose track of task status across multiple tools
2. Managers lack real-time visibility into blockers
3. Context switching between communication and task management wastes time

## Key Features (MVP)

- [ ] [Feature 1]
- [ ] [Feature 2]
- [ ] [Feature 3]

Example:
- [ ] User authentication (OAuth + email/password)
- [ ] Project and task CRUD
- [ ] Real-time task status updates
- [ ] Team dashboards with progress metrics

## Success Metrics

- [Metric 1]
- [Metric 2]

Example:
- Time to onboard a new team < 5 minutes
- Daily active users maintain 80%+ task update rate

## Technical Constraints

- [Constraint 1]
- [Constraint 2]

Example:
- Must support 1000+ concurrent users
- Must work offline with sync capabilities
- Must comply with GDPR

## Out of Scope (for MVP)

- [Not included 1]
- [Not included 2]

Example:
- Mobile native apps (web-responsive only for MVP)
- Advanced reporting and analytics
- Third-party integrations

## Timeline

- **MVP Target:** [Date or sprint number]
- **Beta:** [Date]
- **Launch:** [Date]

---

# Example: SaaS Starter Project Goals

## Vision

Build a modern SaaS starter kit that accelerates development of subscription-based web applications.

## Target Users

- **Primary:** Solo developers and small teams building SaaS products
- **Secondary:** Agencies needing a consistent starting point for client projects

## Core Problems Solved

1. Setting up auth, payments, and user management takes weeks
2. Inconsistent patterns across projects lead to maintenance burden
3. Security best practices are often missed in rush to ship

## Key Features (MVP)

- [x] User authentication (email/password + OAuth)
- [x] Subscription management with Stripe
- [x] User dashboard and settings
- [x] Admin panel for user management
- [ ] Team/organization support
- [ ] Usage-based billing option

## Success Metrics

- New project setup time < 30 minutes
- Zero critical security vulnerabilities
- Production-ready with minimal configuration

## Technical Constraints

- Next.js 16+ with App Router
- PostgreSQL database
- Docker-based development and deployment
- Support for both French and English (i18n)

## Out of Scope (for MVP)

- Native mobile apps
- Custom domain support
- White-labeling capabilities
- Advanced analytics dashboard

---

# Example: Internal Tool Project Goals

## Vision

Build an internal inventory management system to replace spreadsheet-based tracking.

## Target Users

- **Primary:** Warehouse staff managing daily inventory
- **Secondary:** Managers reviewing stock levels and generating reports

## Core Problems Solved

1. Spreadsheets are error-prone and don't support concurrent editing
2. No audit trail for inventory changes
3. Stock alerts are manual and often missed

## Key Features (MVP)

- [ ] Product catalog management
- [ ] Inventory transactions (in/out)
- [ ] Real-time stock levels
- [ ] Low stock alerts
- [ ] Basic reporting (stock value, movement history)

## Success Metrics

- Inventory accuracy > 99%
- Time to record transaction < 10 seconds
- Zero missed low-stock alerts

## Technical Constraints

- Must work on tablets in warehouse (responsive)
- Must integrate with existing ERP for product data
- On-premise deployment (no cloud)

## Out of Scope (for MVP)

- Barcode scanning (manual entry only)
- Purchase order generation
- Supplier management
- Multi-warehouse support
