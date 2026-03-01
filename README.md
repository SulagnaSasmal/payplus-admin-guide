# PayPlus Enterprise — Administration Guide

**A documentation portfolio project by Sulagna Sasmal — Senior Technical Writer | Documentation Architect**

---

## About This Project

This is a fully authored enterprise SaaS administration guide for a fictional payment hub platform — **PayPlus Enterprise v3.2**. It demonstrates enterprise HTML documentation for a complex, regulated FinTech product.

**Documentation style:** Enterprise HTML documentation (MadCap Flare-inspired structure, rendered as a static HTML site)
**Audience:** System administrators and bank IT operations staff
**Product modeled on:** Multi-rail enterprise payment hub systems (Finastra GlobalPAYplus, Temenos Payments Hub, Fiserv Payments Hub) — reflecting real-world experience authoring operations documentation for Fundtech CashIn and Global CASHplus (direct predecessors of GlobalPAYplus) at Fundtech India (2008–2010)

---

## What's Documented

| Section | Description |
|---|---|
| [Introduction & Architecture](index.html) | Product overview, 3-tier architecture, deployment models, document map |
| [Installation](installation.html) | System requirements, pre-installation checklist, installation procedure (step-by-step), database setup (Oracle/SQL Server), post-installation validation, upgrade guidance |
| [User Management](user-management.html) | Built-in roles and permissions matrix, creating users, LDAP/Active Directory integration, SSO (SAML 2.0) configuration, password policy, session management |
| [Payment Rail Connectors](payment-rails.html) | Configuration parameters for ACH, Fedwire (ISO 20022), SWIFT (MT/MX), RTP (TCH), and FedNow connectors; connectivity testing |
| [Workflow Engine](workflow.html) | Payment lifecycle states, approval workflows (STP/single/dual-control), rules engine (routing/approval/block/enrichment/velocity), payment templates, bulk ACH processing |
| [Monitoring & Alerts](monitoring.html) | System dashboard, payment queue monitor, built-in alert rules, SLA monitoring, audit log viewer, settlement reports |
| [Compliance Configuration](compliance.html) | OFAC sanctions screening engine, hold queue management (release/reject/escalate), AML integration, BSA Travel Rule, regulatory reporting |
| [Troubleshooting](troubleshooting.html) | System error codes (1xxx), ACH errors (2xxx), Fedwire/SWIFT errors (3xxx/4xxx), RTP/FedNow errors (5xxx), common issues, log file locations, support escalation |

---

## Documentation Approach

This guide demonstrates:

- **Product administration depth** — not a feature overview. Documents configuration parameters, installation sequences, integration procedures, and operational troubleshooting for a complex enterprise payment platform.
- **Regulatory accuracy** — compliance configuration reflects current US requirements: OFAC SDN screening, BSA Travel Rule (31 CFR 103.33), FinCEN record-keeping, NACHA Operating Rules.
- **Enterprise documentation style** — structured topic architecture with admonitions (Note, Warning, Important, Tip), numbered procedures, decision tables, configuration parameter reference tables, and error code references. Mirrors the HTML documentation style used in MadCap Flare enterprise documentation environments.
- **Segregation of duty controls** — permissions matrix and workflow configuration reflect banking operational controls (dual-control, four-eyes principle) that are standard in regulated financial institutions.

---

## Background

This project draws on real-world experience:

- **Fundtech India (2008–2010):** Authored configuration and operations documentation for CashIn and Global CASHplus — enterprise payment hub systems that are direct predecessors of Finastra GlobalPAYplus. The product documented here is modeled on that class of system.
- **NICE Actimize (2017–present):** Enterprise HTML documentation (MadCap Flare) for financial crime prevention platforms, including compliance controls for payment monitoring — informing the OFAC and AML sections of this guide.

---

## Author

**Sulagna Sasmal**
Senior Specialist Technical Writer | Documentation Architect
19+ years in FinTech, enterprise SaaS, and financial systems documentation
[Portfolio](https://sulagnasasmal.github.io/sulagnasasmal-site/) · [GitHub](https://github.com/SulagnaSasmal) · [US Payments Hub](https://sulagnasasmal.github.io/us-payments-hub/)
