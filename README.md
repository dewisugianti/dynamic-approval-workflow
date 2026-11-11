# Dynamic Multi-Level Approval Workflow Configuration

<p align="center">
  <strong>Acumatica ERP | Business Process Analysis | Workflow Configuration</strong>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Platform-Acumatica%20ERP-2563EB?style=flat-square" alt="Platform: Acumatica ERP">
  <img src="https://img.shields.io/badge/Focus-ERP%20Configuration-64748B?style=flat-square" alt="ERP Configuration">
  <img src="https://img.shields.io/badge/Category-Business%20Process-0F766E?style=flat-square" alt="Business Process">
</p>

---

## Table of Contents

- [1. Project Overview](#1-project-overview)
- [2. Business Background](#2-business-background)
- [3. Business Requirements](#3-business-requirements)
- [4. Proposed Business Process](#4-proposed-business-process)
- [5. Approval Logic Examples](#5-approval-logic-examples)
- [6. Technical Implementation](#6-technical-implementation)
- [7. Key Contributions](#7-key-contributions)
- [8. Project Outcome](#8-project-outcome)

---

## 1. Project Overview

This project focuses on designing and configuring a **Dynamic Multi-Level Approval Workflow** in Acumatica ERP to support complex financial authorization policies across multiple departments.

The Finance team provided the approval requirements in the form of an authorization matrix, specifying transaction amount ranges, applicable subaccounts, and designated approvers.

Based on these business requirements, I analyzed and translated the approval matrix into structured system conditions, determining the appropriate AND/OR logic, approval assignments, and sequential approval hierarchy within Acumatica ERP.

### Project Objectives

- Standardize financial authorization across departments.
- Support different approval limits for each subaccount.
- Automate multi-level approval routing.
- Accommodate complex business rules through conditional logic.
  
---

## 2. Business Background

### Business Requirements Input

The Finance team provided a predefined approval matrix containing:

- Transaction amount ranges.
- Applicable subaccounts.
- Designated approvers for each authorization range.
- Specific approval requirements based on inventory classification and order type.

However, the business requirements were provided in a tabular format and needed to be translated into system-compatible conditional logic.

The main challenge was to interpret the relationships between different approval criteria and determine how they should be structured using AND/OR conditions to achieve the intended approval behavior.

### Key Business Challenges

| Challenge | Business Requirement |
|---|---|
| Different approval thresholds | Each department requires configurable financial limits. |
| Shared executive approvers | The same VP or CTO may oversee multiple departments with different authorization limits. |
| Multi-level authorization | Higher-value transactions require sequential approval from multiple management levels. |
| Different budget ownership | Asset-related inventory must follow the General Affairs (GA) budget approval structure. |
| Complex transaction conditions | Multiple transaction attributes must be evaluated simultaneously. |

### Business Objective

To establish a configurable approval framework that translates financial authorization policies into automated ERP approval workflows.

---

## 3. Business Requirements

### 3.1 Multi-Level Approval Hierarchy

The approval structure follows a sequential authorization model based on transaction amount and configured approval thresholds.

**Example: Subaccount 01**

| Approval Level | Assigned Approver | Authorization Limit |
|---|---|---:|
| Supervisor (SPV) | Approver A | Rp0 – Rp15M |
| Manager | Approver B | >Rp15M – Rp50M |
| Vice President (VP) | Approver C | >Rp50M – Rp300M |
| Chief Technology Officer (CTO) | Approver D | >Rp300M |

Approval requirements are cumulative.

For example, a transaction amounting to Rp60M under Subaccount 01 requires the following approval sequence:

<p align="center">
  <strong>SPV A</strong> &rarr; <strong>Manager B</strong> &rarr; <strong>VP C</strong>
</p>

Each approval level must be completed before the transaction proceeds to the next required approver.

### 3.2 Department-Specific Authorization Limits

Approval thresholds can vary depending on the subaccount, even when the same approver is assigned.

**Example:**

| Subaccount | Approval Level | Approver | Authorization Limit |
|---|---|---|---:|
| 01 | SPV | A | Rp0 – Rp15M |
| 01 | Manager | B | >Rp15M – Rp50M |
| **01** | **VP** | **C** | **>Rp50M – Rp300M** |
| 02 | SPV | E | Rp0 – Rp15M |
| 02 | Manager | F | >Rp15M – Rp25M |
| **02** | **VP** | **C** | **>Rp25M – Rp300M** |
| **03** | **VP** | **C** | **>Rp50M – Rp300M** |

This approach allows a single approver to manage authorization responsibilities across multiple departments without enforcing identical financial thresholds.

### 3.3 Budget Ownership-Based Routing

Inventory classification determines the applicable budget ownership and approval structure.

| Inventory Classification | Budget Ownership | Approval Route |
|---|---|---|
| Asset | General Affairs (GA) | GA Approval Matrix |
| Non-Asset | Relevant Department | Department Approval Matrix |

Asset-related transactions are routed to the GA budget approval structure rather than the standard departmental approval route.

### 3.4 Conditional Approval Rules

The system evaluates multiple transaction attributes through grouped AND/OR conditions.

**AND Condition**

All specified conditions within a rule must be satisfied.

Example:

```text
(Subaccount = 01
AND Transaction Amount > 50M
AND Transaction Amount <= 300M)
```

**OR Condition**

Alternative condition groups can independently satisfy an approval rule.

Example:

```text
(
    (Subaccount = 01
    OR Subaccount = 03)
    AND Amount > 50M
    AND Amount <= 300M
)
OR
(
    Subaccount = 02
    AND Amount > 25M
    AND Amount <= 300M
)
```

This allows the same approver to be assigned to different departments with different authorization conditions.

---

## 4. Proposed Business Process

### 4.1 Business Requirement Analysis & Approval Logic Design

The process began with the approval matrix provided by the Finance team.

My responsibilities included:

   1. Reviewing the approval matrix and identifying its key attributes.
   2. Analyzing the relationship between transaction amount ranges, subaccounts, and designated approvers.
   3. Identifying conditions that require AND logic and alternative conditions that require OR logic.
   4. Mapping the approval requirements into structured conditional rules.
   5. Defining the sequential approval hierarchy.
   6. Translating the designed logic into Acumatica ERP approval configurations.
   7. Validating the configuration against the business requirements.

### 4.2 Transaction Approval Execution

When a transaction is submitted, the system evaluates its attributes against the configured approval rules.

**Process Flow:**

1. User creates and submits a transaction.
2. System identifies inventory classification.
3. System determines the applicable budget owner or department.
4. Transaction attributes are evaluated against configured conditions.
5. System identifies the required approval levels.
6. Transaction is routed to the first designated approver.
7. Upon approval, the transaction proceeds to the next required approval level.
8. The document reaches its final approval status after all required approvals are completed.

<p align="center">
  <img src="diagrams/transaction-approval-flow.png" alt="Transaction Approval Execution Flowchart" width="850">
</p>

---

## 5. Approval Logic Examples

The following scenarios demonstrate how different business rules determine approval assignments and routing.

### Scenario 1 — Standard Sequential Approval

**Transaction Details**

| Attribute | Value |
|---|---|
| Subaccount | 01 |
| Transaction Amount | Rp60M |
| Inventory Classification | Non-Asset |

**Expected Approval Sequence**

<p align="center">
  <strong>SPV A</strong> &rarr; <strong>Manager B</strong> &rarr; <strong>VP C</strong>
</p>

### Scenario 2 — Department-Specific Approval

**Transaction Details**

| Attribute | Value |
|---|---|
| Subaccount | 02 |
| Transaction Amount | Rp30M |
| Assigned Approver | VP C |

**Expected Approval Sequence**

<p align="center">
  <strong>SPV E</strong> &rarr; <strong>Manager F</strong> &rarr; <strong>VP C</strong>
</p>


### Scenario 3 — Asset-Based Approval Routing

**Transaction Details**

| Attribute | Value |
|---|---|
| Inventory Classification | Asset |
| Budget Ownership | General Affairs |

**Expected Approval Route**

<p align="center">
  <strong>GA Budget</strong> &rarr; <strong>GA Approval Matrix</strong>
</p>

---

## 6. Technical Implementation

The implementation focuses on translating business authorization policies into configurable Acumatica ERP approval rules.

### Requirement-to-System Logic Mapping

The provided approval matrix was not directly translated into a one-to-one system configuration.

It required further analysis to identify how multiple criteria should interact within the approval workflow.

Key implementation activities included:

- Interpreting business-defined authorization ranges and approver assignments.
- Identifying criteria that must be evaluated together using AND conditions.
- Grouping alternative approval scenarios using OR conditions.
- Structuring conditional rules to accommodate different subaccount thresholds.
- Translating the resulting logic into Acumatica Approval Map configurations.
- Ensuring that the configured approval hierarchy reflected the intended business authorization process.

### Configuration Screenshots

<p align="center">
  <img src="screenshots/approval-map.png" alt="Acumatica Approval Map Configuration" width="850">
  <br>
  <em>Approval Map Configuration</em>
</p>

<p align="center">
  <img src="screenshots/approval-conditions.png" alt="Approval Conditions Configuration" width="850">
  <br>
  <em>Conditional Approval Rules</em>
</p>

<p align="center">
  <img src="screenshots/approval-assignment.png" alt="Approver Assignment Configuration" width="850">
  <br>
  <em>Approver Assignment Configuration</em>
</p>

---

---

## 7. Key Contributions

My contributions to this project included:

- Analyzed Finance-provided approval matrices and translated tabular business requirements into structured ERP approval logic.
- Independently designed AND/OR conditional groupings based on the relationships between transaction attributes.
- Mapped department-specific authorization thresholds and approver assignments into Acumatica Approval Maps.
- Designed sequential approval routing to support multi-level financial authorization.
- Validated system configurations against the original business requirements.

---

## 8. Project Outcome

The designed approval framework provides a structured approach to managing financial authorization across multiple departments.

It supports:

- Flexible approval thresholds based on departmental requirements.
- Sequential authorization for higher-value transactions.
- Shared executive approvers with differentiated authorization limits.
- Conditional routing based on transaction attributes and budget ownership.
- Configurable approval policies maintained by the Finance team.

The solution demonstrates how complex business authorization requirements can be translated into structured ERP workflow configurations.

---

## Disclaimer

This documentation presents a generalized representation of the business requirements and implementation approach.

Company-specific information, actual approver identities, financial data, and internal configurations have been anonymized or replaced with illustrative examples to maintain confidentiality.

