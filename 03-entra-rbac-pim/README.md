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
| 01 | Role assignment baseline and drift check | Complete |
| 02 | Administrative Unit for HR-scoped User Admin | Complete |
| 03 | Role-assignable groups | Complete |
| 04 | Remaining matrix role assignments | Complete |
| 05 | Least-privilege negative testing | Planned |
| 06 | PIM license gap and compensating controls | Planned |
| 07 | Findings and updated matrix | Planned |

---

## Build Log

### Module 01: Role Assignment Baseline

**Goal:** Before building anything, check what is actually assigned in the tenant today and compare it to the [Lab 01 RBAC matrix](../01-entra-id-hybrid-identity/Lab01_RBAC_Matrix.xlsx).

**Plain-English summary:** The org chart said ten people should each have their own set of keys. In reality, only the three master-key holders have any keys at all. The "check out the master key only when needed" system from Lab 01 quietly disappeared when a license ran out, so the day-to-day admin now carries the master key permanently. And the one custom key that was built doesn't open the doors its paperwork says it does.

**What was checked and what was found:**

| Check | Expected (per Lab 01) | Actual (2026-11-06) |
|---|---|---|
| Audit log, RoleManagement, 30 days | Recent role changes visible | Preview view: **no results**. Classic view: 2 events (role add 11/3, role removal 11/5) |
| Global Administrator holders | 2 break-glass (permanent) + admin via PIM JIT | 2 break-glass + `mtully.admin`, **all permanent, no PIM** |
| Tenant-wide role assignment export | 10 matrix roles in use | **3 assignments total, all Global Admin** |
| IT Manager custom role | Assigned (PIM eligible); resets passwords, manages groups, views audit logs | **0 assignments**; only permission is updating group membership |
| PIM eligible / expired assignments | Global Admin + IT Manager eligible | **None, and no expired records.** PIM now shows "requires P2" |

**Evidence:**

![Preview audit view returns no results for role changes](./01_role_assignment_baseline/01-1a_audit_rolemanagement_30d_preview_no_results.png)

![Classic audit view shows the same two role changes the preview missed](./01_role_assignment_baseline/01-1b_audit_rolemanagement_30d_classic_2_events.png)

![Global Administrator: 3 assignments](./01_role_assignment_baseline/02-1_global_admin_count_3.png)

![Global Administrator holders: two break-glass accounts and mtully.admin, all permanent](./01_role_assignment_baseline/02-2_global_admin_assignments_3_users.png)

![IT Manager Role: no assignments](./01_role_assignment_baseline/03-1_it_manager_assignments_none.png)

![IT Manager Role: description claims audit log access; only permission is group membership update](./01_role_assignment_baseline/03-2_it_manager_description_permissions.png)

![PIM: eligible assignments gone, P2 required banner](./01_role_assignment_baseline/04-1_pim_eligible_none_p2_required.png)

![PIM: no expired assignment records either](./01_role_assignment_baseline/04-2_pim_expired_none.png)

**Skills demonstrated:** Privileged access review, design-vs-implementation drift analysis, cross-checking audit tooling instead of trusting an empty result, license-dependency risk analysis.

### Module 02: Administrative Unit for HR

**Goal:** Implement the matrix row "HR Generalist: User Administrator, scoped via Administrative Unit to HR-managed users only," so HR can manage HR accounts without touching IT, Finance, Sales, or admin accounts.

**Plain-English summary:** We built a fence around the HR department and gave one person a key that only works inside it. The fence fills itself: anyone whose department is set to HR is automatically inside, and anyone who transfers out automatically leaves. Nobody can be added by hand, so the fence can't quietly grow.

**What was built:**

| Item | Setting |
|---|---|
| Administrative Unit | `AU-HR-Managed-Users`, restricted management **No** |
| Membership | **Dynamic User**, rule `user.department -eq "HR"` (manual "Add member" disabled) |
| Members | 8 users, matching the 8 HR users from Lab 01 exactly |
| Role assignment | **User Administrator**, scope **AU-HR-Managed-Users** (not Directory), assigned to Alyssa Morgan |

**Design notes:**
- Dynamic membership ties the admin boundary to the HR attribute, so joiner/mover/leaver changes update the scope with no ticket. The tradeoff: whoever can edit a user's `department` attribute can move that user into or out of HR's control. That attribute is now security-relevant.
- The delegated admin (Alyssa, department Sales) is outside the AU, so she cannot use this role on her own account.
- **Lab shortcut, stated openly:** in production this role would go to an HR staff member. Alyssa was used because her MFA registration already existed, which avoided registering a new test user against CA01.
- The matrix specifies group-based assignment; this module used a direct assignment, which was converted to a role-assignable group in [Module 03](#module-03-role-assignable-groups).

**Evidence:**

![8 HR users in the directory before building the AU](./02_admin_unit_hr_scope/01-1_hr_users_8_found.png)

![Creating the AU: name, description, restricted management off](./02_admin_unit_hr_scope/02-1_au_create_properties.png)

![AU created](./02_admin_unit_hr_scope/02-2_au_created_success.png)

![Dynamic membership rule: department equals HR](./02_admin_unit_hr_scope/02-3_au_dynamic_rule_department_hr.png)

![AU properties: membership type Dynamic User](./02_admin_unit_hr_scope/02-4_au_properties_dynamic_user.png)

![AU members: the same 8 HR users, manual Add member disabled](./02_admin_unit_hr_scope/03-1_au_members_8_hr_users.png)

![User Administrator assigned to Alyssa Morgan at AU scope](./02_admin_unit_hr_scope/04-1_user_admin_assigned_alyssa_au_scope.png)

![Verified from the user side: User Administrator, resource AU-HR-Managed-Users, direct assignment](./02_admin_unit_hr_scope/05-1_alyssa_assigned_roles_user_admin_au_scope.png)

**Skills demonstrated:** Administrative Unit design, dynamic membership rules, scoped (AU-level) role delegation, verifying an assignment from both the role side and the user side.

### Module 03: Role-Assignable Groups

**Goal:** Implement the matrix's "Group-based" assignment method. Move the HR User Administrator role from Alyssa personally to a group, so access follows group membership instead of individual grants.

**Plain-English summary:** Instead of handing the HR key to a person, we attached it to a job badge. Whoever wears the "HR User Admins" badge gets the key; take the badge back and the key goes with it. Because this badge carries an admin key, it's a locked-down type: it has no owners, people can only be added by hand (never by a rule), and only top-level admins can change who's in it. Ordinary department groups can't carry admin keys at all.

**What was built:**

| Item | Setting |
|---|---|
| Group | `RG-HR-UserAdmins`, Security, **role-assignable: Yes** (permanent once set) |
| Membership | **Assigned** (forced; role-assignable groups can't be dynamic) |
| Owners | **None.** An owner could add members and grant themselves the role |
| Members | Alyssa Morgan |
| Role assignment | **User Administrator** at scope **AU-HR-Managed-Users**, assigned to the group |
| Direct assignment | Alyssa's direct assignment from Module 02 **removed** |

**Why role-assignable groups matter:** If an admin role could be assigned to an ordinary group, anyone who manages that group (an owner, a Groups Administrator, or a dynamic rule fed by an editable attribute) could grant admin rights without touching role management. Entra closes that path: only groups flagged role-assignable at creation can hold roles, their membership can only be changed by Privileged Role Administrators or Global Administrators, and they can't use dynamic membership. The role picker confirmed it: searching for the Lab 01 department group `SG-HR` returned **0 results**, with the note "Only groups eligible for role assignment are displayed."

**Evidence:**

![Before: only the four Lab 01 department groups exist](./03_role_assignable_groups/00-1_groups_before_4_lab01_groups.png)

![New group: role-assignable on, membership locked to Assigned, no owners](./03_role_assignable_groups/01-1_new_group_role_assignable_yes_assigned_no_owners.png)

![Group properties: role-assignable Yes, greyed out (cannot be changed)](./03_role_assignable_groups/02-1_group_properties_role_assignable_yes_locked.png)

![Role picker: SG-HR returns 0 results; only role-eligible groups are displayed](./03_role_assignable_groups/03-1_picker_sg_hr_0_results_users_tab.png)

![User Administrator at AU scope: group added alongside Alyssa's direct assignment](./03_role_assignable_groups/03-2_user_admin_assigned_group_and_alyssa_direct.png)

![Alyssa's direct assignment removed](./03_role_assignable_groups/04-1_alyssa_direct_assignment_removed.png)

![Only the group holds the role](./03_role_assignable_groups/04-2_user_admin_group_only.png)

![Alyssa still holds User Administrator at AU scope, now inherited via RG-HR-UserAdmins](./03_role_assignable_groups/05-1_alyssa_user_admin_inherited_via_group.png)

**Skills demonstrated:** Role-assignable group design, group-based role assignment at AU scope, removing standing direct grants, verifying inherited access from the user side, understanding of group-ownership privilege escalation paths.

### Module 04: Matrix Role Assignments

**Goal:** Implement the remaining roles from the [Lab 01 RBAC matrix](../01-entra-id-hybrid-identity/Lab01_RBAC_Matrix.xlsx), each through its own role-assignable group, and correct the IT Manager custom role so its description matches its permissions.

**Plain-English summary:** We cut the other six keys from the plan and put each one on its own locked badge, then fixed the label on the custom key so it says exactly what it opens. On Friday the building had three master keys and nothing else. Now every job in the plan has the one key it needs, and the master keys are still limited to three holders.

**What was built:**

| Group | Role | Matrix job title | Member | Scope |
|---|---|---|---|---|
| `RG-IT-HelpdeskAdmins` | Helpdesk Administrator | IT Support Specialist | Amanda Price (IT) | Directory |
| `RG-IT-CloudAppAdmins` | Cloud Application Administrator | Systems/Cloud Admin | Kevin Brooks (IT) | Directory |
| `RG-IT-Managers` | IT Manager Role (custom) | IT Manager | Daniel Foster (IT) | Directory |
| `RG-FIN-ReportsReaders` | Reports Reader | Financial Analyst | Brian Kim (Finance) | Directory |
| `RG-FIN-BillingAdmins` | Billing Administrator | Controller / Finance Director | Trevor Simmons (Finance) | Directory |
| `RG-HR-GroupsAdmins` | Groups Administrator | HR Director | Stephanie Hayes (HR) | Directory |
| `RG-SALES-LicenseAdmins` | License Administrator | VP of Sales | Eric Watson (Sales) | Directory |
| `RG-HR-UserAdmins` *(Module 03)* | User Administrator | HR Generalist | Alyssa Morgan | AU-HR-Managed-Users |

Every group: Security, role-assignable, Assigned membership, **no owners**, one member.

**IT Manager custom role corrected:** the description now reads "Manage membership of security and Microsoft 365 groups (excludes role-assignable groups). Does not reset passwords (Helpdesk Administrator) or read audit logs." Permissions were left unchanged (`microsoft.directory/groups/members/update` only); the role was right, the label was wrong. See [Finding 3](#findings-report--nist-800-53-control-mapping).

**Before and after (tenant-wide role assignment export):**

| | 2026-11-06 (Module 01) | 2026-11-08 (after Module 04) |
|---|---|---|
| Total role assignments | 3 | 11 |
| Global Administrator holders | 3 users | 3 users (unchanged) |
| Delegated matrix roles in use | 0 of 9 | **9 of 9** |
| Assigned through role-assignable groups | 0 | **8** |
| Direct user assignments outside Global Admin | 0 | 0 |

**Where the tenant differs from the matrix (documented, not hidden):**
- **Helpdesk Administrator** (matrix: "SG-IT group scope") and **License Administrator** (matrix: "SG-Sales group scope") were assigned at directory scope. Neither role can be limited to a group; the options are directory or Administrative Unit. A per-department AU, as built for HR in Module 02, would be the production fix.
- **Cloud Application Administrator** is flagged **PRIVILEGED** by Microsoft: it can add credentials to application registrations, which is a known escalation path. It is not covered by CA04 (admin MFA + 4-hour sign-in frequency), which targets only Global, Privileged Role, and Security Administrator. Evaluated in [Module 06](#module-06-pim-license-gap).
- The matrix lists most roles as "PIM eligible." PIM requires Entra ID P2, so all assignments are active. See [Finding 1](#findings-report--nist-800-53-control-mapping).

**Evidence:**

![RG-IT-HelpdeskAdmins: role-assignable, no owners, Helpdesk Administrator](./04_matrix_role_assignments/01-1_rg_it_helpdeskadmins_create.png)

![RG-IT-CloudAppAdmins: Cloud Application Administrator](./04_matrix_role_assignments/02-1_rg_it_cloudappadmins_create.png)

![IT Manager Role description corrected to match its single permission](./04_matrix_role_assignments/03-1_it_manager_description_corrected.png)

![RG-IT-Managers: IT Manager Role (custom)](./04_matrix_role_assignments/03-2_rg_it_managers_create.png)

![RG-FIN-ReportsReaders: Reports Reader](./04_matrix_role_assignments/04-1_rg_fin_reportsreaders_create.png)

![RG-FIN-BillingAdmins: Billing Administrator](./04_matrix_role_assignments/05-1_rg_fin_billingadmins_create.png)

![RG-HR-GroupsAdmins: Groups Administrator](./04_matrix_role_assignments/06-1_rg_hr_groupsadmins_create.png)

![RG-SALES-LicenseAdmins: License Administrator](./04_matrix_role_assignments/07-1_rg_sales_licenseadmins_create.png)

![All 8 role-assignable groups](./04_matrix_role_assignments/08-1_rg_groups_8_found.png)

**What happened:** A group-list check found 7 of 8 groups; `RG-IT-CloudAppAdmins` had not been created (the Create click was missed while moving to the IT Manager fix). It was recreated and the count re-verified at 8, and the tenant-wide export was used as the final check rather than the per-group screens.

**Skills demonstrated:** Translating an access design into implemented, group-based role assignments; custom role maintenance; before/after verification with a tenant-wide export; documenting where a platform can't express the design exactly.

### Module 05: Least-Privilege Testing

*Planned.*

### Module 06: PIM License Gap

*Planned.*

### Module 07: Findings and Matrix Update

*Planned.*

---

## Findings Report — NIST 800-53 Control Mapping

**Environment:** iampamprointheknow42gmail.onmicrosoft.com (Entra ID P1 trial)

**Finding 1 — Just-in-time admin access silently removed when the P2 license lapsed** *(High)*
*Control: NIST 800-53 AC-6 (Least Privilege), AC-2 (Account Management)*

Lab 01 configured PIM so Global Administrator and the IT Manager role required activation with MFA and justification. After the Entra ID P2 trial lapsed, all eligible assignments were removed, with no expired-assignment record in PIM and no RoleManagement audit event in the 30-day window. The daily admin account now holds Global Administrator permanently. Lab 01's Finding 1 ("standing privileged access replaced with JIT activation") no longer reflects the tenant. Compensating controls in place: CA04 (MFA + 4-hour sign-in frequency for admin roles) and two excluded break-glass accounts. Detail in [Module 06](#module-06-pim-license-gap).

**Finding 2 — RBAC matrix designed but not implemented: 1 of 10 roles in use** *(Medium)*
*Control: NIST 800-53 AC-2, AC-5 (Separation of Duties)*

A tenant-wide role assignment export returned three assignments, all Global Administrator. None of the nine delegated roles in the Lab 01 matrix (Helpdesk, Cloud App, Reports Reader, Billing, HR-scoped User Admin, Groups, License, IT Manager) has a holder. Without delegated roles, any admin task requires Global Administrator, which defeats the least-privilege design. Remediated in Modules 02–04.

**Finding 3 — IT Manager custom role does not match its documentation** *(Medium)*
*Control: NIST 800-53 AC-6(1) (Authorize Access to Security Functions)*

The Lab 01 README and matrix state the role resets passwords, manages group membership, and views audit logs. The role's own description says it manages groups and views audit logs but not passwords. Its only actual permission is updating group membership. Password reset was intentionally moved to Helpdesk Administrator during Lab 01, but the documentation was never updated. Access decisions made from the documentation would be wrong. Corrected in Module 04.

**Finding 4 — Preview audit log view returned no results for real role changes** *(Low)*
*Control: NIST 800-53 AU-6 (Audit Record Review, Analysis, and Reporting)*

With Category = RoleManagement and Date = Last 1 month, the default (preview) audit log experience returned "No results." The classic view, with identical filters, returned two role changes from the same week. A reviewer relying on the default view would conclude no privileged role changes occurred. Mitigation: verify empty audit results against a second view or an export before recording "no activity."

---

## Skills Summary

*To be completed.*

---

## Notes

Built on an Entra ID P1 trial. PIM requires Entra ID P2, which was not renewed for this lab; that gap is documented in Module 06 rather than hidden.

**Environment change (2026-11-08):** Admin work moved from a Firefox private window to a dedicated Firefox profile (`IAM-Admin`). Both keep the admin session separate from personal accounts; the profile also keeps portal settings between sessions, which removed repeated sign-ins and theme resets. Screenshots before this date mix light and dark themes for that reason. Because the profile stays signed in, the workstation is locked when unattended and the session is signed out at the end of each lab day.
