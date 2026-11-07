# Lab 03: Entra RBAC & PIM — Putting the Role Matrix Into Practice

![Status](https://img.shields.io/badge/status-in%20progress-yellow)
![Platform](https://img.shields.io/badge/platform-Microsoft%20Entra%20ID-blue)
![License](https://img.shields.io/badge/license-Entra%20ID%20P1-lightgrey)

## Contents

- [Overview](#overview)
- [Objective](#objective)
- [Status](#status)
- [Build Log](#build-log)
  - [Module 01: Role Assignment Baseline](#module-01-role-assignment-baseline)
  - [Module 02: Administrative Unit for HR](#module-02-administrative-unit-for-hr)
  - [Module 03: Role-Assignable Groups](#module-03-role-assignable-groups)
  - [Module 04: Matrix Role Assignments](#module-04-matrix-role-assignments)
  - [Module 05: Least-Privilege Testing](#module-05-least-privilege-testing)
  - [Module 06: PIM License Gap](#module-06-pim-license-gap)
  - [Module 07: Findings and Matrix Update](#module-07-findings-and-matrix-update)
- [Findings Report — NIST 800-53 Control Mapping](#findings-report--nist-800-53-control-mapping)
- [Skills Summary](#skills-summary)
- [Notes](#notes)

---

## Overview

[Lab 01](../01-entra-id-hybrid-identity/README.md) designed a 10-role RBAC matrix ([`Lab01_RBAC_Matrix.xlsx`](../01-entra-id-hybrid-identity/Lab01_RBAC_Matrix.xlsx)) but implemented only one role from it (the IT Manager custom role) plus PIM on two roles. This lab closes the gap between design and reality: it checks what is actually assigned in the tenant today, builds the remaining matrix roles with proper scoping (Administrative Units, role-assignable groups), and proves the boundaries hold with negative tests.

It also documents what happened to PIM after the Entra ID P2 trial lapsed, since a JIT control that silently disappears is a real-world governance risk.

**Environment:** Microsoft Entra ID P1 (trial), single cloud-only tenant
**Compliance mapped:** NIST 800-53 AC-2, AC-3, AC-5, AC-6, AU-6

---

## Objective

Show that a documented least-privilege role design is implemented as written, scoped correctly, and verified by testing — not just drawn up in a spreadsheet.

---

## Status

| Module | Topic | Status |
|---|---|---|
| 01 | Role assignment baseline and drift check | In progress |
| 02 | Administrative Unit for HR-scoped User Admin | Planned |
| 03 | Role-assignable groups | Planned |
| 04 | Remaining matrix role assignments | Planned |
| 05 | Least-privilege negative testing | Planned |
| 06 | PIM license gap and compensating controls | Planned |
| 07 | Findings and updated matrix | Planned |

---

## Build Log

### Module 01: Role Assignment Baseline

*In progress.*

### Module 02: Administrative Unit for HR

*Planned.*

### Module 03: Role-Assignable Groups

*Planned.*

### Module 04: Matrix Role Assignments

*Planned.*

### Module 05: Least-Privilege Testing

*Planned.*

### Module 06: PIM License Gap

*Planned.*

### Module 07: Findings and Matrix Update

*Planned.*

---

## Findings Report — NIST 800-53 Control Mapping

*Findings will be added as modules are completed.*

---

## Skills Summary

*To be completed.*

---

## Notes

Built on an Entra ID P1 trial. PIM requires Entra ID P2, which was not renewed for this lab; that gap is documented in Module 06 rather than hidden.
