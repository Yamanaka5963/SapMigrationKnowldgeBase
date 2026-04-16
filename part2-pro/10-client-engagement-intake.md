# 10 - Client Engagement Intake Checklist

Use this checklist when receiving a new SAP migration project from a client. The goal is to gather enough information to scope the project and determine the migration approach before the assessment phase.

## 1. Client System Information

- [ ] Current SAP version (ECC 6.0, EHP level, SP level)
- [ ] Current database type and version (Oracle, MSSQL, DB2, etc.)
- [ ] Current OS (AIX, Linux, Windows)
- [ ] Number of application servers / system landscape (DEV/QAS/PRD)
- [ ] System size — DB size, number of users, active company codes
- [ ] Any existing add-ons or ISV solutions installed
- [ ] Unicode or non-Unicode system

## 2. Business Context

- [ ] Target go-live date (hard deadline or flexible?)
- [ ] Budget range / approved budget
- [ ] Internal IT team capacity (can they support migration tasks?)
- [ ] Any planned business changes (M&A, restructuring) that affect scope
- [ ] Target deployment: On-Premise, Private Cloud (RISE), or Public Cloud?
- [ ] Any regulatory or compliance constraints (data residency, audit, etc.)

## 3. Scope & Complexity Indicators

- [ ] Volume of custom ABAP (Z-programs, Z-tables, enhancements)
- [ ] Number of interfaces and integrations (RFC, IDocs, APIs, middleware)
- [ ] Active business processes / modules in use (FI, CO, MM, SD, PP, HR, etc.)
- [ ] History of custom modifications to SAP standard (modifications vs. enhancements)
- [ ] Any third-party tools in the landscape (MES, WMS, BI/BW, etc.)

## 4. Preliminary Output

After intake, you should be able to:
- Identify candidate migration approach (Brownfield / Greenfield / Shell) → see [12 - Approach Selection Framework](12-approach-selection-framework.md)
- Estimate Readiness Check effort
- Propose next steps: Readiness Check run, landscape workshop, or fit-gap analysis

## 5. Key Contacts to Establish

- [ ] Client project sponsor
- [ ] Client IT lead / basis team contact
- [ ] Client business process owner(s) per module
- [ ] SAP account executive (if RISE with SAP is in scope)
