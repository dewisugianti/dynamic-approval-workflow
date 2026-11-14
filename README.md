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
| Different approval limits | Each department has its own approval limits. |
| Shared executive approvers | The same VP or CTO can approve transactions for different departments, with different limits for each. |
| Multi-level authorization | Higher-value transactions must be approved by multiple management levels in order. |
| Different budget ownership | Asset transactions must be approved by GA first, followed by the relevant department approvers. |
| Multiple transaction conditions | The system must check several transaction details to determine the correct approval route simultaneously. |

---

## 3. Business Requirements

### 3.1 Multi-Level Approval Hierarchy

The approval structure follows a sequential authorization model based on transaction amount and configured approval limits.

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

Approval limits can vary depending on the subaccount, even when the same approver is assigned.

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

This approach allows a single approver to manage authorization responsibilities across multiple departments without enforcing identical financial limits.

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

The process began with the approval matrix provided by the Finance team.
When a transaction is submitted, the system evaluates its attributes against the configured approval rules.
Example of Approval Matrix:

<p align="center">
  <a href="https://github.com/dewisugianti/dynamic-approval-workflow/blob/main/ApprovalMapMatrix.jpeg">Approval Matrix</a>
  <br>
</p>
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
  <a href="https://github.com/dewisugianti/dynamic-approval-workflow/blob/main/BF_approvalprocess.png">Transaction Approval Execution Flowchart</a>
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
| Next Approval | Relevant Department / Subaccount |

**Expected Approval Route**

<p align="center">
  <strong>GA Approval</strong> &rarr; <strong> Relevant Department's Approval Levels based on Subaccount and Amount</strong>
</p>

---

## 6. Technical Implementation

The implementation focuses on translating business authorization policies into configurable Acumatica ERP approval rules.

### Configuration Screenshots

<p align="center">
  <a href="https://github.com/dewisugianti/dynamic-approval-workflow/blob/main/ApprovalConfig.jpeg">Approval Map Configuration</a>
  <br>
</p>

<p align="center">
  <a href="https://github.com/dewisugianti/dynamic-approval-workflow/blob/main/Example_ConditionalLogic.jpeg">Conditional Approval Rules</a>
  <br>
  <em></em>
</p>

---

## 7. Key Contributions

My contributions to this project included:

- Reviewed approval requirements from the Finance team and converted them into system rules.
- Created AND/OR conditions based on transaction details and approval requirements.
- Set up approval limits and assigned approvers for different departments in Acumatica.
- Built multi-level approval flows, from SPV to higher management levels.
- Tested the approval setup to make sure it worked as expected.

---

## 8. Project Outcome

It supports:
- Different approval limits for each department.
- Step-by-step approval for higher-value transactions.
- Shared approvers with different approval limits for each department.
- Automatic approval routing based on transaction details and budget ownership.

The solution demonstrates how complex business authorization requirements can be translated into structured ERP workflow configurations.
The following screenshots show an example of a Purchase Order (PO) and the approval assignments generated based on the configured approval rules.

### Purchase Order Details
Example of a Purchase Order with transaction details used to determine the required approval flow. As shown, the Inventory ID is FA, so the transaction is routed to GA for initial approval. The Subaccount is 01, so it then follows Approval Map 01 based on the configured approval limits that cover the order total of approximately IDR 400 million.
<p align="center">
  <a href="https://github.com/dewisugianti/dynamic-approval-workflow/blob/main/ExamplePODetail.jpeg">Purchase Order Details</a>
  <br>
</p>

### Approval Assignment
The approval assignments generated by the system based on the transaction details and configured approval rules.
<p align="center">
  <a href="https://github.com/dewisugianti/dynamic-approval-workflow/blob/main/Result_ApprovalAssignmentInPO.jpeg">Approval Assignments</a>
  <br>
</p>

---

## Disclaimer

This documentation presents a generalized representation of the business requirements and implementation approach.

Company-specific information, actual approver identities, financial data, and internal configurations have been anonymized or replaced with illustrative examples to maintain confidentiality.

